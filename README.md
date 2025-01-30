# C 代码风格指南

这份文档描述了我（bfmhno3）的个人项目的 C 代码风格。

## 通用规则

+ 所有文件始终使用 UTF-8 编码。
+ 使用 `C99` 标准（如果可能的话，使用 `C11` 标准）。
+ 不要使用制表符进行缩进，而是使用空格 ` ` 进行缩进。
+ 每一层缩进使用 4 个空格。
+ 在关键字和左括号（例如 `(`、`{`、`<`）之间添加一个空格 ` `。

以下是一些示例：

```c
/* 正确的示例 */
if (condition)
while (condition)
for (init; condition; step)
do {} while (condition)
    
/* 错误的示例 */
if(condition)
while(condition)
for(init;condition;step)
do {} while(condition)
```

+ 在函数名和左括号 `(` 之间不要使用空格。

```C
int32_t a = sum(4, 3);              /* 正确 */
int32_t b = sum (4, 3);             /* 错误 */
```

+ 不要使用 `__` 和 `_` 作为变量、函数、宏以及自定义类型的前缀。这是为了 C 语言本身而保留的。
  + 对于模块严格私有（静态）函数，优先使用 `prv` 作为前缀。
  + 对于库的内部函数，应该使用 `libname_int_` 或 `libnamei_` 前缀，这些函数不应被用户应用程序使用，但必须在库的不同模块之间使用。
+ 标识符命名要有**描述性**，少用缩写（少数特定的广为人知的缩写是允许的，例如 `src`、`tmp`、`cnt` 等，但尽量使用全称）。
  + 源文件使用 `.c` 为扩展名，头文件则使用 `.h`。
  + 文件名要全部小写，使用下划线 `_` 连接不同单词，例如 `my_usefule_module.c`。
  + 宏的命名则使用全部大写，使用下划线 `_` 分隔每个单词。
  + 自定义类型的的每个单词首字母均应该大写，不包含下划线，例如 `MyExcitingClass`。
  + 变量（包括函数参数）和结构体成员名一律使用小写，单词之间使用下划线 `_` 连接，例如 `int table_name[]`。
  + 布尔变量使用 `is`、`has`、`can` 等描述性的前缀。
  + 声明为 `const` 的变量或在程序运行期间其值始终保持不变的，命名时以 `k` 开头，大小写混合，不包含下划线 `_`，例如 `const int kDaysInAWeek = 7;`。
  + 常规函数使用大小写混合，取值和设值函数则要求与变量名相匹配，同样不包含下划线。对于使用首字母缩写的短语，更倾向于将它们视为一个单词畸形首字母大写。同时，函数应使用定义的前缀标识其返回值类型，前缀将在后文中提及。
  + 枚举的命名应当和使用 `const` 声明的变量或宏一直，即 `kEnumName` 或 `ENUM_NAME`。
+ 左括号应当和关键字以及标识符（函数名等）位于同一行，左括号与前面的代码之间应使用一个空格。

```C
size_t i;
for (i = 0; i < 5; ++i) {           /* 正确 */
    
}

for (i = 0; i < 5; ++i){            /* 错误 */
    
}

for (i = 0; i < 5; ++i)             /* 错误 */
{
}
```

+ 在双目运算符（例如赋值运算符 `=`）的前后各使用一个空格。

```C
int32_t a;
a = 3 + 4;              /* 错误 */
for (a = 0; a < 5; ++a) /* 正确 */
a=3+4;                  /* 错误 */
a = 3+4                 /* 错误 */
for(a=0;a<5;++a)        /* 错误 */
```

+ 在每个逗号 `,` 后使用一个空格。

```C
FuncName(5, 4);         /* 正确 */
FuncName(4,3);          /* 错误 */
```

+ 不要初始化全局变量为某个默认值（或者 `NULL`），如果需要初始化全局变量，则应该实现相应的初始化函数。建议在声明处初始化局部变量。

> 在嵌入式系统中，RAM 存储器通常分散在系统的不同内存位置。处理所有情况会变得很棘手，尤其是用户生命自定义 RAM 区段时。启动脚本赋值设置默认值（`.data` 和 `.bss`），而其他自定义区段可能不会填充默认值，这导致初始化值的变量不起作用。
>
> 为了避免此类问题，为每个模块创建初始化函数，并使用它来为所有变量设置默认值，如下所示：

```C
static int32_t a;       /* 正确 */
static int32_t b = 4;   /* 错误 - 如果链接脚本或启动文件没有被正确执行，这个值不会起作用 */

void vMyModuleInit(void) {
    a = 0;
    b = 4;
}
```

+ 将所有同类型的变量声明在同一行。如果需要进行初始化，则应该分行书写。

