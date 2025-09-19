# Weekly Report Review Command

## Description

Reviews the weekly report as a manager and provides constructive feedback, acknowledgment of achievements, and strategic guidance for the team.

## Usage

```bash
review-weekly-report [page-id]
```

| Parameter | Type   | Description                                                                         |
| --------- | ------ | ----------------------------------------------------------------------------------- |
| `page-id` | string | Notion page ID of the weekly report to review. If omitted, reviews the latest one. |

---

## What this command does

1. **Report Retrieval**
   - Fetch the specified weekly report from Notion
   - If no page-id provided, find the most recent weekly report in the Engineering Meeting Management DB
   - Extract the report content and metadata

2. **Content Analysis**
   - Analyze project progress and achievements
   - Identify potential risks and challenges
   - Evaluate resource allocation and team performance
   - Review strategic alignment with organizational goals

3. **Feedback Generation**
   - Acknowledge accomplishments and team efforts
   - Provide constructive feedback on areas for improvement
   - Offer strategic guidance and suggestions
   - Highlight priority items for the coming week
   - Address any concerns or blockers mentioned

4. **Comment Creation**
   - Generate manager's review in the following structure:
     - 総評 (Overall Assessment)
     - 良かった点 (Positive Points)
     - 改善提案 (Suggestions for Improvement)
     - 来週の重点事項 (Priority Items for Next Week)
     - 質問・確認事項 (Questions/Clarifications)
   - Add the review as a comment on the Notion page

---

## Review Template

```markdown
## 週報レビュー - [Date]

### 総評
[Overall assessment of the week's progress and team performance]

### 良かった点
• [Achievement 1]
• [Achievement 2]
• [Achievement 3]

### 改善提案
• [Suggestion 1]
• [Suggestion 2]

### 来週の重点事項
1. [Priority 1]
2. [Priority 2]
3. [Priority 3]

### 質問・確認事項
• [Question 1]
• [Question 2]

---
レビュー実施者: [Manager Name]
レビュー日時: [DateTime]
```

---

## Implementation Tasks

1. Use **TodoWrite** to plan review tasks
2. Retrieve the weekly report from Notion:
   - If page-id provided: Use API-retrieve-a-page
   - If not: Query Engineering Meeting Management DB for latest 週報
3. Analyze the report content:
   - Project progress evaluation
   - Resource utilization assessment
   - Risk identification
   - Strategic alignment check
4. Generate manager's feedback following the template
5. Post the review as a comment on the Notion page using API-create-a-comment
6. Optionally update the page with a "Reviewed" status or tag
7. Update todo list to mark completion

---

## Manager Persona Guidelines

When reviewing, adopt the perspective of a senior manager who:
- **Supportive**: Acknowledges team efforts and celebrates wins
- **Strategic**: Thinks about long-term goals and organizational alignment
- **Constructive**: Provides actionable feedback for improvement
- **Pragmatic**: Focuses on practical solutions and resource optimization
- **Forward-looking**: Anticipates challenges and opportunities
- **Detail-oriented**: Notices both big picture and important details

## Review Criteria

Evaluate the weekly report based on:
1. **Progress vs. Plans**: Are projects on track?
2. **Resource Efficiency**: Are resources being used effectively?
3. **Risk Management**: Are risks identified and mitigated?
4. **Team Performance**: Is the team working cohesively?
5. **Strategic Alignment**: Do activities align with organizational goals?
6. **Communication Quality**: Is the report clear and comprehensive?
7. **Innovation**: Are there opportunities for improvement or innovation?

## Sample Review Comments

### Positive Feedback Examples:
- "Google Agentspaceの検討開始は素晴らしい判断です。AIコーディングの効率化に大きく貢献することが期待されます。"
- "複数の新規プロジェクトの立ち上げをスムーズに進められていることを評価します。"
- "日テレプロジェクトの安定稼働への移行、お疲れ様でした。"

### Constructive Feedback Examples:
- "京西テクノスの提案準備について、来週中に具体的なスケジュールを設定してください。"
- "採用活動の進捗について、より具体的な数値（面接人数、採用予定等）を含めていただけると助かります。"
- "各プロジェクトのKPIや成功指標を明確にすることで、進捗評価がより客観的になります。"

### Strategic Guidance Examples:
- "AIコーディングツールの選定にあたっては、既存システムとの統合性を重視してください。"
- "中国銀行と大分県のプロジェクトで得たノウハウを、他のプロジェクトにも横展開できる仕組みを検討しましょう。"
- "開発会社とのアライアンスについて、具体的な協業モデルの策定を急ぎましょう。"