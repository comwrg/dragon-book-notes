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

term -> term * digit
      | term / digit
      | digit
```
```
为什么不写成这样
expr -> term + expr
      | term - expr
      | term

term -> term * digit
      | term / digit
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

### 2.4.5
消除左递归公式
```
A -> Aα | β
      ||
A -> βR
R -> αR | ε
```
<details><summary>尝试消除加减乘除左递归</summary>

```
expr -> expr + term
      | expr - term
      | term

term -> term * digit
      | term / digit
      | digit

对于expr
A = expr
α = + term | - term
β = term
   ||
expr      -> term expr_tail
expr_tail -> + term expr_tail | - term expr_tail | ε

对于term
A = term
α = * digit | / digit
β = digit

term      -> digit term_tail
term_tail -> * digit term_tail | / digit term_tail | ε


最后合并
expr      -> term expr_tail
expr_tail -> + term expr_tail 
           | - term expr_tail
           | ε
term      -> digit term_tail
term_tail -> * digit term_tail
           | / digit term_tail 
           | ε
```
</details>

### 2.7
<details><summary>变量作用域实现细节</summary>
 
定义：符号表类`Symbol` 全局符号表`global_symbol` 一个全局符号表指针`p`   
下面时伪代码   
每当进入函数时，`push p; p = global_symbol;`   
每当退出函数时，`pop p;`   
每当进入块时， `new_symbol = new Symbol(); new_symbol.prev = p; p = new_symbol;`   
每当退出块时, `p = p.prev;`   

如果进入的块同时是函数， 先执行`进入函数`， 再执行`进入块`   
如果退出的块同时是函数， 先执行`退出块`， 再执行`退出函数`  

查找符号
```
for (pp = p; pp != NULL; pp = pp.prev) {
    // 然后从符号表里查找即可
}
```

</details>
