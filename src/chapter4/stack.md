# 4.6 层叠布局（Stack、Positioned）
在 Flutter 中，层叠布局（Stack Layout）允许子组件相互堆叠，可以实现类似于 Web 开发中的绝对定位或 Android 中的 FrameLayout 布局的效果。这种布局方式通过 Stack 和 Positioned 两个核心组件协同工作，让开发者能够精确控制子组件在父容器中的位置和层叠顺序。

层叠布局中子组件可以根据距父容器四个角的位置来确定自身的位置。层叠布局允许子组件按照代码中声明的顺序堆叠起来。Flutter中使用`Stack`和`Positioned`这两个组件来配合实现绝对定位。`Stack`允许子组件堆叠，而`Positioned`用于根据`Stack`的四个角来确定子组件的位置。

## 4.6.1 Stack
Stack 是实现层叠布局的基础容器，它允许其子组件按照声明的顺序堆叠在一起。后声明的子组件会覆盖在先声明的子组件之上。Stack 的子组件分为两类：定位子组件（Positioned）和非定位子组件（Non-positioned）。

核心属性

- alignment: 该属性用于决定非定位或部分定位的子组件在 Stack 内的对齐方式。默认值为 AlignmentDirectional.topStart，即左上角对齐。部分定位指的是子组件仅在单个轴（水平或垂直）上被 定位。
- fit: 此参数用于确定非定位子组件如何适应 Stack 的大小。
- StackFit.loose (默认值): 允许子组件根据自身内容决定大小。
- StackFit.expand: 强制非定位的子组件扩展至与 Stack 同等大小。
- clipBehavior (在旧版中为 overflow): 决定如何处理超出 Stack 边界的子组件。例如，Clip.hardEdge 会直接裁切掉溢出的部分。
- textDirection: 用于确定 alignment 属性的参考系。例如，当值为 TextDirection.ltr 时，start 代表左边，end 代表右边。

Stack组件定义如下：

```dart
Stack({
  this.alignment = AlignmentDirectional.topStart,
  this.textDirection,
  this.fit = StackFit.loose,
  this.clipBehavior = Clip.hardEdge,
  List<Widget> children = const <Widget>[],
})
```

- `alignment`：此参数决定如何去对齐没有定位（没有使用`Positioned`）或部分定位的子组件。所谓部分定位，在这里**特指没有在某一个轴上定位：**`left`、`right`为横轴，`top`、`bottom`为纵轴，只要包含某个轴上的一个定位属性就算在该轴上有定位。
- `textDirection`：和`Row`、`Wrap`的`textDirection`功能一样，都用于确定`alignment`对齐的参考系，即：`textDirection`的值为`TextDirection.ltr`，则`alignment`的`start`代表左，`end`代表右，即`从左往右`的顺序；`textDirection`的值为`TextDirection.rtl`，则alignment的`start`代表右，`end`代表左，即`从右往左`的顺序。
- `fit`：此参数用于确定**没有定位**的子组件如何去适应`Stack`的大小。`StackFit.loose`表示使用子组件的大小，`StackFit.expand`表示扩伸到`Stack`的大小。
- `clipBehavior`：此属性决定对超出`Stack`显示空间的部分如何剪裁，Clip枚举类中定义了剪裁的方式，Clip.hardEdge 表示直接剪裁，不应用抗锯齿，更多信息可以查看源码注释。

## 4.6.2 Positioned
Positioned 组件专门用于在 Stack 布局中对其子组件进行精确定位。需要注意的是，Positioned 必须作为 Stack 的直接子组件使用。

- left, top, right, bottom: 分别定义子组件的边缘距离 Stack 容器四个边的距离。
- width, height: 用于直接指定定位子组件的宽度和高度。

一个重要的约束是，在同一坐标轴上，最多只能同时指定两个约束属性。例如，在水平方向上，left、right 和 width 三个属性中只能提供任意两个，第三个属性将由系统自动计算得出。如果同时指定三个，则会引发错误。

