**[掘金文章地址](https://juejin.cn/post/6924866572972457992)**

## 前言

> 好久不见，我是Channing
>
> 因为最近事情比较多，很抱歉已经两个月没有写新文章了
>
> 这几天终于有机会也因为工作上的一些灵感于是肝下这篇文章分享给大家
>
> 希望能对你有一定的启发或帮助🙏



拥有了强大的**Canvas**，我们可以使用JavaScript来控制它从而轻易地绘制出各种各样想要的图形，还可以利用JavaScript将用户的交互与canvas的绘制紧密地连接起来，甚至大可发挥我们的脑洞去做各种各样天马星空的事情。



虽然canvas帮助我们轻松地绘制出了各种各样的图形，但canvas本身并没有**过渡效果**，每一次绘制出来就不会发生变动，那么如何让“静止”的canvas动起来呢？



首先，我们先了解**动画**本身的含义，参考维基百科：

> **动画**（英语：Animation）是一种通过定时拍摄一系列多个静止的固态[图像](https://zh.wikipedia.org/wiki/图像)（[帧](https://zh.wikipedia.org/wiki/帧)）以一定频率连续变化、运动（播放）的速度（如每秒16张）而导致肉眼的[视觉残象](https://zh.wikipedia.org/wiki/视觉残象)产生的[错觉](https://zh.wikipedia.org/wiki/錯覺)——而误以为图画或物体（画面）[活动](https://zh.wikipedia.org/wiki/飛現象)的作品及其视频技术。



如上所述，动画就是一张张静态的图片以一定的速度切换，其中每一张画面就是一帧，只要帧速（切换速度）足够快，这些静态的图像用肉眼看上去就像是在运动了。

那么这个足够快是多快呢？

> 人眼的**视觉残留**特性：是光对视网膜所产生的视觉在光停止作用后，仍保留一段时间的现象，原因是由视神经元的反应速度造成的。其时值是**二十四分之一秒**。



所以，要骗过我们的眼睛，理论上达到二十四分之一秒即**24帧**这个速度切换图片就能达到动画的效果，当然，这个速度越快，这个动画就越细腻流畅。



掌握了这个基本知识，那么实现canvas动画的思路也就变得非常清晰：把每一次canvas绘制的结果作为一帧，足够频繁地进行多次canvas绘制就能达到动画的效果。



这篇文章就以路径动画为例，看看如何绘制一些canvas图形的**路径动画**。





## How——如何控制动画帧

为了实现Canvas的动画效果，首先我们需要一种定时的方法，去周期性地执行Canvas绘制，也就是需要定时地去绘制我们动画的每一帧。



说到定时，首先我们会想到两种方法：**window.setTimeout** 和 **window.setInterval** 。

这两种方法当然可以实现我们需要的动画效果，但我并不打算使用它们，也因为有更合适的方法：**window.requestAnimationFrame。**



如果足够了解JavaScript，你会知道由于其单线程的关系，定时器的实现是在当前任务队列完成后再执行定时器的回调，也就是如果当前队列的执行时间大于定时器设置的时间，那么这个定时器的时间就不是那么靠谱了。



举个栗子：

任务队列执行时间较短时：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36d5ca377c6f4ca1a912b2bf04e6e3c5~tplv-k3u1fbpfcp-zoom-1.image)

队列执行时间较长时：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/481c6d593dcf47e68930ad73293b1b9b~tplv-k3u1fbpfcp-zoom-1.image)



由于定时器的时间只是我们自己设置的一个期望渲染时间，但这个时间点其实并非浏览器一个重绘的时间点，当这两个时间点出现偏差时，可能就会发生丢帧之类的现象。



这个细说起来比较长，有钻研精神的同学可以看看张鑫旭大佬写的这篇文章：[CSS3动画那么强，requestAnimationFrame还有毛线用？](https://www.zhangxinxu.com/wordpress/2013/09/css3-animation-requestanimationframe-tween-动画算法/?_t_t_t=0.3547051663712042)



**requestAnimationFrame**提供了更加平缓并且更加有效率的方式来执行动画，当系统准备好了重绘条件时才会调用我们动画帧的绘制方法。



换句话说，调用requestAnimationFrame方法，向浏览器发送请求我要执行动画帧的绘制，当浏览器要重绘的时候

会调用requestAnimationFrame中我们传入的回调函数，而在我们的回调函数中，只要需要继续绘制动画帧，就再调用requestAnimationFrame，同时把回调函数自身传进去，直到动画绘制完成。



requestAnimationFrame的使用可能听上去有点绕，但是不要紧，看完后面的这个例子应该就理解了。



选好了控制重绘的定时方法，接下来就开始制作我们的路径动画。

## 从一根直线开始



首先是一条直线的基本绘制方法：

```javascript
<body>
<div id="executeButton" onclick="handleExecute()">执行</div>
<canvas id="myCanvas" width="800" height="800"></canvas>
<script>
    function handleExecute() {
        // 获取canvas元素
        const canvas = document.querySelector('#myCanvas')
        // 获取canvas渲染上下文
        const ctx = canvas.getContext('2d')

        // 设置线条样式
        ctx.strokeStyle = 'rgba(81, 160, 255,1)'
        ctx.lineWidth = 3
        // 创建路径
        ctx.beginPath()
        // 移动笔触到(100,100)坐标处
        ctx.moveTo(100,100)
        // 把线连接到(700,700)这个位置
        ctx.lineTo(700,700)
        // 把刚刚的路径绘制出来
        ctx.stroke()
    }
</script>
</body>
```



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27e8ede322a54971b3558c54b3e9e58b~tplv-k3u1fbpfcp-zoom-1.image)

