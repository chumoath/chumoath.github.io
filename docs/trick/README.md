# trick

### 1、调试init进程，或者包装其他进程

```shell
# /usr/local/bin/wget
# chmod +x /usr/local/bin/wget
#!/bin/bash
exec /usr/bin/wget "$@" --no-check-certificate
```

