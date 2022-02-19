## SQL for Data Analysis 

### Setting up the environment:
[Check here to create database and table in BigQuery](https://cloud.google.com/bigquery/docs/tables)

**Database used:**
Parch and Posey, a hypothetical paper company's sales data of different types of paper (regular, gloss, standard). The database consists of different tables linked with a database schema. 

### Entity Relationship Diagrams
An entity relationship diagram (ERD) is a common way to view data in a database. Below is the ERD for the database we will use from Parch & Posey. These diagrams help you visualize the data you are analyzing including:

- The names of the tables.
- The columns in each table.
- The way the tables work together.
- 
![image](https://user-images.githubusercontent.com/92245436/154788897-161d5d01-4dfa-43a6-ac29-9e8b5091f4c9.png)

The "crow's foot" that connects the tables together shows us how the columns in one table relate to the columns in another table.

### What to Notice
In the Parch & Posey database there are five tables (essentially 5 spreadsheets):

- web_events
- accounts
- orders
- sales_reps
- region


## 1. Basic SQL

There are some major advantages to using traditional relational databases, which we interact with using SQL. The five most apparent are:

- SQL is easy to understand.
- Traditional databases allow us to access data directly.
- Traditional databases allow us to audit and replicate our data.
- SQL is a great tool for analyzing multiple tables at once.
- SQL allows you to analyze more complex questions than dashboard tools like Google Analytics.

### SQL vs. NoSQL
You may have heard of NoSQL, which stands for not only SQL. Databases using NoSQL allow for you to write code that interacts with the data a bit differently. These NoSQL environments tend to be particularly popular for web based data, but less popular for data that lives in spreadsheets the way we have been analyzing data up to this point. One of the most popular NoSQL languages is called MongoDB.

### Why Businesses Like Databases
- **Data integrity is ensured** - only the data you want entered is entered, and only certain users are able to enter data into the database.
- **Data can be accessed quickly** - SQL allows you to obtain results very quickly from the data stored in a database. Code can be optimized to quickly pull results.
- **Data is easily shared** - multiple individuals can access data stored in a database, and the data is the same for all users allowing for consistent results for anyone with access to your database

#### The key to SQL is understanding statements. A few statements include:

- **CREATE TABLE** is a statement that creates a new table in a database.
- **DROP TABLE** is a statement that removes a table in a database.
- **SELECT** allows you to read data and display it.<br>
This is called a query.



