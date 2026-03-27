---
name: attendance
description: |
  東京大学の勤務時間管理システム（CWS）で打刻漏れの勤務状況メモ登録と月次申請を自動化するスキル。agent-browserを使用してブラウザ操作を行う。
  Use when: (1) 打刻漏れのメモを登録したい、(2) 月次申請・確定をしたい、(3) 勤怠の打刻漏れを修正したい、(4) 勤務表を確認したい、(5) CWSを操作したい
  Trigger: 打刻漏れ, 勤怠, 月次申請, 月次確定, 勤務表, 出勤, 退勤, attendance, CWS, 勤務状況メモ
---

# 東京大学 勤務時間管理システム（CWS）自動化

東京大学の勤務時間管理システム（CWS: https://ut-ppsweb.adm.u-tokyo.ac.jp/cws/cws ）にアクセスし、打刻漏れの勤務状況メモ登録および月次申請を自動化する。

## Prerequisites

- **UTokyo VPN接続**: 学内ネットワークまたはVPN接続が必要
- **agent-browser**: ブラウザ自動化CLIツール（Chromiumインストール済みであること）
- **Microsoft認証**: UTokyo Accountでのログインが必要（初回のみ手動）

### Chromiumのセットアップ

agent-browserで`Executable doesn't exist`エラーが出る場合、Chromiumをインストールする：

```bash
cd /opt/homebrew/lib/node_modules/agent-browser && npm install
PLAYWRIGHT_BROWSERS_PATH=/opt/homebrew/lib/node_modules/agent-browser/node_modules/playwright-core/.local-browsers \
  npx --prefix /opt/homebrew/lib/node_modules/agent-browser playwright install chromium
```

## 重要な制約

1. **Playwrightの内蔵Chromiumはシステム VPN を経由しないことがある**。接続エラー（`net::ERR_SOCKET_NOT_CONNECTED`）が出る場合は `--headed` オプションで再試行する（一時的な接続問題のことが多い）
2. **Microsoft認証は手動が必要**: 初回アクセス時にMicrosoftログインページにリダイレクトされる。ユーザーにブラウザでログインしてもらう
3. **`--headed` モードを使用する**: ユーザーがログイン操作を行えるよう、ヘッドレスではなくheadedモードでブラウザを起動する
4. **`select`コマンドはrefを壊すことがある**: ドロップダウンの選択には`agent-browser select`ではなく**JavaScript eval**を使うこと（後述）
5. **バッチ処理でのrefキャッシュ**: シェルスクリプトでループ処理する場合、各アクション後に必ず`snapshot -i`でrefを再取得すること。バックグラウンド実行やパイプ処理ではrefが古くなり失敗する
6. **セッションタイムアウト**: 長時間操作しないとセッションが切れる。「再ログイン」リンクが表示されたらクリックして再認証する

## サイト構造

```
CWSトップ → Microsoft認証（SAML）→ メインメニュー
  ├── 勤務表           → 月次の打刻記録一覧・月次申請
  ├── 承認者設定
  ├── 就労申請         → 各種申請メニュー
  │   ├── 勤務状況メモ入力  → 日付指定でメモ登録
  │   ├── 勤務状況メモ取消
  │   ├── 超過勤務(終業時刻後)
  │   └── ...
  └── 就労申請処理状況確認
```

## ワークフロー

### Step 1: サイトにアクセス・ログイン

```bash
agent-browser --headed open https://ut-ppsweb.adm.u-tokyo.ac.jp/cws/cws
```

- Microsoft認証ページにリダイレクトされたら、ユーザーにログインを依頼する
- ログイン完了後、メインメニューが表示される（「勤務表」「就労申請」等のリンク）

### Step 2: 勤務表で打刻漏れを確認

1. メインメニューから「**勤務表**」をクリック
2. 当月の勤務表が表示される
3. **スクリーンショットではなくJavaScript evalでテーブルデータを抽出する**（画像では打刻時間を正確に読み取れないため）

