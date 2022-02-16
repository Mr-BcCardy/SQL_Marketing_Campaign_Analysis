SQL project. Marketing Campaign Analysis
================
Arkadiy Rudenko
2022-02-16

#### **Summary**

For this SQL project I used a completely depersonalized partial
replication of a database. The document is written in RMarkdown with
MySQL syntax. SQL script to replicate the DB will be provided in a
separate .sql file.

#### **DataBase Description**

Before jumping to the data exploration part let’s have a look at what we
are dealing with:

The database consists of 9 tables. Tables with data have “db\_” prefixes
and reference tables have “dic\_” prefixes. Lets have a look at them one
by one. I will also load a couple of libraries to establish connection
between RStudio and MySQL (for the project I’m using MySQL Workbench
8.0.28)

1.  **db_actions** table

``` r
library(odbc)
library(RODBC)
con1 <- odbcConnect("MySQL_DSN")
sqlQuery(con1, 'describe product.db_actions')
```

    ##        Field     Type Null Key Default Extra
    ## 1         id      int   NO  NA       0    NA
    ## 2  action_id      int   NO  NA       0    NA
    ## 3  client_id char(35)   NO  NA      NA    NA
    ## 4    user_id      int   NO  NA       0    NA
    ## 5 created_at datetime  YES  NA      NA    NA

2.  **db_contacts** table

``` r
sqlQuery(con1, 'describe product.db_contacts')
```

    ##        Field     Type Null Key Default Extra
    ## 1         id      int   NO  NA       0    NA
    ## 2    user_id      int   NO  NA       0    NA
    ## 3 created_at datetime  YES  NA      NA    NA

3.  **db_emailactivity** table

``` r
sqlQuery(con1, 'describe product.db_emailactivity')
```

    ##        Field     Type Null Key Default Extra
    ## 1         id      int   NO  NA       0    NA
    ## 2 contact_id      int   NO  NA       0    NA
    ## 3  status_id      int   NO  NA       0    NA
    ## 4 message_id      int   NO  NA       0    NA
    ## 5 created_at datetime  YES  NA      NA    NA

4.  **db_sales** table

``` r
sqlQuery(con1, 'describe product.db_sales')
```

    ##        Field         Type Null Key Default Extra
    ## 1         id          int   NO  NA       0    NA
    ## 2 contact_id          int   NO  NA       0    NA
    ## 3    revenue decimal(3,2)   NO  NA       0    NA
    ## 4 message_id          int   NO  NA       0    NA
    ## 5 created_at     datetime  YES  NA      NA    NA

5.  **db_users** table

``` r
sqlQuery(con1, 'describe product.db_users')
```

    ##        Field     Type Null Key Default Extra
    ## 1         id      int   NO  NA       0    NA
    ## 2      email char(35)   NO  NA      NA    NA
    ## 3 created_at datetime  YES  NA      NA    NA
    ## 4  lastvisit datetime  YES  NA      NA    NA
    ## 5  numvisits      int   NO  NA       0    NA
    ## 6 is_deleted smallint   NO  NA       0    NA
    ## 7  source_id      int   NO  NA       0    NA
    ## 8   verified smallint   NO  NA       0    NA

6.  **dic_actions** table

``` r
sqlQuery(con1, 'describe product.dic_actions')
```

    ##        Field        Type Null Key Default Extra
    ## 1         id         int   NO  NA       0    NA
    ## 2       name varchar(50)   NO  NA      NA    NA
    ## 3 created_at    datetime  YES  NA      NA    NA

7.  **dic_emailstatus** table

``` r
sqlQuery(con1, 'describe product.dic_emailstatus')
```

    ##   Field     Type Null Key Default Extra
    ## 1    id      int   NO  NA       0    NA
    ## 2  name char(35)   NO  NA      NA    NA

8.  **dic_message** table

``` r
sqlQuery(con1, 'describe product.dic_message')
```

    ##   Field     Type Null Key Default Extra
    ## 1    id      int   NO  NA       0    NA
    ## 2  name char(35)   NO  NA      NA    NA

9.  **dic_usersource** table

``` r
sqlQuery(con1, 'describe product.dic_usersource')
```

    ##   Field     Type Null Key Default Extra
    ## 1    id      int   NO  NA       0    NA
    ## 2  name char(35)   NO  NA      NA    NA
