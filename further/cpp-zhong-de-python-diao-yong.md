# cpp 中的Python 调用

{% embed url="https://github.com/pybind/pybind11?tab=readme-ov-file" %}

{% embed url="https://pybind11.readthedocs.io/en/stable/" %}

{% embed url="https://pybind11.readthedocs.io/_/downloads/en/latest/pdf" %}

{% embed url="https://pybind11.readthedocs.io/_/downloads/en/latest/pdf/" %}



```cpp
#include <pybind11/pybind11.h>

namespace py = pybind11;

int add(int i, int j) {
    return i + j;
}

PYBIND11_MODULE(example, m) {
    m.doc() = "pybind11 example plugin"; // optional module docstring
    m.def("add", &add, "A function which adds two numbers");
}

```

```sh
# 编译成动态链接库（Linux/macOS）
c++ -O3 -shared -std=c++11 -fPIC `python3 -m pybind11 --includes` example.cpp -o example`python3-config --extension-suffix`

# 使用 Python 导入并调用
import example
result = example.add(2, 3)
print(result)  # 输出：5

```

## 配置python虚拟环境以支持多版本



> ### 如何管理多个 Python 版本
>
> 在开发过程中，我们经常会遇到需要同时使用多个 Python 版本的情况，比如：
>
> * **不同项目需要不同版本：** 某些库或框架只支持特定的 Python 版本。
> * **测试兼容性：** 需要测试代码在不同 Python 版本下的运行情况。
>
> 为了有效地管理多个 Python 版本，我们可以采用以下几种方法：
>
> #### 1. **虚拟环境**
>
> * **概念：** 创建一个隔离的 Python 环境，每个环境都有自己的 Python 解释器和独立的包。
> * **工具：**
>   * **venv:** Python 自带的虚拟环境创建工具。
>   * **virtualenv:** 一个功能更强大的第三方虚拟环境管理工具。
>   * **conda:** Anaconda 或 Miniconda 提供的包和环境管理系统，可以创建、管理多个 Python 环境。
> * **优点：**
>   * **隔离性好：** 不同项目使用不同的环境，避免相互干扰。
>   * **灵活配置：** 可以自定义每个环境的 Python 版本和安装的包。
>   * **方便管理：** 可以轻松切换不同的环境。
>
> **示例（使用 venv）：**
>
> Bash
>
> ```
> # 创建虚拟环境
> python -m venv my_env
>
> # 激活虚拟环境
> source my_env/bin/activate
>
> # 安装需要的包
> pip install numpy pandas
> ```
>
> #### 2. **Anaconda/Miniconda**
>
> * **概念：** 一个开源的 Python 和 R 的发行版，提供了包管理和环境管理功能。
> * **优点：**
>   * **集成化：** 包含了大量的科学计算库和工具。
>   * **方便管理：** 使用 conda 命令可以轻松创建、激活和删除环境。
>   * **跨平台：** 支持 Windows、Linux 和 macOS。
>
> **示例：**
>
> Bash
>
> ```
> # 创建环境
> conda create -n my_env python=3.8
>
> # 激活环境
> conda activate my_env
>
> # 安装包
> conda install numpy pandas
> ```
>
> #### 3. **pyenv**
>
> * **概念：** 一个简单的 Python 版本管理工具，可以轻松地在多个 Python 版本之间切换。
> * **优点：**
>   * **轻量级：** 只管理 Python 版本，不涉及包管理。
>   * **简单易用：** 使用命令行工具即可管理 Python 版本。
>
> **示例：**
>
> Bash
>
> ```
> # 安装 pyenv
> # 参考官方文档：https://github.com/pyenv/pyenv
>
> # 安装 Python 版本
> pyenv install 3.8.10
>
> # 设置全局 Python 版本
> pyenv global 3.8.10
>
> # 在特定目录下使用指定 Python 版本
> pyenv local 3.7.9
> ```
>
> #### 4. **asdf**
>
> * **概念：** 一个通用的版本管理器，可以管理多个语言的版本，包括 Python、Ruby、Node.js 等。
> * **优点：**
>   * **通用性强：** 可以管理多种语言的版本。
>   * **灵活配置：** 可以自定义插件来管理不同的语言。
>
> **示例：**
>
> Bash
>
> ```
> # 安装 asdf 和 Python 插件
> # 参考官方文档：https://asdf-vm.com/
>
> # 安装 Python 版本
> asdf install python 3.8.10
>
> # 设置全局 Python 版本
> asdf global python 3.8.10
> ```
>
> #### 选择合适的工具
>
> * **项目规模和复杂度：** 小型项目可以使用 venv，大型项目或需要管理多个语言的项目可以考虑 asdf。
> * **团队协作：** 如果团队成员对不同的工具有偏好，可以统一使用一种工具。
> * **个人习惯：** 选择自己熟悉的工具。
>
> **总结**
>
> 通过使用虚拟环境或版本管理工具，我们可以有效地管理多个 Python 版本，避免不同项目之间的冲突，提高开发效率。
>
> **选择哪种工具取决于你的具体需求和偏好。**
>
> **其他注意事项：**
>
> * **配置 IDE：** 将 IDE 配置为使用正确的虚拟环境或 Python 解释器。
> * **requirements.txt:** 使用 requirements.txt 文件记录每个虚拟环境所需的包。
> * **.python-version:** 使用 .python-version 文件指定项目使用的 Python 版本。
>
> **希望这个回答能帮助你更好地管理多个 Python 版本！**
>
> **如果你还有其他问题，欢迎随时提问。**

