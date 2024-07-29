+++
title = 'CS50 Lecture 1-C'
date = 2023-11-01T12:48:55+08:00
draft = false
categories = ["Notes"]
tags = ["CS", "Input", "Notes"]
+++

> 这仅仅是我个人的学习笔记，没有什么干货，可能会有写错的信息，不推荐观看学习!



#### Welcome!

- Recall that machines only understand binary. Where humans write _source code_, a list of instructions for the computer that is human readable, machines only understand what we can now call _machine code_. This machine code is a pattern of ones and zeros that produces a desired effect.

- It turns out that we can convert _source code_ into `machine code` using a very special piece of software called a _compiler_. Today, we will be introducing you to a compiler that will allow you to convert source code in the programming language _C_ into machine code.

#### Hello World
![](https://wangzhrbuckets.s3.bitiful.net/picture/2024/01/f5e69e15f18bfd1725b94767c7506132.png)
Notice that there is a _file explorer_ on the left side where you can find your files. Further, notice that there is a region in the middle called a _text editor_ where you can edit your program. Finally, there is a `command line interface`, known as a _CLI_, _command line_, or _terminal window_ where we can send commands to the computer in the cloud.

- We will be using three commands to write, compile, and run our first program
```
code hello.c

make hello

./hello
```
The first command, `code hello.c` creates a file and allows us to type instructions for this program. The second command, `make hello`, _compiles_ the file from our instructions in C and creates an executable file called `hello`. The last command, `./hello`, runs the program called `hello`.

- We can build your first program in C by typing `code hello.c` into the terminal window. Notice that we deliberately lowercased the entire filename and included the `.c` extension. Then, in the text editor that appears, write code as follows:
```
#include <stdio.h>

int main(void)
{
    printf("hello, world\n");
}
```
Note that every single character above serves a purpose. If you type it incorrectly, the program will not run. `printf` is a function that can output a line of text. Notice the placement of the quotes and the semicolon. Further, notice that the `\n` creates a new line after the words `hello, world`.

- Clicking back in the terminal window, you can compile your code by executing `make hello`. Notice that we are omitting `.c`. `make` is a compiler that will look for our `hello.c` file and turn it into a program called `hello`. If executing this command results in no errors, you can proceed. If not, double-check your code to ensure it matches the above.
- Now, type `./hello` and your program will execute saying `hello, world`.
- Now, open the _file explorer_ on the left. You will notice that there is now both a file called `hello.c` and another file called `hello`. `hello.c` is able to be read by the compiler: It’s where your code is stored. `hello` is an executable file that you can run, but cannot be read by the compiler.

#### Functions
- In Scratch, we utilized the `say` block to display any text on the screen. Indeed, in C, we have a function called `printf` that does exactly this.
- Notice our code already invokes this function:
```C
printf("hello, world\n");
```
Notice that the printf function is called. The argument passed to printf is ‘hello, world\n’. The statement of code is closed with a `;`.

``` C
#include <stdio.h>

int main(void)
{
    printf("hello, world\n");
}
```

- The statement at the start of the code `#include <stdio.h>` is a very special command that tells the compile that you want to use the capabilities of a _library_ called `stdio.h`, a _header file_. This allows you, among many other things, to utilize the `printf` function. You can read about all the capabilities of this library on the [Manual Pages](https://manual.cs50.io/). The Manual Pages provide a means by which to better understand what various commands do and how they function.
- Libraries are collections of pre-written functions that others have written in the past that we can utilize in our code.

#### Variables
- Recall that in Scratch, we had the ability to ask the user “What’s your name?” and say “hello” with that name appended to it.
- In C, we can do the same. Modify your code as follows:
```C
#include <stdio.h>
#include <cs50.h>

int main(void)
{
    string answer = get_string("What's your name? ");
    printf("hello, %s\n", answer);
}
```
The `get_string` function is used to get a string from the user. Then, the variable `answer` is passed to the `printf` function. `%s` tells the `printf` function to prepare itself to receive a `string`.
- `answer` is a special holding place we call a _variable_. `answer` is of type `string` and can hold any string within it. There are many _data types_, such as `int`, `bool`, `char`, and many others.
- `%s` is a placeholder called a _format code_ that tells the `printf` function to prepare to receive a `string`. `answer` is the `string` being passed to `%s`.

- `printf` allows for many format codes. Here is a noncomprehensive list of ones you may utilize in this course:
```
%c

%f

%i

%li

%s
```
`%s` is used for `string` variables. `%i` is used for `int` or integer variables. You can find out more about this on the [Manual Pages](https://manual.cs50.io/)

#### Conditionals
- In C, you can assign a value to an `int` or integer as follows:
```
int counter = 0;
```
Notice how a variable called `counter` of type `int` is assigned the value `0`.

- C can also be programmed to add one to `counter` as follows:
```
counter = counter + 1;
```
Notice how `1` is added to the value of `counter`.
    
- This can be represented also as:
```
counter = counter++;
```
Notice how `1` is added to the value of `counter`. However the `++` is used instead of `counter + 1`.
    
- You can also subtract one from `counter` as follows:
```
counter = counter--;
```
Notice how `1` is removed to the value of `counter`.

``` C
#include <cs50.h>
#include <stdio.h>

int main(void)
{
    // Prompt user to agree
    char c = get_char("Do you agree? ");

    // Check whether agreed
    if (c == 'Y' || c == 'y')
    {
        printf("Agreed.\n");
    }
    else if (c == 'N' || c == 'n')
    {
        printf("Not agreed.\n");
    }
}
```
Notice that single quotes are utilized for single characters. Further, notice that `==` ensure that something _is equal_ to something else, where a single equal sign would have a very different function in C. Finally, notice that `||` effectively means _or_.


#### Loops

- We look at a few examples from Scratch. Consider the following code:
```C
int counter = 3;
while (counter > 0)
{
    printf("meow\n");
    counter = counter - 1;
}
```
Notice that his code assigns the value of `3` to the `counter` variable. Then, the `while` loop says `meow` and removes one from the counter for each iteration. Once the counter is not greater than zero, the loop ends.

  
- Your `meow` function can be further modified to accept input:
```C
#include <stdio.h>

void meow(int n);

int main(void)
{
    meow(3);
}

// Meow some number of times
void meow(int n)
{
    for (int i = 0; i < n; i++)
    {
        printf("meow\n");
    }
}
```

#### Operators and Abstraction
- You can implement a calculator in C. In your terminal, type `code calculator.c` and write code as follows:
```
#include <cs50.h>
#include <stdio.h>
    
int main(void)
{
	// Prompt user for x
    int x = get_int("x: ");    
        
    // Prompt user for y
    int y = get_int("y: ");
    
	// Perform addition
	printf("%i\n", x + y);
    }
```
    
Notice how the `get_int` function is utilized to obtain an integer from the user twice. One integer is stored in the `int` variable called `x`. Another is stored in the `int` variable called `y`. Then, the `printf` function prints the value of `x + y`, designated by the `%i` symbol.

- _Operators_ refer to the mathematical operations that are supported by your compiler. In C, these mathematical operators include:
	- `+` for addition
    - `-` for subtraction
    - `*` for multiplication
    - `/` for division
    - `%` for remainder