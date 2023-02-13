## 数据库表设计

### 简介

- 构成
  - 字典组 -> key-value

- 字典可以做缓存，使用redis和lfu缓存淘汰策略


### 实现

- 字典组：

  - | 字段名   | 类型    | 描述                      |
    | -------- | ------- | ------------------------- |
    | id       | int     | id                        |
    | type     | varchar | 字典组名                  |
    | show     | char    | 表现形式 0 列表 1 树形    |
    | category | varchar | 字典类型 系统内置、业务类 |
    | des      | varchar | 描述                      |
    | remark   | varchar | 备注                      |

- 字典项

  - | 字段名 | 类型 | 描述        |
    | ------ | ---- | ----------- |
    | id     |      |             |
    | pid    |      | 字典组id    |
    | type   |      | 字典组名    |
    | key    |      | 字典项key   |
    | value  |      | 字典项value |
    | sort   |      | 排序值      |
    | des    |      | 描述        |
    | remark |      | 备注        |

- Controller
  - 新增字典组
  - 分页查询字典组信息
  - 修改字典组
  - 通过id删除字典组
  - 新增字典项
  - 分页查询字典项信息
  - 修改字典项
  - 树形查询字典项信息
  - 通过id删除字典项
  - 获取字典值 value
  - 获取key-value
  - 获取树形的key-value
  - 获取key-value 无权限
  - 获取字典值 value 无权限（无权限组）

- service

- Impl
  - 注入DictServiceImpl（可以通过ApplicationContext）    

- Mapper
  - SysDictGroupMapper
  - SysDictMapper



