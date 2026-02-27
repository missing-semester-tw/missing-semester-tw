---
layout: lecture
title: "除錯與效能分析"
description: >
  學習如何用 logging 與除錯器來除錯程式，以及如何進行程式效能分析。
thumbnail: /static/assets/thumbnails/2026/lec4.png
date: 2026-01-15
ready: true
panopto: "https://mit.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=a72c48e3-5eb2-46fa-aa03-b3b700e1ca8d"
video:
  aspect: 56.25
  id: 8VYT9TcUmKs
---

程式設計有一條黃金法則：程式碼不會照你「以為」它會做的事去執行，而是照你「實際告訴它」的方式執行。要補上這個落差，有時候會非常困難。這堂課會介紹處理有 bug、又很吃資源程式碼的實用技巧：除錯與效能分析。

# 除錯

## Printf 除錯與 Logging

> 「最有效的除錯工具，依然是謹慎思考，再搭配放在適當位置的 print 陳述式。」— Brian Kernighan，_Unix for Beginners_

第一種除錯方法，是在你發現問題的附近加上 print 陳述式，然後反覆嘗試，直到你收集到足夠資訊，能理解問題的真正原因。

第二種方法，是在程式裡使用 logging，而不是臨時亂加 print。Logging 本質上是「更有系統的 print」，通常會透過 logging framework 來做，內建支援像是：

- 可以把 log（或其中一部分）導到不同輸出位置；
- 設定嚴重程度等級（例如 INFO、DEBUG、WARN、ERROR 等），並依等級過濾輸出；以及
- 支援結構化 logging，記錄和 log 項目相關的資料，事後更容易擷取與分析。

你通常也會在開發時主動先加一些 logging 陳述式，這樣未來要除錯時，資料可能早就已經在那裡了！
而且，當你用 print 陳述式找到並修好問題後，通常值得在移除前先把那些 print 轉成正式的 log 陳述式。這樣如果未來又出現類似 bug，就不需要再修改程式碼，也已經有需要的診斷資訊。

> **第三方 log**：很多程式支援 `-v` 或 `--verbose` 參數，執行時會印出更多資訊。這對找出某個指令為什麼失敗很有幫助。有些工具甚至可以重複加這個參數來顯示更詳細的內容。除錯服務（資料庫、網頁伺服器等）問題時，也要檢查它們的 log；在 Linux 上通常會在 `/var/log/`。systemd 服務可用 `journalctl -u <service>` 查看 log。若是第三方函式庫，請確認是否能透過環境變數或設定啟用 debug logging。

## 除錯器

當你知道該印什麼、也能輕鬆修改並重跑程式時，print 除錯很有效。但如果你不確定需要哪些資訊、bug 只在很難重現的條件下出現，或是修改與重啟程式成本很高（啟動很久、狀態很難重建等），除錯器就特別有價值。

除錯器是一種工具，讓你在程式執行當下和它互動，能做到：

- 執行到某一行時暫停；
- 一次一步地往下執行；
- 在程式崩潰後查看變數值；
- 在特定條件成立時才暫停；
- 以及更多進階功能。

