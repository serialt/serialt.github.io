+++
title = 'Debug in VSCode'
date = 2024-09-08T20:09:42+08:00
draft = false

tags = ["debug"]
categories = ["vscode","DevOps"]

+++



# Debug In VSCode



## 一、GO

### 1、安装插件：

* golang.go
* r3inbowari.gomodexplorer

安装go debug相关组件

command + shift + p --> go install --> 勾选所有进行安装



### 2、lanuch.json

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch Package",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${workspaceFolder}/",
            "cwd": "${workspaceFolder}",
            "env": {
                "http_proxy": "http://127.0.0.1:8888",
                "DEST_HUB_USERNAME": "github",
                "DEST_HUB_PASSWORD": "github",
            },
            "args": [
                "-c",
                "config.yaml"
            ]
        },
        {
            "name": "Launch test function",
            "type": "go",
            "request": "launch",
            "mode": "test",
            "program": "${fileDirname}",
            "showLog": true,
            "args": [
                "-test.run",
                "Test_f1"
            ],
            "dlvLoadConfig": {
                "followPointers": true,
                "maxVariableRecurse": 1,
                "maxStringLen": 512,
                "maxArrayValues": 64,
                "maxStructFields": -1
            }
        },
    ]
}
```



## 二、python

插件：

* ms-python.python

* ms-python.debugpy
  
* donjayamanne.python-environment-manager

* mgesbert.python-path

* wolfieshorizon.python-auto-venv

* ms-python.autopep8



launch.json

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python",
            "type": "debugpy",
            "request": "launch",
            "pythonPath": "/root/py3/bin/python3",
            "program": "${file}",
            "cwd": "${workspaceRoot}",
            "env": {},
            "envFile": "${workspaceRoot}/.env",
            "debugOptions": [
                "WaitOnAbnormalExit",
                "WaitOnNormalExit",
                "RedirectOutput"
            ]
        }
    ]
}

```





## 三、Java

插件：

* vscjava.vscode-java-pack
* vscjava.vscode-java-debug
* vscjava.vscode-maven
* vscjava.vscode-java-dependency
* vscjava.vscode-spring-initializr
* redhat.java



launch.json

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "DemoApplication",
            "request": "launch",
            "mainClass": "io.local.demo.DemoApplication",
            "projectName": "demo"
        }
    ]
}
```



## 四、bash

#### 1、安装插件

安装`bashdb`插件

* rogalmic.bash-debug



#### 2、增加一个vscode launch.json配置文件

Select **Debug -> Add Configuration** to add custom debug configuration

示例：

```json
{
    "configurations": [
        {
            "type": "bashdb",
            "request": "launch",
            "name": "Bash-Debug",
            "cwd": "${workspaceFolder}",
            "program": "${workspaceFolder}/ccc.sh",
            "args": []
        },
    ]
}
```

就可以像debug其他语言一样进行调试shell脚本



五、环境变量

```
${userHome}
${workspaceFolder} 
${file} 
```

