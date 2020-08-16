---
layout: default
title: "Database Integration"
permalink: /database/
---
## Designing a Database

I made the decision early to use a flavor of SQL as that language is classic among databases due to its power and flexibility. [mySQL](https://www.mysql.com/) was chosen because of its ongoing support for a wider variety of platforms that include a Python module for handling database connections. mySQL can be deployed on a variety of systems, so starting a local service on a workstation is no large feat and what I used to test this project. The program suite provides a variety of tools for managing and altering databases but is no replacement for knowledge of regular SQL syntax like that in the database creation script below. The database itself holds a very abstract version of our data that is stripped of any excess symbols. We rely on intelligent Python scripting to handle data validation and other logic such that the actual SQL database creation commands are quite simple and straightforward. We protect this database by carefully encapsulating the actual transactions. 

## customerTableCreation.sql
```sql
CREATE DATABASE IF NOT EXISTS customer_information_project;

USE customer_information_project;

/* Additional data validation is performed in accompanying python scripts */
CREATE TABLE customer (
    customer_id INT NOT NULL auto_increment,
    first_name VARCHAR(32),
    last_name VARCHAR(32),
    phone_number INT,
    phone_extension INT,
    email VARCHAR(64),
    city VARCHAR(32),
    state VARCHAR(2),
    zipcode INT,
    creation_date DATETIME NOT NULL,
    updated_date DATETIME NOT NULL,
    PRIMARY KEY(customer_id)
) AUTO_INCREMENT=1;

CREATE USER 'customerProjectUser'@'localhost' IDENTIFIED BY 'H-s)6Gd{D.';

GRANT INSERT, SELECT, UPDATE, DELETE ON customer_information_project.customer TO customerProjectUser@'localhost';

/* So that new permissions can immediately take affect */
FLUSH PRIVILEGES 
```

## Using mySQL

For security reasons a database should be treated as a separate service and program. In the sample project I practiced that by writing a separate mySQLTransactionModule.py that could properly encapsulate the database connection and transactions away from the main script. 

the internal ```python __connect()``` method

[bcrypt](https://www.npmjs.com/package/bcrypt)

```python
cursor = conn.cursor()
cursor.execute("SELECT phone_number, phone_extension FROM customer")
customerPhoneList = list(cursor.fetchall())
```

```python
cursor = conn.cursor()
cursor.executemany(sql, val)
conn.commit()
```

## mySQLTransactionModule.py
```python
# Functions to connect Python code with an existing mySQL database for the customer information project.
# Sarah Pressler - July 2020
# mySQL connector provided by https://dev.mysql.com/doc/connector-python/en/

import sys
import logging

# Helper function to pinpoint errors including error type and line number
def errorHandling():
    return 'Error: {}. {}, line: {}'.format(sys.exc_info()[0], sys.exc_info()[1], sys.exc_info()[2].tb_lineno)

# Private method to connection parameters for prebuilt customer information database
# WARNING: This function can offer improved security by prompting for password, suggest bcrypt module examples at http://zetcode.com/python/bcrypt/
def __connect(): 
    conn = None

    try: 
        conn = mysql.connector.connect(host='localhost', database='customer_information_project', user='customerProjectUser', password='H-s)6Gd{D.') 
    except:
        logging.error(errorHandling()) 
    finally:
        return conn

# Return Python searchable results of a SQL query on customer table for purpose of gathering existing phone listing
def retrieveCustomerPhoneList(conn):
    try:
        cursor = conn.cursor()
        cursor.execute("SELECT phone_number, phone_extension FROM customer")
        customerPhoneList = list(cursor.fetchall())
        return customerPhoneList
    except:
        logging.error(errorHandling()) 
    finally: 
        cursor.close() 
        conn.close()   

# Method to insert provided values into customer contact database as new records
def insertCustomerRecords(val):
    conn = __connect()
    sql = "INSERT INTO customer (first_name, last_name, phone_number, phone_extension, email, city, state, zipcode, created_date) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)"

    try:
        cursor = conn.cursor()
        cursor.executemany(sql, val)
        conn.commit()
    except:
        logging.error(errorHandling())
    finally: 
        cursor.close() 
        conn.close()  

# Method to update records in customer contact database that match provided values
def updateCustomerRecords(val):
    conn = __connect()
    sql = "UPDATE customer set first_name = %s, last_name = %s, phone_number = %s, phone_extension = %s, email = %s, city = %s, state = %s, zipcode = %s, updated_date = %s where phone_number = %s"

    try:
        # Tack phone number on to end of values for inclusion in SQL query
        for value in val:
            phoneNumber = value[2]
            value.append(phoneNumber)
        cursor = conn.cursor()
        cursor.executemany(sql, val)
        conn.commit()
    except:
        logging.error(errorHandling())
    finally: 
        cursor.close() 
        conn.close()   
```
