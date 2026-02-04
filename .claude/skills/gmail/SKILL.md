---
name: gmail
description: |
  gogcli (gog) を使用してGmailを操作するスキル。メールの検索、取得、送信、ラベル管理などをCLI経由で実行。
  Use when: (1) メールを検索したい、(2) メールを送信したい、(3) メールを読みたい、(4) Gmailを操作したい、(5) 受信トレイを確認したい
  Trigger: gmail, メール, mail, email, 受信トレイ, inbox, メール送信, send email
---

# Gmail Operations with gogcli

`gog` CLIを使用してGmailを操作する。

## Prerequisites

```bash
# Installation
brew install gogcli

# Authentication (初回のみ)
gog auth login
```

## Commands Reference

### Search Emails

```bash
# 基本検索
gog gmail search "検索クエリ"

# オプション付き検索
gog gmail search "from:example@gmail.com" --max=20 --json

# Gmail検索構文
# - from:送信者
# - to:受信者
# - subject:件名
# - is:unread (未読)
# - is:starred (スター付き)
# - has:attachment (添付あり)
# - after:2024/01/01 before:2024/12/31 (日付範囲)
# - label:ラベル名
```

### Get Message

```bash
# メッセージ取得（スレッドIDから）
gog gmail get <messageId>

# JSON形式で取得
gog gmail get <messageId> --json
```

### Send Email

```bash
# 基本送信
gog gmail send \
  --to="recipient@example.com" \
  --subject="件名" \
  --body="本文"

# CC/BCC付き
gog gmail send \
  --to="to@example.com" \
  --cc="cc@example.com" \
  --bcc="bcc@example.com" \
  --subject="件名" \
  --body="本文"

# 添付ファイル付き
gog gmail send \
  --to="recipient@example.com" \
  --subject="件名" \
  --body="本文" \
  --attach="/path/to/file.pdf"

# ファイルから本文を読み込み
gog gmail send \
  --to="recipient@example.com" \
  --subject="件名" \
  --body-file="/path/to/body.txt"

# 返信
gog gmail send \
  --reply-to-message-id="<message-id>" \
  --reply-all \
  --body="返信本文"
```

### Labels

```bash
# ラベル一覧
gog gmail labels list

# ラベル作成
gog gmail labels create "新しいラベル"

# スレッドにラベル追加
gog gmail thread modify <threadId> --add-labels="STARRED"

# スレッドからラベル削除
gog gmail thread modify <threadId> --remove-labels="UNREAD"
```

### Drafts

```bash
# 下書き一覧
gog gmail drafts list

# 下書き作成
gog gmail drafts create \
  --to="recipient@example.com" \
  --subject="件名" \
  --body="本文"
```

## Common Workflows

### 未読メール確認

```bash
gog gmail search "is:unread" --max=10
```

### 特定の人からのメール検索

```bash
gog gmail search "from:matsuo@example.com" --max=20
```

### 今週のメール確認

```bash
gog gmail search "newer_than:7d" --max=50
```

### メール送信（確認付き）

```bash
# --forceなしで実行すると確認プロンプトが表示される
gog gmail send \
  --to="recipient@example.com" \
  --subject="件名" \
  --body="本文"
```

## Output Formats

| Flag | Description |
|------|-------------|
| `--json` | JSON形式で出力（スクリプト向け） |
| `--plain` | TSV形式で出力（パース容易） |
| (default) | 人間が読みやすい形式 |

## Tips

- `--account=email@example.com` で複数アカウントを切り替え
- `--no-input` でCI/自動化環境向け（プロンプトなし）
- `--force` で確認をスキップ（注意して使用）
