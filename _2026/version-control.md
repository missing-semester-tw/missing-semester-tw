---
layout: lecture
title: "版本控制與 Git"
description: >
  學習 Git 的資料模型，以及如何使用 Git 進行版本控制與協作。
thumbnail: /static/assets/thumbnails/2026/lec5.png
date: 2026-01-16
ready: true
video:
  aspect: 56.25
  id: 9K8lB61dl3Y
---

版本控制系統（VCS）是用來追蹤原始碼（或其他檔案與資料夾集合）變更的工具。如同名稱所示，這些工具能幫助你維護變更歷史；此外，它們也讓協作更容易。
從邏輯上來看，VCS 會以一系列的 _快照（snapshot）_ 追蹤某個資料夾及其內容的變化，其中每個快照都封裝了最上層目錄下檔案／資料夾的完整狀態。VCS 也會維護中繼資料，例如每個快照是誰建立的、快照的訊息等等。

為什麼版本控制很有用？即使你一個人開發，它也能讓你查看專案的舊快照、記錄特定變更為何發生、平行開發不同分支，還有更多用途。當你與他人合作時，它更是不可或缺：你可以看到別人改了什麼，也能處理並行開發時的衝突。

現代的 VCS 也能讓你輕鬆（而且常常是自動地）回答像這樣的問題：

- 這個模組是誰寫的？
- 這個檔案中的某一行是什麼時候被改的？誰改的？為什麼改？
- 在最近 1000 次修訂中，某個單元測試是從什麼時候開始壞掉的？為什麼？

