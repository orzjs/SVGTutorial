# 动态粒子连线

__如果大家玩儿过知乎的话，大家一定对知乎网站的登录页面有深刻的印象，[点我到知乎](https://www.zhihu.com/).__    
可以看到，知乎背景自动位移连线的粒子视觉效果很不错，有点高大上的感觉。    

再来看一个：    
[点我点我](http://vincentgarreau.com/particles.js/)    

__看起来是不是很炫酷，今天我们用他十分之一的代码量来将之还原！(当然只是基本还原啦～)__    


[我的DEMO](https://pengjiyuan.github.io/particles.js/)    

当然，本次的svg实现的这个动画是教科书般的反面教材，我是用的canvas的重绘思想，（remove svg元素，append svg 元素），给浏览器造成了极大的负担，svg本身有很多的动画方式，等有时间了我会重新用比较合理的方法重新还原一下。

为什么还要讲这个实现方法呢？因为这种方法会让大家对js动态控制svg有一定的了解（笨方法也就笨方法的用处..）。

### 废话少说，直接开始。

### Step1 画点

* 首先，定义一些变量。
```javascript
  var width = document.body.clientWidth,//获取屏幕宽度
  height = document.body.clientHeight,//获取屏幕高度
  ns = "http://www.w3.org/2000/svg",//待用明明空间
  distance = 300,//点之间距离上限（判断是否连线）
  lineColor = "rgba(255, 255, 255, 0.5)",//线的颜色
  points = [];//存放点的信息的数组
```


* 我们所有的点的信息（包括圆心，半径，移动轨迹）都随机生成，放到数组里边备用。
```javascript
  for(let i = 0; i < 20; i++) {
    points.push(getPointData(1));
  }

  //随机生成点的信息,返回值为json对象
  function getPointData(radius) {

    var x = Math.ceil(Math.random()*width),
      y = Math.ceil(Math.random()*height),
      r = +(Math.random()*radius + 2).toFixed(4),
      rateX = +(Math.random()*2-1).toFixed(4),
      rateY = +(Math.random()*2-1).toFixed(4);

    return {
      x: x,
      y: y,
      r: r,
      rateX: rateX,
      rateY: rateY
    }

  }
```


* 可以画点了    

```javascript

function drawPoints() {
  var rootSvg = document.getElementById("my");

  points.forEach(function(item) {

    let circle = document.createElementNS(ns, "circle");
    circle.setAttribute("cx", item.x);
    circle.setAttribute("cy", item.y);
    circle.setAttribute("r", item.r);
    //透明度的设置，点越大，透明度越小，反之亦然
    circle.setAttribute("fill", "rgba(255, 255, 255, "+ (item.r-2)/1 +")");
    rootSvg.appendChild(circle);
    
  });
}
```

### Step2 备用函数准备

* 两点之间距离的计算(大家都会吧～)
```javascript
function dis(x1, y1, x2, y2) {
  var disX = Math.abs(x1 - x2),
    disY = Math.abs(y1 - y2);

  return Math.sqrt(disX * disX + disY * disY);
}
```


* 两点之间画线
```javascript
function line(x1, y1, x2, y2, strokeWidth, color) {
  var rootSvg = document.getElementById("my");
  let line = document.createElementNS(ns, "line");
    line.setAttribute("x1", x1);
    line.setAttribute("x2", x2);
    line.setAttribute("y1", y1);
    line.setAttribute("y2", y2);
    line.setAttribute("stroke-width", strokeWidth);
    line.setAttribute("stroke", color);
  rootSvg.appendChild(line);
}
```

### Step3 画线

* 判断两点之间的距离，如果小于我们定义的distance，就连线。    

```javascript
//这边我们用两个for循环来做
function drawLines() {

  for(let i = 0, len = points.length;i < len;i++) {
    for(let j = len - 1; j >= 0; j--) {
      let x1 = points[i].x,
        y1 = points[i].y,
        x2 = points[j].x,
        y2 = points[j].y,
        disPoint = dis(x1, y1, x2, y2);

      if(disPoint <= distance) {
        //线的宽度设置为：距离越长，线越细，反之越粗
        line(x1, y1, x2, y2, 1 - disPoint/distance, lineColor);
      }
    }
  }

}
```

__现在我们得到了不会动的一张点连线图，接下来就是让它动起来。__    

### Step4 Move!

```javascript
function move() {
  //在重绘之前先清除所有的svg元素（不推荐，仅供演示用）
  var child = document.getElementById("my");
  //如果没有svg根元素，直接添加，有的话就清除
  if(child) {
    child.parentNode.removeChild(child);
  }
  //添加svg根元素
  newSvg = document.createElementNS(ns, "svg");
  newSvg.setAttribute("width", "100%");
  newSvg.setAttribute("height", "100%");
  newSvg.setAttribute("id", "my");
  document.body.appendChild(newSvg);
  //对点信息数组进行操作，利用我们随机生成的移动数据对点的圆心进行变换
  points.forEach(function(item, i) {
    if(item.x > 0 && item.x < width && item.y > 0 && item.y < height) {
      item.x += item.rateX*2;
      item.y += item.rateY*2;
    } else {//如果点已经移出去屏幕了，从数组里边移除这个点，同时新增加一个点。
      points.splice(i, 1);
      points.push(getPointData(1));
    }
  });
  drawPoints();
  drawLines();
  //move
  window.requestAnimationFrame(move);

}
```

### BINGO!

canvas实现更好，可以看我的canvas实现。
https://github.com/PengJiyuan/particles.js/





