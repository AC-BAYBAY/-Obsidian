---
tags:
  - godot
  - csv
  - data-driven
created: 2026-05-18
description: Godot 中使用 CSV 管理物品属性与多语言支持的完整方案
---

# 使用 CSV 管理 Godot 中物品属性（含多语言支持）

## 一、核心思路

**数据驱动设计**：将游戏内容和逻辑分离。物品属性、名称、介绍等数据存放在 CSV 文件中，Godot 引擎只负责读取和呈现。

**优势**：

| 特性 | 说明 |
|---|---|
| **热更新** | 修改 CSV 后无需重新编译，游戏运行时自动加载最新数据 |
| **协作友好** | 策划用 Excel 维护数值，程序员专注功能开发 |
| **版本控制** | CSV 是纯文本格式，比二进制资源更适合 Git 管理变更 |
| **扩展性强** | 新增物品只需在表格加一行，不用动代码 |

---

## 二、CSV 文件的格式要求

### 2.1 编码要求

CSV 文件必须使用 **UTF-8 编码**保存，且**不带字节序标记（BOM）**。

> [!warning]
> Microsoft Excel 默认以 ANSI 编码保存 CSV，无法直接保存为 UTF-8。推荐使用 **LibreOffice** 或 **Google Sheets** 来编辑和导出 CSV 文件。

### 2.2 通用 CSV 数据表格式（物品属性）

首行是**列名（字段名）**，后续每行代表一个物品：

```csv
id,name,type,damage,price,icon_path,stackable
1,木剑,weapon,5,50,res://assets/icon/wood_sword.png,false
2,铁剑,weapon,15,200,res://assets/icon/iron_sword.png,false
3,治疗药水,consumable,0,100,res://assets/icon/potion_red.png,true
```

> **要点**：首行字段名将作为字典的键名使用，建议使用英文命名避免兼容性问题。

### 2.3 多语言翻译 CSV 格式

Godot 官方支持 CSV 翻译导入。翻译表的格式如下：

- 第一列是**字符串唯一标识符**（key），通常使用全大写字母
- 第一行是表头，从第二列开始是各语言的 locale 代码（如 `en`、`zh`、`ja` 等）
- 左上角单元格可留空或填任意内容，会被忽略

**翻译表示例**：

```csv
keys,en,zh,ja,de
ITEM_SWORD_NAME,Iron Sword,铁剑,鉄の剣,Eisenschwert
ITEM_SWORD_DESC,A sturdy iron blade.,一把坚固的铁剑。,頑丈な鉄の刃。,Eine robuste Eisenklinge.
ITEM_POTION_NAME,Health Potion,治疗药水,回復薬,Heiltrank
ITEM_POTION_DESC,Restores 50 HP.,恢复50点生命值。,HPを50回復する。,Stellt 50 LP wieder her.
```

**注意事项**：

1. 各语言列的 locale 代码必须是引擎支持的**有效区域设置**（如 `en`、`zh`、`ja`、`fr`、`de` 等），否则不会导入
2. 以 `_` 开头的列会被视为**注释列**，不会被导入，可用于内部备忘
3. 如果单元格内容包含逗号、换行符或双引号，需要用双引号包裹，内部双引号需用两个双引号转义
4. KEY 区分大小写，`ITEM_SWORD` 和 `item_sword` 是不同的键

> [!tip] 详细指南
> 关于 CSV 翻译导入器的完整使用教程（含导入设置、上下文、复数形式等高级特性），请参阅 [[GODOT/CSV翻译|CSV 翻译]] 笔记。

---

## 三、两表分离策略：数据表 + 翻译表

### 3.1 设计理念

推荐将物品数据拆分为两类 CSV 文件：

| 文件 | 用途 | 内容 |
|---|---|---|
| `items_data.csv` | 物品的游戏逻辑属性 | id、type、damage、price、icon_path 等 |
| `items_text.csv` | 物品的多语言显示文本 | 各语言下的名称、描述等 |

**好处**：

- **职责分离**：数值策划关注数据表，翻译人员/文案关注翻译表，互不干扰
- **复用灵活**：同一套翻译机制可用于 UI 字符串、对话文本等
- **符合 Godot 内置机制**：翻译表使用 Godot 的 CSV 翻译导入器自动处理，数据表则通过脚本手动解析

