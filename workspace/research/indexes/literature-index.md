# Literature Index

```dataview
TABLE paper_title, domain, year, venue, status, publish_candidate
FROM "workspace/research/literature"
WHERE note_type = "literature"
SORT year DESC
```
