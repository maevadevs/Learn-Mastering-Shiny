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
  - Used to in R analysis scripts
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
  - If `greeting` (output) needs to be recomputed whenever `name` (input) changes, `greeting` has a reactive dependency on `name`
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
  - Found in between and input and an output

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