## 配置VS项目来链接pybind11库

> 要在 Visual Studio (VS) C++ 项目中使用 `pybind11` 并配置使用远程 Linux 项目中的特定 Python 版本，你需要遵循以下步骤。这些步骤涵盖了如何设置 `pybind11`，配置 Visual Studio 进行远程 Linux 开发，并确保在远程 Linux 系统上使用指定的 Python 版本。
>
> #### 1. 安装和配置 Visual Studio 远程开发环境
>
> 首先，你需要确保 Visual Studio 安装了远程开发工具，并能够连接到你的远程 Linux 机器。
>
> **安装远程开发工具：**
>
> * 在 Visual Studio 安装程序中，选择 "Linux 发展" 组件，这将安装用于远程开发的所有必要工具。
>
> **配置远程连接：**
>
> * 在 Visual Studio 中，打开你的 C++ 项目。
> * 右键点击项目名称，然后选择 "属性"。
> * 在 "配置属性" 中，找到 "常规" 选项卡，并将 "目标平台" 设置为 "远程机器"。
> * 在 "远程机器" 选项卡中，输入远程 Linux 机器的 IP 地址和 SSH 连接设置。
>
> #### 2. 在远程 Linux 上安装指定的 Python 版本和 `pybind11`
>
> **使用 `pyenv` 安装特定 Python 版本：**
>
> 在远程 Linux 机器上，你可以使用 `pyenv` 来管理和安装不同的 Python 版本。
>
> ```bash
> curl https://pyenv.run | bash
> ```
>
> 安装后，添加 `pyenv` 到你的 shell 配置文件 (`~/.bashrc`, `~/.bash_profile`, `~/.zshrc` 等):
>
> ```bash
> export PATH="$HOME/.pyenv/bin:$PATH"
> eval "$(pyenv init --path)"
> eval "$(pyenv init -)"
> eval "$(pyenv virtualenv-init -)"
> ```
>
> 然后安装你需要的 Python 版本（例如 Python 3.8.10）：
>
> ```bash
> pyenv install 3.8.10
> pyenv global 3.8.10
> ```
>
> **安装 `pybind11`：**
>
> 使用 `pip` 安装 `pybind11`：
>
> ```bash
> pip install pybind11
> ```
>
> #### 3. 在 Visual Studio 项目中使用 `pybind11`
>
> **设置项目属性：**
>
> * 在 Visual Studio 中右键点击你的项目，然后选择 "属性"。
> * 在 "配置属性" 中，选择 "C/C++" -> "常规"。
> * 在 "附加包含目录" 中，添加 `pybind11` 头文件的路径。
>
> 你可以使用以下命令获取 `pybind11` 的包含路径：
>
> ```bash
> python3 -m pybind11 --includes
> ```
>
> 添加输出的路径到 Visual Studio 项目的包含目录中。
>
> **配置 CMakeLists.txt (如果使用 CMake)：**
>
> 如果你的项目使用 CMake，可以通过修改 `CMakeLists.txt` 来配置 `pybind11`。
>
> ```cmake
> find_package(pybind11 REQUIRED)
> include_directories(${pybind11_INCLUDE_DIRS})
> target_link_libraries(your_target_name ${PYTHON_LIBRARIES})
> ```
>
> 确保 `PYTHON_LIBRARIES` 和 `pybind11_INCLUDE_DIRS` 的路径正确指向远程 Linux 机器上安装的 Python 版本和 `pybind11` 包。
>
> #### 4. 编写和编译代码
>
> **示例代码：**
>
> 在你的项目中，创建一个使用 `pybind11` 的 C++ 文件，例如 `example.cpp`：
>
> ```cpp
> #include <pybind11/pybind11.h>
>
> int add(int i, int j) {
>     return i + j;
> }
>
> PYBIND11_MODULE(example, m) {
>     m.def("add", &add, "A function that adds two numbers");
> }
> ```
>
> **编译并生成 Python 模块：**
>
> 配置完成后，使用 Visual Studio 编译你的项目，这将通过 SSH 将代码发送到远程 Linux 机器上进行编译，并将生成的 `.so` 文件返回到本地。你可以在 Python 中导入并使用这个模块。
>
> #### 5. 使用指定的 Python 版本运行
>
> 在远程 Linux 上测试编译好的 Python 模块时，确保使用你通过 `pyenv` 配置的 Python 版本：
>
> ```bash
> python -c "import example; print(example.add(2, 3))"
> ```
>
> 这个命令将使用你指定的 Python 版本，并调用你编写的 `example` 模块中的 `add` 函数。
>
> #### 总结
>
> 通过这些步骤，你可以在 Visual Studio 中开发一个基于 `pybind11` 的 C++ 项目，并使用远程 Linux 机器上指定的 Python 版本进行编译和测试。这个设置允许你在 Windows 上使用熟悉的开发环境，同时利用 Linux 上的 Python 环境和工具链进行开发。

