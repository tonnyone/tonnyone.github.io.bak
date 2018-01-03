---
title: 你所不知道的script标签
---
[TOC]

# 加载顺序

- 放在head 头部 ?
- 放在body 底部 ?

放在头部: 此时操作dom 会出错，因为dom 还没有加载出来，head里面的脚本全部加载后dom才开始加载
放在body底部: DOM加载> 顺序执行script 里面的代码
所以大多数情况我们是把js放到底部,可以在dom加载后直接操作dom,或者用事件的方式window.onload

# defer 属性

# sync 属性
