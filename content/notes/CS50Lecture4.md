+++
title = 'CS50 Lecture 4-Memory'
date = 2024-05-22T12:58:30+08:00
draft = false
categories = ["Notes"]
tags = ["Notes", "Input", "CS"]

+++

> 这仅仅是我个人的学习笔记，没有什么干货，可能会有写错的信息，不推荐观看学习!

#### Welcome
- 在前几周，我们谈论到了图像是由称为`pixel`的较小构建块组成的

#### Pixel Art
- 像素是正方形，单个点，颜色排列在上下左右网格之中
- 可以将图像想象为 `bits`，其中`0`代表黑色，`1`代表白色
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/860adf89b5e5e1756701628c8d45d05d.png)

- RGB，其实就是 **red, green, bule**，这些数字是表示这些颜色中每种颜色的数量。在Adoble Photoshop 中，你可以看到这些设置
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/92d8e386c44965e76107694791287939.png)
- 从上图可以看出，颜色不仅仅用三个值来表示。在窗口的底部，有一个由数字和字符组成的特殊值。`255`表示`FF`，这就是`Hexadecimal`

#### Hexadecimal(十六进制)
- *Hexadecimal* 是一种具有16个计数值的计数系统
```
0 1 2 3 4 5 6 7 8 9 a b c d e f
```
其中的`F` 代表的是 `15`

- Hexadecimal 也被称为 *base-16*
- 以 十六进制 计数的时候，每列就是16的幂
- 数字`0` 表示`00`
- 数字`1` 表示 `01`
- 数字`9` 表示 `09`
- 数字`10` 表示 `0A`
- 数字`15` 表示 `0F`
- 数字`16` 表示 `10`(是十六进制中的10而不是我们日常中使用的10)
- 数字`255`表示 `FF`，因为 16 x 15 是240。再加上15，得到的255。这个是你使用两位十六进制系统计算的最高数字

#### Memory
- 将 十六进制 应用于每个内存块，如果我们还想之前一样使用的话，我们就会很容易将十六进制中的`10` 和 二进制中的`10` 混淆
- 因此，按照惯例，所有十六进制的数通常用`0x`作为前缀表示
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/eeb7ea3f1287afa9429207c0719a0381.png)

- 在终端窗口中，输入`code addresses.c`，并按照以下方式编写代码
```C
#include <stdio.h>

int main(void)
{
    int n = 50;
    printf("%i\n", n);
}
```
其中`n`是怎么存储在内存中的，值为`50`
- 你可以可视化此程序
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/edaf3dd9b30d1c7e1840004728a40f05.png)
为什么会占用4个格子呢，因为`int`类型大小是 4 bytes

- 在 `C` 语言中有两个与内存相关的强大运算符
```
  & 提供存储在内存中某物的地址.
  * 指示编译器前往内存中的某个位置.
```

- 我们可以通过修改代码来利用这些知识
```C
#include <stdio.h>

int main(void)
{
    int n = 50;
    printf("%p\n", &n);
}
```
其中的`%p`，它允许我们擦看内存中某个位置的地址。`&n`可以当成`n`的地址。执行此代码将返回以`0x`开头的内存地址

#### Pointers(指针)
- `pointer` 是包含某个值的地址的变量。简而言之，指针是计算机内存中的一个地址

```C
#include <stdio.h>

int main(void)
{
	int n = 50; //声明一个int类型的n变量,将其初始化为50
	int *p = &n; //声明了指针变量p,&n获取n的地址,并将地址赋值指针变量p
	printf("%p\n, p"); //将指针变量p的地址打印输出
}
```
注意，`printf`行在`p`的位置打印整数。`int *p` 创建一个指针，其工作是存储整数的内存地址

- 我们可以将我么的代码可视化如下
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/f95d61c5ee00e147e3d34f0be7ab8b8d.png)
注意，指针看起来相当大。事实上，指针通常存储为 8-byte。`p`存储`50`的地址

