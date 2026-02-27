---
layout: page
title: 你的 CS 教育中遺失的一學期
description: >
  掌握強大的工具，讓你成為更有效率的電腦科學學習者與程式設計者。
subtitle: IAP 2026
nositetitle: true
---

在許多課程裡，你會學到電腦科學的進階主題，從作業系統到機器學習都有，
但有一個關鍵主題常常被忽略，最後只能靠學生自己摸索：如何熟練使用工具。
我們會教你如何精通命令列、使用強大的文字編輯器、活用版本控制系統的進階功能，
以及更多實用技巧！

學生在學習期間會花上數百小時使用這些工具（在職涯中更可能是數千小時），
因此把使用流程變得順暢、低阻力非常值得。熟練這些工具不只可以讓你少花時間
研究「工具到底怎麼用才對」，也能幫你解決以前看起來幾乎不可能處理的複雜問題。

近年來，軟體工程的許多面向也因為 AI 工具與 AI 強化工作流程而快速變化。
只要在理解其限制的前提下正確使用，這些工具通常能為電腦科學實作者帶來
顯著幫助，因此很值得建立實務上的運用能力。由於 AI 是跨領域的賦能技術，
我們不另外開一堂獨立的 AI 課，而是把最新、最實用的 AI 工具與技巧，
直接整合進每一堂講座中。

想了解這門課的起源與理念，請參考[為什麼我們要開這門課](/about/)。

{% comment %}
# 報名

