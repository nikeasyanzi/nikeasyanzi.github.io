+++
date = '2025-07-22T16:10:56+08:00'
draft = 'False'
title = '使用 Ollama 和 Open WebUI 在一分鐘內自架本地 AI'
categories = ["self-hosting"]
tags = ["self-hosting", "Ollama", "OpenWebUI"]
ShowToc = true
TocOpen = true
+++

# 簡介 (Intro)

透過 [ollama](https://ollama.com/) 和 [Open WebUI](https://github.com/open-webui/open-webui)，自架類似 ChatGPT 的服務已成為可能。對於關注與 AI 服務供應商分享資料問題的個人或企業來說，這絕對是個好消息。
在本文中，我將展示如何在本地環境中設置此服務。

# 步驟 (Steps)

首先，我們需要設置 2 個服務，並將這些服務架設在容器上。

1. 拉取 [ollama container image](https://hub.docker.com/r/ollama/ollama)

```bash=
docker pull ollama/ollama:latest
```

2. 在執行服務之前，我們需要從 [ollama model](https://ollama.com/models) 下載模型。作為實驗，我們選擇 llama3.2。

![](https://i.imgur.com/iwluBge.png)

![](https://i.imgur.com/lnBbjDQ.png)

```bash=
docker exec 5eeaf ollama pull llama3.2
```

這裡我選擇了 llama3.2 模型。下載需要一些時間，取決於網路速度和所選模型的大小。
![](https://i.imgur.com/DfxrHZv.png)

3. 執行 ollama 服務

```bash=
docker run -d -v $(PWD):/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

4. 拉取 Open Web UI 映像檔

```bash=
docker pull ghcr.io/open-webui/open-webui:latest
```

5. 我們建立一個名為 open-webui 的資料夾。現在，目錄結構如下所示。
   我們可以看到已下載的 llama3.2 模型以及新建立的 open-webui 資料夾，用於存放 Open Web UI 資料。

![](https://i.imgur.com/OBRCjpR.png)

7. 執行 Open Web UI 映像檔

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v ./open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

容器需要一些時間才能準備好運作。當狀態顯示為 healthy 時，我們可以嘗試開啟 http://localhost:3000
![](https://i.imgur.com/Z1FnT1m.png)

8. 首次登入時，我們需要註冊一個新帳號
   ![](https://i.imgur.com/iA3YErw.png)
9. 成功登入後，我們可以看到 llama3.2 已被選取，現在可以開始使用了。

![](https://i.imgur.com/WJLODOh.png)

# 使用 Docker Compose 整合服務

我也撰寫了一個 Docker Compose 檔案，這樣我們就可以一鍵部署服務。請參考：
https://github.com/nikeasyanzi/my-toolbox/tree/main/ollama_openwebui
