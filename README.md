# Resources
* https://pgexercises.com/questions/basic/
* https://pgexercises.com/questions/joins/
* https://www.w3schools.com/sql/sql_join.asp
* https://www.javatpoint.com/sql-select-from-multiple-tables


* How can you retrieve all the information from the cd.facilities table?
select * from cd.facilities

the WHERE clause allows us to filter for the rows we're interested in - in this case, 

* How can you produce a list of facilities that charge a fee to members, and that fee is less than 1/50th of the monthly maintenance cost? Return the facid, facility name, member cost, and monthly maintenance of the facilities in question.

When we want to test for two or more conditions, we use AND to combine them. We can, as you might expect, use OR to test whether either of a pair of conditions is true.

select facid, name, membercost, monthlymaintenance 
from cd.facilities 
where  
membercost > 0 and (membercost < monthlymaintenance/50)

* How can you produce a list of all facilities with the word 'Tennis' in their name?

SQL's LIKE operator provides simple pattern matching on strings. It's pretty much universally implemented, and is nice and simple to use - it just takes a string with the % character matching any string, and _ matching any single character. In this case, we're looking for names containing the word 'Tennis', so putting a % on either side fits the bill.

select * from cd.facilities where name like '%Tennis%'

* Matching against multiple possible values

https://stackoverflow.com/questions/5803472/sql-where-id-in-id1-id2-idn

select * from cd.facilities where facid in (1,5)

select * 
	from cd.facilities
	where
		facid in (
			select facid from cd.facilities
			);

* How can you produce a list of facilities, with each labelled as 'cheap' or 'expensive' depending on if their monthly maintenance cost is more than $100? Return the name and monthly maintenance of the facilities in question.

select name, 
	case when (monthlymaintenance > 100) then
		'expensive'
	else
		'cheap'
	end as cost
	from cd.facilities;    

* How can you produce an ordered list of the first 10 surnames in the members table? The list must not contain duplicates.

select distinct surname from cd.members 
order by
surname ASC
Limit 10

There's three new concepts here, but they're all pretty simple.

Specifying DISTINCT after SELECT removes duplicate rows from the result set. Note that this applies to rows: if row A has multiple columns, row B is only equal to it if the values in all columns are the same. As a general rule, don't use DISTINCT in a willy-nilly fashion - it's not free to remove duplicates from large query result sets, so do it as-needed.

Specifying ORDER BY (after the FROM and WHERE clauses, near the end of the query) allows results to be ordered by a column or set of columns (comma separated).

The LIMIT keyword allows you to limit the number of results retrieved. This is useful for getting results a page at a time, and can be combined with the OFFSET keyword to get following pages. This is the same approach used by MySQL and is very convenient - you may, unfortunately, find that this process is a little more complicated in other DBs.

* You, for some reason, want a combined list of all surnames and all facility names. Yes, this is a contrived example :-). Produce that list!

select surname from cd.members 
union
select name from cd.facilities

The UNION operator does what you might expect: combines the results of two SQL queries into a single table. The caveat is that both results from the two queries must have the same number of columns and compatible data types.

UNION removes duplicate rows, while UNION ALL does not. Use UNION ALL by default, unless you care about duplicate results.

* You'd like to get the signup date of your last member. How can you retrieve this information?

select joindate from cd.members
order by joindate desc
limit 1

OR

select max(joindate) as latest
	from cd.members;  

This is our first foray into SQL's aggregate functions. They're used to extract information about whole groups of rows, and allow us to easily ask questions like:

What's the most expensive facility to maintain on a monthly basis?
Who has recommended the most new members?
How much time has each member spent at our facilities?
The MAX aggregate function here is very simple: it receives all the possible values for joindate, and outputs the one that's biggest. There's a lot more power to aggregate functions, which you will come across in future exercises.

* You'd like to get the first and last name of the last member(s) who signed up - not just the date. How can you do that?

select firstname, surname, joindate
	from cd.members
	where joindate = 
		(select max(joindate) 
			from cd.members);  

In the suggested approach above, you use a subquery to find out what the most recent joindate is. This subquery returns a scalar table - that is, a table with a single column and a single row. Since we have just a single value, we can substitute the subquery anywhere we might put a single constant value. In this case, we use it to complete the WHERE clause of a query to find a given member.

