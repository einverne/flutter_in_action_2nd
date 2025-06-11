# 4.4 弹性布局（Flex）

弹性布局允许子组件按照一定比例来分配父容器空间。弹性布局的概念在其他UI系统中也都存在，如 H5 中的弹性盒子布局，Android中 的`FlexboxLayout`等。Flutter 中的弹性布局主要通过`Flex`和`Expanded`来配合实现。

## 4.4.1 Flex

`Flex`组件可以沿着水平或垂直方向排列子组件，如果你知道主轴方向，使用`Row`或`Column`会方便一些，**因为`Row`和`Column`都继承自`Flex`**，参数基本相同，所以能使用Flex的地方基本上都可以使用`Row`或`Column`。`Flex`本身功能是很强大的，它也可以和`Expanded`组件配合实现弹性布局。

Flex 组件本身功能强大，它通过以下核心属性来控制布局：

- `direction`: 这是一个必需的属性，用于决定主轴的方向。它有两个值：Axis.horizontal（水平排列）和 Axis.vertical（垂直排列）。
- children: 一个 Widget 列表，包含所有需要在此 Flex 容器中排列的子组件。
- `mainAxisAlignment`: 控制子组件在主轴（direction 指定的方向）上的对齐方式，例如 MainAxisAlignment.spaceEvenly 可以使子组件在主轴上均匀分布空间。
- `crossAxisAlignment`: 控制子组件在交叉轴（与主轴垂直的方向）上的对齐方式，例如 CrossAxisAlignment.stretch 可以使子组件在交叉轴方向上拉伸以填满可用空间。


接下来我们只讨论`Flex`和弹性布局相关的属性(其他属性已经在介绍`Row`和`Column`时介绍过了)。

```dart
Flex({
  ...
  required this.direction, //弹性布局的方向, Row默认为水平方向，Column默认为垂直方向
  List<Widget> children = const <Widget>[],
})
```

`Flex`继承自`MultiChildRenderObjectWidget`，对应的`RenderObject`为`RenderFlex`，`RenderFlex`中实现了其布局算法。

## 4.4.2 Expanded
Expanded 组件会强制其子组件填充 Flex 布局在主轴方向上的所有剩余可用空间。通过 flex 属性（整数），可以指定多个 Expanded 组件之间分配空间的比例。flex 的默认值为 1。例如，在一个 Row 中放置三个 Expanded 组件，其 flex 值分别为 1、2 和 1，那么它们将按照 1:2:1 的比例分配水平方向的可用空间。

Expanded 实际上是 `Flexible` 组件的一个特例，它的 fit 属性被固定为 FlexFit.tight，意味着它会强制子组件填满所有分配到的空间。

Expanded 只能作为 Flex 的孩子（否则会报错），它可以按比例“扩伸”`Flex`子组件所占用的空间。因为  `Row`和`Column` 都继承自 Flex，所以 Expanded 也可以作为它们的孩子。


```dart
const Expanded({
  int flex = 1, 
  required Widget child,
})
```

`flex`参数为弹性系数，如果为 0 或`null`，则`child`是没有弹性的，即不会被扩伸占用的空间。如果大于0，所有的`Expanded`按照其 flex 的比例来分割主轴的全部空闲空间。下面我们看一个例子：

```dart
class FlexLayoutTestRoute extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        //Flex的两个子widget按1：2来占据水平空间  
        Flex(
          direction: Axis.horizontal,
          children: <Widget>[
            Expanded(
              flex: 1,
              child: Container(
                height: 30.0,
                color: Colors.red,
              ),
            ),
            Expanded(
              flex: 2,
              child: Container(
                height: 30.0,
                color: Colors.green,
              ),
            ),
          ],
        ),
        Padding(
          padding: const EdgeInsets.only(top: 20.0),
          child: SizedBox(
            height: 100.0,
            //Flex的三个子widget，在垂直方向按2：1：1来占用100像素的空间  
            child: Flex(
              direction: Axis.vertical,
              children: <Widget>[
                Expanded(
                  flex: 2,
                  child: Container(
                    height: 30.0,
                    color: Colors.red,
                  ),
                ),
                Spacer(
                  flex: 1,
                ),
                Expanded(
                  flex: 1,
                  child: Container(
                    height: 30.0,
                    color: Colors.green,
                  ),
                ),
              ],
            ),
          ),
        ),
      ],
    );
  }
}
```

运行效果如图4-13所示：

![图4-13](../imgs/4-13.png)

示例中的`Spacer`的功能是占用指定比例的空间，实际上它只是`Expanded`的一个包装类，`Spacer`的源码如下：

```dart
class Spacer extends StatelessWidget {
  const Spacer({Key? key, this.flex = 1})
    : assert(flex != null),
      assert(flex > 0),
      super(key: key);
  
  final int flex;

  @override
  Widget build(BuildContext context) {
    return Expanded(
      flex: flex,
      child: const SizedBox.shrink(),
    );
  }
}
```

## Flexable 

`Flexible`组件和`Expanded`类似，但它不会强制子组件填充 Flex 布局在主轴方向上的所有剩余可用空间，而是允许子组件根据自己的需要来决定大小。`Flexible`的`flex`属性也可以指定多个 Flexible 组件之间分配空间的比例。

- flex: 一个整数，用于按比例划分空间，数值越大，获得的空闲空间比例就越高
- fit: 决定子组件填充其所分配空间的方式
    - FlexFit.tight: 效果与 Expanded 相同，强制子组件填满分配到的空间。
    - FlexFit.loose: 默认值，允许子组件在分配到的空间内根据自身大小进行布局，只要不超过分配的空间即可。

`Expanded` 因为使用频率高，被单独封装成一个类，其语义也更明确。

## Spacer
Spacer 是一个用于在 Flex 布局的子组件之间创建可调整间距的便捷组件。它本质上是一个包装了 Expanded 的 SizedBox.shrink()（一个不占空间的空盒子）。通过设置 flex 属性，可以控制 Spacer 所占用的空间比例，从而灵活地撑开不同组件之间的空隙。

## Flex 布局算法

Flex 布局的算法大致遵循以下步骤：

- 布局非弹性子组件: 首先，为 flex 值为 0 或 null 的子组件（非弹性组件）分配空间。在主轴方向上，这些子组件的约束是无限的，因此它们可以根据自身 需要占用空间。
- 分配剩余空间: 接着，将剩余的空间根据弹性子组件（即被 Flexible 或 Expanded 包裹的组件）的 flex 因子按比例进行分配。
- 布局弹性子组件: 为弹性子组件分配其在上一步中计算得到的空间。子组件最终的尺寸取决于其 fit 属性是 tight 还是 loose。
- 最终定位: 确认所有子组件的尺寸后，根据 Flex 的 mainAxisAlignment 和 crossAxisAlignment 属性将它们在父容器中进行对齐和定位。

## 4.4.3 小结

弹性布局比较简单，唯一需要注意的就是`Row`、`Column`以及`Flex`的关系。

Flex 布局非常适用于子组件数量或尺寸不确定、需要响应式设计以及需要自定义复杂布局的场景。通过 Flex、Expanded、Flexible 和 Spacer 等组件的组合，开发者可以构建出高度灵活且适应性强的用户界面。