**codepen传送门**：https://codepen.io/channinghan/pen/oNYXrjV



这是一根完整的直线，那么如何做成一个路径动画？



有两种思路：

1. 使用用多条路径，把这根线想象成若干个线段组成，每一次有序地绘制其中的一段，每一次的绘制就是一个动画帧。
2. 使用同一条路径，每一个动画帧绘制的直线长度逐渐增加，直到达到我们期望的长度。



两种思路的实现方法大同小异，但也存在一些不可忽视的差别：第二种的绘制可能需要在每次重绘时清空上一帧的画布区域，否则不断覆盖会导致一些期望以外的结果，比如当你线条颜色带有透明度时，由于不断覆盖会导致线条颜色不断加深，无法达到我们期望中的透明度。



此外，如果清空画布会带来一定的影响，当你密集的绘制多个图形时，清空画布会影响到其他图形，比如其他图形会被擦除掉部分甚至全部。



为了更加复杂场景不必考虑清空画布带来的影响，后面的例子中我都同一选择使用第一种思路来进行路径动画的绘制。



但为了更直观的看到两种思路的实现或者差别，在这个例子中我先把两种思路都实现一次：



#### 思路一：使用多条路径

每一帧中使用**beginPath**方法创建新的路径（会清空内存中之前的路径，但放心不是清空绘制结果），路径上的起止点计算需要一些简单的数学几何知识，在每一帧最后执行**stroke**方法将路径绘制出来。



动画的进度用**progress**表示，由每一帧绘制执行时距离开始时的时间除以我们设置的动画持续时间**duration**得到。

progress为**1**时则表示已完成。当progress小于1时继续调用**requestAnimationFrame**来向浏览器请求我们绘制动画帧方法的执行。

```javascript
<script>
    function handleExecute() {
        // 获取canvas元素
        const canvas = document.querySelector('#myCanvas')
        // 获取canvas渲染上下文
        const ctx = canvas.getContext('2d')

        // 设置线条样式
        ctx.strokeStyle = 'rgba(81, 160, 255,1)'
        ctx.lineWidth = 4
        ctx.lineJoin = 'round'

        // 定义起点和终点的坐标
        const startX = 100
        const startY = 100
        const endX = 700
        const endY = 700
        let prevX = startX
        let prevY = startY
        let nextX
        let nextY
        // 第一帧执行的时间
        let startTime;
        // 期望动画持续的时间
        const duration = 1000

        /*
        * 动画帧绘制方法.
        * currentTime是requestAnimation执行回调方法step时会传入的一个执行时的时间(由performance.now()获得).
        * */
        const step = (currentTime) => {
            // 第一帧绘制时记录下开始的时间
            !startTime && (startTime = currentTime)
            // 已经过去的时间(ms)
            const timeElapsed = currentTime - startTime
            // 动画执行的进度 {0,1}
            const progress = Math.min(timeElapsed / duration, 1)

            // 绘制方法
            const draw = () => {
                // 创建新的路径
                ctx.beginPath()
                // 创建子路径,并将起点移动到上一帧绘制到达的坐标点
                ctx.moveTo(prevX, prevY)
                // 计算这一帧中线段应该到达的坐标点,并且将prevX/Y更新为此值给下一帧使用.
                prevX = nextX = startX + (endX - startX) * progress
                prevY = nextY = startY + (endY - startY) * progress
                // 用直线将刚刚moveTo中的点连接到(nextX,nextY)上
                ctx.lineTo(nextX, nextY)
                ctx.strokeStyle = `rgba(${81}, ${160}, ${255},${0.25})`
                // 把这一帧的路径绘制出来
                ctx.stroke()
            }
            draw()

            if (progress < 1) {
                requestAnimationFrame(step)
            } else {
                console.log('动画执行完毕')
            }
        }

        requestAnimationFrame(step)
    }
</script>
```

