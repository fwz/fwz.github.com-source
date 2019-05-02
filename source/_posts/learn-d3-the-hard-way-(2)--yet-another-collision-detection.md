title: Learn d3 the Hard Way (2)--yet another collision detection
date: 2013-09-16 13:37:37
tags: [Visualization, D3]
categories: [Engineering, Visualization]

---

![](https://wenzhong-1259152588.cos.ap-beijing.myqcloud.com/learnd3_2.png)
Another collision detection work. Very elegant.

Here's a little bit background about this sketch. As we can see, this sketch is composed by non-touching circles in a canvas. This algorithm generate a random sample by generating K candidates, and choose the best one and add it to the system. The definition of best is that it's the farthest away from the previous sample. By this way, the emergance of new sample would be more natural, compared with the uniform random sampling.

Some people ask, "is this useful? I can't see how I can use it into my work." They are probably wrong, an English artist use exactly the same effect in his work [Silent Buddha](http://www.behance.net/gallery/Silent-Buddha-using-Mitchells-Best-Candidate-algorithm/10360457). It's a really impressive work.

Now take a look from the inside.
<!-- more -->
## Sketch
{% raw %}

<!DOCTYPE html>
<html>
<meta charset="utf-8">
<style>

circle {
  fill: #000;
  stroke: #fff;
  stroke-width: 1.5px;
}

#body_yet_another_collsion {
    width: 600px;
    height : 300px;
}

#svg_yac{
  margin : 5px auto;
  display : block;
}
</style>

<div id="body_yet_another_collision"> </div>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js"></script>
<script>

var maxRadius = 32, // maximum radius of circle
    padding = 1, // padding between circles; also minimum radius
    margin = {top: -maxRadius, right: -maxRadius, bottom: -maxRadius, left: -maxRadius},
    width = 600 - margin.left - margin.right,
    height = 300 - margin.top - margin.bottom;

var k = 1 // initial number of candidates to consider per circle
    m = 10, // initial number of circles to add per frame
    n = 1000, // remaining number of circles to add
    newCircle = bestCircleGenerator(maxRadius, padding);

var svg = d3.select("#body_yet_another_collision").append("svg")
    .attr("id", "svg_yac")
    .attr("width", width)
    .attr("height", height)
  .append("g");

d3.timer(function() {
  for (var i = 0; i < m && --n >= 0; ++i) {
    var circle = newCircle(k);

    svg.append("circle")
        .attr("cx", circle[0])
        .attr("cy", circle[1])
        .attr("r", 0)
      .transition()
        .attr("r", circle[2]);

    // As we add more circles, generate more candidates per circle.
    // Since this takes more effort, gradually reduce circles per frame.
    if (k < 500) k *= 1.01, m *= .998;
  }
  return !n;
});

function bestCircleGenerator(maxRadius, padding) {
  var quadtree = d3.geom.quadtree().extent([[0, 0], [width, height]])([]),
      searchRadius = maxRadius * 2,
      maxRadius2 = maxRadius * maxRadius;

  return function(k) {
    var bestX, bestY, bestDistance = 0;

    for (var i = 0; i < k || bestDistance < padding; ++i) {
      var x = Math.random() * width,
          y = Math.random() * height,
          rx1 = x - searchRadius,
          rx2 = x + searchRadius,
          ry1 = y - searchRadius,
          ry2 = y + searchRadius,
          minDistance = maxRadius; // minimum distance for this candidate

      quadtree.visit(function(quad, x1, y1, x2, y2) {
        if (p = quad.point) {
          var p,
              dx = x - p[0],
              dy = y - p[1],
              d2 = dx * dx + dy * dy,
              r2 = p[2] * p[2];
          if (d2 < r2) return minDistance = 0, true; // within a circle
          var d = Math.sqrt(d2) - p[2];
          if (d < minDistance) minDistance = d;
        }
        return !minDistance || x1 > rx2 || x2 < rx1 || y1 > ry2 || y2 < ry1; // or outside search radius
      });

      if (minDistance > bestDistance) bestX = x, bestY = y, bestDistance = minDistance;
    }

    var best = [bestX, bestY, bestDistance - padding];
    quadtree.add(best);
    return best;
  };
}

</script>
</html>

{% endraw %}

If you read the [previous post about collision detection](/2013/09/15/learn-d3-the-hard-way-1----collision-detection/), you can safely read from setup timer section. If not, and you do not have much experience about d3, you can spend maybe 5 - 10 mins on that post to have a basic sense about the following things.

### Initialization
{% codeblock %}
<!DOCTYPE html>
<html>
<meta charset="utf-8">
<style>

circle {
  fill: #000;
  stroke: #fff;
  stroke-width: 1.5px;
}

</style>

<div id="body_yet_another_collision"> </div>
<script src="/js/d3.min.js"></script>
<script>

var maxRadius = 32, // maximum radius of circle
    padding = 1, // padding between circles; also minimum radius
    margin = {top: -maxRadius, right: -maxRadius, bottom: -maxRadius, left: -maxRadius},
    width = 600 - margin.left - margin.right,
    height = 300 - margin.top - margin.bottom;

