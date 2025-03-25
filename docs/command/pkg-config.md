# pkg-config

## 1、basic

```shell
# 查看指定模块的变量
pkg-config --variable [variable] [module]

# 查看默认的*.pc文件搜索路径
pkg-config --variable pc_path pkg_config

# 查看指定模块的*.pc文件中定义的变量
pkg-config --variable prefix sdl2

# 输出链接参数
pkg-config --libs sdl2

# 输出预处理和编译参数
pkg-config --cflags slirp

# 列出指定模块的*.pc文件中定义的所有变量
pkg-config --print-variables sdl2

# 查看pkg-config内部变量
pkg-config --print-variables pkg-config

# 打印该模块提供的包及版本号
pkg-config --print-provides sdl2

# 查看pkg-config的解析流程
pkg-config --debug --libs sdl2
```

