221013

```dataview
TABLE file.tags as tags
FROM "Topics"
```

Tutorial for how to use dataview:
https://denisetodd.medium.com/obsidian-dataview-for-beginners-a-checklist-to-help-fix-your-dataview-queries-11acc57f1e48

Sample query that actually works:
https://forum.obsidian.md/t/is-there-a-way-to-retrieve-a-list-of-tags-of-e-g-files-in-a-folder/20280

Figure out how to do this:
https://liamca.in/Obsidian/API+FAQ/metadata/get+a+list+of+all+the+tags+in+the+vault

#dataview, #tech/query, #tech/db, #tutorial #obsidian #search

```dataview
table file.cday as "Date Created" 
from "Topics" 
sort file.cday asc
```

