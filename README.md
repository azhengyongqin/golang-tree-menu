# golang-tree-menu
`golang`中实现快速菜单树的生成，包括菜单节点的选中状态、半选中状态，菜单的搜索。
@[toc]
## 1 该包提供两个方法根接口
### 1.1 GenerateTree(nodes, selectedNodes []INode) (trees []Tree)
`GenerateTree` 自定义的结构体实现 `INode` 接口后调用此方法生成树结构。
### 1.2 FindRelationNode(nodes, allNodes []INode) (respNodes []INode)
`FindRelationNode` 在 `allTree` 中查询 `nodes` 中节点的所有父子节点 返回 `respNodes`(包含 `nodes` ， 跟其所有父子节点)

### 1.3 接口 INode
```go
// ConvertToINodeArray 其他的结构体想要生成菜单树，直接实现这个接口
type INode interface {
	// GetTitle 获取显示名字
	GetTitle() string
	// GetId获取id
	GetId() int
	// GetFatherId 获取父id
	GetFatherId() int
	// GetData 获取附加数据
	GetData() interface{}
	// IsRoot 判断当前节点是否是顶层根节点
	IsRoot() bool
}
```

## 2 使用
### 2.1 定义自己的菜单结构体并且实现接口 `INode`
```go
// 定义我们自己的菜单对象
type SystemMenu struct {
	Id       int    `json:"id"`        //id
	FatherId int    `json:"father_id"` //上级菜单id
	Name     string `json:"name"`      //菜单名
	Route    string `json:"route"`     //页面路径
	Icon     string `json:"icon"`      //图标路径
}

func (s SystemMenu) GetTitle() string {
	return s.Name
}
func (s SystemMenu) GetId() int {
	return s.Id
}
func (s SystemMenu) GetFatherId() int {
	return s.FatherId
}
func (s SystemMenu) GetData() interface{} {
	return s
}
func (s SystemMenu) IsRoot() bool {
	// 这里通过FatherId等于0 或者 FatherId等于自身Id表示顶层根节点
	return s.FatherId == 0 || s.FatherId == s.Id
}
```
### 2.2 实现一个将自定义结构体`SystemMenu` 数组转换成 `INode` 数组的方法
```go
type SystemMenus []SystemMenu
// ConvertToINodeArray 将当前数组转换成父类 INode 接口 数组
func (s SystemMenus) ConvertToINodeArray() (nodes []INode) {
	for _, v := range s {
		nodes = append(nodes, v)
	}
	return
}
```

## 3 测试效果
### 3.1 添加测试数据
```go
	// 模拟获取数据库中所有菜单，在其它所有的查询中，也是首先将数据库中所有数据查询出来放到数组中，
	// 后面的遍历递归，都在这个 allMenu中进行，而不是在数据库中进行递归查询，减小数据库压力。
	allMenu := []SystemMenu{
		{Id: 1, FatherId: 0, Name: "系统总览", Route: "/systemOverview", Icon: "icon-system"},
		{Id: 2, FatherId: 0, Name: "系统配置", Route: "/systemConfig", Icon: "icon-config"},

		{Id: 3, FatherId: 1, Name: "资产", Route: "/asset", Icon: "icon-asset"},
		{Id: 4, FatherId: 1, Name: "动环", Route: "/pe", Icon: "icon-pe"},

		{Id: 5, FatherId: 2, Name: "菜单配置", Route: "/menuConfig", Icon: "icon-menu-config"},
		{Id: 6, FatherId: 3, Name: "设备", Route: "/device", Icon: "icon-device"},
		{Id: 7, FatherId: 3, Name: "机柜", Route: "/device", Icon: "icon-device"},
	}
```
### 3.2 生成完全树
```go
// 生成完全树
resp := GenerateTree(SystemMenus.ConvertToINodeArray(allMenu), nil)
bytes, _ := json.MarshalIndent(resp, "", "\t")
fmt.Println(string(bytes))
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191010190933163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIzMTc5MDc1,size_16,color_FFFFFF,t_70)
```json
[
  {
    "title": "系统总览",
    "leaf": false,
    "checked": false,
    "partial_selected": false,
    "children": [
      {
        "title": "资产",
        "leaf": false,
        "checked": false,
        "partial_selected": false,
        "children": [
          {
            "title": "设备",
            "leaf": true,
            "checked": false,
            "partial_selected": false,
            "children": null
          }, 
          {
            "title": "机柜",
            "leaf": true,
            "checked": false,
            "partial_selected": false,
            "children": null
          }
        ]
      }, 
      {
        "title": "动环",
        "leaf": true,
        "checked": false,
        "partial_selected": false,
        "children": null
      }
    ]
  }, 
  {
    "title": "系统配置",
    "leaf": false,
    "checked": false,
    "partial_selected": false,
    "children": [
      {
        "title": "菜单配置",
        "leaf": true,
        "checked": false,
        "partial_selected": false,
        "children": null
      }
    ]
  }
]
```

### 3.3 带选中状态和半选中状态的树
```go
// 模拟选中 '资产' 菜单
selectedNode := []SystemMenu{allMenu[2]}
resp = GenerateTree(SystemMenus.ConvertToINodeArray(allMenu), SystemMenus.ConvertToINodeArray(selectedNode))
bytes, _ = json.Marshal(resp)
fmt.Println(string(pretty.Color(pretty.PrettyOptions(bytes, pretty.DefaultOptions), nil)))
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019101019133053.png)
```json
[
  {
    "title": "系统总览",
    "leaf": false,
    "checked": false,
    "partial_selected": true,
    "children": [
      {
        "title": "资产",
        "leaf": false,
        "checked": true,
        "partial_selected": false,
        "children": [
          {
            "title": "设备",
            "leaf": true,
            "checked": true,
            "partial_selected": false,
            "children": null
          }, 
          {
            "title": "机柜",
            "leaf": true,
            "checked": true,
            "partial_selected": false,
            "children": null
          }
        ]
      }, 
      {
        "title": "动环",
        "leaf": true,
        "checked": false,
        "partial_selected": false,
        "children": null
      }
    ]
  }, 
  {
    "title": "系统配置",
    "leaf": false,
    "checked": false,
    "partial_selected": false,
    "children": [
      {
        "title": "菜单配置",
        "leaf": true,
        "checked": false,
        "partial_selected": false,
        "children": null
      }
    ]
  }
]
```

### 3.4 模拟查询某个节点，然后生成树
```go
// 模拟从数据库中查询出 '设备'
device := []SystemMenu{allMenu[5]}
// 查询 `设备` 的所有父节点
respNodes := FindRelationNode(SystemMenus.ConvertToINodeArray(device), SystemMenus.ConvertToINodeArray(allMenu))
resp = GenerateTree(respNodes, nil)
bytes, _ = json.Marshal(resp)
fmt.Println(string(pretty.Color(pretty.PrettyOptions(bytes, pretty.DefaultOptions), nil)))
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191010191711638.png)
```json
[
  {
    "title": "系统总览",
    "leaf": false,
    "checked": false,
    "partial_selected": false,
    "children": [
      {
        "title": "资产",
        "leaf": false,
        "checked": false,
        "partial_selected": false,
        "children": [
          {
            "title": "设备",
            "leaf": true,
            "checked": false,
            "partial_selected": false,
            "children": null
          }
        ]
      }
    ]
  }
]
```
> 源码地址：[https://github.com/azhengyongqin/golang-tree-menu](https://github.com/azhengyongqin/golang-tree-menu)
