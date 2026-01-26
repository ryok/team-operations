# 週報作成自動化システム

## プロジェクト概要

Claude Codeのスキルとコマンドを使用して、Notionの情報から自動的に週報を作成するシステムです。

## 技術スタック

- **AI**: Claude Code
- **外部連携**: Notion API（MCP経由）
- **設定**: `.mcp.json` でNotion MCPサーバーを設定

## プロジェクト構造

```
team-operations/
├── .claude/
│   ├── commands/
│   │   └── create-weekly-report.md  # 週報作成コマンド（詳細版）
│   ├── skills/
│   │   └── weekly-report/
│   │       ├── SKILL.md             # 週報作成スキル
│   │       └── template.md          # 週報テンプレート
│   ├── settings.json                # プロジェクト設定
│   └── settings.local.json          # ローカル設定（git対象外）
├── .mcp.json                        # MCP設定（Notion API）
├── CLAUDE.md                        # このファイル
└── README.md                        # プロジェクト説明
```

## 使い方

### 週報を作成する

```
/weekly-report
```

または自然言語で:
- 「週報を作成して」
- 「create weekly report」
- 「週次報告を作成したい」

### 週報作成の流れ

1. **先週版の取得**: 直近の週報をNotionから検索してベースとして使用
2. **情報収集**: Engineering Meeting Management DB、AI Coding DBなどから今週の更新を収集
3. **内容の更新**: 先週版をベースに差分を反映
4. **Notion発行**: 新しい週報ページを作成し、MailBodyプロパティも設定
5. **検証**: 作成内容の整合性を確認

### 週報のフォーマット

- ページ名: `【週次報告】<月曜日の日付>週 共同研究 MLシステム開発`
- DB: Engineering Meeting Management
- 詳細なテンプレートは `.claude/skills/weekly-report/template.md` を参照

## 設定

### MCP設定 (`.mcp.json`)

Notion APIへの接続設定。APIキーは環境変数または設定ファイルで管理。

### ローカル設定 (`.claude/settings.local.json`)

個人の設定（git対象外）。

## 注意事項

- 確証が薄い情報には `<確認してください>` タグが付与される
- 出典は `[src: MeetingNote YYYY-MM-DD]` 形式で記載される
- 週報作成後は内容を確認してから送信すること

## 参考資料

- [Notion API Documentation](https://developers.notion.com/)
- [Claude Code Documentation](https://docs.anthropic.com/claude-code)
