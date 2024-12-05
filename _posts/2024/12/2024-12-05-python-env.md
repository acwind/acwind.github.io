---

title: Python 虚拟环境（Virtual Environment）使用方法
categories:
- 运维人生
key: f49c35ea-1ac7-11ed-861d-0242ac120114


---

<img src="https://images.animesdata.com.ap-south-1.linodeobjects.com/news/2024/12/05/python-env.png" width="500" />

最近捣鼓安装各种工具的时候，python的库之间的版本冲突，导致一些工具无法正常使用，所以研究了一下Python 虚拟环境的使用方法，喊AI写了一个帮助文档，放在这里给自己备忘。

在现代 Python 开发中，虚拟环境（Virtual Environment，简称 env）是不可或缺的工具。它能帮助开发者有效地管理项目依赖，避免冲突，并确保项目的可移植性。本篇文章将详细介绍 Python 虚拟环境的用途、创建与使用方法、导出与退出方式，以及一些常见注意事项。

## 1. 虚拟环境的用途

### 1.1 什么是虚拟环境？

虚拟环境是一个隔离的 Python 运行环境，它允许每个项目独立管理自己的依赖，而不会影响全局 Python 环境或其他项目。

### 1.2 为什么需要虚拟环境？

- **依赖隔离：**
不同项目可能需要不同版本的库。例如：
  - 项目 A 需要 Django 3.2。
  - 项目 B 需要 Django 4.0。
虚拟环境能让它们共存而互不干扰。

- **避免污染全局环境：**
全局安装的库可能导致系统环境混乱，甚至出现版本冲突。虚拟环境可避免这些问题。

- **便于迁移和部署：**
虚拟环境记录了项目所需的所有依赖，方便将项目迁移到其他机器或服务器。

## 2. 创建虚拟环境

从 Python 3.3 开始，venv 已成为标准库，用于创建虚拟环境。

### 2.1 创建虚拟环境

在项目目录下执行以下命令：

```bash
python3 -m venv myenv
```

- `python3`：调用 Python 解释器。
- `-m venv`：使用 venv 模块创建虚拟环境。
- `myenv`：虚拟环境的目录名，可自定义。

成功后，当前目录下会生成一个名为 myenv 的文件夹，包含以下内容：
- `bin/`（Linux/macOS）或 `Scripts/`（Windows）：包含虚拟环境的可执行文件。
- `lib/`：存放虚拟环境安装的库。
- `pyvenv.cfg`：虚拟环境的配置信息。

## 3. 激活和使用虚拟环境

### 3.1 激活虚拟环境

在创建虚拟环境后，需要激活它，以便运行 Python 和安装库时使用虚拟环境的环境。

- Linux/macOS：
```bash
source myenv/bin/activate
```

- Windows：
```bash
myenv\Scripts\activate
```

激活后，终端提示符会变为类似以下形式，表明虚拟环境已激活：

```bash
(myenv) $
```

### 3.2 在虚拟环境中安装依赖

虚拟环境激活后，可以使用 pip 安装项目依赖。例如：

```bash
pip install requests
```

- 已安装的库会保存在虚拟环境中，不会影响全局环境。
- 通过 `pip list` 查看当前虚拟环境中的所有依赖。

## 4. 导出和恢复环境

### 4.1 导出环境依赖

为了记录项目的所有依赖，通常会将依赖列表导出到 requirements.txt 文件中：

```bash
pip freeze > requirements.txt
```

生成的 requirements.txt 文件内容如下：

```
requests==2.28.1
numpy==1.23.5
```

### 4.2 恢复环境依赖

在迁移或部署时，可以通过 requirements.txt 文件快速恢复依赖：

```bash
pip install -r requirements.txt
```

## 5. 退出和删除虚拟环境

### 5.1 退出虚拟环境

执行以下命令可以退出虚拟环境，恢复到全局环境：

```bash
deactivate
```

退出后，终端提示符中的虚拟环境名称会消失。

### 5.2 删除虚拟环境

虚拟环境本质上是一个文件夹，删除它只需移除对应的目录：

```bash
rm -rf myenv
```

## 6. 注意事项

### 6.1 Python 版本一致性

确保创建虚拟环境时的 Python 版本与目标环境兼容。可以通过以下命令指定 Python 版本：

```bash
python3.9 -m venv myenv
```

### 6.2 虚拟环境大小

虚拟环境包含独立的 Python 解释器和库文件，可能占用较多磁盘空间。通过使用 --system-site-packages 共享全局库，可以节省空间：

```bash
python3 -m venv myenv --system-site-packages
```

### 6.3 跨平台兼容性

虚拟环境通常绑定当前操作系统。如果需要跨平台（如从 macOS 部署到 Linux），建议使用 requirements.txt 和目标系统重新创建虚拟环境。

### 6.4 IDE 支持

主流 IDE（如 PyCharm、VS Code）都支持虚拟环境。可以在项目设置中选择虚拟环境作为解释器。