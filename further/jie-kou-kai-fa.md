# 接口开发

## API和SPI的区别

API（应用程序接口）和 SPI（服务提供者接口）是两种不同类型的接口概念，尽管它们在软件设计中都起着重要作用，但它们的用途和设计目标有所不同。下面是它们的关系和区别：

#### API（Application Programming Interface）

* **定义**：API 是应用程序接口，是一组定义了如何与软件模块、库或服务进行交互的规则和规范。API 允许不同的软件组件之间进行通信。
* **用途**：API 主要用于暴露功能给外部开发者或系统使用，使他们能够通过预定义的方式调用这些功能。
* **示例**：操作系统的文件读写 API，Java 的 `List` 接口，Web 服务的 REST API。
* **典型场景**：一个应用程序使用其他库、框架或服务的功能，例如调用一个外部服务来获取数据，或使用一个库中的类进行操作。

#### SPI（Service Provider Interface）

* **定义**：SPI 是服务提供者接口，是一组用于提供或扩展服务的接口或抽象类。SPI 定义了如何由第三方实现和提供某种特定功能或服务。
* **用途**：SPI 主要用于让第三方开发者或模块提供自定义的实现，通常是为了扩展框架或平台的功能。SPI 是面向提供者的，而 API 是面向使用者的。
* **示例**：Java 中的 `JDBC` 是一个典型的 SPI，数据库厂商提供自己的 `JDBC` 驱动实现。Java 的 `ServiceLoader` 机制也是实现 SPI 的一种方式。
* **典型场景**：开发者创建一个框架，并允许其他开发者通过实现 SPI 来扩展框架的功能，例如提供自定义的日志记录器或数据库连接池。

#### 关系与区别

* **设计目标**：
  * **API**：定义供外部用户使用的功能，强调如何使用功能。
  * **SPI**：定义供服务提供者实现的接口，强调如何扩展或实现功能。
* **面向对象**：
  * **API**：面向客户端或应用程序使用者。
  * **SPI**：面向服务提供者或扩展者。
* **使用场景**：
  * **API**：开发者调用 API 来使用已有功能。
  * **SPI**：开发者实现 SPI 来提供或扩展功能。

#### 总结

API 是软件系统与外界沟通的桥梁，用于调用功能。而 SPI 是为扩展系统提供定制实现的接口，用于增加新功能或替代默认实现。API 和 SPI 通常在一起工作，例如，一个框架通过 API 提供使用方式，通过 SPI 定制其行为。

## 需求

阅读接口wiki和提供的python接口信息，疑问：websocket?http?命名？项目位置？生成位置？实现接口继承什么基类？user/password?whl放在工程里？/安装好lib复制到工程里？自定义类型转换成Python类型？

websocke和http都需要，因为功能不重叠。算法和普通交易接口均继承，user\password给了测试的，whl放工程里生成时安装，自定义类型根据typedef原定义即可，不需要预先考虑变更。

命名？HPI

成员变量按需要套用，自定义结构按需要套用。

CMessageQueue需要用的时候再用，暂时不需要。

暂时不考虑交易成功但查询更新时的延迟，先假定查询的account都是最新的。

return -4和onRspxxx二选一。

异步、同步？http部分全到spi中？异步通信。

由于无法查询期初持仓，只能之后通过交易单反推。

login初期就查好account\_id,初始化websocketclient

Max\_token因为是算法单不需要管

用一个布尔值来使得回调前查完必要的信息，拖住回调。

如何区分根据类型long,short?不需要，先buy和sell

时间应该是返回的值还是本地时间的值？日期？

qryposition ATO没有填InvestorId.

localid传sumbitid，子单控制不了。查两种母单，再查子单

组合开平标志简单和Direction一致。

母单不需要limitPrice.母单应该用InstructionField，没有开平标志等等。。

没有提供报单时间，可能要自己维护，记录第一次变成A的时间点。记录第一个last\_update\_time的时间点。

Trade,InstructionInsert,Cancel,OnRtn,OnRtn。

先查母单再查Trade是为了绑定OrderRef，而正常再查Trade前就调用了查Order/初始化时……所以不用在每次查之前查母单