```C
void vMyFunc(void) {
	/* 1 */
    char a;             /* 正确 */
    
    /* 2 */
    char a, b;          /* 正确 */
    
    /* 3 */
    char a;
    char b;              /* 错误*/
}
```

+ 按以下顺序声明局部变量：
  1. 自定义结构体和枚举类型
  2. 整型系列，范围更大的无符号整型优先
  3. 单精度或双精度浮点数

```C
int lMyFunc(void) {
    /* 1 */
    MyStruct_t my;     /* 自定义结构体优先 */
    MyStructPtr_t* p;  /* 自定义指针类型也优先 */
    
    /* 2 */
    uint32_t a;
    int32_t b;
    uint16_t c;
    int16_t g;
    char h;
    /* ... */
    
    /* 3 */
    double d;
    float f;
}
```

+ 总是在代码块的顶部，第一条可执行语句之前声明局部变量。
+ 除非结构体十分简单，否则应该使用 `C99` 中的方式来初始化结构体，同时在每一项的末尾添加逗号 `,`。

```C
typedef struct {
    int a, b;
} Str_t;

Str_t s = {
    .a = 1,
    .b = 2,   /* 在最后一项的末尾也添加逗号 */
}
```

+ 将所有计数器声明在 `for` 循环内部，除非需要在外部使用这个值。

```C
/* 正确 */
for (size_t i = 0; i < 10; ++i)

/* 正确，如果需要在外部使用计数器的值 */
size_t i;
for (i = 0; i < 10; ++i) {
    if (...) {
        break;
    }
}
if (i == 10) {
    
}

/* 错误 */
size_t i;
for (i = 0; i < 10; ++i) ...
```

+ 避免在变量声明时使用函数返回值进行初始化，除非只是声明单个变量。

```C
void vA(void) {
    /* 避免在声明变量时使用函数返回值进行初始化 */
    int32_t a, b = sum(1, 2);
    
    /* 正确 */
    int32_t a, b;
    b = sum(1, 2);
    
    /* 正确 */
    uint8_t a = 3, b = 4;
}
```

+ 除了 `char`、`float` 和 `double`，其余类型应使用 C 标准库中 `stdint.h` 中定义的类型，例如使用 `uint8_t` 表示无符号 8 位整型。
+ 需要布尔变量时，使用 C 标准库中的 `stdbool.h` 定义的宏
+ 不要与 `true` 进行比较，例如使用 `if (CheckFunc()) {...}` 而非 `if(CheckFunc() == true)`。
+ 总是将指针与 `NULL` 进行比较

```C
void* ptr;

/* ... */

/* 正确，与 `NULL` 进行比较 */
if (ptr == NULL || ptr != NULL) {
    
}

/* wrong */
if (ptr || !ptr) {
    
}
```

+ 若无特殊需求，尽量使用前缀自增或子健，而非后缀。

```C
int32_t a = 0;
...

a++;            /* 错误 */
++a;            /* 正确 */

for (size_t j = 0; j < 10; ++j) {}  /* 正确 */
```

+ 对于表示长度或大小的变量总是使用 `size_t` 类型。
+ 如果函数不会修改指针指向的内存的值或者指针本身的值，则使用 `const` 修饰指针。

```C
/* d 本身可能会被修改，但 d 指针的内存的值不会被修改 */
void vMyFunc(const void* d) {
    
}

/* d 本身和其指向的内存的值都不会被修改 */
void vMyFunc(const void* const d) {
    
}

/* 非必要，但建议这样写，防止意外修改后再使用值 */
void vMyFunc(const size_t len) {
    
}

/* d 本身不会被修改，但 d 指向的内存的值可能被修改 */
void vMyFunc(void* const d) {
    
}
```

+ 当函数可能接收任意类型的指针是，总是使用 `void *`，而不是使用 `uint8_t *`
  + 函数实现的内部应该处理正确的类型转换

``` C
/*
 * 当发送数据时，函数不应该修改 `data` 所指向的内存的值
 * 因此 `const` 关键字很重要
 * 
 * 为了发送数据功能的通用性（或者将其写入文件中）
 * `data` 可以是任意类型
 * 因此应该使用 `void *`
 */
/* 以下是一个正确的例子 */
void vSendData(const void* data, size_t len) { /* 正确 */
    /* 不要将 `data` 转换为 `void *` 或 `const void *` */
    const uint8_t* d = data; /* 函数实现内部处理正确的类型转换 */
}

void vSendData(const void* data, int len) {    /* 错误，不应该使用 int */

}
```

+ 使用 `sizeof` 运算符时，总是使用小括号 `()`，例如 `sizeof(int)`。
+ 禁止使用变长数组。使用 C 标准库提供的 `malloc` 和 `free` 进行动态内存分配，如果库或项目提供了自定义的内存分配实现，使用该实现。