- 你可以更准确地将指针可视化为指向另一个地址的一个地址
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/b1f4259eed64458a15c4606d61700a8d.png)


#### Strings
- 现在我们学习了基本的指针，我们可以剥离课程前面提供的简化程序(丢掉辅助轮)
- 回想一下，`string` 只是一个字符数组。例如,，`string  s = "HI!"` 可以表示如下
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/9445d852bf53a226d89e26f23cb39c09.png)

- 然而，`s` 到底是什么? `s`存储在内存中的什么位置? 可以想象, `s` 需要存储在某个地方。你可以可视化`s` 与 字符串的关系
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/743a67a436f18ba0eadd6a397b26609c.png)
注意名为`s`的指针如何告诉编译器字符串的第一个字节在内存中的位置

- 按如下方式修改代码
```C
#include <cs50.h>
#include <stdio.h>

int main(void)
{
    string s = "HI!";
    printf("%p\n", s);
    printf("%p\n", &s[0]);
    printf("%p\n", &s[1]);
    printf("%p\n", &s[2]);
    printf("%p\n", &s[3]);
}
```
上面打印了字符串`s`中每个字符的内存位置。`&`符号用于显示字符串中每个元素的地址。运行此代码时，注意元素`0`、`1`、`2`和`3`在内存中彼此相邻
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/3965cad34230cbbfe232af1c2335930d.png)


- 同样，你可以按照以下方法修改代码
```C
#include <stdio.h>

int main(void)
{
    char *s = "HI!";
    printf("%s\n", s);
}
```
此代码将显示从`s`位置开始的字符串。此代码有效地删除了`cs50.h`提供的`string`数据类型的辅助轮。这是原始C代码，没有cs50库的脚手架

#### Pointer Arithmetic
- 可以修改代码以更长的形式完成相同的操作
```C
#include <stdio.h>

int main(void)
{
    char *s = "HI!";
    printf("%c\n", s[0]);
    printf("%c\n", s[1]);
    printf("%c\n", s[2]);
}
```

- 也可以使用以下方法修改代码
```C
#include <stdio.h>

int main(void)
{
    char *s = "HI!";
    printf("%c\n", *s);
    printf("%c\n", *(s + 1));
    printf("%c\n", *(s + 2));
}
```
打印了`s` 位置的第一个字符。然后，打印位置为`s+1`处的字符，以此类推
输出的结果就是
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/52dcc8f671c90f61e4aca54626f8db19.png)

#### String Comparison
- A string 只是由其第一个字节表示的字符数组
- 在课程的前面，我们考虑了整数的比较。我们可以通过在中终端中输入`codecompare.c`并编写代码来咋代码中表示这一点
```C
#include <cs50.h>
#include <stdio.h>

int main(void)
{
    // Get two integers
    int i = get_int("i: ");
    int j = get_int("j: ");

    // Compare integers
    if (i == j)
    {
        printf("Same\n");
    }
    else
    {
        printf("Different\n");
    }
}
```
注意，上面的代码从用户那里获取两个整数并进行比较

- 但是对于`Strings`，不能使用`==` 运算符比较两个字符串
- 使用`==` 运算符尝试比较字符串将尝试比较字符串的内存位置，而不是其中的字符。因此可以使用`strcmp`
- 请按如下方式修改代码

```C 
#include <cs50.h>
#include <stdio.h>

int main(void)
{
    // Get two strings
    char *s = get_string("s: ");
    char *t = get_string("t: ");

    // Compare strings' addresses
    if (s == t)
    {
        printf("Same\n");
    }
    else
    {
        printf("Different\n");
    }
}
```
注意，输入两个`HI!`，仍然会导致`Different` 的输出

