---
date: '{{ time.Now.Format "2025-01-01" }}'
draft: true
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
---