**codePen传送门**：https://codepen.io/channinghan/pen/wvovGNe

效果图：

![直线路径动画——思路一.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9dce2571d5da4747b164aecfa1178605~tplv-k3u1fbpfcp-zoom-1.image)



#### 第二种：使用同一条路径

```javascript
<script>
    function handleExecute() {
        // 获取canvas元素
        const canvas = document.querySelector('#myCanvas')
        // 获取canvas渲染上下文
        const ctx = canvas.getContext('2d')

        // 定义起点和终点的坐标
        const startX = 100
        const startY = 100
        const endX = 700
        const endY = 700
        let nextX
        let nextY

        // 第一帧执行的时间
        let startTime;
        // 期望动画持续的时间
        const duration = 1000

        // 创建路径
        ctx.beginPath()
        // 创建一条子路径,把新的子路径起始点移动到(prevX,prevY)坐标.
        ctx.moveTo(startX, startY)
        // 设置线条样式
        ctx.strokeStyle = `rgba(${81}, ${160}, ${255},${0.25})`
        ctx.lineWidth = 4

        /*
        * 动画帧绘制方法.
        * currentTime是requestAnimation执行回调方法step时会传入的一个执行时的时间(由performance.now()获得).
        * */
        const step = (currentTime) => {
            // ctx.clearRect(startX - 4, startY - 4, Math.abs(endX - startY) + 8, Math.abs(endY - startY) + 8)
            // 第一帧绘制时记录下开始的时间
            !startTime && (startTime = currentTime)
            // 已经过去的时间(ms)
            const timeElapsed = currentTime - startTime
            // 动画执行的进度 {0,1}
            const progress = Math.min(timeElapsed / duration, 1)

            // 绘制方法
            const draw = () => {
                // 计算这一帧中线段应该到达的坐标点
                nextX = startX + (endX - startX) * progress
                nextY = startY + (endY - startY) * progress
                // 用直线连接子路径的最后的点到(nextX,nextY)坐标
                ctx.lineTo(nextX, nextY)
                // 绘制路径(所有子路径都会被绘制一次)
                ctx.stroke()
            }
            draw()

            if (progress < 1) {
                requestAnimationFrame(step)
            } else {
                console.log('动画执行完毕')
            }
        }

        requestAnimationFrame(step)
    }
</script>
```

**codepen传送门**：https://codepen.io/channinghan/pen/GRNJbqg

效果图：

![直线路径动画——思路二.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09c37e45e5764112b7f1b10df7828635~tplv-k3u1fbpfcp-zoom-1.image)

### 加入缓动函数（easing function）

既然要写动画，要让我们的动画效果更加真实、丰富，那么就得应用到**缓动函数**（**easing function**）了，这其实在我们写css中非常常见（在css中也可称之为**timing function**），比如

```
transition:  all 600ms ease-in-out;
```

上面的**ease-in-out**就是指定使用ease-in-out缓动函数来进行动画，而在css中有一定的限制，并不支持所有的缓动函数，或者指定贝塞尔曲线（cubic-bezier）来实现其他的缓动函数。

那么在我们的直线路径动画中也可以加入缓动函数，而这些缓动函数可以在各种地方找到，这里我用的是**tween.js**中的缓动函数：https://github.com/tweenjs/tween.js/blob/master/src/Easing.ts，

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e094c30eade4659b4b655dc471c90b0~tplv-k3u1fbpfcp-zoom-1.image)

其中还有很多缓动函数，而对常用缓动函数也很好理解，**amount**就是对应我们的动画进度**progress**，比如二次方的缓动函数Quadratic.In，就是把返回进度的平方，而基于简单的数学常识，我们直到y=x^2在x的区间[0，1]上的图像是长这样的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a90d2a62008c4568a29f9f33bb638f8a~tplv-k3u1fbpfcp-zoom-1.image)

可以想象得到，这个动画开始是比较缓慢的，然后越来越快。

Quadratic.Out则几乎相反，先快后慢。

而Quadratic.InOut就是先慢再快后慢。



我们在理解记忆缓动函数的名称时可以这么理解，**ease**就是缓动，缓慢的意思，ease后面是**in**就是缓慢的进入，**out**是缓慢地出去，**in-out**则是缓慢地进来和出去。



那么，在我们的这一条直线的例子中，使用缓动函数也非常简单，progress就是缓动函数中的amount参数，把缓动函得到的结果理解为一个**计算属性**,替代我们原来直接的progress即可：

```javascript
let progress = Math.min(timeElapsed / duration, 1)
progress = Easing.Quadratic.In(progress)
```

