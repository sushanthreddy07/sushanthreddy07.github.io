---
layout: archive
title: "Code snippet"
permalink: /Code mid/
author_profile: true

---

{% include base_path %}

** Creating a data set**

```{r}
# Load necessary library
library(tibble)

# Manually enter the data
data <- tibble(
  Country = c("Japan", "South Korea", "Taiwan", "New Zealand", "China", "Bangladesh", "India", "Egypt", "Iraq", "Algeria", "Iran", "Libya", "Gabon", "Mauritius", "Togo", "Rwanda", "Ethiopia", "Liberia", "Venezuela", "Nicaragua", "United States", "UK", "Norway", "Northern Cyprus", "Montenegro", "France", "Switzerland", "Russia", "Ukraine", "Belarus", "Kyrgyzstan", "Uzbekistan", "Turkmenistan"),
  Region = c("Asia", "Asia", "Asia", "Asia", "Asia", "Asia", "Asia", "Middle East and North Africa", "Middle East and North Africa", "Middle East and North Africa", "Middle East and North Africa", "Middle East and North Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Americas", "Americas", "Americas", "Europe", "Europe", "Europe", "Europe", "Europe", "Europe", "Former Soviet Union", "Former Soviet Union", "Former Soviet Union", "Former Soviet Union", "Former Soviet Union", "Former Soviet Union"),
  PercentageBelieveVaccinesSafe = c(30, 40, 50, 60, 70, 80, 90, 100, 30, 40, 50, 60, 70, 80, 90, 100, 30, 40, 50, 60, 70, 80, 90, 100, 30, 40, 50, 60, 70, 80, 90, 100, 30)
)

# You would continue to fill in the data for each country and their respective percentage

# Write the data to a CSV file
write.csv(data, "vaccine_belief_data.csv", row.names = FALSE)

# Print out the data to verify
print(data)

```

** first redesign code**

```{r}
library(plotly)
library(dplyr)
library(readr)


vaccine_data <- read_csv("vaccine_belief_data.csv")

# Define the unique regions, including an 'All' option
unique_regions <- unique(vaccine_data$Region)
all_regions <- c('All', unique_regions)

# Create a list for dropdown menu options
dropdown_buttons <- lapply(all_regions, function(region) {
  if (region == 'All') {
    return(list(method = "restyle",
                args = list('visible', rep(TRUE, length(unique_regions))),
                label = region))
  } else {
    return(list(method = "restyle",
                args = list('visible', 
                            lapply(unique_regions, function(x) x == region)),
                label = region))
  }
})

# Plotly does not natively support this kind of interaction in a straightforward way.
# So we prepare the data by creating a separate trace for each region.
plot_data <- lapply(unique_regions, function(region) {
  subset(vaccine_data, Region == region)
})

# Create the initial plot
p <- plot_ly()

# Add traces to the plot for each region
for(i in seq_along(plot_data)) {
  p <- add_trace(p, data = plot_data[[i]], x = ~Country, y = ~PercentageBelieveVaccinesSafe,
                 type = 'bar', name = unique_regions[i], visible = TRUE)
}

# Add the dropdown menu to the plot
final_plot <- p %>% layout(title = "Percentage of People Who Believe Vaccines Are Safe by Country and Global Region",
                           xaxis = list(title = "Country"),
                           yaxis = list(title = "Percentage Who Believe Vaccines Are Safe"),
                           updatemenus = list(
                             list(
                               active = -1,
                               buttons = dropdown_buttons
                             )
                           ))

# Show the plot
final_plot

```

** Second redesign code**

```{r}
library(shiny)
library(plotly)
library(readr)
library(dplyr)

# Define the UI
ui <- fluidPage(
  titlePanel("Vaccine Safety Belief by Country and Region"),
  sidebarLayout(
    sidebarPanel(
      numericInput("percentage", "Percentage threshold:", min = 0, max = 100, value = 50),
      textInput("country", "Country:", ""),
      actionButton("update", "Update View")
    ),
    mainPanel(plotlyOutput("plot"))
  )
)

# Define the server logic
server <- function(input, output) {
  
  # Reactive value for the filtered dataset
  filtered_data <- reactive({
    req(input$update)  # This makes sure the code waits for the update button to be pressed
    data <- read_csv("vaccine_belief_data.csv")
    data <- data %>% 
      filter(PercentageBelieveVaccinesSafe <= input$percentage,
             grepl(input$country, Country, ignore.case = TRUE))
  })
  
  # Render the Plotly plot
  output$plot <- renderPlotly({
    fig <- plot_ly(filtered_data(), x = ~PercentageBelieveVaccinesSafe, y = ~Country, type = 'scatter', mode = 'markers+text',
                   text = ~paste(Country, ':', PercentageBelieveVaccinesSafe, '%'), textposition = 'right',
                   marker = list(size = 10), color = ~Region, hoverinfo = 'text') %>%
      layout(title = 'Vaccine Safety Belief by Country and Region',
             xaxis = list(title = 'Percentage Believing Vaccines Are Safe'),
             yaxis = list(title = ''))
    fig
  })
  
  # Update the view when the button is pressed
  observeEvent(input$update, {
    filtered_data()
  })
}

# Run the application
shinyApp(ui = ui, server = server)

```