雖然還有其他 VCS，但 **Git** 已經是版本控制的事實標準。這張 [XKCD 漫畫](https://xkcd.com/1597/) 很貼切地呈現了 Git 的名聲：

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

由於 Git 的介面是「抽象層有滲漏」的設計，若用由上而下的方式學 Git（從介面／命令列介面開始）很容易讓人困惑。你可能只背了幾個指令，把它們當成咒語，然後每次出錯就照著上面漫畫那種方式處理。

雖然 Git 的介面確實不太好看，但它底層的設計與概念其實很漂亮。不好看的介面常常只能 _硬背_，漂亮的設計則可以 _理解_。因此我們會用由下而上的方式解釋 Git：先從資料模型開始，再談命令列介面。當你理解資料模型後，就能更清楚指令到底在如何操作底層資料模型。

# Git 的資料模型

Git 的巧妙之處在於它經過深思熟慮的資料模型，正是這個模型讓版本控制的各種功能成為可能，例如維護歷史、支援分支，以及促進協作。

## 快照（Snapshots） {#snapshots}

Git 會把某個最上層目錄中一組檔案與資料夾的歷史，建模成一系列快照。在 Git 術語裡，檔案叫做「blob」，本質上就是一串位元組。資料夾叫做「tree」，它會把名稱對應到 blob 或 tree（所以資料夾裡可以再包含其他資料夾）。一個快照就是被追蹤的最上層 tree。舉例來說，我們可能有這樣一棵 tree：

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

最上層 tree 包含兩個元素：一個是 tree「foo」（它自己又包含一個元素，也就是 blob「bar.txt」），另一個是 blob「baz.txt」。

## 歷史建模：如何關聯快照

版本控制系統該如何把快照彼此關聯起來？一個簡單模型是線性歷史，也就是按時間順序排列的快照清單。但基於很多原因，Git 並沒有採用這種簡單模型。

在 Git 裡，歷史是一張由快照構成的有向無環圖（DAG）。聽起來像很數學的詞，但不用害怕。它的意思只是：Git 中每個快照都會參照一組「父節點（parents）」，也就是它之前的快照。之所以是「一組」而不是單一父節點（像線性歷史那樣），是因為某個快照可能同時來自多個父節點，例如把兩條平行開發分支合併（merge）時就會發生。

Git 把這些快照稱為「commit」。把 commit 歷史視覺化後可能像這樣：

```
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

在上面的 ASCII 圖中，`o` 代表各個 commit（快照）。箭頭指向每個 commit 的父節點（是「先於」關係，不是「後於」關係）。第三個 commit 之後，歷史分岔成兩條獨立分支。這可能對應到兩個不同功能在平行開發、彼此互不依賴。未來這些分支可以再合併，產生一個同時包含兩個功能的新快照，形成如下的新歷史，其中新建立的 merge commit 以粗體標示：

<pre class="highlight">
<code>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \          v
              --- o <-- o
</code>
</pre>

Git 中的 commit 是不可變（immutable）的。不過這不代表錯誤不能修正；只是對 commit 歷史做「修改」時，實際上是在建立全新的 commit，並把參照（見下文）更新成指向新的 commit。

## 用偽程式碼看資料模型

用偽程式碼寫下 Git 的資料模型，通常很有幫助：

```
// 檔案就是一串位元組
type blob = array<byte>

// 資料夾包含具名檔案與子資料夾
type tree = map<string, tree | blob>

// commit 包含父節點、中繼資料與最上層 tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

這是一個乾淨、簡單的歷史模型。

## 物件與內容定址（content-addressing）

「object」可以是 blob、tree 或 commit：

```
type object = blob | tree | commit
```

在 Git 的資料儲存中，所有 object 都透過其 [SHA-1 雜湊值](https://en.wikipedia.org/wiki/SHA-1) 進行內容定址。

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

blob、tree 和 commit 在這裡被統一看待：它們都是 object。當它們參照其他 object 時，磁碟上的表示法裡並不會真的 _包含_ 對方內容，而是透過雜湊值去參照對方。

例如，上面[快照範例](#snapshots)中的資料夾結構，其 tree（用 `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d` 顯示）長這樣：

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

這個 tree 本身包含指向其內容的指標：`baz.txt`（blob）與 `foo`（tree）。如果我們用 `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85` 查看對應 `baz.txt` 雜湊值所定址的內容，會得到：

```
git is wonderful
```

## 參照（References）

現在，所有快照都可以透過 SHA-1 雜湊值識別。不過這很不方便，因為人類不擅長記住 40 個十六進位字元的長字串。

Git 解決這個問題的方法，是替 SHA-1 雜湊值提供人可讀名稱，稱為「references」。reference 是指向 commit 的指標。與不可變的 object 不同，reference 是可變的（可以更新成指向新的 commit）。例如，`master` reference 通常會指向主要開發分支上的最新 commit。

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

有了這個機制，Git 就能用像「master」這樣的人可讀名稱來指向歷史中的特定快照，而不用那串很長的十六進位字串。

其中有個細節是：我們常常需要知道自己「目前位在歷史的哪裡」，這樣在建立新快照時，才知道它是相對於哪個位置（也就是 commit 的 `parents` 欄位該怎麼設定）。在 Git 裡，這個「目前位置」是一個特殊 reference，叫做「HEAD」。

## 儲存庫（Repositories）

最後，我們可以粗略定義什麼是 Git _repository_：它就是 `objects` 與 `references` 這兩類資料。

在磁碟上，Git 儲存的全部內容就是 object 與 reference：這就是 Git 資料模型的全部。所有 `git` 指令，本質上都對應到某種對 commit DAG 的操作：新增 object，或新增／更新 reference。

每次你輸入任何指令時，都可以想想這個指令正在如何操作底層圖形資料結構。反過來說，如果你想對 commit DAG 做某種特定變更，例如「捨棄尚未提交的變更，並讓 `master` ref 指向 commit `5d83f9e`」，通常都會有對應指令（例如此例可用 `git checkout master; git reset --hard 5d83f9e`）。

# 暫存區（Staging area）

這是另一個和資料模型正交的概念，但它是建立 commit 時介面的一部分。

你可能會想像，上面提到的快照機制可以用一個「create snapshot」指令實作，直接依據工作目錄的 _目前狀態_ 建立新快照。有些版本控制工具確實這樣做，但 Git 不是。我們想要的是乾淨的快照，而從目前狀態直接建立快照不一定理想。舉例來說，假設你同時完成兩個獨立功能，希望切成兩個獨立 commit：第一個只包含功能 A，下一個只包含功能 B。又或者你為了除錯在程式到處加了 print，同時也修了一個 bug；你可能想只提交 bug 修正，把那些 print 全部丟掉。

Git 透過「staging area（暫存區）」來支援這些情境：你可以明確指定哪些修改要納入下一個快照。

# Git 命令列介面

為了避免重複，這份講義不會詳細解釋下面的每個指令。更多內容請參考強烈推薦的 [Pro Git](https://git-scm.com/book/en/v2)，或觀看課程影片。

## 基礎

- `git help <command>`：查看某個 Git 指令的說明
- `git init`：建立新的 Git 儲存庫，資料存放在 `.git` 目錄
- `git status`：告訴你目前狀態
- `git add <filename>`：把檔案加入暫存區
- `git commit`：建立新的 commit
    - 請寫出[好的 commit 訊息](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)！
    - 這裡有更多寫[好 commit 訊息](https://chris.beams.io/posts/git-commit/)的理由！
- `git log`：顯示攤平後的歷史紀錄
- `git log --all --graph --decorate`：將歷史以 DAG 形式視覺化
- `git diff <filename>`：顯示相對於暫存區，你做了哪些修改
- `git diff <revision> <filename>`：顯示兩個快照間該檔案的差異
- `git checkout <revision>`：更新 HEAD（若 checkout 的是分支，也會更新目前分支）

## 分支與合併

- `git branch`：顯示分支
- `git branch <name>`：建立分支
- `git switch <name>`：切換到某個分支
- `git checkout -b <name>`：建立分支並切換過去
    - 等同 `git branch <name>; git switch <name>`
- `git merge <revision>`：把指定修訂合併到目前分支
- `git mergetool`：使用圖形化／輔助工具處理 merge 衝突
- `git rebase`：把一組修補（patch）重放到新的基底上

## 遠端（Remotes）

- `git remote`：列出遠端
- `git remote add <name> <url>`：新增遠端
- `git push <remote> <local branch>:<remote branch>`：把 object 傳到遠端，並更新遠端 reference
- `git branch --set-upstream-to=<remote>/<remote branch>`：設定本地分支與遠端分支的對應關係
- `git fetch`：從遠端抓取 object／reference
- `git pull`：等同 `git fetch; git merge`
- `git clone`：從遠端下載儲存庫

## 還原（Undo）

- `git commit --amend`：修改某個 commit 的內容／訊息
- `git reset <file>`：把檔案從暫存區移除（unstage）
- `git restore`：捨棄變更

# 進階 Git

- `git config`：Git [高度可自訂](https://git-scm.com/docs/git-config)
- `git clone --depth=1`：淺層複製，不含完整版本歷史
- `git add -p`：互動式暫存
- `git rebase -i`：互動式 rebase
- `git blame`：顯示每一行最後是誰修改的
- `git stash`：暫時移除工作目錄中的修改
- `git bisect`：在歷史中做二分搜尋（例如找出回歸問題）
- `git revert`：建立新 commit 來反轉較早 commit 的效果
- `git worktree`：同時 checkout 多個分支
- `.gitignore`：用來[指定](https://git-scm.com/docs/gitignore)要忽略、且刻意不追蹤的檔案

# 其他補充

- **GUI**：Git 有很多 [GUI client](https://git-scm.com/downloads/guis)。我們個人不太用，通常還是使用命令列介面。
- **Shell 整合**：把 Git 狀態顯示在 shell prompt 上非常方便（[zsh](https://github.com/olivierverdier/zsh-git-prompt)、[bash](https://github.com/magicmonty/bash-git-prompt)）。像 [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) 這類框架通常都內建。
- **編輯器整合**：和上面類似，也有很多實用整合功能。對 Vim 來說，[fugitive.vim](https://github.com/tpope/vim-fugitive) 是標準選擇。
- **工作流程**：我們教了資料模型與一些基本指令，但沒有規定你在大型專案中一定要採用哪種實務流程（而且其實有[很多](https://nvie.com/posts/a-successful-git-branching-model/)[不同](https://www.endoflineblog.com/gitflow-considered-harmful)[做法](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)）。
- **GitHub**：Git 不是 GitHub。GitHub 有一套將程式碼貢獻到其他專案的方法，叫做 [pull requests](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)。
- **其他 Git 服務商**：GitHub 並不特別，還有很多 Git 儲存庫託管服務，例如 [GitLab](https://about.gitlab.com/) 與 [BitBucket](https://bitbucket.org/)。

# 學習資源

- [Pro Git](https://git-scm.com/book/en/v2) 是**非常推薦**的讀物。既然你已經理解資料模型，讀完第 1～5 章，基本上就能掌握熟練使用 Git 所需的大部分能力。後面章節也有不少有趣的進階內容。
- [Oh Shit, Git!?!](https://ohshitgit.com/) 是一份短指南，教你如何從常見 Git 錯誤中復原。
- [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/) 也簡要說明了 Git 的資料模型，和本講義相比，偽程式碼更少、圖更豐富。
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/) 針對有興趣深入的人，詳細解釋了資料模型之外 Git 的實作細節。
- [How to explain git in simple words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [Learn Git Branching](https://learngitbranching.js.org/) 是一個在瀏覽器中進行、可學習 Git 的遊戲。

# 練習

1. 如果你以前沒有 Git 經驗，可以先讀 [Pro Git](https://git-scm.com/book/en/v2) 前幾章，或做像 [Learn Git Branching](https://learngitbranching.js.org/) 這樣的教學。在練習過程中，試著把 Git 指令和資料模型對應起來。
1. 複製（clone）[課程網站的儲存庫](https://github.com/missing-semester/missing-semester)。
    1. 把版本歷史畫成圖來探索它。
    1. 最後修改 `README.md` 的人是誰？（提示：`git log` 可以加參數）
    1. `_config.yml` 中 `collections:` 那一行最後一次修改所對應的 commit 訊息是什麼？（提示：用 `git blame` 與 `git show`）
1. 學 Git 常見錯誤之一，是把不該由 Git 管理的大檔案提交，或把敏感資訊加進去。試著在儲存庫裡加入一個檔案、做幾次 commit，然後把那個檔案從 _歷史_ 中刪掉（不只是最新 commit）。你可以參考[這篇](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)。
1. 從 GitHub 複製（clone）任一儲存庫，修改其中一個既有檔案。執行 `git stash` 會發生什麼？執行 `git log --all --oneline` 會看到什麼？再用 `git stash pop` 還原你剛剛 `git stash` 做的事。這在什麼情境下有用？
1. 和許多命令列工具一樣，Git 也有設定檔（dotfile），叫做 `~/.gitconfig`。在 `~/.gitconfig` 建立一個 alias，讓你執行 `git graph` 時，得到 `git log --all --graph --decorate --oneline` 的輸出。你可以直接[編輯](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias) `~/.gitconfig`，或用 `git config` 指令新增 alias。關於 git alias 的資訊可見[這裡](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases)。
1. 執行 `git config --global core.excludesfile ~/.gitignore_global` 後，你可以在 `~/.gitignore_global` 定義全域忽略規則。這只會設定 Git 使用的全域忽略檔位置，你仍然需要手動在該路徑建立檔案。請設定你的全域 gitignore，忽略作業系統或編輯器產生的暫存檔，例如 `.DS_Store`。
1. Fork [課程網站儲存庫](https://github.com/missing-semester/missing-semester)，找一個 typo 或其他可改進之處，然後在 GitHub 提交 pull request（可參考[這個](https://github.com/firstcontributions/first-contributions)）。請只提交有幫助的 PR（拜託不要洗版）。如果找不到可改進的地方，也可以跳過這題。
1. 模擬協作情境，練習解決 merge 衝突：
    1. 用 `git init` 建立新儲存庫，並建立名為 `recipe.txt` 的檔案，寫入幾行內容（例如簡單食譜）。
    1. 先 commit，接著建立兩個分支：`git branch salty` 與 `git branch sweet`。
    1. 在 `salty` 分支修改其中一行（例如把 "1 cup sugar" 改成 "1 cup salt"）並 commit。
    1. 在 `sweet` 分支把同一行改成不同內容（例如把 "1 cup sugar" 改成 "2 cups sugar"）並 commit。
    1. 現在切回 `master`，先試 `git merge salty`，再試 `git merge sweet`。會發生什麼？看看 `recipe.txt` 內容，`<<<<<<<`、`=======`、`>>>>>>>` 這些標記代表什麼？
    1. 編輯檔案保留你想要的內容、移除衝突標記，然後用 `git add` 與 `git commit`（或 `git merge --continue`）完成合併。你也可以試試 `git mergetool`，用圖形化或終端機 merge 工具解衝突。
    1. 使用 `git log --graph --oneline` 視覺化你剛建立的合併歷史。