**codepen传送门**：https://codepen.io/channinghan/pen/bGBGQym

效果图：

![直线+缓动函数.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5017f99bd21f4fbd8dd467153f79c6a3~tplv-k3u1fbpfcp-zoom-1.image)

### 在虚线中观察帧动画



一般来说，在浏览器上一秒钟会执行60次回调函数，也就是**60帧**（60fps），但浏览器会尽可能保持帧率的稳定，也就是有可能会降低到其他的帧率，比如页面性能差时浏览器可能会选择降到**30fps**，当浏览器的上下文不可见时会降到**4fps**左右甚至更低。



为了更清晰地直观看到requestAniamtionFrame的调用，基于第一种方法，用一个**count**变量记录调用次数，且count为**奇数**时不进行绘制，以此形成**虚线**：

```javascript
<script>
    function handleExecute() {
        // ......
      
        // 期望动画持续的时间
        const duration = 1000
        let count = 0

        const step = (currentTime) => {
            !startTime && (startTime = currentTime)
            const timeElapsed = currentTime - startTime
            const progress = Math.min(timeElapsed / duration, 1)

            // 绘制方法
            const draw = () => {
                ctx.beginPath()
                ctx.moveTo(prevX, prevY)

                // 计算该次线段绘制的终点,并将prevX/Y更新为此值,给下一次绘制的时候使用
                prevX = nextX = startX + (endX - startX) * progress
                prevY = nextY = startY + (endY - startY) * progress

                if (count % 2 === 0) {
                    // 设置线条样式
                    ctx.lineWidth = 20 * progress
                    ctx.strokeStyle = `rgba(${171 * (1 - progress) + 81}, ${160 * progress}, ${255},1)`
                    ctx.lineTo(nextX, nextY)
                    ctx.stroke()
                }
            }
            draw()

            if (progress < 1) {
                count++
                requestAnimationFrame(step)
            } else {
                console.log('动画执行完毕')
                console.log(`${duration}ms内回调执行次数:${count}次`)
            }
        }

        // 向浏览器发送动画执行请求,当浏览器要进行重绘时,会调用我们传入的step方法
        requestAnimationFrame(step)
    }
</script>
```

**codepen传送门**：https://codepen.io/channinghan/pen/poNoyGV

效果图：![虚线路径动画.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6eac3e2a01ab4c0f9c0c300d8fccc8fa~tplv-k3u1fbpfcp-zoom-1.image)



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ccef49f38e549abb4966fc74f5b5a08~tplv-k3u1fbpfcp-zoom-1.image)



可以看到，当我把动画持续时间设置为**1**秒时，requestAnimationFrame内的step帧动画回调执行了**60**次，也就是达到了60帧，甚至你可以数一数图上的有色线段有**30**段，对应的虚线区间也是**30**段，正好**60**段（不信你数）。



## 折线

掌握了一条直线的路径动画，那么折线的路径动画也就迎刃而解，无非就是将折线根据关键**转折坐标点**拆成一段段**直线**。



拆分的方法可能有多种，这里放上我的一种实现。

**codepen传送门**：https://codepen.io/channinghan/pen/yLVNdVP

效果图：

![折线路径动画.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f88680894143414c94f1020301b85953~tplv-k3u1fbpfcp-zoom-1.image)

