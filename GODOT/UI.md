# Control
所有 GUI 控件的基类。根据其父控件调整其位置和大小。
# CanvasLayer、画布层
它是一个节点，可以为所有子代和孙代添加一个单独的 2D 渲染层。而 CanvasLayer 将在任何数字层处绘制。数字较大的图层将绘制在数字较小的图层之上。
一个例子是创建视差背景（Parallax Background）。这可以通过层为“-1”的 CanvasLayer 完成。带有分数、生命计数器和暂停按钮的屏幕也可以创建在编号为“1”的层中。
# PanelContainer、背景
保证子控件在 StyleBox 区域内的容器。可用来为控件提供轮廓。（自动调整大小）
# BoxContainer、容器
将子控件横向或纵向排列的容器（H是横向，V是纵向）
`AlignmentModea lignment [默认： 0]`
`set_alignment(值)  setter`
`get_alignment()  getter`
该容器子节点的对齐方式（必须是 ALIGNMENT_BEGIN、ALIGNMENT_CENTER、ALIGNMENT_END 之一）。开头，中间，尾对齐。
# ScrollBar、滚动条
# MarginContainer、在子控件周围保留边距的容器。
放在PanelContainer里面，然后里面再放控件

# 主题
创建一个新theme
右下角+号选择更改某一种节点
## 自定义节点
右下角+号手动输入一个新名字，随后点击全部覆盖下方一行最右侧的选项，选择基础类型
# 11

