---
layout: lecture
title: "命令列環境"
description: >
  學習命令列程式如何運作，包含輸入／輸出串流、環境變數，以及透過 SSH 連線遠端主機。
thumbnail: /static/assets/thumbnails/2026/lec2.png
date: 2026-01-13
ready: true
video:
  aspect: 56.25
  id: ccBGsPedE9Q
---

如同上一講提到的，多數 shell 並不只是拿來啟動其他程式的「發射器」，
實務上它提供的是一整套程式語言，裡面包含大量常見模式與抽象概念。
不過和多數程式語言不同的是，shell script 幾乎所有設計都圍繞在「執行程式」與「讓程式彼此高效溝通」這件事上。

特別是，shell script 非常依賴 _慣例（conventions）_。若要讓一個命令列介面（CLI）程式能和整個 shell 生態良好協作，就需要遵守一些常見模式。
接下來我們會介紹理解命令列程式運作所需的核心概念，以及各種常見的使用與設定慣例。

# 命令列介面（CLI）

在大多數程式語言中，撰寫函式通常會長這樣：

```
def add(x: int, y: int) -> int:
    return x + y
```

在這裡，我們可以很明確地看到程式的輸入與輸出。
相較之下，shell script 乍看之下會長得很不一樣。

```shell
#!/usr/bin/env bash

if [[ -f $1 ]]; then
    echo "Target file already exists"
    exit 1
else
    if $DEBUG; then
        grep 'error' - | tee $1
    else
        grep 'error' - > $1
    fi
    exit 0
fi
```

要正確理解這類腳本在做什麼，我們得先認識幾個概念。當 shell 程式彼此溝通，或和 shell 環境互動時，這些概念會一直出現：

- 參數（Arguments）
- 串流（Streams）
- 環境變數（Environment variables）
- 回傳碼（Return codes）
- 訊號（Signals）

## 參數（Arguments）

Shell 程式在執行時，會收到一串參數清單。
在 shell 裡，參數本質上都是字串，至於如何解讀，完全由程式自己決定。
例如執行 `ls -l folder/` 時，本質上就是呼叫 `/bin/ls`，參數為 `['-l', 'folder/']`。

在 shell script 內，我們可透過特殊語法存取這些參數。
第一個參數是 `$1`、第二個是 `$2`，一路到 `$9`。要取得所有參數清單可用 `$@`，參數總數用 `$#`。另外也可用 `$0` 取得程式名稱。

對大多數程式來說，參數通常會混合 _旗標（flags）_ 與一般字串。
旗標的辨識方式通常是前面有單破折號（`-`）或雙破折號（`--`）。
旗標多半是可選的，功能是調整程式行為。
例如 `ls -l` 就是在改變 `ls` 輸出的格式。

你會看到像 `--all` 這種雙破折號長名稱旗標，也會看到像 `-a` 這種單破折號短旗標（通常是一個字母）。
同一個選項常常兩種寫法都可用，`ls -a` 與 `ls --all` 等價。
單破折號旗標常可合併，所以 `ls -l -a` 與 `ls -la` 也等價。
旗標順序通常也不重要，`ls -la` 和 `ls -al` 結果相同。
有些旗標非常常見，熟悉 shell 之後你會自然常用它們，例如 `--help`、`--verbose`、`--version`。

> 旗標是 shell 慣例的一個好例子。shell 語言本身並沒有硬性規定程式一定要用 `-` 或 `--`。
> 理論上你也可以設計成 `myprogram +myoption myfile`，但這會造成混淆，因為大家普遍預期是用破折號。
> 實務上，多數程式語言都有 CLI 旗標剖析函式庫（例如 Python 的 `argparse`）來處理這種破折號語法。

CLI 程式另一個常見慣例是：接受「同型別、數量可變」的參數。也就是一次給多個參數時，指令會對每個參數做相同操作。

```shell
mkdir src
mkdir docs
# is equivalent to
mkdir src docs
```

這種語法糖一開始看起來好像可有可無，但搭配 _globbing_（萬用字元展開）時就非常強大。
Globbing（或叫 globs）是 shell 在呼叫程式前，會先展開的特殊樣式。

假設我們想刪除目前資料夾中（不含子資料夾）所有 `.py` 檔，依照上一講學到的方法可以這樣做：

```shell
for file in $(ls | grep -P '\.py$'); do
    rm "$file"
done
```

但其實可以直接寫成 `rm *.py`！

當你在終端機輸入 `rm *.py` 時，shell 並不會把 `['*.py']` 直接傳給 `/bin/rm`。
它會先在目前資料夾搜尋符合 `*.py` 的檔案，其中 `*` 代表可匹配長度為 0 或以上的任意字串。
所以如果資料夾裡有 `main.py` 與 `utils.py`，那麼 `rm` 實際收到的參數會是 `['main.py', 'utils.py']`。

最常見的 glob 有：`*`（任意長度）、`?`（剛好一個字元）、以及大括號展開。
大括號 `{}` 會把逗號分隔的樣式清單展開成多個參數。

實務上，透過例子最容易理解 globs。

