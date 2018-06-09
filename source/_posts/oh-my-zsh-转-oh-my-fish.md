---
title: oh my zsh 转 oh my fish
date: 2018-06-09 18:07:34
header-img: "post_header.jpg"
tags:
    - shell
    - 工具
categories:
    - 技术
---

刚开始接触`shell`的时候, 同事告诉我有个比系统默认`bash`好用的`shell`叫`zsh`, 有很多强大方便的功能, 而且`zsh`是Mac预装的, 加上有`oh-my-zsh`这个开源项目, 使得zsh的配置难度降低了, 完全兼容`bash`。后来当我装上`oh my zsh` 之后, 确实用起来很强大, 但不知道为何在我的Mac上使用起来特别迟钝，粘贴一个`curl`串要等7-8秒才能反应过来，这果断不能忍。疑惑的是在同事Mac确实很快，有可能也是自己配置哪块出了点小问题，上网搜索了一番，发现还有个`shell`叫`fish`,用的人也很多, 也有类似的`oh-my-fish`项目配置，感觉上是比zsh轻量级。这就有了我从`oh my zsh` 转到 `oh my fish`的一番经历!

### 查看系统当前shell
```
$cat /etc/shells
```

### 卸载oh my zsh
```
$ uinstall_oh_my_zsh
```

### 安装fish shell
```
brew install fish
```

### 切换默认shell为fish
```
chsh -s /usr/local/bin/fish

```

这里你可能会遇到和我类似的报错

```
chsh: /usr/local/bin/fish: non-standard shell

``` 

但是直接运行`/usr/local/bin/fish`却是正常的。

解决方案: `/etc/shells` 文件中, 包含了 `local shell` 的列表, 应用程序通过该文件来判断一个`shell`是不是合法。我的系统里该文件内容如下:

``` 
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh

```

因此，必须先将 `/usr/local/bin/fish` 加入到此文件中，然后才能使用 `chsh` 命令将其设置为默认shell

### 安装 oh my fish
```
 curl -L https://github.com/oh-my-fish/oh-my-fish/raw/master/bin/install | fish
 
```

### 安装Agnoster主题(其他主题自行Google)
```
omf install agnoster

```

总结: `fish`确实比`zsh`更强大, 使用起来很顺畅!