```javascript
 function handleExecute() {
        const canvas = document.querySelector('#myCanvas')
        const ctx = canvas.getContext('2d')

        // 设置线条样式
        ctx.lineWidth = 7
        ctx.lineJoin = 'round'
        ctx.lineCap = 'round'

        // 顺序定义折线上各个转折点的坐标
        const keyPoints = [
            {x: 250, y: 200},
            {x: 550, y: 200},
            {x: 250, y: 500},
            {x: 550, y: 500},
            {x: 250, y: 200}
        ]
        let prevX = keyPoints[0].x
        let prevY = keyPoints[0].y
        let nextX
        let nextY
        // 第一帧执行的时间
        let startTime;
        // 期望动画持续的时间
        const duration = 900


        // 动画被切分成若干段,每一段所占总进度的比例
        const partProportion = 1 / (keyPoints.length - 1)
        // 缓存绘制第n段线段的n值,为了在进行下一段绘制前把这一段线段的末尾补齐
        let lineIndexCache = 1

        /*
        * 动画帧绘制方法.
        * currentTime是requestAnimation执行回调方法step时会传入的一个执行时的时间(由performance.now()获得).
        * */
        const step = (currentTime) => {
            // 第一帧绘制时记录下开始的时间
            !startTime && (startTime = currentTime)
            // 已经过去的时间(ms)
            const timeElapsed = currentTime - startTime
            // 动画执行的进度 {0,1}
            let progress = Math.min(timeElapsed / duration, 1)
            // 加入二次方缓动函数
            progress = Easing.Quadratic.In(progress)

            // 描述当前所绘制的是第几段线段
            const lineIndex = Math.min(Math.floor(progress / partProportion) + 1, keyPoints.length - 1)

            //  当前线段的进度 {0,1}
            const partProgress = (progress - (lineIndex - 1) * partProportion) / partProportion

            // 绘制方法
            const draw = () => {
                ctx.strokeStyle = `rgba(${81 + 175 * Math.abs(1 - progress * 2)}, ${160 - 160 * Math.abs(progress * 2 - 1)}, ${255},${1})`
                ctx.beginPath()
                ctx.moveTo(prevX, prevY)
                // 当绘制下一段线段前,把上一段末尾缺失的部分补齐
                if (lineIndex !== lineIndexCache) {
                    ctx.lineTo(keyPoints[lineIndex - 1].x, keyPoints[lineIndex - 1].y)
                    lineIndexCache = lineIndex
                }
                prevX = nextX = ~~(keyPoints[lineIndex - 1].x + ((keyPoints[lineIndex]).x - keyPoints[lineIndex - 1].x) * partProgress)
                prevY = nextY = ~~(keyPoints[lineIndex - 1].y + ((keyPoints[lineIndex]).y - keyPoints[lineIndex - 1].y) * partProgress)
                ctx.lineTo(nextX, nextY)
                ctx.stroke()
            }
            draw()

            if (progress < 1) {
                requestAnimationFrame(step)
            } else {
                console.log('动画执行完毕')
            }
        }

        requestAnimationFrame(step)
    }
```



主要就是根据**转折点**划分若干个线段，然后把总进度分成若干段线段进度，每段线段所占的进度比例**partProportion**（比如这里分成四段，那么每段就是0.25），再计算出当前线段的进度**partProgress**，其余就跟一条直线的处理方法差不多了。



值得一提的是，当前线段的进度计算方法是：

```javascript
const partProgress = (progress - (lineIndex - 1) * partProportion) / partProportion
```



由于总进度**progress**并不是线性连续增长的，所以线段进度**partProgress**大概率不会刚好等于1，这也就导致每段线段的末尾一小段不会被绘制出来，或者说下一条线段的起点并不是我们所定义的转折点，当动画持续时间较短或总路径较长时就会因此产生折线走“**捷径**”的现象（想象一下在玩赛车游戏时，在一个发卡弯前有一个捷径入口直接到了发卡弯的另一边）。



为了解决这个问题，我用了一个**lineIndexCache**变量来缓存**lineIndex**，当绘制的线段**lineIndex**不等于**lineIndexCache**时，也就是准备进行下一段线段的绘制时，先把上一次绘制末尾的点到对应转折点的一小段绘制出来，并更新**lineIndexCache**，再开始下一段线段的绘制。



如此传入任意/任意多的转折坐标点，都可以绘制成连续的路径动画。



当然，有了这个思路，即使不连续的路径动画也是好做的，不妨可以自己动手试试。



## 圆

看到这里，圆的路径动画其实就很简单了，当然你可以用像webgl中画圆一样用足够多的三角形片元去堆叠出一个圆的思路，用足够多的线段去绘制一个“圆”，但未免有点麻烦了，canvas有封装好的画圆专用api：[`arc(x, y, radius, startAngle, endAngle, anticlockwise)`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/arc)，（其实还有**arcTo**方法，但官方不推荐，因为这个方法的实现有点“不靠谱”）。



圆的路径动画实现思路就是每个动画帧绘制其中一个**角度范围**的**圆弧**。

**codepen传送门**：https://codepen.io/channinghan/pen/poNJXed

效果图：



![圆.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31ca35d6fb5f429bbff696d48af7692d~tplv-k3u1fbpfcp-zoom-1.image)



核心代码：