## conda 版本管理

> ### 如何查看 conda 管理的所有 Python 版本
>
> **conda** 是一个强大的包和环境管理系统，可以帮助我们轻松地管理多个 Python 版本以及对应的包。
>
> #### 查看所有 conda 环境
>
> 要查看 conda 管理的所有 Python 版本，首先需要列出所有已创建的 conda 环境。每个环境都有一个特定的 Python 版本。
>
> Bash
>
> ```
> conda info --envs
> ```
>
> 这条命令会输出一个列表，显示所有已创建的环境，以及每个环境的路径和 Python 版本。
>
> #### 详细查看某个环境的 Python 版本
>
> 如果你想查看某个特定环境的 Python 版本，可以先激活该环境，然后使用以下命令：
>
> Bash
>
> ```
> conda activate your_env_name
> python --version
> ```
>
> * `conda activate your_env_name`：激活名为 `your_env_name` 的环境。
> * `python --version`：查看当前激活环境的 Python 版本。
>
> #### 示例
>
> 假设你有多个 conda 环境，其中一个名为 `myenv`，包含 Python 3.8。
>
> Bash
>
> ```
> # 列出所有环境
> conda info --envs
>
> # 激活 myenv 环境
> conda activate myenv
>
> # 查看 myenv 环境的 Python 版本
> python --version
> ```
>
> #### 总结
>
> 通过以上命令，你可以轻松地查看 conda 管理的各个 Python 版本。**conda** 是一个非常有用的工具，可以帮助你更好地管理你的 Python 开发环境。
>
> **其他注意事项：**
>
> * **conda update:** 如果你想更新 conda 自身或某个环境中的 Python 版本，可以使用 `conda update conda` 或 `conda update python` 命令。
> * **conda create:** 如果你想创建一个新的 conda 环境，可以使用 `conda create -n new_env python=X.X` 命令。
>
> **想了解更多关于 conda 的信息，可以参考官方文档：** [https://docs.conda.io/projects/conda/en/latest/](https://docs.conda.io/projects/conda/en/latest/)
>
> **如果你还有其他问题，欢迎随时提问。**
>
> **希望这个回答能帮助你更好地理解如何使用 conda 管理 Python 版本！**

## pybind11调用python函数，并处理返回的dict类型

> 使用 `pybind11`，可以轻松地在 C++ 中调用 Python 方法，并将返回值转换为 C++ 中可用的类型。对于返回 `dict` 类型的情况，`pybind11` 提供了直接的支持。你可以调用 Python 函数并将返回值转换为 `pybind11::dict` 对象。
>
> #### 1. 安装 `pybind11`
>
> 确保你已经安装了 `pybind11`，可以通过 `pip` 安装：
>
> ```bash
> pip install pybind11
> ```
>
> #### 2. 示例代码：调用 Python 函数并获取 `dict` 返回值
>
> 以下是一个 C++ 示例，它使用 `pybind11` 调用 Python 函数并获取 `dict` 类型的返回值。
>
> **Python 代码**
>
> 首先，假设我们有一个 Python 函数，它返回一个字典：
>
> ```python
> # mymodule.py
>
> def get_data():
>     return {"key1": 100, "key2": "value2", "key3": [1, 2, 3]}
> ```
>
> **C++ 代码**
>
> 下面的 C++ 代码使用 `pybind11` 调用这个 Python 函数，并处理返回的 `dict`：
>
> ```cpp
> #include <pybind11/embed.h>  // 必须在主代码之前包含
> #include <pybind11/stl.h>    // 如果你要使用 C++ STL 容器，请包含这个头文件
> #include <iostream>
>
> namespace py = pybind11;
>
> int main() {
>     // 初始化 Python 解释器
>     py::scoped_interpreter guard{};
>
>     try {
>         // 导入 Python 模块
>         py::object mymodule = py::module::import("mymodule");
>
>         // 调用 Python 函数并获取返回值
>         py::object result = mymodule.attr("get_data")();
>
>         // 将返回值转换为 py::dict 类型
>         py::dict dict_result = result.cast<py::dict>();
>
>         // 遍历 dict_result
>         for (auto item : dict_result) {
>             std::string key = item.first.cast<std::string>();
>             py::object value = item.second;
>
>             std::cout << "Key: " << key << ", Value: " << py::str(value) << std::endl;
>         }
>     } catch (py::error_already_set& e) {
>         std::cerr << "Python error: " << e.what() << std::endl;
>     }
>
>     return 0;
> }
> ```
>
> #### 3. 代码解释
>
> * **`py::scoped_interpreter guard{}`**: 这个对象初始化并管理 Python 解释器的生命周期。在其作用域结束时，解释器会被自动关闭。
> * **`py::module::import("mymodule")`**: 通过 `py::module::import` 导入 Python 模块 `mymodule`。
> * **`mymodule.attr("get_data")()`**: 调用 `mymodule` 模块中的 `get_data` 函数。`attr("get_data")` 获取函数的引用，然后通过 `()` 执行函数。
> * **`result.cast<py::dict>()`**: 将返回的 Python 对象 `result` 转换为 `py::dict` 类型。
> * **遍历 `py::dict`**: 使用 `for (auto item : dict_result)` 遍历字典的键值对。通过 `item.first` 和 `item.second` 分别获取键和值，并转换为适当的 C++ 类型。
> * **错误处理**: 使用 `py::error_already_set` 捕获并处理可能的 Python 异常。
>
> #### 4. 编译和运行
>
> 编译这个 C++ 代码时，你需要确保链接了 Python 和 `pybind11` 库。以下是一个使用 g++ 的编译示例：
>
> ```bash
> g++ -O3 -Wall -shared -std=c++11 -fPIC `python3 -m pybind11 --includes` example.cpp -o example`python3-config --extension-suffix` -lpython3.8
> ```
>
> 确保将 `python3.8` 替换为你系统上的实际 Python 版本。
>
> #### 5. 运行
>
> 在编译完成后，直接运行生成的可执行文件即可：
>
> ```bash
> ./example
> ```
>
> 你应该会看到输出的键值对，表明已经成功从 Python 函数获取并处理了 `dict` 类型的返回值。
>
> 这个例子展示了如何通过 `pybind11` 在 C++ 中调用 Python 函数并获取 `dict` 类型的返回值。`pybind11` 使得 C++ 与 Python 之间的数据交互变得非常简洁和直观。

## pybind处理python异常

> 使用 `pybind11` 时，如果在调用 Python 方法时发生了异常，可以使用 `pybind11` 提供的异常处理机制来捕获和处理这些异常。`pybind11` 能够捕获到 Python 抛出的异常，并将其转换为 C++ 异常，以便在 C++ 代码中进行处理。
>
> #### 1. 捕获 Python 异常
>
> `pybind11` 提供了 `py::error_already_set` 异常类，用于捕获从 Python 中抛出的异常。这个异常类包含了有关异常的详细信息，包括 Python 的异常类型、消息以及回溯信息。
>
> #### 2. 示例代码
>
> 以下是一个示例，展示了如何在 C++ 中使用 `pybind11` 捕获和处理 Python 异常。
>
> **Python 代码**
>
> 首先，假设我们有一个可能抛出异常的 Python 函数：
>
> ```python
> # mymodule.py
>
> def risky_function(x):
>     if x < 0:
>         raise ValueError("Negative value provided!")
>     return {"result": x * 2}
> ```
>
> **C++ 代码**
>
> 接下来，我们在 C++ 中调用这个 Python 函数，并处理可能发生的异常：
>
> ```cpp
> #include <pybind11/embed.h>  // 必须在主代码之前包含
> #include <iostream>
>
> namespace py = pybind11;
>
> int main() {
>     // 初始化 Python 解释器
>     py::scoped_interpreter guard{};
>
>     try {
>         // 导入 Python 模块
>         py::object mymodule = py::module::import("mymodule");
>
>         // 调用 Python 函数，传入一个负值以触发异常
>         py::object result = mymodule.attr("risky_function")(-10);
>
>         // 如果没有抛出异常，处理返回值
>         py::dict dict_result = result.cast<py::dict>();
>         std::cout << "Function succeeded, result: " << py::str(dict_result) << std::endl;
>     } catch (py::error_already_set &e) {
>         // 捕获 Python 异常并输出错误信息
>         std::cerr << "Python exception occurred: " << e.what() << std::endl;
>     }
>
>     return 0;
> }
> ```
>
> #### 3. 代码解释
>
> * **`py::scoped_interpreter guard{}`**: 初始化 Python 解释器，并确保其生命周期受限于 C++ 代码的作用域。
> * **`py::module::import("mymodule")`**: 导入 Python 模块 `mymodule`。
> * **`mymodule.attr("risky_function")(-10)`**: 调用 Python 函数 `risky_function`，传递一个负值 `-10` 以触发异常。
> * **`py::error_already_set &e`**: 捕获 `py::error_already_set` 异常，这个异常表示在调用 Python 代码时发生了错误。
> * **`e.what()`**: 输出异常信息，包括 Python 异常类型、消息和回溯信息。
>
> #### 4. 编译和运行
>
> 编译这个 C++ 代码时，确保链接了 Python 和 `pybind11` 库。以下是一个使用 g++ 的编译示例：
>
> ```bash
> g++ -O3 -Wall -shared -std=c++11 -fPIC `python3 -m pybind11 --includes` example.cpp -o example`python3-config --extension-suffix` -lpython3.8
> ```
>
> 在编译完成后，直接运行生成的可执行文件即可：
>
> ```bash
> ./example
> ```
>
> 运行后，由于我们传递了负值给 `risky_function`，Python 会抛出 `ValueError` 异常，C++ 代码将捕获到该异常，并在控制台上输出相应的错误信息。
>
> #### 5. 捕获特定类型的 Python 异常
>
> 如果你只想捕获特定类型的 Python 异常，而不是所有异常，可以使用 `py::type_error`, `py::value_error`, `py::index_error` 等专门的异常类。例如：
>
> ```cpp
> try {
>     // ... 调用可能抛出异常的 Python 函数
> } catch (py::value_error &e) {
>     // 处理 Python 的 ValueError 异常
> } catch (py::error_already_set &e) {
>     // 处理其他异常
> }
> ```
>
> 这种方法允许你针对不同的 Python 异常类型采取不同的处理措施。
>
> #### 总结
>
> 使用 `pybind11`，你可以方便地捕获和处理在调用 Python 方法时发生的异常。通过捕获 `py::error_already_set` 或特定的异常类型，你可以在 C++ 代码中对异常进行适当的处理，并根据需要采取措施，如记录日志、清理资源、或向用户报告错误。

## pybind的版本限制

> `pybind11` 作为一个连接 C++ 与 Python 的工具，旨在支持多个 Python 版本。它的设计使得它能够兼容多种 Python 版本，但在不同的 Python 版本之间可能会有一些细微的区别或限制。
>
> #### 1. 支持的 Python 版本
>
> `pybind11` 通常支持以下 Python 版本：
>
> * **Python 2.7**: 虽然 Python 2.7 已经在 2020 年停止了官方支持，但 `pybind11` 在旧版本中对 Python 2.7 仍然提供支持。一般来说，建议使用 Python 3 及以上的版本进行开发。
> * **Python 3.4 及以上**: `pybind11` 完全支持 Python 3.4 及以上的版本。这包括所有的主流 Python 3 版本，如 3.5、3.6、3.7、3.8、3.9、3.10 及更新的版本。
>
> `pybind11` 的开发和测试主要围绕这些版本进行，因此使用这些版本通常能够保证最大的兼容性和稳定性。
>
> #### 2. 依赖的 Python 特性
>
> `pybind11` 使用了一些 Python 3 特性来实现其功能，因此它在 Python 3 中的功能更为全面和先进。以下是一些依赖于 Python 版本的功能和限制：
>
> * **类型提示和注释**: Python 3 引入了类型提示 (`type hints`) 和注释，这些特性在 `pybind11` 中被利用来进行更强大的类型推断和错误检查。如果你使用 Python 2.7，这些功能将不可用。
> * **Unicode 处理**: Python 3 改进了对 Unicode 字符串的处理，这在 `pybind11` 中得到了更好的支持。在 Python 2 中，`str` 和 `unicode` 的处理更为复杂。
> * **Python 2 vs Python 3 API 差异**: Python 2 和 Python 3 之间存在一些 API 差异，例如 `print` 函数、`dict.keys()` 返回值类型等。`pybind11` 试图在两者之间提供兼容性，但在某些细节上仍可能存在不同的行为。
>
> #### 3. C++ 标准的要求
>
> 虽然 `pybind11` 与 Python 版本的兼容性很好，但它对 C++ 标准有要求。`pybind11` 要求至少使用 C++11 标准。因此，如果你的编译器不支持 C++11 或更高版本，你将无法使用 `pybind11`。
>
> #### 4. 在不同 Python 版本中的构建
>
> 当使用 `pybind11` 构建扩展模块时，确保你使用的 Python 版本与最终运行时的 Python 版本匹配。不同的 Python 版本通常会有不同的 ABI（应用程序二进制接口），这意味着一个扩展模块通常只能与它编译时使用的 Python 版本兼容。
>
> #### 5. 检查和设置 Python 版本
>
> 在使用 `pybind11` 构建扩展模块时，你可以通过以下命令检查正在使用的 Python 版本：
>
> ```bash
> python --version
> ```
>
> 在编译时，你可以使用 `python3-config` 来确保链接到正确的 Python 版本：
>
> ```bash
> g++ -O3 -Wall -shared -std=c++11 -fPIC `python3 -m pybind11 --includes` example.cpp -o example`python3-config --extension-suffix` -lpython3.8
> ```
>
> #### 6. 小结
>
> * `pybind11` 支持 Python 2.7 以及 Python 3.4 及以上的版本，但 Python 2.7 由于官方支持终止，建议使用 Python 3.4 及以上的版本。
> * 在使用 `pybind11` 时，确保使用的 Python 版本与目标环境中的 Python 版本一致。
> * `pybind11` 对 C++11 及以上的标准有要求，因此在使用时需要确保编译器支持 C++11 或更高版本。
>
> 总之，在开发过程中，推荐使用最新的 Python 3 版本和现代 C++ 标准来确保最佳的性能和功能支持。

## anaconda 和conda的关系

> Conda 和 Anaconda 是两个相关但不同的概念，主要区别在于它们的作用和范围。
>
> 1. **Conda**:
>    * Conda 是一个开源的包管理系统和环境管理器，最初是为 Python 语言设计的，但现在也支持其他编程语言，如 R、Ruby、Lua 等。
>    * Conda 可以帮助用户安装、运行和管理软件包以及它们的依赖关系，同时还可以创建和管理独立的环境。通过 Conda，你可以轻松地在同一台计算机上管理多个不同版本的 Python 和其他包。
> 2. **Anaconda**:
>    * Anaconda 是一个数据科学和机器学习的平台，它包含了 Conda 以及其他许多工具和库。Anaconda 提供了一个开箱即用的解决方案，集成了数百个常用的科学计算、数据分析、机器学习库，如 NumPy、Pandas、Scikit-learn、Jupyter Notebook 等。
>    * Anaconda 包含了 Conda，所以当你安装 Anaconda 时，你也同时安装了 Conda。Anaconda 是为数据科学家、数据分析师和其他开发人员设计的，旨在提供一个功能齐全的开发环境。
>
> 总结来说，**Conda 是 Anaconda 平台的一部分**。你可以单独使用 Conda 管理包和环境，也可以通过安装 Anaconda 获得一个完整的、预配置的开发环境，其中包含 Conda 以及许多其他工具和库。

## 为什么conda目录下有pip？

>
>
> **conda** 和 **pip** 都是 Python 的包管理工具，但它们在功能和设计上有各自的侧重点。你可能会在 conda 创建的环境中发现 pip 的存在，这主要是因为以下原因：
>
> #### 1. **conda 的包生态系统和 pip 的互补**
>
> * **conda** 主要管理 Anaconda 自身维护的包，以及一些第三方包。它提供了更全面的环境管理功能，可以创建隔离的 Python 环境。
> * **pip** 主要从 PyPI（Python Package Index）上下载安装包。PyPI 上的包数量庞大，conda 并不包含所有包。
> * **互补性：** 为了方便用户，conda 环境中通常会预装 pip，这样用户可以直接使用 pip 安装 PyPI 上的包，而无需切换到全局 Python 环境。
>
> #### 2. **conda 环境的灵活性**
>
> * **自定义环境：** conda 可以创建自定义的 Python 环境，每个环境都有独立的包和依赖。
> * **pip 的集成：** 在这些自定义环境中，pip 可以用来安装那些 conda 仓库中没有的包，从而增加了环境的灵活性。
>
> #### 3. **历史原因**
>
> * **Anaconda 的发展：** Anaconda 起初是为了方便数据科学工作者快速搭建 Python 环境，因此集成了 pip。
> * **用户习惯：** 许多 Python 开发者习惯使用 pip，为了满足用户需求，conda 自然也保留了对 pip 的支持。
>
> **总结**
>
> conda 目录下存在 pip，主要是因为 conda 希望提供一个更加灵活和全面的包管理环境。通过集成 pip，conda 用户可以方便地从 PyPI 上安装各种 Python 包，从而满足不同的开发需求。
>
> **何时使用 conda，何时使用 pip？**
>
> * **conda：**
>   * 创建独立的 Python 环境
>   * 安装 Anaconda 自带的包或 conda-forge 上的包
>   * 管理多个 Python 版本
> * **pip：**
>   * 安装 PyPI 上的包
>   * 在 conda 环境中补充缺少的包
>
> **一般来说，conda 更适合用于管理整个 Python 环境，而 pip 更适合用于在 conda 环境中安装特定的包。**
>
> **需要注意的是：**
>
> * 在 conda 环境中使用 pip 安装的包，其位置可能与 conda 安装的包不同。
> * 如果同时使用 conda 和 pip，需要注意包之间的依赖关系，避免冲突。

## makefile

> 在之前的 Makefile 中，有两个目标：
>
> 1. **`all` 目标**：这是默认的构建目标，用于编译所有的源文件并生成共享库（`.so` 文件）。如果你运行 `make` 或 `make all`，它会生成 `libpython3.12.so` 文件。
> 2. **`install` 目标**：这个目标是用来安装已经生成的共享库（`.so` 文件）。它会将生成的 `libpython3.12.so` 文件复制到你指定的安装目录（`INSTALL_DIR`）。通常是在构建成功后使用，以确保生成的文件被放置到正确的位置。如果你运行 `make install`，它会执行 `all` 目标并安装生成的 `.so` 文件。

## 操作记录

1. 安装cmake
2. 安装用户目录下anaconda
3. 修改anaconda的用户配置
4. 在自己目录下创建anaconda环境pybind11\_env,python版本为python3.10，因为11和12安装不了：conda create -n pybind11\_env python=3.10;conda activate pybind11\_env;conda install -c conda-forge pybind11。产生四个pybind 的include文件夹在环境中。conda remove pybind 和 conda remove pybind-global后分别删除。local/include下的pybind11删不掉，估计不是我安装的。
5. 重新安装后出现4个pybind，包括两个pybind11和两个pybind11\_global

## 配置

附加包含目录pybind11头文件目录“pybind11"的父目录和python3.10的头文件目录

附加库目录到python3.10so所在目录

库依赖项python3.10，来动态链接

连接器命令行增加-Wl,-rpath,/home……（附加库目录python运行so的位置）



1. 将pybind源文件直接复制是可以include不报错的
2. 手动编译生成的python的头文件和动态库用于pybind的包含或链接是可行的



## python3.12源代码VS编译配置

* 生成文件项目
* 包括自动包含的文件，\*.in,configure,config.sub,config.guess,install-sh,Modules/Setup

由于改包含文件老是重新上传文件，并且configure标志位比较陌生，改为上传tar.xz，在编译机上解压配置编译。--enable-shared似乎有问题。

## yum安装python 到指定目录

失败，无法控制。

## xz安装包上传并编译

* 解压，改执行权限，执行
* MSBuild生成返回值设置
* configure选项确定
* with-sslxxx选项指定ssl库目录

### 3.12.5

<figure><img src="../.gitbook/assets/image (86).png" alt=""><figcaption><p>报错</p></figcaption></figure>

似乎只要加上PATH和PYTHONPATH的环境变量的修改再重新生成（先make clean)就可以成功生成so

