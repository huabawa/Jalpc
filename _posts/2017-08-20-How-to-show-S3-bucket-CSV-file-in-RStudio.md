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
