不常见但是很有用的 GCC 命令行选项 （二）
============================================================

### 本文导航
1.  [生成 Wall 选项不包括的警告][1]
2.  [通过相等比较生成浮点值警告][2]
3.  [如何更好的管理 gcc 命令行选项][3]
4.  [结论][4]

gcc 编译器提供了几乎无数的命令行选项列表。当然，没有人使用或者精通它所有的命令行选项，但是有一系列被精心挑选出来的命令行选项是每一个 gcc 用户都应该知道的 - 即使不是必须知道。然而它们中的一些很常见，其他一些不太常见，但不常见并不意味着它们的用处没前者大。

在这个系列的文章中，我们集中于一些不常见但是很有用的 gcc 命令行选项，在[第一节][5]已经讲到几个这样的命令行选项。

不知道你是否能够回想起，在这个系列教程的第一部分的开始，我简要的提到了 - Wall 选项，开发者们通常使用它来生成警告，但不包括一些特殊的警告。如果你不认识这些特殊警告并且不知道如何生成它们，不用担心，因为我将在这篇文章中详细讲解关于它们所有的细节。

除此以外，这篇文章也将涉及与浮点值相关的 gcc 警告选项，以及在目录变得很大的时候如何更好的管理 gcc 命令行选项。

但是在往前看之前，请记住，这个教程中的所有例子，命令，指令所使用的环境：基于 Ubuntu 16.04 LTS 操作系统， gcc 版本为 5.4.0.

### 生成 -Wall 选项不包括的警告

尽管 gcc 编译器的 -Wall 选项涵盖了绝大多数警告标记，但依然有一些警告不能生成。为了生成它们，请使用 **-Wextra** 选项。

比如，看一下下面的代码：

```
#include <stdio.h>
#include <stdlib.h>
int main()
{
    int i=0;
    /* ...
       some code here 
       ...
    */

    if(i);
        return 1;
     return 0; 
}
```


我意外的在 'if'　条件后面多打了一个分号。现在，当使用下面的 gcc 命令来进行编译，不会生成任何警告。

gcc -Wall test.c -o test

但是如果同时使用 -Wextra 选项来进行编译：

gcc -Wall -Wextra test.c -o test

会生成下面这样一个警告：

```
test.c: In function ‘main’:
test.c:10:8: warning: suggest braces around empty body in an ‘if’ statement [-Wempty-body]
 if(i);
```

正如上面的警告清楚展示的那样， -Wextra 选项从内部生成了 -Wempty-body 标记，从而检测可疑代码并生成警告。下面是列出了这个选项生成的全部警告标志。
-Wclobbered, -Wempty-body, -Wignored-qualifiers, -Wmissing-field-initializers, -Wmissing-parameter-type (仅 C 语言能生成), -Wold-style-declaration (仅 C 语言能生成), -Woverride-init, -Wsign-compare, -Wtype-limits, -Wuninitialized, -Wunused-parameter (只有和 -Wunused 或 -Wall 选项同时使用时才会生成), 以及 -Wunused-but-set-parameter (只有和 -Wunused 或 -Wall 选项同时使用时才会生成). 

如果你想对上面所提到的标志有更进一步的了解，请查看 [gcc 手册][6]

继续往前看，遇到下面这些情况， -Wextra 选项也会生成警告：

*   一个指针和整数 0 进行 <， <=， >， 或 >=　比较
*   (仅 C++)一个枚举类型和一个非枚举类型同时出现在一个条件表达中
*   (仅 C++)有歧义的虚拟基底
*   (仅 C++)寄存器类型的数组加下标
*   (仅 C++)对寄存器类型的变量进行取址
*   (仅 C++)基类没有在派生类复制构建函数中进行初始化

### 通过相等比较生成浮点值警告

你可能已经知道，浮点值不能进行确切的相等比较（如果不知道，请阅读与浮点值比较相关的 [FAQ][7])。但是如果你意外的这样做了， gcc 编译器是否会忽略这个错误或警告？让我们来测试一下：

下面是一段使用 == 运算符进行浮点值比较的代码：

```
#include<stdio.h>

void compare(float x, float y)
{
    if(x == y)
    {
        printf("\n EQUAL \n");
    }
}

int main(void)
{
    compare(1.234, 1.56789);

    return 0; 
}
```

使用下面的 gcc 命令（包含 -Wall 和 - Wextra 选项）来编译这段代码：

gcc -Wall -Wextra test.c -o test

悲剧的是，上面的命令没有生成任何与浮点值比较相关的警告。快速看一下 gcc 手册，有一个专用的 **-Wfloat-equal** 选项，在这种情形下应该使用。

下面是包含这个选项的命令：

