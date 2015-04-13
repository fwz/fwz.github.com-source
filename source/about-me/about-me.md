tatle: About Me
date: 2012-04-07 20:38:01
---

{% raw %}
<script src="https://cdnjs.cloudflare.com/ajax/libs/processing.js/1.4.8/processing.min.js"></script>
<script type="application/processing" data-processing-target="canvas">
//Info: http://processingjs.org/reference
/* @pjs preload="wenzhong.jpg"; */
  mage img, sample;
int res, n, d, preX;

// list of resolutions
int[] rlist = { 
  25, 50, 100 
};


void setup() {

  size(400, 400);
  img = loadImage("wenzhong.jpg");
  nextRes();
}


void nextRes() {

  // resample the image
  res = (res + 1) % rlist.length;
  n = rlist[res];
  sample = img.get();
  sample.resize(n, n);
  d = width/n;
  update(); 
}

void update() {
    int m = constrain(130 * mouseX / width, 1, 129);
    preX = mouseX;

    loadPixels();

    // iterate over all blocks
    for (int x = 0; x < n; x++) {
      for (int y = 0; y < n; y++) {

        // sample pixel brightness
        float val = 256 - (sample.pixels[y*n+x] & 255);

        // iterate over all pixels of a block
        for (int dx = 0; dx < d; dx++) {
          for (int dy = 0; dy < d; dy++) {

            // get diagonal coordinate
            int z = val % (2*m) < m ? (dx+dy+1) * 255 / d : (dx+d-dy) * 255 / d;

            // black and white dithering
            pixels[ (y*d + dy) * width + (x*d + dx) ] = val > 2 * abs(z - 255) + 2 ? #2F84EA : #ffffff;
          }
        }
      }
    }

    updatePixels();
}

void draw() {
  if (preX != mouseX) { 
    update();
  }
}


void keyPressed() {
  nextRes();
}

</script>
<!--canvas id="canvas" style="display: block; margin:0 auto 0 0;"></canvas-->
{% endraw %}

|                              | |                                                                        |
| ---                          |-|---                                                                    |
| **Career **                  |@|Yahoo data engineer                                                     |
| **favourite activity**                    |@|piano, drum set, swimming, war3, frisbee             |
| **Social Activity **         |@|BUPT TIC ex-chairman / Khan Academy Chinese interpreter |
| **Zhihu**                    |@|[文中](http://www.zhihu.com/people/wen-zhong)                                                                 |
| **Linkedin** |@| [Linkedin](https://www.linkedin.com/profile/view?id=63633491&trk=spm_pic)|


# About this Blog
这是Wenzhong的博客，不定期更新中文生活文章和英语技术文章。
This is a web log contain everything about my work and life. Some technical articles are written in English for a more wide-ranged to communicate with world-wide Programmers. Some other articles related to my daily life are written in Chinese. Update irregularly.

# My Books
[敏捷数据科学](http://book.douban.com/subject/25929433/) (Agile data science: building data analytic application with Hadoop)中译版

# My Musics
[SoundCloud](https://soundcloud.com/wenzhong-1/)
