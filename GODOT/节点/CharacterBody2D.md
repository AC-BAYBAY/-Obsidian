
专门用于通过脚本移动的角色的 2D 物理物体。

## 描述

CharacterBody2D 是针对用户控制的物理体的特化类。它们不会受到物理的影响，但会影响路径上的其他物理体。除了由 `PhysicsBody2D.move_and_collide()` 提供的常见的碰撞检测之外，它们主要用于提供移动对象的高阶 API，能够检测墙壁和斜坡（`move_and_slide()` 方法）。因此适用于需要高度可配置的物理体，因为通常是用户控制的角色，所以必须按照特定的方式移动、与世界发生碰撞。

## 枚举

### enum MotionMode

| 值 | 名称 | 说明 |
|---|---|---|
| `0` | `MOTION_MODE_GROUNDED` | 请在墙壁、天花板、地板等概念有意义时应用。在该模式下，物体运动会对斜坡作出反应（加减速）。该模式适合平台跳跃等侧视角游戏。 |
| `1` | `MOTION_MODE_FLOATING` | 请在没有地板和天花板等概念时应用。所有碰撞都会作为 on_wall（撞墙）汇报。在该模式下，滑动时的速度恒定。该模式适合俯视角游戏。 |

### enum PlatformOnLeave

| 值 | 名称 | 说明 |
|---|---|---|
| `0` | `PLATFORM_ON_LEAVE_ADD_VELOCITY` | 离开移动平台时，将最后的平台速度添加到 velocity 中。 |
| `1` | `PLATFORM_ON_LEAVE_ADD_UPWARD_VELOCITY` | 离开移动平台时，将最后的平台速度添加到 velocity 中，但是忽略向下的运动。如果想要在平台向下移动时保持完整的跳跃高度，就非常有用。 |
| `2` | `PLATFORM_ON_LEAVE_DO_NOTHING` | 离开平台时什么也不做。 |

## 属性

### floor_block_on_wall

- **类型：** `bool`
- **默认值：** `true`
- **Setter：** `set_floor_block_on_wall_enabled(value)`
- **Getter：** `is_floor_block_on_wall_enabled()`

如果为 `true`，则该物体将只能在地板上移动。此选项能够避免在墙壁上行走，但允许沿墙壁向下滑动。

### floor_constant_speed

- **类型：** `bool`
- **默认值：** `false`
- **Setter：** `set_floor_constant_speed_enabled(value)`
- **Getter：** `is_floor_constant_speed_enabled()`

如果为 `false`（默认），则该物体在下坡时会移动得更快，在上坡时会移动得更慢。

如果为 `true`，则无论坡度如何，该物体在地面上都会以相同的速度移动。请注意，你需要使用 `floor_snap_length` 以恒定速度粘着至向下的斜坡。

### floor_max_angle

- **类型：** `float`
- **默认值：** `0.7853982`
- **Setter：** `set_floor_max_angle(value)`
- **Getter：** `get_floor_max_angle()`

调用 `move_and_slide()` 时，斜坡仍被视为地板（或天花板）而不是墙壁的最大角度（单位为弧度）。默认值等于 45 度。

### floor_snap_length

- **类型：** `float`
- **默认值：** `1.0`
- **Setter：** `set_floor_snap_length(value)`
- **Getter：** `get_floor_snap_length()`

设置吸附距离。设为非 `0.0` 值时，该物体在调用 `move_and_slide()` 时会保持附着到斜坡上。吸附向量会根据给定的距离和 `up_direction` 反方向决定。

只要吸附向量与地面有接触，该物体就会逆 `up_direction` 移动，保持附着到表面。如果该物体是沿着 `up_direction` 移动的，则不会应用吸附，这样跳跃时或者被其他物体推动时就能够不再附着地面。如果想要在应用吸附时无视速度，请使用 `apply_floor_snap()`。

### floor_stop_on_slope

- **类型：** `bool`
- **默认值：** `true`
- **Setter：** `set_floor_stop_on_slope_enabled(value)`
- **Getter：** `is_floor_stop_on_slope_enabled()`

如果为 `true`，则该物体静止时，调用 `move_and_slide()` 不会让它在斜坡上发生滑动。

如果为 `false`，则 `velocity` 施加向下的力时，该物体会在地板的斜坡上发生滑动。

### max_slides

- **类型：** `int`
- **默认值：** `4`
- **Setter：** `set_max_slides(value)`
- **Getter：** `get_max_slides()`

调用 `move_and_slide()` 时，物体在停止之前可以改变方向的最大次数。必须大于零。

## 方法

### is_on_ceiling()

- **返回：** `bool`

如果最近一次调用 `move_and_slide()` 时，该物体和天花板发生了碰撞，则返回 `true`。否则返回 `false`。决定表面是否为"天花板"的是 `up_direction` 和 `floor_max_angle`。

### is_on_ceiling_only()

- **返回：** `bool`

如果最近一次调用 `move_and_slide()` 时，该物体仅和天花板发生了碰撞，则返回 `true`。否则返回 `false`。决定表面是否为"天花板"的是 `up_direction` 和 `floor_max_angle`。

### is_on_floor()

- **返回：** `bool`

如果最近一次调用 `move_and_slide()` 时，该物体和地板发生了碰撞，则返回 `true`。否则返回 `false`。决定表面是否为"地板"的是 `up_direction` 和 `floor_max_angle`。

### is_on_floor_only()

- **返回：** `bool`

如果最近一次调用 `move_and_slide()` 时，该物体仅和地板发生了碰撞，则返回 `true`。否则返回 `false`。决定表面是否为"地板"的是 `up_direction` 和 `floor_max_angle`。

### is_on_wall()

- **返回：** `bool`

如果最近一次调用 `move_and_slide()` 时，该物体和墙壁发生了碰撞，则返回 `true`。否则返回 `false`。决定表面是否为"墙壁"的是 `up_direction` 和 `floor_max_angle`。

### is_on_wall_only()

- **返回：** `bool`

如果最近一次调用 `move_and_slide()` 时，该物体仅和墙壁发生了碰撞，则返回 `true`。否则返回 `false`。决定表面是否为"墙壁"的是 `up_direction` 和 `floor_max_angle`。

### move_and_slide()

- **返回：** `bool`

### get_slide_collision_count()

- **返回：** `int`

返回最近一次调用 `move_and_slide()` 时，该物体发生碰撞并改变方向的次数。

### get_collider()

- **返回：** `Object`

返回该碰撞实体所附加的 Object。
