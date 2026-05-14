# AStar3D
A*（A 星）是一种用于寻路和图遍历的计算机算法，会根据一组给定的边（线段）测定顶点（点）之间的短路径。由于高性能和高准确度，A* 算法被广泛使用。Godot 的 A* 实现默认使用 3D 空间中的点和欧几里德距离。
你必须使用 add_point() 手动添加点，并使用 connect_points() 手动创建线段。完成后，就可以使用 are_points_connected() 函数测试两点之间是否存在路径，通过 get_id_path() 获取由索引组成的路径，或使用 get_point_path() 获取由实际坐标组成的路径。
也可以使用非欧几里德距离。做法是创建一个扩展自 AStar3D 的类，然后覆盖 _compute_cost() 和 _estimate_cost() 方法。两者都接受两个点 ID 并返回对应点之间的距离。
## `void add_point(id: int, position: Vector3, weight_scale: float = 1.0)`

在给定的位置添加一个新的点，并使用给定的标识符。id 必须大于等于 0，weight_scale 必须大于等于 0.0。

在确定从邻点到此点的一段路程的总成本时，weight_scale 要乘以 _compute_cost() 的结果。因此，在其他条件相同的情况下，算法优先选择 weight_scale 较低的点来形成路径。
## get_point_connections(id: int)`

返回一个数组（PackedInt64Array），其中包含与给定点形成连接的点的 ID。
## get_id_path(from_id: int, to_id: int, allow_partial_path: bool = false)`

返回一个数组（PackedInt64Array），其中包含构成 AStar3D 在给定点之间找到的路径中的点的 ID。数组从路径的起点到终点排序。

如果没有能够到达目标的有效路径，并且 allow_partial_path 为true，则返回能够到达的最接近目标点的路径。


# AStar2D
A* 算法的一种实现，用于在 2D 空间中的连通图上找到两个顶点之间的最短路径。
AStar2D 是 AStar3D 的包装器，它强制执行 2D 坐标。

# AStarGrid2D
AStarGrid2D 是 AStar2D 的变种，针对疏松 2D 网格进行了优化。因为不需要手动创建点并进行连接，所以用起来更加简单。这个类还支持使用不同的启发方法、斜向移动模式、跳跃模式，从而加速运算。

要使用 AStarGrid2D，你只需要设置网格的 region，cell_size 可以不设置，最后调用 update() 方法即可：

`var astar_grid = AStarGrid2D.new()`
`astar_grid.region = Rect2i(0, 0, 32, 32)`
`astar_grid.cell_size = Vector2(16, 16)`
`astar_grid.update()`
`print(astar_grid.get_id_path(Vector2i(0, 0), Vector2i(3, 4))) 
#`输出 [(0, 0), (1, 1), (2, 2), (3, 3), (3, 4)]`
`print(astar_grid.get_point_path(Vector2i(0, 0), Vector2i(3, 4))) 
#`输出 [(0, 0), (16, 16), (32, 32), (48, 48), (48, 64)]` 
要从寻路网格中移除某个点，必须使用 set_point_solid() 将其设置为“实心”。

# 如何使用
.new()   初始化
.region  总大小
.cell_size  单位大小
.get_id_path（）得到路径
.set_point_solid（） 不可通过
# 寻路方法
## enum Heuristic:

● HEURISTIC_EUCLIDEAN = 0
欧几里德启发式算法  将被用于寻路，使用的公式如下：

dx = abs(to_id.x - from_id.x)
dy = abs(to_id.y - from_id.y)
result = sqrt(dx * dx + dy * dy) 
注意：这也是 AStar3D 和 AStar2D 默认使用的内部启发式算法（包括可能的 z 轴坐标）。

● HEURISTIC_MANHATTAN = 1
曼哈顿启发式算法  将被用于寻路，使用的公式如下：

dx = abs(to_id.x - from_id.x)
dy = abs(to_id.y - from_id.y)
result = dx + dy 
注意：该启发式算法旨在与 4 边正交运动一起使用，4 边正交运动可通过将 diagonal_mode 设置为 DIAGONAL_MODE_NEVER 来提供。

● HEURISTIC_OCTILE = 2
Octile 启发式算法将被用于寻路，使用的公式如下：

dx = abs(to_id.x - from_id.x)
dy = abs(to_id.y - from_id.y)
f = sqrt(2) - 1
result = (dx < dy) ? f * dx + dy : f * dy + dx; 

● HEURISTIC_CHEBYSHEV = 3
切比雪夫启发式算法  将被用于寻路，使用的公式如下：

dx = abs(to_id.x - from_id.x)
dy = abs(to_id.y - from_id.y)
result = max(dx, dy) 

● HEURISTIC_MAX = 4
代表 Heuristic 枚举的大小。

| 运动模式                  | 选启发方式                |
| --------------------- | -------------------- |
| 只允许上下左右（四方向）          | **MANHATTAN**//**1** |
| 允许对角线移动（八方向）          | **OCTILE**//**2**    |
| 任意角度 / 单位可以走斜线但要求严格最优 | **EUCLIDEAN**//**0** |
| 调试 / 不在意路径质量          | **CHEBYSHEV**//**3** |
在实际项目里，**最常用的是 `MANHATTAN`（四方向）和 `OCTILE`（八方向）**。

# 实例
