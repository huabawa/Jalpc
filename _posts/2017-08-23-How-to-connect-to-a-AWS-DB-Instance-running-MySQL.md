How to connect to a AWS DB instance running MySQL

1. cd /usr/loca/mysql/bin
2. ./mysql -h <endpoint> -P <port> -u <user> -p
Example: mysql -h myinstance.123456789012.us-east-1.rds.amazonaws.com -P 3306 -u mymasteruser -p
3. Enter the master user password when prompted.
A message starting with, Welcome to the MySQL monitor., will show to tell you that you have successfully connected to the DB instance.
4. Type SHOW databases; into command prompt.
5. Type USE database;

GRANT syntax
Granting all privileges on database to some user in the DB instance MySQL

1. GRANT ALL ON database_name.* TO ‘user’@’hostname’;
The hostname, if omitted, defaults to ‘%’. (information obtained from https://dev.mysql.com/doc/refman/5.7/en/grant.html) 
Example: GRANT ALL ON db1.* TO 'jeffrey'@'localhost';
2. Refresh the system. Type: flush privileges; 
3. Quit. Type: \q
4. Type the following with the new user: ./mysql -h <endpoint> -P <port> -u <user> -p
5. Type the following to check that the database granted to the user is created: SHOW databases;

Show mysql tables on a webpage with Shiny https://shiny.rstudio.com/articles/persistent-data-storage.html

I got the idea of displaying mysql tables with Shiny when my colleague, Jin, mentioned that this was possible during our discussion with our boss and mentor Dr. Rob on how to show energy data from UW in mysql on a webpage. Having had no experience with mysql or shiny before, I thought the simplest and fastest way to begin learning these two things was to get help from my handy-dandy friend, Google. I simply typed in shiny mysql into the google search bar and the first thing that showed up Shiny - Persistent data storage in Shiny apps was exactly the information that I needed. 

After reading the description on the webpage about how to connect to MySQL with R, my mind was telling me that I need to understand how to use MySQL before I even start using R, and so, I did. However, at the same time, I was wondering if I could find a demo site made already. Luckily, I found live demo of a Shiny app on the persistent data webpage, which was open-sourced on github. This gave me the idea to use the github code for my DB instance mysql data. Meanwhile, I was learning how to use mysql. Although I wanted to go through the mysql reference manual as fast as I could, I realized that I had to go through each step carefully, so that I wasn’t missing out any important steps along the way. As soon as I created a database and a table, I was ready to head over to R. 

The next step was to understand how Shiny and R worked together. To find help with answering this question, I went to google for help again. I typed shiny tutorial into google search and again I found the information that I was looking for, Shiny - Tutorial. Since I knew I was looking for a quick read and a not a video, I scrolled through the page to look for a written tutorial. This lead me to lesson 1, https://shiny.rstudio.com/tutorial/lesson1/, in which I could quickly read about what I needed to know in order to create a shiny webpage. Once I understood the architecture of a shiny app, I went back to the github page of the demo site. 









Connecting to a DB Instance Running the MySQL Database Engine
http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html
