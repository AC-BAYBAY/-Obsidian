# Tween（补间动画）

继承：`RefCounted < Object`

通过脚本进行通用动画的轻量级对象，使用 Tweener。

---

## 描述

`Tween` 主要用于需要将一个数值属性插值到一系列值的动画。"tween" 来自 in-betweening，即指定关键帧后由计算机插入中间帧的动画技术。使用 `Tween` 制作动画被称为**补间动画**。

`Tween` 比 `AnimationPlayer` 更适合**事先不知道最终值**的动画。例如，插值动态选择的相机缩放值用 `Tween` 更合适；`Tween` 也更轻量级，适合简单动画或通用任务。对于由代码完成的某些逻辑，可以"即用即弃"地使用，例如用带延迟的循环 `CallbackTweener` 定期射击。

**创建方式：** 使用 `SceneTree.create_tween()` 或 `Node.create_tween()`。手动 `Tween.new()` 创建的 Tween 无效。

**注意：**
- Tween 不是为重用设计的，每次动画请新建 Tween
- Tween 会立即开始，只在需要时创建
- 当前帧中所有节点处理完后才处理 Tween（即 `_process()` 或 `_physics_process()` 在补间之前调用）

---

## 基本用法

通过 `tween_property()`、`tween_interval()`、`tween_callback()` 或 `tween_method()` 将 Tweener 添加到 Tween：

```gdscript
var tween = get_tree().create_tween()
tween.tween_property($Sprite, "modulate", Color.RED, 1.0)
tween.tween_property($Sprite, "scale", Vector2(), 1.0)
tween.tween_callback($Sprite.queue_free)
```

该序列将 `$Sprite` 变红 → 缩小 → 释放。默认 Tweener **逐个执行**，可用 `parallel()` 和 `set_parallel()` 改变。

### 链式调整 Tweener

```gdscript
var tween = get_tree().create_tween()
tween.tween_property($Sprite, "modulate", Color.RED, 1.0).set_trans(Tween.TRANS_SINE)
tween.tween_property($Sprite, "scale", Vector2(), 1.0).set_trans(Tween.TRANS_BOUNCE)
tween.tween_callback($Sprite.queue_free)
```

### 链式设置 Tween 默认值

```gdscript
var tween = get_tree().create_tween().bind_node(self).set_trans(Tween.TRANS_ELASTIC)
tween.tween_property($Sprite, "modulate", Color.RED, 1.0)
tween.tween_property($Sprite, "scale", Vector2(), 1.0)
tween.tween_callback($Sprite.queue_free)
```

### 动画化一组对象

```gdscript
var tween = create_tween()
for sprite in get_children():
    tween.tween_property(sprite, "position", Vector2(0, 0), 1.0)
```

所有子节点依次移动到 `(0, 0)`。

### 中断并重启动画

```gdscript
var tween
func animate():
    if tween:
        tween.kill()
    tween = create_tween()
```

---

## 方法

| 返回值 | 方法 |
|--------|------|
| `Tween` | `bind_node(node: Node)` |
| `Tween` | `chain()` |
| `bool` | `custom_step(delta: float)` |
| `int` | `get_loops_left() const` |
| `float` | `get_total_elapsed_time() const` |
| `Variant` | `interpolate_value(initial_value, delta_value, elapsed_time, duration, trans_type, ease_type)` static |
| `bool` | `is_running()` |
| `bool` | `is_valid()` |
| `void` | `kill()` |
| `Tween` | `parallel()` |
| `void` | `pause()` |
| `void` | `play()` |
| `Tween` | `set_ease(ease: EaseType)` |
| `Tween` | `set_ignore_time_scale(ignore: bool = true)` |
| `Tween` | `set_loops(loops: int = 0)` |
| `Tween` | `set_parallel(parallel: bool = true)` |
| `Tween` | `set_pause_mode(mode: TweenPauseMode)` |
| `Tween` | `set_process_mode(mode: TweenProcessMode)` |
| `Tween` | `set_speed_scale(speed: float)` |
| `Tween` | `set_trans(trans: TransitionType)` |
| `void` | `stop()` |
| `CallbackTweener` | `tween_callback(callback: Callable)` |
| `IntervalTweener` | `tween_interval(time: float)` |
| `MethodTweener` | `tween_method(method: Callable, from, to, duration: float)` |
| `PropertyTweener` | `tween_property(object: Object, property: NodePath, final_val, duration: float)` |
| `SubtweenTweener` | `tween_subtween(subtween: Tween)` |

