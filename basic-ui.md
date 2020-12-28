# Basic UI

Shiny encourages separation of code:

- User Interface (front end) 
- Codes that drives the app’s behavior (back end)

Shiny provides HTML inputs, outputs, and layouts

## Inputs

### Common Structure of Input Functions

- The first argument is always the `inputId`
  - Identifier used to connect the front-end with the back-end
  - Accessible in the server as `input$inputId`
  - Has 2 constraints:
    - Must contain only letters, numbers, and underscores
    - Must be unique
- The second argument is *usually* the `label`
  - Used to create a human-readable label for the control
  - Make sure that your app is usable by humans
- The third argument is *usually* the `value`
  - Lets you set the default value
- The remaining arguments are unique to the input control

```r
sliderInput("min", 
            "Limit (minimum)",
            value=50, 
            min=0, 
            max=100)
```

### Free Text Inputs

We can use `validate()` on text to make sure they follow specific requirements

- `textInput`
- `passwordInput`
- `textAreaInput`

### Numeric Inputs

Recommended: Only use sliders for small ranges or cases where the precise value is not so important

- `sliderInput`
- `numericInput`

Sliders are extremely customisable and there are many ways to tweak their appearance

### Dates

Provide a convenient calendar picker

- `dateInput`
- `dateRangeInput`

Date format, language, and the day on which the week starts defaults to US standards:
- `format` - Set the date format
- `language` - Set the date language
- `weekstart` - Set the weekstart day

### Limited Options

- Radio Buttons are suitable for short lists
  - `choiceNames`/`choiceValues`: Display options other than plain text
- Select Inputs are suitable for longer lists
  - `multiple`: Allow multiple selection
<!---->
- `selectInput`
- `radioButtons`
- `checkboxGroupInput`
- `checkboxInput`

### File Uploads

Allow user to upload a file, requires special handling on the server side

- `fileInput`

### Action Buttons

Let the user perform an action: These are most naturally paired with `observeEvent()` or `eventReactive()` in the server function

- `actionButton`
- `actionLink`
<!---->
- `class` argument: `btn-primary`, `btn-success`, `btn-info`, `btn-warning`, `btn-danger`, `btn-lg`, `btn-sm`, `btn-xs`, `btn-block`

## Outputs

- Placeholders in the UI that are later filled by the server
- The first argument is always the `outputId`
  - Identifier used to connect the front-end with the back-end
  - Accessible in the server as `output$outputId`
  - Has 2 constraints:
    - Must contain only letters, numbers, and underscores
    - Must be unique
- Each output is coupled with a `render` function in the backend
- There are 3 main types of outputs:
  - Texts
  - Tables
  - Plots

**You should do as little computation in the server's render functions as possible**

### Texts

- `textOutput` - Regular text
- `VerbatimOutput` - Fixed code and console text outputs
<!---->
- `renderText()` - combines the result into a single string
- `renderPrint()` - prints the result

### Tables

- `tableOutput` - Usefule for small tables
- `dataTableOutput` - Usefull for complete data frames
<!---->
- `renderTable()` - render a static table of data, showing all the data at once
- `renderDataTable()` - render a dynamic table, showing a fixed number of rows along with controls to change which rows are visible

### Plots

- `plotOutput` - Display any type of R graphic
  - By default, it will take up the full width of its container and  400px high
  - Use `width` and `height` to override
  - *Recommended: always set `res=96` as that will make your Shiny plots match what you see in RStudio as closely as possible*
- `renderPlot()` - render any type of R graphic
<!---->
- Plots are special because they are outputs that can also act as inputs
  - `click`, `dblclick`, and `hover` for reactive inputs

### Downloads

- `downloadButton`
- `downloadLink`

## Layouts

- Layouts are about arranging the inputs and outputs on the page
- High-level visual structure of an app
- Created by a hierarchy of function calls

```r
fluidPage(
  titlePanel(),
  sidebarLayout(
    sidebarPanel(
      sliderInput("obs")
    ),
    mainPanel(
      plotOutput("distPlot")
    )
  )
)
```

### Page Functions

#### `fluidPage()`

- Uses a *Bootstrap* layout system with Bootstrap's defaults
- Technically, `fluidPage()` is all you need for an app
  - You can put inputs and outputs directly inside of it
  - But dumping all the inputs and outputs in one place doesn’t look very good

#### Page With Sidebar

Create a two-column layout with inputs on the left and outputs on the right

- `sidebarLayout()` - Establish that we are using a sidebar style layout
- `titlePanel()` - Set the overall app title
- `sidebarPanel()` - Define the area for the sidebar and all its contents (inputs)
- `mainPanel()` - Defines the area for the main panel and all of its contents (outputs)

#### Multi-Row

- `sidebarLayout()` is built on top of a flexible multi-row layout
- Each `fluidRow()` can be subdivided into its own number of columns, up to 12
- We can set columns with `column()`
  - The first argument is the `width` of the column
  - `1` is the smallest, `12` is the largest

```r
fluidPage(
  fluidRow(
    column(4, 
      ...
    ),
    column(8, 
      ...
    )
  ),
  fluidRow(
    column(6, 
      ...
    ),
    column(6, 
      ...
    )
  )
)
```

#### Themes

- Creating a complete theme from scratch is a lot of work but often worth it
- But we can get some easy wins by using the [shinythemes](https://rstudio.github.io/shinythemes/) package
- Other good resources:
  - [Theme Selector](https://shiny.rstudio.com/gallery/shiny-theme-selector.html)
  - [fresh](https://dreamrs.github.io/fresh/)
  - [bs4Dash](https://github.com/RinteRface/bs4Dash)

## Under The Hood

- Shiny code **is** R code
- You can use any R techniques with Shiny
- If you copy and paste code more than three times, you should consider writing a function or using a for loop
- All input, output, and layout functions return HTML
  - You can see the HTML by executing UI functions directly in the console
