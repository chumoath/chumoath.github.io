# cstool

- apt install capstone-tool
- cstool问题：指令显示的顺序和在内存中的相反，大小端
- `cstool arm64be d65f03c0`
- `cstool arm64 c0035fd6`
- `cstool -d arm64be 97ed9702 0xffff8000807676fc`