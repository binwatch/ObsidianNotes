# 开天辟地的篇章: 最简单的计算机

大致了解上述的目录树之后, 你就可以开始阅读代码了. 至于从哪里开始, 就不用多费口舌了吧.

main 函数！

````ad-quote
There are two ways to use ccache. You can either prefix your compilation commands with ccache or you can let ccache masquerade as the compiler by creating a symbolic link (named as the compiler) to ccache. The first method is most convenient if you just want to try out ccache or wish to use it for some specific projects. The second method is most useful for when you wish to use ccache for all your compilations.

To use the first method, just make sure that ccache is in your PATH.

To use the second method on a Debian system, it’s easiest to just prepend _/usr/lib/ccache_ to your PATH. _/usr/lib/ccache_ contains symlinks for all compilers currently installed as Debian packages.

Alternatively, you can create any symlinks you like yourself like this:

```shell
ln -s /usr/bin/ccache /usr/local/bin/gcc
ln -s /usr/bin/ccache /usr/local/bin/g++
ln -s /usr/bin/ccache /usr/local/bin/cc
ln -s /usr/bin/ccache /usr/local/bin/c++
```

And so forth. This will work as long as the directory with symlinks comes before the path to the compiler (which is usually in _/usr/bin_). After installing you may wish to run “which gcc” to make sure that the correct link is being used.

**Warning**
The technique of letting ccache masquerade as the compiler works well, but currently doesn’t interact well with other tools that do the same thing. See USING CCACHE WITH OTHER COMPILER WRAPPERS.

**Warning**
Use a symbolic links for masquerading, not hard links.
````

So I add the following line to my _~/.bashrc_, it prepend _/usr/lib/ccache_ to my `PATH`.

```shell
export PATH=:/usr/lib/ccache:$PATH
```

If you want this works for `tmux`, also add the line to _~/.bash_profile_ or other files read by your login shell.

Use `source ~/.bashrc` to reload it, then use `which gcc` I found it works successfully:

> /usr/lib/ccache/gcc

```ad-info
`PATH` is an environmental variable in Linux and other Unix-like operating systems that tells the shell which directories to search for executable files in response to commands issued by a user. It increases both the _convenience_ and the _safety_ of such operating systems and is widely considered to be the single most important environmental variable.
```