create\_batch\_tasks与非batch性能没有区别，用create\_tasks。不需要future。T0和非T0，规范写完后用wiki。

limit\_price?remark?算法单报单错误处理？

InstructionBindSubmitId,externalId?

Cancel回调？不回调。

RtnInstruction应该会在wbsocket中回调

记录submitid是否非TO,查询、报单、撤单成功返回时均需刷新。加不加锁看websocket的OrderRtn线程关系。先写websocket

## gc-sections

`-Wl,-gc-sections` 是用于控制链接器行为的编译选项，常见于 GCC 和 Clang 编译器中。

#### 1. **`-Wl,`前缀**

* `-Wl,` 是一个传递选项，它告诉编译器将后面的参数直接传递给链接器（`ld`）。
* `Wl` 是 “**pass to linker**” 的缩写，逗号分隔后面的实际参数。

#### 2. **`-gc-sections`**

* `-gc-sections` 是链接器的选项，表示 "**garbage collect sections**"（垃圾收集段）。
* 当启用 `-gc-sections` 时，链接器会移除所有未被引用的代码段（如函数）和数据段（如变量）。这有助于减小最终生成的可执行文件或库的体积。

#### 3. **组合起来的含义**

`-Wl,-gc-sections` 表示在链接阶段，启用垃圾收集段功能。具体来说，编译器会将 `-gc-sections` 传递给链接器，链接器则会在构建可执行文件或库时剔除未使用的代码和数据段。

#### 4. **使用场景**

* **嵌入式系统**：特别适用于嵌入式开发中，减小代码尺寸至关重要。例如，去除未使用的函数和数据以节省存储空间。
* **优化可执行文件大小**：在一般应用中，启用此选项可以帮助减小二进制文件的大小。

## pybind11文档阅读

py::types默认初始化时只是空指针，python中没有对应类型。用py::none初始化其它类型会错误/改变类型至None

转型失败会在python和C++中抛出错误

Unpacking of \*args and \*\*kwargs is also possible and can be mixed with other arguments

The implementation in pybind11/iostream.h is NOT thread safe. Multiple threads writing to a redirected ostream concurrently cause data races and potentially buffer overflows.

auto message = "Hello, {name}! The answer is {number}"\_s.format(\*\*kwargs);

The Python interpreter shuts down when scoped\_interpreter is destroyed. After this, creating a new instance will restart the interpreter. Alternatively, the initialize\_interpreter / finalize\_interpreter pair of functions can be used to directly set the state at any time.

Modules created with pybind11 can be safely re-initialized after the interpreter has been restarted. However, this may not apply to third-party extension modules. The issue is that Python itself cannot completely unload extension modules and there are several caveats with regard to interpreter restarting. In short, not all memory may be freed, either due to Python reference cycles or user-created global data. All the details can be found in the CPython documentation.

Warning: Creating two concurrent scoped\_interpreter guards is a fatal error. So is calling initialize\_interpreter for a second time after the interpreter has already been initialized. Donot use the raw CPython API functions Py\_Initialize and Py\_Finalize as these do not properly handle the lifetime of pybind11’s internal data.

不能有两个解释器，所以static变量给spi和api共享

Creating multiple copies of scoped\_interpreter is not possible because it represents the main Python interpreter. Sub-interpreters are something different and they do permit the existence of multiple interpreters. This is an advanced feature of the CPython API and should be handled with care. pybind11 does not currently offer a C++ interface for sub-interpreters, so refer to the CPython documentation for all the details regarding this feature.

次级解释器得用CPython的API

The classes gil\_scoped\_release and gil\_scoped\_acquire can be used to acquire and release the global inter preter lock in the body of a C++ function call. In this way, long-running C++ code can be parallelized using multiple Python threads, but great care must be taken when any gil\_scoped\_release appear: if there is any way that the C++ code can access Python objects, gil\_scoped\_acquire should be used to reacquire the GIL. Taking Overriding virtual functions in Python as an example, this could be realized as follows (important changes highlighted):

c++调用Python中def的函数（call）的时候会自动获取GIL,Python调C++的时候也会（但可以手动释放），需要C++长期运行时可以release再最后acquire，防止访问Python对象竞争

