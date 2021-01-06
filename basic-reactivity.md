# Basic Reactivity

- We use reactivity in server logic
- Specify a graph of dependencies so that when an input changes, all related outputs are automatically updated
- Reactive expressions also allows to eliminate duplicate work

## The Server Function

- Shiny invokes the `server()` function each time a new session starts
- When `server()` is called, it creates a new local environment that is independent of every other invocation of the function
  - Each session has a unique state
  - Isolating the variables created inside the function
  - Almost all of the reactive programming we do in Shiny will be inside the server function

```r
server <- functon(input, output, session) {}
```

- The object parameters are created by Shiny when the session begins

### `input` Parameter

- A list-like object that contains all the input data sent from the browser
  - Named according to the input ID
  - `input` objects are read-only: If we attempt to modify an input inside the server function, an error is thrown
  - `input` reflects what is happening in the browser
  - The browser is Shiny's *single source of truth*
  - Instead of updating `input` directly, we make use of other functions to do so, such as `updateNumericInput()`
  
```r
ui <- fluidPage(
  numericInput("count", label="Number of values", value=100)
)
```

- Here, `input$count` is accessible in `server()`
  - Initial value is 100
  - It is automatically updated based on user interaction
- Reading from `input` is selective
  - We must be in a reactive context created by a function like `renderText()` or `reactive()`
  - It is an important constraint that allows outputs to automatically update when an input changes

### `output` Parameter

- A list-like object used for sending output instead of receiving input
  - Named according to the output ID
  - The ID is quoted in the UI, but not in the server
  - We use `$` in the server instead
  - Always use `output` in concert with a `render` function

```r
ui <- fluidPage(
  textOutput("greeting")
)

server <- function(input, output, session) {
  output$greeting <- renderText("Hello human!")
}
```

- The `render()` function does 2 things:
  - Sets up a special reactive context that automatically tracks what inputs the output uses
  - Converts output of code into HTML suitable for display on a web page
- Error from `output` happens if:
  - You forget the `render()` function
  - You attempt to read from an output

```r
server <- function(input, output, session) {
  message("The greeting is ", output$greeting) #> Error: Reading from shinyoutput object is not allowed.
}
shinyApp(ui, server)
```

## Reactive Programming

The real magic of Shiny happens when an app is with both inputs and outputs working simultaneously and automatically

```r
ui <- fluidPage(
  textInput("name", "What's your name?"),
  textOutput("greeting")
)

server <- function(input, output, session) {
  output$greeting <- renderText({
    paste0("Hello ", input$name, "!")
  })
}
```

**The big idea behind Shiny's reactivity: you don't need to tell an output when to update, because Shiny automatically figures it out for you**

- Shiny performs the action every time we update `input$name` updates
  - The code does not tell Shiny to create the string and send it to the browser
  - It informs Shiny how it could create the string if it needs to
  - It is up to Shiny when the code should be run
  - It is Shiny's responsibility to decide when code is executed
  - Think of your app as providing Shiny with recipes, not giving it commands

### Imperative vs Declarative

- **Imperative Programming** 
  - Issue a specific command and it’s carried out immediately
  - Assertive
  - Used in R analysis scripts
- **Declarative Programming**
  - Express higher-level goals or describe important constraints
  - Rely on someone else to decide how and when to translate that into action
  - Passive-aggressive
  - Used in Shiny
  - Ups: More freeing
  - Downs: Occasional time where you know exactly what you want, but you can't figure out how to frame it in a way that the declarative system understands

### Laziness

- Apps in Shiny are extemely lazy
  - A Shiny app will only ever do the minimal amount of work needed to update the output controls that you can currently see
  - Downs: It is harder to debug. If the function name does not match, it is never run

**If you’re working on a Shiny app and you just can't figure out why your code never gets run, double check that your UI and server functions are using the same identifiers.**

### Reactive Graph

- In most R code, we can understand the order of execution by reading the code from top to bottom
  - That does not work with Shiny
  - The codes are only executed when needed
