---
layout: test
title: "test page2"
permalink: /test2/
---

# Test Header

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