---

## 信号

- `finished()` — 所有补间完成时发出（无限循环时不会发出）
- `loop_finished(loop_count: int)` — 完成一次循环时触发（不触发于最后一次循环，改用 `finished`）
- `step_finished(idx: int)` — 完成一步后触发（一步 = 单个 Tweener 或一组并行的 Tweener）

---

## 枚举

### TweenProcessMode

| 常量 | 值 | 描述 |
|------|----|------|
| `TWEEN_PROCESS_PHYSICS` | 0 | 每物理帧更新（`_physics_process()`） |
| `TWEEN_PROCESS_IDLE` | 1 | 每处理帧更新（`_process()`），默认 |

### TweenPauseMode

| 常量 | 值 | 描述 |
|------|----|------|
| `TWEEN_PAUSE_BOUND` | 0 | 绑定节点可处理时处理，否则同 `STOP`，默认 |
| `TWEEN_PAUSE_STOP` | 1 | `SceneTree` 暂停时 Tween 暂停 |
| `TWEEN_PAUSE_PROCESS` | 2 | 无视 `SceneTree` 暂停始终处理 |

### TransitionType（过渡类型）

| 常量 | 值 | 描述 |
|------|----|------|
| `TRANS_LINEAR` | 0 | 线性插值，默认 |
| `TRANS_SINE` | 1 | 正弦插值 |
| `TRANS_QUINT` | 2 | 五次方插值 |
| `TRANS_QUART` | 3 | 四次方插值 |
| `TRANS_QUAD` | 4 | 二次方插值 |
| `TRANS_EXPO` | 5 | 指数插值 |
| `TRANS_ELASTIC` | 6 | 弹性插值，边缘摆动 |
| `TRANS_CUBIC` | 7 | 三次方插值 |
| `TRANS_CIRC` | 8 | 平方根插值 |
| `TRANS_BOUNCE` | 9 | 末端弹跳插值 |
| `TRANS_BACK` | 10 | 末端回放插值 |
| `TRANS_SPRING` | 11 | 弹簧插值 |

### EaseType（缓动类型）

| 常量 | 值 | 描述 |
|------|----|------|
| `EASE_IN` | 0 | 开始慢，结尾加速 |
| `EASE_OUT` | 1 | 开始快，结尾减速 |
| `EASE_IN_OUT` | 2 | 两端最慢，默认 |
| `EASE_OUT_IN` | 3 | 两端最快 |

---

## 方法说明

### `bind_node(node: Node) → Tween`

将 Tween 绑定到指定节点。绑定后：
- 节点不在树中时 Tween 暂停
- 节点释放时 Tween 自动销毁
- `TWEEN_PAUSE_BOUND` 暂停行为依赖绑定节点

> 使用 `Node.create_tween()` 创建可同时完成绑定。

---

### `chain() → Tween`

在 `set_parallel(true)` 后，用于将两个 Tweener 串联：

```gdscript
var tween = create_tween().set_parallel(true)
tween.tween_property(...)
tween.tween_property(...)       # 与上一条并行
tween.chain().tween_property(...)  # 前两条完成后执行
```

---

### `custom_step(delta: float) → bool`

手动控制 Tween 进度，`delta` 单位为秒。将 `delta` 设大于总时长可立即结束。返回 `true` 表示仍有未完成的 Tweener。

---

### `get_loops_left() → int`

返回剩余循环数。`-1` = 无限循环，`0` = 已结束。

---

### `get_total_elapsed_time() → float`

返回自开始以来经过的**实际时间**（不含暂停），受 `set_speed_scale()` 影响。`stop()` 会重置为 0。

> 因为是帧增量累计，完成后的返回值会略大于实际时长。

---

### `interpolate_value(initial_value, delta_value, elapsed_time, duration, trans_type, ease_type) → Variant` (static)

手动插值，类似 `@GlobalScope.lerp()` 但支持自定义过渡/缓动。

- `initial_value` — 起始值
- `delta_value` — 变化量（`final - initial`）
- `elapsed_time` — 经过秒数（可大于 `duration` 进行外插）
- `duration` — 总时长（为 0 时始终返回最终值）

