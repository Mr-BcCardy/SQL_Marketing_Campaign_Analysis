SQL project. Marketing Campaign Analysis
================

## **Data Exploration**

We’ve got to know the DataBase set up and now are ready to jump into
exploration process. We are dealing with email campaign and need to see
how effective it is.

### **E-mail metrics**

Let’s have a look at some e-mailing metrics and start with some basic
messages analysis. We will try to see the total number of unique users
who interacted with e-mails and total number of sales per each email.

``` r
library(odbc)
library(RODBC)
con1 <- odbcConnect("MySQL_DSN")
sqlQuery(con1, 'with total_interactions as (
 SELECT 
    dic_message.id AS message_id,
    dic_message.name AS message_name,
    COUNT(distinct db_emailactivity.contact_id) as total_users
    from product.dic_message LEFT JOIN product.db_emailactivity
    on dic_message.id = db_emailactivity.message_id
    group by 1,2)
SELECT 
    dic_message.id AS message_id,
    dic_message.name AS message_name,
    COUNT(db_sales.id) AS sales_count,
    COUNT(DISTINCT db_sales.contact_id) AS users_w_sales,
    total_interactions.total_users
FROM product.dic_message LEFT JOIN product.db_sales
ON dic_message.id = db_sales.message_id
JOIN total_interactions 
ON dic_message.id = total_interactions.message_id and dic_message.name = total_interactions.message_name
group by 1,2
order by 3 desc limit 10')
```

    ##    message_id message_name sales_count users_w_sales total_users
    ## 1     2133758     Promo 14        2664          2185       11457
    ## 2     2213803     Promo 12        1384          1101        3679
    ## 3     2122670      Promo 1         968           718        2201
    ## 4     2271015      Promo 6         626           493        8732
    ## 5     2126640     Promo 28         605           477        1803
    ## 6     2126069     Promo 29         346           281         788
    ## 7     2192231     Promo 13         346           290        5523
    ## 8     2134888     Promo 15         261           201        6513
    ## 9     2295343      Promo 8         193           137        4716
    ## 10    2315006     Promo 11         170           111        4387

Leaving aside user conversion rate we can see the top 10 messages by
number (**impotant!** not revenue) of sales. We can play with ordering
if we would like to see the bottom of this list of course.

### **E-mail revenue share**

Speaking about revenue. Let’s try to find each separate e-mail revenue
share from the total revenue.

``` r
sqlQuery(con1,'SELECT message_id, 
  name, 
  sum(revenue) as revenue, 
  round(sum(revenue) * 100 / t.total, 2) as revenue_from_total_perc
 FROM product.dic_message
 LEFT JOIN product.db_sales
 ON dic_message.id = db_sales.message_id
 cross join (select sum(revenue) as total from db_sales) t
 GROUP BY message_id order by revenue desc limit 10')
```

    ##    message_id     name revenue revenue_from_total_perc
    ## 1     2133758 Promo 14 7847.95                   27.60
    ## 2     2213803 Promo 12 4111.20                   14.46
    ## 3     2122670  Promo 1 2895.86                   10.18
    ## 4     2271015  Promo 6 1839.37                    6.47
    ## 5     2126640 Promo 28 1820.54                    6.40
    ## 6     2126069 Promo 29 1004.12                    3.53
    ## 7     2192231 Promo 13  996.04                    3.50
    ## 8     2134888 Promo 15  759.22                    2.67
    ## 9     2295343  Promo 8  562.44                    1.98
    ## 10    2315006 Promo 11  540.45                    1.90

We have quite similar picture just from the different angle, from the
angle of revenue.

### **User metrics**

Next we would like to see some stats on users. The following information
will give us pretty good glimps on user activities:

User-id, email, source, number of clicked unique messages, count of read
unique messages, count of delivered unique messages– revenue

