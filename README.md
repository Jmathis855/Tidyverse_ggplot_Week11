# Introduction to Tidyverse and ggplot

## What are the benefits of using packages in R?

- As we learned last week, R is a very powerful tool for data management, analysis, and visualization. Many different types of analyses and visualizations can be done natively in R with no other packages (base R).
- However, as datasets get larger and more complex, we sometimes need tools not available in base R to help us with our analyses and visualizations in the form of packages designed by other scientists and researchers.
- One such group of packages that has become widely used across many disciplines is called `tidyverse`, and it consists of many interrelated packages that all share the goal of facilitating efficient, easy to read, and well organized code.
CRAN - The Comprehensive R Archive Network 
 ## Installing tidyverse

 To install all packages associated with tidyverse, the following commands can be run in your R session.

 ```
install.packages("tidyverse")
library(tidyverse)
tidyverse_update()
```

## Reading in and manipulating data with tidyverse

The basic functionality of tidyverse is reading in and manipulating data. The following functions can be used to accomplish this:

- `read_csv()` for comma-separated files
- `read_csv2()` for semicolon-separated files
- `read_tsv()` for tab-separated files
- `read_delim()` for a file separated by any other delimiter, e.g. `delim = "$"` for a $ delimiter

To practice reading in and manipulating data files using tidyverse, download the "2023_mpg.csv" data file available in this repository. This dataset contains information about the fuel economy of cars from various car manufacturers. Make sure that you have the tidyverse package loaded and your working directory set to wherever the data file is located (`setwd("~/YourDirectory")`).

First, we read in the data file.

```
mpg <- read_csv("2023_mpg.csv")
```

This reads in our data file in a format known as a **tibble**, which is similar to the data frame format we discussed last week. To get an idea for the data in this file, we can call it's variable name to view the first few columns and rows (similar to using the head command).

```
mpg
```

As you can see, this is a very complex data file with many different columns and types of data. One thing we may want to do is pare down the dataset to remove columns we aren't interested in. We can do this using the `select()` function available through tidyverse. Here, we pass the select function the variable that stores our dataset, as well as the names of the columns we want to retain. This can be done in many different ways to preserve different combinations of columns:

- `select(mpg, "Model Year":"Carline")` preserves every column including and in-between those specified
- `select(mpg, -Carline)` preserves every column except the one specified
- The previous two can also be combined, such as `select(mpg, -("Model Year":"Carline"))` to preserve every column except those including and in-between those specified

***Note***: **Quotation marks are not necessary to specify column names except for when the column name contains a space (e.g. Model Year)**

Let's use the select function to create a new dataset focusing on a few columns of interest. We can do this by calling the select function and assigning it to a new variable, such as:

```
mpg2 <- select(mpg, "Mfr Name", "Carline", "# Cyl", "City FE (Guide) - Conventional Fuel", "Hwy FE (Guide) - Conventional Fuel")
```

This will create a new dataset containing only the columns that specify the car manufacturer's name, the car model, the number of engine cylinders, and the car's city and highway fuel economy. To take a look at the new dataset, we can again call its variable name.

```
mpg2
```

Now that we've created a smaller, more focused dataset, we may want to alter it to facilitate easier analysis later. One thing you may have noticed is that some of our column names are a bit unwieldy, and this can make using and referencing them a bit more difficult. To fix this, we can rename our columns. Renaming our columns requires the use of a special type of operator, `%>%`. Within tidyverse, the `%>%` operator acts the same as the pipe (`|`) operator does in bash. Put simply, it takes whatever is on the right and uses it as input for whatever is on the left. Let's use the `%>%` operator and the `rename()` function to edit the column names in our smaller dataset. Using the `rename()` function, we assign the old column names to new names like so:

```
mpg2 %>% rename(Make = "Mfr Name", Model = "Carline", NumCyl = "# Cyl", CityFE = "City FE (Guide) - Conventional Fuel", HwyFE = "Hwy FE (Guide) - Conventional Fuel")
```

The above command will output a tibble with the updated column names, but will not change the column names in the mpg2 variable. If we want the new column names to be reflected in our dataset, we assign the above command to a new, or the same variable. For example:

```
mpg2 = mpg2 %>% rename(Make = "Mfr Name", Model = "Carline", NumCyl = "# Cyl", CityFE = "City FE (Guide) - Conventional Fuel", HwyFE = "Hwy FE (Guide) - Conventional Fuel")
```

We can now call our variable name again to view the dataset with updated column names:

```
mpg2
```