控制变量，同时去掉，生成不了；去掉PATH，重新生成不了；去掉PYTHONPATH，重新生成不了。两个都得加。

似乎因为结尾多了个/导致编译失败，可能也导致控制变量失败。。。。

分析需要增加PATH的原因：make会在编译目录下生成一个python,后续生成.so时需要用到该python,而python命令为外部命令，默认从PATH中寻找，不添加PATH会调用系统版本python然后出错？



* 结果是，enable-optimizations与gcc 4.8.5低版本不兼容，gcc有优化Bug。去掉就好。之前的偶发成功是因为某个configure没有带enable-optimizations并且没有被彻底清除。与PATH、PYTHONPATH无关。去掉enable-optimizations的configure选项后编译成功。尝试复制到新项目，检验能否一键生成。

{% embed url="https://github.com/python/cpython/issues/94825" %}

<figure><img src="../.gitbook/assets/image (87).png" alt=""><figcaption><p>缺少的模块</p></figcaption></figure>

sqlite3缺少sqlite3\_trace\_v2的符号，

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

这是一个版本问题，我们的sqlite3似乎是0.8.6版本，而要求3.14版本以上。。

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

ssl是路径问题，找不到OpenSSL目录，include文件夹也找不到。设置--with-openssl-rpath来指定动态链接的目录后问题照旧，

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

