<font size = 10>conda</font>

# channels

.condarc文件添加

```bash
channels:
  - default
  - conda-forge
  - bioconda
auto_activate: false

```



# 版本管理

## python3.10.4

```bash
#3.10.4
conda install baileyandrew::python
```

## libzlib 1.2.13

## openssl

```bash
# 3.52
conda install conda-forge::openssl
# 1.1.1q
conda install cctbx202208::openssl

```

## xz

```
#5.2.6
conda install cctbx202211::xz
```

# 命令

## 取消自动激活环境

```bash
conda config --set auto_activate_base false
```

## 环境目录

### 目录列表

```bash
conda config --show env dirs
```

### 添加目录

```bash
conda config --add envs_dirs E:\conda_env
```

### 删除目录

```bash
conda config --remove envs_dirs E:\conda_env
```

## 虚拟环境

### 环境列表

```bash
conda env list
conda info --envs
```



### 创建环境

```bash
conda create -n my_conda01 # 空的
conda create —name my_conda —clone base # 以环境base创建新的环境
```

### 激活环境

```bash
conda activate my_conda01
```

### 退出当前环境

```bash
conda deactivate
```

### 删除环境

```bash
conda env remove -n  my_conda
```

## 导出及导入

### 导出

```bash
conda env export > my_conda.yml
```

### 导入

```bash
conda env create -f my_conda.yml
```

## 包管理

### 查看包

### 安装包

### 更新包

### 删除包

### 数据库

#### MSSQL

```bash

```



# 报错

> The package for ucrt located at D:\Scoop\apps\miniconda3\current\pkgs\ucrt-10.0.22621.0-h57928b3_1 appears to be corrupted. The path 'api-ms-win-crt-convert-l1-1-0.dll' specified in the package manifest cannot be found.

1.删除pkgs目录

2.清理缓存

```bash
conda clean --all yes
```

3.更新/重装

```bash
conda update --all
```



