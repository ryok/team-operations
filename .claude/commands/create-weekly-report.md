# Weekly Report Creation Command

## Description

Automatically creates a weekly report for the ML System Development joint-research project by **copying last week’s report as the base**, then **updating the content to reflect changes (diffs) detected from Notion** for the current week.  
確証が薄い情報は、後で人間が確認しやすいよう **`<確認してください>`** を末尾に付けます。  
***The command must finish by creating a brand-new page in Notion, saving the final report there, and filling the `MailBody` property.***

## Usage

```bash
create-weekly-report [week-date]
```

| Parameter   | Type   | Description                                                                                     |
| ----------- | ------ | ----------------------------------------------------------------------------------------------- |
| `week-date` | string | Week ending date in `YYYY-MM-DD` format.<br>If omitted, the current week is used automatically. |

---

## What this command does

0. **Baseline Retrieval（先週版の取得）**  
   * タイトルパターン `【週次報告】<monday-of-week>週 共同研究 MLシステム開発` に合致する**直近の先週版**を検索し、本文（Markdown）と `MailBody` を取得。  
   * 今週版は**先週版の内容をコピーして開始**し、そこに差分を反映させて「更新」する。  
   * ベースにできる先週版が存在しない場合のみ、テンプレートから新規作成。

1. **Information Collection（今週の素材収集）**
   * 指定週範囲で以下を検索：
     - *Engineering Meeting Management* DB
     - *AI Coding* project-task DB
     - 追加ページ：  
       - https://www.notion.so/AI-PJ-Task-14d4bc176b2b804696e5e9ecb171db55?pvs=9  
       - https://www.notion.so/2424bc176b2b80f4a3a9c33c7b6d481f?v=2424bc176b2b80aa94d8000ce20fc39f
   * ミーティングノートからの進捗を抽出。

2. **Content Updating（内容の更新方針）**  
   * 先週版の本文をベースにし、今週の収集結果を照合して**最新の内容に更新する**。  
   * 更新規則：
     - 新規に発生した事柄 → **追記**  
     - 状況が変わった事柄 → **本文を直接更新（上書き）**  
     - 中止/撤回/完了した事柄 → **本文から削除 or 「完了」と記載変更**
   * **確証が薄い**と判断される項目には末尾に **`<確認してください>`** を付与。  
     - 例：情報ソースが曖昧／未確定な予定／相互矛盾など。

3. **Confidence & Source Tagging（根拠明示の簡易タグ）**  
   * 更新内容の末尾に出典簡易タグを付ける：  
     - `[src: MeetingNote <yyyy-mm-dd>]`, `[src: AI Coding DB#<task-id>]` 等。  
   * 出典が不明確、推定を含む場合は **`<確認してください>`** を必ず付与。

4. **Report Generation（生成）**
   * 各セクションは「更新済みの最新状態」として出力。  
   * 冒頭に**「今週の変更サマリー」**を挿入：
     ```markdown
     ### 今週の変更サマリー
     - 新規: X件 / 更新: Y件 / 完了: Z件
     - 重要トピック: …（3行以内）
     ```
   * `MailBody` も同様に更新済み本文を反映。

5. **Report Publication（Notion 発行）**
   * **Page name**:
     ```
     【週次報告】<monday-of-week>週 共同研究 MLシステム開発
     ```
     - `<monday-of-week>` はその週の月曜日（例: `2025-06-30`）
   * **Database**: Engineering Meeting Management (`12d4bc17-6b2b-80be-907d-e7a2f457658e`)
   * **Properties**
     | Property          | Value                            |
     | ----------------- | -------------------------------- |
     | `date`            | `<week-ending-date>`             |
     | `type`            | `週報`                             |
     | `email recipient` | `松尾先生`（必要に応じて変更）            |
     | `MailBody`        | 更新済み本文（下記テンプレに準拠） |
     | `PrevReport`      | 先週版のNotionページ（Relation）※新設推奨
   * **Content**  
     * ページ本文と `MailBody` に**同一の最終文面**を保存。

6. **Final Recording（最終確認）**
   * 新規ページの存在、プロパティ、本文/`MailBody` の整合を検証。  
   * 更新件数（新規 / 更新 / 完了）と `<確認してください>` 件数をログに記録。

---

## Required Templates

### 1. `MailBody` Field & Page Header

```text
松尾先生
各位
お疲れ様です。共同研究 MLシステム開発の岡田です。
<WEEK> の進捗報告をさせていただきます。
```

**更新規則**  
- 本文は「最新の状態」となるよう更新。  
- 確証が薄い箇所には行末に ` <確認してください>` を付与。例：  
  `推論APIのスループット計測 500rps→700rps（暫定値） [src: MeetingNote 2025-09-18] <確認してください>`

### 2. Report Body (Markdown)

```markdown
## 0. 今週の変更サマリー
- 新規: X件 / 更新: Y件 / 完了: Z件
- 重要トピック: …

## 1. プロジェクト推進と仕組み化

### 1.1 個別プロジェクト
#### 1.1.1 日テレ
• 目的: ...  
• 進捗: ... （更新後の最新情報を反映）

#### 1.1.2 ダイセル
• 目的: ...  
• 進捗: ...

#### 1.1.3 東光電気工事
• 目的: ...  
• 進捗: ...

#### 1.1.4 イトーキ
• 目的: ...  
• 進捗: ...

#### 1.1.5 PwC
• 目的: ...  
• 進捗: ...

### 1.2 仕組み化（AIコーディング）
• 目的: ...  
• 計画: ...  
• 進捗・状況: ...

## 2. 規模の拡大
### 2.1 採用
• 目的: ...  
• 進捗: ...

### 2.2 開発会社とのアライアンス
• 目的: ...  
• 進捗: ...

## 3. 差別化・武器作り
• 目的: ...  
• 進捗: ...

## 4. その他
• ...
```
**注**: 本文は常に「更新済みの最新状態」で残し、差分はサマリーに集約する。

---

## Implementation Tasks

1. Use **TodoWrite** to plan weekly-report tasks.
2. Query Notion DBs for the specified week.  
3. **Fetch last week’s report**（先週版）。見つからない場合はテンプレを使用。
4. **Update content**：差分を反映して本文を最新化。  
5. **Generate the report**：更新済み本文＋変更サマリーを生成。  
6. **Publish to Notion**：指定命名・DB・プロパティで作成し、本文と `MailBody` を保存。  
7. **Finalize**：件数チェック（新規 / 更新 / 完了、`<確認してください>`）とリンク整合性を検証、Todo を完了に更新。