* Do you have any global variables that are pybind11 objects or invoke pybind11 functions in either their constructor or destructor? You are generally not allowed to invoke any Python function in a global static context. We recommend using lazy initialization and then intentionally leaking at the end of the program.

没有解释器环境下的初始化、Python函数调用会违背GIL?

* Do you have any pybind11 objects that are members of other C++ structures? One commonly overlooked requirement is that pybind11 objects have to increase their reference count whenever their copy constructor is called. Thus, you need to be holding the GIL to invoke the copy constructor of any C++ class that has a pybind11 member. This can sometimes be very tricky to track for complicated programs Think carefully when you make a pybind11 object a member in another struct.

pybind11 对象的复制是C++层面的复制，得用GIL(PythonC对象的复制不会call GIL?)

* C++ destructors that invoke Python functions can be particularly troublesome as destructors can sometimes get invoked in weird and unexpected circumstances as a result of exceptions.

由于析构可能被异常触发，这里对Python函数的调用的GIL获取可能陷入死锁（异常处理前解释器被析构？）

* You should try running your code in a debug build. That will enable additional assertions within pybind11 that will throw exceptions on certain GIL handling errors (reference counting operations).

用debug模式构建，检测引用计数操作

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>



## rsync同步

`rsync` 是一个功能强大的工具，用于在本地和远程之间同步文件和目录。它的使用灵活多样，下面是 `rsync` 的基础用法及常见的几个场景。

#### **基础用法**

```bash
rsync [选项] 源路径 目标路径
```

* **源路径**: 要复制的文件或目录，可以是本地路径或远程路径。
* **目标路径**: 目标文件或目录的路径，也可以是本地或远程路径。

#### **常见选项**

* `-a` : 归档模式，递归复制目录并保留符号链接、权限、时间戳等。
* `-v` : 显示详细输出信息。
* `-z` : 在传输过程中压缩文件。
* `-P` : 显示传输进度并保留部分传输的文件，以便中断后可以继续传输。
* `--delete` : 删除目标目录中在源目录中不存在的文件，使目标目录与源目录完全一致。
* `-r` : 递归复制目录（已包含在 `-a` 选项中）。
* `-e ssh` : 通过 SSH 进行传输，用于加密和认证。

#### **本地同步**

1.  **将一个目录同步到另一个目录**:

    ```bash
    rsync -av /path/to/source/ /path/to/destination/
    ```

    * 将 `/path/to/source/` 目录下的所有内容复制到 `/path/to/destination/` 目录下。
    * 注意：结尾的 `/` 表示目录的内容，而不包含该目录本身。

#### **远程同步**

2.  **将本地目录同步到远程服务器**:

    ```bash
    rsync -av /path/to/source/ user@remote_host:/path/to/destination/
    ```

    * 将本地目录 `/path/to/source/` 同步到远程服务器 `remote_host` 上的 `/path/to/destination/` 目录中。
    * 通过 SSH 进行安全的传输。
3.  **从远程服务器同步到本地**:

    ```bash
    rsync -av user@remote_host:/path/to/source/ /path/to/destination/
    ```

    * 将远程服务器 `remote_host` 上的 `/path/to/source/` 目录同步到本地 `/path/to/destination/` 目录中。

#### **增量同步和删除**

4.  **增量同步并删除目标中多余的文件**:

    ```bash
    rsync -av --delete /path/to/source/ /path/to/destination/
    ```

    * 同步时删除目标目录中在源目录中已删除的文件，使目标目录与源目录完全一致。

#### **使用示例**

1.  **本地文件夹同步**:

    ```bash
    rsync -av /home/user/documents/ /backup/documents/
    ```
2.  **通过 SSH 将本地文件夹同步到远程服务器**:

    ```bash
    rsync -avz -e ssh /home/user/documents/ user@remote_server:/backup/documents/
    ```
3.  **从远程服务器同步到本地**:

    ```bash
    rsync -avz -e ssh user@remote_server:/backup/documents/ /home/user/documents/
    ```
4.  **显示进度并压缩传输数据**:

    ```bash
    rsync -avzP /home/user/movies/ user@remote_server:/backup/movies/
    ```

