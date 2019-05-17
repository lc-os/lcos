# Flutter 红包动画实现

公司项目需要一个红包动画开启红包，发给我了一个微信开启红包的视频，抱着尝试的心里去写了一个简单的红包动画，以下是效果图：


![拆红包](https://github.com/LCOSGit/red_packet/blob/master/red_packet.gif)

##自定义的Dialog的方式封装一个红包Dialog

首先自定义一个Dialog继承系统的*Dialog*，然后在build方法里面创建一个透明层

```
return Material(
      ///创建透明层
      type: MaterialType.transparency,
      ///居中
      child: Center(
        child: RedPacketPage(),
      ),
    );
```
在RedPacketPage这个Widget中定义两个参数

```
bool showGif = false;
点击拆开时播放gif图片
bool dismiss = false;
开启红包消失动画

Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.transparent,
      body: _bodyWidget(),
    );
  }


  Widget _bodyWidget() {
    if (dismiss) {
      return DismissWidget(bags);
    } else {
      return _normalWidget();
    }
  }
```

点击开红包按钮后触发 *setState(() {
      showGif = true;
    });* 并在该方法中去请求接口获取红包额度，请求结束后*showGif = false; dismiss = true;* 红包消失动画启动

```
        showGif
                ? Container(
                    margin: EdgeInsets.only(top: ScreenUtil().setWidth(740)),
                    child: Center(
                      child: Center(
                        child: Image.asset(
                          "assets/open_red_packet.gif",
                          width: 100,
                          scale: 1,
                        ),
                      ),
                    ),
                  )
                : Container(
                    margin: EdgeInsets.only(top: ScreenUtil().setWidth(740)),
                    child: Center(
                      child: IconButton(
                        icon: Image.asset(
                          "assets/open_icon.png",
                          // width: 100,
                          // scale: 1,
                        ),
                        iconSize: 100,
                        onPressed: () {
                          _takeRedPacket();
                        },
                      ),
                    ),
                  ),
```

## 红包消失动画模块

### AnimationController

AnimationController用于控制动画，它包含动画的启动forward()、停止stop() 、反向播放 reverse()等方法。AnimationController会在动画的每一帧，就会生成一个新的值。默认情况下，AnimationController在给定的时间段内线性的生成从0.0到1.0（默认区间）的数字。 

### Animation
Animation对象本身和UI渲染没有任何关系。Animation是一个抽象类，它用于保存动画的插值和状态；其中一个比较常用的Animation类是Animation。Animation对象是一个在一段时间内依次生成一个区间(Tween)之间值的类。Animation对象的输出值可以是线性的、曲线的、一个步进函数或者任何其他曲线函数。 根据Animation对象的控制方式，动画可以反向运行，甚至可以在中间切换方向。Animation还可以生成除double之外的其他类型值，如：Animation\ 或 Animation\。可以通过Animation对象的value属性获取动画的当前值。

### Tween

默认情况下，AnimationController对象值的范围是0.0到1.0。如果我们需要不同的范围或不同的数据类型，则可以使用Tween来配置动画以生成不同的范围或数据类型的值。

### Tween.animate

要使用Tween对象，需要调用其animate()方法，然后传入一个控制器对象。例如，以下代码在500毫秒内生成从0到255的整数值。

* 创建一个*animationController*用于控制动画
* 创建两个Animation用于红包上下两部分，形成上下分离动画

```
AnimationController animationController;
Animation topParentAnimation;
Animation bottomParentAnimation;

@override
  void initState() {
    animationController =
        AnimationController(vsync: this, duration: Duration(milliseconds: 200))
          ..addListener(() {
          ///添加监听动画完成
            if (animationController.isCompleted) {
              Navigator.pop(context);
             ///动画结束，跳转详情界面
            }
          });
    topParentAnimation = Tween(begin: 0.0, end: -1.0).animate(
        CurvedAnimation(parent: animationController, curve: Curves.easeIn));
    bottomParentAnimation = Tween(begin: 0.0, end: 1.5).animate(
        CurvedAnimation(parent: animationController, curve: Curves.easeIn));
    animationController.forward();///动画启动
    super.initState();
  }
```

### AnimatedBuilder
AnimatedBuilder可以将渲染逻辑分离出来

```
 AnimatedBuilder(
            animation: animationController,
            builder: (BuildContext context, Widget child) {
              return Transform(
                  transform: Matrix4.translationValues(
                      0.0, topParentAnimation.value * height / 2, 0.0),
                  child: Center(
                    child: Image.asset(
                      "assets/red_packet_top.png",
                      width: bottomParentAnimation.value * width <
                              (width - ScreenUtil().setWidth(140))
                          ? (width - ScreenUtil().setWidth(140))
                          : bottomParentAnimation.value * width,
                      fit: BoxFit.cover,
                    ),
                  ));
            },
          ),
```

动画还没看完，直接整出来了，不是很难，项目地址奉上[红包动画](https://github.com/LCOSGit/red_packet)


