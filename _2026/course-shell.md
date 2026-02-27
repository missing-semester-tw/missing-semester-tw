---
layout: lecture
title: "課程總覽 + Shell 入門"
description: >
  了解這門課的核心動機，並開始學習 shell。
thumbnail: /static/assets/thumbnails/2026/lec1.png
date: 2026-01-12
ready: true
video:
  aspect: 56.25
  id: MSgoeuMqUmU
---

# 我們是誰？

這門課由 [Anish](https://anish.io/)、[Jon](https://thesquareplanet.com/) 與
[Jose](http://josejg.com/) 共同授課。我們都曾是 MIT 的學生，也是在學生時期
創立了這門 MIT IAP 課程。你可以透過
[missing-semester@mit.edu](mailto:missing-semester@mit.edu) 聯絡我們。

我們教這門課沒有領取報酬，也沒有以任何方式營利。我們將所有[課程教材](https://missing.csail.mit.edu/)
與[講座錄影](https://www.youtube.com/@MissingSemester)免費公開在線上。
如果你想支持我們，最好的方式就是幫忙分享這門課。如果你是公司、大學或其他機構，
把這些內容用於較大規模的學員，也歡迎來信分享經驗回饋，讓我們知道實際成效 :)

# 課程動機

身為資訊科學領域的學習者，我們都知道電腦很擅長處理重複工作。
但我們常常忘了：這件事不只適用於「程式要做的運算」，也同樣適用於
「我們自己使用電腦的方式」。其實我們手邊有大量工具，能幫助我們更有效率、
處理更複雜的電腦相關問題。但很多人只用了其中很小一部分；平常只靠死背幾句
「咒語式」指令勉強過關，卡住時就盲目從網路複製貼上。

這門課就是想[改善這件事](/about/)。

我們希望教你把已知工具用到最好、帶你認識更多可加入工具箱的新工具，
也希望培養你主動探索（甚至自己打造）工具的熱情。
我們相信，這正是多數資工課程中缺少的那個學期。

# 課程結構

這門非學分課由九堂一小時講座組成，每堂聚焦一個[特定主題](/2026/)。
各講座大多可獨立學習，但隨課程推進，我們會預設你已熟悉前面講座內容。
我們有提供線上講義，但課堂中某些內容（例如現場 demo）可能不會完整出現在講義裡。
和往年一樣，我們會錄製講座並上傳到
[線上平台](https://www.youtube.com/@MissingSemester)。

由於要在少數幾堂一小時講座涵蓋很多內容，密度會比較高。
為了讓你能依自己的步調熟悉內容，每堂課都附有一組練習，
幫助你掌握講座重點。我們不會安排固定 office hours，
但很歡迎你在 [OSSU Discord](https://ossu.dev/#community) 的 `#missing-semester-forum`
提問，或來信至 [missing-semester@mit.edu](mailto:missing-semester@mit.edu)。

受限於時間，我們無法像完整學期課程那樣，對所有工具做同等深度的講解。
我們會盡量提供延伸資源，幫助你深入某個工具或主題；如果你對某個方向特別有興趣，
也非常歡迎直接聯絡我們請教！

最後，如果你對課程有任何回饋，請寄信到
[missing-semester@mit.edu](mailto:missing-semester@mit.edu)。

# 主題 1：Shell

{% comment %}
lecturer: Jon
{% endcomment %}

## 什麼是 shell？

現代電腦有很多下指令方式：華麗的圖形介面、語音介面、AR/VR，
到最近的 LLM。這些方式可以滿足大約 80% 的需求，但本質上通常有限制——
你不能按不存在的按鈕，也不能下尚未被設計好的語音指令。
若要完整發揮電腦提供的能力，我們得回到更底層的文字介面：Shell。

幾乎所有平台都有某種形式的 shell，而且不少平台同時提供多種 shell 可選。
雖然細節不同，但核心概念都很像：讓你執行程式、提供輸入，並以半結構化方式檢視輸出。

要打開 shell 的 _prompt_（可輸入指令的位置），你先需要 _terminal_
（終端機），也就是 shell 的視覺介面。你的裝置通常已預裝，或可輕鬆自行安裝：

- **Linux:**
  按下 `Ctrl + Alt + T`（多數發行版都適用），或在應用程式選單搜尋
  「Terminal」。
- **Windows:**
  按下 `Win + R`，輸入 `cmd` 或 `powershell` 後按 Enter。
  或在開始功能表搜尋「Terminal」或「Command Prompt」。
- **macOS:**
  按下 `Cmd + Space` 開啟 Spotlight，輸入「Terminal」後按 Enter。
  或到 Applications → Utilities → Terminal 開啟。

在 Linux 與 macOS 上，通常會開到 Bourne Again SHell，也就是常說的
「bash」。它是最常見的 shell 之一，語法也和許多其他 shell 相近。
在 Windows 上，則會看到 `batch` 或 `powershell`（取決於你執行的指令）。
這兩者屬於 Windows 生態，不是本課重點，雖然我們教的大多概念都有對應做法。
如果你用 Windows，建議改用
[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/)
或 Linux 虛擬機。

還有其他 shell，通常在易用性上比 bash 有不少改進（常見像 fish、zsh）。
雖然它們很受歡迎（所有講師也都有在用），但普及度仍不如 bash，且核心概念相通，
所以本講先不聚焦它們。

## 為什麼你該在意？

Shell 不只是（通常）比「點來點去」更快，它還提供單一圖形工具很難給你的表達力。
你會看到，shell 可以把不同程式有創意地 _組合_ 起來，自動化幾乎任何工作。

熟悉 shell 也非常有助於你在開源軟體世界中行走（許多安裝步驟都需要 shell）、
替專案建置持續整合（如[程式碼品質講座](/2026/code-quality/)所述），
以及在其他程式出錯時快速除錯。

## 在 shell 中移動

當你開啟終端機，通常會看到類似這樣的 _prompt_：

```console
missing:~$
```

這是 shell 的主要文字介面。它告訴你目前機器名稱是 `missing`，
而你所在的「目前工作目錄」是 `~`（home 的縮寫）。
`$` 代表你現在不是 root 使用者（後面會再說）。
在這個提示符號後，你可以輸入 _command_，由 shell 解譯執行。
最基本的指令就是執行一個程式：

```console
missing:~$ date
Fri 10 Jan 2020 11:49:31 AM EST
missing:~$
```

這裡我們執行了 `date` 程式，它會（不意外地）印出目前日期與時間。
接著 shell 會繼續等待你輸入下一個指令。你也可以帶 _arguments_ 來執行指令：

```console
missing:~$ echo hello
hello
```

這個例子中，我們要求 shell 執行 `echo`，並傳入參數 `hello`。
`echo` 的功能就是把參數印出來。
shell 會先用空白分割整行指令，再把第一個字當成程式名稱，
其餘每個字都作為該程式可讀取的參數。
若你的參數內含空白或特殊字元（例如資料夾名稱 `"My Photos"`），
可以用 `'` 或 `"` 把整個參數包起來（`"My Photos"`），
或只用 `\` 跳脫必要字元（`My\ Photos`）。

初學時最重要的指令之一是 `man`（manual 的縮寫）。
`man` 可讓你查系統中任一指令的更多資訊。
例如執行 `man date`，會看到 `date` 的用途與可用參數。
另外，多數指令也支援加 `--help` 取得較短版說明。

> 建議除了 `man` 之外，也安裝並使用 [`tldr`](https://tldr.sh/)，
> 它會直接在終端機顯示常見用法範例。LLM 也很擅長解釋指令運作方式，
> 以及怎麼組合參數達成你的目標。

繼 `man` 之後，另一個必學指令是 `cd`（change directory）。
它其實是 shell 的內建指令，不是獨立程式（也就是 `which cd`
通常會顯示找不到外部執行檔）。你傳入一個路徑後，該路徑就會成為
你的目前工作目錄。你也會在提示符中看到目錄變化：

```console
missing:~$ cd /bin
missing:/bin$ cd /
missing:/$ cd ~
missing:~$
```

> 要注意，shell 有自動補完功能，按 `<TAB>` 常可更快補齊路徑！

很多指令在你沒特別指定時，會以「目前工作目錄」作為操作基準。
若你不確定自己在哪裡，可用 `pwd`，或印出 `$PWD`（`echo $PWD`）；
兩者都會顯示目前工作目錄。

目前工作目錄也讓我們能使用 _相對路徑（relative path）_。
到目前為止看到的大多是 _絕對路徑（absolute path）_：
它們以 `/` 開頭，表示從檔案系統根目錄（`/`）一路走到目標位置的完整路徑。
實務上你會更常用相對路徑，因為它是相對於目前工作目錄來解讀。
相對路徑（也就是不以 `/` 開頭）會先在目前目錄找第一層，再照一般規則往下走。例如：

```console
missing:~$ cd /
missing:/$ cd bin
missing:/bin$
```

每個目錄中還有兩個「特殊路徑元件」：`.` 和 `..`。
`.` 代表「目前目錄」，`..` 代表「上一層目錄」。例如：

```console
missing:~$ cd /
missing:/$ cd bin/../bin/../bin/././../bin/..
missing:/$
```

在大多數指令參數中，絕對路徑和相對路徑通常都能互換使用；
只要在用相對路徑時，清楚自己當下工作目錄是哪裡就好！

> 也可以考慮安裝 [`zoxide`](https://github.com/ajeetdsouza/zoxide) 來加速 `cd` 操作；
> `z` 會記住你常去的路徑，讓你用更少輸入快速切換。

## Shell 裡有哪些可用工具？

那 shell 到底怎麼知道要去哪裡找 `date` 或 `echo` 這些程式？
當 shell 被要求執行某個指令時，它會查詢名為 `$PATH` 的
_環境變數（environment variable）_，裡面列出 shell 該去哪些目錄搜尋程式：

```console
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

當我們執行 `echo` 時，shell 會知道要執行名為 `echo` 的程式，
並到 `$PATH` 內以 `:` 分隔的各目錄裡尋找同名檔案。找到後就執行
（前提是該檔案具備可執行權限，後面會提到）。
我們可以用 `which` 查某個程式名稱實際對應到哪個檔案。
你也可以直接給程式檔案的 _路徑（path）_，完全略過 `$PATH`。

這也提示了我們如何找出 shell 可執行的 _所有_ 程式：
把 `$PATH` 裡每個目錄內容列出來。可以用 `ls` 搭配目錄路徑做到：

```console
missing:~$ ls /bin
```

> 可以考慮安裝 [`eza`](https://eza.rocks/) 作為更友善的 `ls` 替代方案。

在多數電腦上，這會印出 _大量_ 程式；這裡只先看幾個最重要的。先從簡單的開始：

- `cat file`：印出 `file` 內容。
- `sort file`：把 `file` 的各行排序後輸出。
- `uniq file`：移除 `file` 中連續重複的行。
- `head file` 與 `tail file`：分別印出 `file` 開頭與結尾幾行。

> 可以考慮安裝 [`bat`](https://github.com/sharkdp/bat) 來取代 `cat`，
> 支援語法上色與更好的瀏覽體驗。

還有 `grep pattern file`，用來找出 `file` 中符合 `pattern` 的行。
它值得多花一點注意力，因為它 _非常_ 實用，功能也比想像中豐富。
其中 `pattern` 其實是 _正規表示式（regular expression）_，
可描述很複雜的匹配規則——我們會在[程式碼品質講座](/2026/code-quality/#regular-expressions)
詳細介紹。你也可以把目標改成目錄（或省略成 `.`），再加 `-r`
遞迴搜尋目錄內所有檔案。

> 可以考慮使用 [`ripgrep`](https://github.com/BurntSushi/ripgrep) 取代 `grep`，
> 它更快、使用體驗也更友善（但可攜性略低）。
> 此外，`ripgrep` 預設就會遞迴搜尋目前工作目錄！

還有一些很有用但介面稍微複雜的工具。第一個是 `sed`，
它是可程式化的檔案編輯器。`sed` 有自己的語言可自動化修改檔案，
最常見用法是：

```console
missing:~$ sed -i 's/pattern/replacement/g' file
```

這會把 `file` 中所有 `pattern` 替換成 `replacement`。
`-i` 表示直接改原檔（而不是只輸出替換後內容、原檔不變）。
`s/` 在 sed 語言裡代表「做替換」。
`/` 用來分隔 pattern 與 replacement。
最後的 `/g` 代表每一行中所有匹配都要替換，而不是只替換第一個。
和 `grep` 一樣，這裡的 `pattern` 也是正規表示式，能提供很強的表達力。
正規表示式替換也允許 `replacement` 參照匹配到的部分；等一下會看到例子。

接著是 `find`，它能依條件（遞迴）尋找檔案。例如：

```console
missing:~$ find ~/Downloads -type f -name "*.zip" -mtime +30
```

這會在下載目錄中找出超過 30 天的 ZIP 檔。

```console
missing:~$ find ~ -type f -size +100M -exec ls -lh {} \;
```

這會在家目錄中找出大於 100M 的檔案，並列出它們。
注意 `-exec` 需要一個以獨立 `;` 結束的 _command_
（像空白一樣需要跳脫），而 `{}` 會由 `find`
替換成每個匹配到的檔案路徑。

```console
missing:~$ find . -name "*.py" -exec grep -l "TODO" {} \;
```

這會找出包含 TODO 的 `.py` 檔案。

`find` 語法一開始可能有點硬，但希望你已感受到它有多實用！

> 可以考慮安裝 [`fd`](https://github.com/sharkdp/fd) 取代 `find`，
> 使用體驗更友善（但可攜性也較低）。

接下來是 `awk`，它和 `sed` 一樣有自己的程式語言。
`sed` 偏向「編輯檔案」，`awk` 偏向「解析檔案」。
`awk` 最常見用途，是處理格式規律的資料檔（例如 CSV），
從每筆紀錄（每一行）擷取你要的欄位：

```console
missing:~$ awk '{print $2}' file
```

這會印出 `file` 每一行以空白分隔的第二欄。
如果加上 `-F,`，就會改成印出每行以逗號分隔的第二欄。
`awk` 能做的遠不只這些——像過濾資料列、聚合統計等；
你可以在練習題中先嘗鮮。

把這些工具組合起來，我們可以做到像下面這種進階操作：

```console
missing:~$ ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \
  | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \
  | sort | uniq -c \
  | sort -nk1,1 | tail -n10 \
  | awk '{print $2}' | paste -sd,
postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root
```

這段會抓遠端伺服器的 SSH 紀錄（下一講會再深入講 `ssh`），
搜尋斷線訊息，從每筆訊息擷取使用者名稱，再輸出前 10 名常見使用者（逗號分隔）。
全部只用一行指令完成！每一步細節留給你當練習。

## Shell 語言（bash）

前一個範例帶出了新概念：pipe（`|`）。
它能把一個程式的輸出，直接接到下一個程式的輸入。
之所以成立，是因為多數命令列程式在未指定 `file` 參數時，
會從「標準輸入」（也就是你平常打字進去的地方）讀資料。
`|` 會把左側程式的「標準輸出」（平常印在終端機上的內容）
當成右側程式的標準輸入。
這讓你能把 shell 程式 _組裝（compose）_ 起來，也是 shell 高生產力的重要原因。

其實，多數 shell 都實作了完整程式語言（例如 bash），
和 Python、Ruby 一樣有變數、條件判斷、迴圈、函式。
你在 shell 裡下指令，本質上就是在寫一小段由 shell 解譯的程式碼。
我們今天不會把 bash 全部教完，但以下幾個部分特別實用：

先看重新導向（redirect）：
`>file` 可把程式標準輸出寫入 `file`，而不是顯示在終端機，
之後更容易分析。`>>file` 代表附加到檔尾，不覆蓋原內容。
`<file` 則是把 `file` 當成程式標準輸入，而不是從鍵盤讀取。

> 這裡也很適合提一下 `tee`。`tee` 會把標準輸入印到標準輸出
>（像 `cat` 一樣），同時 _也會_ 寫入檔案。
> 所以 `verbose cmd | tee verbose.log | grep CRITICAL`
> 可以一邊把完整詳細紀錄存檔，一邊維持終端機輸出精簡。

再來是條件判斷：
`if command1; then command2; command3; fi` 會先執行 `command1`，
若沒有錯誤，就接著執行 `command2` 與 `command3`。
你也可以加上 `else` 分支。
最常拿來當 `command1` 的是 `test`（常簡寫為 `[`），
可判斷像「檔案是否存在」（`test -f file` / `[ -f file ]`）
或「字串是否相等」（`[ "$var" = "string" ]`）。
在 bash 裡還有 `[[ ]]`，它是較「安全」的內建判斷語法，
在引號等情境下怪異行為較少。

bash 也有兩種迴圈：`while` 與 `for`。
`while command1; do command2; command3; done` 很像 `if`，
差別在於只要 `command1` 持續成功，它就會反覆執行。
`for varname in a b c d; do command; done` 會執行 `command` 四次，
每次 `$varname` 依序為 `a`、`b`、`c`、`d`。
實務上你常不會手動列出項目，而是用「指令替換（command substitution）」：

```bash
for i in $(seq 1 10); do
```

這會先執行 `seq 1 10`（輸出 1 到 10），再用該輸出取代整段 `$()`，
變成跑 10 次的 for 迴圈。
你在舊程式碼有時會看到反引號寫法（像 ``for i in `seq 1 10`; do``），
但建議優先使用 `$()`，因為它可巢狀使用。

雖然你 _可以_ 直接在 prompt 寫長 shell script，
但通常還是建議寫進 `.sh` 檔。
例如下面腳本會重複執行測試直到失敗，只顯示失敗那次輸出，
同時在背景施加 CPU 壓力（例如用來重現不穩定測試）：

```bash
#!/bin/bash
set -euo pipefail

# 在背景啟動 CPU 壓力測試
stress --cpu 8 &
STRESS_PID=$!

# 設定記錄檔
LOGFILE="test_runs_$(date +%s).log"
echo "Logging to $LOGFILE"

# 重複執行測試直到失敗
RUN=1
while cargo test my_test > "$LOGFILE" 2>&1; do
    echo "Run $RUN passed"
    ((RUN++))
done

# 清理並輸出結果
kill $STRESS_PID
echo "Test failed on run $RUN"
echo "Last 20 lines of output:"
tail -n 20 "$LOGFILE"
echo "Full log: $LOGFILE"
```

這段腳本包含不少新概念，建議花時間深入，因為它們在寫實用 shell 指令時非常常用，
例如背景工作（`&`）可讓程式並行執行、較進階的
[shell 重新導向](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)，
以及[算術展開](https://www.gnu.org/software/bash/manual/html_node/Arithmetic-Expansion.html)。

另外很值得特別看前兩行。
第一行是 shebang（`#!`），不只 shell script，很多腳本檔案頂端都會有。
當一個檔案以 `#!/path` 開頭並被執行時，shell 會啟動 `/path` 指定的程式，
並把該檔案內容當作輸入傳給它。以 shell script 來說，就是把腳本內容交給 `/bin/bash`；
同理，Python 也可用 `/usr/bin/python` 作為 shebang。

第二行是讓 bash 進入較「嚴格」模式，可降低寫 shell script 時的踩雷機率。
`set` 可接很多參數，簡單說：
`-e` 讓任一指令失敗時腳本立即結束；
`-u` 讓使用未定義變數時直接報錯，而不是默默當空字串；
`-o pipefail` 讓 `|` 管線中任一步失敗時，整個腳本也提早失敗結束。

> Shell 程式設計和其他語言一樣是很深的主題，但要提醒你：bash 的陷阱特別多，
> 多到有[很多網站](https://tldp.org/LDP/abs/html/gotchas.html)
> 專門在[整理踩雷案例](https://mywiki.wooledge.org/BashPitfalls)。
> 強烈建議你在寫腳本時大量使用
> [shellcheck](https://www.shellcheck.net/)。
> LLM 也很適合拿來撰寫與除錯 shell script；
> 當腳本變得太龐大（100+ 行）時，也可協助轉寫成「正式」程式語言（例如 Python）。

# 下一步

到這裡，你已經具備足夠的 shell 基礎，能完成基本任務。
你應該能在系統中移動、找到目標檔案，並使用多數程式的基本功能。
下一講我們會談如何用 shell 與更多實用命令列工具，
來完成並自動化更複雜的工作。

# 練習

本課每一講都搭配一系列練習。
有些題目是明確任務，有些則較開放，例如「試著用 X 與 Y 程式」。
我們非常鼓勵你實際動手做做看。

我們沒有提供練習題標準答案。
如果你卡住了，歡迎在 [Discord](https://ossu.dev/#community) 的
`#missing-semester-forum` 發問，或寄信告訴我們你目前嘗試了什麼，
我們會盡力協助你。這些練習也很適合當作和 LLM 對話的起始提示，
透過互動方式逐步深入。這些題目的真正價值，在於你探索答案的過程，
而不只是答案本身。建議你在解題時多追問「為什麼」並延伸探索，
而不是只找最短路徑拿到結果。

1. 這門課需要使用 Unix shell（例如 Bash 或 ZSH）。
   如果你用 Linux 或 macOS，不需特別設定。
   如果你用 Windows，請確認你不是在用 cmd.exe 或 PowerShell；
   你可以使用 [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/)
   或 Linux 虛擬機來操作 Unix 風格命令列工具。
   要確認你目前 shell 是否符合需求，可執行 `echo $SHELL`。
   若顯示像 `/bin/bash` 或 `/usr/bin/zsh`，表示你用的是正確環境。

1. `ls` 的 `-l` 旗標是做什麼？
   執行 `ls -l /` 並觀察輸出。
   每行開頭 10 個字元分別代表什麼？（提示：`man ls`）

1. 在指令 `find ~/Downloads -type f -name "*.zip" -mtime +30` 中，
   `*.zip` 是一種「glob」。
   什麼是 glob？請建立一個測試資料夾，放幾個檔案後實驗
   `ls *.txt`、`ls file?.txt`、`ls {a,b,c}.txt`。
   可參考 Bash 手冊中的
   [Pattern Matching](https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html)。

1. `'single quotes'`、`"double quotes"`、`$'ANSI quotes'` 有什麼差別？
   請寫一個指令，輸出一段字串，內容需包含字面值 `$`、`!`，以及換行字元。
   可參考 [Quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)。

1. Shell 有三種標準串流：stdin（0）、stdout（1）、stderr（2）。
   執行 `ls /nonexistent /tmp`，把 stdout 重新導向到一個檔案，
   stderr 重新導向到另一個檔案。
   那要如何把兩者都導向同一個檔案？可參考
   [Redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)。

1. `$?` 會保存上一個指令的結束狀態（0 = 成功）。
   `&&` 只在前一個成功時才執行下一個；
   `||` 只在前一個失敗時才執行下一個。
   請寫一行指令：僅在 `/tmp/mydir` 不存在時才建立它。
   可參考 [Exit Status](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)。

1. 為什麼 `cd` 必須是 shell 內建指令，而不能只是獨立程式？
   （提示：想想子行程能與不能影響父行程的哪些狀態。）

1. 寫一個腳本，接收檔名作為參數（`$1`），並使用 `test -f` 或 `[ -f ... ]`
   檢查檔案是否存在。它應該依「存在／不存在」輸出不同訊息。
   可參考
   [Bash Conditional Expressions](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html)。

1. 把上一題腳本存成檔案（例如 `check.sh`）。
   嘗試用 `./check.sh somefile` 執行，會發生什麼？
   再執行 `chmod +x check.sh` 後重試。為什麼這步驟是必要的？
   （提示：比較 `chmod` 前後 `ls -l check.sh` 的結果。）

1. 如果在腳本中的 `set` 旗標再加上 `-x`，會發生什麼？
    請用簡單腳本測試並觀察輸出。
    可參考 [The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)。

1. 寫一個指令，將檔案複製成含有今天日期的備份檔名
    （例如 `notes.txt` → `notes_2026-01-12.txt`）。
    （提示：`$(date +%Y-%m-%d)`）
    可參考 [Command Substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)。

1. 修改講座中的 flaky test 腳本，讓它改為接收測試指令作為參數，
    不要把 `cargo test my_test` 寫死。
    （提示：`$1` 或 `$@`）
    可參考 [Special Parameters](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html)。

1. 使用 pipe 找出你家目錄中最常見的前 5 種副檔名。
    （提示：組合 `find`、`grep` 或 `sed` 或 `awk`、`sort`、`uniq -c`、`head`）

1. `xargs` 會把 stdin 的每一行轉成指令參數。
    請用 `find` 搭配 `xargs`（不要用 `find -exec`），找出某目錄下所有 `.sh` 檔，
    並用 `wc -l` 計算每個檔案行數。
    加分：讓它可正確處理檔名含空白。
    （提示：`-print0` 與 `-0`）
    可參考 `man xargs`。

1. 使用 `curl` 抓取課程網站 HTML
    （`https://missing.csail.mit.edu/`），再 pipe 給 `grep`
    計算列出了幾堂講座。
    （提示：找一個每堂講座只出現一次的樣式；用 `curl -s` 關閉進度輸出）

1. [`jq`](https://jqlang.github.io/jq/) 是處理 JSON 的強大工具。
    用 `curl` 抓取範例資料
    `https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json`，
    再用 `jq` 擷取 version 大於 6 的人名。
    （提示：先 pipe 到 `jq .` 看結構，再試 `jq '.[] | select(...) | .name'`）

1. `awk` 可依欄位條件過濾資料並調整輸出。
    例如 `awk '$3 ~ /pattern/ {$4=""; print}'` 只會印出第三欄匹配 `pattern`
    的行，並省略第四欄。
    請寫一個 `awk` 指令：只印出第二欄大於 100 的行，並交換第一與第三欄。
    可用以下資料測試：`printf 'a 50 x\nb 150 y\nc 200 z\n'`

1. 拆解講座中的 SSH log pipeline：每一步在做什麼？
    然後做一個類似流程，從 `~/.bash_history`（或 `~/.zsh_history`）
    找出你最常用的 shell 指令。