改为Openssl源代码路径，发现版本有问题，小于1.1.1，/usr/local/openssl/include/openssl和/usr/include/openssl中的版本均为1.0.x。/usr/local/bin/openssl的版本为3.0.9，

Openssl1.1.1w编译后将构建目录作为python的openssl选项后ssl构建成功

生成文件共300M+，其中Python的静态.a占89M,使用without-static-libpython选项不构建该静态库，减少到只有200M多。设置--disable-test-modules后只剩100M



将Python和pybind11放到解决方案中。发现将示例程序部署到部署机上不能运行，设置LD\_LIBRARYPATH仍然报错，设置PYTHONHOME为Python构建目录即可正常运行。可以用setenv设置环境变量。成功迁移到部署机上运行。

configure中，prefix必须是绝对路径，否则会报错。

sftp不上传软链接，上传软链接的链接目标。上传后仍能运行，说明暂时不影响。

\--{without,diable}-{\_,}sqlite{3,}均失败

\-L dir会比系统默认库目录有更高的优先级，-I也是（头文件目录）

一般情况下，**`-Wl,-rpath` 的优先级高于 `LD_LIBRARY_PATH`**。

&#x20;/usr/local/lib下的so有trace\_v2，设置-I,-L和-Wl,-rpath后成功编译sqlite3模块

