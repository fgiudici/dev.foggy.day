---
date: '{{ time.Now.Format "2025-01-01" }}'
draft: true
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
summary: 'nice summary'
cover:
  image: images/folders.png
  hiddenInList: true
---
