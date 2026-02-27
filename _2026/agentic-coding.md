---
layout: lecture
title: "代理式程式開發"
description: >
  學習如何有效使用 AI 程式開發代理來完成軟體開發任務。
thumbnail: /static/assets/thumbnails/2026/lec7.png
date: 2026-01-21
ready: true
video:
  aspect: 56.25
  id: sTdz6PZoAnw
---

程式開發代理（coding agents）是一種可對話的 AI 模型，能使用讀寫檔案、網頁搜尋、執行 shell 指令等工具。它們可以存在於 IDE 內，也可以存在於獨立的命令列或圖形介面工具中。這類代理具備高度自主性且功能強大，能支援非常多元的使用情境。

本講座延伸自[開發環境與工具](/2026/development-environment/)中的 AI 輔助開發內容。先來看一個快速示範：我們延續[AI 輔助開發](/2026/development-environment/#ai-powered-development)章節裡的範例：

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')

def extract(content: str) -> list[str]:
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)

print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

我們可以用以下任務來提示一個程式開發代理：

```
把這段改成完整的命令列程式，使用 argparse 解析參數。補上型別註記，並確保程式可通過型別檢查。
```

代理會先讀取檔案理解內容，再進行修改，最後執行型別檢查器，確認型別註記是否正確。如果它犯錯導致型別檢查失敗，通常會自行反覆修正；不過這個任務很單純，發生機率較低。由於代理可以呼叫可能帶來風險的工具，多數 agent harness 預設都會先請使用者確認工具呼叫。

> 如果程式開發代理犯錯——例如你的 `$PATH` 裡已經有 `mypy` 可直接執行，但代理卻去呼叫 `python -m mypy`——你可以透過文字回饋引導它修正方向。

程式開發代理支援多輪互動，你可以在來回對話中持續迭代成果。如果它走錯方向，你也可以中途打斷。實用的心智模型是把自己想成實習生的主管：實習生會處理許多瑣碎工作，但需要你給方向，也偶爾會做錯，需要你校正。

> 若想看更具體的示範，可在下一輪請代理執行產生的腳本。觀察輸出結果後，再請它修改（例如只保留絕對 URL）。

# AI 模型與代理如何運作

