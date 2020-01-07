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
