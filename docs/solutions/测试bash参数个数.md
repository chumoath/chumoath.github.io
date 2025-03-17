# 测试bash参数个数

- 用指数测试参数个数

```shell
#!/bin/bash

args="1"
i=1
while [ $i -le 18 ]; do
    ((i++))
    args="$args $args"
done

echo $args | wc -w 
ls $args
```

- 查询最大参数个数

  ```shell
  grep -rn ARG_MAX /usr/include/linux/limits.h
  ```