---
title: "Github Actions 简介"
date: 2019-11-07T11:46:04+08:00
draft: false
tags: ["github"] 
keywords: ["github", "github actions", "ci", "cd", "持续集成", "持续部署"]
categories:
  - "Github"

thumbnail: "/images/github-actions/actions.png"
---

在开发 [Apache SkyWalking CLI](https://github.com/apache/skywalking-cli) 时，使用到了GitHub Actions，GitHub Actions可以灵活地构建自动化的软件开发工作流。可以编写单个任务，并将其组合以创建自定义工作流程。工作流是自定义的自动化流程，可以在存储库中对其进行设置，以在GitHub上构建，测试，打包，发布或部署任何代码项目。

借助GitHub Actions，可以直接在存储库中构建端到端的持续集成（CI）和持续部署（CD）功能。GitHub Actions支持GitHub的内置连续集成服务。

工作流在Linux，macOS，Windows和GitHub托管服务器上的容器中运行。

## 使用限制
1. 每个存储库最多可以同时执行20个工作流。
1. 每小时可以在一个存储库中的所有操作中执行多达1000个API请求。
1. 工作流程中的每个job最多可以运行6个小时的执行时间。

## 语法预览
1. `name`
1. `on`
1. `on.<event_name>.types`
1. `on.<push|pull_request>.<branches|tags>`
1. `on.<push|pull_request>.<paths>`
1. `on.schedule`
1. `env`
1. `jobs`
1. `jobs.<job_id>`
1. `jobs.<job_id>.if`
1. `jobs.<job_id>.env`
1. `jobs.<job_id>.name`
1. `jobs.<job_id>.needs`
1. `jobs.<job_id>.runs-on`
1. `jobs.<job_id>.steps`
1. `jobs.<job_id>.timeout-minutes`
1. `jobs.<job_id>.strategy`
1. `jobs.<job_id>.container`
1. `jobs.<job_id>.services`

`name`
工作流程的名称。

`on` string or array

定义监听的事件类型

如：在push时触发
```yml
on: push
```
如果需要多个，则写成数组模式
```yml
on: [push, pull_request]
```

`on.<event_name>.types`

定义触发工作流程的事件类型。

如：
```yml
on:
  release:
    types: [published, created]
```

`on.<push|pull_request>.<branches|tags>`

使用push和pull_request事件时，可以将工作流配置为在特定分支或标签上运行。如果仅定义tags或仅定义branches，则工作流将不会运行影响未定义的Git引用的事件。

```yml
on:
  push:
    branches:
      - master
      - 'releases/**'
  tags:
    - v1
    - v1.*
```
排除模式
```yml
on:
  push:
    branches-ignore:
      - master
      - 'releases/**'
  tags-ignore:
    - v1
    - v1.*
```

`on.<push|pull_request>.paths`

使用push和pull_request事件时，可以将工作流配置为在至少一个文件不匹配paths-ignore或至少一个修改的文件与已配置匹配时运行。
```yml
on:
  push:
    paths-ignore:
    - 'docs/**'
```

`on.schedule`

可以使用POSIX cron语法安排工作流在特定UTC时间运行。

```yml
on:
  schedule:
    - cron:  '*/15 * * * *'
```

`env`

可用于工作流程中所有作业和步骤的环境变量

`jobs`

工作流程运行由一个或多个作业组成。默认情况下，作业并行运行。


`jobs.<job_id>`

每个job都可以设置一个id。

```yml
jobs:
  first_job:
    name: first job
```

`jobs.<job_id>.name`

给job起个名字。

`jobs.<job_id>.needs`

设置job依赖，比如job2依赖job1

```yml
jobs:
  job1:
  job2:
    needs: job1
```

`jobs.<job_id>.runs-on`

运行job所需的操作系统

```yml
runs-on: ubuntu-latest
runs-on: [ubuntu-latest, windows-latest]
```

`jobs.<job_id>.steps`

一个job包含一系列steps，steps可以运行命令。

```yml
name: Greeting from Mona

on: push

jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest
    steps:
    - name: Print a greeting
      env:
        MY_VAR: Hi there! My name is
        FIRST_NAME: Mona
        MIDDLE_NAME: The
        LAST_NAME: Octocat
      run: |
        echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```

`jobs.<job_id>.timeout-minutes`

在GitHub自动取消工作流之前，允许工作流运行的最大分钟数。默认值：360

`jobs.<job_id>.strategy`

创建一个策略，在不同的操作系统下运行任务，例如在两个不同的操作系统上运行3个node版本

```yml
runs-on: ${{ matrix.os }}
strategy:
  matrix:
    os: [ubuntu-16.04, ubuntu-18.04]
    node: [6, 8, 10]
steps:
  - uses: actions/setup-node@v1
    with:
      node-version: ${{ matrix.node }}
```

`jobs.<job_id>.container`

指定任务运行容器

```yml
jobs:
  my_job:
    container:
      image: node:10.16-jessie
      env:
        NODE_ENV: development
      ports:
        - 80
      volumes:
        - my_docker_volume:/volume_mount
      options: --cpus 1
```

`jobs.<job_id>.services`

job依赖的其他服务，如mysql，redis

```yml
services:
  nginx:
    image: nginx
    ports:
      - 8080:80
    env:
      NGINX_PORT: 80
  redis:
    image: redis
    ports:
      - 6379/tcp
```

到此,一些 GitHub actions 常用配置参数已经熟悉完了，每个命令还有一些子命令，可以参考GitHub官方文档。
