---
layout: lecture
title: "開發環境與工具"
description: >
  學習 IDE、Vim、語言伺服器與 AI 輔助開發工具。
thumbnail: /static/assets/thumbnails/2026/lec3.png
date: 2026-01-14
ready: true
video:
  aspect: 56.25
  id: QnM1nVzrkx8
---

_開發環境（development environment）_ 是一組用來開發軟體的工具。核心是文字編輯能力，以及語法上色、型別檢查、程式碼格式化、自動補完等功能。像 [VS Code][vs-code] 這類 _整合開發環境（IDE）_，會把這些能力整合在同一個應用程式中。以終端機為主的工作流程則會組合多個工具，例如 [tmux](https://github.com/tmux/tmux)（終端機多工器）、[Vim](https://www.vim.org/)（文字編輯器）、[Zsh](https://www.zsh.org/)（shell），以及語言專屬命令列工具，如 [Ruff](https://docs.astral.sh/ruff/)（Python linter 與 formatter）和 [Mypy](https://mypy-lang.org/)（Python 型別檢查器）。

IDE 與終端機工作流程各有優缺點。例如圖形化 IDE 通常較好上手，且現在多半內建較好的 AI 功能（如 AI 自動補完）；另一方面，終端機流程更輕量，也可能是在沒有 GUI 或無法安裝軟體環境中的唯一選擇。建議你兩者都先有基本熟悉度，並至少精通其中一種。如果你還沒有偏好的 IDE，建議從 [VS Code][vs-code] 開始。

本講會介紹：

- [文字編輯與 Vim](#text-editing-and-vim)
- [程式碼智慧與語言伺服器](#code-intelligence-and-language-servers)
- [AI 輔助開發](#ai-powered-development)
- [擴充套件與其他 IDE 功能](#extensions-and-other-ide-functionality)

[vs-code]: https://code.visualstudio.com/

# 文字編輯與 Vim

寫程式時，你多半是在程式碼間移動、閱讀片段並修改內容，而不是從頭到尾逐行閱讀或一次寫很長段文字。[Vim] 是專門為這種工作分布優化的編輯器。

**Vim 的核心哲學。** Vim 的基礎概念很優雅：它的介面本身就是一門「用來移動與編輯文字」的程式語言。按鍵（具有記憶性命名）是指令，而這些指令可以組合。Vim 盡量避免滑鼠，因為太慢；甚至也盡量少用方向鍵，因為手部移動太多。結果是：它像腦機介面一樣，操作速度能逼近你的思考速度。

**其他軟體中的 Vim 支援。** 你不一定要直接使用 [Vim] 才能受益於它的核心理念。許多涉及文字編輯的工具都支援「Vim mode」，可能是內建或透過外掛。例子包含：VS Code 的 [VSCodeVim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim) 外掛、Zsh 的 [內建 Vim 模擬支援](https://zsh.sourceforge.io/Guide/zshguide04.html)，甚至 Claude Code 也有[內建 Vim 編輯模式](https://code.claude.com/docs/en/interactive-mode#vim-editor-mode)。你常用的文字工具，很可能都能用某種方式啟用 Vim mode。

## 模式化編輯（Modal editing）

Vim 是 _modal editor_：針對不同類型任務有不同操作模式。

- **Normal**：移動與編輯
- **Insert**：插入文字
- **Replace**：取代文字
- **Visual**（一般、整行、區塊）：選取文字區塊
- **Command-line**：執行命令

同一按鍵在不同模式下意義不同。比如 `x` 在 Insert 模式只會輸入字元 `"x"`；在 Normal 模式會刪除游標下字元；在 Visual 模式會刪除選取內容。

預設設定下，Vim 會在左下角顯示目前模式。初始模式是 Normal。你大多時間會在 Normal 與 Insert 之間切換。

按 `<ESC>`（Esc 鍵）可從任何模式回到 Normal。從 Normal 可用 `i` 進入 Insert、`R` 進入 Replace、`v` 進入 Visual、`V` 進入 Visual Line、`<C-v>`（Ctrl-V，也常寫作 `^V`）進入 Visual Block、`:` 進入 Command-line。

使用 Vim 時會大量按 `<ESC>`，可以考慮把 Caps Lock 重映射成 Escape（[macOS 教學](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)），或設定一組[替代按鍵映射](https://vim.fandom.com/wiki/Avoid_the_escape_key#Mappings)。

## 基本操作：插入文字

在 Normal 模式按 `i` 進入 Insert。此時 Vim 行為和一般編輯器類似，直到你按 `<ESC>` 回到 Normal。光靠這些基礎就能開始用 Vim 編輯檔案（雖然若長時間停留在 Insert，效率不會太高）。

## Vim 介面是一門程式語言

Vim 的介面就是程式語言：按鍵是指令，而指令可 _組合_。這使移動與編輯非常高效，尤其當操作變成肌肉記憶後，就像熟悉鍵盤配置後打字會變得很快。

### 移動（Movement）

你應該大部分時間待在 Normal，靠移動指令在檔案中導航。Vim 把移動也稱為「名詞（nouns）」，因為它們描述的是文字範圍。

- 基礎移動：`hjkl`（左、下、上、右）
- 單字：`w`（下個字）、`b`（字首）、`e`（字尾）
- 行：`0`（行首）、`^`（第一個非空白字元）、`$`（行尾）
- 螢幕：`H`（螢幕上方）、`M`（中間）、`L`（下方）
- 捲動：`Ctrl-u`（上捲）、`Ctrl-d`（下捲）
- 檔案：`gg`（檔案開頭）、`G`（檔案結尾）
- 行號：`:{number}<CR>` 或 `{number}G`（跳到第 {number} 行）
    - `<CR>` 指的是 Enter 鍵
- 其他：`%`（跳到配對項目，如括號）
- 尋字元：`f{character}`、`t{character}`、`F{character}`、`T{character}`
    - 在當前行向前／向後尋找（或移動到）指定字元
    - `,` / `;` 可在匹配間移動
- 搜尋：`/{regex}`，搭配 `n` / `N` 切換匹配

### 選取（Selection）

Visual 模式：

- Visual：`v`
- Visual Line：`V`
- Visual Block：`Ctrl-v`

可搭配移動鍵擴展選取範圍。

### 編輯（Edits）

過去用滑鼠做的事，在 Vim 中可用鍵盤與「編輯指令 + 移動指令」組合完成。這正是 Vim 看起來像程式語言的地方。編輯指令也被稱為「動詞（verbs）」，因為動詞會作用在名詞上。

- `i`：進入 Insert 模式
    - 但若要高效率操作／刪改文字，會比一直按 Backspace 更進一步
- `o` / `O`：在下方／上方插入新行
- `d{motion}`：刪除 {motion}
    - 例如 `dw` 刪字、`d$` 刪到行尾、`d0` 刪到行首
- `c{motion}`：變更 {motion}
    - 例如 `cw` 變更單字
    - 可理解為 `d{motion}` 後接 `i`
- `x`：刪字元（等價 `dl`）
- `s`：取代字元（等價 `cl`）
- Visual 模式 + 編輯
    - 選取後用 `d` 刪除，或用 `c` 變更
- `u` 復原，`<C-r>` 重做
- `y` 複製（yank；某些指令如 `d` 也會把內容放入暫存）
- `p` 貼上
- 還有很多可學：例如 `~` 可切換字母大小寫，`J` 可合併行

### 次數（Counts）

名詞與動詞可搭配次數，讓動作重複執行指定次數。

- `3w` 向前移動 3 個字
- `5j` 向下移動 5 行
- `7dw` 刪除 7 個字

### 修飾子（Modifiers）

你可以用修飾子改變名詞語意。常見修飾子包括 `i`（inner/inside）與 `a`（around）。

- `ci(` 變更目前括號內內容
- `ci[` 變更目前中括號內內容
- `da'` 刪除單引號字串（含外層單引號）

## 綜合範例

下面是一個有問題的 [fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz) 實作：

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print("fizz", end="")
        if i % 5 == 0:
            print("fizz", end="")
        if i % 3 and i % 5:
            print(i, end="")
        print()

def main():
    fizz_buzz(20)
```

我們可在 Normal 模式下用以下指令序列修掉問題：

- Main 沒有被呼叫
    - `G` 跳到檔案結尾
    - `o` 在下方 **o**pen 新行
    - 輸入 `if __name__ == "__main__": main()`
        - 如果編輯器有 Python 支援，Insert 模式可能會自動縮排
    - `<ESC>` 回到 Normal
- 迴圈從 0 開始而非 1
    - `/` 後接 `range` 再按 `<CR>` 搜尋 "range"
    - `ww` 向前移動兩個 **w**ord（也可 `2w`，但小次數時常直接重複按鍵）
    - `i` 進入 **i**nsert，加入 `1,`
    - `<ESC>` 回到 Normal
    - `e` 跳到下個字的 **e**nd
    - `a` 開始 **a**ppend，加入 `+ 1`
    - `<ESC>` 回到 Normal
- 5 的倍數被印成 "fizz"
    - `:6<CR>` 跳到第 6 行
    - `ci"`（**c**hange **i**nside `"`）改成 `"buzz"`
    - `<ESC>` 回到 Normal

## 學習 Vim

學 Vim 最好的方式是先掌握基礎（也就是上面內容），接著把常用工具盡量開啟 Vim mode，直接在實作中使用。盡量忍住滑鼠與方向鍵的習慣；有些編輯器可直接取消方向鍵綁定，幫助你養成好習慣。

### 延伸資源

- 上一輪課程的 [Vim 講座](/2020/editors/)——該講更深入
- `vimtutor` 是 Vim 內建教學——若已安裝 Vim，應可在 shell 直接執行 `vimtutor`
- [Vim Adventures](https://vim-adventures.com/)：用遊戲學 Vim
- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/)：各種 Vim 技巧
- [VimGolf](https://www.vimgolf.com/)：把 [code golf](https://en.wikipedia.org/wiki/Code_golf) 搬到 Vim 介面上
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/titles/dnvim2/) (book)

[Vim]: https://www.vim.org/

# 程式碼智慧與語言伺服器

IDE 通常透過連到 _language server_ 的擴充套件來提供語言專屬能力，這些伺服器實作了 [Language Server Protocol](https://microsoft.github.io/language-server-protocol/)，能對程式碼做語意層級理解。例如 VS Code 的 [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python) 依賴 [Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance)，而 [Go extension](https://marketplace.visualstudio.com/items?itemName=golang.go) 依賴官方 [gopls](https://go.dev/gopls/)。安裝你所用語言的 extension 與 language server 後，IDE 可啟用許多語言專屬能力，例如：

- **Code completion。** 更好的自動補完與提示，例如輸入 `object.` 後可看到欄位與方法。
- **Inline documentation。** 滑鼠懸停與補完時可直接看到文件。
- **Jump-to-definition。** 從使用位置跳到定義，例如由 `object.field` 跳到欄位定義處。
- **Find references。** 反向查詢，找出某欄位或型別被引用的所有位置。
- **Import 協助。** 整理 import、移除未使用 import、提示缺少 import。
- **Code quality。** 這些工具可獨立使用，也常由 language server 一併提供。格式化可自動縮排與整理；型別檢查器與 linter 可在輸入時即時找錯。我們會在[程式碼品質講座](/2026/code-quality/)更深入介紹。

## 設定 language server

對某些語言來說，安裝 extension 與 language server 就能直接用。對另一些語言，若要發揮最大效果，你需要讓 IDE 知道你的執行環境。例如在 VS Code 指向你的 [Python environment](https://code.visualstudio.com/docs/python/environments)，language server 才能辨識你已安裝的套件。環境管理會在[打包與交付講座](/2026/shipping-code/)更深入說明。

依語言不同，language server 可能還有額外可調設定。例如在 VS Code 的 Python 支援中，若專案未使用 Python 可選型別註記，你可關閉靜態型別檢查。

# AI 輔助開發

自 2021 年中 [GitHub Copilot][github-copilot] 採用 OpenAI 的 [Codex model](https://openai.com/index/openai-codex/) 推出以來，[LLM](https://en.wikipedia.org/wiki/Large_language_model) 已在軟體工程中廣泛普及。目前主要有三種形式：autocomplete、inline chat、coding agent。

[github-copilot]: https://github.com/features/copilot/ai-code-editor

## Autocomplete

AI autocomplete 在介面上和傳統自動補完很像：你打字時，它會在游標處提供補全。它可以是「自然就生效」的被動功能；進一步也可透過程式碼註解做[提示工程](https://en.wikipedia.org/wiki/Prompt_engineering)來引導結果。

例如我們要寫個腳本：下載這份講義並擷取所有連結。可先從這段開始：

```python
import requests

def download_contents(url: str) -> str:
```

模型可能會自動補完整個函式內容：

```python
    response = requests.get(url)
    return response.text
```

我們也可用註解進一步引導補全。例如若函式命名不夠描述性：

```python
def extract(contents: str) -> list[str]:
```

模型可能會補成這樣：

```python
    lines = contents.splitlines()
    return [line for line in lines if line.strip()]
```

加入註解後可更精準引導：

```python
def extract(content: str) -> list[str]:
    # extract all Markdown links from the content
```

這次模型會給出更好的補全：

```python
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)
```

這裡可看出一個限制：它只能在游標位置做補全。此例更好的寫法其實是把 `import re` 放在模組層級，而不是函式內。

上例故意用較差命名來示範註解如何引導補全；實務中你應使用更清楚的函式名（如 `extract_links`），並撰寫 docstring（模型也應能據此生成類似上面的高品質補全）。

示範起見，我們可把腳本補完整：

```python
print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

## Inline chat

Inline chat 讓你選取某行或區塊後，直接要求 AI 提出修改。在這種互動模式下，模型可直接改既有程式碼（不同於 autocomplete 只能補游標後方）。

延續前例，假設我們不想用第三方 `requests` 函式庫。可選取相關三行後啟用 inline chat，輸入：

```
use built-in libraries instead
```

模型可能建議：

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')
```

## Coding agents

Coding agent 會在 [Agentic Coding](/2026/agentic-coding/) 講座深入介紹。

## 推薦軟體

常見 AI IDE 包含安裝 [GitHub Copilot][github-copilot] 外掛的 [VS Code][vs-code]，以及 [Cursor](https://cursor.com/)。GitHub Copilot 目前對[學生](https://github.com/education/students)、教師與熱門開源專案維護者提供免費方案。這個領域變化很快，主流產品的核心功能也大致接近。

# 擴充套件與其他 IDE 功能

IDE 本身已很強大，加上 _extensions_ 更是如虎添翼。我們無法在一堂課涵蓋所有功能，這裡先提供幾個常見方向。也鼓勵你自行探索，網路上有很多熱門擴充套件清單，例如 Vim 外掛網站 [Vim Awesome](https://vimawesome.com/) 與依安裝數排序的 [VS Code extensions](https://marketplace.visualstudio.com/search?target=VSCode&category=All%20categories&sortBy=Installs)。

- [Development containers](https://containers.dev/)：主流 IDE 都支援（如 [VS Code 支援](https://code.visualstudio.com/docs/devcontainers/containers)）。可在容器內跑開發工具，有助可攜性與隔離性。容器會在[打包與交付講座](/2026/shipping-code/)深入說明。
- Remote development：透過 SSH 在遠端機器開發（例如 VS Code 的 [Remote SSH plugin](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)）。若你要在雲端高效能 GPU 主機開發與執行程式，這很實用。
- Collaborative editing：多人同時編輯同檔（類似 Google Docs；例如 VS Code 的 [Live Share plugin](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare)）。

# 練習

1. 在所有支援 Vim mode 的工具中（如編輯器與 shell）啟用 Vim mode，並在接下來一個月的文字編輯都盡量使用它。只要覺得某件事很低效，或想到「一定有更好做法」，就去搜尋看看；通常真的有更好做法。
1. 完成一題 [VimGolf](https://www.vimgolf.com/) 挑戰。
1. 為你正在開發的專案設定 IDE 擴充套件與 language server。確認預期功能（如跳到第三方函式庫定義）都正常。若手上沒有可用專案，可改用 GitHub 開源專案（例如[這個](https://github.com/spf13/cobra)）。
1. 瀏覽 IDE 擴充套件清單，安裝一個你覺得實用的擴充套件。