### 3.2 通过 KEY 关联

在两个表中使用统一的物品标识符（如 `ITEM_SWORD`）作为关联键：

- `items_data.csv` 中使用此 KEY 作为行的唯一标识
- `items_text.csv` 中使用此 KEY 作为翻译键

---

## 四、在 Godot 中读取和处理 CSV

### 4.1 方式一：手动解析 CSV（物品数据属性）

使用 `FileAccess` 类读取 CSV 文件：

```gdscript
extends Node

## 解析 CSV 文件并返回字典
## @path: CSV 文件路径
## @key_column: 用哪一列的值作为字典的键（如 "id" 或 "name"）
func parse_csv(path: String, key_column: String) -> Dictionary:
	var file = FileAccess.open(path, FileAccess.READ)
	if file == null:
		push_error("无法打开文件: %s" % path)
		return {}

	var result = {}

	# 读取表头行
	var headers = file.get_csv_line()
	if headers.size() == 0:
		return {}

	# 找到 key_column 在表头中的索引
	var key_index = headers.find(key_column)
	if key_index == -1:
		push_error("找不到列: %s" % key_column)
		return {}

	# 逐行读取数据
	while !file.eof_reached():
		var row = file.get_csv_line()
		if row.size() == 0:
			continue

		# 将每行数据构建为字典
		var item_data = {}
		for i in range(min(headers.size(), row.size())):
			item_data[headers[i]] = row[i]

		# 以指定列的值作为键
		var key = row[key_index]
		result[key] = item_data

	file.close()
	return result
```

**使用示例**：

```gdscript
func _ready():
	var items = parse_csv("res://data/items_data.csv", "id")
	print(items["1"])           # 输出木剑的全部属性
	print(items["1"]["damage"]) # 输出 "5"
```

> [!note] Godot 的 `FileAccess.get_csv_line()` 方法会自动处理逗号分隔和双引号转义，比手动 `split(",")` 更可靠。

### 4.2 方式二：使用 Godot 内置 CSV 翻译导入器（多语言文本）[[CSV翻译]]

将 `items_text.csv` 放入项目文件夹后，Godot 会自动识别为翻译文件并导入。导入后会在同级目录生成 `.translation` 资源文件。

**使用翻译文本**：

```gdscript
# 在脚本中使用 tr() 函数获取翻译字符串
func get_item_name(key: String) -> String:
	return tr(key)

func _ready():
	print(tr("ITEM_SWORD_NAME")) # 根据当前语言输出 "铁剑" 或 "Iron Sword"
```

**切换语言**：

```gdscript
# 在项目设置 -> 本地化 -> 翻译中添加 CSV 文件后
# 可以通过代码切换语言
TranslationServer.set_locale("zh")
```

翻译 CSV 导入后会自动添加到 `project.godot` 的翻译列表中，并在游戏运行时加载。

---

## 五、完整工作流示例

### 5.1 文件结构

```
res://
├── data/
│   ├── items_data.csv        # 物品属性数据
│   └── items_text.csv        # 物品多语言文本（Godot 自动导入为翻译）
├── scripts/
│   └── item_database.gd      # 物品数据管理脚本（自动加载）
├── scenes/
│   └── inventory.tscn        # 背包场景
└── assets/
    └── icon/
        ├── wood_sword.png
        └── potion_red.png
```

### 5.2 items_data.csv（物品属性表）

```csv
id,name_key,type,damage,price,icon_path,stackable
1,ITEM_SWORD,weapon,5,50,res://assets/icon/wood_sword.png,false
2,ITEM_POTION,consumable,0,100,res://assets/icon/potion_red.png,true
```

> `name_key` 列存储的是翻译表中的 KEY，而非直接存储显示名称。

### 5.3 items_text.csv（翻译表）

```csv
keys,en,zh,ja
ITEM_SWORD,Iron Sword,铁剑,鉄の剣
ITEM_POTION,Health Potion,治疗药水,回復薬
```

### 5.4 item_database.gd（自动加载脚本）