````ad-note
title: [Difference between `VAR=...` (non-exported variables) and `export VAR=...` (exported variables)](https://unix.stackexchange.com/questions/3507/difference-between-shell-variables-which-are-exported-and-those-which-are-not-in)

- **Exported variables** are carried into the *environment* of commands executed by the shell that exported them
- **Non-exported variables** are *local* to the *current* shell invocation

From the `export` man page:

> The shell shall give the **export attribute** to the variables corresponding to the specified names, which shall *cause them to be in the environment of **subsequently** executed commands*.

- `env` is used to launch programs in a new environment, and with no arguments will output what that new environment would be
- `set` outputs the current environment, which includes any local non-exported variables.

Since `env` is creating a new environment, **only exported variables** are brought through, as is the case for any program launched from that shell.

For example, spawning a second shell within the first ((I used `<$` to represent prompts in the inner shell):

```shell
$ FOO=BAR
$ bash
<$ echo $FOO             # Note the empty line
<$ exit
$ export FOO
$ bash
<$ echo $FOO
BAR
<$
```

Note that **it's the variable that's exported**, **not just its value**. This means that once you `export FOO`, `FOO` becomes a **global variable** and shows up in subsequent environments, even if changed later.
````

````ad-warning
title: **\[Warning\]** 🪓 According to the manual, you need to change your `PATH` exports to **prepend** **not postfix**

exmaple of postfix:

```shell
export PATH=$PATH:/usr/lib/ccache
```

I will explain why below
````

```ad-tip
Use `type <command>` to see if it is an builtin command and use `man builtins` to read the manual of this command
```

````ad-note
title: [What is the difference between a symbolic link and a hard link?](https://stackoverflow.com/questions/185899/what-is-the-difference-between-a-symbolic-link-and-a-hard-link)

A **file** in the file system is basically *a link to an inode*.

![](https://raw.githubusercontent.com/binwatch/images/main/symbolic-and-hard-link.png)

Here is how we get to that picture:
1. Create a name `myfile.txt` in the file system that points to a new **inode** (which contains the _metadata for the file_ and _points to the blocks of data_ that contain its contents, i.e. the text "Hello, World!": 
	```shell
	$ echo 'Hello, World!' > myfile.txt
	```
2. Create a **hard link** `my-hard-link` to the file `myfile.txt`, which means "create a file that should point to the same **inode** that `myfile.txt` points to":
	```shell
	$ ln myfile.txt my-hard-link
	```
3. Create a **soft link** (symbolic link)`my-soft-link` to the file `myfile.txt`, which means "create a file that should point to the **file** `myfile.txt`":
	```shell
	$ ln -s myfile.txt my-soft-link
	```

Look what will now happen if `myfile.txt` is *deleted (or moved)*: 

- `my-hard-link` still points to the **same contents** (inode), and is thus unaffected, 
- whereas `my-soft-link` now points to **nothing**.

When you *delete a file*, it *removes **one** link to the underlying inode*. The inode is **only** deleted (or deletable/over-writable) when **all** links to the inode have been deleted.

**Note**: Hard links are only valid within the same File System. Symbolic links can span file systems as they are simply the name of another file.
````

```ad-note
title: **What is the difference between the 2 method: prepend path to `PATH` and create symlinks?**

The path _/usr/lib/ccache_ is a directory, use `ls` to list the (executable) files in it:

> c++cc gcc x86_64-linux-gnu-g++-11 c89-gcc g++ gcc-11 x86_64-linux-gnu-gcc c99-gcc g++-11 x86_64-linux-gnu-g++ x86_64-linux-gnu-gcc-11

That's why we have to prepend the path to `PATH`, for example, when we executing `gcc` command, system will search it in the paths of `PATH` one by one, util it finds the command (then stop). Prepending _/usr/lib/ccache_ to `PATH` makes sys encounter it before _/usr/local/bin/_.

The target of symlinks `/usr/bin/ccache` is an executable, the latter method creates "virtual" files _/usr/local/bin/..._ as symlinks that points to it. It works when _/usr/local/bin_ exists before _/usr/bin_ in your `PATH`(because these "real" compilers are in _/usr/bin_!).

[Here](https://unix.stackexchange.com/questions/8656/usr-bin-vs-usr-local-bin-on-linux) are more information about _/usr/bin_ and _/usr/local/bin_.
```

````ad-note
title: **What is `@echo`, `$@`, `$<` (...) in makefile?**

- **`@<command>`**: Normally `make` prints each line of the recipe before it is executed. We call this echoing because it gives the appearance that you are typing the lines yourself. *When a line starts with ‘ @ ‘, the echoing of that line is suppressed.* The ‘ @ ‘ is discarded before the line is passed to the shell.

From [Managing Projects with GNU Make, 3rd Edition, p. 16](http://uploads.mitechie.com/books/Managing_Projects_with_GNU_Make_Third_Edition.pdf):

```ad-quote
**Automatic variables** are set by `make` after a rule is matched. They provide access to elements from the target and prerequisite lists so you *don’t have to explicitly specify any filenames*. They are very useful for *avoiding code duplication*, but are critical when defining more general pattern rules.

There are seven “core” automatic variables:

- `$@`: The filename representing the *target*.
- `$%`: The filename element of an archive member specification.
- `$<`: The filename of the ***first** prerequisite*.
- `$?`: The names of ***all** prerequisites* that are _newer than the target_, separated by spaces.
- `$^`: The filenames of ***all** prerequisites*, separated by spaces. This list *has duplicate filenames removed* since for most uses, such as compiling, copying, etc., duplicates are not wanted.
- `$+`: Similar to `$^`, this is the names of all the prerequisites separated by spaces, except that `$+` *includes duplicates*. This variable was created for specific situations such as arguments to linkers where duplicate values have meaning.
- `$*`: The *stem of the target filename*. A stem is typically *a filename without its suffix*. Its use outside of pattern rules is discouraged.
```

In addition, each of the above variables has two **variants** for **compatibility** with other makes:

- One variant returns only the **directory portion** of the value. This is indicated by appending a “D” to the symbol, `$(@D)`, `$(<D)`, etc.
- The other variant returns only the **file portion** of the value. This is indicated by appending an “F” to the symbol, `$(@F)`, `$(<F)`, etc.

Note that these variant names are more than one character long and so must be enclosed in parentheses. GNU make provides a more readable alternative with the dir and notdir functions.
````

## RTFSC

### 配置系统和项目构建

#### 配置系统kconfig

```ad-quote
当你键入`make menuconfig`的时候, 背后其实发生了如下事件:

- <font color="#0FF">检查`nemu/tools/kconfig/build/mconf`程序是否存在, 若不存在, 则编译并生成`mconf`</font>
- <font color="#0FF">检查`nemu/tools/kconfig/build/conf`程序是否存在, 若不存在, 则编译并生成`conf`</font>
- <font color="#0DA">运行命令`mconf nemu/Kconfig`, 此时`mconf`将会解析`nemu/Kconfig`中的描述, 以菜单树的形式展示各种配置选项, 供开发者进行选择</font>
- <font color="#0DA">退出菜单时, `mconf`会把开发者选择的结果记录到`nemu/.config`文件中</font>
- <font color="#0A5">运行命令`conf --syncconfig nemu/Kconfig`, 此时`conf`将会解析`nemu/Kconfig`中的描述, 并读取选择结果`nemu/.config`, 结合两者来生成如下文件:</font>
	- <font color="#0A5">可以被包含到C代码中的宏定义(`nemu/include/generated/autoconf.h`), 这些宏的名称都是形如`CONFIG_xxx`的形式</font>
	- <font color="#0A5">可以被包含到Makefile中的变量定义(`nemu/include/config/auto.conf`)</font>
	- <font color="#0A5">可以被包含到Makefile中的, 和"配置描述文件"相关的依赖规则(`nemu/include/config/auto.conf.cmd`), 为了阅读代码, 我们可以不必关心它</font>
	- <font color="#0A5">通过时间戳来维护配置选项变化的目录树`nemu/include/config/`, 它会配合另一个工具`nemu/tools/fixdep`来使用, 用于在更新配置选项后节省不必要的文件编译, 为了阅读代码, 我们可以不必关心它</font>
```

- `mconf` 程序展示配置选项并将开发者选择的结果记录到 _.config_ 文件中
- `conf` 程序则根据解析 _Kconfig_ 的描述与 _.config_ 中的选项来生成一些宏与 Makefile 中变量定义、依赖规则等，以及维护配置选项变化的目录树

#### 项目构建和 Makefile

NEMU 的 Makefile 具备如下功能:

- **与配置系统进行关联 --** 通过包含`nemu/include/config/auto.conf`, 与 kconfig 生成的变量进行关联. 因此在通过 menuconfig 更新配置选项后, Makefile 的**行为可能也会有所变化**.
- **文件列表 (filelist) - -** 通过文件列表 (filelist) 决定**最终参与编译的源文件**. 在`nemu/src`及其子目录下存在一些名为`filelist.mk`的文件, 它们会根据 menuconfig 的配置维护一些变量，最终 Makefile 汇总得到参与编译的源文件的集合
- **编译和链接**

### 第一个客户程序

> NEMU在开始运行的时候, 首先会调用`init_monitor()`函数(在`nemu/src/monitor/monitor.c`中定义) 来进行一些和monitor相关的初始化工作.

*src/nemu-main.c* 中的 `main` 函数是入口地址

#### `parse_args(argc, argv)`

From [man 3 getopt_long](https://linux.die.net/man/3/getopt_long):

````ad-quote
```C
#include <unistd.h>

int getopt(int argc, char * const argv[],
           const char *optstring);

extern char *optarg;
extern int optind, opterr, optopt;
```
````

Notice the `optarg` char here.

````ad-quote
```C
#include <getopt.h>

int getopt_long(int argc, char * const argv[],
           const char *optstring,
           const struct option *longopts, int *longindex);
```

The `getopt()` function **parses the command-line arguments**. Its arguments `argc` and `argv` are the argument count and array as passed to the `main()` function on program invocation. *An element of argv that starts with '-' (and is not exactly "-" or "--") is an option element.* The characters of this element (aside from the initial '-') are option characters. *If `getopt()` is called repeatedly, it returns successively each of the option characters from each of the option elements.*
````

That why we see a while-loop in `parse_rags()`, it parses each option one by one.

```ad-quote
The variable `optind` is the *index of the next element* to be processed in `argv`. The system initializes this value to 1. The caller can reset it to 1 to restart scanning of the same _argv_, or when scanning a new argument vector.

*If `getopt()` finds another option character*, it **returns** that character, **updating** the *external* variable `optind` and a *static* variable `nextchar` so that the next call to `getopt()` can resume the scan with the following option character or `argv`-element.
```

Here man says it returns the character, but why the returned type is `int` instead of `char`? Is there any benefit?

> If there are **no more** option characters, `getopt()` returns `-1`. Then `optind` is the index in `argv` of the first `argv`-element that is **not** an option.

So it use int for convenience to express no more option characters while (`unsigned`) `char` ranging from 0 to 255. That's why the while condition:

```C
while ( (o = getopt_long(argc, argv, "-bhl:d:p:", table, NULL)) != -1) {
```

> `optstring` is a string containing the legitimate option characters. If such a character is followed by a colon, the option requires an argument

So the option in `table` with `required_argument` followed by a colon `:` in the invocation above

```C
const struct option table[] = {
    {"batch"    , no_argument      , NULL, 'b'},
    {"log"      , required_argument, NULL, 'l'},
    {"diff"     , required_argument, NULL, 'd'},
    {"port"     , required_argument, NULL, 'p'},
    {"help"     , no_argument      , NULL, 'h'},
    {0          , 0                , NULL,  0 },
};
```

> so `getopt()` places a pointer to the following text in the same `argv`-element, or the text of the following `argv`-element, in `optarg`.

So here use `optarg` to get the value of variables:

```C
case 'p': sscanf(optarg, "%d", &difftest_port); break;
case 'l': log_file = optarg; break;
case 'd': diff_so_file = optarg; break;
case 1: img_file = optarg; return 0;
```

> The `getopt_long()` function works like `getopt()` except that it **also accepts long options**, *started with two dashes*. (If the program accepts only long options, then `optstring` should be specified as an empty string (`""`), not `NULL`.)

The following struct `option` is just the type of element in `table` above:

````ad-quote
`longopts` is a pointer to the first element of an array of _struct_ `option` declared in *getopt.h* as

```C
struct option {
    const char *name;
    int         has_arg;
    int        *flag;
    int         val;
};
```

The meanings of the different fields are:
- `name`: the name of the long option.
- `has_arg`:
	- **no_argument** (or 0) if the option does not take an argument;
	- **required_argument** (or 1) if the option requires an argument;
	- **optional_argument** (or 2) if the option takes an optional argument.
- `flag`: specifies how results are returned for a long option.
	- If `flag` is `NULL`, then `getopt_long()` returns `val`. (For example, the calling program may set `val` to the equivalent short option character.)
	- Otherwise, `getopt_long()` returns 0, and `flag`:
		- points to a *variable* which is *set to* `val` if the option is found
		- left *unchanged* if the option is **not** found
- `val`: is the value to return, or to load into the variable pointed to by `flag`.
````

#### `init_rand()`

In _src/utils/rand.c_:

```C
void init_rand() {
  srand(MUXDEF(CONFIG_TARGET_AM, 0, time(0)));
}
```

In _include/macro.h_:

```C
// macro concatenation
#define concat_temp(x, y) x ## y
#define concat(x, y) concat_temp(x, y)

// macro testing
// See https://stackoverflow.com/questions/26099745/test-if-preprocessor-symbol-is-defined-inside-macro
#define CHOOSE2nd(a, b, ...) b
#define MUX_WITH_COMMA(contain_comma, a, b) CHOOSE2nd(contain_comma a, b)
#define MUX_MACRO_PROPERTY(p, macro, a, b) MUX_WITH_COMMA(concat(p, macro), a, b)
// define placeholders for some property
#define __P_DEF_0  X,
#define __P_DEF_1  X,
#define __P_ONE_1  X,
#define __P_ZERO_0 X,
// define some selection functions based on the properties of BOOLEAN macro
#define MUXDEF(macro, X, Y)  MUX_MACRO_PROPERTY(__P_DEF_, macro, X, Y)
```

```ad-hint
Notice the comma in macro `__P_DEF_x` here
``` 

Expand the macro in `init_rand()` to:

`CHOOSE2nd(__P_DEF_ ## CONFIG_TARGET_AM 0, time(0))`

```ad-info
title: **Token-pasting operator (##)**

Allows tokens used as actual arguments to be concatenated to form other tokens. It is often useful to merge two tokens into one while expanding macros. This is called token pasting or token concatenation. The ‘##’ pre-processing operator performs token pasting. When a macro is expanded, the two tokens on either side of each ‘##’ operator are combined into a single token, which then replaces the ‘##’ and the two original tokens in the macro expansion.
```

So it becomes:

`CHOOSE2nd(__P_DEF_CONFIG_TARGET_AM 0, time(0))`

From @[a3f](https://stackoverflow.com/users/1257035/a3f) 's answer to [Test if preprocessor symbol is defined inside macro](https://stackoverflow.com/questions/26099745/test-if-preprocessor-symbol-is-defined-inside-macro):

````ad-quote
Linux' [`kgconfig.h`](https://elixir.bootlin.com/linux/v4.16/source/include/linux/kconfig.h#L41) defines an `__is_defined` macro for this use case:

```C
 #define __ARG_PLACEHOLDER_1 0,
 #define __take_second_arg(__ignored, val, ...) val

/*
 * Helper macros to use CONFIG_ options in C/CPP expressions. Note that
 * these only work with boolean and tristate options.
 */

/*
 * Getting something that works in C and CPP for an arg that may or may
 * not be defined is tricky.  Here, if we have "#define CONFIG_BOOGER 1"
 * we match on the placeholder define, insert the "0," for arg1 and generate
 * the triplet (0, 1, 0).  Then the last step cherry picks the 2nd arg (a one).
 * When CONFIG_BOOGER is not defined, we generate a (... 1, 0) pair, and when
 * the last step cherry picks the 2nd arg, we get a zero.
 */
#define __is_defined(x)         ___is_defined(x)
#define ___is_defined(val)      ____is_defined(__ARG_PLACEHOLDER_##val)
#define ____is_defined(arg1_or_junk)    __take_second_arg(arg1_or_junk 1, 0)
```

It's C99 and works for tristate options (undefined, defined to 0, defined to 1).
````

From [man 2 time:](https://man7.org/linux/man-pages/man2/time.2.html)

````ad-quote
```C
#include <time.h>

time_t time(time_t *tloc);
```

`time()` returns the time as the number of seconds since the Epoch, 1970-01-01 00:00:00 +0000 (UTC).

If tloc is non-NULL, the return value is also stored in the memory pointed to by tloc.
````

```ad-question
title: **\[Doubt\]** 🤔
我的理解是这里如果定义了 `CONFIG_TARGET_AM` 宏，就用 `srand(0)` 初始化，如果没有定义，就用 `time(0)` 初始化
```

#### `init_log(log_file)`

In _src/utils/log.c_:

```C
void init_log(const char *log_file) {
  log_fp = stdout;
  if (log_file != NULL) {
    FILE *fp = fopen(log_file, "w");
    Assert(fp, "Can not open '%s'", log_file);
    log_fp = fp;
  }
  Log("Log is written to %s", log_file ? log_file : "stdout");
}
```

Try to open the log file named _log_file_ as writable, if cannot open it, log to _stdout_.

In _include/debug.h_:

```C
#define Log(format, ...) \
    _Log(ANSI_FMT("[%s:%d %s] " format, ANSI_FG_BLUE) "\n", \
        __FILE__, __LINE__, __func__, ## __VA_ARGS__)
```

In _include/utils.h_:

```C
#define ANSI_NONE       "\33[0m"

#define ANSI_FMT(str, fmt) fmt str ANSI_NONE

#define log_write(...) IFDEF(CONFIG_TARGET_NATIVE_ELF, \
  do { \
    extern FILE* log_fp; \
    extern bool log_enable(); \
    if (log_enable()) { \
      fprintf(log_fp, __VA_ARGS__); \
      fflush(log_fp); \
    } \
  } while (0) \
)

#define _Log(...) \
  do { \
    printf(__VA_ARGS__); \
    log_write(__VA_ARGS__); \
  } while (0)
```

`````ad-note
title: **Why `do {...} while(0)`?**

I noticed that here the macro use `do {...} while(0)`，looks a little bit strange?

Here is a good question [Why use apparently meaningless do-while and if-else statements in macros?](https://stackoverflow.com/questions/154136/why-use-apparently-meaningless-do-while-and-if-else-statements-in-macros) in which @[Lundin](https://stackoverflow.com/users/584518/lundin) asked:

````ad-quote
In many C/C++ macros I'm seeing the code of the macro wrapped in what seems like a meaningless `do while` loop. Here are examples.

```C
#define FOO(X) do { f(X); g(X); } while (0)
#define FOO(X) if (1) { f(X); g(X); } else
```

I can't see what the `do while` is doing. Why not just write this without it?

```C
#define FOO(X) f(X); g(X)
```
````

😃 Notice the omitted semicolon here, when using macro as a statement.

@[Pavel](https://stackoverflow.com/users/650884/pavel)'s pretty answer with examples to it:

````ad-quote
The `do ... while` and `if ... else` are there to **make it so that a semicolon after your macro always means the same thing**. Let's say you had something like your second macro.

```C
#define BAR(X) f(x); g(x)
```

Now if you were to use `BAR(X);` in an `if ... else` statement, where the bodies of the if statement were not wrapped in curly brackets, you'd get a bad surprise.

```C
if (corge)
  BAR(corge);
else
  gralt();
```

The above code would expand into

```C
if (corge)
  f(corge); g(corge);
else
  gralt();
```

which is **syntactically incorrect, as the else is no longer associated with the if**. It doesn't help to wrap things in curly braces within the macro, because **a semicolon after the braces is syntactically incorrect**.

```C
if (corge)
  {f(corge); g(corge)};
else
  gralt();
```

There are two ways of fixing the problem. The first is to **(SOLUTION1)** **use a comma to sequence statements within the macro without robbing it of its ability to act like an expression**.

```C
#define BAR(X) f(x), g(x)
```

The above version of bar `BAR` expands the above code into what follows, which is syntactically correct.

```C
if (corge)
  f(corge), g(corge);
else
  gralt();
```

**(BUT)** **This doesn't work if instead of** **`f(X)`** **you have a more complicated body of code that needs to go in its own block**, say for example to declare local variables. In the most general case the solution is to use something like `do ... while` to cause the macro to be a single statement that takes a semicolon without confusion.

```C
#define BAR(X) do { \
  int i = f(X); \
  if (i > 4) g(i); \
} while (0)
```

You don't have to use `do ... while`, you could cook up something with `if ... else` as well, although when `if ... else` expands inside of an `if ... else` it leads to a "[dangling else](http://en.wikipedia.org/wiki/Dangling_else)", which could make an existing dangling else problem even harder to find, as in the following code.

```C
if (corge)
  if (1) { f(corge); g(corge); } else;
else
  gralt();
```
````
````ad-summary
title: 个人总结
使用 `do {...} while(0)` 或者 `if {...} else` 可以避免将宏作为语句时，添加分号引起的语法错误和语义错误，并且能够表达复杂的代码块（与使用逗号分隔相比）

但是在 `if-else` 语句中使用`if {...} else` 结构定义的宏也会引起 dangling else 的问题，产生二义性，因此最推荐 `do {...} while(0)` 的用法

当然，可能更推荐将需要重复使用的复杂的语句声明为函数而不是宏——[Do macros in C++ improve performance?](https://stackoverflow.com/questions/36212455/do-macros-in-c-improve-performance) 和 [Inline functions vs Preprocessor macros](https://stackoverflow.com/questions/1137575/inline-functions-vs-preprocessor-macros) 中有很多解释与例子。宏没有类型检查，并且通常更难以得到期望的结果、调试与维护，也可能产生副作用。
````
`````

#### `init_mem()`

In _src/memory/paddr.c_:

```C
void init_mem() {
#if   defined(CONFIG_PMEM_MALLOC)
  pmem = malloc(CONFIG_MSIZE);
  assert(pmem);
#endif
#ifdef CONFIG_MEM_RANDOM
  uint32_t *p = (uint32_t *)pmem;
  int i;
  for (i = 0; i < (int) (CONFIG_MSIZE / sizeof(p[0])); i ++) {
    p[i] = rand();
  }
#endif
  Log("physical memory area [" FMT_PADDR ", " FMT_PADDR "]", PMEM_LEFT, PMEM_RIGHT);
}
```

1. If `CONFIG_PMEM_MALLOC` is defined, allocate _CONFIG_MSIZE_ spaces
2. If `CONFIG_MEM_RANDOM` is defined, randomize the data in memory just allocated in the unit of `uint32` size

`````ad-note
title: **Difference between `#if defined()` and `#ifdef`**

From @[user44556](https://stackoverflow.com/users/44556/user44556) and @[truthadjustr](https://stackoverflow.com/users/2856202/truthadjustr)'s answer to the question: [difference between #if defined(WIN32) and #ifdef(WIN32)](https://stackoverflow.com/questions/1714245/difference-between-if-definedwin32-and-ifdefwin32)

````ad-quote
- `#if defined(NAME)` can do **compound conditionals**
- `#ifdef` can **only use a single condition**

For example in your case:

```C
#if defined(WIN32) && !defined(UNIX)
/* Do windows stuff */
#elif defined(UNIX) && !defined(WIN32)
/* Do linux stuff */
#else
/* Error, both can't be defined or undefined same time */
#endif
```
````
`````

#### 三个对调试有用的宏

在 _nemu/include/debug.h_ 中定义

```C
#define Log(format, ...) \
    _Log(ANSI_FMT("[%s:%d %s] " format, ANSI_FG_BLUE) "\n", \
        __FILE__, __LINE__, __func__, ## __VA_ARGS__)

#define Assert(cond, format, ...) \
  do { \
    if (!(cond)) { \
      MUXDEF(CONFIG_TARGET_AM, printf(ANSI_FMT(format, ANSI_FG_RED) "\n", ## __VA_ARGS__), \
        (fflush(stdout), fprintf(stderr, ANSI_FMT(format, ANSI_FG_RED) "\n", ##  __VA_ARGS__))); \
      IFNDEF(CONFIG_TARGET_AM, extern FILE* log_fp; fflush(log_fp)); \
      extern void assert_fail_msg(); \
      assert_fail_msg(); \
      assert(cond); \
    } \
  } while (0)

#define panic(format, ...) Assert(0, format, ## __VA_ARGS__)
```

`````ad-note
title: **About `__VA_ARGS__ 可变参数`**

From @[nos](https://stackoverflow.com/users/126769/nos) and @[phuclv](https://stackoverflow.com/users/995714/phuclv)'s answer to [What does __VA_ARGS__ in a macro mean?](https://stackoverflow.com/questions/26053959/what-does-va-args-in-a-macro-mean):

````ad-quote
It's a [variadic](http://en.wikipedia.org/wiki/Variadic_macro) macro. It means you can call it with any number of arguments. The three `...` is similar to the same construct used in a [variadic function](http://en.wikipedia.org/wiki/Variadic_function) in C

That means you can use the macro like this

```C
DEBUG("foo", "bar", "baz");
```
Or with any number of arguments.

The `__VA_ARGS__` refers back again to the variable arguments in the macro itself.

```C
#define DEBUG(...)  printString (__VA_ARGS__)
               ^                     ^
               +-----<-refers to ----+
```

So `DEBUG("foo", "bar", "baz");` would be replaced with `printString ("foo", "bar", "baz")`
````
`````

#### `Log()`

In _include/utils.h_:

```C
#define ANSI_NONE       "\33[0m"
#define ANSI_FMT(str, fmt) fmt str ANSI_NONE

#define log_write(...) IFDEF(CONFIG_TARGET_NATIVE_ELF, \
  do { \
    extern FILE* log_fp; \
    extern bool log_enable(); \
    if (log_enable()) { \
      fprintf(log_fp, __VA_ARGS__); \
      fflush(log_fp); \
    } \
  } while (0) \
)

#define _Log(...) \
  do { \
    printf(__VA_ARGS__); \
    log_write(__VA_ARGS__); \
  } while (0)

```

```ad-tip
title: **Linux console 的颜色控制**

在字符串前使用转义控制序列 `\033[<special-attribute>;<backgroun-color>;<font-color>m` 可以控制显示字符的特殊属性、背景颜色、字体颜色状态，用 `echo -e` 命令可以显示转义效果。

另外需要在字符串末尾加上 `"\033[0m"` 恢复默认状态
```

`Log()` 会输出其被调用时所在的源文件, 行号和函数（注意其中的 `"[%s:%d %s] "` 格式化字符串和 `__FILE__`、`__LINE__`、`__func__` 参数为 C 的预定义宏）

```C
#define Log(format, ...) \
    _Log(ANSI_FMT("[%s:%d %s] " format, ANSI_FG_BLUE) "\n", \
        __FILE__, __LINE__, __func__, ## __VA_ARGS__)
```

#### `Assert()`

当测试条件 `cond` 为假时, 在 assertion fail 之前输出一些信息

#### `panic()`

以 `cond = 0` “调用” `Assert()` 宏，输出信息并结束程序, 相当于无条件的 assertion fail


monitor 负责将客户程序读入到客户计算机中

1. NEMU 在开始运行的时候, 首先会调用 `init_monitor()`函数 (在`nemu/src/monitor/monitor.c`中定义) 来进行一些和 **monitor 相关的初始化工作**.
2. 接下来 monitor 会调用`init_isa()`函数(在`nemu/src/isa/$ISA/init.c`中定义), 来进行一些**ISA相关的初始化工作**.
	1. 第一项工作就是将一个内置的**客户程序读入到内存**中
	2. `init_isa()`的第二项任务是**初始化寄存器**, 这是通过`restart()`函数来实现的
3. NEMU 返回到`init_monitor()`函数中, 继续调用`load_img()`函数 (在`nemu/src/monitor/monitor.c`中定义). 这个函数会将**一个有意义的客户程序从镜像文件读入到内存, 覆盖刚才的内置客户程序**. 这个镜像文件是运行 NEMU 的一个可选参数, 在运行 NEMU 的命令中指定. 如果运行 NEMU 的时候没有给出这个参数, NEMU 将会运行内置客户程序.
4. ...
5. 最后 monitor 会调用`welcome()`函数输出欢迎信息
6. Monitor 的初始化工作结束后, `main()`函数会继续调用`engine_start()`函数 (在`nemu/src/engine/interpreter/init.c`中定义). 代码会进入**简易调试器**(Simple Debugger)的主循环`sdb_mainloop()` (在`nemu/src/monitor/sdb/sdb.c`中定义), 并输出NEMU的命令提示符. **简易调试器是 monitor 的核心功能**
	- 在命令提示符后键入`c`后, NEMU开始进入指令执行的主循环`cpu_exec()` (在`nemu/src/cpu/cpu-exec.c`中定义). `cpu_exec()`又会调用`execute()`, 后者模拟了CPU的工作方式: 不断执行指令. 具体地, 代码将在一个for循环中不断调用`exec_once()`函数, 这个函数的功能就是我们在上一小节中介绍的内容: 让CPU执行当前PC指向的一条指令, 然后更新PC.
7. NEMU 将不断执行指令, 直到遇到以下情况之一, 才会退出指令执行的循环:
	- 达到要求的循环次数.
	- 客户程序执行了`nemu_trap`指令. 这是一条虚构的特殊指令, 它是为了在NEMU中让客户程序指示执行的结束而加入的, NEMU在ISA手册中选择了一些用于调试的指令, 并将`nemu_trap`的特殊含义赋予它们. 例如在riscv32的手册中, NEMU选择了`ebreak`指令来充当`nemu_trap`. 为了表示客户程序是否成功结束, `nemu_trap`指令还会接收一个表示结束状态的参数. 当客户程序执行了这条指令之后, NEMU将会根据这个结束状态参数来设置NEMU的结束状态, 并根据不同的状态输出不同的结束信息, 主要包括
		- `HIT GOOD TRAP` - 客户程序正确地结束执行
		- `HIT BAD TRAP` - 客户程序错误地结束执行
		- `ABORT` - 客户程序意外终止, 并未结束执行
8. ?

sdb 中输入 c 后调用的函数：

```C
static int cmd_c(char *args) {
  cpu_exec(-1);
  return 0;
}
```



```C
/* Simulate how the CPU works. */
void cpu_exec(uint64_t n) {
  // some code ...

  execute(n);

  // some code ...
  }
}

static void execute(uint64_t n) {
  Decode s;
  for (;n > 0; n --) {
    exec_once(&s, cpu.pc);
    g_nr_guest_inst ++;
    trace_and_difftest(&s, cpu.pc);
    if (nemu_state.state != NEMU_RUNNING) break;
    IFDEF(CONFIG_DEVICE, device_update());
  }
}

static void exec_once(Decode *s, vaddr_t pc) {
  s->pc = pc;
  s->snpc = pc;
  isa_exec_once(s);
  cpu.pc = s->dnpc;
#ifdef CONFIG_ITRACE
  char *p = s->logbuf;
  p += snprintf(p, sizeof(s->logbuf), FMT_WORD ":", s->pc);
  int ilen = s->snpc - s->pc;
  int i;
  uint8_t *inst = (uint8_t *)&s->isa.inst.val;
  for (i = ilen - 1; i >= 0; i --) {
    p += snprintf(p, 4, " %02x", inst[i]);
  }
  int ilen_max = MUXDEF(CONFIG_ISA_x86, 8, 4);
  int space_len = ilen_max - ilen;
  if (space_len < 0) space_len = 0;
  space_len = space_len * 3 + 1;
  memset(p, ' ', space_len);
  p += space_len;

  void disassemble(char *str, int size, uint64_t pc, uint8_t *code, int nbyte);
  disassemble(p, s->logbuf + sizeof(s->logbuf) - p,
      MUXDEF(CONFIG_ISA_x86, s->snpc, s->pc), (uint8_t *)&s->isa.inst.val, ilen);
#endif
}
```

## 蓝框思考题

### 究竟要执行多久?

> 在`cmd_c()`函数中, 调用`cpu_exec()`的时候传入了参数`-1`, 你知道这是什么意思吗?

```C
static void execute(uint64_t n) {
  // ...
  for (;n > 0; n --) {
    // ...
  }
}
```

`cpu_exec()` 将这个 -1 传递给 `execute()` 函数，而它们接收的参数都是无符号类型 `uint64_t`,会强制将 -1 转换成最大的无符号数，而 `execute()` 函数从这个最大的无符号数开始循环直到 0，执行指令

### 潜在的威胁 (建议二周目思考)

> "调用`cpu_exec()`的时候传入了参数`-1`", 这一做法属于未定义行为吗? 请查阅C99手册确认你的想法.