```C
/* 正确 */
#include <stdlib.h>
void vMyFunc(size_t size) {
    int32_t* arr;
    arr = malloc(sizeof(*arr) * n); /* 正确 */
    arr = malloc(sizeof *arr * n); /* sizeof 应使用小括号 () */
    if (arr == NULL) {
        /* 错误，无内存 */
    }
    
    free(arr); /* 使用完后释放内存 */
}

/* 错误 */
void vMyFunc(size_t size) {
    int32_t arr[size]; /* 错误，不要使用变长数组 */
}
```

+ 始终将变量与 `0` 进行比较，除非它被视为布尔类型。
+ 不要将布尔类型的变量与 `0` 和 `1` 进行比较来判断真假，在需要的情况下使用 `!` 来判断。

``` C
size_t length = 5; /* 计数器 */
bool is_ok = 0;    /* 布尔变量 */
if (length)        /* 错误，length 不是布尔值 */
if (length > 0)    /* 正确，length 是计数器，可能包含多种值，不止 0 和 1 */
if (length == 0)   /* 正确，length 是计数器，可能包含多种值，不止 0 和 1 */
    
if (is_ok)         /* 正确，is_ok 是布尔值 */
if (!is_ok)        /* 正确 */
if (is_ok == 1)    /* 错误，不要将布尔值与 1 进行比较 */
if (is_ok == 0)    /* 错误，判断 假时使用 `!` */
```

+ 始终在头文件中使用 `extern` 关键字包含对 C++ 的检查。
+ 每个函数都必须包含启用 `doxygen` 的注释，即使这个函数是静态的。
+ 始终使用英语给函数、变量、自定义类型命名，**禁止使用汉语拼音命名**。
+ 允许使用汉语进行注释，但注释所使用的语言（英语还是汉语）应该与该项目保持一致。
+ 不要对返回 `void *` 的函数的返回值进行强制类型转换，因为 `void *` 可以被安全地转换为任意指针类型。

```C
uint8_t* ptr = (uint8_t*)FuncReturnVoidPtr(); /* 错误 */
uint8_t* ptr = FuncReturnVoidPtr();            /* 正确 */
```

+ 使用 `<>` 包含 C 标准库所提供的头文件；使用 `""` 包含任意非 C 标准库提供的头文件，包括第三方库。
+ 当对指针类型进行强制类型转换时，始终将 `*` 与类型对齐，例如 `uint8_t* t = (uint8_t*)var_width_diff_type`。
+ 始终检查项目或库中的代码风格。有时旧的代码可能没有及时跟进新的代码风格，当检查到了，应及时修改。

## 注释

+ 允许使用 `//` 进行单行注释，但如果有三行及以上连续的单行注释，应该使用 `/**/` 进行多行注释，可以使用 `/**/` 进行单行注释。

```C
// 这是一行单行注释 （正确）

// 这是第 1 行注释
// 这是第 2 行注释
// 这是第 3 行注释 （错误）

/*
 * 这是第 1 行注释
 * 这是第 2 行注释
 * 这是第 3 行注释 （正确）
 */
```

+ 对于多行注释，每一行注释前应使用空格和星号开头 `  *`。

```C
/*
 * 这是多行注释
 * 注意每一行的写法
 */

/**
 * 错误，只在 doxygen 文档注释中使用双星号
 */

/*
* 错误，每行第一个字符应该是空格
*/

/*
 * 错误，这是多行注释的写法
 */

/* 正确，这是单行注释的写法 */
```

+ 使用单行注释时，注释应从每行的第 48 个（`12 * 4`）字符处开始书写；如果代码语句超过了 48 个字符，则应该从最近的 4 的整数倍数处开始书写，如果发现注释刚好紧随代码语句，则应再空 `4` 个字符。

```C
void vMyFunc(void) {
    char a, b;
    
    a = CallFuncReturningCharA(a);          /* 从第 48 个字符开始书写 */
    b = CallFuncReturningCharAButFuncNameIsVeryLong(a);     /* 从第 60 字符开始书写 */
}
```

+ 规定多行注释用于解释其下方紧随的代码，如果后方的代码不由该注释解释，则应该空行书写，多行注释与其上的代码应有一个空行分隔，如果其上刚好是函数名，则不用空行。多行注释应与其下的代码保持相同的缩进。