#### **总结**

`rsync` 是一个非常强大的工具，适用于本地和远程之间的高效文件同步和备份。通过不同的选项，你可以根据需要优化传输速度、保留文件属性、显示进度、删除目标中的多余文件等。

`rsync` 判断文件是否发生变化主要基于以下几个标准：

#### **1. 文件大小**

`rsync` 首先比较源文件和目标文件的大小。如果文件大小不同，则认为文件发生了变化，需要同步。

#### **2. 文件修改时间戳**

`rsync` 默认还会检查文件的修改时间戳（mtime）。如果源文件的修改时间比目标文件的新，或者文件大小不同，那么 `rsync` 会认为文件发生了变化，并将其同步到目标位置。

#### **3. 校验和（可选）**

虽然默认情况下 `rsync` 主要依赖文件大小和修改时间戳来判断文件是否发生变化，但你也可以使用 `-c` 选项启用基于文件内容校验和的比较。启用此选项时，`rsync` 会计算源文件和目标文件的校验和（通常是 MD5 或 SHA1），并根据校验和是否相同来决定是否同步文件。

```bash
rsync -avc /path/to/source/ /path/to/destination/
```

**注意**：启用 `-c` 选项会增加同步的时间，因为计算校验和需要额外的处理。

#### **总结**

默认情况下，`rsync` 通过文件大小和修改时间戳来判断文件是否发生变化，这是一个快速且高效的方法。如果需要更精确的比较，可以使用 `-c` 选项进行基于校验和的比较，但这会增加同步的时间成本。

## std::tm

`std::tm` 是 C++ 标准库中用于表示时间和日期的结构体。它通常用于与时间相关的函数，例如格式化时间、解析时间字符串、计算时间差等。这个结构体包含了年、月、日、时、分、秒等时间信息。

#### **`std::tm` 结构体的成员**

`std::tm` 的定义通常如下：

```cpp
struct tm {
    int tm_sec;   // 秒，范围为 [0, 60]，包括闰秒
    int tm_min;   // 分，范围为 [0, 59]
    int tm_hour;  // 小时，范围为 [0, 23]
    int tm_mday;  // 一个月中的天数，范围为 [1, 31]
    int tm_mon;   // 月，范围为 [0, 11]，0 表示一月
    int tm_year;  // 年，从 1900 年起计算，比如 2024 年表示为 124
    int tm_wday;  // 一周中的天数，范围为 [0, 6]，0 表示星期天
    int tm_yday;  // 一年中的天数，范围为 [0, 365]
    int tm_isdst; // 夏令时标志，正值表示夏令时，0 表示非夏令时，负值表示未知
};
```

#### **使用 `std::tm` 的常见场景**

1.  **获取当前时间并将其转换为 `std::tm`：**

    ```cpp
    #include <iostream>
    #include <ctime>

    int main() {
        std::time_t t = std::time(nullptr);  // 获取当前时间
        std::tm* now = std::localtime(&t);   // 将 time_t 转换为 tm 结构体

        std::cout << "Year: " << (now->tm_year + 1900) << "\n";
        std::cout << "Month: " << (now->tm_mon + 1) << "\n";
        std::cout << "Day: " << now->tm_mday << "\n";
        std::cout << "Time: " << now->tm_hour << ":" 
                  << now->tm_min << ":" << now->tm_sec << "\n";

        return 0;
    }
    ```

    这个例子中，`std::localtime` 将当前时间转换为本地时间的 `std::tm` 结构体。
2.  **将 `std::tm` 转换为字符串：**

    ```cpp
    #include <iostream>
    #include <ctime>
    #include <iomanip>

    int main() {
        std::time_t t = std::time(nullptr);
        std::tm* now = std::localtime(&t);

        // 格式化输出时间
        std::cout << std::put_time(now, "%Y-%m-%d %H:%M:%S") << std::endl;

        return 0;
    }
    ```

    在这个例子中，`std::put_time` 用于将 `std::tm` 格式化为字符串，输出为 `"YYYY-MM-DD HH:MM:SS"` 格式。
