# CLAUDE.md

## プロジェクト概要

松尾研究所 共同研究 MLシステム開発チームのあらゆる業務を効率化するプロジェクトです．

## Notion

共同研究 MLシステム開発チームのNotionページ:
https://www.notion.so/1304bc176b2b80189051dd7137f86054?v=2424bc176b2b8066af31000c9a3168a6

## 役割

Notionから情報を取得し，あらゆる事務作業やマネジメント業務を支援してください．

## 週報作成の使い方

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

## 開発環境

- Python環境: 必ず`uv`を使用してください
- **ブラウザ自動化**: agent-browserスキル または chrome-devtools MCP

## 注意事項

- 週報作成時に，確証が薄い情報には `<確認してください>` タグが付与される
- 出典は `[src: MeetingNote YYYY-MM-DD]` 形式で記載される
- 週報作成後は内容を確認してから送信すること
