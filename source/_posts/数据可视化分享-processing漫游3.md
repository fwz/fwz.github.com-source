title: "数据可视化分享——Processing漫游(3)"
id: 284
date: 2012-11-21 18:02:39
tags: 
- Processing
categories: 
- Engineering
- Visualization
---

今天突然想写一个小游戏，按捺不住就做了个Processing的版本。

游戏规则：两个或以上相连的同色棋子，点击后可以消去，增加消去棋子个数**平方**的分数。尽可能获取最高分数。 

各路大神来贴分吧!

<!--more-->
## Sketch
{% raw %}
<script src="https://wenzhong-1259152588.cos.ap-beijing.myqcloud.com/js/processing.min.js"></script>
<script type="text/processing" data-processing-target="processing-canvas">
// Stone board config
int row = 10;
int column = 12;
int grid_size = 50;
int margin = 4;
int draw_mode = 1;
int color_scheme = 0;

// board status
Stone[][] stones;    // stone info
int [] bar_height = new int[column];    // current height of each column
boolean[][] toclear; // whether this grid is mark as same color from trigger point.

int display_score;
int score;
int clear_num;

// Color and Image
color[][] clrs = new color[][] {
  { #F60018, #FF9C00, #0E53A7, #25D500, #FFFFFF}, //red, orange, blue, green,background,
  { #6666CC, #6633CC, #990033, #CC3333, #FFFFFF}, //trash,...,

};

// Four direction to search
int [][] dir = new int[][] {
  {0, 1}, {0, -1},
  {1, 0}, {-1, 0}
};

void setup() {
  size(column*grid_size, row*grid_size);
  reset();
}

void reset() {
  stones = new Stone[column][row];  //so we can use stone[i][j] for co-ordinate(i,j)
  toclear = new boolean[column][row];
  score = 0;
  display_score = 0;
  ellipseMode(CORNER);

  for (int i=0; i<column; i++) {
    bar_height[i] = row;
    for (int j=0; j<row; j++) {
      int clr_ind = int(random(0, clrs[color_scheme].length - 1));
      stones[i][j] = new Stone(i, j, clr_ind, grid_size);
      toclear[i][j] = false;
    }
  }

  //PFont font;
  //font = loadFont("HelveticaNeue-CondensedBold-192.vlw");
  //textFont(font,192);

  colorMode(RGB, 255);
  frameRate(20);

}

void draw() {
  background(clrs[color_scheme][clrs[color_scheme].length-1]);

  fill(160, map(score, -100,2000, 0, 255));
  textAlign(CENTER);
  textSize(192);

  if (score - display_score > 12) {
    display_score += 12;
  }
  else if (score-display_score > 0) {
    display_score += 1;
  }
  text(str(display_score), width/2, height/2);

  for (int i=0; i<column; i++) {
    for (int j=0; j<row; j++) {
      stones[i][j].draw(draw_mode, color_scheme);
    }
  }

}

// Every Click
void mouseClicked() {


  int x = (int)(mouseX / grid_size);
  int y = (int)((height - (int)mouseY) / grid_size); // in Processing JS, please intefy float.
  //println ((str(x) + "," + str(y)));

  //2 or more stone should be removed
  if (!valid_clear(x,y)) {
    return;
  }

  for (int i = 0 ; i < column; i++) {
    for (int j = 0 ; j < row; j++) {
      toclear[i][j] = false;
    }
  }

  //search all linked stone with the same color, using stupid DFS/BFS
  //update toclear matrix

  clear_num = 1;
  stones[x][y].enable = false;
  toclear[x][y] = true;

  //DFS
  find_neighbor(x,y, stones[x][y].clr_index);

  //update stone still exist by column
  for (int i=0; i<column; i++) {
    // update each column
    Stone[] temp = new Stone[row];
    int dec = 0;

    for (int j=0; j < bar_height[i]; j++) {
      temp[j] = new Stone(stones[i][j]);
      if (toclear[i][j])
        dec++;

      else {
        int down_step = temp[j].calc_down_step();

        if ((down_step) > 0) {
          stones[i][j - down_step].clr_index = temp[j].clr_index;
          stones[i][j].enable = false;
          stones[i][j - down_step].enable = true;
        }
      }
    }

    bar_height[i] -= dec;
  }

  score += clear_num * clear_num;

  //eliminate gaps
  for (int i = 0; i < column - 1; i++) {
    while (bar_height[i] == 0) {
      //System.out.println("merge");
      merge_column(i);
    }
  }
}

boolean valid_clear(int x, int y) {
  if (!stones[x][y].enable) {return false;}

  for (int i = 0; i < 4; i++) {
      int xn = x + dir[i][0];
      int yn = y + dir[i][1];

      if (xn >= 0 && xn < column && yn >= 0 && yn < row &&
          stones[xn][yn].enable && stones[x][y].clr_index == stones[xn][yn].clr_index) {
          return true;
      }
  }
  return false;
}

void merge_column(int empty_column) {
  for (int i = empty_column; i < column-1; i++ ) {
    bar_height[i] = bar_height[i+1];
    for (int j = 0; j < bar_height[i]; j++) {
      stones[i][j].y = stones[i+1][j].y;
      stones[i][j].clr_index = stones[i+1][j].clr_index;
      stones[i][j].enable = stones[i+1][j].enable;
      stones[i+1][j].enable = false;
    }
  }
}

void find_neighbor(int x, int y,  color clr) {
  for (int i = 0; i < 4; i++) {
      int xn = x + dir[i][0];
      int yn = y + dir[i][1];

      if (xn >= 0 && xn < column && yn >= 0 && yn < row &&
        stones[xn][yn].enable && toclear[xn][yn] == false && stones[xn][yn].clr_index == clr) {
        clear_num++;
        toclear[xn][yn] = true;
        stones[xn][yn].enable = false;

        find_neighbor(xn, yn, clr);
      }
    }
}

void keyPressed() {
  if (key == 'r') {
    reset();
  }
  if (key == 's') {
    //draw_mode += 1;
    //draw_mode %= 2;
  }
  if (key == 'c') {
    color_scheme += 1;
    color_scheme %= clrs.length;
  }
}

class Stone {
  public int x;
  public int y;
  public int clr_index;
  public boolean enable = true;
  private float grid_size;

  public Stone(int x, int y, int clr_index, int grid_size) {
    this.x = x;
    this.y = y;
    this.clr_index = clr_index;
    this.grid_size = grid_size;
  }

  public Stone(Stone s) {
    this.x = s.x;
    this.y = s.y;
    this.clr_index = s.clr_index;
    this.grid_size = s.grid_size;
    this.enable = s.enable;
  }

  public void disable () {
    enable = false;
  }

  public void draw(int draw_mode, int color_scheme) {
    if (enable) {
      noStroke();
      fill(clrs[color_scheme][clr_index], 200);
      ellipse(x * grid_size, (row - y - 1) * grid_size, grid_size - margin, grid_size - margin);
    }
  }

  public int calc_down_step () {
    int down_step = 0;

    // for stone not being clear
    // use toclear matrix to calculate how many step go get down;
    for (int i = 0 ; i < y; i++) {
      if (toclear[x][i]) {
        down_step++;
      }
    }
    return down_step;
  }
}
</script>

<canvas id="processing-canvas"> </canvas>
{% endraw %}


