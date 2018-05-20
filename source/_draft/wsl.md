# wsl环境配置

## zsh 配置

### 安装

```bash
apt-get install zsh
# 安装oh my zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

## 优化

`git config --global oh-my-zsh.hide-status` 1 关闭对status的检查，
然后disable auto upgrade To disable automatic upgrades, set the following in your ~/.zshrc:DISABLE_AUTO_UPDATE=true

安装autojump

```bash
apt-get install autojump
```

### 常用的shortcut

1. 直接输入d 列出之前所有的路径

## ubuntu 字体下载
(ubuntu字体)[https://design.ubuntu.com/font/] 解压复制到控制面板字体
(dircolors)[https://github.com/seebi/dircolors-solarized]

## TMUX

### session 

`tmux new-session -s basic` 等同于`tmux new -s basic`  新建session     
`tmux detach`  断开当前会话，会话在后台运行
`tmux a`  默认进入第一个会话
`tmux a -t demo`  进入到名称为demo的会话
`tmux ls` 查看会话 
`tmux kill-session -t demo` # 关闭demo会话
`tmux kill-server` # 关闭服务器，所有的会话都将关闭

`prefix $ 重命名session`

#### 命令模式
所有的命令都可以用命令的方式执行,一般用在`tmux`内部执行命令的时候如下:

'tmux new -s basic' 等同于 `ctrl+b :new -s basic`
`ctrl+b s` 查看会话,用方向键选择,回车键进入

### 窗口管理
ctrl+b c    新建窗口
Ctrl+b &   关闭当前窗口（关闭前需输入y or n确认）
Ctrl+b 0~9 切换到指定窗口
Ctrl+b p   切换到上一窗口
Ctrl+b n   切换到下一窗口
Ctrl+b w   打开窗口列表，用于且切换窗口
Ctrl+b ,   重命名当前窗口
Ctrl+b .   修改当前窗口编号（适用于窗口重新排序）
Ctrl+b f   快速定位到窗口（输入关键字匹配窗口名称）      

### pane 管理
Ctrl+b " 当前面板上下一分为二，下侧新建面板
Ctrl+b % 当前面板左右一分为二，右侧新建面板
Ctrl+b x 关闭当前面板（关闭前需输入y or n确认）
Ctrl+b z 最大化当前面板，再重复一次按键后恢复正常（v1.8版本新增）
Ctrl+b ! 将当前面板移动到新的窗口打开（原窗口中存å¨两个及以上面板有效）
Ctrl+b ; 切换到最后一次使用的面板
Ctrl+b q 显示面板编号，在编号消失前输入对应的     数字可切换到相应的面板
Ctrl+b { 前置换当前面板
Ctrl+b } 向后置换当前面板
Ctrl+b Ctrl+o 顺时针旋转当前窗中的所有面板
Ctrl+b 方向键 移动光标切换面板
Ctrl+b o 选择下一面板
Ctrl+b 空格键 在自带的面板布局中循环切换
Ctrl+b Alt+方向键 以5个单元格为单位调整当前面板边缘
Ctrl+b Ctrl+方向键 以1个单元格为单位调整当前面板边缘（Mac下被系统快捷键覆盖
Ctrl+b t 显示时钟


## 参考链接

- [wsl-guide](http://wsl-guide.org/en/latest/update.html)
- [Running Node.js on WSL from Visual Studio Code](https://blogs.msdn.microsoft.com/commandline/2017/10/27/running-node-js-on-wsl-from-visual-studio-code/)
- [how to i user ubuntu on window wsl form vs code](https://stackoverflow.com/questions/44450218/how-do-i-use-bash-on-ubuntu-on-windows-wsl-for-my-vs-code-terminal)
- [Run oh my zsh as integrated shell in VSCode on Windows](https://winsmarts.com/run-oh-my-zsh-as-integrated-shell-in-vscode-on-windows-7d69f72bafa3)
-[](https://www.hanselman.com/blog/SettingUpAShinyDevelopmentEnvironmentWithinLinuxOnWindows10.aspx)
-[](http://blog.questionable.services/article/windows-subsystem-linux-zsh-tmux-docker/)
-[Mouse mode in tmux](https://github.com/Microsoft/WSL/issues/531)
-[gnome-terminal wsl](https://www.youtube.com/watch?v=GMHxSvuXDYc)
-[tmux book](https://pityonline.gitbooks.io/tmux-productive-mouse-free-development_zh/content/book-content/Chapter1.html)