``` r
sqlQuery(con1, 'SELECT 
    db_users.id AS user_id,
    db_users.email AS email,
    dic_usersource.name AS source,
    counts.unique_delivered AS deliv,
    counts.unique_red AS rread,
    counts.unique_clicked AS clicked,
    revenue.total_revenue AS revenue
FROM
    product.db_users
        LEFT JOIN
    product.dic_usersource ON db_users.source_id = dic_usersource.id
        LEFT JOIN
    (SELECT 
        db_contacts.user_id,
            COUNT(DISTINCT CASE WHEN status_id = 1 THEN message_id END) AS unique_clicked,
            COUNT(DISTINCT CASE WHEN status_id = 3 THEN message_id END) AS unique_red,
            COUNT(DISTINCT CASE WHEN status_id = 2 THEN message_id END) AS unique_delivered
    FROM
        product.db_emailactivity
    LEFT JOIN product.db_contacts ON db_emailactivity.contact_id = db_contacts.id
    GROUP BY db_contacts.user_id) counts ON db_users.id = counts.user_id
        LEFT JOIN
    (SELECT 
        user_id, SUM(revenue) AS total_revenue
    FROM
        product.db_sales
    LEFT JOIN db_contacts ON db_sales.contact_id = db_contacts.id
    GROUP BY contact_id) revenue ON db_users.id = revenue.user_id limit 10')
```

    ##    user_id                  email           source deliv rread clicked revenue
    ## 1  3098663 email3098663@email.com   Google account     6     3       1    4.71
    ## 2  3098664 email3098664@email.com   Google account     1     1       1    3.53
    ## 3  3098665 email3098665@email.com Facebook account     2     0       0      NA
    ## 4  3098666 email3098666@email.com   Google account     1     0       0      NA
    ## 5  3098667 email3098667@email.com   Google account     1     0       0      NA
    ## 6  3098668 email3098668@email.com            Email     2     2       0      NA
    ## 7  3098669 email3098669@email.com   Google account     3     1       0      NA
    ## 8  3098670 email3098670@email.com   Google account    NA    NA      NA      NA
    ## 9  3098671 email3098671@email.com   Google account     1     0       0      NA
    ## 10 3098672 email3098672@email.com Facebook account     1     0       0      NA

I deliberately used *left joins* to include all users, thus it gives an
overview if there are users who have never received any emails. We can
also receive a complete list of such users with the following query:

``` r
sqlQuery(con1, 'SELECT 
    id, email
FROM
    product.db_users
WHERE
    id NOT IN (SELECT DISTINCT
            db_contacts.user_id
        FROM
            product.db_emailactivity
                JOIN
            product.db_contacts ON db_emailactivity.contact_id = db_contacts.id) limit 10')
```

    ##         id                  email
    ## 1  3098670 email3098670@email.com
    ## 2  3098680 email3098680@email.com
    ## 3  3098682 email3098682@email.com
    ## 4  3098683 email3098683@email.com
    ## 5  3098720 email3098720@email.com
    ## 6  3098726 email3098726@email.com
    ## 7  3098727 email3098727@email.com
    ## 8  3098737 email3098737@email.com
    ## 9  3098750 email3098750@email.com
    ## 10 3098753 email3098753@email.com

### **User visits**

We may also want to look if there is significant difference between user
profile creation and last visit date.

``` r
sqlQuery(con1, 'SELECT CASE WHEN DATEDIFF(lastvisit, created_at) IS NULL THEN "without_login"
WHEN DATEDIFF(lastvisit, created_at) < 1 THEN "DATEDIFF = 0"
WHEN DATEDIFF(lastvisit, created_at) = 1 THEN "1 day"
WHEN DATEDIFF(lastvisit, created_at) BETWEEN 2 AND 5 THEN "2-5 days"
WHEN DATEDIFF(lastvisit, created_at) > 5 THEN "more 5 days"
END AS diff_type,
    COUNT(id) AS count_user_id
FROM
    product.db_users
GROUP BY 1 limit 10')
```

    ##      diff_type count_user_id
    ## 1 DATEDIFF = 0         24323
    ## 2  more 5 days          1274
    ## 3        1 day           807
    ## 4     2-5 days           915
