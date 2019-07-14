---
title: git commit 规范工具的应用
date: 2019-07-14 16:27:01
tags: 工程化
categories: 前端技术
comments: true
---

## git commit 规范

### Angular 提交规范

编码需要规范，git 提交代码也应该有其规范，业内普遍参考 Angular 提交的规范。下面来介绍 Angular 的提交规范：

```
<type>(<scope>): <subject> #header
// 空一行
<body>
// 空一行
<footer> 
```
从格式上看，整个 message 分为了 三个部分：
- header: 整个 header 部分尽量不要这行。它包含三个字段：type（必需）、scope（可选）和 subject(必需)。
    - type
        - feat: 新功能 feature
        - fix: 修补 bug
        - docs: 文档 documentation
        - style: 格式——不影响代码运行的变动
        - refactor: 重构——既不是新增功能，也不是修改 bug 的代码变动
        - test: 增加测试
        - chore: 构建多次或辅助工具的变动
        - revert: 用于撤销以前的 commit，有着特定的格式：
            - header: `rever: <被撤销的header>`
            - body: `this reverts commit <被撤销 commit 的 SHA 标识符>`
        ```
        revert: feat(pencil): add 'graphiteWidth' option

        This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
        ```
    - scope: 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。
    - subject: 是对 commit 目的的简短描述
        - 以动词开头，使用第一人称现在时，比如 change，而不是 changed 或 changes
        - 第一个字母小写
        - 结尾不加句号
- body: 是对 commit 的详细描述，可以分成多行
    - 使用第一人称现在时
    - 应该说明代码变动的动机，以及与之前行为的对比
```
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. 

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
```
- footer: 通常只用于两种情况
    - 不兼容变动——如果当前代码与上一个版本不兼容，则 footer 部分以 `BREAKING CHANGE` 开头，后面是对变动的描述，以及变动理由和迁移方法
    ```
    BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
    ```
    - 关闭 issue——如果当前 commit 针对某个 issue，那么可以在 footer 部分关闭这个issue
    ```
    Closes #123, #245, #992
    ```

### 规范的意义

这里说说规范的好处，看看是否能够击中你工作中的痛点：

1. 提供更多的历史信息，方便快速浏览。每个 commit 占据一行，只看行首，就知道其目的。
2. 可以过滤某些 commit（比如文档改动），便于快速查找信息
3. 可以直接从 commit 生产 change log。change log 是发布新版本时，用来说明与上一次版本差异的文档

### 最佳实践

1. change log 中，最后只包括 type、feat、fix 类型，其他（docs、chore、style、refactor、test）建议不添加。change log 作为说明与上一个版本差异的文档，其主要受众是产品和测试。
2. revert commit 与 被撤销的 commit，在同一个发布版本（release）中，那么他们都不会出现在 chang log 中。如果两者在不同的发布版本，那么 revert commit 应该出现。

## git commit 工具

### Commitizen

[Commitizen](https://github.com/commitizen/cz-cli) 是一个用来撰写合格的 commit message 的工具。

安装：
```
npm install -g commitizen
```

### cz-conventional-changelog 

cz-conventional-changelog(Angular 规范) 用于为 Commitizen 提供 commit message 格式规范的 npm 包

其概念就是一个规范 [Adapters](https://github.com/commitizen/cz-cli#adapters)

安装：
```
npm install -g cz-conventional-changelog
```
通过 package.json 的配置引入 Commitizen：
```
// 配置在 package.json 的根节点
"config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }

// 或者通过 .czrc 配置文件进行配置
// .czrc
{ "path": "cz-conventional-changelog" }
```

### git-cz
git-cz 是 commitizen 的命令行工具，有了这个 npm 包，才能使用命令调出交互界面，方便快捷的生成合格的commit message。

安装：
```
npm install -g git-cz
git-cz

// npm 版本要求 5.2+
npx git-cz
```

运行命令可看到如下界面：
```
git-cz
// or
git cz
// or
npx git-cz
```
![commitizen_1](https://user-images.githubusercontent.com/33422051/59660560-7945e580-91db-11e9-8e4a-c58470549e09.png)

### 总结

- 配置 Commitizen 需要三个 npm 包：Commitizen、cz-conventional-changelog、git-cz。
- commitizen 有两个配置文件
    - .czrc
    ```
    // .czrc
    { "path": "cz-conventional-changelog" }
    ```
    - changelog.config.js
    ```
    // changelog.config.js
    module.exports = {
        "list": [
            "test",
            "feat",
            "fix",
            "chore",
            "docs",
            "refactor",
            "style",
            "ci",
            "perf"
        ],
        "maxMessageLength": 64,
        "minMessageLength": 3,
        "questions": [
            "type",
            "scope",
            "subject",
            "body",
            "breaking",
            "issues",
            "lerna"
        ],
        "scopes": [],
        "types": {
            "chore": {
            "description": "Build process or auxiliary tool changes",
            "emoji": "🤖",
            "value": "chore"
            },
            "ci": {
            "description": "CI related changes",
            "emoji": "🎡",
            "value": "ci"
            },
            "docs": {
            "description": "Documentation only changes",
            "emoji": "✏️",
            "value": "docs"
            },
            "feat": {
            "description": "A new feature",
            "emoji": "🎸",
            "value": "feat"
            },
            "fix": {
            "description": "A bug fix",
            "emoji": "🐛",
            "value": "fix"
            },
            "perf": {
            "description": "A code change that improves performance",
            "emoji": "⚡️",
            "value": "perf"
            },
            "refactor": {
            "description": "A code change that neither fixes a bug or adds a feature",
            "emoji": "💡",
            "value": "refactor"
            },
            "release": {
            "description": "Create a release commit",
            "emoji": "🏹",
            "value": "release"
            },
            "style": {
            "description": "Markup, white-space, formatting, missing semi-colons...",
            "emoji": "💄",
            "value": "style"
            },
            "test": {
            "description": "Adding missing tests",
            "emoji": "💍",
            "value": "test"
            }
        }
    };
    ```
    - npx 相关配置，请移步[官网](https://github.com/commitizen/cz-cli)
    - 为公司多个项目统一规范，可生成自己的 Commitizen npm 包，具体操作参加[commitizen for multi-repo projects](https://github.com/commitizen/cz-cli#commitizen-for-multi-repo-projects)

## 校验 commit 是否符合规范
https://conventional-changelog.github.io/commitlint/#/

## 生成 change log
https://github.com/conventional-changelog/conventional-changelog/tree/master/packages/conventional-changelog-cli