```bash
agent-browser eval "
const rows = document.querySelectorAll('table tr');
const results = [];
rows.forEach(r => {
  const cells = r.querySelectorAll('td');
  if (cells.length >= 5) {
    const day = cells[0]?.innerText?.trim();
    const dow = cells[1]?.innerText?.trim();
    const kind = cells[2]?.innerText?.trim();
    const stamp = cells[3]?.innerText?.trim();
    if (day && day.match(/^\\d+\\//)) {
      results.push(day + ' ' + dow + ' ' + kind + ' | 打刻: [' + stamp + ']');
    }
  }
});
results.join('\\n');
"
```

4. 出力から打刻漏れを特定する

**打刻漏れのパターン**:

| 打刻時間の表示 | 意味 |
|---|---|
| `09:24 -- 18:01` | 出勤・退勤両方あり（正常） |
| `09:24 --` | 出勤のみ、**退勤打刻漏れ** |
| `-- 18:04` | 退勤のみ、**出勤打刻漏れ** |
| （空白） | **出勤・退勤両方打刻漏れ** |

**勤務内容の判定**:
- 「出勤日」と表示されている日のみ対象
- 「法定休日」「所定休日」「勤務日でない日」は対象外

### Step 3: ユーザーに打刻漏れ一覧と方針を確認

打刻漏れのある日をリストアップし、ユーザーに以下を確認する：
- 打刻漏れ一覧の正確性
- **未来の日付は除外する**（まだ勤務していないため）
- **出張情報がある日は除外する**（出張は打刻不要の場合がある）
- メモに記載する出勤・退勤時刻（例: 出勤漏れは一律9:29、退勤漏れは一律18:01 など）

### Step 4: 勤務状況メモを登録

1. 「**就労メインページ**」→「**就労申請**」→「**勤務状況メモ入力**」に遷移
2. フォームの構造（ref番号は都度 `snapshot -i` で確認すること）:
   - 年テキストボックス（通常変更不要）
   - 月ドロップダウン（combobox）
   - 日ドロップダウン（combobox, nth=1）
   - 勤務状況メモテキストボックス（nth=1）
   - 「次へ」ボタン
3. **各日を1件ずつ、個別のコマンドで処理する**（バッチ/ループ処理はrefの不整合で失敗しやすいため）

#### 日付ドロップダウンの選択方法（重要）

**`agent-browser select` コマンドはドロップダウン操作後にrefが無効化される問題がある。代わりにJavaScript evalで値をセットすること：**

```bash
# NG: selectコマンドはrefを壊す
# agent-browser select @e18 "05"

# OK: JavaScript evalで直接値をセット
agent-browser eval "document.querySelectorAll('select')[1].value = '05'; document.querySelectorAll('select')[1].dispatchEvent(new Event('change'))"
```

`document.querySelectorAll('select')[1]` は日ドロップダウン（2番目のselect要素）を指す。月ドロップダウンは `[0]`。

#### 1件のメモ登録フロー

```bash
# 1. 日付をJS evalでセット
agent-browser eval "document.querySelectorAll('select')[1].value = '{日}'; document.querySelectorAll('select')[1].dispatchEvent(new Event('change'))"

# 2. snapshot -i でrefを再取得（eval後はrefがリセットされている可能性あり）
agent-browser snapshot -i
# → メモテキストボックスとボタンのrefを確認

# 3. メモを入力して「次へ」
agent-browser fill @e51 "{メモ内容}"   # ref番号は都度確認
agent-browser click @e52               # ref番号は都度確認
agent-browser wait --load networkidle

# 4. 確認画面で「送信」
agent-browser snapshot -i
agent-browser click @e4                # 「送信」ボタン（ref番号は都度確認）
agent-browser wait --load networkidle

# 5. 完了画面から「就労申請」に戻る
agent-browser snapshot -i
agent-browser click @e4                # 「就労申請」リンク（ref番号は都度確認）
agent-browser wait --load networkidle

# 6. 「勤務状況メモ入力」をクリックして次の日へ
agent-browser snapshot -i
agent-browser click @e3                # 「勤務状況メモ入力」リンク（ref番号は都度確認）
agent-browser wait --load networkidle
```

