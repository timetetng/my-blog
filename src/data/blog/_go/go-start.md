---
title: Go 学习记录(1)
author: 行简
pubDatetime: 2025-12-23T00:40:52+08
slug: go-learning-note-1
featured: true
draft: false
tags:
  - Golang
  - Go
  - newbee
  - coding
description: 我的 Golang 之路——基础介绍
---

## 什么是 Go, 为什么学 Go

`Go` 又称 `Golang`, 是`Google` 开源的一款计算机编程语言。是 2009 年新生的一本语言，其是 `Docker`、`k8s` 等容器应用的母语。Go 的体积很小，同时支持高并发，并且其构建是 `Self-Contained` 的，这意味它天生就是为了云上服务的，因此它也成为当今`云原生`里应用最多的语言。

之所以选择 Go 而不是 Java, 一方面在于, 本人之前因各种原因，更多接触了服务器开发和容器管理，天生对 Go 有好感。并且在初步了解 Go 和 Java 的特性和当今的市场环境后，我认为 Java 由于历史原因，在就业上仍然不可撼动，但是 Go 的现代性和云原生的特性，将成为那些拥抱新技术的、有活力的公司的最终选择。

思考数日，我选择了 Go。

## Go 基础

### 1. Intall & Initial

> 作为 archlinux 用户，这一步会相当简单:
> Just in one line!

```bash
sudo pacman -S go
```

创建一个项目目录

```bash
mkdir my-go
cd my-go
go mod init my-go
```

`go.mod` 类似 Python 的 `pyproject.toml`，用于存放包依赖和版本控制，go-cli 的现代化也是我最喜欢的一部分，几乎可以让我从 uv 无缝适应。

### Go 与 uv 常用命令对比

| 功能             | Go                        | uv                      | 说明                        |
| :--------------- | :------------------------ | :---------------------- | :-------------------------- |
| **初始化项目**   | `go mod init <name>`      | `uv init`               | 创建项目                    |
| **添加依赖**     | `go get <pkg>`            | `uv add <pkg>`          | 下载依赖包                  |
| **移除依赖**     | (手动删除后执行 tidy)     | `uv remove <pkg>`       | 移除依赖包                  |
| **整理依赖**     | `go mod tidy`             | `uv lock` / `uv sync`   | 更新锁定文件并同步依赖      |
| **运行程序**     | `go run`                  | `uv run`                | 执行代码                    |
| **构建/编译**    | `go build`                | `uv build`              | 打包                        |
| **运行测试**     | `go test ./...`           | `uv run pytest`         | 执行项目单元测试            |
| **安装全局工具** | `go install <pkg>@latest` | `uv tool install <pkg>` | 安装工具                    |
| **管理语言版本** | `go env`                  | `uv python install`     | 安装或切换解释器/编译器版本 |
| **创建虚拟环境** | 无需                      | `uv venv`               | 创建虚拟环境                |
