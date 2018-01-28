# wsl环境配置

## zsh 配置

### 安装

```bash
apt-get install zsh
# 安装oh my zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

优化

`git config --global oh-my-zsh.hide-status` 1 关闭对status的检查，
然后disable auto upgrade To disable automatic upgrades, set the following in your ~/.zshrc:DISABLE_AUTO_UPDATE=true

安装autojump

```bash
apt-get install autojump
```

### 常用的shortcut

1. 直接输入d 列出之前所有的路径

## 参考链接

- [wsl-guide](http://wsl-guide.org/en/latest/update.html)
- [Running Node.js on WSL from Visual Studio Code](https://blogs.msdn.microsoft.com/commandline/2017/10/27/running-node-js-on-wsl-from-visual-studio-code/)
- [how to i user ubuntu on window wsl form vs code](https://stackoverflow.com/questions/44450218/how-do-i-use-bash-on-ubuntu-on-windows-wsl-for-my-vs-code-terminal)
- [Run oh my zsh as integrated shell in VSCode on Windows](https://winsmarts.com/run-oh-my-zsh-as-integrated-shell-in-vscode-on-windows-7d69f72bafa3)
-[](https://www.hanselman.com/blog/SettingUpAShinyDevelopmentEnvironmentWithinLinuxOnWindows10.aspx)
-[](http://blog.questionable.services/article/windows-subsystem-linux-zsh-tmux-docker/)