3.  **将字符串解析为 `std::tm`：**

    ```cpp
    #include <iostream>
    #include <sstream>
    #include <ctime>

    int main() {
        std::tm tm = {};
        std::istringstream ss("2024-08-21 14:30:00");
        ss >> std::get_time(&tm, "%Y-%m-%d %H:%M:%S");

        if (ss.fail()) {
            std::cerr << "Failed to parse time\n";
        } else {
            std::cout << "Year: " << (tm.tm_year + 1900) << "\n";
            std::cout << "Month: " << (tm.tm_mon + 1) << "\n";
            std::cout << "Day: " << tm.tm_mday << "\n";
            std::cout << "Hour: " << tm.tm_hour << "\n";
            std::cout << "Minute: " << tm.tm_min << "\n";
            std::cout << "Second: " << tm.tm_sec << "\n";
        }

        return 0;
    }
    ```

    在这个例子中，`std::get_time` 用于将字符串解析为 `std::tm` 结构体。

#### **注意事项**

* **`tm_year` 的基准年是 1900**，因此需要加上 1900 来获取实际的年份。
* **`tm_mon` 的范围是 0 到 11**，0 表示一月，因此要显示实际的月份，需要加 1。

#### **总结**

`std::tm` 是 C++ 中表示时间的核心结构体，用于处理日期和时间相关的操作。通过结合标准库函数（如 `std::localtime`、`std::mktime`、`std::put_time`、`std::get_time`），你可以轻松地将 `std::tm` 与时间戳或字符串进行转换与操作。

在 `/proc/[PID]/maps` 文件中的权限字段 `rwxp`，最后一个字符 `p` 表示的是内存区域的 **共享属性**。具体来说，它用于描述内存区域的 **私有性** 或 **共享性**。这个标志控制了该内存区域如何在不同进程之间共享。

## `p`（Private） vs `s`（Shared）

* **`p` (Private)**:
  * 表示内存区域是私有的。当该内存区域被映射到进程的地址空间时，如果进程对该区域进行了写操作，操作系统会为该进程创建该内存区域的私有副本。这意味着修改不会影响其他进程对同一库的映射。
  * **用途**: 私有映射通常用于堆栈、堆等进程特有的内存区域，或者动态库的私有数据区域。
* **`s` (Shared)**:
  * 表示内存区域是共享的。多个进程可以映射同一个内存区域，并且所有进程都可以看到该区域的相同内容。如果一个进程修改了共享内存区域的内容，所有其他映射到该区域的进程也会看到这些修改。
  * **用途**: 共享映射常用于共享库的代码段和只读数据段，或者用于不同进程之间的共享内存区。

#### 示例

假设 `/proc/[PID]/maps` 文件中有如下内容：

```
7f3b5c000000-7f3b5c021000 r-xp 00000000 08:01 1234567 /usr/lib/libexample.so
7f3b5c021000-7f3b5c022000 r--p 00021000 08:01 1234567 /usr/lib/libexample.so
7f3b5c022000-7f3b5c023000 rw-p 00022000 08:01 1234567 /usr/lib/libexample.so
```

在这些行中：

* **`r-xp`**:
  * **`r`**: 可读
  * **`x`**: 可执行
  * **`p`**: 私有
* **`r--p`**:
  * **`r`**: 可读
  * **`--`**: 不可写
  * **`p`**: 私有
* **`rw-p`**:
  * **`r`**: 可读
  * **`w`**: 可写
  * **`p`**: 私有

这里的 `p` 表示这些内存区域是私有的，因此每个进程对这些区域的写操作不会影响到其他进程。

#### 总结

权限字段的最后一个字符 `p` 指示内存区域是否是 **私有** 的，意味着该内存区域的内容在不同进程之间不会共享。`s` 则表示 **共享**，允许不同进程之间共享相同的内存内容。这个属性对于内存管理和进程间通信具有重要影响。

## dlopen

`dlopen` 是一个用于动态加载共享库（`.so` 文件）的函数，它允许程序在运行时加载和链接共享库。`dlopen` 函数属于 POSIX 标准的动态链接库接口，通常在 C/C++ 程序中使用。以下是 `dlopen` 函数的详细介绍及其各个参数的用途。

