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
		
		<script> 
		  //a scalable horizontal bar chart without all the fuss:
		  
		  var data= [8,7,6,5,3,4,5,5,6,3,4,2,4, 9];  
		  
		  var max = d3.max(data)
		  
		  // set horizontal and vertical bars:
		  var w = 1000
		  var h = 500
		  
		  var svg = d3.select("body")
		              .append("svg")
		              .attr("width", w)
		              .attr("height", h)
		              
		
		      svg.selectAll("rect")
		      .data(data)
		      //.data(data.sort(d3.descending))
		      .enter() //for every element in data do the following:
		      .append("rect")
		      .attr("y", function(d, i){return (h / data.length)*i})
		      .attr("width", 0)
		      .transition()
		      .duration(3000)
		      .attr("height", h / data.length - 10)
		      .attr("width", function(i) {return i*(w-400)/max})
		      .attr("fill", "teal")
		      
		      svg.selectAll("text")
		      .data(data)
		      .enter() //for every element in data do the following:
		      .append("text")
		      .text(function(i){return i})
		      .attr("y", function(d, i){return (h /data.length)*(i+.6)})
		      .attr("fill", "grey")
		      .attr("x", 1000)
		      .attr("font-size", 250/data.length)
		      .transition()
		      .duration(4000)
		      .attr("x", function(i) {return i*(w-380)/max})
		
		  
		</script>
		
		
		<script src="//d3js.org/d3.v3.min.js" \\the d3 script reference></script> 
 <p>Click me.</p>

<script> 

  var data = [ 5, 10, 13, 19, 21, 25, 22, 18, 15, 13,
                11, 12, 15, 20, 18, 17, 16, 18, 23, 25 ];  
  var max = d3.max(data)
  
  // set horizontal and vertical bars:
  var w = 1000
  var h = 500
  
  var svg = d3.select("body")
              .append("svg")
              .attr("width", w)
              .attr("height", h)
  

              

      svg.selectAll("rect")
      .data(data)
      //.data(data.sort(d3.descending))
      .enter() //for every element in data do the following:
      .append("rect")
      .attr("y", function(d, i){return (h / data.length)*i})
      .attr("height", h / data.length - 5)
      .attr("width", function(i) {return i*(w-400)/max})
      
      
      
 
               
  d3.select("p")
    .on("click", function() {
    
    var numValues = data.length;               //Count original length of dataset
    data = [];                                       //Initialize empty array
for (var i = 0; i < numValues; i++) {               //Loop numValues times
    var newNumber = Math.floor(Math.random()*25+4); //New random integer (0-24)
    data.push(newNumber);                        //Add new number to array
}  


    svg.selectAll("rect")
      .data(data)
      .transition()
      .duration(200)
      .delay(function(i){return i*100})
      .attr("y", function(d, i){return (h / data.length)*i})
      .attr("height", h / data.length - 5)
      .attr("width", function(i) {return i*(w-400)/max});

    });

  
              
</script>
	</body>
</html>
