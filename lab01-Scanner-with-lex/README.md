# μRust: A Simple Rust Programming Language

[Homework Description](https://hackmd.io/@visitor-ckw/compiler_hw1)

Your assignment is to write a **scanner** for the μRust language with **lex**. This document gives the lexical definition of the language, while the syntactic definition and code generation will follow in subsequent assignments.

Your programming assignments are based around this division and later assignments will use the parts of the system you have built in the earlier assignments. That is, in the first assignment you will implement the scanner using lex, in the second assignment you will implement the syntactic definition in yacc.

## environment
* For Linux
    * Ubuntu 20.04 (os2022)
    * VScode
* Install Dependencies
```=
sudo apt install flex bison git python3 python3-pip
```
或者可以使用
```=
sudo apt install make
sudo apt install flex
```
### local judge
```=
pip3 install local-judge
```
![](https://i.imgur.com/QpH6xWS.png)

**There might be some problems according to the PATH of pip3 installation.** Enter the following command before `judge`
```=
PATH="$HOME/.local/bin:$PATH"
```
## Grammar of Lex (Lex 語法)
[A Lex Tutorial PDF](https://www.csie.cgu.edu.tw/~jhchen/course/PLP/Lex/lextut.pdf)
### 符號意思
* letter宣告為a-z, A-z, _的符號
* digit宣告為0-9的數字
* "."為特殊字元，表示全部的符號，因此宣告浮點數時，要多加反斜線跳出
* "+"代表至少要一個
* FloatNumber宣告為x.y，x跟y都至少要有一個數字
* String的宣告:
    *  `\"` 代表開頭要是引號""
    * `\.` 代表全部的字元
    * `|` 或
    * `[^“\\]`，`^`代表不包含，`”\\`代表引號
    * `*`代表零或多個
* Single Comment:
    * `//` 就是註解符號
    * `.` won't match a newline. So the following will match from `//`to the end of the line, and then do nothing.
```cpp
letter [a-zA-Z_]
digit [0-9]
FloatNumber {digit}+\.{digit}+
StringLiteral \"(\\.|[^"\\])*\"
single_comment "//".*
```

### precedence
優先權很重要！

重要的 keyword 應放在前面。
例如說，如果 判斷 IDENT 放在判斷 IF 前面，則，當 scanner 看見 "if" 時，會將其認成 IDENT 而非 IF


### Multi_line Comment
[Stack overflow 關於進入、離開 state](https://stackoverflow.com/questions/2130097/difficulty-getting-c-style-comments-in-flex-lex)
#### state 
使用 state 來處理多行註解 `/* multi line comment */`
* 定義 state
```=
%x state_name
```
* 進入、退出 state、implement
```=
BEGIN(state_name);               // enter state
BEGIN(INITIAL);                  // exist state (back to initial)
<state_name>"參數" {做什麼事;}     // do things in state
```

#### yylineno
要先宣告
```=
%option yylineno
```
yylineno 是用來記錄行數，目前掃到哪裡
可以記錄上次掃到的最後的註解行數，下次碰到註解時，**去看當前的 yylineno 是不是 = 上次的最後註解行**
* 例如：

```cpp=
fn main() { // Your first μrust program
    println("Hello World!"); 
    /* Hello 
    World */ /*    <- 在這裡，前面的尾 ＝ 後面的頭
    */
}
```

以上，有兩個 multiline comment 但整體只有三行