#### 函数原型

```c
#include <dlfcn.h>

void *dlopen(const char *filename, int flag);
```

#### 参数说明

1. **`filename`**
   * **类型**: `const char *`
   * **用途**: 指定要加载的共享库的路径或名称。`filename` 可以是一个绝对路径、相对路径，或者仅是库的名称。
   * **特殊情况**:
     * 如果 `filename` 是 `NULL`，`dlopen` 将返回程序的主可执行文件的句柄。这通常用于在运行时获取应用程序自身的符号。
     * 如果 `filename` 不包含斜杠（即纯库名称），`dlopen` 将在标准库路径中（如 `LD_LIBRARY_PATH`、`/lib`、`/usr/lib`）搜索该库。
     * 如果指定路径不完整（如不包含文件扩展名），`dlopen` 可能会自动附加 `.so` 扩展名来寻找库。
2. **`flag`**
   * **类型**: `int`
   * **用途**: 指定加载共享库时的行为选项。这是一个位掩码，可以由以下标志组合而成：
     * **`RTLD_LAZY`**: 延迟解析符号。在库被加载时，仅解析所需的符号。其他符号直到它们被实际使用时才解析。如果符号无法解析，会导致程序崩溃。这是常用选项之一。
     * **`RTLD_NOW`**: 立即解析所有符号。如果无法解析任何符号，`dlopen` 将返回 `NULL` 并设置错误状态。这种模式更为严格。
     * **`RTLD_GLOBAL`**: 将该库的符号公开，使得后续加载的其他库可以访问这些符号。这对于多个库共享符号的场景很有用。
     * **`RTLD_LOCAL`**: 默认选项，限制该库的符号仅在自身可见，其他库无法访问。这种方式更为安全。
     * **`RTLD_NODELETE`**: （GNU 扩展）防止在 `dlclose` 调用时卸载该库。即使 `dlclose` 被调用，库也会保留在内存中，符号依然可用。
     * **`RTLD_NOLOAD`**: （GNU 扩展）如果库已经加载，返回其句柄。如果库未加载，则返回 `NULL`。不加载新库。
     * **`RTLD_DEEPBIND`**: （GNU 扩展）优先解析库内部定义的符号，而不是全局符号。这可以防止全局符号与库内的符号冲突。

#### 返回值

* **成功**: 返回一个指向共享库的句柄 (`void *`)。可以通过此句柄使用 `dlsym` 来获取库中的符号。
* **失败**: 返回 `NULL`。可以使用 `dlerror` 函数获取详细的错误信息。

#### 示例代码

以下是一个使用 `dlopen` 动态加载库的示例代码：

```c
#include <stdio.h>
#include <dlfcn.h>

int main() {
    // 加载共享库
    void *handle = dlopen("libm.so.6", RTLD_LAZY);
    if (!handle) {
        fprintf(stderr, "%s\n", dlerror());
        return 1;
    }

    // 清除错误
    dlerror();

    // 获取库中的符号
    double (*cosine)(double) = (double (*)(double)) dlsym(handle, "cos");
    const char *dlsym_error = dlerror();
    if (dlsym_error) {
        fprintf(stderr, "%s\n", dlsym_error);
        dlclose(handle);
        return 1;
    }

    // 使用符号
    printf("%f\n", (*cosine)(2.0));

    // 关闭共享库
    dlclose(handle);

    return 0;
}
```

#### 常见用途

* **插件系统**: `dlopen` 可用于加载插件，以扩展程序功能，而不需要在编译时链接所有可能的插件。
* **延迟加载库**: 减少初始加载时间和内存占用，仅在实际需要时加载库。
* **多版本库**: 根据需要在运行时选择特定版本的库加载。

#### 总结

`dlopen` 是一个强大的函数，可以在运行时动态加载和链接共享库。通过设置不同的标志，可以控制符号解析的时机、库的可见性以及符号冲突的处理方式。与 `dlsym` 和 `dlclose` 配合使用，可以在 C/C++ 程序中实现灵活的动态库管理。

## HPI文档阅读

* login是httpclient的方法。

## API/SPI规范

