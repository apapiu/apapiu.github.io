<!DOCTYPE html>
<html>
  
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
  
 </head>
  
<body>
<script src="//d3js.org/d3.v3.min.js" \\the d3 script reference></script> 
  
  

  
<div id="option">
    <input name="updateButton" 
           type="button" 
           value="New Values" 
           onclick="updateData()" />
</div>  

 
<script>
 
var n = 2000;
var w = 600;
var h = 600;
  
var color = d3.scale.category10()


  
var data = [];
for (var i = 0; i < n; i ++) {
  data.push([Math.random(), Math.random()]);
}
  

  
var svg = d3.select("body")
  .append("svg")
  .attr("width", w)
  .attr("height", h);
  
  svg.append("rect")
  .attr("height", h)
  .attr("width", w)
  .attr("fill", "white")
  
  svg.selectAll("circle")
  .data(data)
  .enter()
  .append("circle")
  .attr("cx", function(d){return (w-50)*d[0]+25;})
  .attr("cy", function(d){return (w-50)*d[1]+25;})
  .attr("fill", function(d){
    return color((d[0]-1/2)*(d[0]-1/2) +(d[1]-1/2)*(d[1]-1/2) < 1/4)})
  .attr("r", 0)
  .transition()
  .duration(500)
  .delay(500)
  .attr("r", Math.sqrt(50000/(n+3000))) //random function to make circles how I want
  

updateData = function() {
     
var n = 4000;

var data = [];

for (var i = 0; i < n; i ++) {
  data.push([Math.random(), Math.random()]);
} // generating new data
    
   svg.selectAll("circle")
  .data(data)
  .transition()
  .duration(1000)
  .attr("cx", function(d){return (w-50)*d[0]+25;})
  .attr("cy", function(d){return (h-50)*d[1]+25;})
  .transition()
  .duration(1000)
  .delay(1000)
  .attr("fill", function(d){
    return color((d[0]-1/2)*(d[0]-1/2) +(d[1]-1/2)*(d[1]-1/2) < 1/4)})
    
 };
 
</script>
</body>
</html>
