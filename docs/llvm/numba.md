# numba

### 1、numba的安装和使用

- 重点: llvmlite

```shell
# ubuntu22 - scipy依赖指定的numpy版本
pip3 install numba

pip3 uninstall numpy
pip3 install numpy==1.21.5

numba 
  --annotate            Annotate source
  --dump-llvm           Print generated llvm assembly
  --dump-optimized      Dump the optimized llvm assembly
  --dump-assembly       Dump the LLVM generated assembly
  --annotate-html ANNOTATE_HTML
                        Output source annotation as html
  -s, --sysinfo         Output system information for bug reporting
  -g, --gdbinfo         Output system information about gdb
  --sys-json SYS_JSON   Saves the system info dict as a json file
```

### 2、简单测试性能

```shell
import numpy as np
from numba import njit
import time

# 使用 @njit 装饰器，等价于 @jit(nopython=True)
@njit
def euclidean_distance_numba(x, y):
    """计算两个向量之间的欧几里得距离"""
    dist = 0.0
    for i in range(len(x)):
        dist += (x[i] - y[i]) ** 2
    return np.sqrt(dist)

# 不使用 Numba 的纯 Python 版本作为对比
def euclidean_distance_python(x, y):
    dist = 0.0
    for i in range(len(x)):
        dist += (x[i] - y[i]) ** 2
    return np.sqrt(dist)

# 生成两个大的随机向量
size = 10_000_000
a = np.random.rand(size)
b = np.random.rand(size)

# 第一次调用会包含编译时间
_ = euclidean_distance_numba(a, b)

# 正式测试性能
start = time.time()
result_numba = euclidean_distance_numba(a, b)
end = time.time()
print(f"Numba 加速版耗时: {end - start:.4f} 秒")

start = time.time()
result_python = euclidean_distance_python(a, b)
end = time.time()
print(f"纯 Python 版耗时: {end - start:.4f} 秒")
```

### 3、导出asm/IR

```shell
# 导出asm
from numba import jit
import llvmlite.binding as llvm

@jit(nopython=True)
def myfunc(x):
    return x * x + 1

# 触发编译（传入具体类型）
myfunc(1.0)

# 获取 LLVM IR
llvm_ir = myfunc.inspect_llvm()[(numba.types.float64,)]

# 保存到文件
with open('myfunc.ll', 'w') as f:
    f.write(llvm_ir)
    
asm = myfunc.inspect_asm()[(numba.types.float64,)]
with open('myfunc.s', 'w') as f:
    f.write(asm)
    
# gcc -c myfunc.s -o myfunc.o
```