```shell
touch folder/{a,b,c}.py
# 會展開成
touch folder/a.py folder/b.py folder/c.py

convert image.{png,jpg}
# 會展開成
convert image.png image.jpg

cp /path/to/project/{setup,build,deploy}.sh /newpath
# 會展開成
cp /path/to/project/setup.sh /path/to/project/build.sh /path/to/project/deploy.sh /newpath

# Globbing 技巧也可以組合使用
mv *{.py,.sh} folder
# 會移動所有 *.py 與 *.sh 檔案
```

> 有些 shell（例如 zsh）支援更進階的 globbing，像是 `**` 可展開為遞迴路徑。也就是說 `rm **/*.py` 會遞迴刪除所有 `.py` 檔案。


## 串流（Streams）

每當我們執行像這樣的 pipeline：

```shell
cat myfile | grep -P '\d+' | uniq -c
```

可以看到 `grep` 同時在和 `cat` 以及 `uniq` 溝通。

這裡有個重要觀察：三個程式是「同時」執行的。
shell 並不是先跑 `cat`，再跑 `grep`，最後才跑 `uniq`。
相反地，三者會一起被啟動，shell 只負責把 `cat` 的輸出接到 `grep` 的輸入，再把 `grep` 的輸出接到 `uniq` 的輸入。
使用 pipe 運算子 `|` 時，shell 是在處理一段段從一個程式流到下一個程式的資料串流。

我們可以示範這種並行行為：pipeline 裡的指令會立即全部啟動。

```console
$ (sleep 15 && cat numbers.txt) | grep -P '^\d$' | sort | uniq  &
[1] 12345
$ ps | grep -P '(sleep|cat|grep|sort|uniq)'
  32930 pts/1    00:00:00 sleep
  32931 pts/1    00:00:00 grep
  32932 pts/1    00:00:00 sort
  32933 pts/1    00:00:00 uniq
  32948 pts/1    00:00:00 grep
```

可以看到除了 `cat` 以外，其他行程都立刻在跑。shell 會先把所有行程建立起來並把串流接好，而不是等某個先跑完。`cat` 會等 `sleep` 結束才開始，接著 `cat` 輸出會送到 `grep`，後續依此類推。

每個程式都有輸入串流，稱為 stdin（standard input）。使用 pipe 時，stdin 會自動接好。在腳本中，很多程式接受 `-` 當檔名，表示「從 stdin 讀取」：

```shell
# 當資料來自 pipe 時，這兩種寫法等價
echo "hello" | grep "hello"
echo "hello" | grep "hello" -
```

同樣地，每個程式有兩種輸出串流：stdout 與 stderr。
標準輸出（stdout）是最常見的輸出，也就是會透過 pipe 傳給下一個指令的那條串流。
標準錯誤（stderr）則是另一條輸出，通常拿來回報警告或錯誤，避免被 pipeline 下一個指令當成一般資料去解析。

```console
$ ls /nonexistent
ls: cannot access '/nonexistent': No such file or directory
$ ls /nonexistent | grep "pattern"
ls: cannot access '/nonexistent': No such file or directory
# 錯誤訊息仍會出現，因為 stderr 不會被 pipe
$ ls /nonexistent 2>/dev/null
# 沒有輸出，因為 stderr 被重新導向到 /dev/null
```

shell 提供了重新導向這些串流的語法。下面是一些常見例子。

```shell
# 將 stdout 重新導向到檔案（覆蓋）
echo "hello" > output.txt

# 將 stdout 重新導向到檔案（附加）
echo "world" >> output.txt

# 將 stderr 重新導向到檔案
ls foobar 2> errors.txt

# 將 stdout 與 stderr 都重新導向到同一個檔案
ls foobar &> all_output.txt

# 從檔案重新導向 stdin
grep "pattern" < input.txt

# 重新導向到 /dev/null 來丟棄輸出
cmd > /dev/null 2>&1
```

