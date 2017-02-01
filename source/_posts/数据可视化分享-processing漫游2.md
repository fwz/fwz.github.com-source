title: "数据可视化分享——Processing漫游(2)"
id: 284
date: 2012-11-11 18:02:39
tags: 
- Processing
categories: 
- Engineering
- Visualization
---

今天看到一个非常性感的作品，按捺不住就做了个Processing的版本。所以抱歉了，这个漫游系列硬生生地把它插入。
继续阅读查看效果吧，按S保存截图，R清空屏幕重新绘图。

<!--more-->

这个例子来自Patrick Gunderson，看到这个例子的时候，我非常好奇，这究竟是用什么算法实现的？旁边有图形帝说可以用热学传导模型加偏微分方程模拟，我瞬间就给跪了。后来找到了作者的介绍，发现算法简单得让人发指。不敢独美，与大家分享。

## Sketch
{% raw %}
<script src="http://7ktqal.com1.z0.glb.clouddn.com/processing.min.js"></script>
<script type="text/processing" data-processing-target="processing-canvas">
// This is a naive implement of ablaze.js in processing
// Thanks Patrick Gunderson's great work

// Wenzhong @ Beijing, Oct 2012
// wenzhong.work@gmail.com


// build config
int PARTICLE_NUM = 60;
int MIN_DIST = 70;
float MIN_SPEED = 0.25;
float MAX_SPEED = 0.75;
float RING_SIZE = 150;
boolean line_connect = true;

// show config
boolean use_color_map = false;
boolean use_color_gradient = false;
float clr;
float NOISE_SCALE = 0.02;
float frame_cnt = 0;
PImage clrmap;

Particle [] ps;

class Particle {
  public float x, y, rot_angle;
  private float vx, vy;
  private int step;

  Particle(float x, float y, float v, float v_angle) {
    this.x = x;
    this.y = y;
    this.vx = v*cos(v_angle);
    this.vy = v*sin(v_angle);
    generate_new_curve();
  }

  public void next() {
    turn(rot_angle);
    x += vx;
    y += vy;
    step--;
    if (step <= 0) {
      generate_new_curve();
    }
  }

  public void turn(float rot) {
    float pvx = vx;
    float pvy = vy;
    vx = pvx * cos(rot) - pvy * sin(rot);
    vy = pvx * sin(rot) + pvy * cos(rot);
  }

  private void generate_new_curve() {
    step = (int)random(80, 300);
    rot_angle = random(-0.005, 0.005);
  }
}

void setup() {
  reset();
}
void reset() {
  size( 600, 600);
  background(0);
  smooth();

  ps = new Particle[PARTICLE_NUM];

  // init particles
  for (int i = 0; i < PARTICLE_NUM; i++) {
    float v = random(MIN_SPEED, MAX_SPEED);
    float angle = random(0, 2*PI);
    float v_angle = random(0, 2*PI);

    ps[i] = new Particle(RING_SIZE * cos(angle), 
    RING_SIZE * sin(angle), 
    v, 
    v_angle);
  }
  clr = random(1.0);

  // load color map
  if (use_color_map)
    load_color_map();
}

void draw() {
  if (use_color_map) {
    colorMode(RGB, 255);
    generateGraph( "colormap");
  } else {
    filter(INVERT);
    colorMode(HSB, 1);
    if (use_color_gradient) {
      clr += random(0.001);
      clr = clr % 1;
    }
    generateGraph("rand");
    filter(INVERT);
  }
}

void changeColorGradient() {
  use_color_gradient = !use_color_gradient;
}

