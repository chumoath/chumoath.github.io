# phosphor-objmgr

## 一、导出到dbus上各方法

#### 1. GetAncestors

```shell
# 获取该 path 的所有父path，且父path的接口 和 interfaces 有交集
# 注意：父对象path会存在于很多个app，比如：/xyz/openbmc_project/
args: GetAncestors(std::string reqPath, std::vector<std::string>& interfaces)
# [ 父对象路径 : [ 父对象所在app的busName : 所在connection父对象的所有接口 ] ]
return: { path : { connection : [ interfaces ] } }

dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetAncestors string:"/xyz/openbmc_project/sensors/temperature" array:string:

dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetAncestors string:"/xyz/openbmc_project/sensors/temperature" array:string:"org.freedesktop.DBus.Introspectable",string:"org.freedesktop.DBus.Peer"

# 伪代码：
array output;
for path : all_path:
    if reqPath.starts_with(path):
    	output.append(path)
```

#### 2. GetObject

```shell
# 获取包含该 path 的所有 [ connection : path 的interfaces ]
args: GetObject( std::string path, std::vector<std::string> interfaces )
return: { connection : [ interfaces ] }

# 注意: 对象路径最后不能有 /
dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetObject string:"/xyz/openbmc_project/sensors/temperature" array:string:
```

#### 3. GetSubTree

```shell
# 获取该path的所有子对象；根据子对象后面的 '/' 数量判断当前深度；若 depth <= 0，则depth设置为 int::max
args: GetSubTree( std::string reqPath, int32_t depth, std::vector<std::string> interfaces)
return: { path : { connection : [ interfaces ] } }

dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetSubTree string:"/xyz/openbmc_project/sensors/temperature" int32:0 array:string:"xyz.openbmc_project.Sensor.Value"

# 伪代码：
array output;
for path : all_path:
    if path.starts_with(reqPath) and right_slash_num <= depth:
    	output.append(path)
```

#### 4. GetSubTreePaths

```shell
# 同 GetSubTree，但只返回 子对象 path，不获取其所在 connection 和 其interfaces
args: GetSubTree( std::string reqPath, int32_t depth,
 std::vector<std::string> interfaces )
return: [ path ]

dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetSubTree string:"/xyz/openbmc_project/sensors/temperature" int32:0 array:string:
```

#### 5. GetSubTreePathsById

```shell
# id一定是一个实体，即一个实际的物理对象，比如 chassis 或 sensor 的名字
# 通过id获取子对象path，函数没有导出到dbus，只在objmgr内部使用；
# 枚举所有对象，路径叶子名称 和 参数id 匹配的，再和父对象路径匹配
# 注意：只有 GetSubTreePathsById 没有对参数 interfaces 为空的处理，所以 getAssociatedSubTreeById 的 subtreeInterfaces不能为空，其他的接口的 interfaces 都可以为空

args: GetSubTreePathsById( std::string id, std::string objectPath, std::vector<std::string> interfaces )
return: [ path ]

# 导出到Dbus上后，可如下使用
dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetSubTreePathsById string:"chassis" string:"/xyz/openbmc_project/sensors/temperature" array:string:

# 伪代码：
array output;
for path : all_path:
    if path.ends_with("/" + id):
    	if path.starts_with(reqPath):
    		output.append(path)
```

#### 6. GetAssociatedSubTree

```shell
# associationPath 为 有 xyz.openbmc_project.Association 接口的对象，其有 endpoints 属性
# reqPath 为 要执行 GetSubTree 的对象路径
# 流程
# 1. 根据 associationPath 获取其 endpoints 属性
# 2. 使用 reqPath 获取所有子对象
# 3. 返回 在endpoints 中的所有子对象
args: GetAssociatedSubTree( object_path associationPath, object_path reqPath, int32_t depth, std::vector<std::string> interfaces )
return: { path : { connection : [ interfaces ] } }

dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetAssociatedSubTree objpath:"/xyz/openbmc_project/inventory/system/board/Virt_Board/all_sensors" objpath:"/xyz/openbmc_project/sensors" int32:0 array:string:""

# 伪代码：
array output;
endpoints = associationPath.[xyz.openbmc_project.Association].endpoints
subpaths = GetSubTree(reqPath, depth, interfaces)
for path : subpaths:
	if endpoints.contains(path):
		output.append(path)
```

