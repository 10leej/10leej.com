---
title: "{{ replace .Name "-" " " | title }}"
type: post
date: {{ .Date }}
url: /{{ .Name }}/
image: /images/2020-thumbs/{{ .Name }}.jpg
categories:
  - Blog
  - News
tags:
  - Ubuntu
draft: false
---
<!--more-->