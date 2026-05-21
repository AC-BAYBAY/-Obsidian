# `clamp()`钳制到最大值和最小值之间

`Variant clamp(value: Variant, min: Variant, max: Variant)`

钳制 **value**，返回不小于 **min** 且不大于 **max** 的 Variant。任何能用 `<` 和 `>` 运算符进行比较的值都能工作。

```gdscript
var a = clamp(-10, -1, 5)   # a 是 -1
var b = clamp(8.1, 0.9, 5.5) # b 是 5.5
```

> [!note] 类型安全建议
> 为了更好的类型安全，优先使用类型特定的 clamp 方法：
>
> - `clampf()`
> - `clampi()`
> - `Vector2.clamp()` / `Vector2i.clamp()`
> - `Vector3.clamp()` / `Vector3i.clamp()`
> - `Vector4.clamp()` / `Vector4i.clamp()`
> - `Color.clamp()`（当前不受支持）

# 颜色
```gdscript
const PIPE_COLOR = Color(0.21, 0.67, 0.29)
VAR PIPE_COLOR = Color(0.21, 0.67, 0.29)
```
# 删除节点（场景）queue_free()
queue_free()