# SQL Injection
SQL Injection is an attack technique used to exploit applications that construct SQL statements from user-supplied input. When successful, the attacker is able to change the logic of SQL statements executed against the database.

 

Structured Query Language (SQL) is a specialized programming language for sending queries to databases. The SQL programming language is both an ANSI and an ISO standard, though many database products supporting SQL do so with proprietary extensions to the standard language. Applications often use user-supplied data to create SQL statements. If an application fails to properly construct SQL statements it is possible for an attacker to alter the statement structure and execute unplanned and potentially hostile commands. When such commands are executed, they do so under the context of the user specified by the application executing the statement. This capability allows attackers to gain control of all database resources accessible by that user, up to and including the ability to execute commands on the hosting system.

 

SQL Injection using Dynamic Strings
A web based authentication form might build a SQL command string using the following method:

 

 

SQLCommand = "SELECT Username FROM Users WHERE Username = '" 
SQLCommand = SQLComand & strUsername
SQLCommand = SQLComand & "' AND Password = '" 
SQLCommand = SQLComand & strPassword
SQLCommand = SQLComand & "'"
strAuthCheck = GetQueryResult(SQLQuery)
 

 

Example 1 - Dynamically built SQL command string

 

In this code, the developer combines the input from the user, strUserName and strPassword, with the logic of the SQL query. Suppose an attacker submits a login and password that looks like the following:

 

Username: foo
Password: bar' OR ''='
 

The SQL command string built from this input would be as follows:

 

SELECT Username FROM Users WHERE Username = 'foo' 
AND Password = 'bar' OR ''=''
 

This query will return all rows from the user's database, regardless of whether "foo" is a real user name or "bar" is a legitimate password. This is due to the OR statement appended to the WHERE clause. The comparison ''='' will always return a "true" result, making the overall WHERE clause evaluate to true for all rows in the table. If this is used for authentication purposes, the attacker will often be logged in as the first or last user in the Users table.

 

SQL Injection in Stored Procedures
It is common for SQL Injection attacks to be mitigated by relying on parameterized arguments passed to stored procedures. The following examples illustrate the need to audit the means by which stored procedures are called and the stored procedures themselves.

 

SQLCommand = "exec LogonUser '" + strUserName + "','" + strPassword + "'"
 

Example 2 - SQL Injection in stored procedure execute statement

 

Using a stored procedure does not imply that the statement used to call the stored procedure is safe. An attacker could supply input like the following to execute additional statements:

 

Username: foo
Password: '; DROP TABLE Users--
 

The generated SQLCommand string would be:

 

exec LogonUser 'foo',''; DROP TABLE Users--'

 

On a Microsoft SQL server, using the above SQL command string will execute two statements: the first will likely not identify a user to log in, and the second would remove the Users table from the database.

 

The following example would be problematic even if the stored procedure were executed using a prepared or parameterized statement:

 

CREATE PROCEDURE LoginUser 
@Username varchar(50) = '', 
@Password varchar(50) = ''
AS
BEGIN
DECLARE @command varchar(100)
set @command = 'select * from Users where Username = ''' +
@Username + 
''' and Password = ''' +
@Password +
''''
EXEC (@command)
END
GO
 

Example 3 - SQL Injection within a stored procedure

 

Stored procedures themselves can build dynamic statements, and these are susceptible to SQL Injection attacks. The attack against this stored procedure would be carried out in an identical fashion to Example 1.

 

It should be noted that attempts to escape dangerous characters are not sufficient to address these flaws, even within stored procedures as in Example 3. The referenced article "New SQL Truncation Attacks And How To Avoid Them" ([8]) demonstrates how assigning strings to fixed-size variables, like the varchars in Example 3, can cause those strings to be truncated and lead to SQL Injection attacks.

 

SQL Injection Identification and Exploitation
There are two commonly known methods of identifying a SQL injection attack: SQL Injection and Blind SQL Injection.

 

SQL Injection

The first method commonly used to identify and exploit SQL Injection used information provided by errors generated during testing. These errors often would include the text of the offending SQL statement and details on the nature of the error. Such information is very helpful when creating reliable exploits for SQL Injection attacks.

 

By appending a union select statement to the parameter, the attacker can test for access to other tables in the target database:

 

http://example/article.asp?ID=2+union+all+select+name+from+sysobjects
 

The database server might return an error similar to this:

 

Microsoft OLE DB Provider for ODBC Drivers error 
'80040e14' 
[Microsoft][ODBC SQL Server Driver][SQL Server]All 
queries in an SQL statement containing a UNION 
operator must have an equal number of expressions
in their target lists. 

 

This error informs the attacker that the query structure was slightly incorrect, but that it will likely be successful once the test query's column count matches the original query statement.

 

Blind SQL Injection

Blind SQL Injection techniques must be used when detailed error messages are not provided to the attacker. It is often the case that web applications will display a user-friendly error page with minimal technical data, effectively "blinding" those exploitation techniques described above.

 

In order to exploit SQL Injection in such scenarios, the attacker gathers information by other means, such differential timing analysis or the manipulation of user-visible state. One common example of the latter is to analyze the behavior of a system when passed values that would evaluate to a false and true result when used in a SQL statement.

 

If a SQL Injection weakness is present, then executing the following request on a web site:

 

http://example/article.asp?ID=2+and+1=1 
 

should return the same web page as:

 

http://example/article.asp?ID=2 
 

because the SQL statement and 1=1 is always true.

Executing the following request to a web site:

 

http://example/article.asp?ID=2+and+1=0 
 

would then cause the web site to return a friendly error or no page at all. This is because the SQL statement and 1=0 is always false.

Once the attacker discovers that a site is susceptible to Blind SQL Injection, exploitation can proceed using established techniques.

 

References
"Advanced SQL Injection in SQL Server Applications", Chris Anley - NGSSoftware

[1] http://www.nextgenss.com/papers/advanced_sql_injection.pdf

 

"More advanced SQL Injection", Chris Anley - NGSSoftware

[2] http://www.nextgenss.com/papers/more_advanced_sql_injection.pdf

 

"Web Application Disassembly with ODBC Error Messages", David Litchfield - @stake

[3] http://www.nextgenss.com/papers/webappdis.doc

 

"SQL Injection Walkthrough"

[4] http://www.securiteam.com/securityreviews/5DP0N1P76E.html

 
