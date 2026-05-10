# 实例化场景
## 预加载
var A: PackedScene = preload("res:/A.tscn")
PackedScene:被打包场景
preload：预加载
## 实例化
var a = A.instantiate()
## 将实例加入场景树
使用 `add_child(node)` 将新节点挂到当前节点的子节点下。
$"A集".add_child(a)
$"A集"为一个组
## 补充：移除实例
当子弹飞出屏幕或需要销毁时，调用 `queue_free()` 安全删除节点（会在帧末释放）：
a.queue_free()
## 补充2：load用法
var 场景资源 = load("res://场景/子弹.tscn")
参数：资源的路径字符串，如 "res://场景/子弹.tscn"。
返回值：如果路径有效，返回对应的资源对象，这里得到的是一个 PackedScene。如果路径错误或文件不存在，返回 null。


# 自定义信号

## 信号发出
signal AAA
信号初始化
`AAA.emit()`
信号传递
## 传递参数
`extends Node2D`

`signal AAA(BB, CC)`

`var B = 10`
`func take_damage(amount):`
	`var C = B`
	`B -= 0`
	AAA.emit(B, C)`
## 信号连接
`AAA.connect(_AA)`

`_AA():`

