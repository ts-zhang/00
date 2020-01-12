# 工作流使用 - [使用 GitHub 操作自动化工作流程](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions)

![](https://github.com/ts-zhang/00/workflows/push-greet/badge.svg)  ![](https://github.com/ts-zhang/00/workflows/not-pass/badge.svg)  ![](https://github.com/ts-zhang/00/workflows/build%20iview/badge.svg) ![](https://github.com/ts-zhang/00/workflows/Go/badge.svg)

## action使用

hello使用github的hello-world-javascript-action

my-hello使用自定义action [hello-world-javascript-action](https://github.com/ts-zhang/hello-world-javascript-action)
```yml
#push时候打招呼
name: push-greet
#触发事件：push
on: [push]

jobs:
    build:
        #作业名称greet
        name: greet
        #运行环境
        runs-on: windows-latest
        steps:
          - name: hello
            uses: actions/hello-world-javascript-action@v1
            with:
                who-to-greet: "2020"
            id: hello
          - name: Echo time
            run: echo 'The time was ${{ steps.hello.outputs.time }}'
          
          - name: "my-hello"
            uses: ts-zhang/hello-world-javascript-action@v1
            with:
                  name: "world"
            id: my-action
```

## docker使用

```yml
#push时候执行脚本
name: push-script
#触发事件：push
on: [push]

jobs:
    job1:
        name: ip
        #运行环境
        runs-on: ubuntu-latest
        steps:
          - name: curl google
            run: curl http://www.google.com
            shell: bash
    
    job2:
        name: docker
        runs-on: macos-latest
        steps:
          - name: Display the path by python
            run: |
              import os
              print(os.environ['PATH'])
            shell: python
          - name: Display the path by shell
            run: echo $PATH
            shell: bash
          - name: curl
            run: 'curl -X GET "http://www.httpbin.org/anything" -H "accept: application/json"'
            shell: bash
```

## 变量使用

变量分为三种类型

1. [默认环境变量](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions/using-environment-variables#default-environment-variables) - job1
2. [通过env传递的环境变量](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions/using-environment-variables#about-environment-variables) - job2
3. 用户创建的[encrypted-secrets](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets#creating-encrypted-secrets)变量 - job3

```yml
#push时候显示变量的值
name: show-variables
#触发事件：push
on: [push]

jobs:
    job1:
        #显示默认环境变量
        name: show-home
        #运行环境
        runs-on: windows-latest
        steps:
          - name: home
            run: echo $HOME
            id: home
    
    job2:
        #显示自定义环境变量
        name: show-env-variables
        runs-on: ubuntu-latest
        steps:
          - name: Hello world
            run: echo Hello world $FIRST_NAME $middle_name $Last_Name!
            env:
              FIRST_NAME: Mona
              middle_name: The
              Last_Name: Octocat
    job3:
        #显示密码变量
        name: show-secrets
        runs-on: macos-latest
        steps:
          - name: secrets
            run: echo 'password is ${{ secrets.password }}!'
```

## 托管主机

操作系统
- Ubuntu 16.04 LTS
- Ubuntu 18.04 LTS
- Windows Server 2016
- Windows Server 2019
- macOS 10.15

[已安装软件列表](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions/software-installed-on-github-hosted-runners)

## 运行在自托管主机上

点击 Settings->Actions->Self-hosted runners->Add runner按钮添加自定义托管主机

按照提示步骤按照并运行

Self-hosted runners中将显示运行的自托管节点

将需要运行在自托管主机上的job修改为runs-on: self-hosted即可

push代码后发现自托管节点输出如下：
```
2020-01-08 14:37:02Z: Running job: run-on-my-pc(win)
2020-01-08 14:37:11Z: Job run-on-my-pc(win) completed with result: Succeeded
```
job在自托管节点中执行成功

启动多个自托管节点，执行任务，发现任务在其中一个节点执行，如果一个节点挂掉，下次任务会在另一个节点执行，通过此特性可以实现任务托管的高可用

## 持久化job数据

主要使用[upload-artifact](https://github.com/actions/upload-artifact)和[download-artifact](https://github.com/actions/download-artifact)组件

通过**upload-artifact**上传文件，主要用途如下

- 日志文件和核心转储文件
- 测试结果、失败和屏幕截图
- 二进制或存档文件
- 压力测试性能输出和代码覆盖结果
- 在不同job中共享数据 需要使用**download-artifact**下载文件

nodejs项目自动打包

步骤：

1. 代码checkout

2. 执行 ```npm install``` 和 ```npm build``` 命令

3. 将打包后的dist目录上传

```yml
name: build iview

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v1
      - name: npm install, build
        run: |
          cd iview-admin
          npm install
          npm run build --if-present
      - name: Archive production artifacts
        uses: actions/upload-artifact@v1
        with:
          name: dist
          path: iview-admin/dist
```

使用[cache](https://github.com/actions/cache)将npm包缓存起来

```yml
- name: Cache node modules
      uses: actions/cache@v1
      with:
        path: ~/.npm # npm cache files are stored in `~/.npm` on Linux/macOS
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-build-${{ env.cache-name }}-
          ${{ runner.os }}-build-
          ${{ runner.os }}-
```
针对不同语言的包缓存[cache](https://github.com/actions/cache/blob/master/examples.md)

## [触发事件](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions/events-that-trigger-workflows)

| Webhook事件 | 描述 | 活动类型 |
| ----- | ----- | ----- |
| check_run | 检查运行 | created,rerequested,completed,requested_action |
| check_suite | 检查套件 | completed,requested,rerequested |
| create | 创建分支/标记 | n/a |
| delete | 删除分支/标记 | n/a |
| deployment | 部署 | n/a |
| deployment_status | 部署状态 | n/a	|
| fork | 复刻 | n/a |
| gollum | wiki更新 | n/a |
| issue_comment | 议题评论 | created,edited,deleted |
| issues | 议题 | opened,edited,deleted,... |
| label | 标签 | created,edited,deleted |
| member | 成员 | added,edited,deleted |
| milestone | 里程碑 | opened,deleted |
| page_build | 页面构建 | n/a |
| project | 项目 | created,deleted |
| project_card | 项目卡 | opened,deleted |
| project_column | 项目列 | created,updated | 
| public | 私库公开 | n/a |
| pull_request | 拉请求 | assigned,unassigned,labeled... |
| pull_request_review | 拉请求审查 | submitted,edited,dismissed | 
| pull_request_review_comment | 拉请求审查评论 | created,edited,deleted |
| push | 推送 | n/a |
| release | 版本发布 | published,unpublished,created... |
| status | 提交状态变化 | n/a |
| watch | 关注 | started |
| schedule | 定时调度 | n/a |
| repository_dispatch | 创建仓库分发 | n/a |

## [创建自己的javascript action](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions/creating-a-javascript-action)

预备工作

1. 安装nodejs和npm
2. 在github中创建仓库js-echo-action
3. clone到本地
4. 使用```npm init -y```命令初始化

开始创建action

1. 创建元数据文件action.yml

```yml
name: 'echo'
description: 'Echo someone and record the time'
inputs:
  echo-what:  # 输入 id
    description: 'What to echo'
    required: true
    default: 'hello'
outputs:
  time: # 输出 id
    description: 'The time we greeted you'
runs:
  using: 'node12'
  main: 'index.js'
```

2.添加操作工具包

```
npm install @actions/core
npm install @actions/github
```

3.编写操作js代码 index.js

```js
const core = require('@actions/core');
const github = require('@actions/github');

try {
  // `echo-what` input defined in action metadata file
  const whatToEcho = core.getInput('echo-what');
  console.log(`Hello ${whatToEcho}!`);
  const time = (new Date()).toTimeString();
  core.setOutput("time", time);
  // Get the JSON webhook payload for the event that triggered the workflow
  const payload = JSON.stringify(github.context.payload, undefined, 2)
  console.log(`The event payload: ${payload}`);
} catch (error) {
  core.setFailed(error.message);
}
```

4.编写描述文件 README.md

```md
# Echo javascript action

此操作将 "Hello World" 或 "Hello" + 要问候的人员的姓名打印到日志。

## Inputs

### `echo-what`

**必填** 要问候的人员的姓名。 默认值为 `"World"`。

## Outputs

### `time`

我们问候您的时间。

## Example usage

使用：actions/js-echo-action@v1
及：
  echo-what: 'Tom'

```

5.提交到git仓库
```
git add .
git commit -m "添加action文件"
git push
```

6.使用自己的action

```yml
#push时候使用自己的action输出
name: use-my-echo
#触发事件：push
on: [push]

jobs:
    job1:
        name: js-echo
        runs-on: windows-latest
        steps:
          - name: echo
            uses: ts-zhang/js-echo-action@master
            with:
                echo-what: "my js echo action works."
            id: echo
    
```

## [创建自己的docker action](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions/creating-a-docker-container-action)

1. 创建元数据文件action.yml
```yml
name: 'echo'
description: 'Echo someone and record the time'
inputs:
  echo-what:  # 输入 id
    description: 'What to echo'
    required: true
    default: 'hello'
outputs:
  time: # 输出 id
    description: 'The time we greeted you'
runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.echo-what }}
```
2.编写Dockerfile
```Dockerfile
# 运行代码的容器图像
FROM alpine:3.10

# 从操作仓科到容器的文件系统路径 `/`的副本
COPY entrypoint.sh /entrypoint.sh

# 添加执行权限
RUN chmod +x /entrypoint.sh

# Docker 容器启动时执行的代码文件 (`entrypoint.sh`)
ENTRYPOINT ["/entrypoint.sh"]
```
3.编写脚本文件 entrypoint.sh
```bash
#!/bin/sh -l

echo "Hello $1"
time=$(date)
echo ::set-output name=time::$time
```
4.编写描述文件 README.md
```md
# Echo docker action

此操作将 "Hello World" 或 "Hello" + 要问候的人员的姓名打印到日志。

## Inputs

### `echo-what`

**必填** 要问候的人员的姓名。 默认值为 `"World"`。

## Outputs

### `time`

我们问候您的时间。

## Example usage

使用：actions/docker-echo-action@master
及：
  echo-what: 'Tom'
```
5.提交到git仓库
```
git add .
git commit -m "添加action文件"
git push
```
6.使用自己的action

```yml
#push时候使用自己的action输出
name: docker-echo
#触发事件：push
on: [push]

jobs:
    job1:
        name: docker-echo
        runs-on: ubuntu-latest
        steps:
          - name: echo
            uses: ts-zhang/docker-echo-action@master
            with:
                echo-what: "my docker echo action works."
            id: echo

```

## golang项目持续集成

1.创建golang项目go-demo

2.创建go.yml文件
```yml
name: Go
on: [push]
jobs:

  build:
    name: Build
    runs-on: ubuntu-latest
    steps:

    - name: Set up Go 1.13
      uses: actions/setup-go@v1
      with:
        go-version: 1.13
      id: go

    - name: Check out code into the Go module directory
      uses: actions/checkout@v2

    - name: Get dependencies
      run: |
        cd go-demo
        go get -v -t -d ./...
        if [ -f Gopkg.toml ]; then
            curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
            dep ensure
        fi
    - name: Build
      run: |
        cd go-demo
        go build -v ./go-demo
```
3.提交代码后会自动执行```go build```

4.[更多ci模板](https://github.com/actions/starter-workflows/tree/master/ci)