- 为什么会这样，因为比较的两个字符串的内存地址，而不是比较其中的字符。可以通过以下内容来直观地说明原因
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/611731eeeb827585bd4bc6b25eb95d0a.png)
- 上面地`compare.c` 实际上是在比较内存地址是是否相同，而不是字符本身
- 使用`strcmp`, 我们可以纠正我们地代码:
```C
#include <cs50.h>
#include <stdio.h>
#include <string.h>

int main(void)
{
    // Get two strings
    char *s = get_string("s: ");
    char *t = get_string("t: ");

    // Compare strings
    if (strcmp(s, t) == 0)
    {
        printf("Same\n");
    }
    else
    {
        printf("Different\n");
    }
}
```
注意，如果字符串相同，`strcmp` 可以返回`0`

- 为了进一步说明这两个字符串如果存在与两个位置，可以修改成以下代码
```C
#include <cs50.h>
#include <stdio.h>

int main(void)
{
    // Get two strings
    char *s = get_string("s: ");
    char *t = get_string("t: ");

    // Print strings
    printf("%s\n", s);
    printf("%s\n", t);
}
```
注意，我们现在有两个不同的字符串，分别存储在两个不同的地址位置

- 只需稍作修改，就能看到这两个字符串存储的地址
```C
#include <cs50.h>
#include <stdio.h>

int main(void)
{
    // Get two strings
    char *s = get_string("s: ");
    char *t = get_string("t: "); 

    // Print strings' addresses
    printf("%p\n", s); //%p 就是pointer代表输出s的地址
    printf("%p\n", t);
}
```
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/431b393216f1b05b1a53a59eb6ff8c82.png)

#### Copying
- 在编程的一个常见需求就是将一个字符串复制到另一个字符串
- 创建一个新的`copy.c` 文件，并编写代码如下
``` C
#include <cs50.h> 
#include <ctype.h>
#include <stdio.h>
#include <string.h>

int main(void)
{
    // Get a string
    string s = get_string("s: ");

    // Copy string's address
    string t = s;

    // Capitalize first letter in string
    t[0] = toupper(t[0]);

    // Print string twice
    printf("s: %s\n", s);
    printf("t: %s\n", t);
}
```
注意，`string t = s ` 将 `s` 的地址复制到了 `t`。 这并没有达到我们的目的。字符串并没有被复制，复制的只是地址

- 你可以将上述代码可视化成如下

![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/be93353d4913c7c005caf5f87caff7b4.png)
请注意，`s`和`t`仍然指向相同的内存块。这并不是字符串的真实COPY。相反，这两个指针指向的是同一个字符串。

- 在解决这个问题之前，我们必须要确保我们的代码不会出现 ***[[segmentation fault]]***， 即将`sstring s` 复制到 `string t`中，而`string t` 并不存在。我们可以使用`strlen`函数来帮助解决这个问题:
```C 
#include <cs50.h>
#include <ctype.h>
#include <stdio.h>
#include <string.h>

int main(void)
{
    // Get a string
    string s = get_string("s: ");

    // Copy string's address
    string t = s;

    // Capitalize first letter in string
    if (strlen(t) > 0)
    {
        t[0] = toupper(t[0]);
    }

    // Print string twice
    printf("s: %s\n", s);
    printf("t: %s\n", t);
}
```
`strlen` 是用来确保 `string t` 是否存在。如果不存在，就不会复制任何内容

- 为了能够制作字符串的真实COPY，我们需要引入两个新的构建模块。首先，`malloc` 允许程序员分配一个特定大小的内存块。其次，`free`允许你告诉编译器释放之前分配的内存块
- 我们可以修改代码，如下所示创建字符串的真实副本(COPY)
```C
#include <cs50.h>
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // Get a string
    char *s = get_string("s: ");

    // Allocate memory for another string
    char *t = malloc(strlen(s) + 1);

    // Copy string into memory, including '\0'
    for (int i = 0; i <= strlen(s); i++)//给字符串末尾的空字符('\0')创建空间
    {
        t[i] = s[i];
    }

    // Capitalize copy
    t[0] = toupper(t[0]);

    // Print strings
    printf("s: %s\n", s);
    printf("t: %s\n", t);
}
```
注意，使用`malloc` 和 `free` 确保已经引入`stdlib.h` 头文件。`malloc(strlen(s) + 1 )` 创建的内存块长度是字符串`s` 的长度加1。这样就可以在最终复制的字符串中包含空`\0` 字符。然后，`for`循环遍历字符串`s`，并将每个值赋值到字符串`t`的相同位置

