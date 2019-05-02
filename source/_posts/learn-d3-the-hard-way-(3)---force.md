title: Learn d3 the Hard Way (3)-- Force
date: 2013-09-20 21:05:05
layout: landscape
tags: [Visualization, D3]
categories: [Engineering, Visualization]

---

![](https://wenzhong-1259152588.cos.ap-beijing.myqcloud.com/learnd3_3.png)
This is a very useful sketch for node placement. Rather than the original force only move nodes, this sketch also place label in it. Origin from Mortiz's [Force-based label placement](http://bl.ocks.org/MoritzStefaner/1377729).

The basic idea is to have labels orbit around their target node at a fixed distance, but repeal each other, so that they don't overlap, and orient themselves to the outside of clusters. To support that, labels on the right of their target node are left-aligned, and labels on the left of their target node are right-aligned; in between, we interpolate. In this example, one force layout governs the node placement, and the second one the label placement, but of course, the node placement could be computed by any other algorithm.

<!--more-->
## Sketch 
{% raw %}

<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Force based label placement</title>
        <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js"></script>
    </head>
    <div id="div_Force_based_label_placement" />
    <style>
        #svg_Force_based_label_placement{
            margin : 5px auto;
            display : block;
    </style>

<script type="text/javascript" charset="utf-8">
            var w = 700, h = 450;

            var labelDistance = 0;

            var vis = d3.select("#div_Force_based_label_placement").append("svg:svg").attr("width", w).attr("height", h).attr("id", "svg_Force_based_label_placement");

            var nodes = [];
            var labelAnchors = [];
            var labelAnchorLinks = [];
            var links = [];

            for(var i = 0; i < 60; i++) {
                var node = {
                    label : "node " + i
                };
                nodes.push(node);
                labelAnchors.push({
                    node : node
                });
                labelAnchors.push({
                    node : node
                });
            };

            for(var i = 0; i < nodes.length; i++) {
                for(var j = 0; j < i; j++) {
                    if(Math.random() > .95)
                        links.push({
                            source : i,
                            target : j,
                            weight : Math.random()
                        });
                }
                labelAnchorLinks.push({
                    source : i * 2,
                    target : i * 2 + 1,
                    weight : 1
                });
            };

            var force = d3.layout.force().size([w, h]).nodes(nodes).links(links).gravity(1.5).linkDistance(20).charge(-1500).linkStrength(function(x) {
                return x.weight * 5
            });


            force.start();

            var force2 = d3.layout.force().nodes(labelAnchors).links(labelAnchorLinks).gravity(0).linkDistance(0).linkStrength(8).charge(-100).size([w, h]);
            force2.start();

            var link = vis.selectAll("line.link").data(links).enter().append("svg:line").attr("class", "link").style("stroke", "#CCC");

            var color = d3.scale.category10();
            var node = vis.selectAll("g.node").data(force.nodes()).enter().append("svg:g").attr("class", "node");
            node.append("svg:circle").attr("r", 5).style("fill", function(d, i) { return color(i % 5); }).style("stroke", "#FFF").style("stroke-width", 3);
            node.call(force.drag);


            var anchorLink = vis.selectAll("line.anchorLink").data(labelAnchorLinks)//.enter().append("svg:line").attr("class", "anchorLink").style("stroke", "#999");

            var anchorNode = vis.selectAll("g.anchorNode").data(force2.nodes()).enter().append("svg:g").attr("class", "anchorNode");
            anchorNode.append("svg:circle").attr("r", 0).style("fill", "#FFF");
                anchorNode.append("svg:text").text(function(d, i) {
                return i % 2 == 0 ? "" : d.node.label
            }).style("fill", "#555").style("font-family", "Arial").style("font-size", 12);

            var updateLink = function() {
                this.attr("x1", function(d) {
                    return d.source.x;
                }).attr("y1", function(d) {
                    return d.source.y;
                }).attr("x2", function(d) {
                    return d.target.x;
                }).attr("y2", function(d) {
                    return d.target.y;
                });

            }

            var updateNode = function() {
                this.attr("transform", function(d) {
                    return "translate(" + d.x + "," + d.y + ")";
                });

            }


            force.on("tick", function() {

                force2.start();

                node.call(updateNode);

                anchorNode.each(function(d, i) {
                    if(i % 2 == 0) {
                        d.x = d.node.x;
                        d.y = d.node.y;
                    } else {
                        var b = this.childNodes[1].getBBox();

                        var diffX = d.x - d.node.x;
                        var diffY = d.y - d.node.y;

                        var dist = Math.sqrt(diffX * diffX + diffY * diffY);

                        var shiftX = b.width * (diffX - dist) / (dist * 2);
                        shiftX = Math.max(-b.width, Math.min(0, shiftX));
                        var shiftY = 5;
                        this.childNodes[1].setAttribute("transform", "translate(" + shiftX + "," + shiftY + ")");
                    }
                });


                anchorNode.call(updateNode);

                link.call(updateLink);
                anchorLink.call(updateLink);

            });

        </script>
    </body>
</html>
{% endraw %}

## Codes

{% codeblock %}
var w = 700, h = 450;

var labelDistance = 0;

var vis = d3.select("#div_Force_based_label_placement").append("svg:svg").attr("width", w).attr("height", h).attr("id", "svg_Force_based_label_placement");

var nodes = [];
var labelAnchors = [];
var labelAnchorLinks = [];
var links = [];

for(var i = 0; i < 60; i++) {
    var node = {
        label : "node " + i
    };
    nodes.push(node);
    labelAnchors.push({
        node : node
    });
    labelAnchors.push({
        node : node
    });
};

for(var i = 0; i < nodes.length; i++) {
    for(var j = 0; j < i; j++) {
        if(Math.random() > .95)
            links.push({
                source : i,
                target : j,
                weight : Math.random()
            });
    }
    labelAnchorLinks.push({
        source : i * 2,
        target : i * 2 + 1,
        weight : 1
    });
};

{% endcodeblock %}

Just like the introduction said, two force is used in this sketch. 

`labelAnchorLinks` is used to maintain the links between every node and it's label.

{% codeblock %}
var force = d3.layout.force().size([w, h]).nodes(nodes).links(links).gravity(1.5).linkDistance(20).charge(-1500).linkStrength(function(x) {
    return x.weight * 5
});

force.start();

var force2 = d3.layout.force().nodes(labelAnchors).links(labelAnchorLinks).gravity(0).linkDistance(0).linkStrength(8).charge(-100).size([w, h]);
force2.start();

var link = vis.selectAll("line.link").data(links).enter().append("svg:line").attr("class", "link").style("stroke", "#CCC");

var color = d3.scale.category10();
var node = vis.selectAll("g.node").data(force.nodes()).enter().append("svg:g").attr("class", "node");
node.append("svg:circle").attr("r", 5).style("fill", function(d, i) { return color(i % 5); }).style("stroke", "#FFF").style("stroke-width", 3);
node.call(force.drag);

{% endcodeblock %}

Then run both forces and draw them by adding svn elements.
`node.call(force.drag)` allow interactive dragging. So you can drag one node in the force layout and see how the whole layout react to the drag. Note: the layout would be resumed when mousemove event is triggered, meaning that the friction will be reset, simulation will be reheated.


{% codeblock %}
var anchorLink = vis.selectAll("line.anchorLink").data(labelAnchorLinks)//.enter().append("svg:line").attr("class", "anchorLink").style("stroke", "#999");

var anchorNode = vis.selectAll("g.anchorNode").data(force2.nodes()).enter().append("svg:g").attr("class", "anchorNode");
anchorNode.append("svg:circle").attr("r", 0).style("fill", "#FFF");

anchorNode.append("svg:text").text(function(d, i) {
    return i % 2 == 0 ? "" : d.node.label
}).style("fill", "#555").style("font-family", "Arial").style("font-size", 12);

var updateLink = function() {
    this.attr("x1", function(d) {
        return d.source.x;
    }).attr("y1", function(d) {
        return d.source.y;
    }).attr("x2", function(d) {
        return d.target.x;
    }).attr("y2", function(d) {
        return d.target.y;
    });

}

var updateNode = function() {
    this.attr("transform", function(d) {
        return "translate(" + d.x + "," + d.y + ")";
    });

}


force.on("tick", function() {

    force2.start();

    node.call(updateNode);

    anchorNode.each(function(d, i) {
        if(i % 2 == 0) {
            d.x = d.node.x;
            d.y = d.node.y;
        } else {
            var b = this.childNodes[1].getBBox();

            var diffX = d.x - d.node.x;
            var diffY = d.y - d.node.y;

            var dist = Math.sqrt(diffX * diffX + diffY * diffY);

            var shiftX = b.width * (diffX - dist) / (dist * 2);
            shiftX = Math.max(-b.width, Math.min(0, shiftX));
            var shiftY = 5;
            this.childNodes[1].setAttribute("transform", "translate(" + shiftX + "," + shiftY + ")");
        }
    });


    anchorNode.call(updateNode);

    link.call(updateLink);
    anchorLink.call(updateLink);

});

{% endcodeblock %}
