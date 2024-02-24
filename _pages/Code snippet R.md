<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>R Code Examples</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.4.0/styles/default.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.4.0/highlight.min.js"></script>
</head>
<body>

<h2>Creating a Data Set</h2>
<pre><code class="language-r"># Load necessary library
library(tibble)

# Manually enter the data
data &lt;- tibble(
  Country = c("Japan", "South Korea", "Taiwan", "New Zealand", "China", "Bangladesh", "India", "Egypt", "Iraq", "Algeria", "Iran", "Libya", "Gabon", "Mauritius", "Togo", "Rwanda", "Ethiopia", "Liberia", "Venezuela", "Nicaragua", "United States", "UK", "Norway", "Northern Cyprus", "Montenegro", "France", "Switzerland", "Russia", "Ukraine", "Belarus", "Kyrgyzstan", "Uzbekistan", "Turkmenistan"),
  Region = c("Asia", "Asia", "Asia", "Asia", "Asia", "Asia", "Asia", "Middle East and North Africa", "Middle East and North Africa", "Middle East and North Africa", "Middle East and North Africa", "Middle East and North Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Sub-Saharan Africa", "Americas", "Americas", "Americas", "Europe", "Europe", "Europe", "Europe", "Europe", "Europe", "Former Soviet Union", "Former Soviet Union", "Former Soviet Union", "Former Soviet Union", "Former Soviet Union", "Former Soviet Union"),
  PercentageBelieveVaccinesSafe = c(30, 40, 50, 60, 70, 80, 90, 100, 30, 40, 50, 60, 70, 80, 90, 100, 30, 40, 50, 60, 70, 80, 90, 100, 30, 40, 50, 60, 70, 80, 90, 100, 30)
)

# Write the data to a CSV file
write.csv(data, "vaccine_belief_data.csv", row.names = FALSE)

# Print out the data to verify
print(data)
</code></pre>

<h2>First Redesign Code</h2>
<pre><code class="language-r">library(plotly)
library(dplyr)
library(readr)

vaccine_data &lt;- read_csv("vaccine_belief_data.csv")

# Define the unique regions, including an 'All' option
unique_regions &lt;- unique(vaccine_data$Region)
all_regions &lt;- c('All', unique_regions)

# Create a list for dropdown menu options
dropdown_buttons &lt;- lapply(all_regions, function(region) {
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
plot_data &lt;- lapply(unique_regions, function(region) {
  subset(vaccine_data, Region == region)
})

# Create the initial plot
p &lt;- plot_ly()

# Add traces to the plot for each region
for(i in seq_along(plot_data)) {
  p &lt;- add_trace(p, data = plot_data[[i]], x = ~Country, y = ~PercentageBelieveVaccinesSafe,
                 type = 'bar', name = unique_regions[i], visible = TRUE)
}

# Add the dropdown menu to the plot
final_plot &lt;- p %>% layout(title = "Percentage of People Who Believe Vaccines Are Safe by Country and Global Region",
                           xaxis = list(title = "Country"),
                           yaxis
