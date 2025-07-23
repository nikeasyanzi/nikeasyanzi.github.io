+++
date = '2025-07-22T16:10:56+08:00'
draft = 'False'
title = 'Self hosting a local AI in a minute with Ollama and Open WebUI'
categories = ["self-hosting"]
tags = ["self-hosting", "Ollama", "OpenWebUI"]
ShowToc = true
TocOpen = true
+++

# Intro

With [ollama](https://ollama.com/) and [Open WebUI](https://github.com/open-webui/open-webui), self-hosting a chatgpt like service is possible. It's definitely a great news for people or corporations concerning about sharing their data with AI service providers.
In this article, I will show you how to setup a service in local environment.

# Steps

First of all, there are 2 services we need to setup, and we will host the service on container.

1. Pull [ollama container image](https://hub.docker.com/r/ollama/ollama)

```bash=
docker pull ollama/ollama:latest
```

2. Before we run the service, we need to download a model from [ollama model](https://ollama.com/models). For experiment, we select llama3.2.

![](https://i.imgur.com/iwluBge.png)

![](https://i.imgur.com/lnBbjDQ.png)

```bash=
docker exec 5eeaf ollama pull llama3.2
```

Here, I select model llama3.2. It takes a while to download depends on the network and size of selected model.  
![](https://i.imgur.com/DfxrHZv.png)

3. Run ollama service

```bash=
docker run -d -v $(PWD):/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

4. Pull Open Web UI image

```bash=
docker pull ghcr.io/open-webui/open-webui:latest
```

5. We create a folder named open-webui. Now, the directory lay out looks like the following.
   We see the downloaded llama3.2 model and the new created open-webui folder for Open Web UI data placement..

![](https://i.imgur.com/OBRCjpR.png)

7. Run Open Web UI image

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v ./open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

It would takes a while to get the container ready to work. Once, the status shows healthy, we can try to open http://localhost:3000
![](https://i.imgur.com/Z1FnT1m.png)

8. For first time login, we will need to register a new account
   ![](https://i.imgur.com/iA3YErw.png)
9. Once we log in with success, we can see the llama3.2 is selected and we can start to play with it.

![](https://i.imgur.com/WJLODOh.png)

# Docker compose with service

I also write up a docker compose file, so we can host the file with one click. Check it out.
https://github.com/nikeasyanzi/my-toolbox/tree/main/ollama_openwebui
