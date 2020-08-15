---
layout: default
title: "Algorithms and Design"
permalink: /design/
---

## Simulated Data Collection Project

The primary goal of this sample project was to gain practice in scripting a scenario that marries real world application with software design principles. This project is primarily concerned with building a contact list of names with phone or email and other associated data. The first step was to use Python, a popular and flexible language for automation tasks, to read an Excel workbook and validate its data. By using [openpyxl](https://openpyxl.readthedocs.io/en/stable/) I gained the capability to casually read Microsoft's .xlsx filetype. The data is then validated within the Python script where we set up our expectations for the spreadsheet data and validate it, primarily through the use of [regular expressions](https://www.regular-expressions.info/). Data is not very useful without somewhere to store it, and so a database connection was formed with [mySQL](https://www.mysql.com/), a popular SQL platform that provides tools for integration with today's multitude of platforms and services. Included in their toolset is a feature rich Python module for assisting with database transactions.

## Additional Design Decisions

The code in customerWorkbookScript.py below consists of several Python try/except blocks such as the one in this function:

```python
# Helper function to determine handling of given cell.value
def validateCellValue(cell, regex):
	try:
		fullMatch = regex.fullmatch((str(cell.value)))
		if fullMatch is None:
			return None
		return fullMatch.string
	except TypeError:
			return None
	except AttributeError: 
			return None
	except:
		logging.error(errorHandling())
```

This is a common best practice for Python in order to control program flow in edge cases or unexpected behaviors. In the case of a 'TypeError' or 'AttributeError' the function knows to return an empty value for the Excel cell provided. This is because the data provided does not match our validation criteria and we will simply dump that data so it is not stored in our eventual database. Later in this process we make sure the records from Excel have either a valid phone number or email along with a customer name as that was the stated primary goal of the sample project. Any other issue is caught by the general 'except' where the script is halted and a detail error message written to our log for future improvement of the script.

Regular expressions can provide multiple paths to data validation, such as with phone numbers in the form of '###-###-####' or '(###)###-####'. These various extra characters can then be stripped from the information for simple storage of digits into a SQL database. The 'phoneRegex' in the sample code below is capable of reading phone numbers in either of the above formats and with an optional extenseion such as 'ext###' or 'x#####'.

```python
nameRegex = re.compile("^[a-zA-Z '.-]*$")
phoneRegex = re.compile("^((\d{3}-)|(\(\d{3}\)))\d{3}-\d{4}((x|ext)\d{1,6})?$")
emailRegex = re.compile("(^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$)")
zipcodeRegex = re.compile("^\d{5}(?:[-\s]\d{4})?$")
```

## Code Quality


## customerWorkbookScript.py
```python
# Script to read an Excel workbook for customer information collection project. Row information is first validated through regular expressions and then built in Python for use in mySQL transactions.
# Sarah Pressler - July 2020
# Excel read/write features provided by "openpyxl" - https://openpyxl.readthedocs.io/en/stable/
# Workbook columns - First name \ Last name \ Phone \ Email \ City \ State \ Zipcode 
import sys
import logging
import re
from mySQLTransactionModule import retrieveCustomerPhoneList, insertCustomerRecords, updateCustomerRecords
from openpyxl import load_workbook

# Helper function to pinpoint errors including error type and line number
def errorHandling():
    return 'Error: {}. {}, line: {}'.format(sys.exc_info()[0], sys.exc_info()[1], sys.exc_info()[2].tb_lineno)

# Helper function to determine handling of given cell.value
def validateCellValue(cell, regex):
	try:
		fullMatch = regex.fullmatch((str(cell.value)))
		if fullMatch is None:
			return None
		return fullMatch.string
	except TypeError:
			return None
	except AttributeError: 
			return None
	except:
		logging.error(errorHandling())
										 
# Script initializations, static variables and reusable regex expressions
staticStateList = ["AL", "AK", "AZ", "AR", "CA", "CO", "CT", "DE", "DC", "FL", "GA", "HI", "ID", "IL", "IN", "IA", "KS", "KY", "LA", "ME", "MD", "MA", "MI", "MN", "MS", "MO", "MT", "NE", "NV", "NH", "NJ", "NM", "NY", "NC", "ND", "OH", "OK", "OR", "PA", "RI", "SC", "SD", "TN", "TX", "UT", "VT", "VA", "WA", "WV", "WI", "WY"]
try:
	nameRegex = re.compile("^[a-zA-Z '.-]*$")
	phoneRegex = re.compile("^((\d{3}-)|(\(\d{3}\)))\d{3}-\d{4}((x|ext)\d{1,6})?$")
	emailRegex = re.compile("(^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$)")
	zipcodeRegex = re.compile("^\d{5}(?:[-\s]\d{4})?$")
except:
	sys.exit("Error in compiling regex expressions")
	
# Try to open expected CustomerInformation.xlsx workbook
# Option: Different path than same directory should be placed in filename value
try:
	currentWorkbook = load_workbook(filename = 'CustomerInformation.xlsx')
	currentSheet = currentWorkbook['CustomerInformation']
except FileNotFoundError:
	sys.exit("Workbook not found")
except KeyError:
	sys.exit("CustomerInformation sheet not found")
except:
	logging.error(errorHandling())

# Read CustomerInformation.xlsx rows starting at row 3 & columns 1-7
# Majority of cells run through validateCellValue() which will give entry a NULL result if invalid
# Check for minimum of valid first name, last name, and phone number
try:
	workbookRows = []
	firstNameCheck = False
	lastNameCheck = False
	phoneNumberCheck = False

	for row in currentSheet.iter_rows(min_row=3):
		currentRow = [None] * 7
		for cell in row:
			if cell.column == 1: # TODO: could initialize a dictionary or list for Excel column flexibility
				# First name 
				currentRow[0] = validateCellValue(cell, nameRegex)
				if currentRow[0] != None:
					firstNameCheck = True
			elif cell.column == 2:
				# Last name 
				currentRow[1] = validateCellValue(cell, nameRegex)
				if currentRow[1] != None:
					lastNameCheck = True
			elif cell.column == 3:
				# Phone 
				currentRow[2] = validateCellValue(cell, phoneRegex)
				if currentRow[2] != None:
					phoneNumberCheck = True
			elif cell.column == 4:
				# Email 
				currentRow[3] = validateCellValue(cell, emailRegex)
			elif cell.column == 5:
				# City 
				currentRow[4] = validateCellValue(cell, nameRegex)
			elif cell.column == 6:
				# State	
				try:
					enteredState = str(cell.value)
					if enteredState in staticStateList:
						currentRow[5] = enteredState
				except:
					currentRow[5] = None
			elif cell.column == 7:
				# Zipcode 
				currentRow[6] = validateCellValue(cell, zipcodeRegex)
		if firstNameCheck and lastNameCheck and phoneNumberCheck:
			workbookRows.append(currentRow)
except:
	logging.error(errorHandling())

# Determine if each valid workbook row has any duplicate contact info from current customers
# WARNING: Script does not currently check duplicates within provided workbook data
try:
	phoneList = retrieveCustomerPhoneList()
	phoneList.sort()
	insertValues = []
	updateValues = []

	# If phone number is existing list, update rather than insert
	for currentRow in workbookRows:
		if currentRow[2] in phoneList:
			matchIndex = phoneList.index(currentRow[2])
			updateValues.append(currentRow)
		else:
			insertValues.append(currentRow)

	insertCustomerRecords(insertValues)
	updateCustomerRecords(updateValues)
except:
	logging.error(errorHandling()) 
```
