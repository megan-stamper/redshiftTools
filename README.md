<!-- README.md is generated from README.Rmd. Please edit that file -->
redshiftTools
=============

This is an R Package meant to ease common operations with Amazon Redshift. The package makes bulk uploads easier by uploading data to an S3 bucket and then calling a copy command on the server. All those tasks are enclosed in encapsulated functions.

Installation
------------

To install this package:

``` r
    devtools::install_github("megan-stamper/redshiftTools")
```

Usage
-----

### Creating tables

To create tables, use `rs_create_statement`, which receives a data.frame and returns the query for creating the same table in Amazon Redshift.

``` r
n=1000
testdf = data.frame(
a=rep('a', n),
b=c(1:n),
c=rep(as.Date('2017-01-01'), n),
d=rep(as.POSIXct('2017-01-01 20:01:32'), n),
e=rep(as.POSIXlt('2017-01-01 20:01:32'), n),
f=rep(paste0(rep('a', 4000), collapse=''), n) )

cat(rs_create_statement(testdf, table_name='dm_great_table'))
```

This returns:

``` sql
CREATE TABLE dm_great_table (
a VARCHAR(8),
b int,
c date,
d timestamp,
e timestamp,
f VARCHAR(4096)
);
```

### Uploading data

To upload data, there are two functions: `rs_replace_table` and `rs_upsert_table`. Both have almost the same parameters, except upsert allows you to specify keys to search for matching rows.

For example, suppose we have a table to load with two columns of integers, we could use the following code:

``` r
    library("aws.s3")
    library(RPostgres)
    library(redshiftTools)

    a=data.frame(a=seq(1,10000), b=seq(10000,1))
    n=head(a,n=10)
    n$b=n$a
    nx=rbind(n, data.frame(a=seq(5:10), b=seq(10:5)))

    con <- dbConnect(RPostgres::Postgres(), dbname="dbname",
    host='my-redshift-url.amazon.com', port='5439',
    user='myuser', password='mypassword',sslmode='require')

    b=rs_replace_table(a, dbcon=con, table_name='mytable', bucket="mybucket", split_files=4)
    c=rs_upsert_table(nx, dbcon=con, table_name = 'mytable', split_files=4, bucket="mybucket", keys=c('a'))
```

### Creating tables with data

A conjunction of `rs_create_statement` and `rs_replace_table` can be found in `rs_create_table`. You can create a table from scratch from R and upload the contents of the data frame, without needing to write SQL code at all.

``` r
    library("aws.s3")
    library(RPostgres)
    library(redshiftTools)

    a=data.frame(a=seq(1,10000), b=seq(10000,1))
    
    con <- dbConnect(RPostgres::Postgres(), dbname="dbname",
    host='my-redshift-url.amazon.com', port='5439',
    user='myuser', password='mypassword',sslmode='require')

    b=rs_create_table(a, dbcon=con, table_name='mytable', bucket="mybucket", split_files=4)
```
