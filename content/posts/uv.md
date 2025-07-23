+++
date = '2025-07-01T22:40:45+08:00'
draft = 'False'
title = 'UV: A Python Package Management Tool'
categories = ["Python"]
tags = ["Python","UV"]
ShowToc = true
TocOpen = true
+++

It's been a headache to manage Python packages. But I feel I am saved from that when I meet [UV](https://github.com/astral-sh/uv).

## Introduction

There are two main concerns related to the Python package management problem.

- Python version: Different projects may need different Python interpreters. No one wants to mix them with the system default Python interpreters.

- The packages/libraries needed by the project: Intuitively, different projects have different library dependencies. Even if they all rely on a popular library such as OpenSSL, they may depend on different library versions.

Some tools are trying to solve the problem.
For example,

- [venv](https://docs.python.org/3/library/venv.html) and [pip](https://github.com/pypa/pip) provide package management but are unable to switch between different versions of Python.

- [Pyenv](https://github.com/pyenv/pyenv) solves the Python version switching problem but does not support package management.

[UV](https://github.com/astral-sh/uv) is developed and aimed at solving the aforementioned issues.

Here, I walk through how to use **uv** to manage your project.

## Walkthrough

#### Installation

For macOS and Linux. I would use

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

#### Running Python scripts with UV

Let's take a look on the main.py

```python=
$ cat main.py
def main():
    print("Hello from my-uv!")


if __name__ == "__main__":
    main()
```

While executing the main.py, a virtual environment is created automatically.

```
$ uv run main.py
Using CPython 3.13.4
Creating virtual environment at: .venv
Hello from my-uv!
```

#### Updating dependencies

Here, we use requests as a new library to be added. We can see the content pyproject.toml is also updated.

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

#### Managing Python Versions in UV

In the following prompt, we see that for the my-uv project, Python3.13 is the default option, while in the system, it is installed with Python3.9.6

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

#### Export requirements in UV

```
 uv export -o requirements.txt
```

## Conclusion

We have shown how to use uv to start a new project.
I also recommend [a comprehensive article about uv](https://www.datacamp.com/tutorial/python-uv) for people interest in this topic.
