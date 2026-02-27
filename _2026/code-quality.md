---
layout: lecture
title: "程式碼品質"
description: >
  學習格式化、Lint、測試、持續整合等提升程式碼品質的方法。
thumbnail: /static/assets/thumbnails/2026/lec9.png
date: 2026-01-23
ready: true
video:
  aspect: 56.25
  id: XBiLUNx84CQ
---

有許多工具與方法可以協助開發者寫出高品質程式碼。本講會介紹：

- [格式化（Formatting）](#formatting)
- [Lint（靜態檢查）](#linting)
- [測試（Testing）](#testing)
- [Pre-commit hooks](#pre-commit-hooks)
- [持續整合（Continuous integration）](#continuous-integration)
- [指令執行器（Command runners）](#command-runners)

加碼主題還會談到[正規表示式](#regular-expressions)。它是跨領域的實用工具，不只可用在程式碼品質（例如只執行符合特定樣式的測試），也常用於 IDE（例如搜尋與取代）。

這些工具有些是語言專屬（例如 Python 的 [Ruff](https://docs.astral.sh/ruff/) linter/formatter），有些則支援多語言（例如 [Prettier](https://prettier.io/)）。不過核心觀念幾乎通用——任何程式語言都能找到格式化工具、linter、測試函式庫等。

# 格式化（Formatting）

程式碼自動格式化工具會自動整理表層語法。這樣你能把注意力放在真正困難的問題上，讓工具處理瑣碎細節，例如字串使用 `'` 或 `"` 的一致性、二元運算子前後空白（`x + y` 而非 `x+y`）、`import` 排序、以及過長行等。格式化工具的一大好處是：能統一整個程式碼庫的風格，降低團隊協作摩擦。

有些工具（如 Prettier）[可高度客製化](https://prettier.io/docs/configuration)；建議把設定檔納入專案的[版本控制](/2026/version-control/)。另外有些工具（如 [Black](https://github.com/psf/black) 與 [gofmt](https://pkg.go.dev/cmd/gofmt)）幾乎不提供可調參數，用意是減少[無謂爭論](https://en.wikipedia.org/wiki/Law_of_triviality)。

你可以把格式化工具和 [IDE 整合](/2026/development-environment/#code-intelligence-and-language-servers)，讓程式碼在輸入或儲存時自動格式化。你也可以在專案加入 [EditorConfig](https://editorconfig.org/) 檔案，向 IDE 傳達專案層級設定（例如各檔案類型的縮排大小）。

# Lint（靜態檢查）

Linter 會做靜態分析（不執行程式就分析程式碼），找出反模式與潛在問題。這類工具比格式化器更深入，不只看表面語法；不同 linter 的分析深度也不一樣。

Linter 通常內建一組 _rules_，並提供可在專案層級調整的預設組合。有些規則可能出現誤報（false positives），因此可在檔案或單行層級停用。

好的 linter 會有清楚文件，說明每條規則在抓什麼、為何不好、以及更佳替代寫法。例如可以看 [Ruff](https://docs.astral.sh/ruff/) 的 [SIM102](https://docs.astral.sh/ruff/rules/collapsible-if/) 規則，它會抓出 Python 中不必要的巢狀 `if`。

有些 linter 不只會標記問題，還能自動修正部分問題。

除了語言專屬 linter，另一個常用工具是 [semgrep](https://github.com/semgrep/semgrep)。它是「語意版 grep」，在 AST 層級運作（不像 grep 僅是字元層級），支援多種語言。你可以用 semgrep 輕鬆為專案寫客製規則。例如想禁止 Python 中危險的 `subprocess.Popen(..., shell=True)`，可用：

```bash
semgrep -l python -e "subprocess.Popen(..., shell=True, ...)"
```

# 測試（Testing）

軟體測試是提升「程式正確性信心」的標準方法。你先寫功能程式碼，再寫測試程式碼去驗證行為；若結果不符預期，測試就會報錯。

你可以在不同粒度撰寫測試：單一函式的 _unit tests_、模組／服務互動的 _integration tests_、端到端情境的 _functional tests_。你也可以採用 _test-driven development_（先寫測試再寫實作）。當發現 bug 時，可以補 _regression tests_，避免未來同類問題回歸。你也可以做 _property-based tests_（最早由 Haskell 的 [QuickCheck](https://hackage.haskell.org/package/QuickCheck) 推廣，Python 的 [Hypothesis](https://hypothesis.readthedocs.io/) 等函式庫也有實作）。適合哪種方法取決於專案，實務上多半會混合使用。

如果程式依賴外部資源（例如資料庫或 Web API），測試時通常以 _mock_ 取代真實依賴會更穩定，也更容易除錯。

## 程式碼覆蓋率（Code coverage）

Code coverage 是衡量測試覆蓋程度的指標。它會記錄測試執行時哪些程式行被跑過，幫你檢查是否覆蓋到不同程式路徑。覆蓋率工具通常可提供逐行結果，協助補強測試。[Codecov](https://app.codecov.io) 等服務也提供網頁介面，追蹤專案歷史覆蓋率。

和其他指標一樣，覆蓋率並不完美；不要只追數字，重點仍是寫出高品質測試。

# Pre-commit hooks

Git 的 pre-commit [hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)（搭配 [pre-commit](https://pre-commit.com/) 框架更方便）會在每次 commit 前自動執行你指定的檢查。專案常用它在每次提交前自動跑 formatter、linter，有時也會跑測試，確保提交內容符合專案風格且不含特定問題。

# 持續整合（Continuous integration）

持續整合（CI）服務（如 [GitHub Actions](https://github.com/features/actions)）可在每次 push、每個 pull request 或排程時間自動執行腳本。開發者常用 CI 跑程式碼品質工具，包含 formatter、linter、測試。對編譯型語言可驗證是否可編譯；對靜態型別語言可驗證型別檢查。每次 push 跑 CI 可及早發現主分支新引入的錯誤；在 PR 上跑 CI 可攔截貢獻內容問題；排程跑 CI 則可提早發現外部依賴變動造成的影響（例如某套件誤發了雖標示 [semver 相容](/2026/shipping-code/#releases--versioning)但其實破壞相容性的版本）。

由於 CI 腳本是在開發者機器之外執行，你可以更容易安排長時間作業。例如跑跨作業系統、跨語言版本的 _matrix_ 測試，確認軟體在各環境都能正常運作。

一般來說，CI 不會直接改你的程式碼，而是用「只檢查（check-only）」模式而非「自動修復（fix）」模式執行工具。例如格式不符時，formatter 會報錯而不會幫你改檔。

專案常在 README 放上[狀態徽章](https://docs.github.com/en/actions/how-tos/monitor-workflows/add-a-status-badge)，顯示 CI 結果與覆蓋率等資訊。以下是 Missing Semester 目前的建置狀態。

[![Build Status](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml) [![Links Status](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml)

> 我們的[連結檢查器](https://github.com/missing-semester/missing-semester/blob/master/.github/workflows/links.yml)使用 [proof-html](https://github.com/anishathalye/proof-html) GitHub Action，常因第三方網站問題而失敗。即便如此，它仍幫我們抓出並修正許多失效連結（有些是拼字錯誤，多數則是網站搬動內容卻未設轉址，或網站直接消失）。

學 CI、formatter、linter、測試函式庫的最佳方式之一是看範例。到 GitHub 找高品質開源專案——越接近你專案的語言、領域、規模越好——研究它們的 `pyproject.toml`、`.github/workflows/`、`DEVELOPMENT.md` 等相關檔案。

## 持續部署（Continuous deployment）

持續部署會利用 CI 基礎設施直接 _部署_ 變更。例如 Missing Semester 專案使用持續部署到 GitHub Pages：每次 `git push` 更新講義後，網站就會自動建置與部署。你也可以在 CI 產出其他[產物（artifacts）](/2026/shipping-code/)，例如應用程式執行檔或服務用 Docker 映像。

# 指令執行器（Command runners）

像 [just](https://github.com/casey/just) 這類 command runner 可簡化專案內指令執行。當你逐步建立品質流程時，不會希望團隊死背像 `uv run ruff check --fix` 這種長指令。透過 command runner 可改成 `just lint`，也可提供 `just format`、`just typecheck` 等一致入口。

有些語言的專案／套件管理器已內建類似功能，因此你不一定要用語言無關工具（如 `just`）。例如 Node.js 的 [npm](https://nodejs.org/en/learn/getting-started/an-introduction-to-the-npm-package-manager) 在 `package.json` 的 `scripts` 區塊，或 Python 的 [Hatch](https://hatch.pypa.io/) 在 `pyproject.toml` 的 `tool.hatch.envs.*.scripts` 區塊，都能做到。

# 正規表示式（Regular expressions）

_Regular expressions_（常簡稱 regex）是一種用來表示字串集合的語言。Regex 樣式常用於各種情境下的樣式匹配，例如命令列工具與 IDE。像 [ag](https://github.com/ggreer/the_silver_searcher) 支援全專案 regex 搜尋（例如 `ag "import .* as .*"` 可找出 Python 中被重新命名的 import），[go test](https://pkg.go.dev/cmd/go#hdr-Test_packages) 也支援 `-run [regexp]` 來挑選測試子集合。此外，多數程式語言都有內建或第三方 regex 函式庫，可用於匹配、驗證與解析。

為了建立直覺，下面列出幾個 regex 範例。本講使用 [Python regex syntax](https://docs.python.org/3/library/re.html)。Regex 有很多方言，尤其在進階功能上會有差異。你可用 [regex101](https://regex101.com/) 這類線上工具開發與除錯。

- `abc` --- 匹配字面值 "abc"。
- `missing|semester` --- 匹配字串 "missing" 或 "semester"。
- `\d{4}-\d{2}-\d{2}` --- 匹配 YYYY-MM-DD 格式日期，例如 "2026-01-14"。它只保證格式為四位數-兩位數-兩位數，不保證日期真的有效，因此 "2026-01-99" 也會匹配。
- `.+@.+` --- 匹配含有 `@` 的字串（前後都有內容），可粗略抓 email。但它只做最基本驗證，像 "nonsense@@@email" 也會匹配。理論上有更完整的 email regex [可用](https://pdw.ex-parrot.com/Mail-RFC822-Address.html)，但不實務。

## Regex 語法

完整語法可參考[官方文件](https://docs.python.org/3/library/re.html#regular-expression-syntax)（或其他線上資源）。以下是一些基本構件：

- `abc`：當字元無特殊意義時，匹配字面值字串（此例為 "abc"）
- `.`：匹配任意單一字元
- `[abc]`：匹配中括號內任一字元（此例為 "a"、"b"、"c"）
- `[^abc]`：匹配不在中括號內的任一字元（例如 "d"）
- `[a-f]`：匹配範圍內任一字元（例如 "c"，不含 "q"）
- `a|b`：匹配任一分支（例如 "a" 或 "b"）
- `\d`：匹配任一數字字元（例如 "3"）
- `\w`：匹配任一單字字元（例如 "x"）
- `\b`：匹配單字 _邊界_（例如在 "missing semester" 中，可匹配 m 前、g 後、s 前、r 後）
- `(...)`：匹配一個分組
- `...?`：匹配某樣式的 0 或 1 次，例如 `words?` 可匹配 "word" 或 "words"
- `...*`：匹配某樣式的任意次數，例如 `.*` 匹配任意字元任意次
- `...+`：匹配某樣式的 1 次以上，例如 `\d+` 匹配 1 個以上數字
- `...{N}`：匹配某樣式剛好 N 次，例如 `\d{4}` 匹配 4 位數
- `\.`：匹配字面值 `.`
- `\\`：匹配字面值 `\`
- `^`：匹配行首
- `$`：匹配行尾

## 捕獲群組與參照

若使用 regex 群組 `(...)`，可參照匹配到的子區段做擷取或搜尋取代。例如要從 YYYY-MM-DD 日期中擷取月份，可用：

```python
>>> import re
>>> re.match(r"\d{4}-(\d{2})-\d{2}", "2026-01-14").group(1)
'01'
```

在文字編輯器中，你也可在取代樣式裡引用捕獲群組。不同 IDE 語法可能不同。例如 VS Code 可用 `$1`、`$2`，Vim 可用 `\1`、`\2` 參照群組。

## 限制

[Regular languages](https://en.wikipedia.org/wiki/Regular_language) 雖然強大但有限制；有些字串集合無法用標準 regex 表達（例如[不可能](https://en.wikipedia.org/wiki/Pumping_lemma_for_regular_languages)寫出匹配 {a^n b^n \| n &ge; 0} 的正規表示式，也就是同數量 a 後接同數量 b；更實務地說，像 HTML 這類語言不是 regular language）。現代 regex 引擎雖支援 lookahead、backreference 等擴充，實用性很高，但表達能力依然有限。面對更複雜語言時，你可能需要更完整的 parser（例如 [pyparsing](https://github.com/pyparsing/pyparsing)，屬於 [PEG](https://en.wikipedia.org/wiki/Parsing_expression_grammar) parser）。

## 如何學 regex

我們建議先掌握基礎（也就是本講內容），之後按需查語法，而不是試圖一次背完整個 regex 語言。

對話式 AI 工具也很適合協助產生 regex。你可以試著對喜歡的 LLM 輸入以下提示：

```
Write a Python-style regex pattern that matches the requested path from log lines from Nginx. Here is an example log line:

169.254.1.1 - - [09/Jan/2026:21:28:51 +0000] "GET /feed.xml HTTP/2.0" 200 2995 "-" "python-requests/2.32.3"
```

# 練習

1. 在你正在開發的專案中設定 formatter、linter 與 pre-commit hooks。若錯誤很多：格式問題可先靠 autoformatting。對 linter 問題，試著用 [AI agent](/2026/agentic-coding/) 修正全部錯誤。請確保 AI agent 能實際執行 linter 並看到結果，才能迭代修到乾淨。最後務必人工檢查，確認 AI 沒把程式改壞。
1. 選一個你熟悉語言的測試函式庫，為手邊專案寫一個單元測試。執行 coverage 工具，產出 HTML 覆蓋率報告並觀察：你看得出哪些行被覆蓋嗎？一開始覆蓋率通常很低。先手動補一些測試來提升，再試著用 [AI agent](/2026/agentic-coding/) 協助提高覆蓋率；請確保 agent 能跑含 coverage 的測試並產出逐行報告，才能知道該補哪裡。AI 生成的測試真的有品質嗎？
1. 為你的專案設定每次 push 都會執行的 CI。讓 CI 跑格式化、lint 與測試。接著故意引入錯誤（例如 linter 違規），確認 CI 能正確攔截。
1. 嘗試自己寫一個[regex 樣式](#regular-expressions)，並用 `grep` 這個[命令列工具](/2026/course-shell/) 在你的程式碼中找 `subprocess.Popen(..., shell=True)`。接著想辦法「打破」你的 regex。當 grep 失手時，[semgrep](#linting) 是否仍能抓到危險寫法？
1. 在 IDE 或文字編輯器中練習 regex 搜尋與取代：把[這份講義](https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/code-quality.md)中的 `-` [Markdown 項目符號](https://spec.commonmark.org/0.31.2/#bullet-list-marker) 改成 `*`。注意：不能直接把檔案所有 `-` 都取代，因為很多 `-` 不是項目符號。
1. 寫一個 regex，從 `{"name": "Alyssa P. Hacker", "college": "MIT"}` 這種 JSON 結構中擷取 name（此例為 `Alyssa P. Hacker`）。提示：第一版你可能會錯抓成 `Alyssa P. Hacker", "college": "MIT`；可閱讀 [Python regex 文件](https://docs.python.org/3/library/re.html)中的 greedy quantifier 說明來修正。
    1. 讓 regex 在 name 內含 `"` 時也能正常運作（JSON 可用 `\"` 跳脫雙引號）。
    1. 實務上我們**不建議**用 regex 解決複雜解析問題。請改用你使用語言的 JSON parser 完成這題：寫一個命令列程式，從 stdin 讀入上述 JSON 結構，在 stdout 輸出 name。這題通常只要幾行程式；以 Python 來說，扣掉 `import json`，一行就能完成。
