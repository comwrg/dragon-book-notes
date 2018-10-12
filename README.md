# dragon-book-notes
编译原理第二版笔记

## 第 2 章

### 2.2

<details><summary>上下文无关文法 构建一些想法</summary>

```
加减乘除无括号的表达式
expr -> expr + term
      | expr - term
      | term

term -> term * factor
      | term / factor
      | digit
```
```
为什么不写成这样
expr -> term + expr
      | term - expr
      | term

term -> term * factor
      | term / factor
      | digit

+ - 是左结合
这样的话 + - 就变成里右结合
1 + 2 + 3
语法分析树就会变成这样
         expr
        / | \
       1  +  expr
            / | \
           2  +  3
```
```
或者这样
expr -> expr + digit
      | expr - digit
      | expr * digit
      | expr / digit
也就是说优先级越高的符号， 要越往里（下）写
表达式的本质就是一颗树， 肯定是从下往上算， 所以要往里（下）写
```
</details>