```javascript
 function handleExecute() {
        // 获取canvas元素
        const canvas = document.querySelector('#myCanvas')
        // 获取canvas渲染上下文
        const ctx = canvas.getContext('2d')

        // 设置线条样式
        ctx.lineWidth = 7
        ctx.lineJoin = 'round'
        ctx.lineCap = 'round'

        // 定义圆心的坐标点
        // const center = {x: ctx.canvas.width / 2, y: ctx.canvas.height / 2}
        const center = {x: 400, y: 400}
        // 定义圆的半径大小
        const radius = 200
        // 定义起点和终点的角度
        const startAngle = 0
        const endAngle = 2 * Math.PI
        let prevAngle = startAngle
        let nextAngle
        // 第一帧执行的时间
        let startTime;
        // 期望动画持续的时间
        const duration = 900

        /*
        * 动画帧绘制方法.
        * currentTime是requestAnimation执行回调方法step时会传入的一个执行时的时间(由performance.now()获得).
        * */
        const step = (currentTime) => {
            // 第一帧绘制时记录下开始的时间
            !startTime && (startTime = currentTime)
            // 已经过去的时间(ms)
            const timeElapsed = currentTime - startTime
            // 动画执行的进度 {0,1}
            let progress = Math.min(timeElapsed / duration, 1)
            progress = Easing.Cubic.In(progress)
            // 绘制方法
            const draw = () => {
                // 创建新的路径
                ctx.beginPath()
                // 计算这一帧中圆弧应该到达的角度
                nextAngle = startAngle + (endAngle - startAngle) * progress
                // 创建一段圆弧
                ctx.arc(center.x, center.y, radius, prevAngle, nextAngle, false)
                // 设置渐变的颜色
                ctx.strokeStyle = `rgba(${81 + 171 * Math.abs(1 - progress * 2)}, ${160 - 160 * Math.abs(1 - progress * 2)}, ${255},1)`
                // 把这一帧的圆弧绘制出来
                ctx.stroke()
                //将prevAngle更新为这一帧中的nextAngle给下一帧使用
                prevAngle = nextAngle
            }
            draw()

            if (progress < 1) {
                requestAnimationFrame(step)
            } else {
                console.log('动画执行完毕')
            }
        }

        requestAnimationFrame(step)
    }
```

关键的其实就只有这么一行：

```javascript
// 计算这一帧中圆弧应该到达的角度
                nextAngle = startAngle + (endAngle - startAngle) * progress
```

你会发现这跟我们在做直线路径动画的时候原理是一样的。



说到圆，很难不想到圆弧，然后你可能就会想起一位**名人**....

![优弧.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed0b6c07e4fa4e54bebca8ba2bfa6325~tplv-k3u1fbpfcp-zoom-1.image)

gif有点掉帧了

**codepen传送门**：https://codepen.io/channinghan/pen/ExNjBmd

## 贝塞尔曲线

canvas中有**二次**和**三次****贝塞尔曲线**，本身使用它有一定的难度，但有足够的耐心你将可以用来绘制复杂而有规律的图形。



我们先看看绘制它们的Canvas API：

> 绘制二次贝塞尔曲线：`quadraticCurveTo(cp1x, cp1y, x, y)`
>
> `cp1x,cp1y`为一个控制点，`x,y为`结束点
>
> 
>
> 绘制三次贝塞尔曲线：`bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)`
>
> `cp1x,cp1y`为控制点一，`cp2x,cp2y`为控制点二，`x,y`为结束点。



你可以理解为**lineTo**方法，只不过它需要多传一个或两个**控制点**的坐标参数进去，其底层的运算也更为复杂一些。



如果你不了解贝塞尔曲线又对它感兴趣的话可以推荐看一篇非常不错的文章：

[用canvas绘制一个曲线动画——深入理解贝塞尔曲线](https://github.com/hujiulong/blog/issues/1)



绘制贝塞尔曲线的路径动画岂不是难上加难？Canvas是绘制一条完整的贝塞尔曲线，我们似乎没办法直接去画其中的一部分。



于是只能另寻他路：

1. 方法一：计算出贝塞尔曲线上的点，并且将足够多的点用**直线**连接起来。
2. 方法二：通过观察分析贝塞尔曲线的特征，将**部分贝塞尔曲线**中的控制点找出来，据此画出这些部分贝塞尔曲线。





首先，无论是方法一还是方法二，都需要将**贝塞尔曲线上的任意某个点**的坐标求出来。

如果让自己推导出来贝塞尔曲线上的某个点坐标那是十分困难的，但我们可以轻易找到它的**曲线方程**，百度/google一下就完事了。



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08b49d869a3d4877adb84b56e2d6dee9~tplv-k3u1fbpfcp-zoom-1.image)



**P0**就是**起点坐标**（canvas中默认为当前路径的起点），**P2**(二次贝塞尔)或**P3**（三次贝塞尔）则是**终点坐标**，其余中间的是控制贝塞尔曲线的**控制点坐标**。



根据公式，**B点**就是我们要求的贝塞尔曲线上的某个点，知道这一点就其实足以让我们用方法一绘制出它们的路径动画了，但这种方式比较丑陋，因为我们已经知道如何得到任意进度下点的坐标，将足够多的路径上的点用**直线**连接起来即可。



鉴于第一种方法实现起来并不“优雅”，所以我还是想折腾折腾，尝试用**第二种方法**来完成。



