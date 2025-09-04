+++
date = '2025-09-04T16:10:56+08:00'
draft = 'False'
title = 'Self hosting local n8n service'
categories = ["self-hosting"]
tags = ["self-hosting", "n8n"]
ShowToc = true
TocOpen = true
+++

# Steps

## 1. 創建 Docker Compose 檔案

首先，建立一個專案目錄用來存放 n8n 的相關檔案。在終端機中執行以下命令：

```
mkdir n8n-dockercd n8n-docker
```

在專案目錄中創建一個 `docker-compose.yml` 檔案，用來定義 n8n 和 PostgreSQL 的 Docker 容器。

docker-compose.yml

```yaml
services:
  n8n:
    image: n8nio/n8n:nightly
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - ./n8n_storage:/home/node/.n8n
    depends_on:
      - postgres

  postgres:
    image: postgres:13.22-alpine3.22
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - ./postgres_storage:/var/lib/postgresql/data

volumes:
  n8n_storage:
  postgres_storage:
```

說明：

- 使用 n8n 的 Docker 映像檔，並將容器的 `5678` 埠映射到主機的 `5678` 埠。
- `postgres` 服務：使用 PostgreSQL 的 Docker 映像檔，並設定用戶名、密碼和資料庫名稱(放在 `.env` 檔案中)。
- `volumes`：分別用來保存 n8n 和 PostgreSQL 的資料。
- GENERIC_TIMEZONE 和 TZ 環境變數：設定時區為台北時區。
- 相關設定可以參考 [n8n 官方文件](https://docs.n8n.io/hosting/installation/docker)。

## 2. 創建 `.env` 檔案

在專案目錄中創建一個 `.env` 檔案，用來設定 PostgreSQL 的用戶名、密碼和資料庫名稱。
根據 [docker official document](https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/), the **.env** will be treated as env variable file

```bash
POSTGRES_USER=your_username
POSTGRES_PASSWORD=your_password
POSTGRES_DB=your_n8n_database
```

## 3. 啟動 n8n 服務

在終端機中執行以下命令，啟動 n8n 和 PostgreSQL 服務：你應該會看到建立的容器正在背景運行。

```
docker compose up -d
```

![](https://i.imgur.com/pNVWBDo.png)

我們可以確認一下postgres 的log

```
docker logs n8n-postgres-1
```

![](https://i.imgur.com/WxdS2hT.png)

## 4. 打開瀏覽器前往 n8n 頁面

<http://localhost:5678>
![](https://i.imgur.com/J8ycuRo.png)

![](https://i.imgur.com/aYferHm.png)

# Reference

<https://darrenjon.com/docs/n8n/n8n-docker-setup/>
