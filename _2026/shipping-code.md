---
layout: lecture
title: "打包與交付程式碼"
description: >
  學習專案打包、環境管理、版本管理，以及部署函式庫、應用程式與服務。
thumbnail: /static/assets/thumbnails/2026/lec6.png
date: 2026-01-20
ready: true
video:
  aspect: 56.25
  id: KBMiB-8P4Ns
---

讓程式碼如預期運作已經不容易；要讓同一份程式碼在不是你自己的機器上也能跑，通常更困難。

交付程式碼的意思是，把你寫的程式轉成別人可以使用的形式，讓對方不需要和你電腦一模一樣的設定也能執行。
交付方式有很多種，會受到程式語言、系統函式庫、作業系統等多種因素影響。
也會取決於你在做什麼：軟體函式庫、命令列工具、網頁服務，需求和部署步驟都不同。
不過這些情境有共同模式：我們都需要定義可交付物是什麼（也就是 _artifact_），以及它對周邊執行環境有哪些假設。

這堂課會涵蓋：

- [相依性與環境](#dependencies--environments)
- [產物與打包](#artifacts--packaging)
- [釋出與版本管理](#releases--versioning)
- [可重現性](#reproducibility)
- [虛擬機與容器](#vms--containers)
- [設定](#configuration)
- [服務與協調](#services--orchestration)
- [發佈](#publishing)

我們會用 Python 生態系的例子來解釋這些概念，因為具體範例比較容易理解。雖然不同語言生態系用的工具不同，但核心觀念大多相同。

# 相依性與環境 {#dependencies--environments}

在現代軟體開發中，抽象層幾乎無所不在。
程式很自然會把部分邏輯交給其他函式庫或服務。
但這也會引入程式與函式庫之間的 _相依性_ 關係：你的程式要正常運作，就需要那些函式庫。
例如在 Python 中，要抓網站內容我們常會寫：

```python
import requests

response = requests.get("https://missing.csail.mit.edu")
```

不過 `requests` 這個函式庫不會隨 Python 執行階段一起安裝，所以如果在沒安裝 `requests` 的情況下執行這段程式，Python 會報錯：

```console
$ python fetch.py
Traceback (most recent call last):
  File "fetch.py", line 1, in <module>
    import requests
ModuleNotFoundError: No module named 'requests'
```

要讓這個函式庫可用，我們要先執行 `pip install requests` 安裝它。
`pip` 是 Python 提供的命令列套件安裝工具。
執行 `pip install requests` 大致會做以下步驟：

1. 到 Python Package Index（[PyPI](https://pypi.org/)）搜尋 requests
1. 依照目前平台找出合適的產物（artifact）
1. 解決相依性 --- `requests` 本身還依賴其他套件，所以安裝器需要先找出所有傳遞相依套件（transitive dependencies）的相容版本並先安裝
1. 下載產物，解壓縮後把檔案放到檔案系統中的正確位置

```console
$ pip install requests
Collecting requests
  Downloading requests-2.32.3-py3-none-any.whl (64 kB)
Collecting charset-normalizer<4,>=2
  Downloading charset_normalizer-3.4.0-cp311-cp311-manylinux_x86_64.whl (142 kB)
Collecting idna<4,>=2.5
  Downloading idna-3.10-py3-none-any.whl (70 kB)
Collecting urllib3<3,>=1.21.1
  Downloading urllib3-2.2.3-py3-none-any.whl (126 kB)
Collecting certifi>=2017.4.17
  Downloading certifi-2024.8.30-py3-none-any.whl (167 kB)
Installing collected packages: urllib3, idna, charset-normalizer, certifi, requests
Successfully installed certifi-2024.8.30 charset-normalizer-3.4.0 idna-3.10 requests-2.32.3 urllib3-2.2.3
```

這裡可以看到 `requests` 有自己的相依套件，例如 `certifi`、`charset-normalizer`，必須先安裝它們才能安裝 `requests`。
安裝完成後，Python 在 `import` 時就能找到這個函式庫。

```console
$ python -c 'import requests; print(requests.__path__)'
['/usr/local/lib/python3.11/dist-packages/requests']

$ pip list | grep requests
requests        2.32.3
```

不同程式語言在安裝與發佈函式庫時，有不同工具、慣例與實務做法。
有些語言像 Rust，工具鏈是整合的 --- `cargo` 同時處理建置、測試、相依管理與發佈。
有些像 Python，整合發生在「規範」層級 --- 不是只有單一工具，而是用標準規範定義打包方式，讓每個任務可以有多個競爭工具（`pip` vs [`uv`](https://docs.astral.sh/uv/)、`setuptools` vs [`hatch`](https://hatch.pypa.io/) vs [`poetry`](https://python-poetry.org/)）。
也有像 LaTeX 這樣的生態系，TeX Live 或 MacTeX 這類發行版會預先附帶上千個套件。

引入相依套件，也會引入相依衝突。
衝突發生在不同程式需要同一套件但版本要求不相容時。
例如 `tensorflow==2.3.0` 需要 `numpy>=1.16.0,<1.19.0`，而 `pandas==1.2.0` 需要 `numpy>=1.16.5`，那麼任何符合 `numpy>=1.16.5,<1.19.0` 的版本都可以。
但若你的另一個套件要求 `numpy>=1.19`，就會出現無法同時滿足所有限制條件的衝突。

這種「多個套件需要彼此不相容的共用相依版本」的情況，通常稱為 _dependency hell_（相依地獄）。
處理衝突的方法之一，是把每個程式的相依套件隔離到各自的 _環境_。
在 Python 裡，我們可以這樣建立虛擬環境：

```console
$ which python
/usr/bin/python
$ pwd
/home/missingsemester
$ python -m venv venv
$ source venv/bin/activate
$ which python
/home/missingsemester/venv/bin/python
$ which pip
/home/missingsemester/venv/bin/pip
$ python -c 'import requests; print(requests.__path__)'
['/home/missingsemester/venv/lib/python3.11/site-packages/requests']

$ pip list
Package Version
------- -------
pip     24.0
```

你可以把環境想成一份獨立的語言執行階段，並有自己的已安裝套件集合。
這個虛擬環境（venv）會把相依套件和系統層級的 Python 安裝隔離開來。
最佳實務是每個專案都有自己的虛擬環境，只放該專案需要的相依套件。

> 雖然許多現代作業系統都會內建 Python 這類語言執行階段，但不建議直接修改這些安裝，因為作業系統本身可能依賴它們。請優先使用獨立環境。

在某些語言中，安裝流程不是由單一工具定義，而是由規範定義。
在 Python 中，[PEP 517](https://peps.python.org/pep-0517/) 定義建置系統介面，[PEP 621](https://peps.python.org/pep-0621/) 則定義如何把專案中繼資料放在 `pyproject.toml`。
這讓開發者可以在 `pip` 之外做出更優化的工具，例如 `uv`。要安裝 `uv`，執行 `pip install uv` 即可。

用 `uv` 取代 `pip` 時，操作介面幾乎一樣，但速度明顯更快：

```console
$ uv pip install requests
Resolved 5 packages in 12ms
Prepared 5 packages in 0.45ms
Installed 5 packages in 8ms
 + certifi==2024.8.30
 + charset-normalizer==3.4.0
 + idna==3.10
 + requests==2.32.3
 + urllib3==2.2.3
```

> 我們強烈建議在可行時使用 `uv pip` 取代 `pip`，因為它能大幅縮短安裝時間。

除了隔離相依套件，環境也能讓你同時使用不同版本的語言執行階段。

```console
$ uv venv --python 3.12 venv312
Using CPython 3.12.7
Creating virtual environment at: venv312

$ source venv312/bin/activate && python --version
Python 3.12.7

$ uv venv --python 3.11 venv311
Using CPython 3.11.10
Creating virtual environment at: venv311

$ source venv311/bin/activate && python --version
Python 3.11.10
```

這在你需要跨多個 Python 版本測試程式，或專案指定特定版本時特別有幫助。

> 在有些語言裡，每個專案會自動擁有自己的相依環境，而不是手動建立，但原理相同。現在多數語言也支援在同一台系統管理多個語言版本，並為個別專案指定要用哪個版本。

# 產物與打包 {#artifacts--packaging}

在軟體開發中，我們會區分原始碼（source code）與產物（artifact）。開發者撰寫與閱讀的是原始碼；產物則是從原始碼產出的可打包、可散佈結果，可直接安裝或部署。
產物可以很簡單，例如一個可執行的程式檔；也可以很複雜，例如包含應用程式所有必要元件的完整虛擬機。
看這個例子，假設目前目錄有個 Python 檔案 `greet.py`：

```console
$ cat greet.py
def greet(name):
    return f"Hello, {name}!"

$ python -c "from greet import greet; print(greet('World'))"
Hello, World!

$ cd /tmp
$ python -c "from greet import greet; print(greet('World'))"
ModuleNotFoundError: No module named 'greet'
```

移動到其他目錄後 `import` 失敗，因為 Python 只會在特定位置找模組（目前目錄、已安裝套件、以及 `PYTHONPATH` 中的路徑）。打包可以透過把程式安裝到已知位置來解決這個問題。

在 Python 中，打包函式庫的做法是產出可被 `pip` 或 `uv` 這類套件安裝器使用的產物。
Python 的產物稱為 _wheel_，其中包含安裝套件所需的所有資訊：程式碼檔案、套件中繼資料（名稱、版本、相依套件）以及檔案該放到環境哪裡的說明。
建置產物前，我們需要先寫一份專案檔（常稱 manifest），描述專案細節、必要相依套件、套件版本與其他資訊。在 Python 裡，通常用 `pyproject.toml`。

> `pyproject.toml` 是現代且建議使用的方式。雖然 `requirements.txt` 或 `setup.py` 這些較早期方法仍受支援，但在可行時請優先使用 `pyproject.toml`。

下面是一個最小化的 `pyproject.toml`，用於同時提供函式庫與命令列工具的專案：

```toml
[project]
name = "greeting"
version = "0.1.0"
description = "A simple greeting library"
dependencies = ["typer>=0.9"]

[project.scripts]
greet = "greeting:main"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

`typer` 是常見的 Python 套件，可以用很少樣板程式快速建立命令列介面。

對應的 `greeting.py` 如下：

```python
import typer


def greet(name: str) -> str:
    return f"Hello, {name}!"


def main(name: str):
    print(greet(name))


if __name__ == "__main__":
    typer.run(main)
```

有了這些檔案後，就可以建置 wheel：

```console
$ uv build
Building source distribution...
Building wheel from source distribution...
Successfully built dist/greeting-0.1.0.tar.gz
Successfully built dist/greeting-0.1.0-py3-none-any.whl

$ ls dist/
greeting-0.1.0-py3-none-any.whl
greeting-0.1.0.tar.gz
```

`.whl` 檔就是 wheel（具有特定結構的 zip 壓縮檔），`.tar.gz` 則是給需要從原始碼建置的系統使用的 source distribution。

你可以檢查 wheel 內容，看看實際被打包了哪些東西：

```console
$ unzip -l dist/greeting-0.1.0-py3-none-any.whl
Archive:  dist/greeting-0.1.0-py3-none-any.whl
  Length      Date    Time    Name
---------  ---------- -----   ----
      150  2024-01-15 10:30   greeting.py
      312  2024-01-15 10:30   greeting-0.1.0.dist-info/METADATA
       92  2024-01-15 10:30   greeting-0.1.0.dist-info/WHEEL
        9  2024-01-15 10:30   greeting-0.1.0.dist-info/top_level.txt
      435  2024-01-15 10:30   greeting-0.1.0.dist-info/RECORD
---------                     -------
      998                     5 files
```

如果把這個 wheel 給別人，對方就能這樣安裝：

```console
$ uv pip install ./greeting-0.1.0-py3-none-any.whl
$ greet Alice
Hello, Alice!
```

這會把我們剛剛建好的函式庫安裝到對方環境，包含 `greet` CLI 工具。

這種做法也有侷限。特別是當函式庫依賴平台特定函式庫（例如 GPU 加速用的 CUDA）時，產物只會在已安裝那些函式庫的系統上運作，而且可能需要為不同平台（Linux、macOS、Windows）與架構（x86、ARM）各自建 wheel。


安裝軟體時，有個重要區別：從原始碼安裝，或安裝預先建好的二進位檔。從原始碼安裝表示下載原始程式並在你的機器上編譯 --- 這需要編譯器與建置工具，大型專案可能會花很多時間。

安裝預建二進位檔則是下載別人已經編譯好的產物 --- 比較快也比較簡單，但二進位檔必須符合你的平台與架構。
例如 [ripgrep 的 releases 頁面](https://github.com/BurntSushi/ripgrep/releases) 就提供 Linux（x86_64、ARM）、macOS（Intel、Apple Silicon）與 Windows 的預建版本。


# 釋出與版本管理 {#releases--versioning}

程式碼的建置通常是持續進行，但釋出是離散批次進行。
在軟體開發中，開發環境與正式環境有明確區別。
程式碼必須先在開發環境驗證可用，才會被 _ship_ 到正式環境。
釋出流程包含許多步驟：測試、相依管理、版本管理、設定、部署與發佈。


軟體函式庫不是靜態的，會隨時間修正問題並加入新功能。
我們用離散的版本識別碼追蹤這個演進，每個版本對應某個時間點的函式庫狀態。
函式庫行為變更可能從小修補（不影響關鍵功能）、向下相容的新功能，到破壞向後相容性的變更。
Changelog 會記錄某個版本帶來哪些改動，讓開發者能對外溝通新釋出的內容。

但要持續追蹤每一個相依套件的變更非常不實際，尤其還要加上傳遞相依（也就是相依套件本身的相依套件）。

> 你可以用 `uv tree` 把整個專案的相依樹視覺化，它會用樹狀格式顯示所有套件及其傳遞相依。

為了簡化這個問題，社群發展出版本編號慣例，其中最常見的是 [Semantic Versioning](https://semver.org/)（語意化版本，SemVer）。
在 Semantic Versioning 下，版本格式是 MAJOR.MINOR.PATCH，每一段都是整數。簡單來說，升版規則如下：

- PATCH（例如 1.2.3 → 1.2.4）應只包含修 bug，並且完全向後相容
- MINOR（例如 1.2.3 → 1.3.0）加入新功能，但維持向後相容
- MAJOR（例如 1.2.3 → 2.0.0）表示有破壞性變更，可能需要修改程式碼

> 以上是簡化版，建議閱讀完整 SemVer 規範，了解例如為何 0.1.3 升到 0.2.0 也可能有破壞性變更，或 `1.0.0-rc.1` 代表什麼。
Python 打包原生支援語意化版本，所以指定相依套件版本時可以使用多種條件寫法：

在 `pyproject.toml` 中，我們可用不同方式限制相依套件的相容版本範圍：

```toml
[project]
dependencies = [
    "requests==2.32.3",  # 精確版本：只允許這個版本
    "click>=8.0",        # 最低版本：8.0 以上
    "numpy>=1.24,<2.0",  # 區間：至少 1.24、但小於 2.0
    "pandas~=2.1.0",     # 相容釋出：>=2.1.0 且 <2.2.0
]
```

許多套件管理器（npm、cargo 等）都有版本條件語法，但精確語意不完全相同。`~=` 是 Python 的「相容釋出」運算子 --- `~=2.1.0` 表示「任何與 2.1.0 相容的版本」，也就是 `>=2.1.0` 且 `<2.2.0`。這大致對應 npm 與 cargo 的插入號（`^`）運算子，遵循 SemVer 的相容概念。

不是所有軟體都用語意化版本。常見替代方案是 Calendar Versioning（CalVer），版本號以釋出日期為主，而非語意。像 Ubuntu 會用 `24.04`（2024 年 4 月）和 `24.10`（2024 年 10 月）。CalVer 的優點是容易看出版本新舊，但不會表達相容性。最後，語意化版本也不是萬無一失，維護者有時仍會在 minor 或 patch 釋出中不小心引入破壞性變更。


# 可重現性 {#reproducibility}

在現代軟體開發中，你寫的程式碼建立在許多抽象層之上。
這包含語言執行階段、第三方函式庫、作業系統，甚至是硬體本身。
這些層級只要任何一層有差異，就可能改變程式行為，甚至讓程式無法如預期運作。
此外，底層硬體差異也會影響你交付軟體的能力。

Pinning（版本釘選）指的是指定精確版本，而不是版本範圍，例如用 `requests==2.32.3` 而不是 `requests>=2.0`。

套件管理器的一項工作，是考量所有相依套件與傳遞相依套件給出的限制條件，計算出一組可同時滿足條件的有效版本清單。
這份版本清單可儲存成檔案來確保可重現性，這類檔案稱為 _lock file_。

```console
$ uv lock
Resolved 12 packages in 45ms

$ cat uv.lock | head -20
version = 1
requires-python = ">=3.11"

[[package]]
name = "certifi"
version = "2024.8.30"
source = { registry = "https://pypi.org/simple" }
sdist = { url = "https://files.pythonhosted.org/...", hash = "sha256:..." }
wheels = [
    { url = "https://files.pythonhosted.org/...", hash = "sha256:..." },
]
...
```

在處理相依版本與可重現性時，一個關鍵差異是「函式庫」與「應用程式／服務」不同。
函式庫要被其他程式匯入使用，而那些程式可能有自己的相依套件，所以版本限制設得太嚴格，容易和使用者其他相依套件衝突。
相對地，應用程式或服務是軟體的最終使用方，通常透過 UI 或 API 提供功能，而不是透過程式介面。
對函式庫來說，使用版本範圍通常是較好的實務，可最大化生態系相容性；對應用程式來說，釘選精確版本可確保可重現性 --- 每位執行者都使用完全相同的相依套件。


對可重現性要求極高的專案，可以使用 [Nix](https://nixos.org/) 或 [Bazel](https://bazel.build/) 這類工具提供 _hermetic_ build --- 包含編譯器、系統函式庫、甚至建置環境本身在內的所有輸入都被釘選且可內容定址。這能保證不論何時何地建置，都產生位元層級一致的輸出。

> 你甚至可以用 NixOS 管理整台電腦，輕鬆複製出相同的系統設定，並用版本控制的設定檔管理完整配置。

軟體開發中有個長期拉扯：新版本軟體可能有意或無意造成破壞；但舊版本又會隨時間暴露出安全性弱點。
可行作法是建立持續整合流程（在 [程式碼品質與 CI](/2026/code-quality/) 一課會講更多），持續測試應用程式對新版本軟體的相容性，並搭配自動化工具偵測相依套件新版本，例如 [Dependabot](https://github.com/dependabot)。

即使有 CI 測試，升級軟體版本時仍可能出問題，常見原因是開發環境與正式環境無可避免的不一致。
這時最佳做法是準備 _rollback_（回滾）方案，把升級還原並重新部署已知穩定版本。

# 虛擬機與容器 {#vms--containers}

當你開始依賴更複雜的套件時，程式相依項目很可能超出套件管理器可處理的範圍。
常見原因之一是需要介接特定系統函式庫或硬體驅動。
例如在科學運算與 AI 領域，程式常需要專用函式庫與驅動程式來使用 GPU。
許多系統層級相依項目（GPU 驅動、特定編譯器版本、像 OpenSSL 的共享函式庫）仍需要系統層級安裝。

傳統上，這種更廣泛的相依問題會用虛擬機（VM）處理。
VM 會抽象化整台電腦，提供一個完全隔離、擁有自己作業系統的環境。
較現代的做法是容器：把應用程式、相依套件、函式庫與檔案系統一起打包，但不虛擬整台電腦，而是共用主機的作業系統核心。
容器因為共用核心，比 VM 更輕量，啟動更快、執行更有效率。

最常見的容器平台是 [Docker](https://www.docker.com/)。Docker 提供了標準化方式來建置、散佈與執行容器。底層上 Docker 使用 containerd 作為容器執行階段，這也是像 Kubernetes 等工具採用的業界標準。

執行容器很直接。例如要在容器內跑 Python 直譯器，可以用 `docker run`（`-it` 參數會讓容器與終端機互動；離開後容器就停止）。

```console
$ docker run -it python:3.12 python
Python 3.12.7 (main, Nov  5 2024, 02:53:25) [GCC 12.2.0] on linux
>>> print("Hello from inside a container!")
Hello from inside a container!
```

實務上，你的程式可能依賴整個檔案系統內容。
為了解決這件事，可以用容器映像檔把應用程式的整個檔案系統一起當作產物交付。
容器映像檔是用程式化方式建立的。在 Docker 中，我們透過 Dockerfile 語法精確描述映像檔所需的相依套件、系統函式庫與設定：

```dockerfile
FROM python:3.12
RUN apt-get update
RUN apt-get install -y gcc
RUN apt-get install -y libpq-dev
RUN pip install numpy
RUN pip install pandas
COPY . /app
WORKDIR /app
RUN pip install .
```

一個重要差異：Docker **image** 是打包好的產物（像模板），而 **container** 是該 image 的執行實例。你可以從同一個 image 啟動多個 container。Image 由多層組成，Dockerfile 中每一個指令（`FROM`、`RUN`、`COPY` 等）都會建立新層。Docker 會快取這些層，所以如果你只改 Dockerfile 的某一行，通常只需要重建該層與後續層。

前面的 Dockerfile 有幾個問題：它使用完整 Python image 而非 slim 版本、把 `RUN` 拆成多條導致不必要的層、沒有釘選版本、也沒有清理套件管理器快取，造成多帶不必要檔案。其他常見錯誤還包括以 root 身分不安全地執行容器，或不小心把密鑰放進 image 層。

改良版本如下：

```dockerfile
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc libpq-dev && \
    rm -rf /var/lib/apt/lists/*
COPY pyproject.toml uv.lock ./
RUN uv pip install --system -r uv.lock
COPY . /app
```

在上面的範例裡，我們不是從原始碼安裝 `uv`，而是從 `ghcr.io/astral-sh/uv:latest` 映像檔直接複製預建二進位檔。這叫做 _builder pattern_。用這種模式就不必把所有編譯工具一起打包，只需要交付執行應用程式真正需要的最終二進位檔（此例中是 `uv`）。

Docker 也有重要限制要注意。第一，容器映像檔常是平台特定 --- 為 `linux/amd64` 建的 image 無法在 `linux/arm64`（Apple Silicon Mac）原生執行，通常要靠模擬，速度較慢。第二，Docker 容器需要 Linux 核心，所以在 macOS 與 Windows 上，Docker 底層其實會跑一個輕量 Linux VM，帶來額外負擔。第三，Docker 的隔離性弱於 VM --- 容器共用主機核心，在多租戶環境會有安全疑慮。

> 近年也有更多專案透過 [nix flakes](https://serokell.io/blog/practical-nix-flakes)，用 nix 以「每專案」方式管理原本偏系統層級的函式庫與應用程式。

# 設定 {#configuration}

軟體本質上就是可設定的。在[命令列環境](/2026/command-line-environment/)這堂課裡，我們看過程式可以透過旗標、環境變數，甚至設定檔（dotfiles）接收選項。更複雜的應用程式也是如此，而且已有成熟做法可在規模化時管理設定。
軟體設定不應寫死在程式碼中，而應在執行時提供。
常見方式是環境變數與設定檔。

以下是使用環境變數設定應用程式的例子：

```python
import os

DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///local.db")
DEBUG = os.environ.get("DEBUG", "false").lower() == "true"
API_KEY = os.environ["API_KEY"]  # 必填：若未設定會拋出錯誤
```

應用程式也可以透過設定檔（例如 Python 程式用 `yaml.load` 讀取設定）來設定，例如 `config.yaml`：

```yaml
database:
  url: "postgresql://localhost/myapp"
  pool_size: 5
server:
  host: "0.0.0.0"
  port: 8080
  debug: false
```

思考設定的一個實用原則是：同一份程式碼應能只靠設定差異，就部署到不同環境（開發、預備、正式），而不需要改程式碼。

在各種設定中，常常會包含 API 金鑰等敏感資料。
這些 secrets 必須謹慎處理，避免外洩，也絕對不能放進版本控制。


# 服務與協調 {#services--orchestration}

現代應用程式很少單獨存在。典型的網頁應用可能需要資料庫做持久化儲存、快取提升效能、訊息佇列處理背景工作，還有其他支援服務。現代架構常把功能拆成可獨立開發、部署與擴展的服務，而不是全部綁在單體應用程式中。

舉例來說，若我們判斷應用程式適合加快取，與其自己重造輪子，不如使用經過實戰驗證的方案，例如 [Redis](https://redis.io/) 或 [Memcached](https://memcached.org/)。
我們可以把 Redis 直接打進應用程式容器，但那代表要協調 Redis 與應用程式所有相依關係，實作上可能困難甚至不可行。
另一個方式是把每個元件各自部署在獨立容器。
這通常稱為微服務架構：每個元件是獨立服務，透過網路溝通，常見是 HTTP API。

[Docker Compose](https://docs.docker.com/compose/) 是定義與執行多容器應用程式的工具。你不必逐一管理容器，而是用單一 YAML 檔宣告所有服務並一起協調啟動。此時完整應用就包含多個容器：

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - REDIS_URL=redis://cache:6379
    depends_on:
      - cache

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

執行 `docker compose up` 後，兩個服務會一起啟動，Web 應用程式可用主機名稱 `cache` 連線到 Redis（Docker 內建 DNS 會自動解析服務名稱）。
Docker Compose 讓我們宣告一或多個服務的部署方式，並處理一起啟動、網路連線設定與資料持久化用的共享 volume 管理。

在正式環境部署時，你通常會希望 docker compose 服務在開機時自動啟動，失敗時自動重啟。常見做法是用 systemd 管理 docker compose 部署：

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down

[Install]
WantedBy=multi-user.target
```

這個 systemd unit 檔能確保系統開機後（Docker 就緒時）啟動你的應用程式，並提供標準控制指令，例如 `systemctl start myapp`、`systemctl stop myapp`、`systemctl status myapp`。

當部署需求變得更複雜 --- 例如要跨多台機器擴展、服務故障時容錯、以及高可用保證 --- 組織通常會採用像 Kubernetes（k8s）這類進階容器協調平台，它可以跨機器叢集管理數千個容器。不過 Kubernetes 學習曲線陡峭、維運成本也高，對小型專案常常太重。

多容器架構之所以可行，部分原因是現代服務通常透過標準化 API（尤其 HTTP REST API）彼此溝通。舉例來說，當程式和 OpenAI 或 Anthropic 這類 LLM 供應商互動時，底層其實是在送出 HTTP 請求並解析回應：

```console
$ curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "content-type: application/json" \
    -H "anthropic-version: 2023-06-01" \
    -d '{"model": "claude-sonnet-4-20250514", "max_tokens": 256,
         "messages": [{"role": "user", "content": "Explain containers vs VMs in one sentence."}]}'
```

# 發佈 {#publishing}

當你已證明程式碼可運作後，下一步可能是把它散佈出去，讓其他人下載安裝。
散佈方式很多，且和你使用的程式語言與執行環境高度相關。

最簡單的散佈方式是上傳產物，讓使用者自行下載並在本機安裝。
這種方式現在仍常見，例如 [Ubuntu 套件封存](http://archive.ubuntu.com/ubuntu/pool/main/) 本質上就是 `.deb` 檔案的 HTTP 目錄列表。

近年來，GitHub 已成為發佈原始碼與產物的事實標準平台。
雖然原始碼通常會公開，但 GitHub Releases 還能讓維護者把預建二進位檔與其他產物附加在 tag 版本上。


套件管理器有時也支援直接從 GitHub 安裝，不論是從原始碼或預建 wheel：

```console
# 從原始碼安裝（會 clone 並建置）
$ pip install git+https://github.com/psf/requests.git

# 從指定 tag/branch 安裝
$ pip install git+https://github.com/psf/requests.git@v2.32.3

# 直接從 GitHub release 安裝 wheel
$ pip install https://github.com/user/repo/releases/download/v1.0/package-1.0-py3-none-any.whl
```

其實像 Go 這類語言使用的是去中心化散佈模型 --- 不依賴中央套件倉庫，Go modules 直接從原始碼儲存庫散佈。
像 `github.com/gorilla/mux` 這類模組路徑會指出程式碼位置，`go get` 會直接從那裡抓取。不過大多數套件管理器（`pip`、`cargo`、`brew`）仍有集中式索引，方便散佈與安裝預先打包好的專案。若我們執行：

```console
$ uv pip install requests --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: requests==2.32.5 [compatible] (requests-2.32.5-py3-none-any.whl)
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl.metadata
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl
```

我們可以看到 `requests` wheel 是從哪裡抓下來的。注意檔名中的 `py3-none-any` --- 這代表 wheel 可用於任何 Python 3 版本、任何作業系統、任何架構。若套件含有編譯程式碼，wheel 就會是平台特定：

```console
$ uv pip install numpy --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: numpy==2.2.1 [compatible] (numpy-2.2.1-cp312-cp312-macosx_14_0_arm64.whl)
```

這裡的 `cp312-cp312-macosx_14_0_arm64` 表示此 wheel 專用於 CPython 3.12、macOS 14+、ARM64（Apple Silicon）。若你在其他平台，`pip` 會下載不同 wheel，或改為從原始碼建置。

反過來說，若要讓別人找得到我們建立的套件，就需要把它發佈到這些 registry 之一。
在 Python 中，主要 registry 是 [Python Package Index（PyPI）](https://pypi.org)。
和安裝一樣，發佈套件也有多種方式。`uv publish` 提供了上傳套件到 PyPI 的現代化介面：

```console
$ uv publish --publish-url https://test.pypi.org/legacy/
Publishing greeting-0.1.0.tar.gz
Publishing greeting-0.1.0-py3-none-any.whl
```

這裡我們使用的是 [TestPyPI](https://test.pypi.org) --- 一個獨立套件 registry，讓你測試發佈流程而不污染真正的 PyPI。上傳後可從 TestPyPI 安裝：

```console
$ uv pip install --index-url https://test.pypi.org/simple/ greeting
```

發佈軟體時有個關鍵考量是信任。使用者要怎麼確認下載的套件真的是你發佈，而且沒被竄改？套件 registry 會用 checksum 驗證完整性，有些生態系也支援套件簽章，提供密碼學上的作者證明。

不同語言有各自的套件 registry：Rust 用 [crates.io](https://crates.io)、JavaScript 用 [npm](https://www.npmjs.com)、Ruby 用 [RubyGems](https://rubygems.org)、容器映像檔常用 [Docker Hub](https://hub.docker.com)。至於私有或內部套件，組織常會自建套件倉庫（例如私有 PyPI 或私有 Docker registry），或使用雲端供應商的託管方案。

把網頁服務部署到網際網路，還需要額外基礎設施：網域註冊、把網域指向伺服器的 DNS 設定，以及常見的反向代理（如 nginx）來處理 HTTPS 與流量轉送。若是文件或靜態網站這類較簡單情境，[GitHub Pages](https://pages.github.com/) 可直接從儲存庫提供免費託管。

<!--
## 文件

到目前為止，我們強調了可交付 _artifact_ 是打包與交付程式碼的主要輸出。
除了 artifact，還需要為使用者撰寫文件，說明程式功能、安裝方式與使用範例。

像 [Sphinx](https://www.sphinx-doc.org/)（Python）和 [MkDocs](https://www.mkdocs.org/) 這類工具，可以從 docstring 與 markdown 檔自動產生可瀏覽文件，常部署在 [Read the Docs](https://readthedocs.org/) 等服務上。
對 HTTP API 而言，[OpenAPI 規範](https://www.openapis.org/)（舊稱 Swagger）提供描述 API 端點的標準格式，工具可據此自動產生互動式文件與用戶端函式庫。 -->


# 練習題

1. 用 `printenv` 把目前環境存成檔案，建立 venv 並啟用，再用 `printenv` 存另一份檔案，接著執行 `diff before.txt after.txt`。環境哪些地方改變了？為什麼 shell 會優先使用 venv？（提示：看啟用前後的 `$PATH`。）再執行 `which deactivate`，推理 `deactivate` 這個 bash function 做了什麼。
1. 用 `pyproject.toml` 建立一個 Python 套件，並安裝到虛擬環境。建立 lockfile 並檢視其內容。
1. 安裝 Docker，並用 docker compose 在本機建置 Missing Semester 課程網站。
1. 為簡單的 Python 應用程式撰寫 Dockerfile。接著再寫一個 `docker-compose.yml`，讓應用程式與 Redis 快取一起執行。
1. 將 Python 套件發佈到 TestPyPI（除非真的值得分享，否則不要發佈到正式 PyPI！）。接著用該套件建置 Docker image，並推送到 `ghcr.io`。
1. 使用 [GitHub Pages](https://docs.github.com/en/pages/quickstart) 做一個網站。額外（不）加分：設定自訂網域。
