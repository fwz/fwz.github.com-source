title: "An interactive tree in Processing"
id: 181
date: 2012-06-07 13:28:09
tags: 
- Processing
- Visualization
categories: [Engineering, Visualization]

---

这是一份processing写就的树形可视化，大家可以猜猜需要用多少代码。有经验的朋友应该能看出来核心代码不会很多，大概40行。出处在[这里](http://www.openprocessing.org/sketch/8941 "treecursion")。为了能在用js打开以及流畅的用户体验，我修改了里面代码的一些地方。我觉得这份代码有些奇妙的地方，所以学习了一下。<!--more-->
## Sketch
<html>
      <script src="/js/processing.min.js"></script>
      <canvas data-processing-sources="tree.pde"></canvas>
</html>

## Code
{% raw %}

float rotx = 0;
float roty = 0;
float desc_factor = sqrt(2)/2.0;
float growth = 0;
float prop_scale = 0;

void setup() {
    size(600, 450, P2D);
}

void draw() {
    background(240);
    stroke(0);

    rotx += (radians(360./height*mouseX)-rotx)/10;
    roty += (radians(360./height*mouseY)-roty)/10;

    translate( width/2, height/3*2);
    line(0, 0, 0, width/3); // the trunk
    branch (height/4.0, 11);
    if (keyPressed) {
        if (key == 'j') prop_scale += 0.02;
        else if (key == 'k') prop_scale -=0.02;
    }
    growth += (1 - growth + prop_scale) / 10;
}

void branch(float len, int num) {
    len *= desc_factor;
    num -= 1;
    if (len >= 2 && num > 0) {
        pushMatrix();
        rotate(rotx);
        line(0, 0, 0, -len);
        translate(0, -len);
        branch(len, num);
        popMatrix();

        len *= growth;

        pushMatrix();
        rotate(rotx - roty);
        line(0, 0, 0, -len);
        translate(0, -len);
        branch( len, num);
        popMatrix();
    }
}

{% endraw %}

简单介绍一下processing程序的工作方式，setup是程序开始之前需要做的事情，一般就是设置一下画布大小，设定画笔颜色等等，和awk的BEGIN块类似。
然后processing不断地去运行draw()函数，每个draw()函数运行完毕，就会生成一个帧，而每秒钟跑多少帧取决于frameRate()函数，默认值好像是30。

从代码比较容易能够看到，使用的是DFS写成的递归生成树，随着鼠标的移动，树的形态会不断变化。

那么奇妙的地方在哪里呢？这棵树的交互非常平滑，当我们不再移动鼠标的时候，树枝会平滑地过渡到结果的点，为什么呢？

首先我们看，这样一棵树，需要哪几个因素就可以确定下来，代码里面又是如何处理的。

1.  两个分支长度的比值，growth变量确定
2.  两个分支之间的夹角，branch里面使用(curlx 和 curly 的值决定）
3.  递归的停止条件（这里是树枝的最短长度以及最大代数），branch定义。
然后我们分析一下，最重要的树枝旋转平滑。其实就是缓动两个分支的角度。功能由这个式子完成

{% raw %}
rotx += (radians(360./height*mouseX)-rotx)/delay;
{% endraw %}

我们可以考虑curlx 在draw()中每一个时刻的值所形成的序列是一个数列 {An}。同时，我们认为mouseX（鼠标在画布中的横坐标）在缓动过程中也是不变的，所以radians(360./height*mouseX)这个数可以定为常量C，delay也是常量,记为d,我硬编码为10了。
那么我们可以得到An的通项公式：An+1 = An + (C - An) / d
利用中学的数列知识可以将上面的式子化为： An+1 - C = (An - C) * (d-1) / d，也就是说{An - C}是一个公差为 (d-1)/d 的等比数列。由于(d-1) / d &lt; 1，所以{An - C}会收敛于0，即最终An -&gt; C。因此这行代码就实现了对旋转角度的逐渐逼近。

假如觉得这种推论太坑爹的话，也可以理解为，假设当前到目的地的距离为S，每一帧会移动10/S，下一秒的S'=9S/10

growth是作者提供的一个用于调整左右枝的长短比例的值，同样使用上述的方式实现平滑的过渡。
作者提供了一个可以通过鼠标滚轮进行的操作（抱歉这里processing.js不支持java的库）。我将其改成用"jk"键实现了。

很简单的实现，效果也很自然。假如在processing客户端中实现，树枝的层次可以开到17，仍然非常流畅的显示。