而第二种方法还需要一个条件：**部分贝塞尔曲线上的控制点**，我暂且给它取个名字**SC**（sub control point），意为**子控制点**。



那么这个**子控制点**如何得到呢？



### 二次贝塞尔曲线



首先，我们来看一个**二次贝塞尔曲线**的动图：





![image](https://camo.githubusercontent.com/0f3f1e0cef5bfe1f6c8742be581df2118d36e91df286f6f7438a44432eb3627b/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031372f31322f32352f313630386531393239373836333535623f773d32343026683d31303026663d67696626733d3734323734)



然后就是格物致知，当你看了若干次这个动图之后，会产生这么一种**直觉**：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebca647e5b59453d86b183f65ad779e1~tplv-k3u1fbpfcp-zoom-1.image)



子控制点就在**P0-P1**这条线段上，并且其位置与我们的进度**progress**相关，现在我们大胆假设：

```javascript
sc.x = p0.x + p1.x - p0.x
sc.y = p0.y + p1.y - p0.y
```

然后小心求证，用具体代码实现它看看：

```javascript
function handleExecute() {

        // 计算出子控制点的坐标
        function calSC(t) {
            SC.x = p0.x + (p1.x - p0.x) * t
            SC.y = p0.y + (p1.y - p0.y) * t
        }

        // 计算出子贝塞尔曲线的终点
        function calB(t) {
            B.x = Math.pow(1 - t, 2) * p0.x + 2 * t * (1 - t) * p1.x + Math.pow(t, 2) * p2.x
            B.y = Math.pow(1 - t, 2) * p0.y + 2 * t * (1 - t) * p1.y + Math.pow(t, 2) * p2.y
        }


        // 获取canvas元素
        const canvas = document.querySelector('#myCanvas')
        // 获取canvas渲染上下文
        const ctx = canvas.getContext('2d')


        // 设置线条样式
        ctx.strokeStyle = 'rgba(81, 160, 255,1)'
        ctx.lineWidth = 4
        ctx.lineJoin = 'round'

        // 第一帧执行的时间
        let startTime;
        // 期望动画持续的时间
        const duration = 1000

        // 起点
        const p0 = {x: 100, y: 500}
        // 控制点
        const p1 = {x: 200, y: 100}
        // 终点
        const p2 = {x: 700, y: 500}
        // 子控制点(这里初始化的坐标不重要，先设置成p0的值)
        const SC = {...p0}
        // 子贝塞尔曲线上的终点(这里初始化的坐标不重要，先设置成p0的值)
        let B = {...p0}

        // 先画一条完整的贝塞尔曲线以验证贝塞尔曲线动画的准确性
        ctx.beginPath()
        ctx.moveTo(p0.x, p0.y)
        ctx.strokeStyle = '#e3e3e3'
        ctx.quadraticCurveTo(p1.x, p1.y, p2.x, p2.y)
        ctx.stroke()

        // 随便画个眼睛（不重要）
        function drawEye(color) {
            ctx.beginPath()
            ctx.strokeStyle = color
            ctx.arc(p0.x + 100, p0.y - 50, 50, 0, 2 * Math.PI, false)
            ctx.stroke()
            ctx.moveTo(p0.x + 300, p0.y - 50)
            ctx.arc(p0.x + 250, p0.y - 50, 50, 0, 2 * Math.PI, false)
            ctx.stroke()
        }

        drawEye('rgb(227, 227, 227)')



        /*
        * 动画帧绘制方法.
        * currentTime是requestAnimation执行回调方法step时会传入的一个执行时的时间(由performance.now()获得).
        * */
        const step = (currentTime) => {
            // 第一帧绘制时记录下开始的时间
            !startTime && (startTime = currentTime)
            // 已经过去的时间(ms)
            const timeElapsed = currentTime - startTime
            // 动画执行的进度 {0,1}
            let progress = Math.min(timeElapsed / duration, 1)
            progress = Easing.Quadratic.In(progress)


            // 绘制方法
            const draw = () => {
                ctx.beginPath()
                ctx.moveTo(p0.x, p0.y)
                // 计算并更新B和SC的坐标
                calB(progress)
                calSC(progress)
                // 用直线将刚刚moveTo中的点连接到(nextX,nextY)上
                ctx.quadraticCurveTo(SC.x, SC.y, B.x, B.y)
                ctx.strokeStyle = `rgba(${171 * (1 - progress) + 81}, ${160 * progress}, ${255},1)`
                ctx.stroke()
                // 眼睛渐变色
                drawEye(`rgba(${227 - (227 - 81) * progress}, ${227 - (227 - 160) * progress}, ${255},1)`)
            }
            draw()

            if (progress < 1) {
                requestAnimationFrame(step)
            } else {
                console.log('动画执行完毕')
            }
        }

        setTimeout(() => {
            requestAnimationFrame(step)
        }, 1000)
    }
```

