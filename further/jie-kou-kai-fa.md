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

## HPI文档阅读

* login是httpclient的方法。

## API/SPI规范

* 返回-4表示错误，0正常。ErrorId非零错误，零正常。

## Debug

纯虚函数仅声明不实现导致vtable报错。

其它地方报连接错误是因为winapi没链接

pip报错重新编译Python即可

找不到beforetarget.sh清理项目重新生成即可

segmentation fault是因为没有注册enum和if else的名称加载。

CreateTradeApi是一个已声明未定义的工厂方法，需要在Api的cpp文件中定义它。



