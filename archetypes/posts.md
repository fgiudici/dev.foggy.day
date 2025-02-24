---
date: '{{ time.Now.Format "2006-1-2" }}'
draft: true
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
summary: 'nice summary'
cover:
  image: images/folders.png
  hiddenInList: true
---
