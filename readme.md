# 工作流使用 - [使用 GitHub 操作自动化工作流程](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions)

![](https://github.com/ts-zhang/00/workflows/push-greet/badge.svg)  ![](https://github.com/ts-zhang/00/workflows/not-pass/badge.svg)  ![](https://github.com/ts-zhang/00/workflows/build%20iview/badge.svg)

## action使用

hello使用github的hello-world-javascript-action

my-hello使用自定义action [hello-world-javascript-action](https://github.com/ts-zhang/hello-world-javascript-action)
```
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

```
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

```
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

```
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

使用cache将npm包缓存起来

```
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