```C
void vMyFunc(void) {
    /*
     * 这是对下面的代码的注释，
     * 注意上方没有空行
     */
    char a = CallFuncReturningCharA(a);          /* 代码与上方的注释没有空行 */
    
    ...
        
        /*
         * 注释与要注释的代码保持相同的缩进
         * 注意这里上方有空行
         */
        b = CallFuncReturningCharAButFuncNameIsVeryLong(a);
    
    	c = CallFuncC(c); // 这段代码不由上方的注释解释，则应该空行
}
```

## 函数

+ 如果函数需要在外部调用，则其配套的头文件应包含其**函数原型**（*Function Prototype*）。
+ 常规函数使用大小写混合，取值和设值函数则要求与变量名相匹配，同样不包含下划线。对于使用首字母缩写的短语，更倾向于将它们视为一个单词即首字母大写。
+ 函数应使用定义的前缀标识其返回值类型。

```C
/* 正确 */
void           vMyFunc(void);
static uint8_t prvMyFunc(void);

void           MyFunc(void);    /* 错误，未使用前缀 */
static uint8_t ucMyFunc(void);  /* 错误，应使用 prv */
void           vMYFunc(void);   /* 错误，只有每个单词的首字母应该大写 */
void           myFunc(void);    /* 错误 */
void           my_func(void);   /* 错误 */
```

+ 对于具有相同或相似功能（例如提供对变量的读写操作）的函数原型，对齐函数名，以提高可读性。

```C
/* 正确 */
void     vSet(int32_t a);
MyType_t xGet(void);
MyPtr_t* xpGetPtr(void);

/* 错误 */
void vSet(int32_t a);
const char* cpGet(void);
```

+ 函数返回值类型应该与函数名在同一行。

```C
/* 错误 */
int32_t
lFoo(void) {
    return 0;
}

/* 错误 */
static const char*
prvGetString(void) {
    return "Hello World!\r\n";
}

/* 正确 */
int32_t lFoo(void) {
    return 0;
}
```

+ 只把 10 行以下的小函数定义为内联函数。
+ 内联函数中禁止使用循环或 `switch` 语句，即若函数中含有循环或 `switch` 语句，则不应噶将其声明为内联函数。
+ 函数不能超过 40 行，如果超出了，则应该将其分割为一些小函数，然后在主函数中调用这些小函数。

### 前缀命名规则

1. 私有函数（使用 `static` 声明的函数）：使用 `prv` 为前缀。私有函数只使用 `prv` 为前缀，不使用其余前缀标识其返回值类型。
2. 基础数据类型
   + 无符号类型（`unsigned`）以 `u` 开头：
     + `uc`：`unsigned char` / `uint8_t`
     + `us`：`unsigned short` / `uint16_t`
     + `ul`：`unsigned long` / `uint32_t`
     + `ull`：`unsigned long long` / `uint64_t`
   + 有符号类型（默认）：
     + 若不使用 `u` 作为前缀，则默认为有符号类型
     + 其余类型的前缀付下：
       + `b`：`bool`
       + `f`：`float`
       + `d`：`double`
3. 复合类型
   + `x`：自定义结构体（`struct`）
   + `e`：自定义枚举类型（`enum`）
4. 特殊类型
   + `v`：`void`
   + `p`：指针类型（可与其他前缀组合）：
     + `vp`：`void *`
     + `ulp`：`unsigned long *`

## 变量

+ 变量命名应该使用全部小写，每个单词之间使用 `_` 分隔。

```C
/* 正确 */
int32_t a;
int32_t my_var;

/* 错误 */
int32_t myvar;
int32_t A;
int32_t myVar;
int32_t MYVar;
```

+ 相同类型的变量声明在同一行，如果需要初始化，则应该分行写。

```C
/* 正确 */
void vFoo(void) {
    int32_t a, b;   /* 正确 */
    char a;
    char b;         /* 错误 */
}
```

+ 在第一条可执行代码后不要再声明变量。

```C
void vFoo(void) {
    int32_t a;
    a = Bar();
    int32_t b; /* 错误 */
}
```

+ 可以在下一个代码块中声明变量。

```C
int32_t a, b;
a = foo();
if (a) {
    int32_t c, d; /* 正确 */
    c = Foo();
    int32_t e;    /* 错误 */
}
```

+ 声明指针变量时，将星号 `*` 与类型名对齐。

```C
/* 正确 */
char* a;

/* 错误 */
char *a;
char * a;
```

+ 在同一行声明多个指针变量时，星号与变量名对齐。

```C
/* 正确 */
char *p, *n;
```

## 结构体、枚举和 `typedef`

+ 结构体和枚举类型命名应大写每个单词首字母，不包含下划线。
+ 结构体和枚举类型必须使用 `typedef` 起别名。
+ 所有的结构体成员必须使用变量的命名规则，即全部小写，并使用 `_` 分隔。
+ 所有的枚举成员可以使用两种命名方式：类似宏的全大写风格，并使用 `_` 分隔；类似使用 `const` 声明的变量，使用 `k` 为前缀，并只大写每个单词的首字母，不包含 `_`。在具体项目中，应只使用一种命名方式。

