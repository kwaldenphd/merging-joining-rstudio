# Merging and joining data in RStudio using `dplyr`

<a href="http://creativecommons.org/licenses/by-nc/4.0/" rel="license"><img style="border-width: 0;" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" alt="Creative Commons License" /></a>
This tutorial is licensed under a <a href="http://creativecommons.org/licenses/by-nc/4.0/" rel="license">Creative Commons Attribution-NonCommercial 4.0 International License</a>.

## Lab Goals

There are many situations where data sets are broken into multiple tables, and multiple reasons that this might make sense—sometimes it is easier to collect data in different pieces, other times it is to reduce the file size. Regardless of the reason for splitting data sets into multiple tables, they should always formatted in such a way that there is at least one common column between the tables so that they can be merged as needed. In this example we will explore how to use the join suite of functions found in the dplyr package to merge these data tables.

Note: In R, most tabular sets of data are called data frames. Data frames can store objects of different classes (e.g. some columns can be text and other columns can be integers). A synonymous term is data table, which is used by some textbooks and other languages, such as the structued query language (SQL).

## Acknowledgements

This lab procedure is adapted from and based on Ryan Miller's ["Introduction to Merging Data Tables in R"](https://remiller1450.github.io/s230f19/merging.html) (Fall 2019, Intro to Data Science STA 230 course, Grinnell College).

# Table of Contents

# Data and Environment Setup

To begin, let’s make sure that our sample data and necessary packages are loaded:
```R
# install.packages("dplyr")
library(dplyr)
```

To illustrate the different join functions we will use a small example of a customer database. 

We will focus on two tables: 
- `orders` contains the order number, customer ID, and date of the order
- `customers` contains the customer ID and customer name. 

These are intentionally small data tables so that is easier to see how the join statements are working.

```R
orders <- read.csv("https://raw.githubusercontent.com/kwaldenphd/main/data/orders.csv", as.is = TRUE)
orders
```

```R
customers <- read.csv("https://raw.githubusercontent.com/kwaldenphd/main/data/customers.csv", as.is = TRUE)
customers
```

Google Drive link if needed: https://drive.google.com/drive/folders/1u4_Yk4DJ1M0J6kFisJecCb1KJ6SxPgMu?usp=sharing

# Joining Data Tables

117- According to W3Schools'[SQL Joins page](https://www.w3schools.com/sql/sql_join.asp), "A JOIN clause is used to combine rows from two or more tables, based on a related column between them."

FIGURE 1