- 原来我们的代码效率不高。修改代码如下:
```C
#include <cs50.h>
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // Get a string
    char *s = get_string("s: ");

    // Allocate memory for another string
    char *t = malloc(strlen(s) + 1);

    // Copy string into memory, including '\0'
    for (int i = 0, n = strlen(s); i <= n; i++)
    {
        t[i] = s[i];
    }

    // Capitalize copy
    t[0] = toupper(t[0]);

    // Print strings
    printf("s: %s\n", s);
    printf("t: %s\n", t);
}
```
注意，`n = strlen(s)` 现在定义在`for loop` 的左侧。做好不要在`for` 循环的中间条件中调用不需要的函数，因为它会反复运行。将`n = strlen(s)` 移到左侧后，函数`strlen` 只运行一次

- C语言有一个用于复制字符串的内置函数，称为`strcpy`
```C
#include <cs50.h>
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // Get a string
    char *s = get_string("s: ");

    // Allocate memory for another string
    char *t = malloc(strlen(s) + 1);

    // Copy string into memory
    strcpy(t, s);

    // Capitalize copy
    t[0] = toupper(t[0]);

    // Print strings
    printf("s: %s\n", s);
    printf("t: %s\n", t);
}
```
注意，`strcpy` 所做的工作与之前的`for` 循环相同

- 如果出现问题， `get_string` 和 `malloc` 都会返回 `NULL`， 即内存中的一个特殊值。可以编写以下代码来检查`NULL` 条件

```C
#include <cs50.h>
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // Get a string
    char *s = get_string("s: ");
    if (s == NULL)
    {
        return 1;
    }

    // Allocate memory for another string
    char *t = malloc(strlen(s) + 1);
    if (t == NULL)
    {
        return 1;
    }

    // Copy string into memory
    strcpy(t, s);

    // Capitalize copy
    if (strlen(t) > 0)
    {
        t[0] = toupper(t[0]);
    }

    // Print strings
    printf("s: %s\n", s);
    printf("t: %s\n", t);

    // Free memory
    free(t);
    return 0;
}
```
注意，如果获得的字符串长度为`0` 或 *malloc fails*， 则返回`NULL`。此为，请注意`free`可以让计算机知道你已经完成了通过`malloc`创建的内存块，从而告诉编译器释放之前分配的内存块

#### malloc and Valgrind
- *Valgrind* 是一款可以检查程序是否存在内存相关问题的工具， 在这些问题中，你使用了`malloc`。具体来说，它会检查你是否`free` 分配的所有内存
- 以下是`memory.c`的代码
``` C
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *x = malloc(3 * sizeof(int));
    x[1] = 72;
    x[2] = 73;
    x[3] = 33;
}
```
注意， 运行该程序不会导致任何错误。虽然`malloc` 用于为数组分配足够的内存， 但是没有`free` 释放之前分配的内存块

- 这个时候输入`make memory` 然后执行 `valgrind ./memory`，你就会从 valgrind 收到一份报告，告诉你的程序在哪些地方导致了内存泄漏。 valgrind 显示的一个错误是， 我们试图在数组的第4个位置赋值为`33`， 但我们只分配了一个大小为`3` 的数组。另一个错误是我们未释放`x`
![image.png](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/6d63cfb99692b091ee2cbbbc4315e13e.png)

- 可以按照如下方式修改代码
```C
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *x = malloc(3 * sizeof(int));
    x[0] = 72;
    x[1] = 73;
    x[2] = 33;
    free(x);
}
```
再次运行 valgrind 不会出现内存泄漏
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/df439b490d9b5c2da0cf6c757adb4dc3.png)