填寫這份[報名表](https://forms.gle/j2wMzi7qeiZmzEWy9)即可報名 IAP 2026 課程。
{% endcomment %}

# 課程時程

{% comment %}
**講座**: [35-225](https://whereis.mit.edu/?go=35), 1:30--2:30pm（_例外_：1/16（五）為 3--4pm）<br>
**討論**: [OSSU Discord](https://ossu.dev/#community)（`#missing-semester-forum` 可當作 Piazza 使用，`#missing-semester` 可與課程同學和講師交流）
{% endcomment %}

<ul>
{% assign lectures = site['2026'] | sort: 'date' %}
{% for lecture in lectures %}
    {% if lecture.phony != true %}
        <li>
        <strong>{{ lecture.date | date: '%-m/%d/%y' }}</strong>:
        {% if lecture.ready %}
            <a href="{{ lecture.url }}">{{ lecture.title }}</a>
        {% else %}
            {{ lecture.title }} {% if lecture.noclass %}[停課]{% endif %}
        {% endif %}
        </li>
    {% endif %}
{% endfor %}
</ul>

{% comment %}
講座影片會在課後立即提供給 MIT 學生（透過 Panopto）。由於系統限制，只有擁有 MIT Kerberos 帳號的人可以存取原始講座影片。我們正在剪輯影片並上傳到 YouTube，目前已有部分影片上線，預計在二月中前全部上傳完成。

如果你不想等到 2026 年 1 月，也可以先看看[先前開課版本](/2020/)的講座內容，涵蓋了許多相同主題。
{% endcomment %}

你可以在 [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L) 觀看講座影片。

你可以在 [OSSU Discord](https://ossu.dev/#community) 討論課程（`#missing-semester-forum` 可當作 Piazza 使用，`#missing-semester` 可與課程同學和講師交流）。

# 課程資訊

**授課團隊**: 本課程由 [Anish](https://anish.io/)、[Jon](https://thesquareplanet.com/) 與 [Jose](https://josejg.com/) 共同授課。<br>
**聯絡方式**: 如有問題，請來信 [missing-semester@mit.edu](mailto:missing-semester@mit.edu)。

# MIT 之外

我們也將這門課分享給 MIT 之外的社群，希望更多人能從這些資源受益。你可以在以下平台看到貼文與討論：

 - Hacker News ([2026](https://news.ycombinator.com/item?id=47124171), [2020](https://news.ycombinator.com/item?id=22226380), [2019](https://news.ycombinator.com/item?id=19078281))
 - Lobsters ([2026](https://lobste.rs/s/q4ykw7/missing_semester_your_cs_education_2026), [2020](https://lobste.rs/s/ti1k98/missing_semester_your_cs_education_mit), [2019](https://lobste.rs/s/h6157x/mit_hacker_tools_lecture_series_on))
 - r/learnprogramming ([2026](https://www.reddit.com/r/learnprogramming/comments/1r93yk6/the_missing_semester_of_your_cs_education_2026/), [2020](https://www.reddit.com/r/learnprogramming/comments/eyagda/the_missing_semester_of_your_cs_education_mit/), [2019](https://www.reddit.com/r/learnprogramming/comments/an42uu/mit_hacker_tools_a_lecture_series_on_programmer/))
 - r/programming ([2020](https://www.reddit.com/r/programming/comments/eyagcd/the_missing_semester_of_your_cs_education_mit/), [2019](https://www.reddit.com/r/programming/comments/an3xki/mit_hacker_tools_a_lecture_series_on_programmer/))
 - X ([2026](https://x.com/anishathalye/status/2024521145777848588), [2020](https://twitter.com/jonhoo/status/1224383452591509507), [2019](https://x.com/jonhoo/status/1090323977766137858))
 - Bluesky ([2026](https://bsky.app/profile/jonhoo.eu/post/3mfa2bhyuj22i))
 - Mastodon ([2026](https://fosstodon.org/@jonhoo/116098318361854057))
 - LinkedIn ([2026](https://www.linkedin.com/posts/anishathalye_i-returned-to-mit-during-iap-january-term-activity-7430285026933522433-Ehr9))
 - YouTube ([2026](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L), [2020](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J), [2019](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuiujH1lpn8cA9dsyulbYRv))

# 各語言翻譯

{% comment %} 請維持依字母順序排列 {% endcomment %}

- [阿拉伯語](https://missing-semester-ar.github.io/)
- [孟加拉語](https://missing-semester-bn.github.io/)
- [中文（簡體）](https://missing-semester-cn.github.io/)
- [德語](https://missing-semester-de.github.io/)
- [義大利語](https://missing-semester-it.github.io/)
- [日語](https://missing-semester-jp.github.io/)
- [康納達語](https://missing-semester-kn.github.io/)
- [韓語](https://missing-semester-kr.github.io/)
- [波斯語](https://missing-semester-fa.github.io/)
- [葡萄牙語](https://missing-semester-pt.github.io/)
- [俄語](https://missing-semester-rus.github.io/)
- [塞爾維亞語](https://netboxify.com/missing-semester/)
- [西班牙語](https://missing-semester-esp.github.io/)
- [泰語](https://missing-semester-th.github.io/)
- [土耳其語](https://missing-semester-tr.github.io/)
- [越南語](https://missing-semester-vn.github.io/)

註：這些是社群提供的外部翻譯連結，尚未經過我們正式審核。

你有製作這門課講義的翻譯版本嗎？歡迎提交
[pull request](https://github.com/missing-semester/missing-semester/pulls)，
我們就能把它加入清單！

## 致謝

{% comment %}
2026 年致謝；歷年致謝請見各年度頁面
{% endcomment %}

感謝 Elaine Mello 與 [MIT Open Learning](https://openlearning.mit.edu/) 協助我們完成講座錄影。也感謝 Luis Turino 與 [SIPB](https://sipb.mit.edu/) 對本課程的支持，讓本課程成為 [SIPB IAP 2026](https://sipb.mit.edu/iap/) 的一部分。

---

<div class="small center">
<p><a href="https://github.com/missing-semester/missing-semester">原始碼</a>。</p>
<p>採用 CC BY-NC-SA 授權。</p>
<p>貢獻與翻譯指引請見<a href="/license/">此處</a>。</p>
</div>
