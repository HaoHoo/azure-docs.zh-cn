---
title: "Azure 容器服务教程 - 准备应用 | Microsoft Docs"
description: "Azure 容器服务教程 - 准备应用"
services: container-service
documentationcenter: 
author: neilpeterson
manager: timlt
editor: 
tags: acs, azure-container-service
keywords: "Docker, 容器, 微服务, Kubernetes, DC/OS, Azure"
ms.assetid: 
ms.service: container-service
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/25/2017
ms.author: nepeters
ms.custom: mvc
ms.translationtype: HT
ms.sourcegitcommit: bfd49ea68c597b109a2c6823b7a8115608fa26c3
ms.openlocfilehash: 851ce819b9a1a0d917981223cc54e959b3306709
ms.contentlocale: zh-cn
ms.lasthandoff: 07/25/2017

---

# <a name="create-container-images-to-be-used-with-azure-container-service"></a>创建要与 Azure 容器服务一起使用的容器映像

在本教程的第 1 部分（共 7 部分），将准备一个要在 Kubernetes 中使用的多容器应用程序。 已完成的步骤包括：  

> [!div class="checklist"]
> * 克隆 GitHub 中的应用程序源  
> * 根据应用程序源创建容器映像
> * 在本地 Docker 环境中测试应用程序

完成后，可在本地开发环境中访问以下应用程序。

![Azure 上的 Kubernetes 群集映像](media/container-service-kubernetes-tutorials/azure-vote.png)

在后续教程中，此容器映像会被上传到 Azure 容器注册表中，然后在 Azure 托管的 Kubernetes 群集中运行。

## <a name="before-you-begin"></a>开始之前

本教程假定基本了解核心 Docker 的概念，如容器、容器映像和基本的 Docker 命令。 如需要，请参阅 [Docker 入门]( https://docs.docker.com/get-started/)，了解容器基本知识。 

若要完成本教程，需要 Docker 开发环境。 Docker 提供的包可在任何 [Mac](https://docs.docker.com/docker-for-mac/)、[Windows](https://docs.docker.com/docker-for-windows/) 或 [Linux](https://docs.docker.com/engine/installation/#supported-platforms) 系统上轻松配置 Docker。

## <a name="get-application-code"></a>获取应用程序代码

本教程使用的示例应用程序是一个基本的投票应用。 该应用程序由前端 Web 组件和后端 Redis 实例组成。 Web 组件打包到自定义容器映像中。 Redis 实例使用 Docker 中心提供的未修改的映像。  

使用 git 可将应用程序的副本下载到开发环境。

```bash
git clone https://github.com/Azure-Samples/azure-voting-app-redis.git
```

克隆的目录内包含应用程序源代码、预先创建的 Docker Compose 文件和 Kubernetes 清单文件。 整套教程都使用这些文件创建资产。 

## <a name="create-container-images"></a>创建容器映像

使用 [Docker Compose](https://docs.docker.com/compose/)，可自动根据容器映像生成并部署多容器应用程序。

运行 docker compose.yaml 文件，以创建容器映像、下载 Redis 映像和启动应用程序。

```bash
docker-compose -f ./azure-voting-app-redis/docker-compose.yaml up -d
```

完成后，使用 [docker images](https://docs.docker.com/engine/reference/commandline/images/) 命令查看创建的映像。

```bash
docker images
```

请注意，已下载或创建三个映像。 *azure-vote-front* 映像包含应用程序。 它派生自 *nginx-flask* 映像。 Redis 映像是从 Docker 中心下载的。

```bash
REPOSITORY                   TAG        IMAGE ID            CREATED             SIZE
azure-vote-front             latest     9cc914e25834        40 seconds ago      694MB
redis                        latest     a1b99da73d05        7 days ago          106MB
tiangolo/uwsgi-nginx-flask   flask      788ca94b2313        9 months ago        694MB
```

运行 [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) 命令，查看正在运行的容器。

```bash
docker ps
```

输出：

```bash
CONTAINER ID        IMAGE             COMMAND                  CREATED             STATUS              PORTS                           NAMES
82411933e8f9        azure-vote-front  "/usr/bin/supervisord"   57 seconds ago      Up 30 seconds       443/tcp, 0.0.0.0:8080->80/tcp   azure-vote-front
b68fed4b66b6        redis             "docker-entrypoint..."   57 seconds ago      Up 30 seconds       0.0.0.0:6379->6379/tcp          azure-vote-back
```

## <a name="test-application-locally"></a>在本地测试应用程序

浏览到 http://localhost:8080 查看正在运行的应用程序。

![Azure 上的 Kubernetes 群集映像](media/container-service-kubernetes-tutorials/azure-vote.png)

## <a name="clean-up-resources"></a>清理资源

现已验证应用程序功能，可停止并删除正在运行的容器。 请勿删除容器映像。 在下一教程中，会将 *azure-vote-front* 映像上传到 Azure 容器注册表实例。

运行以下命令，停止正在运行的容器。

```bash
docker-compose -f ./azure-voting-app-redis/docker-compose.yaml stop
```

使用以下命令删除已停止的容器。

```bash
docker-compose -f ./azure-voting-app-redis/docker-compose.yaml rm
```

完成后，便拥有包含 Azure Vote 应用程序的容器映像。

## <a name="next-steps"></a>后续步骤

本教程测试了应用程序并针对应用程序创建了容器映像。 已完成以下步骤：

> [!div class="checklist"]
> * 克隆 GitHub 中的应用程序源  
> * 根据应用程序源创建容器映像
> * 在本地 Docker 环境中测试应用程序

请转到下一教程，了解如何在 Azure 容器注册表中存储容器映像。

> [!div class="nextstepaction"]
> [向 Azure 容器注册表推送映像](./container-service-tutorial-kubernetes-prepare-acr.md)