void generateGraph( String coloring_type) {

  translate(width/2, height/2);
  for (int i = 0; i < PARTICLE_NUM; i++) {
    for (int j = i+1; j < PARTICLE_NUM; j++) {
      float dis = dist(ps[i].x, ps[i].y, ps[j].x, ps[j].y);
      if ( dis < MIN_DIST ) {
        if (coloring_type == "rand") {
          float k = (0.5 + float(i)/2.0) / PARTICLE_NUM;
          fill(clr, pow(k, 0.1), 0.9 * sqrt(1-k), 0.1 * (1  - dis / MIN_DIST));
          stroke(clr, pow(k, 0.1), 0.9 * sqrt(1-k), 0.1 * (1  - dis / MIN_DIST));
        } else if (coloring_type == "colormap") {
          color c = set_color_by_map(ps[i]);
          fill((c >> 16) & 0xFF, (c >> 8) & 0xFF, c & 0xFF, 32 * (1 - dis / MIN_DIST));
          stroke((c >> 16) & 0xFF, (c >> 8) & 0xFF, c & 0xFF, 32 * (1 - dis / MIN_DIST));
        }
        buildConnect(ps[i], ps[j], dis);
      }
    }
    ps[i].next();
  }
}

void keyPressed() {
  switch(key) {
  case 'g':
    changeColorGradient();
    break;
  case 't': 
    changeConnectionBuild(); 
    break;
  case 'r': 
    reset(); 
    break;
  case 's': 
    saveFrame(); 
    break;
  case 'c': 
    changeColor(); 
    break;
  }
}

void buildConnect(Particle p1, Particle p2, float dis) {

  if (line_connect) {
    line(p1.x, p1.y, p2.x, p2.y);
  } else {
    float cx = (p1.x + p2.x)/2;
    float cy = (p1.y + p2.y)/2;
    noFill();
    ellipse(cx, cy, dis/2, dis/2);
  }
}

void changeConnectionBuild() {
  line_connect = ! line_connect;
}

void changeColor() {
  use_color_map = ! use_color_map;
}

void load_color_map() {
  clrmap = loadImage("chocolate.png");
  blurImage(clrmap);
}

color set_color_by_map(Particle p) {
  if (clrmap.width > 0) {
    if (abs(p.x) >= width/2 - 1 || abs(p.y) >= height/2 - 1) return 0;
    float x = map(p.x, -width/2, width/2, 0, clrmap.width);
    float y = map(p.y, -height/2, height/2, 0, clrmap.height);
    if ((int)((y) * clrmap.width) + (int) (x-1) >= clrmap.width * clrmap.height)
      return 0;
    color c = clrmap.pixels[(int)((y) * clrmap.width) + (int) (x-1)];
    return c;
  }
  return 0;
}

// shamlessly taken from the processing.org's, for blurring image
void blurImage(PImage img) {
  float v = 1.0 / 25.0;
  float[][] kernel = {
    { 
      v, v, v, v, v
    }
    , 
    { 
      v, v, v, v, v
    }
    , 
    { 
      v, v, v, v, v
    }
    , 
    { 
      v, v, v, v, v
    }
    , 
    { 
      v, v, v, v, v
    }
  };

  // Loop through every pixel in the image
  for (int y = 2; y < img.height-2; y++) {   // Skip top and bottom edges
    for (int x = 2; x < img.width-2; x++) {  // Skip left and right edges
      float sum_r = 0, sum_g = 0, sum_b = 0; // Kernel sum for this pixel
      for (int ky = -2; ky <= 2; ky++) {
        for (int kx = -2; kx <= 2; kx++) {
          // Calculate the adjacent pixel for this kernel point
          int pos = (y + ky)*img.width + (x + kx);
          float val_r = red(img.pixels[pos]);
          float val_g = green(img.pixels[pos]);
          float val_b = blue(img.pixels[pos]);

          // Multiply adjacent pixels based on the kernel values
          sum_r += kernel[ky+2][kx+2] * val_r;
          sum_g += kernel[ky+2][kx+2] * val_g;
          sum_b += kernel[ky+2][kx+2] * val_b;
        }
      }
      // For this pixel in the new image, set the gray value
      // based on the sum from the kernel
      img.pixels[y*img.width + x] = color(sum_r, sum_g, sum_b);
    }
  }
}


</script>
<canvas id="processing-canvas"> </canvas>
{% endraw %}

## Code
{% codeblock lang:java %}
// This is a naive implement of ablaze.js in processing
// Thanks Patrick Gunderson's great work

