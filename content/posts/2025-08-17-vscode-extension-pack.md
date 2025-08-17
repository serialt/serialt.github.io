+++
title = 'VSCode Extension Pack'
date = 2025-08-17T20:45:01+08:00
draft = false

tags = ["vscode","Extension-Pack"]
categories = ["VSCode Extension Pack"]

+++
VSCode Extension Pack

vscode扩展包

在使用vscode的时候，需要安装各种各类的插件来满足不同的功能，在新安装vscode或者切换其他机器的时候，会因为插件过多不方便管理，vscode提供一个功能，通过一个Extension Pack来实现一键安装多个插件。



项目结构

```shell
[root@Krab sugar-extension-pack]🐳 tree .
.
├── CHANGELOG.md
├── LICENSE
├── Makefile
├── README.md
├── images
│   └── icon.png
├── package-lock.json
└── package.json

2 directories, 7 files
```

文件内容

```json
# package.json
{
  "name": "sugar-extension-pack",
  "displayName": "Sugar's Extension Pack",
  "description": "A custom VS Code Extension Pack",
  "version": "1.0.2",
  "publisher": "serialt",
  "engines": {
    "vscode": "^1.70.0"
  },
  "categories": [
    "Extension Packs"
  ],
  "extensionPack": [
    "alefragnani.project-manager",
    "arjun.swagger-viewer",
    "aykutsarac.jsoncrack-vscode",
    "christian-kohler.path-intellisense",
    "codezombiech.gitignore",
    "cweijan.vscode-office",
    "cweijan.vscode-typora",
    "donjayamanne.githistory",
    "donjayamanne.python-environment-manager",
    "eamodio.gitlens",
    "felipecaputo.git-project-manager",
    "gerrnperl.outline-map",
    "golang.go",
    "gruntfuggly.todo-tree",
    "hashicorp.hcl",
    "hashicorp.terraform",
    "howardzuo.vscode-git-tags",
    "inferrinizzard.prettier-sql-vscode",
    "ionutvmi.path-autocomplete",
    "jeff-hykin.better-dockerfile-syntax",
    "jkillian.custom-local-formatters",
    "kevinrose.vsc-python-indent",
    "letmaik.git-tree-compare",
    "lkqm.gitblame-annotations",
    "lunuan.kubernetes-templates",
    "mads-hartmann.bash-ide-vscode",
    "meezilla.json",
    "mhutchie.git-graph",
    "moshfeu.diff-merge",
    "ms-kubernetes-tools.vscode-kubernetes-tools",
    "ms-python.autopep8",
    "ms-python.debugpy",
    "ms-python.python",
    "ms-python.vscode-pylance",
    "ms-vscode-remote.remote-ssh",
    "ms-vscode-remote.remote-ssh-edit",
    "ms-vscode.remote-explorer",
    "neonxp.gotools",
    "pascalreitermann93.vscode-yaml-sort",
    "premparihar.gotestexplorer",
    "quicktype.quicktype",
    "quzhen.tasks-button",
    "r3inbowari.gomodexplorer",
    "redhat.vscode-xml",
    "redhat.vscode-yaml",
    "redis.redis-for-vscode",
    "rogalmic.bash-debug",
    "shaharkazaz.git-merger",
    "technosophos.vscode-helm",
    "tim-koehler.helm-intellisense",
    "tomoki1207.pdf",
    "visualstudioexptteam.intellicode-api-usage-examples",
    "visualstudioexptteam.vscodeintellicode",
    "vscode-icons-team.vscode-icons",
    "waderyan.gitblame",
    "wholroyd.jinja",
    "xaver.clang-format",
    "xmtt.go-mod-grapher",
    "yzhang.markdown-all-in-one",
    "ziyasal.vscode-open-in-github",
    "zxh404.vscode-proto3"
  ],
  "icon": "images/icon.png",
  "bugs": {
    "url": "https://github.com/serialt/sugar-extension-pack/issues"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/serialt/sugar-extension-pack"
  }
}
```

```shell
# 安装 vsce
npm install -g vsce

# 打包（打包不是必须的）
vsce package

# 发布
vsce publish
```

发布的时候需要先登陆 VSCode Marketplace，创建一个publisher，修改项目package.json里的publisher同创建的相同，切换到Azure创建一个token

```shell
# 先登陆
vsce login your-new-publisher-id

# 然后发布
vsce publish
```



