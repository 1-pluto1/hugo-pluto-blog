# Publish Candidates

```dataview
TABLE paper_title, domain, status
FROM "workspace/research/literature"
WHERE publish_candidate = true
SORT updated DESC
```
