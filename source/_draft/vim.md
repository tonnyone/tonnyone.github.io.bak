## vim 配置

[推荐vim 配置文件](https://github.com/amix/vimrc)

### vim 自身快捷键

### vim 中执行shell命令

```bash
:!            :!{command}     execute {command} with a shell
```

### buffer 操作

`:ls`, `:buffers`       列出所有缓冲区
`new {file}` 水平分割一个窗口
`vnew {file}` 垂直分割一个窗口打开

`b {num}` 全窗口打开buffer
`sb {num}` 分割窗口打开,这里是水平分割

`:ls` 显示所有buffer区的文件
`:bdelete num`  `:bd num` 删除一个buffer区的文件序号为num
`:3,5bd` 删除文件序号3,5的buffer
`Ctrl + ^` buffer区切换

```bash
:bn[ext]            下一个缓冲区
:bp[revious]        上一个缓冲区
:b {number, expression}     跳转到指定缓冲区
:bfirst
:brewind
:sbfirst
:sbrewind
:bnext
:sbnext
:bprevious
:bNext
:sbprevious
:sbNext
:blast
:sblast
:ball // 打开所有
:sball// 打开所有
```

### 窗口

> A buffer is the in-memory text of a file.
> A window is a viewport on a buffer.
> A tab page is a collection of windows.

`vim -o file1 file2 file3` 水平分割窗口(默认)
`vim -O file1 file2 file3` 垂直分割窗口

1. Ctrl+w , 加 h,j,k,l 窗口操作
2. Ctrl+w + w ,多窗口切换
3. 水平分割出多个窗口 `Ctrl+w s` 和 `Ctrl+w S` 或者 `:sp` `:split`
4. 垂直分割出多个窗口 `Ctrl+w v` 和 `Ctrl+w V` 或者 `:vsp` `:vsplit`
5. 水平新开一个空的窗口 `Ctrl+w n` 和 `Ctrl+w Ctrl+n` 或者 `:new` `:vsplit`
6. 垂直新开一个空的窗口 `:vne` 或者 `:vnew`
7. 关闭一个窗口 `Ctrl+w q` 或者 `Ctrl+w Ctrl+q`
8. 窗口调整大小 `Ctrl+w +/-` 或者 `Ctrl+w Ctrl+q`
9. 窗口调整位置 `Ctrl+w r` 或者 `Ctrl+w R` 或者 `Ctrl+w x` 或者 `Ctrl+w X`
  `Ctrl+w K` move current to very top
  `Ctrl+w J` move current to very bottom
  `Ctrl+w H` move current to very left 
  `Ctrl+w L` move current to very right
  `Ctrl+w T` move current to new tab

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

#### nerdtree

配置我使用的配置文件切换方式,我使用的leader 是 `,`号

```bash
 map <leader>nn :NERDTreeToggle<cr>
 map <leader>nb :NERDTreeFromBookmark<Space>
 map <leader>nf :NERDTreeFind<cr>
```

## 参考链接

- [wimhelp_quickref](http://vimhelp.appspot.com/quickref.txt.html)
- [why-do-vim-experts-prefer-buffers-over-tabs](https://stackoverflow.com/questions/26708822/why-do-vim-experts-prefer-buffers-over-tabs)
- [vim 上古神器vim插件：你真的学会用NERDTree了吗？](https://www.jianshu.com/p/3066b3191cb1)
- [Vim 多文件编辑：缓冲区](http://harttle.land/2015/11/17/vim-buffer.html)
- [vim实用技巧之高效的buffer操作]( http://fishcried.com/2014-10-25/vim%E5%AE%9E%E7%94%A8%E6%8A%80%E5%B7%A7%E4%B9%8B%E9%AB%98%E6%95%88%E7%9A%84buffer%E6%93%8D%E4%BD%9C/)
- [vim 中文doc](http://vimcdoc.sourceforge.net/doc/)
