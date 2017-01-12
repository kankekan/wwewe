---
title: 小书匠更新说明
tags: 更新说明,小书匠
grammar_wavedrom: true
grammar_decorate: true
---

1. [1.12.0](#1.12.0)
	1. [1.12.0 新功能](#1.12.0%20%E6%96%B0%E5%8A%9F%E8%83%BD)
2. [1.12.0 修改](#1.12.0%20%E4%BF%AE%E6%94%B9)
3. [详细介绍](#%E8%AF%A6%E7%BB%86%E4%BB%8B%E7%BB%8D)
	1. [wavedrom 示例](#wavedrom%20%E7%A4%BA%E4%BE%8B)
	2. [添加注释方式html属性自定义语法](#%E6%B7%BB%E5%8A%A0%E6%B3%A8%E9%87%8A%E6%96%B9%E5%BC%8Fhtml%E5%B1%9E%E6%80%A7%E8%87%AA%E5%AE%9A%E4%B9%89%E8%AF%AD%E6%B3%95%3C!--%20%7B%20style%3D%22font-size%3A%2060px%3Bcolor%3Ared%3Bline-height%3A60px%3B%22%7D%20--%3E)
	3. [codemirror 英文拼写检查](#codemirror%20%E8%8B%B1%E6%96%87%E6%8B%BC%E5%86%99%E6%A3%80%E6%9F%A5)

[toc]

# 1.12.0

### 1.12.0 新功能

1. 新增自定义图床功能 [详细使用][1]
2. 新增图床迁移功能
3. codemirror 编辑器实现图片和 todo 即时预览
默认为打开。通过 `设置>编辑器>codemirror` 里的 `自动预览图片` 和 `自动预览checkbox` 两个选项，打开或者关闭。用户也可以通过临时快捷键 <kbd><kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>[</kbd></kbd> 来关闭或者打开自动预览功能
4. 新增 nunjuck 模板语言语法 [详细说明][2]
5. 新增 html 属性自定义语法
6. 新增注释方式 html 属性自定义语法
7. 新增 codemirror 英文拼写检查功能 #266
8. 新增 wavedrom 语法 [详细语法][3]
9. 新增快速插入当前时间按钮（快捷键 <kbd><kbd>Ctrl</kbd> + <kbd>.</kbd></kbd>）
10. 新增 codeChunks 语法支持 [详细说明][4]


# 1.12.0 修改

1. evernote/印象笔记提供是否锁定同步文档选项（在绑定页面进行配置）#292
2. evernote/印象笔记/有道笔记/为知笔记提供同步时是否转换网络图片选项（在绑定页面进行配置）
3. 重构绑定界面，提供图床绑定
4. 拖拽文本文件到编辑器行为修改。“做为当前文件的附件”，“创建新的文件”和 “将文本文件的内容追加到鼠标位置”这三种行为，系统默认为 “做为当前文件的附件”，可以到 `设置>编辑器` 里进行修改
5. 客户端发送邮件提供一个通过小书匠直接发送的帐号选项
6. 升级 semantic 到 2.2.6
7. 升级 sequence 到 2.0.1
8. 修复 同步到印象笔记，安卓客户端查看格式有问题 #293
9. 修复图片附件重名被覆盖问题 #290
10. 修复 表格自动刷新与 "自定义Class"功能冲突 #281
11. 修复 上边的工具菜单总是消失 #279
12. 修复 新文件保存github时路径名称比较奇怪 #274
13. 重构大纲，实现自动跟随定位
14. 导出时，如果没有相应的资源，直接显示错误，而不是预览里重构的错误提示
15. 导出 zip 时提供是否抓取网络图片
16. 修正为知保存时间
17. 重构统计文字状态栏，添加光标当前位置，及选中范围长度等信息
18. 重构 checkbox 生成的 html 结构
19. codemirror 引入 emoji 快捷输入提示键 <kbd><kbd>Ctrl</kbd> + <kbd>/</kbd></kbd>
20. 升级 markdown-it 引擎到 8.1.0
21. 解决 window 客户端路径问题
22. 修复本地绑定如果没有提前创建相关文件夹报错的问题
23. 修改 solarized_light 主题，并设置为默认主题
24. 调整错误日志输出文件
25. 调整生成的代码块 html 结构
26. 关于页面重构


# 详细介绍


## wavedrom 示例
```wavedrom!
{ signal: [
  { name: 'A', wave: '01..0..',  node: '.a..e..' },
  { name: 'B', wave: '0.1..0.',  node: '..b..d.', phase:0.5 },
  { name: 'C', wave: '0..1..0',  node: '...c..f' },
  {                              node: '...g..h' }
  ],
  edge: [
    'b-|a t1', 'a-|c t2', 'b-|-c t3', 'c-|->e t4', 'e-|>f more text',
    'e|->d t6', 'c-g', 'f-h', 'g<->h 3 ms'
  ]
}
```

``` wavedrom!
{ signal: [{ name: "A", wave: "01.zx=ud.23.45" }] }

```

``` wavedrom!
{ signal: [
  { name: 'A', wave: '01........0....',  node: '.a........j' },
  { name: 'B', wave: '0.1.......0.1..',  node: '..b.......i' },
  { name: 'C', wave: '0..1....0...1..',  node: '...c....h..' },
  { name: 'D', wave: '0...1..0.....1.',  node: '....d..g...' },
  { name: 'E', wave: '0....10.......1',  node: '.....ef....' }
  ],
  edge: [
    'a~b t1', 'c-~a t2', 'c-~>d time 3', 'd~-e',
    'e~>f', 'f->g', 'g-~>h', 'h~>i some text', 'h~->j'
  ]
}
```


##  添加注释方式html属性自定义语法<!-- { style="font-size: 60px;color:red;line-height:60px;"} -->

```
grammar_decorate: true

## 标题 <!-- { style="font-size: 60px;"} -->
```

## codemirror 英文拼写检查

错误的英文单词 goood
正确的英文单词 good


  [1]: https://github.com/suziwen/blogxiaoshujiang/blob/master/2016-12-27%20%E5%B0%8F%E4%B9%A6%E5%8C%A0%E5%9B%BE%E5%BA%8A%E4%BD%BF%E7%94%A8.md
  [2]: https://github.com/suziwen/blogxiaoshujiang/blob/master/2016-12-28%20%E5%B0%8F%E4%B9%A6%E5%8C%A0%20nunjucks%20%E6%A8%A1%E6%9D%BF%E6%A0%87%E7%AD%BE%E8%AF%AD%E6%B3%95%E6%89%8B%E5%86%8C.md
  [3]: http://wavedrom.com/tutorial.html
  [4]: https://github.com/suziwen/blogxiaoshujiang/blob/master/2016-12-28%20%E5%B0%8F%E4%B9%A6%E5%8C%A0%20code%20chunks%20%E4%BB%A3%E7%A0%81%E5%9D%97%E8%AF%AD%E6%B3%95.md