// Wenzhong @ Beijing, Oct 2012
// wenzhong.work@gmail.com


// build config
int PARTICLE_NUM = 60;
int MIN_DIST = 70;
float MIN_SPEED = 0.25;
float MAX_SPEED = 0.75;
float RING_SIZE = 150;
boolean line_connect = true;

// show config
boolean use_color_map = false;
boolean use_color_gradient = false;
float clr;
float NOISE_SCALE = 0.02;
float frame_cnt = 0;
PImage clrmap;

Particle [] ps;

class Particle {
  public float x, y, rot_angle;
  private float vx, vy;
  private int step;

  Particle(float x, float y, float v, float v_angle) {
    this.x = x;
    this.y = y;
    this.vx = v*cos(v_angle);
    this.vy = v*sin(v_angle);
    generate_new_curve();
  }

  public void next() {
    turn(rot_angle);
    x += vx;
    y += vy;
    step--;
    if (step <= 0) {
      generate_new_curve();
    }
  }

  public void turn(float rot) {
    float pvx = vx;
    float pvy = vy;
    vx = pvx * cos(rot) - pvy * sin(rot);
    vy = pvx * sin(rot) + pvy * cos(rot);
  }

  private void generate_new_curve() {
    step = (int)random(80, 300);
    rot_angle = random(-0.005, 0.005);
  }
}

void setup() {
  reset();
}
void reset() {
  size( 600, 600);
  background(0);
  smooth();

  ps = new Particle[PARTICLE_NUM];

  // init particles
  for (int i = 0; i < PARTICLE_NUM; i++) {
    float v = random(MIN_SPEED, MAX_SPEED);
    float angle = random(0, 2*PI);
    float v_angle = random(0, 2*PI);

    ps[i] = new Particle(RING_SIZE * cos(angle), 
    RING_SIZE * sin(angle), 
    v, 
    v_angle);
  }
  clr = random(1.0);

  // load color map
  if (use_color_map)
    load_color_map();
}

void draw() {
  if (use_color_map) {
    colorMode(RGB, 255);
    generateGraph( "colormap");
  } else {
    filter(INVERT);
    colorMode(HSB, 1);
    if (use_color_gradient) {
      clr += random(0.001);
      clr = clr % 1;
    }
    generateGraph("rand");
    filter(INVERT);
  }
}

void changeColorGradient() {
  use_color_gradient = !use_color_gradient;
}

void generateGraph( String coloring_type) {

  translate(width/2, height/2);
  for (int i = 0; i < PARTICLE_NUM; i++) {
    for (int j = i+1; j < PARTICLE_NUM; j++) {
      float dis = dist(ps[i].x, ps[i].y, ps[j].x, ps[j].y);
      if ( dis < MIN_DIST ) {
        if (coloring_type == "rand") {
          float k = (0.5 + float(i)/2.0) / PARTICLE_NUM;
          fill(clr, pow(k, 0.1), 0.9 * sqrt(1-k), 0.1 * (1  - dis / MIN_DIST));
          stroke(clr, pow(k, 0.1), 0.9 * sqrt(1-k), 0.1 * (1  - dis / MIN_DIST));
        } else if (coloring_type == "colormap") {
          color c = set_color_by_map(ps[i]);
          fill((c >> 16) & 0xFF, (c >> 8) & 0xFF, c & 0xFF, 32 * (1 - dis / MIN_DIST));
          stroke((c >> 16) & 0xFF, (c >> 8) & 0xFF, c & 0xFF, 32 * (1 - dis / MIN_DIST));
        }
        buildConnect(ps[i], ps[j], dis);
      }
    }
    ps[i].next();
  }
}

void keyPressed() {
  switch(key) {
  case 'g':
    changeColorGradient();
    break;
  case 't': 
    changeConnectionBuild(); 
    break;
  case 'r': 
    reset(); 
    break;
  case 's': 
    saveFrame(); 
    break;
  case 'c': 
    changeColor(); 
    break;
  }
}