- To understand the flow of execution, we need to look at the *Reactive Graph*
  - Describes how inputs and outputs are connected
  - Contains one symbol for every input and output
  - Connect an input to an output whenever the output accesses the input
  - **If `greeting` (output) needs to be recomputed whenever `name` (input) changes, `greeting` has a reactive dependency on `name`**
    - Dep Graph: `input$name -> output$greeting`
    - Graphical conventions we used for the inputs and outputs: `name` input naturally fits into the greeting `output`

**The reactive graph is a powerful tool for understanding how the app works**

- Often useful to make a quick high-level sketch of the reactive graph
- Reminder on how all the pieces fit together

### Reactive Expressions

- One more important component of the Reactive Graph
- A tool that reduces duplication in your reactive code by introducing additional nodes into the reactive graph
- Take inputs and produce outputs
  - Have a shape that combines features of both inputs and outputs
  - Found in between an input and an output

```r
server <- function(input, output, session) {
  # Reacive Expression
  getString <- reactive(paste0("Hello ", input$name, "!"))
  # Render to output
  output$greeting <- renderText(getString())
}
```

- Dep Graph: `input$name -> rx$getString() -> output$greeting`

### Execution Order

- **Execution order is solely determined by the reactive graph and the laziness of Shiny**
- Completely different from typical programming execution orders
  - Not by the layout in the server function
  
```r
server <- function(input, output, session) {
  # The order of these two lines does not matter
  output$greeting <- renderText(string())
  string <- reactive(paste0("Hello ", input$name, "!"))
}
```

## Reactive Expresssions

Reactive expressions are important for 2 main reasons:

- Give Shiny more information so that it can do less recomputation when inputs change
- Make it easier for humans to understand the app by simplifying the reactive graph

A flavor of both inputs and outputs:

- We can use the results of a reactive expression in an output
- Reactive expressions depend on inputs
- Know when they need updating
<!---->
- *Producers*: Some functions work with either reactive inputs or expressions
- *Consumers*: Some functions work with either reactive expressions or reactive outputs

### Reactive Graph

```r
server <- function(input, output, session) {
  output$hist <- renderPlot({
    x1 <- rnorm(input$n1, input$mean1, input$sd1)
    x2 <- rnorm(input$n2, input$mean2, input$sd2)
    
    histogram(x1, x2, binwidth = input$binwidth, xlim = input$range)
  }, res = 96)

  output$ttest <- renderText({
    x1 <- rnorm(input$n1, input$mean1, input$sd1)
    x2 <- rnorm(input$n2, input$mean2, input$sd2)
    
    t_test(x1, x2)
  })
}
```


- Shiny is smart enough to update an output only when the inputs it refers to change
- It is not smart enough to only selectively run pieces of code inside an output

```r
x1 <- rnorm(input$n1, input$mean1, input$sd1)
x2 <- rnorm(input$n2, input$mean2, input$sd2)
t_test(x1, x2)
```

- Normally, we would update `x1` when `n1`, `mean1`, or `sd1` changes
- And update `x2` when `n2`, `mean2`, or `sd2` changes
- However, Shiny only looks at the output as a whole, so it will update both `x1` and `x2` every time
- The reactive graph would be very dense: every input is connected directly to every output
  - The app is hard to understand because there are so many connections
  - The app is inefficient because it does more work than necessary
- We can simplify these scenarios by using Reactive Expressions
  - Pull out the repeated code into two new reactive expressions
  - To create a reactive expression, we call `reactive()` and assign the results to a variable
  - To use the expression, we call the variable like a function

```r
server <- function(input, output, session) {

  # Creating reactive expressions
  x1 <- reactive(rnorm(input$n1, input$mean1, input$sd1))
  x2 <- reactive(rnorm(input$n2, input$mean2, input$sd2))

  output$hist <- renderPlot({
    histogram(x1(), x2(), binwidth = input$binwidth, xlim = input$range)
  }, res = 96)

  output$ttest <- renderText({
    t_test(x1(), x2())
  })
}
```

