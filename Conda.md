<font size = 10>conda</font>

# 命令

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





