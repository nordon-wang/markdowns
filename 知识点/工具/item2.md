## Iterm

### 查看 Mac 上已有的 shell

```shell
cat /etc/shells

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

### 将默认 shell 改成 zsh

```shell
chsh -s /bin/zsh
```

### 安装 oh my zsh
安装 Oh My Zsh 方法
可以通过 curl 或 wget 两种方式来安装，用一条命令即可安装。
curl 安装
GitHub:
```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

Gitee ( 国内镜像 )
```
sh -c "$(curl -fsSL https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)"
```
wget 安装
GitHub:
```
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```
Gitee ( 国内镜像 )
```
sh -c "$(wget -O- https://gitee.com/pocmon/mirrors/raw/master/tools/install.sh)"
```


### 配置 zsh

```shell
git clone git://github.com/altercation/solarized.git
# 然后打开 solarized/osx-terminal.app-colors-solarized/Solarized Dark ansi.terminal 配置文件
```

### 安装 Homebrew

[mac下镜像飞速安装Homebrew教程](https://zhuanlan.zhihu.com/p/90508170)

### 安装 autojump 插件

```shell
brew install autojump # 确保有 brew 命令
```

编辑～/.zshrc 文件

```shell
vim ~/.zshrc

plugins=(git brew laravel5 autojump)
[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh
```

### 安装 zsh-autosuggestions 插件

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

编辑～/.zshrc 文件

```shell
vim ~/.zshrc
plugins=(git brew laravel5 autojump zsh-autosuggestions)
```

### 安装 zsh-syntax-highlighting 插件

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

编辑～/.zshrc 文件

```shell
vim ~/.zshrc

ZSH_THEME="ys"
plugins=(git brew laravel5 autojump zsh-autosuggestions zsh-syntax-highlighting)
```

为了使全部的更改生效

```shell
source ~/.zshrc
```

### shell终端的前缀信息

修改~/.oh-my-zsh/themes下的主题文件， 之前设置的是`ZSH_THEME="ys"`，找到对应的`ys.zsh-theme`文件进行修改

```shell
PROMPT="
%{$terminfo[bold]$fg[blue]%}#%{$reset_color%} \
%{$terminfo[bold]$fg[yellow]%}%~%{$reset_color%}\
${hg_info}\
${git_info}\
 \
$exit_code
%{$terminfo[bold]$fg[red]%}$ %{$reset_color%}"

```