- This transformation yields a substantially simpler dependency graph
- It is easier to understand the app: We can understand connected components in isolation
  - The values of the distribution parameters only affect the output via `x1` and `x2`
  - The app is much more efficient because it does much less computation
<!---->
**The rule of one: whenever you copy and paste something once, you should consider extracting the repeated code out into a reactive expression**

- Reactive expressions do not simply make it easier for humans to understand the code
- They also improve Shiny's ability to efficiently rerun code

### Why We Need Reactive Expressions

- Functions and variables do not work in a reactive environment
- We need reactive contexts for not computing values only once but whenever needed or changed
  - Variables calculate the value only once
  - Functions will work but any input will cause all outputs to be recomputed
  - Reactive expressions automatically cache their results, and only update when their inputs change

## Controlled Timing

- Two more techniques to either increase or decrease how often a reactive expression is executed
  - Timed Invalidation
  - On-click

### Timed Invalidation: `reactiveTimer()`

- We can adjust the frequency of updates with `reactiveTimer()`
  - Reactive expression that has a dependency on a hidden input: the current time
  - Use when you want a reactive expression to invalidate itself more often than it otherwise would
  
```r
server <- function(input, output, session) {
  timer <- reactiveTimer(500)
  
  x1 <- reactive({
    timer()
    rpois(input$n, input$lambda1)
  })
  x2 <- reactive({
    timer()
    rpois(input$n, input$lambda2)
  })
  
  output$hist <- renderPlot({
    histogram(x1(), x2(), binwidth = 1, xlim = c(0, 40))
  }, res = 96)
}
```

- We use the `reactiveTimer` instance `timer` in the reactive expressions that compute
  - We call it, but don’t use the value
  - This lets those expressions take a reactive dependency on `timer`, without worrying about exactly what value it returns

### On-Click: `actionButton()` and `eventReactive()`

- Shiny could have a lot to do in the background
- Too much backlog leads to poor performance
- We might want to require the user to opt-in to performing the expensive calculation by requiring them to click a button
  - We can use `actionButton()` for this with `eventReactive()`
- `eventReactive()` is a way to use input values without taking a reactive dependency on them
  - First argument specifies what to take a dependency on
  - Second argument specifies what to compute

```r
ui <- fluidPage(
  fluidRow(
    column(3, 
      numericInput("lambda1", label = "lambda1", value = 3),
      numericInput("lambda2", label = "lambda2", value = 3),
      numericInput("n", label = "n", value = 1e4, min = 0),
      actionButton("simulateBttn", "Simulate!")
    ),
    column(9, plotOutput("hist"))
  )
)

server <- function(input, output, session) {
  x1 <- eventReactive(input$simulateBttn, {
    rpois(input$n, input$lambda1)
  })
  x2 <- eventReactive(input$simulateBttn, {
    rpois(input$n, input$lambda2)
  })

  output$hist <- renderPlot({
    histogram(x1(), x2(), binwidth = 1, xlim = c(0, 40))
  }, res = 96)
}
```

## Observers: `observeEvent()`

- Sometimes, we need to reach outside of the app and cause side-effects to happen elsewhere in the world
  - Saving a file to a shared network drive
  - Sending data to a web API
  - Updating a database
  - Printing a debugging message to the console
- These are background tasks
  - We cannot use an output and a render function
  - Intead, we use *observers* or events
  - These are similar to callback functions
- We can create observers with `observeEvent()`
  - Very similar to `eventReactive()`
  - 2 arguments:
    - **`eventExpr`**: input or expression to take a dependency on
    - **`handlerExpr`**: code that will be run

```r
server <- function(input, output, session) {
  string <- reactive(paste0("Hello ", input$name, "!"))
  
  output$greeting <- renderText(string())
  
  # When name is updated, send message to the console
  observeEvent(input$name, {
    message("Greeting performed")
  })
}
```

### `observeEvent()` vs `eventReactive()`

- Do not assign the result of `observeEvent()` to a variable
- Cannot refer to `observeEvent()` from other reactive consumers
<1---->
- Observers and outputs are closely related
