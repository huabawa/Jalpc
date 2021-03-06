# Connecting to AWS DB instance running MySQL with Shiny

In this tutorial, I will be showing you how to connect an RDS MySQL instance to Rstudio and display your data on a Shiny webpage. I will also be showing you how to display your S3 bucket CSV data on a Shiny webpage, how to display data on an online Shiny dashboard, and how to create a timeseries graph with Shiny. Before we get into R programming, you need to understand the basics of MySQL and, most importantly, know what Shiny is. 

**What is Shiny?**

Shiny is dynamic web pages built on R.

Before you start this tutorial, make sure that you have MySQL community version installed. If you have not installed it, please go to this [link](https://www.mysql.com/products/community/), to install it. If you don't have any experience in MySQL or R, this is the right tutorial for you, but if you do, that's great! When I started writing this tutorial, I had absolutely no experience in MySQL and R, so if I can understand MySQL and R, so can you. 

You should also install RStudio from [here](https://www.rstudio.com/products/rstudio/download/).
Download and install this free version, RStudio Desktop Open Source License.

After you have installed the MySQL community version and RStudio, follow these steps to access your RDS MySQL.

**Here are the steps:**

Go to Terminal.
1. ```cd directory to mysql folder/bin```
2. ```./mysql -h <endpoint> -P <port> -u <user> -p```
Example: mysql -h myinstance.123456789012.us-east-1.rds.amazonaws.com -P 3306 -u mymasteruser -p
3. Enter the master user password when prompted.
A message starting with, Welcome to the MySQL monitor., will show to tell you that you have successfully connected to the DB instance.
4. Type ```SHOW databases;``` into Terminal.
5. Type ```USE nameOfDatabase;``` into Terminal.

For more information on how to do this, go to this [link](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html).

The following information is not required to understand how to do the rest of this tutorial. You can skip this part, **GRANT syntax**, if you would like.

### GRANT syntax
Granting all privileges on database to some user in the DB instance running MySQL

1. ```GRANT ALL ON databaseName.* TO ‘user’@’hostname’;```
The hostname, if omitted, defaults to ‘%’. (information obtained from [here](https://dev.mysql.com/doc/refman/5.7/en/grant.html)) 
Example: GRANT ALL ON db1.* TO 'jeffrey'@'localhost';
2. Refresh the system. Type: ```flush privileges;```
3. Quit. Type: ```\q```
4. Type the following with the new user: ```./mysql -h <endpoint> -P <port> -u <user> -p```
5. Type the following to check that the database granted to the user is created: ```SHOW databases;```

## Introduction to Showing MySQL tables in a Shiny webpage

I got the idea of displaying mysql tables with Shiny when my colleague mentioned that this was possible during our discussion with our boss on how to show energy data from the UW in mysql on a webpage. Having had no experience with mysql or shiny before, I thought the simplest and fastest way to begin learning these two things was to get help from my handy-dandy friend, Google. I simply typed in Shiny mysql into the google search bar and the first thing that showed up, Shiny - Persistent data storage in Shiny apps, was exactly the information that I needed. 

After reading the description on the webpage about how to connect to MySQL in RStudio, my mind was telling me that I need to understand how to use MySQL before I even start using R, and so, I did. However, at the same time, I was wondering if I could find a demo site made already. Luckily, I found a live demo of a Shiny app on the persistent data webpage, which was open-sourced on github. This gave me the idea to use the github code for my DB instance mysql data. Meanwhile, I was learning how to use mysql. Although I wanted to go through the mysql reference manual as fast as I could, I realized that I had to go through each step carefully, so that I wasn’t missing out any important steps along the way. As soon as I created a database and a table, I was ready to head over to RStudio. 

The next step was to understand how Shiny and R worked together. To get the help that I needed to answer this question, I went to google for help again. I typed Shiny tutorial into google search and again I found the information that I was looking for, Shiny - Tutorial. Since I knew I was looking for a quick read and not a video, I scrolled through the page to look for a written tutorial. This lead me to Shiny tutorial [lesson 1](https://shiny.rstudio.com/tutorial/written-tutorial/lesson1/), which allowed me to quickly read about what I needed to know in order to create a Shiny webpage. Once I understood the architecture of a shiny app, I went back to the github page of the demo site to implement my own MySQL database.

[Here](https://github.com/daattali/shiny-server/tree/master/persistent-data-storage) is the demo site's github source code.

## How to display a MySQL table on a Shiny webpage
In order to use the github source code, you need to enter your own host, port, user, and password in storage.R.
*Note: The following code is obtained from [here](https://shiny.rstudio.com/articles/persistent-data-storage.html#mysql).*
**WARNING: The host, port, user, and password in the code SHOULD BE DELETED when you upload this code to github or anywhere else online**

1. Make a copy of the github folder persistent-data-storage on your computer.
2. Delete everything in storage.R. Copy and paste the following code into storage.R.

```R
library(RMySQL)

options(mysql = list(
  "host" = "127.0.0.1",
  "port" = 3306,
  "user" = "myuser",
  "password" = "mypassword"
))
databaseName <- "myshinydatabase"
table <- "responses"

saveData <- function(data) {
  # Connect to the database
  db <- dbConnect(MySQL(), dbname = databaseName, host = options()$mysql$host, 
      port = options()$mysql$port, user = options()$mysql$user, 
      password = options()$mysql$password)
  # Construct the update query by looping over the data fields
  query <- sprintf(
    "INSERT INTO %s (%s) VALUES ('%s')",
    table, 
    paste(names(data), collapse = ", "),
    paste(data, collapse = "', '")
  )
  # Submit the update query and disconnect
  dbGetQuery(db, query)
  dbDisconnect(db)
}

loadData <- function() {
  # Connect to the database
  db <- dbConnect(MySQL(), dbname = databaseName, host = options()$mysql$host, 
      port = options()$mysql$port, user = options()$mysql$user, 
      password = options()$mysql$password)
  # Construct the fetching query
  query <- sprintf("SELECT * FROM %s", table)
  # Submit the fetch query and disconnect
  data <- dbGetQuery(db, query)
  dbDisconnect(db)
  data
}
```
3. When you run ui.R, you should be able to see the MySQL table on the demo Shiny webpage.

## How to display S3 bucket CSV file data on a Shiny webpage 
**WARNING: DO NOT upload this code to github or anywhere else online WITH YOUR AWS ACCESS KEY ID and AWS SECRET ACCESS KEY**
**!!! This part of the code is for storage.R in the github demo source code. But, you could use it in your github webpage, if you'd like.**
*Note: The following code is obtained from [here](https://shiny.rstudio.com/articles/persistent-data-storage.html#mysql).*

1. Do the same thing as the instructions in the last section for MySQL

2. Do the same thing as the instructions in the last section for MySQL, except, copy and paste the following code into storage.R.

```R

library(aws.s3)

s3BucketName <- "my-unique-s3-bucket-name"
Sys.setenv("AWS_ACCESS_KEY_ID" = "key",
           "AWS_SECRET_ACCESS_KEY" = "secret",
           "AWS_DEFAULT_REGION" = "region")

saveData <- function(data) {
  # Create a temporary file to hold the data
  data <- t(data)
  file_name <- paste0(
    paste(
      get_time_human(),
      digest(data, algo = "md5"),
      sep = "_"
    ),
    ".csv"
  )
  file_path <- file.path(tempdir(), file_name)
  write.csv(data ,file_path, row.names = FALSE, quote = TRUE)

  # Upload the file to S3
  put_object(file = file_path, object = file_name, bucket = s3BucketName)
}

loadData <- function() {
  # Get a list of all files
  file_names <- get_bucket_df(s3BucketName)[["Key"]]
  # Read all files into a list
  data <- lapply(file_names, function(x) {
    object <- get_object(x, s3BucketName)
    object_data <- readBin(object, "character")
    read.csv(text = object_data, stringsAsFactors = FALSE)
  })
  # Concatenate all data together into one data.frame
  data <- do.call(rbind, data)
  data  
}
```
3. Run ui.R.

## How to display a MySQL data timeseries graph on a Shiny webpage
**WARNING: The host, port, user, and password in the code SHOULD BE DELETED when you upload this code to github or anywhere else online**

```R
library(shiny)
library(RMySQL)
library(plotly)

ui <- fluidPage(
  plotlyOutput("plot"),
  verbatimTextOutput("event")
)

server <- function(input, output) {
  library(RMySQL)
  library(plotly)
  options(mysql = list(
    "host" = "",
    "port" = 0000,
    "user" = "",
    "password" = ""
  ))
  
  DB_NAME <- ""
  TABLE_NAME <- ""
  
  save_data_mysql <- function(data) {
    db <- dbConnect(MySQL(), dbname = DB_NAME,
                    host = options()$mysql$host,
                    port = options()$mysql$port,
                    user = options()$mysql$user,
                    password = options()$mysql$password)
    query <-
      sprintf("INSERT INTO %s (%s) VALUES ('%s')",
              TABLE_NAME,
              paste(names(data), collapse = ", "),
              paste(data, collapse = "', '")
      )
    dbGetQuery(db, query)
    dbDisconnect(db)
  }
  
  load_data_mysql <- function() {
    db <- dbConnect(MySQL(), dbname = DB_NAME,
                    host = options()$mysql$host,
                    port = options()$mysql$port,
                    user = options()$mysql$user,
                    password = options()$mysql$password)
    query <- sprintf("SELECT * FROM %s", TABLE_NAME)
    data <- dbGetQuery(db, query)
    dbDisconnect(db)
    
    data 
  }
  
  
  # renderPlotly() also understands ggplot2 objects!
  output$plot <- renderPlotly({
    plot_ly(data, x = ~, y = ~, type = "scatter") # type an x and y variable from your table for x and y after the ~.
  })
  
  output$event <- renderPrint({
    d <- event_data("plotly_hover")
    if (is.null(d)) "Hover on a point!" else d
  })
}

shinyApp(ui, server)
```
## How to show S3 bucket CSV file in RStudio
**WARNING: DO NOT upload this code to github or anywhere else online WITH YOUR AWS ACCESS KEY ID and AWS SECRET ACCESS KEY**
```R
library("aws.s3")

Sys.setenv("AWS_ACCESS_KEY_ID" = "", "AWS_SECRET_ACCESS_KEY" = "")

usercsvobj <-get_object("s3://directory to csv")

csvcharobj <- rawToChar(usercsvobj)
con <- textConnection(csvcharobj)
data <- read.csv(con)
close(con)
data
```
## How to display your MySQL table and graph on a Shiny dashboard
**WARNING: DO NOT upload this code online with your HOST, PORT, USER, and PASSWORD**
```R
library(shiny)
library(shinydashboard)
library(ggplot2)
library(RMySQL)

options(mysql = list(
  "host" = "",
  "port" = ,
  "user" = "",
  "password" = ""
))

DB_NAME <- ""
TABLE_NAME <- ""

save_data_mysql <- function(data) {
  db <- dbConnect(MySQL(), dbname = DB_NAME,
                  host = options()$mysql$host,
                  port = options()$mysql$port,
                  user = options()$mysql$user,
                  password = options()$mysql$password)
  query <-
    sprintf("INSERT INTO %s (%s) VALUES ('%s')",
            TABLE_NAME,
            paste(names(data), collapse = ", "),
            paste(data, collapse = "', '")
    )
  dbGetQuery(db, query)
  dbDisconnect(db)
}
load_data_mysql <- function() {
  db <- dbConnect(MySQL(), dbname = DB_NAME,
                  host = options()$mysql$host,
                  port = options()$mysql$port,
                  user = options()$mysql$user,
                  password = options()$mysql$password)
  query <- sprintf("SELECT * FROM %s", TABLE_NAME)
  data <- dbGetQuery(db, query)
  dbDisconnect(db)
  
  data
}

ui <- dashboardPage(
  dashboardHeader(title = "My Dashboard"),
  sidebar <- dashboardSidebar(),
  body <- dashboardBody(
    box (
      plotlyOutput("plot"),
      verbatimTextOutput("event")
    ),
    box(title = "Histogram", status = "primary", solidHeader = TRUE,
        collapsible = TRUE,
      fluidPage(
      titlePanel("Basic DataTable"),
      
      # Create a new Row in the UI for selectInputs
      fluidRow(
        column(4,
               selectInput("dealer", # select an input from your table
                           "dealer:",
                           c("All",
                             unique(as.character(data$date))))
        ),
        column(4,
               selectInput("price", # select an input from your table
                           "Price:",
                           c("All",
                             unique(as.character(data$price))))
        )
      ),
      # Create a new row for the table.
      fluidRow(
        DT::dataTableOutput("table")
      )
    ))
    
  )
)

server <- function(input, output) {
  output$table <- DT::renderDataTable(DT::datatable({
    data <- data
    if (input$dealer != "All") { # select an input from your table
      data <- data[data$date == input$date,]
    }
    if (input$price != "All") { # select an input from your table
      data <- data[data$price == input$price,]
    }
    data
  }))
  
  output$plot <- renderPlotly({ 
    plot_ly(data, x = ~dealer, y = ~price, type = "scatter") # select x and y from your table
  })
  
  output$event <- renderPrint({
    d <- event_data("plotly_hover")
  })
}

shinyApp(ui, server)
```

#### Useful links:

Connecting to a DB Instance Running the MySQL Database Engine
http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html

https://shiny.rstudio.com/articles/persistent-data-storage.html

https://shiny.rstudio.com/tutorial/lesson1/
