<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="utf-8">
		<title>D3: A true bar chart with SVG rects</title>
      <script type="text/javascript" src="http://d3js.org/d3.v3.min.js"></script>
		<style type="text/css">
			/* No style rules here yet */		
		</style>
	</head>
  
	<body>
		<script type="text/javascript">

			var w = 500;
			var h = 500;
			var barPadding = 3;
			
			var dataset = [3, 20, 5, 19, 34, 18, 23, 14];
          
          
			//Create SVG element - this is the whole plot
			var svg = d3.select("body")
						.append("svg")
						.attr("width", w)
						.attr("height", h);
          
          svg.selectAll("rect")
            .data(dataset)
            .enter()
            .append("rect")
            .attr("y", function(d, i){return i*35})
            .attr("x", 0)
            .attr("width", function(i){return i*10})
            .attr("height", 30)
            .attr("fill", "teal")
            .transition()
            .duration(2000)
            .attr("height",15)
            .transition()
            .duration(2000)
            .delay(2000)
            .attr("x", function(d, i){return i*33})
            .attr("y", 0)
            .attr("height", function(i){return i*10})
            .attr("width", 30)
            .attr("fill", "black")
          

            
		</script>
	</body>
</html>