完整解釋現代[大型語言模型（LLM）](https://en.wikipedia.org/wiki/Large_language_model)與 agent harness 等基礎設施的內部原理，超出本課程範圍。不過，掌握一些高層次的關鍵概念，能幫助你更有效地_使用_這項前沿技術，也更清楚它的限制。

你可以把 LLM 視為：在給定提示字串（輸入）後，去建模補全文字字串（輸出）的機率分布。LLM 推論（例如你在對話聊天工具送出問題時發生的事）會從這個機率分布中進行_取樣_。LLM 也有固定的_上下文視窗_，也就是輸入與輸出字串總長度的上限。

{% comment %}
> In mathematical notation, the LLM models the probability distribution $\pi_\theta$ of completions $y$ conditioned on prompts $x$, and we sample from this distribution: $\hat{y} \sim \pi_\theta(\cdot \mid x)$.
{% endcomment %}

對話式聊天與程式開發代理等 AI 工具，都是建立在這個基礎之上。多輪互動時，聊天工具與代理會用回合標記，並在每次新提問時，把整段對話歷史都當成提示字串，再執行一次 LLM 推論。對於可呼叫工具的代理，harness 會把某些 LLM 輸出解讀成工具呼叫請求，並把工具回傳結果再餵給模型作為提示字串的一部分（所以每次工具呼叫與回應都會再跑一次 LLM 推論）。工具呼叫型代理的核心概念，甚至可以[用 200 行程式碼實作](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/)。

## 隱私

大多數 AI 程式工具在預設設定下，會把不少資料送到雲端。有時是 harness 在本機、LLM 推論在雲端；有時甚至更多軟體元件都在雲端執行（例如服務提供者可能實質上會取得你整個儲存庫副本，以及你與 AI 工具的所有互動內容）。

目前已有不錯的開源 AI 程式工具與開源 LLM（雖然通常還比不上封閉模型），但以現況來說，受限於硬體條件，多數使用者仍難以在本機跑最新一代的開源 LLM。

# 使用情境

程式開發代理可協助各式各樣的工作。以下是一些例子：

- **實作新功能。** 就像前面的範例，你可以請程式開發代理實作某個功能。目前「怎麼寫出好規格」比較像是一門藝術，不太是精準科學：你希望輸入夠具體，讓代理能照你的需求做（至少方向正確，便於後續迭代），但又不要描述過頭到變成你自己把大部分工作都做完。測試驅動開發特別有效：先寫測試（或讓代理協助你寫測試）、檢查測試是否真的捕捉到你要的行為，再請代理實作功能。模型能力持續進步，因此你也要持續更新自己對模型能力邊界的直覺。
    > 我們曾用 Claude Code 來[實作](https://github.com/missing-semester/missing-semester/pull/345)這些 Tufte 風格的旁註。
{%- comment %}
No need to demo this, since the intro of a lecture was a small demo of adding a new feature.
{% endcomment %}
- **修正錯誤。** 如果編譯器、linter、型別檢查器或測試回報錯誤，你可以請代理協助修正，例如提示「fix the issues with mypy」。當模型處在回饋循環裡時，通常特別有效，因此盡量讓模型能直接執行失敗的檢查，讓它可自主反覆修正。若實作上不方便，也可以手動提供回饋給模型。
    > 在 missing-semester 儲存庫的 commit [f552b55](https://github.com/missing-semester/missing-semester/commit/f552b5523462b22b8893a8404d2110c4e59613dd)，我們曾用提示「Review the agentic coding lecture for typos and grammatical issues」請 Claude Code 審閱內容，接著再請它修正找到的問題，最後把修正提交為 [f1e1c41](https://github.com/missing-semester/missing-semester/commit/f1e1c417adba6b4149f7eef91ff5624de40dc637)。
{%- comment %}
Demo a coding agent fixing the bug in https://github.com/anishathalye/dotbot/commit/cef40c902ef0f52f484153413142b5154bbc5e99.

Write the failing tests to demo the bug, and then ask the agent to fix. Prepped in branch demo-bugfix.

Can run the failing test with:

    hatch test tests/test_cli.py::test_issue_357

Can prompt coding agent with:

    There is a bug I wrote a failing test for, you can repro it with `hatch test tests/test_cli.py::test_issue_357`. Fix the bug.

Get it to commit the changes.
{% endcomment %}
- **重構。** 你可以用程式開發代理做各種重構，從簡單的函式改名（這類重構也可由[程式碼智慧](/2026/development-environment/#code-intelligence-and-language-servers)支援）到更複雜的任務，例如把功能拆分到獨立模組。
    > 我們曾用 Claude Code 將 agentic coding[拆分](https://github.com/missing-semester/missing-semester/pull/344)成獨立講座。
{%- comment %}
Show usage in Missing Semester, point out that the agent did make some mistakes.
{% endcomment %}
- **程式碼審查。** 你可以請程式開發代理做 code review。你可以先給基本指示，例如「review 我最新但尚未 commit 的變更」。若你想審查 pull request，且代理支援網頁抓取，或你已安裝像 [GitHub CLI](https://cli.github.com/) 這類命令列工具，甚至可以直接說「Review the pull request {link}」，之後交給它處理。
{%- comment %}
In Porcupine repo, prompt agent with:

    Review this PR: https://github.com/anishathalye/porcupine/pull/39
{% endcomment %}
- **理解程式碼。** 你可以針對某個程式碼庫向代理提問，這在新成員上手（onboarding）時特別有幫助。
{%- comment %}
Some prompts to try in the missing-semester repo:

    How do I run this site locally?

    How are the social preview cards implemented?
{% endcomment %}
- **當作 shell 使用。** 你可以請代理用指定工具解決任務，因此能用自然語言去呼叫 shell 指令，例如「use the find command to find all files older than 30 days」或「use mogrify to resize all the jpgs to 50% of their original size」。
{%- comment %}
In Dotbot repo, prompt agent with:

    Use the ag command to find all Python renaming imports
{% endcomment %}
- **Vibe coding。** 代理已經強大到你可以在自己一行程式都不寫的情況下完成某些應用。
    > [這裡有一個範例](https://github.com/cleanlab/office-presence-dashboard)，是其中一位講師用 vibe coding 完成的真實專案。
{%- comment %}
In missing-semester repo, prompt agent with:

    Make this site look retro.
{% endcomment %}

# 進階代理

以下簡要介紹一些更進階的使用模式與能力。

- **可重複使用的提示。** 建立可重複使用的提示或模板。例如，你可以寫一個很完整的 code review 提示，並儲存成可重複使用的提示。
    > 代理工具演進很快。在某些工具中，「可重複使用提示」作為獨立功能已逐漸被淘汰。例如在 Codex 與 Claude Code 中，這項能力被 [skills](https://code.claude.com/docs/en/skills) 所[整合](https://developers.openai.com/codex/custom-prompts)。
- **平行代理。** 程式開發代理可能偏慢：你給它提示後，它可能要花數十分鐘處理問題。你可以同時執行多個代理副本，不管是處理同一個任務（LLM 有隨機性，重跑幾次再挑最佳解常常有幫助）或不同任務（例如同時做兩個互不重疊的功能）。為了避免不同代理的變更互相干擾，可以使用 [git worktree](https://git-scm.com/docs/git-worktree)；我們會在[版本控制](/2026/version-control/)講座中介紹。
- **MCP。** MCP 是 _Model Context Protocol_ 的縮寫，是一個開放協定，可把程式開發代理與各種工具連接起來。例如這個 [Notion MCP server](https://github.com/makenotion/notion-mcp-server) 可以讓代理讀寫 Notion 文件，進而支援像是「讀取 {Notion doc} 連結的規格、在 Notion 新頁面草擬實作計畫、再實作原型」這類流程。若要探索 MCP，可以參考 [Pulse](https://www.pulsemcp.com/servers) 與 [Glama](https://glama.ai/mcp/servers) 這類目錄。
- **上下文管理。** 如同我們在[前文](#ai-模型與代理如何運作)提到，作為程式開發代理基礎的 LLM 具有有限的_上下文視窗_。要有效使用代理，就必須好好管理上下文。你需要確保代理拿到必要資訊，同時避免塞入不必要內容，以免超出上下文視窗或讓模型效能下降（即便沒有超出上限，隨著上下文增長也常會發生）。Agent harness 會自動提供、且在一定程度上管理上下文，但許多控制權仍在使用者手上。
    - **清空上下文視窗。** 最基本的控制方式是清空上下文視窗（開新對話）。遇到不相關任務時，建議這麼做。
    - **回溯對話。** 有些程式開發代理支援復原對話歷史中的步驟。某些情況下，比起補一句新訊息把代理導回另一方向，直接「undo」會更有效地管理上下文。
{%- comment %}
Make up a quick demo.
{% endcomment %}
    - **壓縮（compaction）。** 為了支援理論上無上限長度的對話，程式開發代理通常提供上下文_壓縮_：當對話歷史太長時，會自動呼叫 LLM 摘要前段內容，並以摘要取代原始歷史。某些代理也允許使用者在需要時手動觸發壓縮。
{%- comment %}
Show `/compact` in Claude Code, show full summary.
{% endcomment %}
    - **llms.txt。** `/llms.txt` 是一個提案中的[標準位置](https://llmstxt.org/)，用來放給 LLM 在推論時讀取的文件。產品（例如 [cursor.com/llms.txt](https://cursor.com/llms.txt)）、軟體函式庫（例如 [ai.pydantic.dev/llms.txt](https://ai.pydantic.dev/llms.txt)）與 API（例如 [apify.com/llms.txt](https://apify.com/llms.txt)）都可能提供 `llms.txt`，對開發很有幫助。這類文件每個 token 的資訊密度較高，因此在上下文效率上，通常比叫代理去抓整個 HTML 頁面更好。當代理對你要用的依賴套件缺乏內建知識時（例如該套件發布於模型知識截止日之後），外部文件特別實用。
{%- comment %}
Side-by-side comparison in an empty repo (on Desktop or some other self-contained place, with `git init` run in it):

    Write a single-file Python program example in demo.py using semlib to sort "Ilya Sutskever", "Soumith Chintala", and "Donald Knuth" in terms of their fame as AI researchers.

    Write a single-file Python program example in demo.py using semlib to sort "Ilya Sutskever", "Soumith Chintala", and "Donald Knuth" in terms of their fame as AI researchers. See https://semlib.anish.io/llms.txt. Follow links to Markdown versions of any pages linked in llms.txt files.

Not sure why the agent doesn't do this by default. You'd probably put that last sentence in a CLAUDE.md file.
{% endcomment %}
    - **AGENTS.md。** 多數程式開發代理支援 [AGENTS.md](https://agents.md/) 或類似機制（例如 Claude Code 會找 `CLAUDE.md`）作為代理專用 README。代理啟動時，會把 `AGENTS.md` 全文先放進上下文。你可以把跨多個工作階段都通用的指引寫在這裡（例如要求每次改完程式都跑型別檢查、說明單元測試怎麼跑、或提供可供代理瀏覽的第三方文件連結）。有些代理可自動產生這個檔案（例如 Claude Code 的 `/init` 指令）。真實案例可參考[這份](https://github.com/pydantic/pydantic-ai/blob/main/CLAUDE.md) `AGENTS.md`。
{%- comment %}
Dotbot example, CLAUDE.md that includes @DEVELOPMENT.md and says to always run the type checker and code formatter after making any changes to Python code.

Example prompt, off of master:

    Remove the "--version" command-line flag.

This is something that'll be fast, for demonstration purposes.
{% endcomment %}
    - **Skills。** `AGENTS.md` 的內容會完整且持續載入代理的上下文。_Skills_ 透過一層間接機制避免上下文膨脹：你可以先提供技能清單與描述，代理再視需要「開啟」某個技能（載入到上下文視窗）。
    - **Subagents。** 有些程式開發代理允許你定義子代理，用於特定任務流程。上層代理可呼叫子代理完成指定工作，讓上層代理與子代理都更有效管理上下文。上層代理不必塞滿子代理看到的全部內容；子代理也只需拿到任務所需資訊。舉例來說，有些代理把網路研究做成子代理：上層代理先丟查詢給子代理，子代理再負責網頁搜尋、抓取頁面、分析內容，最後回覆答案給上層代理。這樣上層代理不用承擔所有網頁全文造成的上下文膨脹，子代理也不需要載入上層代理完整的對話歷史。

對於許多需要撰寫提示的進階功能（例如 skills 或 subagents），你可以先用 LLM 幫你打草稿。有些程式開發代理甚至內建這種能力。例如 Claude Code 能根據短提示直接產生子代理（執行 `/agents` 並建立新代理）。你可以試著用以下提示建立子代理：

```
一個 Python 程式檢查代理，使用 `mypy` 與 `ruff` 對自上次 git commit 以來有修改的所有檔案進行型別檢查、lint，以及格式化檢查。
```

接著你可以在上層代理明確下指令，例如「use the code checker subagent」，來呼叫這個子代理。你也可能讓上層代理在適當時機自動呼叫它，例如每次改動 Python 檔案之後。

# 注意事項

AI 工具會犯錯。它們建立在 LLM 之上，本質上只是機率式的下一個 token 預測模型，並不是與人類同等的「智慧」。請務必審查 AI 產出，確認正確性與安全性漏洞。有時驗證程式碼的成本甚至高於你自己手寫；對關鍵程式碼，建議考慮手寫。AI 也可能鑽牛角尖，甚至用看似合理的說法把你帶偏，請留意除錯螺旋。不要把 AI 當拐杖，也要避免過度依賴或只停留在表面理解。仍有大量程式任務是 AI 做不到的；運算思維依然非常重要。

# 推薦軟體

許多 IDE / AI 程式擴充套件都內建程式開發代理（可參考[開發環境講座](/2026/development-environment/)的推薦）。其他常見代理還有 Anthropic 的 [Claude Code](https://www.claude.com/product/claude-code)、OpenAI 的 [Codex](https://openai.com/codex/)，以及像 [opencode](https://github.com/anomalyco/opencode) 這類開源代理。

# 練習

1. 用同一個程式任務分別做四次，比較「手寫程式」、「AI 自動補全」、「行內聊天（inline chat）」與「代理」的體驗差異。最適合的題目，是你正在進行專案中的小功能。若想找其他題目，可考慮 GitHub 開源專案的「good first issue」類型任務，或 [Advent of Code](https://adventofcode.com/) / [LeetCode](https://leetcode.com/) 題目。
1. 使用 AI 程式開發代理探索一個你不熟悉的程式碼庫。最理想情境是你真的想在某個專案除錯或新增功能。如果暫時沒有目標，可試著用 AI 代理理解 [opencode](https://github.com/anomalyco/opencode) 中與安全性相關功能是如何運作的。
1. 從零開始 vibe code 一個小型應用，過程中不要手寫任何一行程式碼。
1. 針對你選擇的程式開發代理，建立並測試 `AGENTS.md`（或對應檔案，例如 `CLAUDE.md`）、一個 skill（例如 [Claude Code 的 skill](https://code.claude.com/docs/en/skills) 或 [Codex 的 skill](https://developers.openai.com/codex/skills/)），以及一個 subagent（例如 [Claude Code 的 subagent](https://code.claude.com/docs/en/sub-agents)）。思考在什麼情境下該用哪一種機制。注意：你選的代理可能不支援全部功能；你可以略過，或改用其他有支援的代理。
1. 用程式開發代理完成與[程式碼品質講座](/2026/code-quality/)中「Markdown 項目符號 regex 練習」同樣的目標。它是不是直接編輯檔案來完成任務？若代理用直接改檔方式完成，缺點與限制是什麼？再想辦法調整提示，讓代理不要靠直接改檔完成。提示：請代理使用[第一堂課](/2026/course-shell/)提到的某個命令列工具。
1. 大多數程式開發代理都支援某種「yolo mode」（例如 Claude Code 的 `--dangerously-skip-permissions`）。直接使用這種模式並不安全，但你可以在虛擬機或容器等隔離環境中執行代理，再啟用自主模式，通常會比較可接受。請在你的電腦上完成這種設定。像是 [Claude Code devcontainers](https://code.claude.com/docs/en/devcontainer) 或 [Docker Sandboxes / Claude Code](https://docs.docker.com/ai/sandboxes/agents/claude-code/) 這類文件可能很有幫助。這件事有不只一種做法。
