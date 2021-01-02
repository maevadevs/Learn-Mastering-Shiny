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
  - 
