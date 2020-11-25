# Hello Shiny

A Shiny app has 2 main components:

- UI - defines how your app looks
- Server - defines how your app works
- Reactive Expressions - reactive programming to automatically update outputs when inputs change

## Installing and Loading Shiny

```R
# Installation
install.packages("shiny")

# Loading
library(shiny)
```

## App Directory

```
RootDir
|-> `app.R`
```

- `app.R` - Handles booth looks and functionalities

## Minimum Boilerplate

```R
# app.R
# -----

# Load library
library(shiny)

# Define UI
ui <- fluidPage(
  "Hello, world!"
)

# Define Server Logic
server <- function(input, output, session) {}

# Combine the App: UI + Server
shinyApp(ui, server)
```

## Starting and Stopping Shiny App

There are multiple options for this:
- Click the *Run App* button in the document toolbar
- `ctrl + shit + enter` or `cmd + shift + enter`
- `shiny::runApp('path/to/app.R')`

To stop the app:
- Press `Esc` or `ctrl + c`
- Click the stop button
- Close the Shiny app browser window

## Adding UI Controls

```R
# Define UI
ui <- fluidPage(
  selectInput("dataset", label = "Dataset", choices = ls("package:datasets")),
  verbatimTextOutput("summary"),
  tableOutput("table")
)
```

- `fluidPage()` - A layout function that sets up the basic visual structure of the page
- `selectInput()` An input control that lets the user interact with the app by providing a value
- `verbatimTextOutput()` -  An output controls that tell Shiny where to put rendered output: displays code
- `tableOutput()` - An output controls that tell Shiny where to put rendered output: displays tables

To make things work, we need to handle the behavior next

## Adding Behavior

```R
# Define Server Logic
server <- function(input, output, session) {
  output$summary <- renderPrint({
    dataset <- get(input$dataset, "package:datasets")
    summary(dataset)
  })
  
  output$table <- renderTable({
    dataset <- get(input$dataset, "package:datasets")
    dataset
  })
}
```