* 返回-4表示错误，0正常。ErrorId非零错误，零正常。&#x20;
* OrderSysID由sysOrderId构建,将sysQuotedId转换成ins\_id,根据ins\_id转换为submit\_id,若有submitId则记录sysid到submitid的映射关系，否则用sysOrderId构建OrderRef。总体而言是通过instructionId获取Orderref(绑定sysid->submit\_id)。
* sysorderid、instructionid均为对方提供
* 母单用ParseIntructionField解析，所以母单不会出现在sysidtoorderref的表里。所以顺序无所谓。
* 子单OrderRef在成功查到submitid后不需要手动填，AlgoSPid onRtnOrder会自动填。失败后填sysOrderId
* 解析成功后都会调AlgoSpi的Onxxx，而这个AlogoSpi就是基类的m\_pAlgoSpi，且m\_pSpi也指向它，直接在找不到子类方法时调它就好。

## Debug

纯虚函数仅声明不实现导致vtable报错。

其它地方报连接错误是因为winapi没链接

pip报错重新编译Python即可

找不到beforetarget.sh清理项目重新生成即可

segmentation fault是因为没有注册enum和if else的名称加载。

CreateTradeApi是一个已声明未定义的工厂方法，需要在Api的cpp文件中定义它。

gil\_scoped\_acquire析构时释放锁，release析构时获取锁，因此作用域很重要。

模块未找到发现是pip路径错误，修改后可在命令行python环境中import，却在Pybind中找不到。

设置PYTHONPATH变量后不出现模块找不到错误，出现调用\_\_init\_\_的错误ImportError。

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

ldd -r后发现确实没有符号的定义

HPI.so设置-Wl,--no-as-needed无效。

已加载的libpython3.12的so可能因为**`RTLD_LOCAL的dlopen HPI.so`**而不可见。

增加dlopen libpython.so，RTLD\_GLOGAL并-ldl后不再importError。

没加https://或http协议报错

加了http报错ReadError:\[Errno 104] Connection reset by peer，可能是协议不对。改为https后报错ConnectError:\[SSL:WRONG\_VERSION\_NUMBER] wrong version number。服务器开个域名白名单+http就可以了。

返回的account对应InvestorId，但似乎查询用的是account\_id

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

py::object的cast\<const char\*>会导致static assert 失败，应该是内存管理问题。cast为std::string就好。

\[0]被识别为\[NULL]导致segfault,用py::int\_{0}替代，不报Segfault了。

测试程序中可能出现还没有loginin初始化httpclient就调用reqaccount导致segfault,设置提前判断终止运行后避免。

出现py::int转std::string失败，改为转py::str再转std::string

limit=false(0)时确实一次返回所有

构造WSClient对象时出现ValueError:signal only works in main thread of the main interpreter以及AttributeError: 'WSClient' object has no attribute '_sio_client'。由于在pythonCLI中正常初始化，怀疑是主线程问题。~~确实，在Interpreter被初始化的线程里用这个不报错~~。是在主线程的子线程中用不报错，主线程的子线程的子线程里用报错？？经测试，C++中init,httplogin,wsclient线程id各不相同，python线程号也各不相同，醉了。。

updatetime为py::int类型(可能是大数字），得先转str再转string

改用CMessage线程中初始化解释器并进行一切python操作，python线程和C++线程号都保持不变了。

py::list作为成员变量会初始化，缺少解释器，segmentation fault。设为object，靠转型来操作。

ResponseError:-1001发现是因为limit\_upxxxdown参数没传

ValueError:dictionary update sequence element #0xxx或str不能用str作索引是因为匿名py::dict对象在\_a=后自动转变为str，需要用.attr('xxx')=xxx来保留dict类型。

security alogo not allow什么的是因为股票不可买，换个可买的就可以了

std::f

ConnectionError: One or more namespaces failed to connect似乎是因为使用了两次print?与pirntthreadId和print try to use的顺序无关,与print内容无关。与LOG无关。似乎仅仅是巧合。while循环一直尝试直到成功即可。



蹊跷的错误无法捕获，仅仅打印信息后就接收不到回调了。券商问题，需要自己捕获。
