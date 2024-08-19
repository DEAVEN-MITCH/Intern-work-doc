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



阅读接口wiki和提供的python接口信息，疑问：websocket?http?命名？项目位置？生成位置？实现接口继承什么基类？user/password?whl放在工程里？/安装好lib复制到工程里？自定义类型转换成Python类型？

命名？

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
