# 地图集 Tilemap

TileMapLayer 节点所使用的图块集，用于定义和管理 TileMap 中的图块资源。

---

## 枚举

### enum TileShape

| 值   | 名称                              | 说明                   |
| --- | ------------------------------- | -------------------- |
| `0` | `TILE_SHAPE_SQUARE`             | 矩形图块形状。              |
| `1` | `TILE_SHAPE_ISOMETRIC`          | 钻石图块形状（用于等轴外观）。      |
| `2` | `TILE_SHAPE_HALF_OFFSET_SQUARE` | 矩形图块形状，每隔一行/列偏移半个图块。 |
| `3` | `TILE_SHAPE_HEXAGON`            | 六边形图块形状。             |
|     |                                 |                      |

> **注意：** 等轴 TileSet 在所有同级 TileMapLayer 及其派生自 Node2D 的父节点都启用了 Y 排序时效果最好。

### enum TileLayout

| 值 | 名称 | 说明 |
|---|---|---|
| `0` | `TILE_LAYOUT_STACKED` | 图块坐标布局，两个轴与对应的局部水平轴和垂直轴保持一致。 |
| `1` | `TILE_LAYOUT_STACKED_OFFSET` | 与 `TILE_LAYOUT_STACKED` 相同，但第一个半偏移偏向负方向，而不是正方向。 |
| `2` | `TILE_LAYOUT_STAIRS_RIGHT` | 图块坐标布局，水平轴保持水平，垂直轴朝向右下方。 |
| `3` | `TILE_LAYOUT_STAIRS_DOWN` | 图块坐标布局，垂直轴保持垂直，水平轴朝向右下方。 |
| `4` | `TILE_LAYOUT_DIAMOND_RIGHT` | 图块坐标布局，水平轴朝向右上方，垂直轴朝向右下方。 |
| `5` | `TILE_LAYOUT_DIAMOND_DOWN` | 图块坐标布局，水平轴朝向右下方，垂直轴朝向左下方。 |

### enum TileOffsetAxis

| 值 | 名称 | 说明 |
|---|---|---|
| `0` | `TILE_OFFSET_AXIS_HORIZONTAL` | 水平半偏移。 |
| `1` | `TILE_OFFSET_AXIS_VERTICAL` | 垂直半偏移。 |

### enum CellNeighbor

| 值 | 名称 | 说明 |
|---|---|---|
| `0` | `CELL_NEIGHBOR_RIGHT_SIDE` | 右侧相邻单元格。 |
| `1` | `CELL_NEIGHBOR_RIGHT_CORNER` | 右角相邻单元格。 |
| `2` | `CELL_NEIGHBOR_BOTTOM_RIGHT_SIDE` | 右下侧相邻单元格。 |
| `3` | `CELL_NEIGHBOR_BOTTOM_RIGHT_CORNER` | 右下角相邻单元格。 |
| `4` | `CELL_NEIGHBOR_BOTTOM_SIDE` | 下侧相邻单元格。 |
| `5` | `CELL_NEIGHBOR_BOTTOM_CORNER` | 下角相邻单元格。 |
| `6` | `CELL_NEIGHBOR_BOTTOM_LEFT_SIDE` | 左下侧相邻单元格。 |
| `7` | `CELL_NEIGHBOR_BOTTOM_LEFT_CORNER` | 左下角相邻单元格。 |
| `8` | `CELL_NEIGHBOR_LEFT_SIDE` | 左侧相邻单元格。 |
| `9` | `CELL_NEIGHBOR_LEFT_CORNER` | 左角相邻单元格。 |
| `10` | `CELL_NEIGHBOR_TOP_LEFT_SIDE` | 左上侧相邻单元格。 |
| `11` | `CELL_NEIGHBOR_TOP_LEFT_CORNER` | 左上角相邻单元格。 |
| `12` | `CELL_NEIGHBOR_TOP_SIDE` | 上侧相邻单元格。 |
| `13` | `CELL_NEIGHBOR_TOP_CORNER` | 上角相邻单元格。 |
| `14` | `CELL_NEIGHBOR_TOP_RIGHT_SIDE` | 右上侧相邻单元格。 |
| `15` | `CELL_NEIGHBOR_TOP_RIGHT_CORNER` | 右上角相邻单元格。 |

### enum TerrainMode

| 值 | 名称 | 说明 |
|---|---|---|
| `0` | `TERRAIN_MODE_MATCH_CORNERS_AND_SIDES` | 要求与相邻图块地形的角和边都匹配。 |
| `1` | `TERRAIN_MODE_MATCH_CORNERS` | 要求与相邻图块地形的角相匹配。 |
| `2` | `TERRAIN_MODE_MATCH_SIDES` | 要求与相邻图块地形的边相匹配。 |

---

## 方法

### local_to_map(position) → Vector2i

将全局位置转换为地图坐标。

- **参数：** `position`（`Vector2`）— 全局坐标位置
- **返回：** `Vector2i` — 对应的地图图块坐标