另一個很能體現 Unix 哲學的工具是 [`fzf`](https://github.com/junegunn/fzf)（模糊搜尋器）。它會從 stdin 讀入多行資料，並提供互動式介面讓你篩選與選取：

```console
$ ls | fzf
$ cat ~/.bash_history | fzf
```

`fzf` 能和很多 shell 操作整合。講到 shell 客製化時，我們會再看到更多應用。


## 環境變數（Environment variables）

在 bash 裡，指定變數用 `foo=bar`，存取變數值用 `$foo`。
要注意 `foo = bar` 是錯誤語法，因為 shell 會把它解讀成：呼叫程式 `foo`，參數為 `['=', 'bar']`。
在 shell script 裡，空白字元很重要，因為它會觸發參數切割（argument splitting）。
這個行為一開始常讓人混淆，請特別留意。

shell 變數沒有型別，全部都是字串。
另外，shell 的單引號與雙引號不可互換。
被 `'` 包住的是字面值字串，不會展開變數、不會做指令替換，也不會處理跳脫字元；被 `"` 包住則會。

```shell
foo=bar
echo "$foo"
# 會印出 bar
echo '$foo'
# 會印出 $foo
```

如果要把指令輸出放進變數，要用 _command substitution_（指令替換）。
當我們執行：
```shell
files=$(ls)
echo "$files" | grep README
echo "$files" | grep ".py"
```
`ls` 的輸出（精確地說是 stdout）會被放進 `$files` 變數，後續就可再取用。
`$files` 內容會保留 `ls` 的換行，這也是像 `grep` 這類程式能逐項處理資料的原因。

一個較少被提到、但很像的功能是 _process substitution_（行程替換）。`<( CMD )` 會執行 `CMD`，把輸出放進暫存檔，然後用那個暫存檔路徑取代 `<()`。
當某個指令要求你「傳入檔案」而不是從 STDIN 讀取時，這招很有用。
例如 `diff <(ls src) <(ls docs)` 就能比較 `src` 與 `docs` 兩個資料夾的內容差異。

當 shell 程式呼叫另一個程式時，會一起傳遞一組變數，這組變數通常稱為 _環境變數（environment variables）_。
在 shell 中可用 `printenv` 查看目前的環境變數。
若要明確傳入某個環境變數，可在指令前先加變數指定：

> 環境變數慣例上會用全大寫（例如 `HOME`、`PATH`、`DEBUG`）。這是慣例，不是技術限制；但遵守它能更容易區分環境變數與通常用小寫的本地 shell 變數。

```shell
TZ=Asia/Tokyo date  # 會印出東京目前時間
echo $TZ  # 這裡會是空的，因為 TZ 只對該子行程生效
```

另一種做法是使用 `export` 內建指令，它會修改目前 shell 環境，因此後續所有子行程都會繼承該變數：

```shell
export DEBUG=1
# 從這裡開始執行的所有程式都會帶有 DEBUG=1
bash -c 'echo $DEBUG'
# 會印出 1
```

要刪除變數可用 `unset` 內建指令，例如 `unset DEBUG`。

> 環境變數也是 shell 的重要慣例之一。它讓你可以「隱式」調整很多程式行為，而不必每次都寫成明確參數。舉例來說，shell 會把目前使用者家目錄路徑設在 `$HOME`，程式可直接讀取這個值，不必額外要求 `--home /home/alice`。另一個常見例子是 `$TZ`，很多程式會根據它指定的時區格式化日期與時間。

## 回傳碼（Return codes）

如前面所見，shell 程式主要透過 stdout/stderr 串流與檔案系統副作用來輸出結果。

預設情況下，shell script 的結束碼（exit code）是 0。
慣例上，0 代表成功；非 0 代表過程中遇到問題。
要回傳非 0，必須用 `exit NUM` 這個 shell 內建指令。
最近一次執行指令的回傳碼可透過特殊變數 `$?` 取得。

shell 有布林運算子 `&&` 與 `||`，分別代表 AND 與 OR。
和一般程式語言不同，shell 這兩個運算子是看「指令回傳碼」來判斷。
它們都屬於[短路求值（short-circuiting）](https://en.wikipedia.org/wiki/Short-circuit_evaluation)運算子。
因此可依照前一個指令成功或失敗，條件式地執行下一個指令；其中成功的定義是回傳碼是否為 0。例子如下：

```shell
# 只有 grep 成功（找到匹配）時才會執行 echo
grep -q "pattern" file.txt && echo "Pattern found"

# 只有 grep 失敗（找不到匹配）時才會執行 echo
grep -q "pattern" file.txt || echo "Pattern not found"

# true 是永遠成功的 shell 程式
true && echo "This will always print"

# false 是永遠失敗的 shell 程式
false || echo "This will always print"
```

同樣原理也適用於 `if` 與 `while`：它們都是根據回傳碼做判斷。

```shell
# if 會看條件指令的回傳碼（0 = true，非 0 = false）
if grep -q "pattern" file.txt; then
    echo "Found"
fi

# while 會在指令持續回傳 0 時繼續迴圈
while read line; do
    echo "$line"
done < file.txt
```

## 訊號（Signals）

有時候你會需要在程式執行中斷它，例如某個指令跑太久。
最簡單的方式是按 `Ctrl-C`，通常該指令就會停止。
但這背後到底怎麼運作？又為什麼有時候它不會停？

```console
$ sleep 100
^C
$
```

> 注意，這裡的 `^C` 是終端機顯示 `Ctrl` 的方式。

底層實際發生的事如下：

1. 你按下 `Ctrl-C`
2. shell 辨識到這個特殊按鍵組合
3. shell 行程送出 `SIGINT` 訊號給 `sleep` 行程
4. 這個訊號中斷了 `sleep` 的執行

訊號是一種特殊的行程溝通機制。
當行程收到訊號時，會先中斷目前執行，處理該訊號，並可能依訊號內容改變後續流程。因此，訊號可視為一種 _軟體中斷（software interrupts）_。


以這個例子來說，按下 `Ctrl-C` 會讓 shell 對行程送出 `SIGINT`。
下面是一個最小化 Python 範例：它會攔截 `SIGINT` 並忽略它，因此 `Ctrl-C` 不再能停止程式。這時可以改用 `SIGQUIT`（按 `Ctrl-\`）來結束。

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

下面示範先對此程式送兩次 `SIGINT`，再送 `SIGQUIT` 會發生什麼。注意 `^` 是終端機顯示 `Ctrl` 的方式。

```console
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

雖然 `SIGINT` 與 `SIGQUIT` 常見於終端機互動情境，但若要更通用地要求行程「優雅結束」，通常會用 `SIGTERM`。
你可以用 [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) 送出這個訊號，語法是 `kill -TERM <PID>`。

訊號不只用來終止行程。像 `SIGSTOP` 就能暫停行程。在終端機按 `Ctrl-Z` 時，shell 會送出 `SIGTSTP`（Terminal Stop），可視為終端機版本的 `SIGSTOP`。

接著可用 [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) 或 [`bg`](https://man7.org/linux/man-pages/man1/bg.1p.html) 把暫停工作恢復到前景或背景執行。

[`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) 指令會列出目前終端機 session 尚未結束的工作。
你可以用 pid 來指它們（可用 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 查到）。
更直覺的方式是用 `%` 加工作編號（由 `jobs` 顯示）。若要指向最近一次背景化的工作，可用特殊參數 `$!`。

另一個要點是：在指令後面加 `&` 會讓它進背景執行，提示字元會立刻回來；但它仍可能使用 shell 的 STDOUT，畫面可能很干擾（這時可搭配重新導向）。等價地，若程式已在前景執行，可用 `Ctrl-Z` 再接 `bg` 讓它轉到背景。


要注意，背景行程仍是目前終端機的子行程；若你關閉終端機，它們通常也會一起結束（會收到 `SIGHUP` 訊號）。
要避免這件事，可用 [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) 執行程式（它會忽略 `SIGHUP`），或在程式已啟動後用 `disown`。
另一種做法是使用終端機多工器，下一節會介紹。

下面是一段示範 session，展示上述概念。

```
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ kill -SIGHUP %2   # nohup 會保護行程不受 SIGHUP 影響

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000
```

`SIGKILL` 是一個特殊訊號：行程無法攔截它，收到後一定會立即終止。不過它可能造成副作用，例如留下孤兒子行程。

想進一步了解各種訊號，可參考[這裡](https://en.wikipedia.org/wiki/Signal_(IPC))，或在終端機輸入 [`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html) 與 `kill -l`。

在 shell script 裡，你可以用 `trap` 內建指令在收到訊號時執行特定命令。這在清理資源時很有用：

```shell
#!/usr/bin/env bash
cleanup() {
    echo "正在清理暫存檔..."
    rm -f /tmp/mytemp.*
}
trap cleanup EXIT  # 腳本結束時執行清理
trap cleanup SIGINT SIGTERM  # Ctrl-C 或 kill 時也執行
```
{% comment %}
### Users, Files and Permissions

Lastly, another way programs have to indirectly communicate with each other is using files.
For a program to be able to correctly read/write/delete files and folders, the file permissions must allow the operation.

Listing a specific file will give the following output

```console
$ ls -l notes.txt
-rw-r--r--  1 alice  users  12693 Jan 11 23:05 notes.txt
```

Here `ls` is listing what is the owner of the file, user `alice`, and the group `users`. Then the `rw-r--r--` are a shorthand notation for the permissions.
In this case, the file `notes.txt` has read/write permissions for the user alice `rw-`, and only read permissions for the group and the rest of users in the file system.

```console
$ ./script.sh
# permission denied
$ chmod +x script.sh
$ ls -l script.sh
-rwxr-xr-x  1 alice  users  3125 Jan 11 23:07 script.sh
$ ./script.sh
```

For a script to be executable, the executable rights must be set, hence why we had to use the `chmod` (change mode) program.
`chmod` syntax, while intuitive, is not obvious when first encountered.
If you, like me, prefer to learn by example, this is a good usecase of the `tldr` tool (note that you need to install it first).

```console
❯ tldr chmod
  Change the access permissions of a file or directory.
  More information: <https://www.gnu.org/software/coreutils/chmod>.

  Give the [u]ser who owns a file the right to e[x]ecute it:

      chmod u+x path/to/file

  Give the [u]ser rights to [r]ead and [w]rite to a file/directory:

      chmod u+rw path/to/file_or_directory

  Give [a]ll users rights to [r]ead and e[x]ecute:

      chmod a+rx path/to/file
```

Run `tldr chmod` to see more examples, including recursive operations and group permissions.

> Your shell might show you something like `command not found: tldr`. That is because it is a more modern tool and it is not pre-installed in most systems. A good reference for how to install tools is the [https://command-not-found.com](https://command-not-found.com) website. It contains instructions for a huge collection of CLI tools for popular OS distributions.

Each program is run as a specific user in the system. We can use the `whoami` command to find our user name and `id -u` to find our UID (user id) which is the integer value that the OS associates with the user.

When running `sudo command`, the `command` is run as the root user which can bypass most permissions in the system.
Try running `sudo whoami` and `sudo id -u` to see how the output changes (you might be prompted for your password).
To change the owner of a file or folder, we use the `chown` command.

You can learn more about UNIX file permissions [here](https://en.wikipedia.org/wiki/File-system_permissions#Traditional_Unix_permissions)

So far we've focused on your local machine, but many of these skills become even more valuable when working with remote servers.

{% endcomment %}

# 遠端主機

現在程式設計師在日常工作中使用遠端伺服器已經越來越常見。最常用的工具是 SSH（Secure Shell），它能幫我們連到遠端伺服器，並提供熟悉的 shell 介面。連線指令通常像這樣：

```bash
ssh alice@server.mit.edu
```

這裡代表我們要用使用者 `alice` 的身分連到 `server.mit.edu`。

`ssh` 有個常被忽略的功能：可用非互動方式直接執行指令。`ssh` 會正確處理指令的 stdin 與 stdout，所以可以很自然地和其他指令串接：

```shell
# 這裡 ls 在遠端執行，wc 在本機執行
ssh alice@server ls | wc -l

# 這裡 ls 與 wc 都在遠端伺服器執行
ssh alice@server 'ls | wc -l'

```

> 可以試試 [Mosh](https://mosh.org/) 作為 SSH 替代方案。它能更好處理斷線、電腦睡眠喚醒、網路切換，以及高延遲連線。

要讓 `ssh` 允許我們在遠端執行指令，必須先證明我們有權限。
這可以透過密碼或 SSH 金鑰完成。
金鑰驗證使用公開金鑰密碼學，讓客戶端在不洩漏私鑰的情況下，證明自己持有該私鑰。
金鑰驗證通常更方便也更安全，建議優先使用。
請注意，私鑰（常見像 `~/.ssh/id_rsa`，近年更常見 `~/.ssh/id_ed25519`）本質上就是你的密碼，務必妥善保管，絕對不要外流內容。

要產生一組金鑰，可執行 [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html)。
```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

如果你曾設定過用 SSH 金鑰推送到 GitHub，你很可能已做過[這裡](https://help.github.com/articles/connecting-to-github-with-ssh/)提到的步驟，也已經有可用的金鑰組。要檢查金鑰是否有 passphrase 並驗證它，可執行 `ssh-keygen -y -f /path/to/key`。

在伺服器端，`ssh` 會查看 `.ssh/authorized_keys` 來判斷允許哪些客戶端登入。要把公鑰複製到遠端，可用：

```bash
cat .ssh/id_ed25519.pub | ssh alice@remote 'cat >> ~/.ssh/authorized_keys'

# 或更簡單（若系統有 `ssh-copy-id`）

ssh-copy-id -i .ssh/id_ed25519 alice@remote
```

除了執行指令之外，SSH 建立的連線也可用來安全地在本機與伺服器之間傳檔。最傳統的工具是 [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html)，語法為 `scp path/to/local_file remote_host:path/to/remote_file`。[`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) 則進一步改善 `scp`：它會偵測本機與遠端已相同的檔案，避免重複複製。它也提供更細緻的符號連結與權限控制，還有像 `--partial` 這種可從中斷處續傳的功能。`rsync` 語法與 `scp` 類似。

SSH 用戶端設定檔在 `~/.ssh/config`，可用來定義主機別名並為它們設定預設參數。這個檔案不只 `ssh` 會讀，`scp`、`rsync`、`mosh` 等工具也會使用。

```bash
Host vm
    User alice
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# 設定也可使用萬用字元
Host *.mit.edu
    User alice
```




# 終端機多工器（Terminal Multiplexers）

使用命令列時，你常會想同時跑不只一件事。
例如一邊開編輯器、一邊跑程式。
雖然可以靠開多個終端機視窗達成，但使用終端機多工器會更彈性。

像 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) 這類終端機多工器，能透過分割窗格與分頁來同時管理多個 shell session，讓操作更有效率。
此外，多工器可讓你把目前 session 分離（detach），之後再接回來（reattach）。
因此在遠端主機工作時特別方便，常能省去 `nohup` 這類技巧。

目前最流行的終端機多工器是 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html)。`tmux` 可高度客製化，透過快捷鍵可以快速建立多個分頁與窗格並在其間移動。

`tmux` 需要你熟悉它的快捷鍵，格式通常是 `<C-b> x`，意思是：(1) 按 `Ctrl+b`、(2) 放開 `Ctrl+b`、(3) 再按 `x`。`tmux` 物件層級如下：
- **Sessions** - 一個 session 是獨立工作空間，內含一個或多個視窗
    + `tmux`：啟動新 session
    + `tmux new -s NAME`：以指定名稱啟動
    + `tmux ls`：列出目前 sessions
    + 在 `tmux` 內按 `<C-b> d`：分離目前 session
    + `tmux a`：接回最近一次 session，可用 `-t` 指定目標

- **Windows** - 類似編輯器或瀏覽器分頁，是同一個 session 中視覺上分開的區塊
    + `<C-b> c`：建立新視窗。要關閉可在該 shell 按 `<C-d>` 結束
    + `<C-b> N`：移動到第 _N_ 個視窗（視窗有編號）
    + `<C-b> p`：移到上一個視窗
    + `<C-b> n`：移到下一個視窗
    + `<C-b> ,`：重新命名目前視窗
    + `<C-b> w`：列出目前視窗

- **Panes** - 類似 vim 的 split，同一個畫面可同時放多個 shell
    + `<C-b> "`：水平分割目前窗格
    + `<C-b> %`：垂直分割目前窗格
    + `<C-b> <direction>`：移動到指定方向窗格（方向鍵）
    + `<C-b> z`：切換目前窗格縮放
    + `<C-b> [`：進入捲動回看模式。可按 `<space>` 開始選取、`<enter>` 複製選取內容
    + `<C-b> <space>`：輪換窗格排列方式

> 想進一步學 tmux，可先讀[這篇](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)快速教學，再看[這篇](https://linuxcommand.org/lc3_adv_termmux.php)較完整說明。

有了 tmux 與 SSH 之後，你會希望在任何機器上都能快速打造熟悉環境。這就是 shell 客製化登場的地方。

# 客製化 Shell

很多命令列程式都透過純文字設定檔來設定，這些檔案通常稱為 _dotfiles_
（因為檔名以 `.` 開頭，例如 `~/.vimrc`，所以預設在 `ls` 清單中是隱藏的）。

> Dotfiles 也是 shell 慣例之一。前面的點是為了在列表時把它們「隱藏」起來（對，又是一個慣例）。

Shell 就是透過這類檔案設定的典型例子。啟動時，shell 會讀取多個檔案來載入設定。
視你使用哪個 shell，以及是否啟動 login / interactive session，整個流程可能相當複雜。
關於這點，[這篇文章](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)很值得參考。

對 `bash` 來說，多數系統編輯 `.bashrc` 或 `.bash_profile` 就能生效。
其他可透過 dotfiles 設定的工具還有：

- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` and the `~/.vim` folder
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

一個常見設定是把新路徑加入 shell 搜尋程式的位置。你安裝軟體時很常看到這種寫法：

```shell
export PATH="$PATH:path/to/append"
```

這行是在告訴 shell：把 `$PATH` 設成「目前 PATH + 新路徑」，並讓所有子行程繼承更新後的 PATH。
如此一來，子行程就能找到位於 `path/to/append` 之下的程式。


客製化 shell 往往代表要安裝新的命令列工具。套件管理器能讓這件事變簡單，會幫你處理下載、安裝與更新。不同作業系統有不同套件管理器：macOS 用 [Homebrew](https://brew.sh/)、Ubuntu/Debian 用 `apt`、Fedora 用 `dnf`、Arch 用 `pacman`。我們會在 shipping code 講座更深入介紹套件管理器。

以下示範在 macOS 用 Homebrew 安裝兩個實用工具：

```shell
# ripgrep：更快、預設更好的 grep
brew install ripgrep

# fd：更快、對使用者更友善的 find
brew install fd
```

安裝後，你就可以用 `rg` 取代 `grep`、用 `fd` 取代 `find`。

> **關於 `curl | bash` 的警告**：你常會看到像 `curl -fsSL https://example.com/install.sh | bash` 的安裝指令。這種做法會下載腳本後立刻執行，雖然方便但有風險，因為你在執行尚未檢查的程式碼。較安全做法是先下載、檢查，再執行：
> ```shell
> curl -fsSL https://example.com/install.sh -o install.sh
> less install.sh  # 先檢查腳本內容
> bash install.sh
> ```
> 有些安裝器會用稍微安全一點的變體：`/bin/bash -c "$(curl -fsSL https://url)"`，至少能確保由 bash 解析腳本，而不是你當下使用的 shell。

當你執行尚未安裝的指令時，shell 會顯示 `command not found`。網站 [command-not-found.com](https://command-not-found.com) 很實用，可查詢任一指令在不同套件管理器與發行版上的安裝方式。

另一個實用工具是 [`tldr`](https://tldr.sh/)，它提供精簡、以範例為主的 man page。你不用啃長篇文件，也能快速掌握常見用法：

```console
$ tldr fd
  An alternative to find.
  Aims to be faster and easier to use than find.

  Recursively find files matching a pattern in the current directory:
      fd "pattern"

  Find files that begin with "foo":
      fd "^foo"

  Find files with a specific extension:
      fd --extension txt
```

有時你不需要安裝新程式，只想幫既有指令做固定旗標捷徑，這就是 alias 的用途。

我們可以用 shell 內建的 `alias` 來建立自己的指令別名。
shell alias 是另一個指令的縮寫，shell 會在解析前自動替換成原指令。
以 bash 為例，格式如下：

```bash
alias alias_name="command_to_alias arg1 arg2"
```

> 注意 `=` 前後不能有空白，因為 [`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) 是只接受單一參數的 shell 指令。

alias 有很多方便的用途：

```bash
# 幫常用旗標做縮寫
alias ll="ls -lh"

# 常用指令可大幅減少輸入字數
alias gs="git status"
alias gc="git commit"

# 避免手誤
alias sl=ls

# 覆寫既有指令，改成更好的預設行為
alias mv="mv -i"           # -i 覆蓋前會提示
alias mkdir="mkdir -p"     # -p 需要時自動建立父資料夾
alias df="df -h"           # -h 以人類可讀格式顯示

# Alias 可以組合
alias la="ls -A"
alias lla="la -l"

# 若要忽略 alias，可在前面加上 \
\ls
# 或用 unalias 直接停用該 alias
unalias la

# 想查看 alias 定義，直接用 alias 查詢
alias ll
# 會印出 ll='ls -lh'
```

alias 也有限制：它無法在指令中間靈活接收參數。若需要更複雜行為，應改用 shell function。

多數 shell 支援 `Ctrl-R` 反向搜尋歷史指令。按下 `Ctrl-R` 後開始輸入即可搜尋過去指令。前面提過 `fzf` 是模糊搜尋器；若設定好 fzf 的 shell 整合，`Ctrl-R` 會變成互動式模糊搜尋整份歷史，比預設功能強很多。

Dotfiles 應該如何管理？建議把它們放在獨立資料夾、納入版本控制，並用腳本建立 **symlink** 到正確位置。這樣有幾個好處：

- **安裝快速**：登入新機器時，幾分鐘內就能套用全部個人化設定。
- **可攜性**：你的工具在任何環境都維持一致行為。
- **可同步**：在任何地方更新 dotfiles，都能保持一致。
- **可追蹤變更**：dotfiles 很可能會跟著你整個職涯，長期專案有版本歷史非常有價值。

Dotfiles 裡該放什麼？
你可以透過線上文件或 [man page](https://en.wikipedia.org/wiki/Man_page) 了解工具設定。另一種好方法是搜尋特定工具的部落格文章，作者通常會分享偏好的客製化方式。你也可以參考別人的 dotfiles；GitHub 上有大量 [dotfiles repositories](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)，最熱門之一在[這裡](https://github.com/mathiasbynens/dotfiles)（但不建議盲目照抄設定）。
[這裡](https://dotfiles.github.io/)也是不錯的資源。

本課講師們的 dotfiles 也都公開在 GitHub： [Anish](https://github.com/anishathalye/dotfiles)、
[Jon](https://github.com/jonhoo/configs)、
[Jose](https://github.com/jjgo/dotfiles)。

**框架與外掛** 也能提升 shell 體驗。常見大型框架像 [prezto](https://github.com/sorin-ionescu/prezto) 或 [oh-my-zsh](https://ohmyz.sh/)，也有聚焦特定功能的小型外掛：

- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) - 輸入時即時標示指令是否有效
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) - 依歷史指令提供輸入建議
- [zsh-completions](https://github.com/zsh-users/zsh-completions) - 額外補完定義
- [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search) - 類 fish 的歷史搜尋
- [powerlevel10k](https://github.com/romkatv/powerlevel10k) - 快速且可高度客製的提示字元主題

像 [fish](https://fishshell.com/) 這類 shell，預設就包含很多這些功能。

> 你不一定需要像 oh-my-zsh 這種大型框架才能擁有這些功能。單獨安裝外掛通常更快，也更可控。大型框架可能明顯拖慢 shell 啟動速度，建議只安裝你真的會用到的項目。


# Shell 裡的 AI

在 shell 中整合 AI 工具的方法很多。以下是幾種不同整合層級的例子：

**指令產生**：像 [`simonw/llm`](https://github.com/simonw/llm) 這類工具可根據自然語言描述產生 shell 指令：

```console
$ llm cmd "find all python files modified in the last week"
find . -name "*.py" -mtime -7
```

**Pipeline 整合**：LLM 可整合進 shell pipeline 來做資料處理與轉換。特別是當資料格式不一致、用 regex 會很痛苦時，它很有幫助：

```console
$ cat users.txt
Contact: john.doe@example.com
User 'alice_smith' logged in at 3pm
Posted by: @bob_jones on Twitter
Author: Jane Doe (jdoe)
Message from mike_wilson yesterday
Submitted by user: sarah.connor
$ INSTRUCTIONS="Extract just the username from each line, one per line, nothing else"
$ llm "$INSTRUCTIONS" < users.txt
john.doe
alice_smith
bob_jones
jdoe
mike_wilson
sarah.connor
```

注意這裡使用 `"$INSTRUCTIONS"`（有加引號），因為變數內有空白；同時用 `< users.txt` 把檔案內容重新導向到 stdin。

**AI shell**：像 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 這類工具可作為「meta-shell」，接收英文需求並轉換成 shell 操作、檔案修改，甚至更複雜的多步驟任務。

# 終端機模擬器（Terminal Emulators）

除了客製化 shell，也很值得花點時間挑選並設定你使用的 **終端機模擬器**。
終端機模擬器是提供文字介面的 GUI 程式，shell 就是在裡面執行。
市面上有很多不同選擇。

你可能會花上數百到數千小時在終端機裡工作，所以投資時間調整設定很划算。常見可調項目包括：

- 字型選擇
- 色彩主題
- 鍵盤快捷鍵
- 分頁／窗格支援
- 回捲（scrollback）設定
- 效能（部分新終端機如 [Alacritty](https://github.com/alacritty/alacritty) 或 [Ghostty](https://ghostty.org/) 支援 GPU 加速）



# 練習

## 參數與 Globs

1. 你可能看過像 `cmd --flag -- --notaflag` 這種指令。`--` 是特殊參數，代表程式從這裡開始停止解析旗標，後面的內容都當作位置參數。這為什麼有用？試著執行 `touch -- -myfile`，再嘗試不用 `--` 把它刪掉看看。

1. 閱讀 [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html)，寫出一個 `ls` 指令，讓輸出符合以下條件：
    - 包含所有檔案（含隱藏檔）
    - 大小用人類可讀格式顯示（例如 454M 而非 454279954）
    - 依照最近更新排序
    - 輸出有顏色

    範例輸出如下：

    ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

{% comment %}
ls -lath --color=auto
{% endcomment %}

1. 行程替換 `<(command)` 可讓你把指令輸出當成檔案使用。請用 process substitution 搭配 `diff` 比較 `printenv` 與 `export` 的輸出。為什麼它們不同？（提示：試試 `diff <(printenv | sort) <(export | sort)`）

## 環境變數

1. 寫兩個 bash function：`marco` 與 `polo`。需求如下：每次執行 `marco` 時，要把目前工作目錄存起來；之後不管你人在什麼目錄，執行 `polo` 都要把你 `cd` 回執行 `marco` 時的位置。為了方便除錯，你可以把程式寫在 `marco.sh`，再用 `source marco.sh`（重新）載入到 shell。

{% comment %}
marco() {
    export MARCO=$(pwd)
}

polo() {
    cd "$MARCO"
}
{% endcomment %}

## 回傳碼

1. 假設你有個很少失敗的指令。為了除錯，你需要收集它的輸出，但要等到失敗案例可能要跑很久。請寫一個 bash script，重複執行下面程式直到失敗，並把它的標準輸出與錯誤輸出存成檔案，最後一次印出完整結果。加分：同時回報總共跑了幾次才失敗。

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ n -eq 42 ]]; then
       echo "Something went wrong"
       >&2 echo "The error was using magic numbers"
       exit 1
    fi

    echo "Everything went according to plan"
    ```

{% comment %}
#!/usr/bin/env bash

count=0
until [[ "$?" -ne 0 ]];
do
  count=$((count+1))
  ./random.sh &> out.txt
done

echo "found error after $count runs"
cat out.txt
{% endcomment %}

## 訊號與工作控制（Job Control）

1. 在終端機啟動 `sleep 10000`，用 `Ctrl-Z` 把它丟到背景，再用 `bg` 繼續執行。接著用 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 找出 pid，並使用 [`pkill`](https://man7.org/linux/man-pages/man1/pgrep.1.html) 在「不手動輸入 pid」的情況下把它結束。（提示：使用 `-af` 旗標）

1. 假設你不想在某行程結束前啟動另一個行程，該怎麼做？在這題中，限制行程固定是 `sleep 60 &`。其中一種做法是用 [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html)。請試著先啟動 sleep，讓 `ls` 等到背景行程結束後才執行。

    不過如果你換到另一個 bash session，這招會失效，因為 `wait` 只能等待子行程。講義中尚未提到的是：`kill` 指令成功時 exit status 為 0，失敗則為非 0。`kill -0` 不會送訊號，但若行程不存在會回傳非 0。請寫一個名為 `pidwait` 的 bash function，接收 pid 並等待該行程結束。請搭配 `sleep` 避免不必要的 CPU 浪費。

## 檔案與權限

1. （進階）寫一個指令或腳本，遞迴找出某資料夾中最近修改的檔案。更一般地說，你能否列出所有檔案並按修改時間排序？

## 終端機多工器

1. 跟著這份 `tmux` [教學](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)操作，接著依照[這些步驟](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)學一些基本客製化。

## Aliases 與 Dotfiles

1. 建立一個 `dc` alias，對應到 `cd`，用來防止打錯字。

1. 執行 `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10`，找出你最常用的前 10 個指令，並考慮替它們設定更短的 alias。注意：這在 Bash 可用；若你用 ZSH，請把 `history` 改成 `history 1`。

1. 建立一個 dotfiles 資料夾並設定版本控制。

1. 至少為一個程式加入設定（例如 shell），做一些客製化（起手式可先從設定 `$PS1` 自訂提示字元開始）。

1. 設計一種能在新機器上快速（且不需手動）安裝 dotfiles 的方法。可以是簡單的 shell script，逐一執行 `ln -s`，或使用[專門工具](https://dotfiles.github.io/utilities/)。

1. 在一台全新虛擬機上測試你的安裝腳本。

1. 把你目前所有工具設定都遷移到 dotfiles repository。

1. 將你的 dotfiles 發佈到 GitHub。

## 遠端主機（SSH）

請先安裝一台 Linux 虛擬機（或使用既有虛擬機）來做以下練習。若你不熟悉虛擬機，可參考[這篇](https://hibbard.eu/install-ubuntu-virtual-box/)安裝教學。

1. 到 `~/.ssh/` 檢查是否已有 SSH 金鑰組。若沒有，請用 `ssh-keygen -a 100 -t ed25519` 產生。建議設定密碼並使用 `ssh-agent`，更多資訊看[這裡](https://www.ssh.com/ssh/agent)。

1. 編輯 `.ssh/config`，加入以下設定：

    ```bash
    Host vm
        User username_goes_here
        HostName ip_goes_here
        IdentityFile ~/.ssh/id_ed25519
        LocalForward 9999 localhost:8888
    ```

1. 使用 `ssh-copy-id vm` 把你的 SSH 公鑰複製到伺服器。

1. 在 VM 內執行 `python -m http.server 8888` 啟動網頁伺服器。接著在你的本機打開 `http://localhost:9999` 存取 VM 內的網站。

1. 用 `sudo vim /etc/ssh/sshd_config` 編輯 SSH 伺服器設定，透過調整 `PasswordAuthentication` 停用密碼登入；再調整 `PermitRootLogin` 停用 root 登入。接著用 `sudo service sshd restart` 重啟 SSH 服務，並重新測試登入。

1. （挑戰）在 VM 安裝 [`mosh`](https://mosh.org/) 並建立連線，然後中斷伺服器／VM 的網路介面。mosh 能否正確恢復連線？

1. （挑戰）查一下 `ssh` 的 `-N` 與 `-f` 旗標用途，並找出可達成背景埠轉發（background port forwarding）的指令。
