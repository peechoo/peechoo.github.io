---
title: "[Scala] 字符串插值器忽略双引号转义字符"
date: 2021-03-19
author: zyk
---

## 问题来源

用 `\"` 在字符串中插入双引号，遇到报错：

[<img src="https://s3.ax1x.com/2021/03/19/6RbNM4.png" alt="vscode snap" style="zoom: 75%;" />](https://imgtu.com/i/6RbNM4)

## tl;dr

它不是 bug ，它是 **feature** ！（至少是[明确 won't fix](https://groups.google.com/d/topic/scala-sips/d2K23f__6b0/discussion) 的 bug）

正确的插入方式：

```scala
val color = "#0000ff"
s"""color="$color""""
// or
s"color=${'"'}$color${'"'}"
```

以上2个表达式的值均为 `color="#0000ff"` 。

当然，还可以用 `+` 运算符拼接多个字符串（如 `"color=\"" + color + "\""` ），这种解决方案不在本文的讨论范围内。

## 类似情况

（下面的例子来自 GitHub issue ）

```scala
scala> val a="42"
a: String = 42

scala> "a=\"$a\""
res33: String = a="$a"    // ok

scala> s"a=\"$a\""
<console>:1: error: ';' expected but string literal found.
       s"a=\"$a\""
                ^         // oops

scala> s"a=\'$a\'"
res34: String = a='42'    // ok with \'

scala> """a=\"$a\""""
res35: String = a=\"$a\"  // escape not interpreted, ok

scala> s"""a=\"$a\""""
res36: String = a="42"    // escape interpreted, should it ?
```

**总结**：

1. 在不用插值器的时候，`\"` 可以正常被转换为双引号。
2. 使用了 `s` 插值器之后，再使用 `\"` ，编译器就会报错。
3. 单引号 `\'` 可以在插值器中使用。
4. 在使用 `"""` 声明的字符串中，可以直接使用 `"` ， `\"` 会被识别为2个字符。

> **冷知识**：
>
> - 形如 `"..."` 的字符串字面量（加上转义字符）可以表示所有的字符串
> - 形如 `"""..."""` 的字符串不能表示 `"""` 
> - 使用插值器之后，形如 `s"..."` 的字符串和形如 `s"""..."""` 的字符串都无法表示所有的字符串

## 详细解释

在 [SIP-11](https://docs.scala-lang.org/sips/string-interpolation.html) 中， `id"string content"` 这样的字符串被称为 *processed string* （准确地说， string content 可以是 standard - `"` 类型，也可以是 multi-line - `"""` 类型）。

按照上面手册中的定义，编译器在遇到

```scala
id"text_0${expr_1}text_1...${expr_n}text_n"
// or
id"""text_0${expr_1}text_1...${expr_n}text_n"""
```

时，会将其替换为

```scala
StringContext("""text_0""", ..., """text_n""").id(expr_1, ..., expr_n)
```

SIP-11 中提到：

> Inside a processed literal **none** of the usual escape characters are interpreted (except for unicode escapes) no matter whether the string literal is normal (enclosed in single quotes) or multi-line (enclosed in triple quotes). Instead, there is are two new forms of dollar sign escape. The most general form encloses an expression in `${` and `}`, i.e. `${expr}`.

大意是说， processed literal 中除 unicode 转义字符外，其余所有的转义字符都不会被替换。另外，这里的 `${` 和 `}` 可以视为新的转义字符。

SIP-11 在 Standard String Interpolation 一节还提到，字符串插值器 `s` 返回的字符串为

```scala
// scala.StringContext("""text_0""", ..., """text_n""").s(expr_1, ..., expr_n)
esc("""text_0""") + (expr_1) + esc("""text_1""") + ... + (expr_n) + esc("""text_n""")
```

此处的 `esc(str)` 代表将字符串 `str` 中的转义字符替换为对应的符号。

所以，如果 `esc` 在这里把 `\"` 替换为 `"` 的话，替换后的代码中可能出现语法错误，比如

```scala
val a = 10
s" \"\"\" $a" // expected to be string """10

// then
esc(""" \"\"\" """) + (10)
// and then
""" """ """ + 10
// something bad could happen
```

在手册的定义中， `StringContext` 接受的字符串不含转义字符，插值器需要自己进行转义字符替换的工作。这样定义的好处是，可以实现一个这样的正则表达式插值器：它不识别转义字符，从而书写正则表达式时不必考虑诸如 `'` 需要写为 `\'` 的问题。同时，标准的插值器 `s` 和 `f` 等可以正常处理转义字符。

## 参考资料

- [Stack Overflow](https://stackoverflow.com/) 上的提问 [How to insert double quotes into String with interpolation in scala](https://stackoverflow.com/questions/21086263/how-to-insert-double-quotes-into-string-with-interpolation-in-scala)
- [scala/bug](scala/bug) 仓库里的 [issue#6476](https://github.com/scala/bug/issues/6476)
- Google Groups 里的会话 [round up the usual escape characters](https://groups.google.com/g/scala-sips/c/d2K23f__6b0/discussion)
- Scala Doc on [STRING INTERPOLATION](https://docs.scala-lang.org/overviews/core/string-interpolation.html)
- [SIP 11](https://docs.scala-lang.org/sips/string-interpolation.html)