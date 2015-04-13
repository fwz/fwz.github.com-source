title: Learn d3 the Hard Way (1) -- collision detection
date: 2013-09-15 23:36:19
tags: [Visualization, D3]
categories: [Engineering, Visualization]
---

![](http://wenzhong.qiniudn.com/learnd3_1.png)
Finally, make up my mind to learn d3 with focus. I decide to learn the example on [d3's gallery](mbostock.github.io/d3/) line-by-line. This should not be a hard way as the title of this post because:

*   Most example written by author.
*   the [API doc](https://github.com/mbostock/d3/wiki/) is perfect. Yes, perfect.
*   You can choose the your favourite effect to dive in, never lose interest.


# Javascript Basic

If you do not have any javascript experience, try to take some. As I always recommend, the [javascript course](http://www.codeschool.com/courses/javascript-road-trip-part-1) in Codeschool. If you have time, take the [Try jQuery](http://try.jquery.com/) course also.


Also, we might want to try write some js but I hate write in a file and refresh the browser to see whether I make it correctly. If it's the case to you, then you can try [JSFiddle](http://jsfiddle.net/). you can type js and run to see output all in browser. What's more, JSFiddle provide most popular js library (such as D3 and Processing.js) so you don't need to care about this when you are prototyping.

OK, we are ready to go! 
# [Collision](http://mbostock.github.io/d3/talk/20111018/collision.html)

<!-- more -->
## Sketch
{% raw %}
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html;charset=utf-8"/>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js"></script>
    <style type="text/css">

circle {
  stroke: #000;
  stroke-opacity: .5;
}

    </style>
  </head>
  <body>
    <div id="body"> </div>
    <script type="text/javascript">

var w = 600,
    h = 400;

var nodes = d3.range(200).map(function() { return {radius: Math.random() * 12 + 4}; }),
    color = d3.scale.category10();

var force = d3.layout.force()
    .gravity(0.05)
    .charge(function(d, i) { return i ? 0 : -2000; })
    .nodes(nodes)
    .size([w, h]);

var root = nodes[0];
root.radius = 0;
root.fixed = true;

force.start();

var svg = d3.select("#body").append("svg:svg")
    .attr("width", w)
    .attr("height", h);

svg.selectAll("circle")
    .data(nodes.slice(1))
  .enter().append("svg:circle")
    .attr("r", function(d) { return d.radius - 2; })
    .style("fill", function(d, i) { return color(i % 5); });

force.on("tick", function(e) {
  var q = d3.geom.quadtree(nodes),
      i = 0,
      n = nodes.length;

  while (++i < n) {
    q.visit(collide(nodes[i]));
  }

  svg.selectAll("circle")
      .attr("cx", function(d) { return d.x; })
      .attr("cy", function(d) { return d.y; });
});

svg.on("mousemove", function() {
  // var p1 = d3.svg.mouse(this);
  var p1 = d3.mouse(this);
  root.px = p1[0];
  root.py = p1[1];
  force.resume();
});

function collide(node) {
  var r = node.radius + 16,
      nx1 = node.x - r,
      nx2 = node.x + r,
      ny1 = node.y - r,
      ny2 = node.y + r;
  return function(quad, x1, y1, x2, y2) {
    if (quad.point && (quad.point !== node)) {
      var x = node.x - quad.point.x,
          y = node.y - quad.point.y,
          l = Math.sqrt(x * x + y * y),
          r = node.radius + quad.point.radius;
      if (l < r) {
        l = (l - r) / l * .5;
        node.x -= x *= l;
        node.y -= y *= l;
        quad.point.x += x;
        quad.point.y += y;
      }
    }
    return x1 > nx2
        || x2 < nx1
        || y1 > ny2
        || y2 < ny1;
  };
}

    </script>
  </body>
</html>
{% endraw %}


## Code  

### Layout and Create data
{% codeblock %}
var w = 600,
    h = 400;
{% endcodeblock %}
Define the width and height of the layout, much similar to Processing.

{% codeblock %}
var nodes = d3.range(200).map(function() { return {radius: Math.random() * 12 + 4}; }),
    color = d3.scale.category10();
{% endcodeblock %}

[range()](https://github.com/mbostock/d3/wiki/Arrays#wiki-d3_range) is just like it in python, so `d3.range(200)` would create an array with 200 members.
[array.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FGlobal_Objects%2FArray%2Fmap) would receive a function, which here is an anonymous function, will return a dict, which only have one key "radius" in it (which indicate the size of the node), and the value is a randomized number.

[d3.scale.category10()](https://github.com/mbostock/d3/wiki/Ordinal-Scales#wiki-category10) is going to create a color list.

So we can think of the above code do some initialization for the viz.

## Setup the force system
{% codeblock %}
var force = d3.layout.force()
    .gravity(0.05)
    .charge(function(d, i) { return i ? 0 : -2000; })
    .nodes(nodes)
    .size([w, h]);

{% endcodeblock %}
[d3.layout.force()](https://github.com/mbostock/d3/wiki/Force-Layout#wiki-force) is a physical system, which we can manipulate object inside it by defining physical principle.

[gravity()](https://github.com/mbostock/d3/wiki/Force-Layout#wiki-gravity) may be misleading, but what this method do is to setup a constrain to ensure that no nodes will escape from the system.

[charge()](https://github.com/mbostock/d3/wiki/Force-Layout#wiki-charge) I don't know what's that, at first I suppose it's liek LinkStrength, but not.

So the previous code setup an new force layout.

{% codeblock %}
var root = nodes[0];
root.radius = 0;
root.fixed = true;

force.start();

{% endcodeblock %}

Here, select a fix node and start to run the layout (This node is used for user interaction, which controlled by user mouse, we will see it later). After the layout run, then the system manipulate the object itself.

### Create an SVG container to present the graph with data
{% codeblock %}
var svg = d3.select("#body").append("svg:svg")
    .attr("width", w)
    .attr("height", h);

svg.selectAll("circle")
    .data(nodes.slice(1))
  .enter().append("svg:circle")
    .attr("r", function(d) { return d.radius - 2; })
    .style("fill", function(d, i) { return color(i % 3); });
{% endcodeblock %}

now we start to modify the DOM. Just like jQuery, we select the `<body>` node and append an `<svg>` node in it, and assign height and width attributes.

And then, it use the "circle" selector to select all element whose class == "circle", which here is an empty selection. Then the [selection.data()](https://github.com/mbostock/d3/wiki/Selections#wiki-data) method "Joins the specified array of data with the current selection." 

`nodes.slice(1)` will remove the first node from `nodes` (which is selected as root)

the [enter()](https://github.com/mbostock/d3/wiki/Selections#wiki-enter) function is the one worth understand when learning d3 and its philosophy. It returns the entering selection: placeholder nodes for each data element for which no corresponding existing DOM element was found in the current selection. In our case, all data is new and they do not in the DOM yet, so we create placeholder nodes for all nodes except the root node.

Note that the enter operator merely returns a reference to the entering selection, and it is up to you to add the new nodes. So the [append()](https://github.com/mbostock/d3/wiki/Selections#wiki-append) create a "circle" element in the SVG namespace.

Then for each element, we assign it a "r" attribute via a anonymous function, indicating the radius of this node. Remember, d3 use `d` to indicate the datum. So here 

{% codeblock %}
 function(d) { return d.radius - 2; } 
{% endcodeblock %}
will assign current circle an `r` attribute with the radius vaule of current datum.

Finally we add some color to this circle by applying attribute using `style()`.

### Collision detection
{% codeblock %}
force.on("tick", function(e) {
  var q = d3.geom.quadtree(nodes),
      i = 0,
      n = nodes.length;

  while (++i < n) {
    q.visit(collide(nodes[i]));
  }

  svg.selectAll("circle")
      .attr("cx", function(d) { return d.x; })
      .attr("cy", function(d) { return d.y; });
});
{% endcodeblock %}

Here is an event handler, the function inside it will be triggered when tick event is happened.

Firstly, it create a [quadtree](http://bl.ocks.org/mbostock/4343214) object, using the coordinate of the nodes. Then the quadtree call the [visit()](https://github.com/mbostock/d3/wiki/Quadtree-Geom#wiki-visit) to check whether it have collision. We will check the `collide` method later, but from the code suggests, it change the (x,y) coordinate so we need to update the `cx` and `cy` attribute accordingly. 

{% codeblock %}
svg.on("mousemove", function() {
  // var p1 = d3.svg.mouse(this);  this line of code will report error
  var p1 = d3.mouse(this);
  root.px = p1[0];
  root.py = p1[1];
  force.resume();
});
{% endcodeblock %}

This code transit the mouse position as the fixed root node. when the mouse moves, the layout redefine the root position and recalculate the layout.

**Note**: if you are using d3 3+, then you might encounter an error that 
`var p1 = d3.svg.mouse(this);` is reporting error. It is caused by the d3 upgrade (non forward competiable). Check the latest API spec and use `d3.mouse(this)` instead.

{% codeblock %}
function collide(node) {
  var r = node.radius + 16,
      nx1 = node.x - r,
      nx2 = node.x + r,
      ny1 = node.y - r,
      ny2 = node.y + r;
  return function(quad, x1, y1, x2, y2) {
    if (quad.point && (quad.point !== node)) {
      var x = node.x - quad.point.x,
          y = node.y - quad.point.y,
          l = Math.sqrt(x * x + y * y),
          r = node.radius + quad.point.radius;
      if (l < r) {
        l = (l - r) / l * .5;
        node.x -= x *= l;
        node.y -= y *= l;
        quad.point.x += x;
        quad.point.y += y;
      }
    }
    return x1 > nx2
        || x2 < nx1
        || y1 > ny2
        || y2 < ny1;
  };
}
{% endcodeblock %}

The previous code calculate a square bounding-box (nx1, nx2, ny1, ny2), which the node can be put inside of it.

This function (quad, x1, x2, y1, y2) is required by the `quad.visit()` function. The function now create compare whether bounding boxes of two nodes overlap by a simple collision detection algorithm.

{% codeblock %}
    return x1 > nx2
        || x2 < nx1
        || y1 > ny2
        || y2 < ny1;
{% endcodeblock %}

The bounding-box collision detection algorithm.

{% codeblock %}
      if (l < r) {
        l = (l - r) / l * .5;
        node.x -= x *= l;
        node.y -= y *= l;
        quad.point.x += x;
        quad.point.y += y;
      }
{% endcodeblock %}

if two nodes are overlaped, adjust them.

## That's All!
To summarize, this viz use the force layout to move nodes, following some physical principle. Then it build a quadtree at every "tick" event of the layout, and detect collision. When collision is detected, then it will adjust the node immediately.

## What's Next? 
I am going to study this block -- [Mitchell’s Best-Candidate](http://bl.ocks.org/mbostock/1893974). Also using the Quadtree to build a fasinating effect with limited code.