#### Garbage Values
- 当你要求编译器提供一个内存块时，并不能保证这个内存块是空的
- 你分配的这些内存很有可能之前已经被计算机使用过。因此，你可能会看到 *garbage values*。 这是因为你获得了一个内存块，但没有对其进行初始化。例如，下面是`garbage.c`的代码
```C
#include <stdio.h>

int main(void)
{
    int scores[1024];
    for (int i = 0; i < 1024; i++)
    {
        printf("%i\n", scores[i]);
    }
}
```
注意，运行这段代码将为数组分配`1024`个内存位置，但`for`循环可能会显示其中并非所有值都是`0`。如果不将内存块初始化为其他值(如0或其他值)，就有可能产生垃圾值

#### Pointer Fun with Binky
- 我们观看[斯坦福大学的一段视频](https://www.youtube.com/watch?v=5VnDaHBi8dM)，它帮助我们直观地理解了指针

#### Swap
- 在现实世界中， 编程的一个常见需求是交换两个值。当然，如果没有临时的容纳空间，就很难交换两个变量。在实际操作中，你可以输入`code swap.c` 并编写如下如下代码来了解这种情况(可以在[Lecture4视频](https://youtu.be/F9-yqoS7b8w?t=5597)查看
```C
#include <stdio.h>

void swap(int a, int b);

int main(void)
{
    int x = 1;
    int y = 2;

    printf("x is %i, y is %i\n", x, y);
    swap(x, y);
    printf("x is %i, y is %i\n", x, y);
}

void swap(int a, int b)
{
    int tmp = a;
    a = b;
    b = tmp;
}
```
注意， 这段代码在运行时并不工作。即使在发送到`swap` 函数后， 数值也不会交换，为什么呢

当你向函数传递值时，你只是提供了副本。在前几周，我们讨论了作用域的概念。在`main`函数的大括号`{}`中创建的`x`和`y`值只有`main`函数的作用域

![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/217170d9a70856e81f19fcc6c49682ab.png)
注意，全局变量(我们在本课程中没有使用过的)存储在内存的一个位置。不同的函数存储在内存中另一个区域的`stack`中

- 请看下图:
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/9f23a3b5178d67904eafad11af2a566e.png)
注意， main和swap 有两个独立的内存框架或区域。因此，我们不能简单地将值从一个函数传递给另一个函数来改变它们
所以，可以使用`return`语句来返回值。但是在C语言中，不能直接通过`return`来返回多个值来交换两个变量地值，因为C语言地函数只能返回一个值

##### passing by reference
也就是 *passing by pointer*，来实现这个功能
```C
#include <stdio.h>

void swap(int *a, int *b);

int main(void)
{
    int x = 1;
    int y = 2;

    printf("x is %i, y is %i\n", x, y);
    swap(&x, &y);
    printf("x is %i, y is %i\n", x, y);
}

void swap(int *a, int *b)
{
    int tmp = *a;
    *a = *b;
    *b = tmp;
}
```

注意，变量不是通过 *value* 传递的， 而是通过 **reference** 传递的。也就是说， `a` 和 `b` 的地址是提供给函数的。因此，`swap` 函数可以知道从 主函数 中对实际的`a` 和 `b` 进行修改的位置

- 可以将形象化如下
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/05/d94241d9e7d8c2e94b0caeefda9b6554.png)


#### Overflow
- **A heap overflow** 是指内存栈溢出，触及到不该触及的内存区域
- **A stack overflow** 是指调用的函数过多，导致可用的内存量溢出
- 这两种情况都属于`buffer overflows`
- 当我们使用`malloc` 并向计算机请求内存时，它来自`heap` 区域
- 当你用带有变量的和参数的函数时，就是正在使用`stack memory`

#### `scanf`
- 在CS50中， 我们创建了`get_int` 等函数来简化从用户获取输入的过程
- `scanf` 是一个内置函数，可以获取用户输入
- 我们可以很容易地用`scanf` 重新实现 `get_int`的功能
```C
#include <stdio.h>

int main(void)
{
    int x;
    printf("x: ");
    scanf("%i", &x);
    printf("x: %i\n", x);
}
```
请注意， `x`的值存储在行`scanf("%i", &x)` 中`x`的位置

- 但是呢，试图重新实现`get_string` 并不容易
```C
#include <stdio.h>

int main(void)
{
    char *s;
    printf("s: ");
    scanf("%s", s);
    printf("s: %s\n", s);
}
```
由于字符串比较特殊，所以不需要`&`。尽管如此，这个程序还是无法运行。在这个程序中，我们没有为字符串分配所需的内存量。事实上，我们并不知道用户可能输入多长的字符串

- 此外，代码可以修改如下。不过，我们必须为字符串预分配一定量的内存
```C
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = malloc(4);
    if (s == NULL)
    {
        return 1;
    }
    printf("s: ");
    scanf("%s", s);
    printf("s: %s\n", s);
    free(s);
    return 0;
}
```
如果提供的字符串长度为6 `bytes` ，则可能会出现错误

- 将代码简化，我们就能进一步理解预分配这一重要问题:
``` C 
#include <stdio.h>

int main(void)
{
    char s[4];
    printf("s: ");
    scanf("%s", s);
    printf("s: %s\n", s);
}
```
如果我们预分配了一个大小为`4` 的数组，我们可以输入`cat` 并执行程序功能。但是，如果字符串大于这个大小，就会产生错误。

- 有时，编译器或运行编译器的系统可能会分配比我们指示的更多的内存。但从根本上，上述代码是不安全的。我们不能相信用户输入的字符串会适合我们预先分配的内存

#### File I/O
- 你可以读取和操作文件，下面是`phonebook.c` 的代码
```C
#include <cs50.h>
#include <stdio.h>
#include <string.h>

int main(void)
{
    // Open CSV file
    FILE *file = fopen("phonebook.csv", "a");

    // Get name and number
    char *name = get_string("Name: ");
    char *number = get_string("Number: ");

    // Print to file
    fprintf(file, "%s,%s\n", name, number);

    // Close file
    fclose(file);
}
```
注意，这段代码使用指针访问文件

- 你可以运行上面代码前创建一个名为`phonebook.csv`的文件。运行上述程序并输入姓名和电话号码后，就会发现这些数据一直存在CSV文件中
- 如果我们想在运行程序之前确保`phonebook.csv` 存在，可以对代码作如下修改:

``` C
#include <cs50.h>
#include <stdio.h>
#include <string.h>

int main(void)
{
    // Open CSV file
    FILE *file = fopen("phonebook.csv", "a");
    if (!file)
    {
        return 1;
    }

    // Get name and number
    char *name = get_string("Name: ");
    char *number = get_string("Number: ");

    // Print to file
    fprintf(file, "%s,%s\n", name, number);

    // Close file
    fclose(file);
}
```
注意，该程序通过调用`return 1` 来防止出现`NULL` 指针

- 我们可以通过输入`code cp.c ` 并编写如下代码来实现自己的复制程序:
```C
#include <stdio.h>
#include <stdint.h>

typedef uint8_t BYTE;

int main(int argc, char *argv[])
{
    FILE *src = fopen(argv[1], "rb");
    FILE *dst = fopen(argv[2], "wb");

    BYTE b;

    while (fread(&b, sizeof(b), 1, src) !=0)
    {
        fwrite(&b, sizeof(b), 1, dst);
    }

    fclose(dst);
    fclose(src);
}
```
注意，该文件创建了我们自己的数据类型，称为`BYTE`，其大小与`uint8_t` 相同。然后，文件读取`BYTE` 并将其写入文件

#### Summing Up
在本课中，您将学习指针的相关知识，它为你提供了在特定内存位置访问和操作数据的能力。具体来说，我们深入研究了...
- Pixel art
- Hexadecimal
- Memory
- Pointers
- Strings
- Pointer Arithmetic
- String Comparison
- Copying
- malloc and Valgrind
- Garbage values
- Swapping
- Overflow
- `scanf`
- File I/O
