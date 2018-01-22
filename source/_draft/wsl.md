# wsl环境配置

## zsh 配置

### 安装

```bash
apt-get install zsh
# 安装oh my zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### 常用的shortcut

1. 直接输入d 列出之前所有的路径

## vim 配置

[vim 配置文件](https://github.com/amix/vimrc)

### vim 自身快捷键

#### 窗口

1. Ctrl+w , 加 h,j,k,l 窗口操作

#### buffers

1. buffers操作

```bash
:ls, :buffers       列出所有缓冲区
:bn[ext]            下一个缓冲区
:bp[revious]        上一个缓冲区
:b {number, expression}     跳转到指定缓冲区
```

### remap

```bash
 " Quickly open a buffer for scribble
 map <leader>q :e ~/buffer<cr>
 " Quickly open a markdown buffer for scribble
 map <leader>x :e ~/buffer.md<cr>
 " Toggle paste mode on and off
 map <leader>pp :setlocal paste!<cr>
 ```

### vim 插件

 [vim 上古神器vim插件：你真的学会用NERDTree了吗？](https://www.jianshu.com/p/3066b3191cb1)

#### nerdtree

配置我使用的配置文件切换方式,我使用的leader 是 `,`号

```bash
 map <leader>nn :NERDTreeToggle<cr>
 map <leader>nb :NERDTreeFromBookmark<Space>
 map <leader>nf :NERDTreeFind<cr>
```

## 参考链接

- [Vim 多文件编辑：缓冲区](http://harttle.land/2015/11/17/vim-buffer.html)
- [vim实用技巧之高效的buffer操作]( http://fishcried.com/2014-10-25/vim%E5%AE%9E%E7%94%A8%E6%8A%80%E5%B7%A7%E4%B9%8B%E9%AB%98%E6%95%88%E7%9A%84buffer%E6%93%8D%E4%BD%9C/)