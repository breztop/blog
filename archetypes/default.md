+++
date = '{{ .Date }}'
draft = false
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
description = ''
author = '{{ .Site.Params.author | default "breeze" }}'
tags = []
categories = []
series = []
keywords = []
aliases = []
language = 'zh-CN'

[cover]
  image = ''
  alt = ''
  caption = ''
  relative = false

[images]
  - = ""

[sitemap]
  changefreq = 'monthly'
  priority = 0.5

[extra]
  toc = true
  reading_time = true
  comments = false
+++