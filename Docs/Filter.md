# Filter方法

## 简介
Filter是Pathfinder为string类型提供的扩展方法

## 用处讲解

我们会碰到把字符串写入文件时自替换字符不会被替换的问题

因为HN只会自动替换扩展XML的自替换字符

而Filter用于把Hacknet中的自替换字符替换

解决了自体换问题

## 实战

使用这个方法非常简单，只需要字符串.Filter就行了

示例：
```CSharp
"#SSH_CRACK".Filter()
```