将sqlite3的头文件和so.0.8.6复制到构建目录下，链接失败。

再将.so复制到构建目录下，configure符号查看成功。（软链接）但链接失败

再将.so.0复制过去，链接成功。

不知道为什么.so.0被优先链接。把.so.0.8.6改名为.so.0放到工程下，使得编译成功。

由于编译机上的目录与部署机上不同，部署机可能需要设置LD\_LIBRARY\_PATH才能找到libsqlite3.so.0，也有可能因为已经设置了PYTHONHOME而不需要了？

checking 找不到trace\_v2不重要，按上面设置找不到trace\_v2后面链接上就行。

由于rpath对中间编译生成的so和构建的so都需要，中间编译生成的so需要借助rpath加载libsqlite3.so.0，而中间编译生成的so的目录在Python中间目录的结构中，无法借助.或./lib达成加载。由于构建得到的libpython.so需要移植，绝对目录不行。找到CodeSources中Modules和libsqlite.so.0的关系，rpath加个../..，尝试编译，不行，改为../../..，不行，都加上也不行。推测rpath是根据当前工作目录来寻找相对路径，与so路径无关，因此估测当前工作目录为Python3-12-5项目目录，则libsqlite3.so.0在..目录下，尝试libsqlite3.so.0

由于rpath与当前路径相关，尝试用$ORIGIN来让rpath更可预期一些。