> [!tip]
>
> 建议对宏常量使用全大写和 `_` 进行命名，而使用 `k` 前缀的方式为枚举成员和常变量进行命名，有助于分辨逻辑上的常量。

+ 枚举类型的所有成员应显式地写出其值。

```C
typedef enum Week { // 没有 _t 后缀
    kMonday    = 1,
    kTuesday   = 2,
    kWednesday = 3,
    kThursday  = 4,
    kFriday    = 5,
    kSaturday  = 6,
    kSunday    = 7, // 注意这里的逗号
} Week_t; // 使用 _t 后缀
```

+ 禁止将结构体和枚举的成员放在同一行中，而应该分行书写。
+ 结构体或枚举的注释应使用 `doxygen` 的语法。
+ 禁止使用 `.` 或 `->` 直接访问结构体的成员，开发者应该提供相应的宏或函数进行成员的读写操作。

使用 `typedef` 的规则：

+ 所有结构体和枚举类型都必须使用 `typedef` 以省略使用时的 `struct` 和 `enum`。
+ 所有未使用 `typedef` 重命名的类型禁止包含 `_t` 后缀，所有使用 `typedef` 重命名的类型必须包含 `_t` 后缀。
+ 若需要重命名指针类型，则应使用 `_pt` 后缀，其中 `p` 表示这是一个指针类型。
  + 函数指针是一个特例，重命名后的函数指针应该使用 `_ft` 为后缀，以强调这是一个可调用对象。

```C
typedef uint8_t (*MyFuncTypedef_pt)(uint8_t p1, const char* p2);
```

## 复合语句

+ 每个复合语句都必须包含一对完整的花括号 `{}`，即使它只包含 1 条嵌套语句。
+ 在复合语句中嵌套其余语句时，必须缩进，每一层 `4` 个空格。

```C
/* 正确 */
if (c) {
    DoA();
} else {
    DoB();
}

/* 错误 */
if (c)
    DoA();
else
    DoB();

/* 错误 */
if (c) DoA();
else DoB();
```

+ 当使用 `if-else` 语句时，`else` 必须和右花括号 `}` 在同一行中。

```C
/* 正确 */
if (a) {
    
} else if (b) {
    
} else {
    
}

/* 错误 */
if (a) {
    
}
else {
    
}

/* 错误 */
if (a) {
    
}
else
{
    
}
```

+ 对于 `do-while` 语句，`while` 必须和右花括号 `}` 在同一行，左花括号 `{` 必须和 `do` 在同一行。

```C
/* 正确 */
do {
    int32_t a;
    a = DoA();
    DoB(a);
} while (check());

/* 错误 */
do
{
/* ... */
} while (check());

/* 错误 */
do {
/* ... */
}
while (check());
```

+ 每个左花括号都需要缩进。

```C
if (a) {
    DoA();
} else {
    DoB();
    if (c) {
        DoC();
    }
}
```

+ 空循环（`for`、`while`、`do-while`）必须包含一对完整的花括号。

```C
/* 正确 */
while (IsRegisterBitSet()) {}

/* 错误 */
while (IsRegisterBitSet());
while (IsRegisterBitSet()) { }
while (IsRegisterBitSet()) {
}
```

+ 空循环应使用一对空的花括号，并将其与关键字放在一行中。‘

```C
/* 等待嵌入式硬件单元中的位被设置 */
volatile uint32_t* add = HW_PERIPH_REGISTER_ADDR;

/* 等待第 13 位被设置 */
while (*addr & (1 << 13)) {}        /* 正确 */
while (*addr & (1 << 13)) { }       /* 正确 */
while (*addr & (1 << 13)) {         /* 正确 */
    
}
while (*addr & (1 << 13));           /* 错误 */
```

+ 按一下顺序使用循环：`for`、`do-while`、`while`。
+ 尽量避免在循环体内增加变量。

```C
/* 不推荐 */
int32_t a = 0;
while (a < 10) {
    .
    ..
    ...
    ++a;
}

/* 推荐 */
for (size_t a = 0; a < 10; ++a) {
    
}

/* 推荐 */
for (size_t a = 0; a < 10; ) {
    if (...) {
        ++a;
    }
}
```

+ 只在需要赋值或传参时使用三目运算符 `?`，禁止使用三目运算符进行函数调用，应该使用 `if` 语句。