void buildConnect(Particle p1, Particle p2, float dis) {

  if (line_connect) {
    line(p1.x, p1.y, p2.x, p2.y);
  } else {
    float cx = (p1.x + p2.x)/2;
    float cy = (p1.y + p2.y)/2;
    noFill();
    ellipse(cx, cy, dis/2, dis/2);
  }
}

void changeConnectionBuild() {
  line_connect = ! line_connect;
}

void changeColor() {
  use_color_map = ! use_color_map;
}

void load_color_map() {
  clrmap = loadImage("chocolate.png");
  blurImage(clrmap);
}

color set_color_by_map(Particle p) {
  if (clrmap.width > 0) {
    if (abs(p.x) >= width/2 - 1 || abs(p.y) >= height/2 - 1) return 0;
    float x = map(p.x, -width/2, width/2, 0, clrmap.width);
    float y = map(p.y, -height/2, height/2, 0, clrmap.height);
    if ((int)((y) * clrmap.width) + (int) (x-1) >= clrmap.width * clrmap.height)
      return 0;
    color c = clrmap.pixels[(int)((y) * clrmap.width) + (int) (x-1)];
    return c;
  }
  return 0;
}

// shamlessly taken from the processing.org's, for blurring image
void blurImage(PImage img) {
  float v = 1.0 / 25.0;
  float[][] kernel = {
    { 
      v, v, v, v, v
    }
    , 
    { 
      v, v, v, v, v
    }
    , 
    { 
      v, v, v, v, v
    }
    , 
    { 
      v, v, v, v, v
    }
    , 
    { 
      v, v, v, v, v
    }
  };

  // Loop through every pixel in the image
  for (int y = 2; y < img.height-2; y++) {   // Skip top and bottom edges
    for (int x = 2; x < img.width-2; x++) {  // Skip left and right edges
      float sum_r = 0, sum_g = 0, sum_b = 0; // Kernel sum for this pixel
      for (int ky = -2; ky <= 2; ky++) {
        for (int kx = -2; kx <= 2; kx++) {
          // Calculate the adjacent pixel for this kernel point
          int pos = (y + ky)*img.width + (x + kx);
          float val_r = red(img.pixels[pos]);
          float val_g = green(img.pixels[pos]);
          float val_b = blue(img.pixels[pos]);

          // Multiply adjacent pixels based on the kernel values
          sum_r += kernel[ky+2][kx+2] * val_r;
          sum_g += kernel[ky+2][kx+2] * val_g;
          sum_b += kernel[ky+2][kx+2] * val_b;
        }
      }
      // For this pixel in the new image, set the gray value
      // based on the sum from the kernel
      img.pixels[y*img.width + x] = color(sum_r, sum_g, sum_b);
    }
  }
}


{% endcodeblock %}

## Some explanation
代码好像很多，但其实核心只有三部分：

### 一个简单的粒子类 Particle
    Particle类简单写了一个，和一般的物理库应该比较类似。保存当前粒子的位置和速度，以及画弧的进度。当弧画完以后，重新生成一条待画弧（边数和转向）。

### 控制Particle对象运动的逻辑
Particle的轨迹由很多段弧组成。每帧我们会将粒子的运动方向改变一点点（turn方法），然后前进一点点。turn方法的关键是实现转向的这两行（回忆一下三角函数的和差化积吧）：
{% codeblock lang:java %}
    vx = pvx * cos(rot) - pvy * sin(rot);
    vy = pvx * sin(rot) + pvy * cos(rot);
{% endcodeblock %}

### 构图的逻辑
当某两个粒子之间的距离小于某个设定阈值的时候，就在他们当前位置间连一条直线。

That's it.

新的知识点很少：

    *   Sketch之中，可以自定义类。我们每新建一个Processing的Sketch（假设叫skt.pde）时，其实都是新建了一个类，名为skt。我们可以在skt里面自定义任意的类（在这里就是Particle，会被Processing视作java里面的内部类），所以他们能看到所谓“全局变量”。
    *   keyPressed()函数用于接收用户输入。key变量就是触发keyPressed函数的键的代号。

这应该是半年来我看到的最具备美感的sketch了。你同意否？