NODIST选项传的rpath不会到最终的python产物中??似乎还是会传过去

ssl找不到，很可能跟--with--openssl-rpath有关，因为sqlite3可以引入。

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

把--with--openssl-rpath设为no就可以import ssl了。不明所以，难以理解。

将HPITrade.so的rpath改为lib优先，应该无用，因为最后加载看out的rpath。HPI中增加LD\_LIBRARY\_PATH。将out的RPATH中的错误版本ssl、crypto移除仍然无效。

将ssl、crypto移到Interfaces下，正常运行。

缺libffi.so无法编译\_ctypes，。。。

## 为什么LD\_LIBRARY\_PATH=../Python/lib xx.out 能正确找到libssl.so而在xx.out程序运行时setenv("LD\_LIBRARY\_PATH","../Python/lib",1)就找不到？

这是因为 `LD_LIBRARY_PATH` 的作用机制在运行时和程序启动时有不同的行为。

#### 背景：

* **`LD_LIBRARY_PATH`** 是一个环境变量，用于指定在程序运行时查找共享库的路径。当你运行程序时，系统的动态链接器会根据这个变量的值来定位并加载所需的共享库（如 `libssl.so`）。
* **`setenv("LD_LIBRARY_PATH", ...)`** 是在程序运行时设置环境变量的函数。