```C
/* 正确 */
int a = condition ? if_yes : if_no;       /* 赋值 */
FuncCall(condition ? if_yes : if_no);     /* 函数传参 */
switch (condition ? if_yes : if_no) {...} /* 类似函数传参 */

/* 错误，这段代码难以维护 */
condition ? CallToFunctionA() : CallToFunctionB();

/* 应该使用 if 语句 */
if (condition) {
    CallToFunctionA();
} else {
    CallToFunctionB();
}
```

## `switch` 语句

+ 对于每个 `case` 语句，都应该使用一层缩进。
+ 对于 `case` 语句中的内容，也使用一层缩进。

```C
/* 正确，每一个 case 语句都有一层缩进 */
/* 正确，case 内的语句都添加一层缩进 */
switch (check()) {
    case 0:
        DoA();
        break;
    case 1:
        DoB();
        break;
    default:
        break;
}

/* 错误，case 没有缩进 */
switch (check()) {
case 0:
    DoA();
    break;
case 1:
    DoB();
    break;
default:
    break;
}

/* 错误 */
switch (check()) {
    case 0:
        DoA();
    break;
    case 1:
        DoB();
    break;
    default:
    break;
}
```

+ 始终包含 `default` 语句。

+ 如果需要使用局部变量，使用 `{}` 声明一个代码块，并将 `break` 语句放入其中。
  + 将左花括号 `{` 与 `case` 放在同一行。

```C
switch (a) {
    /* 正确 */
    case 0: {
        int32_t a, b;
        char c;
        a = 5;
        /* ... */
        break;
    }
    /* 错误 */
    case 1:
    {
        int32_t a;
        break;
    }
    
    /* 错误 */
    case 2: {
        int32_t a;
    }
    break;
    
    default:
        ...
}
```

## 宏

+ 对于字面常量，始终使用宏，特别是数字。
+ 所有的宏应该使用全大写，并使用 `_` 分隔单词。

```C
/* 正确 */
#define SQUARE(x) ((x) * (x))

/* 错误 */
#define square(x) ((x) * (x))
```

+ 始终使用小括号 `()` 保护输入参数。

```C
/* 正确 */
#define MIN(x, y) ((x) < (y) ? (x) : (y))

/* 错误 */
#define MIN(x, y)  x < y ? x : y
```

+ 始终使用小括号 `()` 保护宏的值

```C
/* 错误 */
#define MIN(x, y) (x) < (y) ? (x) : (y)
#define SUM(X, y) (x) + (y)

/* 正确 */
#define MIN(x, y) ((x) < (y) ? (x) : (y))
#define SUM(x, y) ((x) + (y))
```

+ 如果宏有多行，应该使用 `do {} while (0)` 语句进行保护

```C
typedef struct {
    int32_t px, py;
} Point_t;
Point_t p;                  /* 定义新的 Point 类 */

/* 错误的实现 */

/* 定义宏来设置 Point 的值 */
#define SET_POINT(p, x, y)  (p)->px = (x); (p)->py = (y)

SET_POINT(&p, 3, 4);        /* 将 Point 的位置设置为 (3, 4)，这将被预处理器转换为 */
(&p)->px = (3); (&p)->py = (4); /* 这个样子。在这个简单的例子中，不会有任何问题 */

/* 考虑下面这段丑陋的代码，虽然它在 C 中是有效的（不推荐） */
if (a)                      /* 如果 a 为真 */
    if (b)                  /* 如果 b 为真 */
        SET_POINT(&p, 3, 4);/* Set point to x = 3, y = 4 */
    else
        SET_POINT(&p, 5, 6);/* Set point to x = 5, y = 6 */

/* 上面的代码将被预处理器转换为下面的代码。你能发现问题吗？ */
if (a)
    if (b)
        (&p)->px = (3); (&p)->py = (4);
    else
        (&p)->px = (5); (&p)->py = (6);

/* 我们可以稍微重写一下上面的代码 */
if (a)
    if (b)
        (&p)->px = (3);
        (&p)->py = (4);
    else
        (&p)->px = (5);
        (&p)->py = (6);



/*
 * 问你自己一个问题：`else` 关键字是属于哪个 `if` 关键字的？
 *
 * 如果基于第一段代码，答案可能是显而易见的。`else` 是属于内部的 `if` 关键字的，当检查 `b` 条件为假时，应该会执行 `else` 内的代码。
 * 但真实情况是，编译报错：`else` without a previous `if`
 */

/* 这个宏的正确实现如下 */
#define SET_POINT(p, x, y)  do { (p)->px = (x); (p)->py = (y); } while (0)
/* 更好的实现 */
#define SET_POINT(p, x, y)  do {    \
    (p)->px = (x);                  \
    (p)->py = (y);                  \
} while (0)

/* 现在那段代码就会被转换成如下所示 */
if (a)
    if (b)
        do { (&p)->px = (3); (&p)->py = (4); } while (0);
    else
        do { (&p)->px = (5); (&p)->py = (6); } while (0);

/* 每一个 `if` 和 `else` 语句都只包含一个内部语句，所以这是有效的 */

/* 最好的做法是使用 `{}` 包含代码 */
if (a) {
    if (b) {
        SET_POINT(&p, 3, 4);
    } else {
        SET_POINT(&p, 5, 6);
    }
}
```

