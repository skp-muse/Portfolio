---
layout: test
title: "Algorithms and Design"
permalink: /design/
---

# Overall Design




# customerWorkbookScript.py
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