核心就是提取了两个计算方法calcB和calSC用于计算并更新子贝塞尔曲线的终点和子控制点，然后使用贝塞尔曲线的绘制api：

```javascript
ctx.quadraticCurveTo(SC.x, SC.y, B.x, B.y)
```

**codepen传送门**：https://codepen.io/channinghan/pen/eYBNwRK

在执行动画前，我先将完整的贝塞尔曲线用灰色线条绘制出来，再执行动画看看能不能准确覆盖上以检验我们的动画方法是否正确：

![二次贝塞尔曲线.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1b8b5c8c817401eba35f8096759f82d~tplv-k3u1fbpfcp-zoom-1.image)

god bless🙏



### 三次贝塞尔曲线

对于三次贝塞尔曲线，老样子还是先看它的动图：


![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc07705955394a038147494a560991cb~tplv-k3u1fbpfcp-zoom-1.image)



在二次贝塞尔曲线中找到了感觉，那么不难感觉到两个子控制点在哪里了，大胆猜测：



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7594e250731e446b98e1508e09ce1ef4~tplv-k3u1fbpfcp-zoom-1.image)

那么我们只需要添加**SC1**、**SC2**、**SC3**和修改**B**坐标的计算方法即可，其中SC1、SC2为**子控制点**，SC3用于计算出SC2的坐标。

这些点的计算方法为：

```javascript
// 计算出子控制点1的坐标
        function calSC1(t) {
            SC1.x = p0.x + (p1.x - p0.x) * t
            SC1.y = p0.y + (p1.y - p0.y) * t
        }

        // 计算用于计算子控制点2的坐标的点坐标
        function calSC3(t) {
            SC3.x = p1.x + (p2.x - p1.x) * t
            SC3.y = p1.y + (p2.y - p1.y) * t
        }

        // 计算出子控制点2的坐标
        function calSC2(t) {
            SC2.x = SC1.x + (SC3.x - SC1.x) * t
            SC2.y = SC1.y + (SC3.y - SC1.y) * t
        }

        // 计算出子贝塞尔曲线的终点
        function calB(t) {
            B.x = Math.pow(1 - t, 3) * p0.x + 3 * t * Math.pow(1 - t, 2) * p1.x + 3 * p2.x * Math.pow(t, 2) * (1 - t) + Math.pow(t, 3) * p3.x
            B.y = Math.pow(1 - t, 3) * p0.y + 3 * t * Math.pow(1 - t, 2) * p1.y + 3 * p2.y * Math.pow(t, 2) * (1 - t) + Math.pow(t, 3) * p3.y
        }
```

帧动画中的核心方法：

```javascript
ctx.beginPath()
ctx.moveTo(p0.x, p0.y)
// 计算并更新B和SC1、SC2、SC3的坐标
calB(progress)
calSC1(progress)
calSC3(progress)
calSC2(progress)
// 用三次贝塞尔曲线将刚刚moveTo中的点连接到B上
ctx.bezierCurveTo(SC1.x, SC1.y, SC2.x, SC2.y, B.x, B.y)
ctx.strokeStyle = `rgba(${171 * (1 - progress) + 81}, ${160 * progress}, ${255},1)`
ctx.stroke()
```

其余跟二次贝塞尔曲线的路径动画差不多了。

**codepen传送门**：https://codepen.io/channinghan/pen/wvoaLqg

让我们再次验证一下三次贝塞尔曲线的路径动画：

![三次贝塞尔曲线.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5dbde405cbc641c4a8187fdad48db9e7~tplv-k3u1fbpfcp-zoom-1.image)

god bless again🙏



其实我也不是一次就成功，当子控制点选得不对就会出现一些有趣的现象：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26f30bf74fb64ccfb0a4c791a272f712~tplv-k3u1fbpfcp-zoom-1.image)



## 最后

到这里canvas的各种图形的路径动画做法基本已经分享完了，其实并不一定会将此运用到日常工作中，同时也有其他的途径去完成路径动画，如css、svg等。



之所以我花这么大的篇幅写这么一件事，首先主要是因为自己觉得有意思，其次在这个过程中既更加了解canvas也更明白动画背后的一些原理。



> 感谢大家可以耐心地读到这里。
>
> 当然，文中或许会存在不足、错误之处，非常欢迎大家在**评论**里与我交流。
>
> 文中所用的所有**Demo**都已放在：**[GitHub传送门](https://github.com/ChanningHan/canvas-path-animation-demos)**。
>
> 最后，希望朋友们可以**点赞、评论、关注**三连，这些就是我分享的全部动力来源🙏