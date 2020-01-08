# 工作流使用 - [使用 GitHub 操作自动化工作流程](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions)

![](https://github.com/ts-zhang/00/workflows/push-greet/badge.svg)  ![](https://github.com/ts-zhang/00/workflows/not-pass/badge.svg)

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

启动多个自托管节点，执行任务