#### バッチ処理の注意点

- **バックグラウンド実行やシェルスクリプトのループは避ける**。agent-browserのデーモンに対する並行アクセスで`Resource temporarily unavailable`エラーが発生する
- 各日を**1件ずつ順番にコマンドを実行**する。各ステップの間で必ず`snapshot -i`でrefを再取得する
- 失敗した場合は、まず現在のページ状態を`snapshot -i`で確認してからリトライする

**メモ内容のフォーマット**:
- 出勤漏れ: `打刻漏れ。実際は{時刻}に出勤`
- 退勤漏れ: `打刻漏れ。実際は{時刻}に退勤`
- 両方漏れ: `打刻漏れ。実際は{出勤時刻}に出勤、{退勤時刻}に退勤`

**注意**: ref番号（@e1, @e2...）はページ遷移のたびに変わる。必ず各ページで `snapshot -i` を実行して最新のrefを取得すること。

### Step 5: 月次申請（確定）

1. 「**勤務表**」ページに移動
2. 「**月次申請**」ボタンをクリック
3. 確認画面で全日分の勤務データとメモが正しいことを確認
4. 「**確定**」ボタンをクリック
5. 「勤務実績を提出しました」のメッセージを確認

```bash
# 勤務表ページで月次申請ボタンをクリック
agent-browser snapshot -i
agent-browser click @e11  # 月次申請ボタン（ref番号は都度確認）
agent-browser wait --load networkidle

# 確認画面で確定
agent-browser snapshot -i
agent-browser click @e4  # 確定ボタン（ref番号は都度確認）
agent-browser wait --load networkidle
```

## ナビゲーションパス

| 操作 | 遷移先 |
|------|--------|
| メインメニュー → 勤務表 | 月次打刻一覧 |
| メインメニュー → 就労申請 | 申請メニュー一覧 |
| 就労申請 → 勤務状況メモ入力 | メモ入力フォーム |
| メモ入力 → 次へ | 確認画面 |
| 確認画面 → 送信 | 完了画面 |
| 完了画面 → 就労申請 | 申請メニューに戻る |

## トラブルシューティング

### Chromiumが見つからない（Executable doesn't exist）
```bash
cd /opt/homebrew/lib/node_modules/agent-browser && npm install
PLAYWRIGHT_BROWSERS_PATH=/opt/homebrew/lib/node_modules/agent-browser/node_modules/playwright-core/.local-browsers \
  npx --prefix /opt/homebrew/lib/node_modules/agent-browser playwright install chromium
```

### 接続エラー（ERR_SOCKET_NOT_CONNECTED）
- VPN接続を確認
- `agent-browser close` してから `--headed` で再試行
- `curl -s -o /dev/null -w "%{http_code}" https://ut-ppsweb.adm.u-tokyo.ac.jp/cws/cws` で疎通確認（302なら正常）

### ブラウザが見つからない
- `--headed` オプションを付けているか確認
- 既存セッションがある場合は `agent-browser close` してから再起動

### ref番号が無効（Unsupported token "@eXX"）
- ページ遷移後は必ず `agent-browser snapshot -i` で新しいrefを取得
- フォーム送信後のページ変更で古いrefは無効になる
- **`select`コマンドの実行後もrefが壊れる**。selectの代わりにJS evalを使うこと

### Resource temporarily unavailable（os error 35）
- agent-browserデーモンへの並行アクセスが原因
- **バッチ処理やバックグラウンド実行を避ける**
- 発生した場合は数秒待ってから `snapshot -i` で状態を確認し、リトライする

### セッションタイムアウト
- 長時間操作しないとCWSのセッションが切れる
- `snapshot -i` で「再ログイン」リンクが表示されたらクリックする
- Microsoft認証のセッションが残っていれば自動でメインメニューに戻る

### 誤って不要なメモを登録してしまった場合
- 「就労申請」→「勤務状況メモ取消」から取り消し可能
- ドロップダウンから対象のメモ（番号と日付で特定）を選択して取り消す