---

### `is_running() → bool`

Tween 正在执行中（未暂停且未完成）。

---

### `is_valid() → bool`

Tween 是否有效（被场景树包含）。无效情况：补间完成、被销毁、用 `Tween.new()` 创建。无效的 Tween 不能追加 Tweener。

---

### `kill()`

中止所有补间操作并使 Tween 失效。

---

### `parallel() → Tween`

让下一个 Tweener 与上一个**并行**执行：

```gdscript
var tween = create_tween()
tween.tween_property(...)
tween.parallel().tween_property(...)
tween.parallel().tween_property(...)
```

所有 Tweener 同时执行。可用 `set_parallel()` 设为默认行为。

---

### `pause()` / `play()`

`pause()` 暂停补间，`play()` 恢复。

> 暂停且未绑定节点的 Tween 会无限存在，直到手动启动或失效。可通过 `SceneTree.get_processed_tweens()` 找回。

---

### `set_ease(ease: EaseType) → Tween`

设置之后追加的 `PropertyTweener` 和 `MethodTweener` 的默认缓动类型。调用前默认为 `EASE_IN_OUT`。

```gdscript
var tween = create_tween()
tween.tween_property(self, "position", Vector2(300, 0), 0.5)       # EASE_IN_OUT
tween.set_ease(Tween.EASE_IN)
tween.tween_property(self, "rotation_degrees", 45.0, 0.5)          # EASE_IN
```

---

### `set_ignore_time_scale(ignore: bool = true) → Tween`

为 `true` 时 Tween 忽略 `Engine.time_scale`，按实际时间更新。默认 `false`。

---

### `set_loops(loops: int = 0) → Tween`

设置循环次数。`set_loops(2)` 执行两次。不传参 = 无限循环。

> **警告：** 无限循环时务必加入时长/延迟。0 时长循环动画（如单个无延迟的 `CallbackTweener`）会在若干次后停止。若 Tween 生命周期依赖节点，务必使用 `bind_node()`。

---

### `set_parallel(parallel: bool = true) → Tween`

设为 `true` 后追加的 Tweener 默认并行运行，否则默认串行。

> 与 `parallel()` 类似，调用前的那一个 Tweener 也属于并行步骤。

```gdscript
tween.tween_property(self, "position", Vector2(300, 0), 0.5)
tween.set_parallel()
tween.tween_property(self, "modulate", Color.GREEN, 0.5)  # 与位置补间一同运行
```

---

### `set_pause_mode(mode: TweenPauseMode) → Tween`

设置 `SceneTree` 暂停时的行为。默认 `TWEEN_PAUSE_BOUND`。

---

### `set_process_mode(mode: TweenProcessMode) → Tween`

设置在 `_process()`（处理帧）还是 `_physics_process()`（物理帧）执行。默认 `TWEEN_PROCESS_IDLE`。

---

### `set_speed_scale(speed: float) → Tween`

补间速度缩放，影响所有 Tweener 及其延迟。

---

### `set_trans(trans: TransitionType) → Tween`

设置之后追加的 `PropertyTweener` 和 `MethodTweener` 的默认过渡类型。调用前默认为 `TRANS_LINEAR`。

```gdscript
var tween = create_tween()
tween.tween_property(self, "position", Vector2(300, 0), 0.5)    # TRANS_LINEAR
tween.set_trans(Tween.TRANS_SINE)
tween.tween_property(self, "rotation_degrees", 45.0, 0.5)       # TRANS_SINE
```

---

### `stop()`

停止补间并重置为初始状态，**不**移除已追加的 Tweener。

> 不会将属性重置为初始值。例如位置从 0 → 500 补间到一半时停止，位置停在 ~250，再 `play()` 会从 250 → 500 但速度减半。

---

### `tween_callback(callback: Callable) → CallbackTweener`

创建并追加 `CallbackTweener`，用于调用任意方法。用 `Callable.bind()` 绑定额外参数。

每隔 1 秒射击一次：
```gdscript
var tween = get_tree().create_tween().set_loops()
tween.tween_callback(shoot).set_delay(1.0)
```