gcc -Wall -Wextra -Wfloat-equal test.c -o test

下面是这条命令产生的输出：

```
test.c: 在函数 ‘compare’ 中:
test.c:5:10: 警告: 对浮点值进行 == 或 ！= 比较是不安全的 [-Wfloat-equal]
 if(x == y)
```

正如上面你所看到的输出那样， -Wfloat-equal 选项会强制 gcc 编译器生成一个与浮点值比较相关的警告。

这儿是[gcc 手册][8]关于这一选项的说明：

```
The idea behind this is that sometimes it is convenient (for the programmer) to consider floating-point values as approximations to infinitely precise real numbers. If you are doing this, then you
need to compute (by analyzing the code, or in some other way) the maximum or likely maximum error that the computation introduces, and allow for it when performing comparisons (and when producing
output, but that's a different problem). In particular, instead of testing for equality, you shouldcheck to see whether the two values have ranges that overlap; and this is done with the relational operators, so equality comparisons are probably mistaken.
```
译文：
、、、这背后的想法是，有时，把浮点值考虑成近似无限精确的实数（对程序员来说）是方便的。如果你这样做，那么你需要通过分析代码，或者其他方式来计算所介绍的计算方式的最大值或可能的最大误差，然后允许它进行比较（并产生输出，但这是一个不同的问题）。特别地，你应该检查两者是否可能出现范围重叠；如果这时候使用关系运算符，相等比较就可能会出错。
、、、

### 如何更好的管理 gcc 命令行选项

如果在你使用的 gcc 命令中，命令行选项列表变得很大而且很难管理，那么你可以把它放在一个文本文件中，然后把文件名作为 gcc 命令的一个参数。之后，你必须使用 **@file** 命令行选项。

比如，下面这行是你的 gcc 命令：

gcc -Wall -Wextra -Wfloat-equal test.c -o test

然后你可以把这三个和警告相关的选项放到一个文件里，文件名叫做 'gcc-options'：

$ cat gcc-options 
-Wall -Wextra -Wfloat-equal


从而你的 gcc 命令会变得更加简洁并且易于管理：

gcc @gcc-options test.c -o test

这儿是 gcc 手册关于 @file 的说明：

```
Read command-line options from file. The options read are inserted in place of the original @file option. If file does not exist, or cannot be read, then the option will be treated literally, and not removed.

Options in file are separated by whitespace. A whitespace character may be included in an option by surrounding the entire option in either single or double quotes. Any character (including a backslash) may be included by prefixing the character to be included with a backslash. The file may itself contain additional @file options; any such options will be processed recursively.
```
译文
、、、
从文件中读取命令行选项。读取到的选项随之被插入到原始 @file 选项所在的位置。如果文件不存在或者无法读取，那么这个选项就会被当成文字处理，而不会被删除。
文件中的选项以空格分隔。空白字符可能被包含在一个由单引号或双引号所包围的完整选项中。任何字符（包括反斜杠: '\')均可能通过一个 '\' 前缀而包含在一个选项中。如果该文件本身包含额外的 @file 选项，那么它将会被递归处理。
、、、

### 结论

在这个系列的教程中，我们讲解了 5 个不常见但是很有用的 gcc 命令行选项： -Save-temps, -g, -Wextra, -Wfloat-equal 以及 @file。记得花时间练习使用每一个选项，同时不要忘了浏览 gcc 手册上面所提供的关于它们的全部细节。

你是否知道或使用其他像这样有用的 gcc 命令行选项，并希望把它们在全世界范围内分享？请在下面的评论区留下所有的细节。


--------------------------------------------------------------------------------

via: https://www.howtoforge.com/tutorial/uncommon-but-useful-gcc-command-line-options-2/

作者：[Ansh][a]
译者：[ucasFL](https://github.com/ucasFL)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]:https://twitter.com/howtoforgecom
[1]:https://www.howtoforge.com/tutorial/uncommon-but-useful-gcc-command-line-options-2/#enable-warnings-that-arent-covered-by-wall
[2]:https://www.howtoforge.com/tutorial/uncommon-but-useful-gcc-command-line-options-2/#enable-warning-fornbspfloating-point-values-in-equity-comparisons
[3]:https://www.howtoforge.com/tutorial/uncommon-but-useful-gcc-command-line-options-2/#how-to-better-manage-gcc-command-line-options
[4]:https://www.howtoforge.com/tutorial/uncommon-but-useful-gcc-command-line-options-2/#conclusion
[5]:https://www.howtoforge.com/tutorial/uncommon-but-useful-gcc-command-line-options/
[6]:https://linux.die.net/man/1/gcc
[7]:https://isocpp.org/wiki/faq/newbie
[8]:https://linux.die.net/man/1/gcc
