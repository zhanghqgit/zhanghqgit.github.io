---
title: markdown-syntax markdown 语法示例 （一）
date: 2016-09-07 21:43:26
tags: markdown
categories: markdown
---

# H1,两个#就是H2,以此类推
```
# Markdown
## Markdown
```

### 粗体和斜体

**强调** **还可以和_斜体_混合使用**
**有的时候你可能需要很长一段文字都是粗体，可以直接换行
就像现在这样**
```
强调，使用两个*号或者双下划线_
斜体，使用一个*号或者单下划线_
```

### 引用
> 右扩折号 &gt; 用来当做引用标记，`注意`：如果要使用源生&gt;,就需要使用转义符号`&gt;`

```
> 右扩折号 &gt; 用来当做引用标记，`注意`：如果要使用源生&gt;,就需要使用转义符号`&gt;`
```

### 链接和邮箱

简单的邮箱<example@example.com>链接.

简单的行链接 <https://zhanghqgit.github.io>.

带名称的行链接 [张恒强](https://zhanghqgit.github.io)

带提示的多行链接[张恒强](https://zhanghqgit.github.io 
"张恒强的简单博客")

引用链接：[reference style][id].`id`可以在文档的任何地方定义引用的链接地址
[id]: https://zhanghqgit.github.io "张恒强的简单博客"

定义引用链接的地址时，链接的提示是可选的

格式： `[链接名称](链接地址 链接提示(提示是可选的))`
```
简单的邮箱<example@example.com>链接

简单的行链接 <https://zhanghqgit.github.io>.

带名称的行链接 [张恒强](https://zhanghqgit.github.io)

带提示的多行链接 [张恒强](https://zhanghqgit.github.io 
"张恒强的简单博客")

引用链接:[reference style][id].`id`可以在文档的任何地方定义引用的链接地址
[id]: https://zhanghqgit.github.io "张恒强的简单博客"

定义引用链接的地址时，链接的提示是可选的

```

### 图片

简单定义图片 ![GitHub](https://avatars0.githubusercontent.com/u/7802877?v=3&amp;s=34 "这里是提示")
提示是可选的

图片的源也可以使用引用，和链接引用一样![GitHub][2]
[2]: https://avatars0.githubusercontent.com/u/7802877?v=3&amp;s=34 "提示"

格式：`![图片名称](图片src 图片提示)`
```
简单定义图片 ![GitHub](https://avatars0.githubusercontent.com/u/7802877?v=3&amp;s=34 "这里是提示")
提示是可选的

图片的源也可以使用引用，和链接引用一样![GitHub][2]

[2]: https://avatars0.githubusercontent.com/u/7802877?v=3&amp;s=34 "提示"
```

### 行代码及块代码

`行代码`是用英文的反引号括起来.

	块代码的每一行都使用至少一个Tab或者4个空格的缩进
	alert('Hello World!')

```
`行代码`是使用英文的反引号括起来.

****快代码的每一行都使用至少一个Tab或者4个空格的缩进
****alert('Hello World!')
`注意`:此处使用四个*代替了Tab或者4个空格
```

### 有序列表

有序列表使用数字 "1.",`注意`: 后面一定要加一个空格，且数字的大小不代表列表项目的顺序

1. 有序列表1
3. 有序列表3
2. 有序列表2

```
有序列表使用数字 "1.",`注意`:后面一定要加一个空格，切腹数字的大小不代表列表项目的顺序

1. 有序列表1
3. 有序列表3
2. 有序列表2
```

### 无序列表

无序列表使用 "*" 或者 "-",`注意`:同有序列表一样，都需要再后面加一个空格

* 无序列表1
* 无序列表2
* 无序列表3
- 无序列表4
- 无序列表5
- 无序列表6

```
无序列表使用 "*" 或者 "-",`注意`:同有序列表一样，都需要再后面加一个空格
* 无序列表1
* 无序列表2
* 无序列表3
- 无序列表4
- 无序列表5
- 无序列表6

```