#### 7. GetAssociatedSubTreePaths

```shell
# reqPath 为要执行 GetSubTreePaths 的对象路径；只返回匹配的对象路径，没有 connection_name 和 interfaces
args: GetAssociatedSubTreePaths( object_path associationPath, object_path reqPath, int32_t depth, std::vector<std::string> interfaces )
return: [ path ]

dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetAssociatedSubTreePaths objpath:"/xyz/openbmc_project/inventory/system/board/Virt_Board/all_sensors" objpath:"/xyz/openbmc_project/sensors" int32:0 array:string:""
```

#### 8. GetAssociatedSubTreeById

```shell
# id一定是一个实体，即一个实际的物理对象，比如 chassis 或 sensor 的名字
# 通过 id 获取 associationPath，再调用 GetAssociatedSubTree
# 流程：
# 1. 用 id、objectPath、subtreeInterfaces，调用 GetSubTreePathsById，获取 (1)叶子对象名为id; (2)是objectPath的子路径;(3)包含subtreeInterfaces其中一个接口; 的对象；
# 2. 使用之前获取的对象路径 和 association 拼成 associationPath
# 3. 调用 GetAssociatedSubTree 获取所有 (1)在 associationPath 的 endpoints中；(2)是objectPath的子对象； 的 对象，这些子对象必须包含 endpointInterfaces 中的一个

args: GetAssociatedSubTreeById( std::string id, std::string objectPath, std::vector<std::string> subtreeInterfaces, std::string association, std::vector<std::string> endpointInterfaces )
return: { path : { connection : [ interfaces ] } }

dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetAssociatedSubTreeById string:"Virt_Board" string:"/xyz/openbmc_project" array:string:"xyz.openbmc_project.Inventory.Item.Board" string:"all_sensors" array:string:""

# 伪代码：
array output;
entity_path = GetSubTreePathsById(id, objectPath, subtreeInterfaces)
associationPath = entity_path + "/" + association

endpoints = associationPath.[xyz.openbmc_project.Association].endpoints
subpaths = GetAssociatedSubTree(associationPath, objectPath, 0, endpointInterfaces)
output.append(subpaths)
```

#### 9. GetAssociatedSubTreePathsById

```shell
# id一定是一个实体，即一个实际的物理对象，比如 chassis 或 sensor 的名字
# 调用 getAssociatedSubTreePaths

args: GetAssociatedSubTreePathsById( std::string id, std::string objectPath, std::vector<std::string> subtreeInterfaces, std::string association, std::vector<std::string> endpointInterfaces )
return: [ path ]

dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetAssociatedSubTreePathsById string:"Virt_Board" string:"/xyz/openbmc_project" array:string:"xyz.openbmc_project.Inventory.Item.Board" string:"all_sensors" array:string:""

dbus-send --system --print-reply --dest=xyz.openbmc_project.ObjectMapper /xyz/openbmc_project/object_mapper  xyz.openbmc_project.ObjectMapper.GetAssociatedSubTreePathsById string:"virt_temp" string:"/xyz/openbmc_project" array:string:"xyz.openbmc_project.Sensor.Value" string:"chassis" array:string:""

# 伪代码：
array output;
entity_path = GetSubTreePathsById(id, objectPath, subtreeInterfaces)
associationPath = entity_path + "/" + association

endpoints = associationPath.[xyz.openbmc_project.Association].endpoints
subpaths = getAssociatedSubTreePaths(associationPath, objectPath, 0, endpointInterfaces)
output.append(subpaths)
```

