+++
date = '2025-07-01T22:40:45+08:00'
draft = 'False'
title = 'UV：Python 套件管理工具'
categories = ["Python"]
tags = ["Python","UV"]
ShowToc = true
TocOpen = true
+++

管理 Python 套件一直是個令人頭痛的問題。但當我遇到 [UV](https://github.com/astral-sh/uv) 後，我覺得自己得救了。

## 簡介 (Introduction)

Python 套件管理問題主要有兩個考量：

- Python 版本：不同專案可能需要不同的 Python 直譯器。沒有人想把它們與系統預設的 Python 直譯器混在一起。

- 專案所需的套件／函式庫：直覺上，不同專案有不同的函式庫相依性。即使它們都依賴像 OpenSSL 這樣的熱門函式庫，也可能依賴不同的函式庫版本。

有些工具嘗試解決這個問題。
例如：

- [venv](https://docs.python.org/3/library/venv.html) 和 [pip](https://github.com/pypa/pip) 提供套件管理，但無法在不同 Python 版本之間切換。

- [Pyenv](https://github.com/pyenv/pyenv) 解決了 Python 版本切換問題，但不支援套件管理。

[UV](https://github.com/astral-sh/uv) 的開發目標就是解決上述問題。

在這裡，我將逐步介紹如何使用 **uv** 來管理你的專案。

## 實作演練 (Walkthrough)

#### 安裝 (Installation)

對於 macOS 和 Linux，我會使用

```
curl -LsSf https://astral.sh/uv/install.sh | sh
```

or through **brew** for macOS

```
brew install uv
```

#### Important files

The are some key files for UV to manage the project dependency.
In addition, the good news is UV automatically generates and updates these files. Let's take a quick look.

- .python-version: contains the Python version used for the project

- pyproject.toml: serves as the main configuration file for project metadata and dependencies.

- uv.lock: Lock files for dependency management in UV.

#### Initialization

```
$ uv init my-uv
Initialized project `my-uv` at `/Users/craigyang/workplace/my-uv`

$ cd my-uv

$ tree -a -L 1
.
├── .git
├── .gitignore
├── .python-version
├── main.py
├── pyproject.toml
└── README.md

2 directories, 5 files
```

#### 使用 UV 執行 Python 腳本 (Running Python scripts with UV)

讓我們看一下 main.py

```python=
$ cat main.py
def main():
    print("Hello from my-uv!")


if __name__ == "__main__":
    main()
```

執行 main.py 時，虛擬環境會自動建立。

```
$ uv run main.py
Using CPython 3.13.4
Creating virtual environment at: .venv
Hello from my-uv!
```

#### 更新相依性 (Updating dependencies)

在這裡，我們使用 requests 作為要新增的新函式庫。我們可以看到 pyproject.toml 的內容也被更新了。

```
$ uv add requests
Resolved 6 packages in 605ms
Prepared 2 packages in 223ms
Installed 5 packages in 10ms
 + certifi==2025.4.26
 + charset-normalizer==3.4.2
 + idna==3.10
 + requests==2.32.4
 + urllib3==2.4.0
```

```
$ cat pyproject.toml
[project]
name = "my-uv"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.13"
dependencies = [
    "requests>=2.32.4",
]
```

#### 在 UV 中管理 Python 版本 (Managing Python Versions in UV)

在以下提示中，我們看到對於 my-uv 專案，Python3.13 是預設選項，而在系統中安裝的是 Python3.9.6

```
$ uv python list
cpython-3.14.0b1-macos-aarch64-none                 <download available>
cpython-3.14.0b1+freethreaded-macos-aarch64-none    <download available>
cpython-3.13.4-macos-aarch64-none                   /opt/homebrew/bin/python3.13 -> ../Cellar/python@3.13/3.13.4/bin/python3.13
cpython-3.13.4-macos-aarch64-none                   /opt/homebrew/bin/python3 -> ../Cellar/python@3.13/3.13.4/bin/python3
cpython-3.13.4-macos-aarch64-none                   /Users/craigyang/.local/share/uv/python/cpython-3.13.4-macos-aarch64-none/bin/python3.13
cpython-3.13.4+freethreaded-macos-aarch64-none      <download available>
cpython-3.12.11-macos-aarch64-none                  <download available>
cpython-3.11.13-macos-aarch64-none                  <download available>
cpython-3.10.18-macos-aarch64-none                  <download available>
cpython-3.9.23-macos-aarch64-none                   <download available>
cpython-3.9.6-macos-aarch64-none                    /usr/bin/python3
```

#### 在 UV 中匯出 requirements (Export requirements in UV)

```
 uv export -o requirements.txt
```

## 結論 (Conclusion)

我們已經展示了如何使用 uv 來開始一個新專案。
我也推薦[這篇關於 uv 的完整文章](https://www.datacamp.com/tutorial/python-uv)給對此主題有興趣的人。