```gdscript
extends Node

var items := {}
var current_locale := "zh"

func _ready():
	items = parse_csv("res://data/items_data.csv", "id")

## 获取物品的属性数据
func get_item(item_id: String) -> Dictionary:
	return items.get(item_id, {})

## 获取物品的本地化名称
func get_item_name(item_id: String) -> String:
	var item = get_item(item_id)
	if item.is_empty():
		return ""
	var name_key = item.get("name_key", "")
	return tr(name_key) if name_key else ""

## 解析 CSV 的方法
func parse_csv(path: String, key_column: String) -> Dictionary:
	var file = FileAccess.open(path, FileAccess.READ)
	if file == null:
		push_error("无法打开文件: %s" % path)
		return {}
	var result = {}
	var headers = file.get_csv_line()
	var key_index = headers.find(key_column)
	if key_index == -1:
		push_error("找不到列: %s" % key_column)
		return {}
	while !file.eof_reached():
		var row = file.get_csv_line()
		if row.size() == 0:
			continue
		var item_data = {}
		for i in range(min(headers.size(), row.size())):
			item_data[headers[i]] = row[i]
		result[row[key_index]] = item_data
	file.close()
	return result
```

### 5.5 在场景中使用

```gdscript
# 获取物品数据并显示本地化名称
var item = ItemDatabase.get_item("1")
if not item.is_empty():
	var display_name = ItemDatabase.get_item_name("1")  # 根据当前语言自动返回
	print(display_name)  # 中文环境输出 "铁剑"
```

---

## 六、进阶：使用 Resource 系统结合 CSV

如果物品属性较复杂（嵌套结构、资源引用等），可以将 CSV 解析后转换为 Godot 的 Resource 对象，获得更好的编辑器集成体验。

**定义物品 Resource 脚本**（`item_resource.gd`）：

```gdscript
class_name ItemResource extends Resource
@export var id: int
@export var name_key: String
@export var type: String
@export var damage: int
@export var price: int
@export var icon: Texture2D
@export var stackable: bool
```

**CSV 到 Resource 的转换**：

```gdscript
func csv_to_resources(path: String) -> Array[ItemResource]:
	var raw_data = parse_csv(path, "id")
	var resources: Array[ItemResource] = []

	for key in raw_data:
		var data = raw_data[key]
		var res = ItemResource.new()
		res.id = int(data.get("id", "0"))
		res.name_key = data.get("name_key", "")
		res.type = data.get("type", "")
		res.damage = int(data.get("damage", "0"))
		res.price = int(data.get("price", "0"))
		res.icon = load(data.get("icon_path", ""))
		res.stackable = data.get("stackable", "false") == "true"
		resources.append(res)

	return resources
```

---

## 七、CSV 导入注意事项

1. **Godot 4 默认将 CSV 视为翻译文件**：如果只是把 `.csv` 文件放入项目，Godot 4 会尝试将其当作翻译资源导入，可能导致无法正常读取或报错。
   - **解决方案**：将数据用的 CSV 文件放入带 `.gdignore` 文件的文件夹中，Godot 会忽略该文件夹的导入；或将数据文件后缀改为 `.txt` / `.json` 等其他文本格式。

2. **导出时的文件包含**：确保在导出项目时，CSV 数据文件被包含在导出资源中，否则游戏打包后可能找不到文件。

3. **分隔符问题**：如果单元格文本中包含逗号，建议在电子表格中将该列内容用双引号包裹，或使用 `|`、`;` 等非逗号分隔符保存 CSV。

---

## 八、社区插件推荐

| 插件 | 功能 |
|---|---|
| **Edit Resources as Table 2** | 在编辑器中以表格形式编辑 Resource，支持 CSV 导入/导出 |
| **Gnumaru's Static Data Importer** | 支持导入 CSV、JSON、YAML 等多种格式的静态数据 |
| **GDDataForge** | Godot 4.4 的灵活数据管理插件，支持 CSV、JSON 等格式 |

---

## 九、总结

**核心策略**：

1. **数据分离**：物品游戏属性（伤害、价格等）用一个 CSV 表，多语言文本（名称、介绍）用另一个翻译 CSV 表
2. **统一 KEY**：通过唯一的字符串标识符关联两个表
3. **读取方式**：
   - 物品属性数据 → `FileAccess` 手动解析
   - 多语言文本 → Godot 内置的 CSV 翻译导入器自动处理
4. **编码规范**：确保 CSV 以 UTF-8 编码保存
5. **工作流**：策划在电子表格中编辑数据 → 导出为 CSV → Godot 加载并解析
