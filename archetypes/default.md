---
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
date: {{ .Date }}
draft: false
image: 'main.jpg'
categories:
  - カテゴリ
tags:
  - タグ
slug: '{{ replace .File.ContentBaseName "-" " " | title }}'
---

## 記事
