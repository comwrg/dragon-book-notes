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

## 第 3 章

### 3.4.5
<details><summary>K（看）M（毛）P（片）算法</summary>
这个龙书的KMP算法讲的好难懂的样子， 我晕！可能大佬有大佬的思维吧 orz <br />
以前我试图想学过这个算法， 但是看了好多文章， 感觉都讲的不是很清楚， 到现在还是迷迷糊糊的 <br />
今晚， 对，就是今晚 一定要搞懂， ***， 我就不信了 <br />
细细一看龙书 - -， 哇， 别看短短几行， 讲的还是很清楚的。 <br />
<br />

唯一的难点应该是*失陪函数*的计算 <br />
需要明确*失陪函数*计算是关键字的长度，这个关键字要满足既是最长的真前缀又是后缀。 <br />
比如， 就拿龙书的例子：（龙书上的索引是从1开始）
<pre>
字符串：ababaa

f(1) 要计算的字符串：a           计算出的关键字：没有        长度：0
f(2) 要计算的字符串：ab          计算出的关键字：没有        长度：0
f(3) 要计算的字符串：aba         计算出的关键字：a          长度：1
f(4) 要计算的字符串：abab        计算出的关键字：ab         长度：2
f(5) 要计算的字符串：ababa       计算出的关键字：aba        长度：3
f(6) 要计算的字符串：ababaa      计算出的关键字：a          长度：1
</pre>

苦思冥想， 为啥龙书上的函数要这么写， 加上搜索各种资料， 终于。。。 <br/>
龙书最大的失策是直接放伪代码， 不放解释， 我晕， 下面还要证明， 没接触的直接凉凉 <br />

<pre>
失配函数的算法流程是这样的
先是从第一个字符和第二个字符开始比较， 如果相同呢那么 f(1) = 1， 不同 f(1) = 0 （不懂看上面的例子）
假设第一个字符和第二个字符相同
那么现在在第二个字符后面加一个字符（第三个字符），然后是不是就要比较第一个字符后面的哪个字符， 也就是第二个字符， 和第三个字符是否相同
这个相同的流程

那假如不同咋办，不同的话需要回退， 回退到现在的关键字中的关键字那个地方， 为什么可以这么回退呢， 因为关键字的属性就是头和尾是一样的， 画张图可能可以理解快一点
</pre>

书上伪代码，翻译成c， 如果不懂的话， 看的脑壳疼
```c
#include <stdio.h>
#include <memory.h>

int main() {
    char *b = "-ababaa"; // 龙书中索引开始是1， 所以这里头部加了一个字符
    int n = strlen(b);
    int f[n];

    int t = 0; // 指向现在的关键字
    f[1] = 0;
    for (int s = 1; s < n; s++) {
        while (t > 0 && b[s+1] != b[t+1]) // 这里是回退用的， t > 0 是说 如果 t == 0 的话， 就没法继续回退了， b[s+1] != b[t+1] 是回退的条件， 不等于才要回退
            t = f[t]; // 回退关键字， 至于为什么可以回退， 画图， 关键字中的关键字头和尾是一样的
        if (b[s+1] == b[t+1]) { // 等于， 直接下一步
            ++t; // 关键字长度加一
            f[s+1] = t; // 赋值
        } else {
            f[s+1] = 0; // 很明显， 这里肯定回退到不可以回退了， t肯定等于0， 关键字长度肯定等于0
        }
    }

    for (int s = 1; s < n; s++) { // 打印
        printf("%d ", f[s]);
    }
}
```

用索引从0开始写的代码
```c
#include <stdio.h>
#include <memory.h>

int main() {
    char *b = "ababaa";
    int n = strlen(b);
    int f[n];

    // 这里为什么用-1呢
    // 因为s和t肯定要表示同一样东西， s表示的是头部匹配的长度， t表示的是尾部匹配的长度
    // 那么开始的时候肯定是字符0和字符1开始比较
    // 那么s和t肯定在最开始的时候肯定要相差一
    // 无非s=-1， t=0， or， s=0，t=-1
    // 如果s=-1， t=0这种， 那么是s从-1到n-2循环， 下面也肯定是f[s+1] = t; 所以肯定没法写
    int t = -1;
    f[0] = -1;
    for (int s = 0; s < n-1; s++) {
        while (t > -1 && b[s+1] != b[t+1])
            t = f[t];
        if (b[s+1] == b[t+1]) {
            ++t;
            f[s+1] = t;
        } else {
            f[s+1] = -1;
        }
    }

    for (int s = 0; s < n; s++) {
        printf("%d ", f[s]+1);
    }
}
```
</details>

### 3.6 有穷自动机
<details><summary>NFA和DFA的区别</summary>

从名子里面就可以知道了，一个是确定，一个是不确定，那是什么不确定呢？ <br />
是从一个状态转移到另一个状态的路径（对于同样的输入）， NFA可能有一个或多个， 而DFA肯定只有一个。

所以DFA就是NFA的一种特殊情况

</details>