2 秒延迟后变红再变蓝：
```gdscript
var tween = get_tree().create_tween()
tween.tween_callback($Sprite.set_modulate.bind(Color.RED)).set_delay(2)
tween.tween_callback($Sprite.set_modulate.bind(Color.BLUE)).set_delay(2)
```

---

### `tween_interval(time: float) → IntervalTweener`

创建并追加 `IntervalTweener`，在补间中创建延迟。`time` 为秒数。

创建执行间隔：
```gdscript
# ... 一些代码
await create_tween().tween_interval(2).finished
# ... 更多代码
```

每隔几秒来回移动并跳跃：
```gdscript
var tween = create_tween().set_loops()
tween.tween_property($Sprite, "position:x", 200.0, 1.0).as_relative()
tween.tween_callback(jump)
tween.tween_interval(2)
tween.tween_property($Sprite, "position:x", -200.0, 1.0).as_relative()
tween.tween_callback(jump)
tween.tween_interval(2)
```

---

### `tween_method(method: Callable, from, to, duration: float) → MethodTweener`

创建并追加 `MethodTweener`，持续调用方法并传入补间值（从 `from` 到 `to` 插值，时长 `duration` 秒）。用 `Callable.bind()` 绑定额外参数。

让 3D 对象面向另一个点：
```gdscript
var tween = create_tween()
tween.tween_method(look_at.bind(Vector3.UP), Vector3(-1, 0, -1), Vector3(1, 0, -1), 1.0)
```

延迟后用中间方法设置 Label 文本：
```gdscript
func _ready():
    var tween = create_tween()
    tween.tween_method(set_label_text, 0, 10, 1).set_delay(1)

func set_label_text(value: int):
    $Label.text = "Counting " + str(value)
```

---

### `tween_property(object: Object, property: NodePath, final_val, duration: float) → PropertyTweener`

创建并追加 `PropertyTweener`。将 `object` 的 `property` 在初始值和 `final_val` 之间补间，持续 `duration` 秒。初始值为 Tweener 启动时属性的值。

```gdscript
var tween = create_tween()
tween.tween_property($Sprite, "position", Vector2(100, 200), 1.0)
tween.tween_property($Sprite, "position", Vector2(200, 300), 1.0)
```

精灵先到 (100, 200) 再到 (200, 300)。

> 属性名可在检查器中悬停查看。支持 `"属性:组件"` 格式（如 `position:x`）只修改特定组件。

不同过渡类型从同一位置移动两次：
```gdscript
var tween = create_tween()
tween.tween_property($Sprite, "position", Vector2.RIGHT * 300, 1.0).as_relative().set_trans(Tween.TRANS_SINE)
tween.tween_property($Sprite, "position", Vector2.RIGHT * 300, 1.0).as_relative().from_current().set_trans(Tween.TRANS_EXPO)
```

---

### `tween_subtween(subtween: Tween) → SubtweenTweener`

创建并追加 `SubtweenTweener`，将子 Tween 嵌套进当前 Tween，便于创建复杂动画序列：

```gdscript
# 子补间：旋转
var subtween = create_tween()
subtween.tween_property(self, "rotation_degrees", 45.0, 1.0)
subtween.tween_property(self, "rotation_degrees", 0.0, 1.0)

# 父补间：将子补间作为一步执行
var tween = create_tween()
tween.tween_property(self, "position:x", 500, 3.0)
tween.tween_subtween(subtween)
tween.tween_property(self, "position:x", 300, 2.0)
```

> 注意：`pause()`、`stop()`、`set_loops()` 等可能导致父 Tween 卡在子补间步骤。子 Tween 的 `set_pause_mode()` 和 `set_process_mode()` 会被父 Tween 覆盖。

---

## 使用建议

- **过渡与缓动组合：** 不知道选哪种时，先试 `EASE_IN_OUT` 配合不同 `TransitionType`，选效果最好的
- **避免冲突：** 不要对同一对象的同一属性使用多个 Tween，最后创建的优先
- **绑定节点：** 尽量用 `bind_node()` 或 `Node.create_tween()`，防止 Tween 变成"孤儿"
- **Tweener 速查：**
  - `tween_property` — 属性插值（最常用）
  - `tween_callback` — 在序列中调用方法
  - `tween_interval` — 创建延迟/间隔
  - `tween_method` — 自定义值驱动方法
  - `tween_subtween` — 嵌套子序列
