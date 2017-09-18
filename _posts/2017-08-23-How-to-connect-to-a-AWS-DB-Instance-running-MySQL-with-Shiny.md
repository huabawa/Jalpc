## How to connect to an AWS DB instance running MySQL with Shiny

Here are the steps:
Go to Terminal.
1. ```cd /usr/local/mysql/bin```
2. ```./mysql -h <endpoint> -P <port> -u <user> -p```
Example: mysql -h myinstance.123456789012.us-east-1.rds.amazonaws.com -P 3306 -u mymasteruser -p
3. Enter the master user password when prompted.
A message starting with, Welcome to the MySQL monitor., will show to tell you that you have successfully connected to the DB instance.
4. Type ```SHOW databases;``` into Terminal.
5. ```Type USE database;```

For more information on how to do this, go to this link. http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html

### GRANT syntax
Granting all privileges on database to some user in the DB instance running MySQL

1. ```GRANT ALL ON database_name.* TO ‘user’@’hostname’;```
The hostname, if omitted, defaults to ‘%’. (information obtained from https://dev.mysql.com/doc/refman/5.7/en/grant.html) 
Example: GRANT ALL ON db1.* TO 'jeffrey'@'localhost';
2. Refresh the system. Type: ```flush privileges;```
3. Quit. Type: ```\q```
4. Type the following with the new user: ```./mysql -h <endpoint> -P <port> -u <user> -p```
5. Type the following to check that the database granted to the user is created: ```SHOW databases;```

### Show mysql tables on a webpage with Shiny
https://shiny.rstudio.com/articles/persistent-data-storage.html

I got the idea of displaying mysql tables with Shiny when my colleague, Jin, mentioned that this was possible during our discussion with our boss and mentor Dr. Rob on how to show energy data from UW in mysql on a webpage. Having had no experience with mysql or shiny before, I thought the simplest and fastest way to begin learning these two things was to get help from my handy-dandy friend, Google. I simply typed in shiny mysql into the google search bar and the first thing that showed up Shiny - Persistent data storage in Shiny apps was exactly the information that I needed. 

After reading the description on the webpage about how to connect to MySQL with R, my mind was telling me that I need to understand how to use MySQL before I even start using R, and so, I did. However, at the same time, I was wondering if I could find a demo site made already. Luckily, I found live demo of a Shiny app on the persistent data webpage, which was open-sourced on github. This gave me the idea to use the github code for my DB instance mysql data. Meanwhile, I was learning how to use mysql. Although I wanted to go through the mysql reference manual as fast as I could, I realized that I had to go through each step carefully, so that I wasn’t missing out any important steps along the way. As soon as I created a database and a table, I was ready to head over to R. 

The next step was to understand how Shiny and R worked together. To find help with answering this question, I went to google for help again. I typed shiny tutorial into google search and again I found the information that I was looking for, Shiny - Tutorial. Since I knew I was looking for a quick read and a not a video, I scrolled through the page to look for a written tutorial. This lead me to lesson 1 (https://shiny.rstudio.com/tutorial/lesson1/) in which I could quickly read about what I needed to know in order to create a shiny webpage. Once I understood the architecture of a shiny app, I went back to the github page of the demo site to implement my own MySQL database.

Here is the demo site's github source code:

https://github.com/daattali/shiny-server/tree/master/persistent-data-storage

In order to use the github source code, you need to enter your own host, port, user, and password in storage.R.
```R
  "host" = "", # ex. ooo-sth.fdsafdsaa.us-west-2.rds.amazonaws.com
  "port" = , # 3306
  "user" = "", # "root"
  "password" = ""
``` 
Here is the full storage.R code:
```R
library(RMySQL)
library(shiny)

options(mysql = list(
  "host" = "",
  "port" = 3306,
  "user" = "root",
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
```
When you're done, you should be able to see the MySQL table that you created.

## How to Show Your Data in the MySQL database in a timeseries graph in Shiny

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
  
  
  # renderPlotly() also understands ggplot2 objects!
  output$plot <- renderPlotly({
    plot_ly(data, x = ~, y = ~, type = "scatter")
  })
  
  output$event <- renderPrint({
    d <- event_data("plotly_hover")
    if (is.null(d)) "Hover on a point!" else d
  })
}

shinyApp(ui, server)

  



## How to show S3 bucket CSV file in RStudio

Type the following into RStudio
```
library("aws.s3")

Sys.setenv("AWS_ACCESS_KEY_ID" = "", "AWS_SECRET_ACCESS_KEY" = "")

usercsvobj <-get_object("s3://directory to csv")

csvcharobj <- rawToChar(usercsvobj)
con <- textConnection(csvcharobj)
data <- read.csv(con)
close(con)
data
```

Useful links:

Connecting to a DB Instance Running the MySQL Database Engine
http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html

https://shiny.rstudio.com/articles/persistent-data-storage.html

https://shiny.rstudio.com/tutorial/lesson1/
