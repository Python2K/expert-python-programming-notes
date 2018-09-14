## 编写一个包

编写并发布Python包的可重复过程，目的：

*  开始实际工作前，缩短设置所有内容所需时间
* 提供编写包的标准化方法
* 简化测试驱动开发方法的使用
* 使发布过程更加简单

本章4部分：

* 所有包的通用模式(common pattern)描述Python包之间的相似之处以及distutils和setuptools如何发挥核心作用
* 什么是命名空间包(namespace packages)以及它为什么有用
* 在Python包索引（Python Packages Index，PyPI）中如何注册并上传包，重点强调安全性和常见错误
* 独立可执行文件(stand-alone executables)作为将Python应用打包并分发的替代方法

---



#### 创建一外包

**Python Packaing Authority(PyPA)组织**：https://github.com/pypa规范了打包标准化过程，打包指南。

使用wheel格式，未来可能有全新的方法

**工具推荐**：

* 使用pip安装来自PyPI的包
* 将virtualenv或venv用于Python环境的应用级隔离
* Python打包用户指南中，推荐包的创建与分发的工具如下：
  * 使用setuptools来定义项目并创建源代码发行版(source distributions)
  * 使用wheel而不是egg来创建构建发行版(built distrbutions)
  * 使用twine向PyPI上传包的发行版

#### 项目配置

