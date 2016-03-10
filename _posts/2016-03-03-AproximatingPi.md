---
layout: post
title: A Shiny App honoring Pi
---

With March 14 looming on the horizon I decided to make a little interactive visualizaton on how to approximate Pi by randomly generating points in a square. This is a certainly [not a novel](https://en.wikipedia.org/wiki/Monte_Carlo_integration) idea but I think it touches on two really important topics: the nature of volume/area and the nature of uncertainty/randomness and it does so in an accesible and engaging way. Hopefully we'll see more interactive tools in math and science classes - I think they have a lot of potential.

Anyways check the app out below! It might take a few seconds to load.



<iframe 
src="https://apapiu.shinyapps.io/Pies/" 
style="border: none; width: 1000px; height: 1100px">
</iframe>

Here's the code in R - nothing too fancy.

{% highlight r %}

library(shiny)
library(shinythemes)
library(ggplot2)

ui <- fluidPage( theme = shinytheme("flatly"),
        

     fluidRow( column (verticalLayout( 
         
                  wellPanel("We will aproximate Pi by generating random points
                  in a square and then counting what fraction of those
                  points are inside the circle contained in the square.
                  You can think of this as randomly throwing darts at a square board.
                  Since we know what the area of a square is we've 
                  replaced a calculus problem with a counting problem! Play around with the slider on the
                  right - you should get a better approximation of Pi the more points you use."),
                      
                  wellPanel(sliderInput(inputId = "num",
                    label = "Number of Points",
                    value = 1000, min = 10, max = 30000)),
                
                  wellPanel("Your approximation of pi is:",
                    verbatimTextOutput("pi"))) , width = 3),
                       
     
         column(plotOutput("plot", width = "700px", height = "700px"), 
                  width = 8)
        )
     )


server <- function(input, output) {
    
    dat <- function(num){
        x <- replicate(2, runif(num ,min = -1, max = 1))
        inside <- (apply(x, 1, function(x) {x[1]^2 +x[2]^2}))<=1
        data.frame(x, inside)
    }
    
    rv <- reactive({dat(input$num)})
    
    output$pi <- renderPrint({
        
        4 *sum(rv()$inside)/input$num
    })
    
    output$plot <- renderPlot({
        
        qplot(rv()[,1], rv()[,2], data = rv(),color = inside, xlab = "x", ylab = "y") +
                     scale_color_brewer(palette = "Set1") +
                     theme_minimal() +
                     theme(legend.position="none")  
    })
    }

shinyApp(ui = ui, server = server)

{% endhighlight %}
