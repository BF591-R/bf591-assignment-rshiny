## Assignment Rshiny
> Now working on my companion package: R Dully

> You telling me R shinied this app?

### Problem Statement
Communicating data and results is one of the most nefariously troubling aspects of data science. In bioinformatics we often have colleagues who are less experienced when it comes to viewing and manipulating data, such as clinicians or biologists who think computers are living objects of evil (they are). We must utilize better ways to communicate our work to them.

### Learning Objectives
1. Develop the two sides of an R Shiny application: server and UI.
2. Understand reactivity when it comes to an application.
3. Take in user input and use it to change the state of a web application.

### Skill List
- R Shiny
- Troubleshooting R Shiny
- Testing R Shiny? Like, _nobody_ does this so it could be a pretty good edge for you.

### Instructions
Complete `app.R` (not `main.R`), including the `server` and `ui` functions. We are not looking for exact parity when it comes to the _style_ of the application, just that you replicate the user experience and inputs and create the two methods of output: a table and a volcano plot.  

The sample application is hosted here: [https://taylorfalk.shinyapps.io/bf591-assignment-7/](https://taylorfalk.shinyapps.io/bf591-assignment-7/)

View this document along with the sample application and try to determine which parts of the UI and output mentioned in this document are associated with the functioning application.

Test are available for this assignment, because of course they are. You can run them using `testthat::test_file("test_app.R")`. You can also use `source("test_app.R")` (but it's less informative). Shiny apps add another dimensionality of complexity, so we are relatively limited in what we can test for. As always, if you feel a test isn't working correctly please let a TA know.

<span style="color:red">Note:</span> By design, these instructions are rather sparse. This is likely entirely new territory for you, but you have gotten this far and there is a lot of information available for R Shiny. Please read documentation or look for help online to help yourself understand how this application works.

### "Function" Details
Without writing an entire treatise on R Shiny (see the book), a Shiny application is broken into two components: the UI function and the Server function. In web development parlance, these represent the "front end" and the "back end", respectively. The front end is all the code that controls what a user sees and does (the goop that is in the web browser), and the back end is what connects that front end to the server where most of the code, processing power, and data will live.  

You can use two files for this, but we have set it up for `app.R` to contain the entire Shiny application. It's general format is like so:
```
library(shiny)
ui <- fluidPage(
  # front end
) 
server <- function(input, output, session) {
   # back end
}
shinyApp(ui = ui, server = server) # run app
```
After loading the Shiny library, we create the front end parameters with the `fluidPage()` function. Fluid page is a simple wrapper for designing a Shiny page, and inside we can use functions like `titlePanel()`, `sidebarLayout()`, and `tabPanel()` to organize what is displayed. 

Next, we create the `server` function which is much more similar to writing base R. `input`, `output`, and `server` are automatically handled by Shiny, and we use these objects to pass data between front end (UI) and back end (server). A user can select a color for their graph, we can store that color in `input$color1`, read that `input` inside server, and pass the resulting graph to `output` to place onto the page.

Finally, the `shinyApp()` is called to run everything. It does most of the heavy lifting so we can worry more about themes and pretty colors for our app (`library(bslib)`, themeing will not be graded). It goes without saying, much like working on the fonts and headers in an essay, you should wait until your app is mostly complete before you start changing the aesthetics.

#### 1. `ui <- fluidPage(...)`
R Shiny is a _highly_ customization package, so we will describe what inputs and outputs we're expecting, but leave the theme, placement, arrangement, and design of these elements up to you. R Shiny's default layout is the very usable `sidebayLayout()`, but if you want more control feel free to use `fluidRow()`.

In our sidebar, we have six elements for user input. There is the `fileInput()`, followed by two `radioButtons()`, followed by two `colourInputs()`, and finally a `sliderInput()`. The most important part for these inputs is the `inputId` argument, which later is used inside the server function's `input`. If I make my `inputId=fileupload`, then I can access that upload using `input$fileupload` later on.

##### File Input
The file input function will take in a CSV file, helpfully included in this repository in the `data/` folder. Not vital, but a nice method we can do is _only_ allow CSV file types using `accept=`. This is useful if your users are easily confused and try to enter other data.

##### Radio Buttons
Since we are ultimately plotting a graph based on the data uploaded, we want to select what variables are being plotted. We set up radio buttons using the names from the column header, and can match those later on in ggplot when selecting data to plot. Make sure your two radio button functions have different `inputIds`.

##### Colour Inputs
This is a bonus library that is very useful when working with color. Installing `colourpicker` and using `colourInput()` inside your side panel layout lets us choose two colors to place on our graph. Feel free to choose your own default colors, and again make sure these have separate input IDs.

##### Slider Input
Out last piece of input will be the slider input. This allows the user to select the magnitude of adjusted p-value they would like to color on their graph. This is the $x$ in $1 \times 10^{x}$.

##### The Button
Finally, there is a button to hit when everything is ready. You can use a `submitButton()` or an `actionButton()`. Both work slightly differently but having a button just lets us control when a plot is created. You can use `icon = icon()` to choose a different icon for the button.

#### 2. Tabs and Outputs
Once the inputs are established on the page, we want somewhere to put everything once it's done being run! This is accomplished by the `*Output()` functions. We have two outputs to consider, a plot and a table, so we can use `plotOutput()` and `tableOutput()` respectively, using the names inside `output` from the server function. These can be placed simply on the `mainPanel()`.

There is one additional step if you would like to use tabs instead, which lets us create more space for our outputs. Instead of using the output functions directly, we can first use `tabsetPanel()` and place two `tabPanel()` functions inside. We then finally place our outputs. A tab setup will look something like this:
```
ui <- fluidPage(
  ...
  mainPanel(
    tabsetPanel(
      tabPanel("Tab1 Name"
        tableOutput(...)
      ),
      tabPanel("Tab2 Name"
        plotOutput(...)
      )
    )
  )
)
```
It might look a little messy but if you can keep an eye on hierarchy it will start to make more sense.

#### 2. `server <- function(...`
Our second main component, and where we will be doing a lot of the "work", is the server function. This is written like a normal function in R, with three specific parameters or arguments. These arguments can be used _inside_ server to communicate with the elements we wrote above. The `input` object can be used to take in user selections, such as what color was chosen. The `output` object is then use to assign results to the output functions we included above. 

##### `load_data()`
Much like a normal R script, we can save ourselves some trouble by splitting our code blocks into functions. In this case, we can load the user select data into different parts of our app, like displaying a plot and then a table. 

This `load_data()` function should be written as a reactive expression, covered in more detail in the book chapter. Because the user is being asked to upload data we do not want to process that data twice, which would slow down our application.  

This reaction function will update when the inputs inside it change, in our case the file name from our file uploader. Ultimately, we just want this function to return the data frame from loading whatever is in the upload box. 

<span style="color:red">Note</span> You will need to specify the **datapath** from the `fileInput()` function's input ID. If my input ID was "taylorsfile" then my actual input will be `input$taylorsfile$datapath`.

##### `volcano_plot()`
This should feel like meeting an old friend: make a volcano plot. The function takes in a data frame, column names to choose as axes, a negative integer to use as a cutoff, and two colors to make points above and below that cutoff. Simple and exciting. Please do all the normal ggplot things like labeling axes and changing the theme.

***Hints***

A little hint, sometimes tidy elements don't play nice with string names (like this function asks for). You can use `!!sym(x_name)` to access those columns. There are also tidy friendly ways to do this, but uhh I like this way!


##### `draw_table()`
Same as above, this function is _even_ simpler. Take in a data frame and the slider magnitude, and cut off any rows that aren't lower than the slider threshold. See the second tab on the example app for how the slider value interacts with what populates the table. We also want to format some of our p-value columns to display more digits, as otherwise they will show up as "0" in the table (not informative). You can use tidyverse functions or `formatC()` to accomplish this.

##### `output$...`
Finally we will cram our results onto the web page. Use the functions `renderPlot()` and `renderTable()` to **return** your volcano plot and filtered data table using the above two functions. These outputs _are_ checked by the testing script, so there is a bare minimum as to what we are looking for. Please reference the test file `test_app.R` to see what is being tested. The table's p-values should have a magnitude, not just `0`.

If the outputs show up correctly when you load the CSV file, congrats! You've built your app correctly. If it didn't work, I am so sorry. R Shiny can be difficult to troubleshoot so do not be afraid to look up the strange errors you see or ask for help.
