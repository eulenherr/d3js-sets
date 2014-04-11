# Manipulating Sets of Elements using D3.js

I recently had a closer look at the [D3.js examples](https://github.com/mbostock/d3/wiki/Gallery). Unfortunately, none of them applies the core D3 functionality (changing elements when data changes with enter(), update() & exit()) to sets of elements. Say I want to visualize something like the [bubble chart](http://bl.ocks.org/4063269), but I want to bind it to data, updating all elements that correspond to one datum (the colored circle and the text) when the datum changes. The current examples only render everything once but does not support updating or exiting elements.

The bubble chart example is static, it only has to react to entering elements. Updating and exiting elements don’t matter. So updating and exiting is what we have to implement additionally in our example to really manipulate multiple elements for each datum.

Let’s assume we just want to display and update a simple bar chart:

	var svg = d3.select("div").append("svg")
	   .attr("width", 800)
	   .attr("height", 220);

	var data = [0.2,0.4,0.3,0.6];
	var node;
	var scale = d3.scale.linear()
		.domain([0,1])
		.range([0,800]);
	var color = d3.scale.category20c();

	// functions to calculate y positions
	var y_rect = function(d,i) { return i*40+16; };
	var y_text = function(d,i) { return i*40+30; };

	function update(data)
	{
		// initial select, like all D3 examples, bind it to data
		node = svg.selectAll(".node")
		.data(data, String)

		// update changing elements with new percentage and bar width
		node.each(function(d, i) {
			single = d3.select(this)
			single.selectAll("rect")
				.transition()
				.attr("width", scale)
				.attr("y", y_rect(d,i))
				.duration(300)

			single.selectAll("text")
				.transition()
				.attr("y", y_text(d,i))
				.text(function() { return Math.round(d*100) + "%"; })
				.duration(300)
		})	

		// Create entering nodes with the bar (<rect>) and the percentage (<text>)
		// grouped in a <g class="node"> element
		var enter = node.enter().append("g")
			.attr("class", "node")

		enter.append("rect")
			.attr("width", scale)
			.attr("height", 20)
			.attr("x", 20)
			.attr("y", y_rect)
			.style("fill", color)
			.attr("rx", "5px")
			.attr("ry", "5px")

		enter.append("text")
			.text(function(d,i) { return Math.round(d*100) + "%"; })
			.style("fill", "#000")
			.attr("x", function(d,i) { return scale(d)-8; })
			.attr("y", y_text)
			.style("text-anchor", "right")

		// Add a fade in transition to all entering elements and child elements
		enter.selectAll("*")
			.style("opacity", 0)
			.transition()
			.duration(300)
			.style("opacity", 1)

		// fade out and remove all exiting elements
		node.exit()
			.style("opacity", 1)
			.transition()
			.duration(300)
			.style("opacity", 0)
			.remove()
	}

	update(data);

You can try changing the data via the buttons in the example.html.

Using the selection features of D3, it isn’t hard to have one datum correspond to multiple rendered elements, but it’s something that’s missing in the examples so far.