Positioned 的默认构造函数如下：

```dart
const Positioned({
  Key? key,
  this.left, 
  this.top,
  this.right,
  this.bottom,
  this.width,
  this.height,
  required Widget child,
})
```

`left`、`top` 、`right`、 `bottom`分别代表离`Stack`左、上、右、底四边的距离。`width`和`height`用于指定需要定位元素的宽度和高度。注意，`Positioned`的`width`、`height` 和其他地方的意义稍微有点区别，此处用于配合`left`、`top` 、`right`、 `bottom`来定位组件，举个例子，在水平方向时，你只能指定`left`、`right`、`width`三个属性中的两个，如指定`left`和`width`后，`right`会自动算出(`left`+`width`)，如果同时指定三个属性则会报错，垂直方向同理。

## 4.6.3 示例

在下面的例子中，我们通过对几个`Text`组件的定位来演示`Stack`和`Positioned`的特性：

```dart
//通过ConstrainedBox来确保Stack占满屏幕
ConstrainedBox(
  constraints: BoxConstraints.expand(),
  child: Stack(
    alignment:Alignment.center , //指定未定位或部分定位widget的对齐方式
    children: <Widget>[
      Container(
        child: Text("Hello world",style: TextStyle(color: Colors.white)),
        color: Colors.red,
      ),
      Positioned(
        left: 18.0,
        child: Text("I am Jack"),
      ),
      Positioned(
        top: 18.0,
        child: Text("Your friend"),
      )        
    ],
  ),
);
```

运行效果见图4-17：

![图4-17](../imgs/4-17.png)

由于第一个子文本组件`Text("Hello world")`没有指定定位，并且`alignment`值为`Alignment.center`，所以它会居中显示。第二个子文本组件`Text("I am Jack")`只指定了水平方向的定位(`left`)，所以属于部分定位，即垂直方向上没有定位，那么它在垂直方向的对齐方式则会按照`alignment`指定的对齐方式对齐，即垂直方向居中。对于第三个子文本组件`Text("Your friend")`，和第二个`Text`原理一样，只不过是水平方向没有定位，则水平方向居中。

我们给上例中的`Stack`指定一个`fit`属性，然后将三个子文本组件的顺序调整一下：

```dart
Stack(
  alignment:Alignment.center ,
  fit: StackFit.expand, //未定位widget占满Stack整个空间
  children: <Widget>[
    Positioned(
      left: 18.0,
      child: Text("I am Jack"),
    ),
    Container(child: Text("Hello world",style: TextStyle(color: Colors.white)),
      color: Colors.red,
    ),
    Positioned(
      top: 18.0,
      child: Text("Your friend"),
    )
  ],
),
```

显示效果如图4-18所示：

![图4-18](../imgs/4-18.png)

可以看到，由于第二个子文本组件没有定位，所以`fit`属性会对它起作用，就会占满`Stack`。由于`Stack`子元素是堆叠的，所以第一个子文本组件被第二个遮住了，而第三个在最上层，所以可以正常显示。

## 工作原理

Stack 和 Positioned 的结合使用，提供了非常灵活的布局能力：

- 堆叠顺序: Stack 的 children 列表中的组件顺序决定了它们的堆叠层次。列表中的最后一个组件会显示在最顶层。
- 定位机制:
    - 被 Positioned 包裹的子组件会根据其 top、left 等属性相对于 Stack 的边界进行定位。
    - 没有被 Positioned 包裹的非定位子组件，则会根据 Stack 的 alignment 属性进行对齐。
- 尺寸确定: Stack 的最终尺寸默认由其所有非定位子组件中尺寸最大的那个来决定。如果设置了 fit: StackFit.expand，则 Stack 会尽可能地扩展以填充其父容器。

常见应用场景:

- 在图片上叠加文字或按钮。
- 为图标添加角标（Badge）。
- 实现复杂的、有重叠效果的自定义用户界面。