多數程式語言都支援（或內建）某種除錯器。最通用的是 **通用型除錯器**，像 [`gdb`](https://www.gnu.org/software/gdb/)（GNU Debugger）與 [`lldb`](https://lldb.llvm.org/)（LLVM Debugger），可除錯任何原生二進位程式。許多語言也有 **語言專用除錯器**，和執行階段整合得更緊密（例如 Python 的 pdb、Java 的 jdb）。

`gdb` 是 C、C++、Rust 等編譯語言事實上的標準除錯器。它可以檢查幾乎任何行程，並取得目前機器狀態：暫存器、堆疊、程式計數器等等。

一些實用的 GDB 指令：

- `run` - 啟動程式
- `b {function}` 或 `b {file}:{line}` - 設定中斷點
- `c` - 繼續執行
- `step` / `next` / `finish` - 進入函式 / 略過函式 / 跳出函式
- `p {variable}` - 印出變數值
- `bt` - 顯示 backtrace（呼叫堆疊）
- `watch {expression}` - 值改變時中斷

> 可考慮使用 GDB 的 TUI 模式（`gdb -tui`，或在 GDB 內按 `Ctrl-x a`），以分割畫面同時顯示原始碼與指令列。

### 記錄-重播除錯

有些最讓人挫折的 bug 是 [_`海森堡bug`_](https://zh.wikipedia.org/zh-tw/%E6%B5%B7%E6%A3%AE%E5%A0%A1bug)：你一觀察，它就消失或改變行為。競態條件、依賴時序的 bug、只在特定系統條件下出現的問題，都屬於這類。傳統除錯在這種情況常常幫不上忙，因為重跑一次程式就會有不同結果（例如加上 print 後程式變慢，慢到競態條件不再發生）。

**記錄-重播除錯** 的解法是先記錄程式執行，再用可重現（deterministic）的方式重播，想重播幾次都可以。更棒的是，你還能反向執行，精準找到哪一刻開始出錯。

[rr](https://rr-project.org/) 是 Linux 上很強大的工具，能記錄程式執行並進行可重現重播，且保有完整除錯能力。它和 GDB 整合，所以你已經熟悉操作介面。

基本用法：

```bash
# 記錄一次程式執行
rr record ./my_program

# 重播記錄（會開啟 GDB）
rr replay
```

真正厲害的是在重播時。因為執行是可重現的，你可以使用 **反向除錯** 指令：

- `reverse-continue` (`rc`) - 反向執行直到撞到中斷點
- `reverse-step` (`rs`) - 反向單步一行
- `reverse-next` (`rn`) - 反向執行，略過函式呼叫
- `reverse-finish` - 反向執行直到進入目前函式

這對除錯非常強大。假設你遇到程式崩潰，不用先猜 bug 在哪裡再設中斷點，你可以：

1. 先跑到崩潰點
2. 檢查已損壞的狀態
3. 在被破壞的變數上設 watchpoint
4. 用 `reverse-continue` 精準找出它在哪一行被改壞

**什麼時候適合用 rr：**
- 偶發失敗、不穩定的測試
- 競態條件與多執行緒 bug
- 很難重現的崩潰
- 任何讓你想「回到過去」的 bug

> 注意：rr 只支援 Linux，且需要硬體效能計數器。若 VM 沒有暴露這些計數器（例如多數 AWS EC2 instance），就無法使用；它也不支援 GPU 存取。macOS 可參考 [Warpspeed](https://warpspeed.dev/)。

> **rr 與並行性**：因為 rr 會以可重現方式記錄執行，它會把執行緒排程序列化。這代表某些依賴特定時序的競態條件，在 rr 下可能不會出現。rr 仍然很適合除錯競態：一旦你錄到失敗執行，就能可靠重播；但要抓到偶發 bug，可能需要多錄幾次。若 bug 與並行無關，rr 的優勢就更明顯：你總是能重現完全相同的執行，並用反向除錯追出破壞來源。

## 系統呼叫追蹤

有時你需要了解程式如何和作業系統互動。程式會透過 [system call](https://en.wikipedia.org/wiki/System_call) 向核心要求服務——開檔案、配置記憶體、建立行程等等。追蹤這些呼叫，能看出程式為什麼卡住、它正嘗試存取哪些檔案，或是時間到底耗在哪裡等待。

### strace（Linux）與 dtruss（macOS）

[`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html) 可以讓你觀察程式發出的每一個系統呼叫：

```bash
# 追蹤所有系統呼叫
strace ./my_program

# 只追蹤檔案相關呼叫
strace -e trace=file ./my_program

# 跟著子行程一起追蹤（對會啟動其他程式的程式很重要）
strace -f ./my_program

# 追蹤已在執行中的行程
strace -p <PID>

# 顯示時間資訊
strace -T ./my_program
```

> 在 macOS 與 BSD 上，可使用 [`dtruss`](https://www.manpagez.com/man/1/dtruss/)（封裝 `dtrace`）達到類似功能：

> 若想更深入了解 `strace`，可參考 Julia Evans 很棒的 [strace zine](https://jvns.ca/strace-zine-unfolded.pdf)。

### bpftrace 與 eBPF

[eBPF](https://ebpf.io/)（extended Berkeley Packet Filter）是 Linux 上強大的技術，可在核心中執行沙箱化程式。[`bpftrace`](https://github.com/iovisor/bpftrace) 提供高階語法來撰寫 eBPF 程式。這些程式能在核心任意執行，因此表達能力很強（雖然語法有點像 awk、相對彆扭）。最常見用途是調查哪些系統呼叫被觸發，並做彙總（例如次數或延遲統計），或觀察（甚至過濾）系統呼叫參數。

```bash
# 全系統追蹤檔案開啟（立即輸出）
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'

# 依名稱統計系統呼叫次數（Ctrl-C 後輸出摘要）
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* { @[probe] = count(); }'
```

不過，你也可以使用 [`bcc`](https://github.com/iovisor/bcc) 這類工具鏈，直接用 C 撰寫 eBPF 程式。它也附帶了[很多實用工具](https://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html)，例如 `biosnoop` 可印出磁碟操作延遲分佈，`opensnoop` 可印出所有被開啟的檔案。

`strace` 的優點是很容易「馬上上手」，而 `bpftrace` 則適合你需要更低負擔、想追到核心函式、或需要做彙總分析等情況。不過要注意 `bpftrace` 通常得用 `root` 執行，而且它一般監控的是整個核心，不是單一行程。若要鎖定特定程式，可以依指令名稱或 PID 過濾：

```bash
# 依指令名稱過濾（Ctrl-C 後輸出摘要）
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /comm == "bash"/ { @[probe] = count(); }'

# 用 -c 從啟動開始追蹤特定指令（cpid = 子行程 PID）
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /pid == cpid/ { @[probe] = count(); }' -c 'ls -la'
```

`-c` 旗標會執行指定指令，並把 `cpid` 設成該行程 PID，適合從程式一啟動就開始追蹤。當被追蹤指令結束時，bpftrace 會輸出彙總結果。

### 網路除錯

遇到網路問題時，可用 [`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) 與 [Wireshark](https://www.wireshark.org/) 擷取並分析封包：

```bash
# 擷取 80 port 的封包
sudo tcpdump -i any port 80

# 擷取並存檔，供 Wireshark 分析
sudo tcpdump -i any -w capture.pcap
```

對 HTTPS 流量來說，因為有加密，tcpdump 能看到的資訊較有限。像 [mitmproxy](https://mitmproxy.org/) 這類工具可作為攔截代理來檢視加密流量。對 Web 應用而言，瀏覽器開發者工具（Network 分頁）通常是除錯 HTTPS 請求最簡單的方法，因為會顯示解密後的 request/response 資料、標頭與時間資訊。

## 記憶體除錯

記憶體 bug——像是緩衝區溢位（buffer overflow）、釋放後使用（use-after-free）、記憶體洩漏（memory leak）——是最危險、也最難除錯的一類。它們常常不會立刻崩潰，而是先把記憶體悄悄弄壞，之後才引發問題。

### Sanitizer 工具

找記憶體 bug 的一種做法是使用 **sanitizer**。這是編譯器提供的功能，會在程式中插入額外檢查，於執行時偵測錯誤。以常見的 **AddressSanitizer（ASan）** 為例，可偵測：
- 緩衝區溢位（stack、heap、global）
- 釋放後使用（use-after-free）
- 返回後使用（use-after-return）
- 記憶體洩漏

```bash
# 用 AddressSanitizer 編譯
gcc -fsanitize=address -g program.c -o program
./program
```

還有不少實用的 sanitizer：

- **ThreadSanitizer (TSan)**：偵測多執行緒程式中的資料競爭（`-fsanitize=thread`）
- **MemorySanitizer (MSan)**：偵測讀取未初始化記憶體（`-fsanitize=memory`）
- **UndefinedBehaviorSanitizer (UBSan)**：偵測未定義行為，例如整數溢位（`-fsanitize=undefined`）

Sanitizer 需要重新編譯，但速度通常夠快，可放進 CI pipeline 與日常開發流程。

### Valgrind：無法重編譯時

[Valgrind](https://valgrind.org/) 會在類似虛擬機的環境中執行你的程式，藉此偵測記憶體錯誤。它比 sanitizer 慢，但不需要重新編譯：

```bash
valgrind --leak-check=full ./my_program
```

適合使用 Valgrind 的情境：
- 你沒有原始碼
- 你無法重編譯（例如第三方函式庫）
- 你需要 sanitizer 沒提供的特定工具

Valgrind 其實是很強的受控執行環境，後面講效能分析時還會再看到它！

## 用 AI 協助除錯

大型語言模型（LLM）已經成為意外好用的除錯助手。它們在某些除錯任務上特別出色，能補足傳統工具。

**LLM 擅長的地方：**

- **解釋難懂的錯誤訊息**：編譯器錯誤，特別是 C++ template 或 Rust borrow checker，常常很難讀。LLM 可以把它翻成白話並提供修正方向。

- **跨語言與抽象層追查**：如果你在除錯橫跨多語言的問題（例如 C 函式庫的 bug 透過 Python binding 顯現），LLM 可以幫你梳理不同層次。它們特別擅長理解 FFI 邊界、建置系統問題與跨語言除錯（例如：我的程式報錯，但我懷疑是某個依賴套件的 bug）。

- **把症狀對應到根因**：「程式功能正常，但記憶體用量是預期的 10 倍」這種模糊症狀，LLM 能幫忙推理可能原因與下一步檢查方向。

- **分析 crash dump 與 stack trace**：貼上 stack trace，請它推測可能成因。

> **關於 debug symbol 的注意事項**：若要有意義的 stack trace 與除錯資訊，請確保你的二進位檔（以及連結的函式庫）使用 debug symbol（`-g` 旗標）編譯。除錯資訊通常採 DWARF 格式。另外，加上 frame pointer（`-fno-omit-frame-pointer`）可讓 stack trace 更可靠，對效能分析工具尤其重要。沒有這些資訊時，stack trace 可能只剩記憶體位址，或內容不完整。這對原生編譯程式（C++、Rust）影響比 Python、Java 更大。

**也要記得它的限制：**
- LLM 可能會「一本正經地胡說八道」
- 它可能提出「遮住 bug」而非「修好 bug」的做法
- 一定要用真實除錯工具驗證建議
- 它最適合當輔助，不是取代你理解程式碼

> 這和 Development Environment 章節提到的[一般 AI 寫程式能力](/2026/development-environment/#ai-powered-development)不同。這裡特別聚焦在「把 LLM 當除錯輔助工具」。

# 效能分析

就算程式功能都正確，如果它把 CPU 或記憶體吃光，依然不夠好。演算法課常教 big _O_ 記號，但不一定教你如何找出程式中的 hot spot。既然[過早最佳化是萬惡之源](https://wiki.c2.com/?PrematureOptimization)，你更該學會使用 profiler 與監控工具。它們能幫你找出程式中最耗時／最耗資源的部分，讓你把最佳化力氣用在刀口上。

## 計時

衡量效能最簡單的方法就是計時。很多情況下，只要印出程式在兩個時間點之間花了多久，就已經很有用。

不過，牆鐘時間（wall clock time）有時會誤導你，因為電腦可能同時跑其他行程，或是在等待某些事件。`time` 指令會區分 _Real_、_User_、_Sys_：

- **Real** - 從開始到結束的牆鐘時間，包含等待時間
- **User** - CPU 執行使用者程式碼所花時間
- **Sys** - CPU 執行核心程式碼所花時間

```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real	0m0.272s
user	0m0.079s
sys	    0m0.028s
```

這個例子中，請求總共花了接近 300 毫秒（real），但 CPU 真正工作只花 107ms（user + sys），其餘時間都在等網路。

## 資源監控

有時要分析程式效能，第一步是先搞清楚它實際消耗多少資源。程式變慢，常常是因為資源受限。

- **一般監控**：[`htop`](https://htop.dev/) 是 `top` 的加強版，能顯示目前執行中行程的各種統計。常用快捷鍵：`<F6>` 排序行程、`t` 顯示樹狀階層、`h` 切換執行緒顯示。另有 [`btop`](https://github.com/aristocratos/btop)，可監看更多項目。

- **I/O 操作**：[`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html) 可即時顯示 I/O 使用資訊。

- **記憶體使用量**：[`free`](https://www.man7.org/linux/man-pages/man1/free.1.html) 顯示記憶體總量、已用與可用量。

- **開啟中的檔案**：[`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html) 列出行程開啟檔案的資訊。可用來查某個檔案被哪個行程開啟。

- **網路連線**：[`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) 可監看網路連線。常見用途是查某個 port 被哪個行程占用：`ss -tlnp | grep :8080`。

- **網路使用量**：[`nethogs`](https://github.com/raboof/nethogs) 與 [`iftop`](https://pdw.ex-parrot.com/iftop/) 都是很實用的互動式 CLI 工具，可監看每個行程的網路用量。

## 視覺化效能資料

人類看圖找規律的速度，通常比看數字表快很多。分析效能時，把資料畫成圖常能看出趨勢、尖峰與異常，這些在原始數字中可能完全看不出來。

**讓資料可繪圖**：加 print 或 log 陳述式時，建議把輸出格式化成日後容易畫圖。像 CSV 格式的時間戳與數值（`1705012345,42.5`）就比一句文字敘述更好畫圖。結構化 JSON log 也能用很少成本完成解析與繪圖。換句話說，請用[整潔資料（tidy data）](https://vita.had.co.nz/papers/tidy-data.pdf)的方式記錄。

**用 gnuplot 快速畫圖**：若是簡單命令列繪圖，[`gnuplot`](http://www.gnuplot.info/) 可以直接從資料檔產生圖表：

```bash
# 繪製 timestamp,value 的簡單 CSV
gnuplot -e "set datafile separator ','; plot 'latency.csv' using 1:2 with lines"
```

**用 matplotlib 與 ggplot2 反覆探索**：更深入分析可用 Python 的 [`matplotlib`](https://matplotlib.org/) 與 R 的 [`ggplot2`](https://ggplot2.tidyverse.org/)。和一次性畫圖不同，這些工具讓你快速切分、轉換資料以驗證假設。ggplot2 的 facet 圖尤其有力：你可以依分類把同一份資料切成多個子圖（例如依 API endpoint 或時段切 request latency），找出原本被隱藏的模式。

**範例情境：**
- 把 request latency 隨時間畫圖，可看出週期性變慢（垃圾回收、排程工作、流量模式），這些在百分位數裡可能被掩蓋
- 視覺化成長中資料結構的插入時間，可暴露演算法複雜度問題——vector 插入時間圖會在底層陣列容量倍增時出現典型尖峰
- 依不同維度（請求類型、使用者群、伺服器）做 facet，常能發現所謂「全系統問題」其實只集中在某一類

## CPU Profiler

多數情況下，大家說的 _profiler_ 指的是 _CPU profiler_。主要分兩類：

- **Tracing profiler**：記錄程式每次函式呼叫
- **Sampling profiler**：定期（常見每毫秒）抽樣程式並記錄當下堆疊

Sampling profiler 負擔較低，通常更適合在正式環境使用。

### perf：抽樣式 profiler

[`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) 是 Linux 的標準 profiler。不需重編譯就可分析任何程式：

`perf stat` 可以快速概覽時間花在哪裡：

```bash
$ perf stat ./slow_program

 Performance counter stats for './slow_program':

         3,210.45 msec task-clock                #    0.998 CPUs utilized
               12      context-switches          #    3.738 /sec
                0      cpu-migrations            #    0.000 /sec
              156      page-faults               #   48.587 /sec
   12,345,678,901      cycles                    #    3.845 GHz
    9,876,543,210      instructions              #    0.80  insn per cycle
    1,234,567,890      branches                  #  384.532 M/sec
       12,345,678      branch-misses             #    1.00% of all branches
```

真實世界程式的 profiler 輸出通常資訊量很大。人類對大量數字其實很不擅長。[火焰圖（Flame graph）](https://www.brendangregg.com/flamegraphs.html) 是一種視覺化方式，能讓效能分析資料更容易理解。

火焰圖在 Y 軸呈現函式呼叫階層，在 X 軸以寬度表示耗時。它通常可互動，你可以點擊放大特定區塊。

[![FlameGraph](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

從 `perf` 資料產生火焰圖：

```bash
# 記錄 profile
perf record -g ./my_program

# 產生火焰圖（需安裝 flamegraph scripts）
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

> 也可考慮用 [Speedscope](https://www.speedscope.app/) 這種互動式網頁火焰圖檢視器，或用 [Perfetto](https://perfetto.dev/) 做完整的系統層級分析。

### Valgrind 的 Callgrind：追蹤式 profiler

[`callgrind`](https://valgrind.org/docs/manual/cl-manual.html) 是記錄程式呼叫歷史與指令計數的分析工具。和抽樣式 profiler 不同，它提供精確呼叫次數，並顯示呼叫者與被呼叫者關係：

```bash
# 用 callgrind 執行
valgrind --tool=callgrind ./my_program

# 用 callgrind_annotate（文字）或 kcachegrind（GUI）分析
callgrind_annotate callgrind.out.<pid>
kcachegrind callgrind.out.<pid>
```

Callgrind 比抽樣式 profiler 慢，但提供精確呼叫次數，且可選擇模擬快取行為（`--cache-sim=yes`）以取得這類資訊。

> 若你使用特定語言，通常也有更專門的 profiler。例如 Python 有 [`cProfile`](https://docs.python.org/3/library/profile.html) 與 [`py-spy`](https://github.com/benfred/py-spy)，Go 有 [`go tool pprof`](https://pkg.go.dev/cmd/pprof)，Rust 有 [`cargo-flamegraph`](https://github.com/flamegraph-rs/flamegraph)。

## 記憶體 Profiler

記憶體 profiler 可幫你了解程式隨時間如何使用記憶體，並找出記憶體洩漏。

### Valgrind 的 Massif

[`massif`](https://valgrind.org/docs/manual/ms-manual.html) 用來分析 heap 記憶體使用量：

```bash
valgrind --tool=massif ./my_program
ms_print massif.out.<pid>
```

它會顯示 heap 使用量隨時間變化，有助找出記憶體洩漏與過度配置。

> 若是 Python，可使用 [`memory-profiler`](https://pypi.org/project/memory-profiler/) 取得逐行記憶體使用資訊。

## 基準測試

當你要比較不同實作或工具的效能時，[`hyperfine`](https://github.com/sharkdp/hyperfine) 是很棒的命令列程式基準測試工具：

```bash
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

> 在 Web 開發中，瀏覽器開發者工具也內建很好的 profiler。可參考 [Firefox Profiler](https://profiler.firefox.com/docs/) 與 [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/rendering-tools) 文件。

# 練習

## 除錯

1. **除錯排序演算法**：下列偽程式碼實作了 merge sort，但裡面有 bug。請用你選擇的語言實作，接著用除錯器（gdb、lldb、pdb，或 IDE 內建除錯器）找出並修正這個 bug。

   ```
   function merge_sort(arr):
       if length(arr) <= 1:
           return arr
       mid = length(arr) / 2
       left = merge_sort(arr[0..mid])
       right = merge_sort(arr[mid..end])
       return merge(left, right)

   function merge(left, right):
       result = []
       i = 0, j = 0
       while i < length(left) AND j < length(right):
           if left[i] <= right[j]:
               append result, left[i]
               i = i + 1
           else:
               append result, right[i]
               j = j + 1
       append remaining elements from left and right
       return result
   ```

   測試向量：`merge_sort([3, 1, 4, 1, 5, 9, 2, 6])` 應回傳 `[1, 1, 2, 3, 4, 5, 6, 9]`。請用中斷點並逐步執行 merge 函式，找出在哪裡選到了錯誤元素。

1. 安裝 [`rr`](https://rr-project.org/)，並用反向除錯找出資料毀損 bug。將以下程式存成 `corruption.c`：

   ```c
   #include <stdio.h>

   typedef struct {
       int id;
       int scores[3];
   } Student;

   Student students[2];

   void init() {
       students[0].id = 1001;
       students[0].scores[0] = 85;
       students[0].scores[1] = 92;
       students[0].scores[2] = 78;

       students[1].id = 1002;
       students[1].scores[0] = 90;
       students[1].scores[1] = 88;
       students[1].scores[2] = 95;
   }

   void curve_scores(int student_idx, int curve) {
       for (int i = 0; i < 4; i++) {
           students[student_idx].scores[i] += curve;
       }
   }

   int main() {
       init();
       printf("=== Initial state ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       curve_scores(0, 5);

       printf("\n=== After curving ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       if (students[1].id != 1002) {
           printf("\nERROR: Student 1's ID was corrupted! Expected 1002, got %d\n",
                  students[1].id);
           return 1;
       }
       return 0;
   }
   ```

   用 `gcc -g corruption.c -o corruption` 編譯並執行。Student 1 的 ID 會被破壞，但破壞發生在一個看似只操作 student 0 的函式裡。請用 `rr record ./corruption` 與 `rr replay` 找出元兇。對 `students[1].id` 設 watchpoint，並在資料被破壞後使用 `reverse-continue`，精準找出是哪一行覆寫了它。

1. 使用 AddressSanitizer 除錯一個記憶體錯誤。將下列程式存成 `uaf.c`：

   ```c
   #include <stdlib.h>
   #include <string.h>
   #include <stdio.h>

   int main() {
       char *greeting = malloc(32);
       strcpy(greeting, "Hello, world!");
       printf("%s\n", greeting);

       free(greeting);

       greeting[0] = 'J';
       printf("%s\n", greeting);

       return 0;
   }
   ```

   先不用 sanitizer 編譯並執行：`gcc uaf.c -o uaf && ./uaf`。它可能看起來正常。接著用 AddressSanitizer 編譯：`gcc -fsanitize=address -g uaf.c -o uaf && ./uaf`。閱讀錯誤報告。ASan 找到了什麼 bug？請修正它指出的問題。

1. 使用 `strace`（Linux）或 `dtruss`（macOS）追蹤 `ls -l` 這類指令的系統呼叫。它實際呼叫了哪些 system call？再試著追蹤更複雜的程式，看看它開啟了哪些檔案。

1. 使用 LLM 協助除錯難懂的錯誤訊息。試著貼一段編譯器錯誤（特別是 C++ template 或 Rust），請它解釋並提出修正方式。也可以把 `strace` 或 address sanitizer 的部分輸出貼進去試試看。

## 效能分析

1. 使用 `perf stat` 取得你選擇程式的基本效能統計。不同 counter 各代表什麼意思？

1. 用 `perf record` 進行效能分析。將下列程式存成 `slow.c`：

   ```c
   #include <math.h>
   #include <stdio.h>

   double slow_computation(int n) {
       double result = 0;
       for (int i = 0; i < n; i++) {
           for (int j = 0; j < 1000; j++) {
               result += sin(i * j) * cos(i + j);
           }
       }
       return result;
   }

   int main() {
       double r = 0;
       for (int i = 0; i < 100; i++) {
           r += slow_computation(1000);
       }
       printf("Result: %f\n", r);
       return 0;
   }
   ```

   用 debug symbol 編譯：`gcc -g -O2 slow.c -o slow -lm`。執行 `perf record -g ./slow`，再用 `perf report` 查看時間花在哪裡。再試著用 flamegraph scripts 產生火焰圖。

1. 使用 `hyperfine` 為同一任務的兩種不同實作做基準測試（例如 `find` vs `fd`、`grep` vs `ripgrep`，或你自己程式的兩個版本）。

1. 執行高資源需求程式時，用 `htop` 監看系統。再試 `taskset` 限制行程可用的 CPU：`taskset --cpu-list 0,2 stress -c 3`。為什麼 `stress` 沒有使用三顆 CPU？

1. 常見問題之一是你要監聽的 port 已經被其他行程占用。請練習如何找出該行程：先執行 `python -m http.server 4444` 在 4444 port 啟動最小化 Web server。再開另一個終端機執行 `ss -tlnp | grep 4444` 找到行程，最後用 `kill <PID>` 結束它。
