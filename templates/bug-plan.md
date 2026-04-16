You are reviewing a bug in planning.

Ticket: {{ issue.key }} — {{ issue.fields.summary }}
Priority: {{ issue.fields.priority.name }}
Status:   {{ issue.fields.status.name }}

Description:
{{ issue.fields.description }}

Follow the team's plan-review policy:

{{doc:docs/plan-review-policy.md}}

Your output should be a short review comment suitable for posting back
to Jira. Focus on scope, risk, and test strategy — not copy edits.
