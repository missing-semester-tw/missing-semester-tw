---
layout: page
title: 歷年開課
description: >
  查看 Missing Semester 歷年的所有開課版本。
---

{% comment %} 使用 pop 移除預設的 "posts" collection {% endcomment %}
{% assign sorted_collections = site.collections | sort: 'label' | pop | reverse %}
<ul>
{% for collection in sorted_collections %}
    {% if forloop.index == 1 %}
        <li><a href="/">{{ collection.label }}</a>（目前）</li>
    {% else %}
        <li><a href="/{{ collection.label }}/">{{ collection.label }}</a></li>
    {% endif %}
{% endfor %}
</ul>

每一年的講座內容都可以獨立學習。我們建議你先從最新版本的教材開始。由於每年涵蓋的主題會有些差異，我們也持續保留本課程早期版本的講義與影片。