var k = 1 // initial number of candidates to consider per circle
    m = 10, // initial number of circles to add per frame
    n = 2000, // remaining number of circles to add
    newCircle = bestCircleGenerator(maxRadius, padding);

var svg = d3.select("#body_yet_another_collision").append("svg")
    .attr("id", "svg_yac")
    .attr("width", width)
    .attr("height", height)
  .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

{% endcodeblock %}

### Setup timer
{% codeblock %}

d3.timer(function() {
  for (var i = 0; i < m && --n >= 0; ++i) {
    var circle = newCircle(k);

    svg.append("circle")
        .attr("cx", circle[0])
        .attr("cy", circle[1])
        .attr("r", 0)
      .transition()
        .attr("r", circle[2]);

    // As we add more circles, generate more candidates per circle.
    // Since this takes more effort, gradually reduce circles per frame.
    if (k < 500) k *= 1.01, m *= .998;
  }
  return !n;
});
{% endcodeblock %}

The `timer` function is not like the `tick` function. `tick` will be trigger forever, but timer will stop. [timer](https://github.com/mbostock/d3/wiki/Transitions#wiki-d3_timer) receive a function, and then invoking that function until it return true.

There are also a new method we have not seen before [transition](https://github.com/mbostock/d3/wiki/Selections#wiki-transition). it animate change smoothly over time rather than applying instantaneously.
The `r` attribute in a `circle` will be transited from `0` to a given value `circle[2]`.


So here's what it has done in this fucntion. 

1.  create a new circle. this circle is selected from `k` candidates.
2.  add it to the svg container 
3.  do step 1&2 `m` time for each timer
4.  if have add enough circle, then stop the timer.
 
### Circle Generator
As you might already noticed, the secret lies in the `newCircle(k)` method.

    newCircle = bestCircleGenerator(maxRadius, padding);

What? Why we need to pass a `k`? 
All right, take a close look of the `bestCircleGenerator`.

`bestCircleGenerator` return an anonymous function, which receive an integer `k` as number of candidates. And look at the anonymous function, we can see that the function do mainly 3 things.

1.  create k candidates and select the best one.
2.  add it to the quadtree
3.  return the coordinate and the radius

Then in the timer, we can add this candidate to svg and present it.


{% codeblock %}
function bestCircleGenerator(maxRadius, padding) {
  var quadtree = d3.geom.quadtree().extent([[0, 0], [width, height]])([]),
      searchRadius = maxRadius * 2,
      maxRadius2 = maxRadius * maxRadius;

  return function(k) {
    var bestX, bestY, bestDistance = 0;

    for (var i = 0; i < k || bestDistance < padding; ++i) {
      var x = Math.random() * width,
          y = Math.random() * height,
          rx1 = x - searchRadius,
          rx2 = x + searchRadius,
          ry1 = y - searchRadius,
          ry2 = y + searchRadius,
          minDistance = maxRadius; // minimum distance for this candidate

      quadtree.visit(function(quad, x1, y1, x2, y2) {
        if (p = quad.point) {
          var p,
              dx = x - p[0],
              dy = y - p[1],
              d2 = dx * dx + dy * dy,
              r2 = p[2] * p[2];
          if (d2 < r2) return minDistance = 0, true; // within a circle
          var d = Math.sqrt(d2) - p[2];
          if (d < minDistance) minDistance = d;
        }
        return !minDistance || x1 > rx2 || x2 < rx1 || y1 > ry2 || y2 < ry1; // or outside search radius
      });

      if (minDistance > bestDistance) bestX = x, bestY = y, bestDistance = minDistance;
    }

    var best = [bestX, bestY, bestDistance - padding];
    quadtree.add(best);
    return best;
  };
}

{% endcodeblock %}

When firstly `bestCircleGenerator` is called, an empty Quadtree is generated, which will cover the whole svg container via the [extent](https://github.com/mbostock/d3/wiki/Quadtree-Geom#wiki-extent) method. And it also setup an searching range.

Again, it need some effort to understand the [quadtree.visit](https://github.com/mbostock/d3/wiki/Quadtree-Geom#wiki-visit). Here, it will visit nodes in this quadtree.But because quadtree have some feature that we can reduce the depth of visit, so we need to tell the quadtree whether we want to continue our visit to deeper node. the anonymous fucntion here, return a boolean value indicate whether we need to continue our visit. If function return true, then the children of current node won't be visited.

So what have been done inside this function is:
1.  get the position and radius of current node
2.  calculate the whether current node is apart from candidate node
3.  If not, return true and stop visiting this node
4.  If so, update the minimum distance. if the distance is zero and current node is outside of the search radius, also return true and stop visiting this node.
5.  check whether the new distance is the "best" distance
6.  If so, update the "best" distance

And finally, if a new circle is founded, add it to the quadtree, return the position and radius. So in the d3.timer() method, it can be drawn on the svg container.

That's all for this simple and pretty sketch.
