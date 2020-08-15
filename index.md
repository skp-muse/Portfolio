## Introduction

This portfolio is a space to showcase my growth in software development principles. In my work with IT organizations both small and large I have gained appreciation for the need to research new skills and methods for solving problems. Technology is a tool to be used for finding logical solutions that both adhere to development best practices as well as real world considerations. 

This portfolio starts with a retrospective code review of some of my earlier work in Python and SQL. Since then I have learned much about designing software, particulary how it can help with real world applications of data analyis and validation. Similarly, intelligent software should include efficent algorithms that limit how much we need to examine data both for the protection of that data and the efficency of operations. And finally, working with data implies a secure connection to a database that cannot be misused by malciious actors.

## Code Review

On this page I have placed a code review of some earlier work in Python and SQL that comes from my now completed bachelor’s program of computer science. This review covers important considerations such as security and overall soundness of design, some of which is clearly missed in this initial example.  

[![Image](/assets/codeReview.png)](https://drive.google.com/file/d/1TOHhEwz8-G-FAkldC9sZEASqkqkWopwd/view?usp=sharing)

## Simulated Project

I’ve taken these important considerations and produced further work that simulates a simple information collection project for customer contacts. This project is primarily concerned with building a contact list of names with phone or email and other associated data. Data is read by a Python module specifically written to support Microsoft Excel’s most recent .xlsx format. It is then validated though regular expressions that determine if information entered fits certain parameters. These expressions can provide multiple paths to this validation, such as with phone numbers in the form of ###-###-#### or (###)###-####. These various extra characters can then be stripped from the information for simple storage of digits into a SQL database.  