We can also add a new column to our data using the `%>%` operator and the `mutate()` function. In this case, perhaps we're interested in adding a column that represents the difference between Highway and City fuel economy. This can be done like so:

```
mpg2 <- mpg2 %>% mutate(HwyCityDiff = (HwyFE - CityFE))
```

Often when analyzing datasets, we may want to summarize the data using a summary statistic, and tidyverse provides a helpful way to do this by allowing us to group our data. For example, if we want to calculate the average City and Highway fuel economy of each auto manufacturer (Make), we can do the following:

```
avg_mpg <- mpg2 %>% group_by(NumCyl) %>% summarize(avgCityFE = mean(CityFE), avgHwyFE = mean(HwyFE)) %>% arrange(NumCyl)
```

The above command calculates the average city and highway fuel economy of each engine configuration (number of cylinders), arranges them by lowest to highest number of engine cylinders, and stores this as a new tibble named `avg_mpg`. If we wanted to arrange them from, say, lowest to highest average city fuel economy, we could instead do `arrange(AvgCityFE)`

## Visualizing data with ggplot

Like we learned last week, many visualizations are possible in base R. However, similarly to tidyverse, many data scientists have turned to a package known as `ggplot2` for assistance in creating clear, highly customizable, and publication ready figures. When combined, tidyverse and ggplot form an extremely powerful way to both wrangle and visualize data. To demonstrate this, we'll be using the `avg_mpg` dataset that we created earlier using tidyverse.

Firstly, we need to install and load ggplot. This can be done with the following commands:

```
install.packages("ggplot2")
library(ggplot2)
```

Ggplot operates using a common syntax to specify the different parts of a plot. Some of these include:

- **Data** - This is the dataset that we want to use in our visualization.
- **Mapping** - This is how we tell ggplot which part of our dataset will be mapped to a particular part of the graph, e.g. the x and y axes. In ggplot2, we specify this using the `aes()` function.
- **Geometry** - ggplot can create many different types of plots, and these are specified by indicating the type of "geometry" (geom) that we want our plot to take (e.g. line graph, scatter plot, bar graph, etc.)

To demonstrate the syntax used by ggplot, let's create a basic plot using the `avg_mpg` dataset that we created previously. In this example, we want to plot number of cylinders on the x-axis and average city fuel economy on the y-axis. We can do this with the following command:

```
p = ggplot(data = avg_mpg, mapping = aes(x = NumCyl, y = avgCityFE)) + geom_point()
show(p)
```

In the above command, `data` specifies our dataset, and `mapping` allows us to specify the x and y axes through the `aes()` function. Note that the first part of this command, encompassed by the `ggplot()` function, only tells ggplot what our data is and how to map it. In order to tell ggplot how we want our data to be visualized, we have to specify the geometry. Here, we use `+ geom_point()` to tell ggplot that we want our data visualized as a scatter plot. If we wanted to, say, visualize our data as a line graph, we could instead do `+ geom_line()`. Many different types of plots are available in ggplot, and it's recommended that you explore some of them on your own in the [ggplot documentation](https://ggplot2.tidyverse.org/index.html). Here, we store this whole command as a variable. This will allows us to easily add on to it in the future.

Ggplot also allows us to do statistical tests using our data and overlay these on our plots. Using the above example, we may want to see if there is a correlation between number of engine cylinders and average city fuel economy. To do this, we can add another geometry to our plot variable:

```
p = p + geom_smooth(method = "lm")
show(p)
```

Here, the `geom_smooth()` function tries to smooth our data using some method, and we tell ggplot to accomplish this smoothing using a linear model (`method = "lm"`). When you run the above command, you'll see that your previously made scatter plot is overlayed with a best-fitting linear regression line complete with standard error ranges (to not plot the standard error, we can specify `se = FALSE` in the `geom_smooth()` function).

Now we have a plot that shows the negative correlation between number of engine cylinders and city fuel economy! However, the plot still looks a little rough, as the axis titles are being taken straight from our dataset. To change these, we can add the `xlab()` and `ylab()` functions to our plot variable.

```
p + xlab("Number of Cylinders") + ylab("Average City Fuel Economy (mpg)")
show(p)
```

Now our axes have more informative titles! We may also want to alter the appearance of our plot, and this can be done largely using themes. For example, adding ` + theme_bw()` to our plot variable will give us a cleaner looking black and white theme without the gray grid in the background. Many different types of themes are available through ggplot, and here again we recommend exploring these on your own in the [ggplot documentation](https://ggplot2-book.org/themes).

