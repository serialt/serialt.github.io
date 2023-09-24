+++
title = 'Debug Shell in VSCode'
date = 2023-09-24T18:09:42+08:00
draft = false

tags = ["shell"]
categories = ["vscode"]

+++



# Debug shell



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

