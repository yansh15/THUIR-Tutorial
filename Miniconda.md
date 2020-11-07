# Miniconda

> Miniconda下载地址
>
> Linux： [Miniconda3-latest-Linux-x86_64.sh](https://mirrors.bfsu.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh)
>
> macOS： [Miniconda3-latest-MacOSX-x86_64.sh](https://mirrors.bfsu.edu.cn/anaconda/miniconda/Miniconda3-latest-MacOSX-x86_64.sh)
>
> Windows：[Miniconda3-latest-Windows-x86_64.exe](https://mirrors.bfsu.edu.cn/anaconda/miniconda/Miniconda3-latest-Windows-x86_64.exe)

## 基本概念
### Conda
Conda是在Windows、macOS和Linux上运行的开源包管理系统和环境管理系统。

作为包管理系统，Conda可以快速安装或更新软件包及其依赖项。尽管Conda通常用来管理Python包，但也可以管理其他软件或者语言包（比如cudatoolkit）。

作为环境管理系统，Conda可以轻松创建、加载与切换虚拟环境。虚拟环境可以视为一个完全独立的Python运行环境，拥有独立的Python版本与软件包，不需要担心污染系统环境或者其他虚拟环境。

Anaconda与Miniconda都可以提供`conda`命令，但Anaconda中会默认附带大量软件包而Miniconda仅包含Python解释器与管理系统功能。因此通常使用Miniconda来获取`conda`命令。

### Package（包）
Package（包）可以是Python或者其他语言的包。

### Channel（通道）
Channel（通道）是存储包的位置，通常形式为一个远程URL，标明包的存储目录。Conda会在指定的一组通道中按照优先级搜索包。默认设置的通道URL实际位置在国外，连接速度较慢，因此可以设置为国内镜像的通道URL。

### Environment（环境）
Environment（环境）是一系列包的集合（注：Python解释器本身也是一个包）。不同的虚拟环境之间互相独立，且与系统环境独立。这意味着可以使用与系统环境不同版本的Python解释器，且为每个虚拟环境安装不同种类不同版本的包，不需要担心改变一个虚拟环境会影响另外一个虚拟环境。

### `conda` VS `pip`
在虚拟环境中，`conda`命令与`pip`命令都可以安装包，但建议使用`conda`命令，除非某个包只发布在Pypi而不在任何通道中。`conda`命令的优势在于可以安装Python语言之外的包且更好地维护包之间的依赖关系与兼容性。

## Conda目录结构
* <ROOT_DIR>（Miniconda安装目录）
	* envs（环境目录）
		* <ENV_NAME_1>（某一环境目录）
			* bin（可执行文件目录）
				* python（该环境的Python解释器）
				* ……
			* conda-meta（元数据目录）
				* ……
		* ……
	* ……
## 配置文件
### 全局配置
在用户目录下建立文件`.condarc` ，写入如下内容。
1. 开启conda自动更新；
2. 在默认通道前添加`conda-forge`、`pytorch`与`aibox`通道；
	1. `conda-forge`是一个Github组织，其通道由社区主导，包含了绝大多数常用包且更新及时，可以作为默认通道的替代；
	2. `pytorch`是PyTorch官方维护的通道，提供最新的PyTorch与相关包；
	3. `aibox`提供了开源推荐系统框架`RecBole`；
3. 通道优先级设置为`strict`，如果在高级别通道中查找到软件包，则不再搜索低级别通道，提高搜索速度且降低冲突可能性；
4. 除`aibox`通道（无国内镜像）外，将所有其他通道设置为BFSU镜像站通道URL。
```
auto_update_conda: true
channels:
  - conda-forge
  - pytorch
  - aibox
  - defaults
show_channel_urls: true
channel_priority: strict
default_channels:
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/main
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/free
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/r
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/pro
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.bfsu.edu.cn/anaconda/cloud
  pytorch: https://mirrors.bfsu.edu.cn/anaconda/cloud
```

### 环境配置
在环境目录下的`conda-meta`目录下，可以通过`pinned`文件避免部分包被更新。该文件格式如下例：
```
cudatoolkit 10.1.243
pytorch 1.7.0
```

## 常用命令
### 基本命令
* 查看版本：`conda --version`
* 升级Conda：`conda update -n base conda`
* 清除缓存与不使用的包：`conda clean -a`

### 环境相关命令
* 创建环境：`conda create -n {env_name} python={python_version}`
* 查看环境：`conda env list`
* 激活环境：`conda activate {env_name}`
* 退出环境：`conda deactivate`
* 删除环境：`conda remove -n {env_name} --all`

### 包相关命令
> 使用本节命令时需要激活目标虚拟环境！

不建议直接使用`conda update --all`升级环境内所有包，可能会将CUDA升级至最新版但服务器的NVIDIA驱动版本不支持，虽然可以使用`pinned`文件固定cudatoolkit版本，但可能也会因为部分软件包升级导致无法精确复现结果。

安装包时可以指定版本，例如`{pkg_name}={version}`则安装指定版本，如果不指定则安装最新版。
* 打印当前环境包列表：`conda list`
* 搜索包：`conda search {pkg_name}`
* 安装包：`conda install {pkg_name} [-c {channel_name}]`
* 升级包：`conda update {pkg_name} [-c {channel_name}]`
* 删除包：`conda uninstall {pkg_name}`

### 克隆虚拟环境
> 由于包版本与依赖关系在不同系统中可能不同，本节命令只适用于在相同系统与相同指令集架构中使用。

在不同的服务器上克隆环境可以保证环境完全一致，避免因为环境不同引起结果误差。
* 获得环境包列表文件（需要提前激活环境）：`conda list --explicit > {pkg_list_filename}`
* 根据包列表文件建立环境：`conda create -n {env_name} --file {pkg_list_filename}`

## 常用环境
### Python-3.8 + PyTorch-1.7 + JupyterLab（CPU）
```
conda create -n pytorch-1.7 python=3.8
conda activate pytorch-1.7
conda install pytorch=1.7 recbole jupyterlab
```

### Python-3.8 + PyTorch-1.7 + JupyterLab（GPU）
只适用于deeper、deepest、fast服务器，deep服务器因为NVIDIA驱动版本过低无法安装CUDA10.1。
```
conda create -n pytorch-1.7 python=3.8
conda activate pytorch-1.7
conda install pytorch=1.7 cudatoolkit=10.1 recbole jupyterlab
```
下面给出一份2020.11.08生成的包列表：[pkgs.txt](https://cloud.tsinghua.edu.cn/f/7c233332008348cf83d1/)。