+ 除了头文件保护和 C++ 检查之外，避免使用 `#ifdef` 和 `#ifndef`，尽量使用 `defined()` 和 `!defined()`。
+ 始终注释 `if/elif/else/endif` 语句。

```C
/* 正确 */
#if defined(XYZ)
/* ... */
#else /* defined(XYZ) */
/* ... */
#endif /* !defined(XYZ) */

/* 错误 */
#if defined(XYZ)
/* ... */
#else
/* ... */
#endif
```

+ 不要缩进 `#if` 的子语句。

```C
/* 正确 */
#if defined(XYZ)
#if defined(ABC)
/* ... */
#endif /* defined(ABC) */
#else /* defined(XYZ) */
/* ... */
#endif /* !defined(XYZ) */

/* 错误 */
#if defined(XYZ)
	#if defined(ABC)
		/* ... */
	#endif /* defined(ABC) */
#else /* defined(XYZ) */
	/* ... */
#endif /* !defined(XYZ) */
```

## 代码文档

编写有注释的代码可以让 Doxygen 解析并生成 HTML、PDF、$\LaTeX$ 输出，因此在项目初期正确地完成这项工作非常重要。

+ 对变量、函数、结构体和枚举使用支持 Doxygen 的文档风格
+ 始终使用 `@` 符号用于doxygen，不要使用 `\`
+ 文本始终从行首偏移 5x4 个空格（5 个制表符）

```C
/**
 * @brief           Holds pointer to first entry in linked list
 *                  Beginning of this text is 5 tabs (20 spaces) from beginning of line
 */
static type_t* list;
```

+ 结构体和枚举的每个成员都需要包含文档。
+ 将不同结构体或枚举的文档对齐。

```C
/**
 * @brief           This is point struct
 * @note            This structure is used to calculate all point
 *                      related stuff
 */
typedef struct {
    int32_t x;                                  /*!< Point X coordinate */
    int32_t y;                                  /*!< Point Y coordinate */
    int32_t size;                               /*!< Point size.
                                                    Since comment is very big,
                                                    you may go to next line */
} Point_t;

/**
 * @brief           Point color enumeration
 */
typedef enum {
    COLOR_RED,                                  /*!< Red color */
    COLOR_GREEN,                                /*!< Green color */
    COLOR_BLUE,                                 /*!< Blue color */
} PointColor_t;
```

+ 函数的注释必须写在源文件 `.c` 中。
+ 函数注释必须包括简介和所有参数的注释。
+ 每个参数都必须标记 `in` 或 `out` 用来表示它是否是输出参数。
+ 函数必须包含对返回值的注释，但 `void` 不必。
+ 函数可以包含其他 Doxygen 的关键字，例如 `note` 或 `warning`
+ 在参数名和描述之间使用冒号 `:`。

```C
/**
 * @brief           Sum `2` numbers
 * @param[in]       a: First number
 * @param[in]       b: Second number
 * @return          Sum of input values
 */
int32_t lSum(int32_t a, int32_t b) {
    return a + b;
}

/**
 * @brief           Sum `2` numbers and write it to pointer
 * @note            This function does not return value, it stores it to pointer instead
 * @param[in]       a: First number
 * @param[in]       b: Second number
 * @param[out]      result: Output variable used to save result
 */
void vSum(int32_t a, int32_t b, int32_t* result) {
    *result = a + b;
}
```

+ 如果函数返回了枚举的成员，使用 `ref` 来指定是哪一个枚举。

```C
/**
 * @brief           My enumeration
 */
typedef enum {
    MY_ERR,                                     /*!< Error value */
    MY_OK                                       /*!< OK value */
} MyEnum_t;

/**
 * @brief           Check some value
 * @return          @ref MY_OK on success, member of @ref my_enum_t otherwise
 */
MyEnum_t eCheckValue(void) {
    return MY_OK;
}
```

+ 对于常量和数字使用特殊标记。

```C
// 使用 `` 标记常量和数字
/**
 * @brief           Get data from input array
 * @param[in]       in: Input data
 * @return          Pointer to output data on success, `NULL` otherwise
 */