#### 原因分析：

当你在程序启动前设置 `LD_LIBRARY_PATH`（例如通过 `LD_LIBRARY_PATH=../Python/lib ./xx.out`），系统的动态链接器会在启动时使用这个环境变量来查找并加载共享库。

然而，当你在程序运行过程中调用 `setenv("LD_LIBRARY_PATH", "../Python/lib", 1)` 来设置或修改 `LD_LIBRARY_PATH` 时，已经在启动时完成的动态链接加载不会重新执行。因此，即使你在程序运行时修改了 `LD_LIBRARY_PATH`，它也不会影响动态链接器的行为，无法加载新的共享库。

#### 关键点：

1. **动态链接器的行为**：
   * 动态链接器（如 `ld.so` 或 `ld-linux.so`）在程序启动时会读取 `LD_LIBRARY_PATH` 并加载所需的共享库。
   * 一旦程序启动并加载了所需的共享库，动态链接器不会再重新检查 `LD_LIBRARY_PATH` 的值，即使在运行时通过 `setenv` 修改了它。
2. **`setenv` 的局限性**：
   * `setenv("LD_LIBRARY_PATH", ...)` 修改环境变量是对当前进程的内存环境的修改，但这仅会影响后续通过 `exec()` 创建的新进程，而不会影响当前进程中已经进行的共享库加载。
   * 由于共享库在程序启动时已经被加载，运行时再修改 `LD_LIBRARY_PATH` 无法重新加载或者找到新的共享库。

#### 为什么 `LD_LIBRARY_PATH=../Python/lib ./xx.out` 能正常工作：

在这种情况下，动态链接器会在程序启动时读取 `LD_LIBRARY_PATH` 的值，并根据指定的路径查找并加载所有需要的共享库（如 `libssl.so`）。一旦库加载成功，程序就能够正确运行。

#### 总结：

* **启动时设置 `LD_LIBRARY_PATH`**：在启动程序时通过环境变量设置（例如 `LD_LIBRARY_PATH=../Python/lib ./xx.out`），能够正确引导动态链接器加载共享库。
* **运行时通过 `setenv` 修改 `LD_LIBRARY_PATH`**：无法影响已经启动的程序中的动态链接行为，动态链接器不会重新加载库，除非你通过 `exec()` 系列函数重新启动进程。

如果你想在运行时加载共享库，可以使用 `dlopen` 来动态加载库，而不是依赖 `LD_LIBRARY_PATH`。
