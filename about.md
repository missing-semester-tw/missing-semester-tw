---
layout: lecture
title: "為什麼我們要開這門課"
---

在傳統的電腦科學教育中，你很可能會修很多進階課程，內容從作業系統、
程式語言到機器學習都有。不過在許多學校裡，有一個關鍵主題很少被正式教授，
通常只能讓學生自己摸索：對運算生態系的整體素養。

多年來，我們在 MIT 參與過多門課程教學，也一再發現許多學生對自己手上
可用的工具並不熟悉。電腦原本就是為了自動化重複的人工工作而生，但學生
常常還是手動做重複性任務，或沒有好好利用版本控制、文字編輯器等強大工具。
情況好一點時，這會造成效率低落與時間浪費；嚴重一點時，可能導致資料遺失，
甚至讓某些任務無法完成。

這些內容通常不在大學正式課綱中：學生往往沒被教過如何使用這些工具，
或至少沒有學到「有效率地使用」。結果就是把大量時間與心力花在本來
_應該_ 很簡單的事情上。標準 CS 課程缺少了許多與運算生態系相關、但其實
能大幅改善學習與工作體驗的重要主題。

# 你的 CS 教育中遺失的一學期

為了補上這個缺口，我們開設了這門課，涵蓋我們認為成為高效電腦科學學習者
與程式設計者所必備的主題。課程強調務實與可立即上手，會帶你實際操作各種工具
與技巧，讓你在未來遇到的多種情境都能馬上應用。這門課最新版（教材已大幅更新）
於 2026 年 1 月在 MIT 的「Independent Activities Period」期間開設。這是一個
為期一個月、以短期學生主導課程為特色的學期。雖然講座本身僅開放 MIT 社群參與，
但我們會對外公開所有教材與講座錄影。

如果你覺得這門課很適合你，以下是一些課程內容的具體例子：

## 命令列 shell

如何用 aliases、scripts 與建置系統來自動化常見的重複性工作。
不再需要從文字檔反覆複製貼上指令，不再需要「這 15 個指令請照順序跑完」，
也不再發生「你忘了跑這一步」或「你忘了帶這個參數」。

例如，快速搜尋指令歷史紀錄就能省下大量時間。下面的例子示範了幾個技巧，幫你更快在 shell 歷史中找到 `convert` 相關指令。

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/history.mp4" type="video/mp4">
</video>

## 版本控制

如何_正確地_使用版本控制，並活用它來避免災難、與他人協作、快速找出並隔離有問題的變更。
不再 `rm -rf; git clone` 重來，不再被合併衝突搞到崩潰（至少會大幅減少），
不再在程式裡留下整大段註解掉的舊程式碼，也不再苦惱「到底是哪裡壞掉了」，
更不會出現「天啊，我們是不是把能跑的版本刪掉了？」。我們甚至會教你如何
用 pull request 參與別人的專案！

下面的例子會用 `git bisect` 找出是哪個 commit 讓單元測試壞掉，接著再用 `git revert` 修正。
<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/git.mp4" type="video/mp4">
</video>

## 文字編輯

如何在本機與遠端都能透過命令列高效率編輯檔案，並善用編輯器的進階功能。
不再來回搬移檔案，不再重複做一樣的編輯操作。

Vim macro 是 Vim 最強大的功能之一。下面的例子會示範如何用巢狀 vim macro，快速把 HTML 表格轉成 CSV 格式。
<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/vim.mp4" type="video/mp4">
</video>

## 遠端主機

如何在操作遠端主機時，透過 SSH 金鑰與 terminal multiplexing 維持穩定工作流程。
不再為了同時跑兩個指令就開一堆終端機，不再每次連線都重打密碼，也不再因為
網路斷線或筆電重開機就把目前進度全部弄丟。

下面的例子會用 `tmux` 讓遠端伺服器上的工作階段持續存在，並用 `mosh` 支援網路切換與斷線重連。

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/ssh.mp4" type="video/mp4">
</video>

## 尋找檔案

如何快速找到你要的檔案。不再一層一層點資料夾，找半天才找到目標程式碼。

下面的例子會用 `fd` 快速找檔案、用 `rg` 快速找程式片段，並透過 `fasd` 快速 `cd` 到常用路徑或直接 `vim` 最近/常用檔案與資料夾。

<video autoplay="autoplay" loop="loop" controls muted playsinline  oncontextmenu="return false;"  preload="auto"  class="demo">
  <source src="/static/media/demos/find.mp4" type="video/mp4">
</video>

## 資料整理

如何直接在命令列快速完成資料與檔案的修改、檢視、解析、繪圖與運算。
不再從 log 檔手動複製貼上，不再手算統計值，也不再凡事都靠試算表畫圖。

## 程式碼品質與持續整合

如何使用自動排版、lint、測試與覆蓋率工具提升程式碼品質。
不再有難以維護的醜程式碼，不再反覆回歸錯誤，也不再出現「你電腦可以跑、別人電腦就當掉」的情況。

## 不只程式碼

如何撰寫高品質文件、清楚地與開源維護者溝通、提交可執行的 issue，
以及送出更容易被合併的 pull request。讓使用者不再因為文件不清楚而無法上手，
也降低你被維護者「已讀不回」的機率。

# 結語

這些內容與更多實用主題，都會在 9 堂講座中完整介紹，每堂也都包含練習，
幫助你自己動手熟悉這些工具。如果你不想等到 2026 年 1 月，也可以先看看
[先前開課版本](/2020/)的講座內容，裡面涵蓋了很多相同主題。

希望一月能見到你，不論是線上或實體參與！

祝你寫程式愉快，<br>
[Anish](https://anish.io/)、[Jon](https://thesquareplanet.com/) 與 [Jose](https://josejg.com/)