const void * vpGetData(const void* in) {
    return in;
}
```

## 头文件和源文件

+ 在文件的末尾添加空行。
+ 每个文件必须包含文件的 Doxygen 注释和简短的描述，并在后面添加空行。

```C
/**
 * @file            template.h
 * @brief           Template include file
 */
                    /* Here is empty line */
```

+ 每个文件必须包含许可证，使用单个星号。

```C
/**
 * @file            template.h
 * @brief           Template include file
 */

/*
 * Copyright (c) year FirstName LASTNAME
 *
 * Permission is hereby granted, free of charge, to any person
 * obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, sublicense, and/or sell copies of the Software,
 * and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be
 * included in all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 * AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
 * HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
 * OTHER DEALINGS IN THE SOFTWARE.
 *
 * This file is part of library_name.
 *
 * Author:          FirstName LASTNAME <optional_email@example.com>
 */
```

+ 头文件必须包含头文件保护以防止重复包含，防护符的格式为 `<项目>_<路径>_<文件名>_H_`。例如，`foo` 项目中的文件 `foo/src/bar/baz.h` 应该有如下防护：

```C
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_
...
#endif // FOO_BAR_BAZ_H_
```

+ 头文件需要包含所有需要的其余头文件以保证可以独立编译，如果头文件中包含了源文件需要的头文件，则应该在源文件中再包含一次，以表明其依赖关系。
+ 头文件的包含顺序：配套的头文件、C 语言标准库头文件、第三方库的头文件、本项目的其他头文件。
+ 禁止使用 Unix 的目录别名，例如 `.`（当前目录）、`..`（上级目录）、`~`（家目录）等。

```C
// google-awsome-project/src/foo/internal/fooserver.c
// 配套的头文件
#include "foo/server/fooserver.h"

// C 标准库头文件
#include <sys/types,h>
#include <unistd.h>

// 第三方库的头文件
#include "third_party/absl/flags/flag,h"

// 本项目的其他头文件
#include "base/basictypes.h"
#include "foo/server/bar.h"
```

+ 头文件必须包含对 C++ 的检查。
+ 在 C++ 检查之外包含头文件。
+ 在头文件中对需要在外部访问的全局变量使用 `extern` 申明，然后在源文件中定义它们。

```C
/* file.h ... */
#ifndef ...

extern int32_t my_variable; /* This is global variable declaration in header */

#endif

/* file.c ... */
int32_t my_variable;        /* Actually defined in source */
```

+ 禁止在一个 `.c` 文件中包含另外一个 `.c` 文件。
+ 头文件和源文件的代码应按照以下顺序进行：头文件的包含、`typedef` 重命名、宏定义、全局变量（外部变量）、（私有）函数原型、（内联）函数实现。
+ 源文件中，私有函数实现放在文件最后。

头文件示例：

```C
/**
 * @file            template.h
 * @brief           Template include file
 */

/*
 * Copyright (c) year FirstName LASTNAME
 *
 * Permission is hereby granted, free of charge, to any person
 * obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, sublicense, and/or sell copies of the Software,
 * and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be
 * included in all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 * AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
 * HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
 * OTHER DEALINGS IN THE SOFTWARE.
 *
 * This file is part of library_name.
 *
 * Author:          FirstName LASTNAME <optional_email@example.com>
 */

#ifndef TEMPLATE_HDR_H
#define TEMPLATE_HDR_H

/* 头文件的包含 */

#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */

/* typedef 重命名 */

/* 宏定义 */

/* 外部变量 */
    
/* 函数原型 */

/* 内联函数实现 */
    
#ifdef __cplusplus
}
#endif /* __cplusplus */

#endif /* TEMPLATE_HDR_H */
```

源文件示例：

```C
/**
 * @file            template.c
 * @brief           Template source file
 */

/*
 * Copyright (c) year FirstName LASTNAME
 *
 * Permission is hereby granted, free of charge, to any person
 * obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, sublicense, and/or sell copies of the Software,
 * and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be
 * included in all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 * AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
 * HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
 * OTHER DEALINGS IN THE SOFTWARE.
 *
 * This file is part of library_name.
 *
 * Author:          FirstName LASTNAME <optional_email@example.com>
 */

/* 头文件包含 */
/* 注意头文件包含的顺序 */
#include "template.h"

#include <stdbool.h>

/* typedef 重命名 */
typedef int int32_t;

/* 宏定义 */
#define PI 3.14159

/* 全局变量 */
int count; /* 全局变量不初始化 */

/* 私有函数原型 */
static int32_t prvSum(int32_t a, int32_t b);

/* 函数实现 */
bool bIsTrue(bool flag) {
    return flag;
}

/* 私有函数实现放在最后 */
static int32_t prvSum(int32_t a, int32_t b) {
    return a + b;
}
```

