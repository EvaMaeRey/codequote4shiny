
<!-- README.md is generated from README.Rmd. Please edit that file -->

# codequote4shiny

<!-- badges: start -->

<!-- badges: end -->

Problem

‘… and therefore you must write your application in JavaScript. And that
can be a problem if you are doing data science, because a lot of the
tools you’re used to might not exist in JavaScript.’
<https://youtu.be/kwLu3hxwn5M?t=318> Winston Chang | Running Shiny
without a server | RStudio (2022)

This is also a problem in terms of *teaching* apps that live on the web.
There is a tools disconnect. <http://www.isi-stats.com/isi/applets.html>

What might be really cool is to make connection between what’s in the
app teaching tool, and real-world data science.

# The goal of codequote4shiny is to:

1.  Allow you to write some awesome R code…
2.  Generate an awesome shiny app that demonstrates something about the
    real world (for starters statistical principles); provide guard
    rails that will focus learners on math/stats (or whatever\!)
    principles of interest.
3.  Quote back the R code; jumping off point; give a taste of what’s to
    come. (Taking off the guard rails and training wheels\!)

Goals are pretty limited.

outcome will be like:

  - pnorm.R app to be included
  - <https://github.com/EvaMaeRey/mytidytuesday/blob/master/2023-02-14-heart/heart_app.R>
  - <https://github.com/EvaMaeRey/mytidytuesday/blob/master/2023-02-14-heart/heart_app_automate.R>
  - <https://daattali.com/shiny/ggExtra-ggMarginal-demo/>

Who’s also doing this:

  - dean attali <https://deanattali.com/2015/04/21/r-package-shiny-app/>
  - brain code quote: sidchop.shinyapps.io/braincoder

# How you might manage w/o codequote4shiny

``` r
library(tidyverse)
### Design code to be featured in app ####
return_heart_df(n_vertices = 16) %>%
  ggplot() +
  aes(x = x, y = y, group = group) +
  geom_polygon(
    fill = "darkred",
    color = "magenta",
    linewidth = 4,
    alpha = 0.8,
    linetype = "dashed"
  ) +
  coord_equal()


#### version with variables predefined ###

#### UIish ###
input_num_vertices <- 16
input_char_fill <-"darkred"
input_char_color <- "magenta"
input_num_linewidth <- 4
input_num_alpha <- 0.8
input_char_linetype <- "dashed"

#### Server ish ####
return_heart_df(n_vertices = input_num_vertices) %>%
  ggplot() +
  aes(x = x, y = y, group = group) +
  geom_polygon(
    fill = input_char_fill,
    color = input_char_color,
    linewidth = input_num_linewidth,
    alpha = input_num_alpha,
    linetype = input_char_linetype
  ) +
  coord_equal()

### What we need 'for shiny' translation of above;  Original approach for reference
'
# library(returnheart)

return_heart_df(n_vertices = input$num_vertices) %>%
  ggplot() +
  aes(x = x, y = y, group = group) +
  geom_polygon(
           fill = input$char_fill,
           color = input$char_color,
           linewidth = input$num_linewidth,
           alpha = input$num_alpha,
           linetype = input$char_linetype
           ) +
  coord_equal()

' ->
  for_shiny1

### What we need 'for shiny'; using string manipulation to get bindings
'
# library(my_return_heart_package)

return_heart_df(n_vertices = input_num_vertices) %>%
  ggplot() +
  aes(x = x, y = y, group = group) +
  geom_polygon(
           fill = input_char_fill,
           color = input_char_color,
           linewidth = input_num_linewidth,
           alpha = input_num_alpha,
           linetype = input_char_linetype
           ) +
  coord_equal()

' -> code_general_form


code_general_form %>% 
  str_replace_all("input_", "input$") ->
  for_shiny_interactivity

####### Shiny Stuff ####

library(shiny)
library(tidyverse)



##### Define server logic ########

server <- function(input, output) {


  output$distText <- renderText({

    for_shiny_interactivity %>%
      # numeric
      str_replace_all("input\\$num_vertices", as.character(input$num_vertices)) %>%
      str_replace_all("input\\$num_alpha", as.character(input$num_alpha)) %>%
      str_replace_all("input\\$num_linewidth", as.character(input$num_linewidth)) %>%
      # character
      str_replace_all("input\\$char_color", as.character(input$char_color) %>% paste0('\\"', ., '\\"')) %>%
      str_replace_all("input\\$char_linetype", as.character(input$char_linetype) %>% paste0('\\"', ., '\\"')) %>%
      str_replace_all("input\\$char_fill", as.character(input$char_fill) %>% paste0('\\"', ., '\\"'))


  })

  output$distPlot <- renderPlot({

    eval(parse(text = for_shiny_interactivity))

  })

}

## Define UI for application that draws a heart ##
ui <- fluidPage(

  # Application title
  titlePanel("When your heart has some rough edges..." ),

  titlePanel("... add vertices!" ),

  # Sidebar with a slider input for number of bins
  sidebarLayout(
    sidebarPanel(
      sliderInput(inputId = "num_vertices",
                  label = "vertices",
                  step = 1,
                  min = 10,
                  max = 200,
                  value = 16
                  ),
      selectInput(inputId = "char_color",
                  label = "color",
                  selected = "magenta",
                  choices = colors()
                  ),
      selectInput(inputId = "char_fill",
                  label = "fill",
                  selected = "darkred",
                  choices = colors(),
                  ),
      radioButtons(inputId = "char_linetype",
                   label = "linetype",
                   selected = "dashed",
                   choices = c("dashed", "dotted", "solid")),
      sliderInput(inputId = "num_alpha",
                  label = "alpha",
                  value = .8,
                  min = 0,
                  max = 1,
                  step = .02
                  ),
      sliderInput(inputId = "num_linewidth",
                  label = "linewidth",
                  value = 4,
                  min = 1,
                  max = 5,
                  step = 1
                  )

    ),

    # Show a plot of the generated distribution
    mainPanel(
      verbatimTextOutput("distText"),
      plotOutput("distPlot")
    )

      )


)

# Run the application
shinyApp(ui = ui, server = server)
```

You can install the released version of codequote4shiny from
[CRAN](https://CRAN.R-project.org) with:

``` r
#   install.packages("codequote4shiny")
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("EvaMaeRey/codequote4shiny")
```

## Example

This is a basic example which shows you how to solve a common problem:

``` r
# library(codequote4shiny)
## basic example code
```