Image credit: [Tweet by Hiroaki Yutani](https://twitter.com/yutannihilation/status/551572539697143808?s=20) @yutannihilation (3 January 2015)

120- There are four main types of `join` functions.
- `inner_join` returns matching records in both tables
- `left_join` returns all records from the left table and only matching records from the right table
- `right_join` returns all records from the right table and only matching records from the left table
- `full_join` returns all matching records from both the left and right tables

We can express these `JOIN` functions programmatically in R.

The `dplyr` package provides six different join functions, each merges two data tables together in a different way. 

The best way to understand these functions is to see how each works in our small example.

All of the functions have the same three arguments:

argument | explanation
--- | ---
`x` | the first (left) table to join
`y` | the second (right) table to join
`by` | a character vector of variables to join by; notice the column name must always be in quotes

## `inner_join`

`inner_join` creates a new table which is restricted to cases where the values of “by variable” exist in both data sets. 

All columns from both data sets are returned for these cases.

```R
inner_join(x = orders, y = customers, by = "id")
```

In this `inner_join`, we lose the row for order 4 from the `orders` table because customer 50 does not appear in the `customers` data table. 

We also lose orders 15, 16 and 23 from the `customers` table. 

Note the columns `order`, `date`, and `name` are also returned.

## `left_join`

`left_join` returns all cases from the `x` data table, regardless of whether there are matching values of the by variable(s) in `y`. 

All columns from both data tables are returned for these cases.

```R
left_join(x = orders, y = customers, by = "id")
```

This new data frame now includes the `name` column.

Since customer 50 does not appear in the customers data table, an `NA` (missing value) is used for their name.

## `right_join`

`right_join` returns all cases from the `y` data table, regardless of whether there are matching values of the by variable(s) in `x`. 

All columns from both data tables are returned for these cases.

```R
right_join(x = orders, y = customers, by = "id")
```

We have added the `order` and `date` columns to the `customers` data table. 

Customers 15, 16, and 23 did not make purchases during this time frame, so missing values (NAs) are used for their `order` and `date` values.

## `full_join`

`full_join` returns all rows and columns from both `x` and `y`.

```R
full_join(x = orders, y = customers, by = "id")
```

We have fully merged the `orders` and `customers` data tables.

Thus, we get all of the columns and all of the rows from both data tables. 

`NA`s fill in the necessary values for customers not making purchases and for orders without a customer record.

## `semi_join`

`semi_join` returns all rows from the `x` data table where there are matching values of the by variable(s) in `y`, and only the columns from `x`.

```R
semi_join(x = orders, y = customers, by = "id")
```

We lose the row for order 4 because customer 50 does not appear in the `customers` data table.

## `inner_join` versus `semi_join`

In the examples above, the `inner_join` and `semi_join` returned the same number of rows.

This will not always be the case. 

For example, suppose that customer 42 also placed an order on May-01 so that we have multiple orders from the same customer.

A few examples that illustrate how this might occur:

```R
extra_order <- data.frame(order = 5, id = 42, date = "May-01")
extra_order
```

```R
orders2 <- rbind(orders, extra_order)
orders2
```

```R
inner_join(x = customers, y = orders2, by = "id")
```

```R
semi_join(x = customers, y = orders2, by = "id")
```

The result of the `inner_join` includes two rows for customer 42 because `inner_join` returns all of the columns from both data tables for `id`s that match. 

The result of the `semi_join` only returns one row for each customer because it only returns the rows from customers that have matching `id`s in `orders2`.

## `anti_join`

`anti_join` returns all rows from the `x` data table where there are not matching values of the `by` variable(s) in `y`, and only the columns from `x`.

```R
anti_join(x = customers, y = orders, by = "id")
```
Customers 15, 16, and 23 did not place orders during this time frame, so their entries from the `customers` data table are returned.

# Common Complications

All of our examples have only used a single column to match the entries between the data tables.

These examples also assumed that the columns will have identical names. 

This will not always be the case. 

Below we detail how to refine what variables you merge by.

If you want to join by multiple variables, then you need to specify a vector of variable names: 
```R
by = c("var1", "var2", "var3")
```

Here all three columns must match in both tables.

If you want to use all variables that appear in both tables, then you can leave the `by` argument blank.

If the variable you wish to join by is not named identically in both tables, then you specify `by = c("left_var" = "right_var")`

Another issue that crops up occasionally is duplicate entries in the variable(s) that you wish to merge by. 

We saw one example of this above when there were two orders from the same customer. 

In that case the `id` value was unique in the `customer` table, but not in the `orders` table. 

The result of this join is quite logical, as seen above. 

If, however, both tables contain duplicate entries in the variable(s) that you wish to merge by, all possible combinations of these entries are returned. 

A simple example for a `full_join` is shown below:
```R
# Creating example data frames
table1 <- data.frame(key = c("a", "a", "b", "b", "c"), var = 1:5)
table1
```

```R
# create new table
table2 <- data.frame(key = c("a", "a", "b", "b", "c"), var = LETTERS[1:5])
table2
```

```R
# full join
full_join(x = table1, y = table2, by = "key")
```

In this situation, the results for `left_join`, `right_join`, and `full_join` will be identical.

# Practice Problems

Question #1:

The files books.csv, authors.csv, and book-authors.csv give details about the planned summer reading of a statistics student. You can find the files at:
- https://raw.githubusercontent.com/kwaldenphd/main/data/books.csv
- https://raw.githubusercontent.com/kwaldenphd/main/data/authors.csv
- https://raw.githubusercontent.com/kwaldenphd/main/data/book-authors.csv

books.csv provides details for each book (isbn, title, year, and genre), authors.csv provides details about each author (authorid, first name, last name, and nationality), and book-authors.csv provides the author identification (authorid) for each isbn (books with multiple authors will have multiple rows).

- A. Read the three files into R, naming them books, authors, and book_authors.
- B. Use the appropriate join statement to add the ISBNs to the authors data table. Why does the resulting data frame have 31 rows instead of 11?
- C. Eliminate the duplicate rows of your data frame from Part B (which we’ll assume you named df) run the following code (change the object names to align with your code as necessary): `df2 <- unique(df)`
- D. Use the appropriate join statement to add the updated author information from Part B/C to the books data table.
- E. Are there any authors in the authors data table that do not correspond to books in the books data table? Use an appropriate join statement to determine this.

Question #2:

The nycflights13 package contains a database of all flights departing New York City in 2013.
- A. Write code that adds weather information to each record in the flights table. You should match each flight with the nearest weather recording at that airport, meaning you should use the provided code to get the hour nearest to when the flight departs.
- B. Write code to print the dimensions of the flights data.frame before and after the merge. Should they be the same? Briefly explain in a comment.

```R
library(nycflights13)
data("flights")
flights$nearest_hour <- round(flights$dep_time, -2)/100
```

# Additional Resources

[RStudio’s data wrangling cheat sheet](https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf) provides a nice summary of how to combine data sets.

[`dplyr` vignette on two table verbs](https://cran.r-project.org/web/packages/dplyr/vignettes/two-table.html) provides additional examples of merging data tables.

[William Surles course on joining using dplyr](https://rpubs.com/williamsurles/293454) (contains several extra commands and examples)


# Lab Notebook Questions
