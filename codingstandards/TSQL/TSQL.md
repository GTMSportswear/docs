## T-SQL Coding Standards
### Naming Conventions
* Table names should be singular. (Ex: Customer rather than Customers)
* Avoid underscores in names, except when concatenating several identifiers for the name of a database constraint. Rather than underscores in names, use proper/mixed casing.
* Error on the side of verbose naming, using few abbreviations. (Ex: ItemNumber rather than ItemNo)
* Used mixed casing (proper casing with first letter capitalized), even on abbreviations. For example, use __Id__ or __Api__ rather than ID or API. This makes for more readable identifiers when multiple abbreviations are used, such as GtmApi.
* When creating or referencing database objects, use __fully-qualified names__. Even if a table you are referencing exists in __dbo__, use __dbo__ as the schema in your reference. For example, use dbo.TableName rather than just TableName.
```SQL
CREATE TABLE Sales.Customer
(
   -- Notice the use of Id rather than ID.
   CustomerId INT NOT NULL,
   FirstName NVARCHAR(32) NOT NULL,
   LastName NVARCHAR(32) NOT NULL,
   CreatedOn DATETIMEOFFSET NOT NULL
      CONSTRAINT [DF_Sales_CustomerOrder_CreatedOn] DEFAULT(SYSDATETIMEOFFSET()),
 
   -- A constraint name is the only time underscores are appropriate.  Notice it is
   -- a combination of a prefix, schema name, table name, and column name(s).
   CONSTRAINT [PK_Sales_CustomerOrder_CustomerId] PRIMARY KEY CLUSTERED
   (
      CustomerOrderId ASC
   )
);
```
Table variables should also follow a similar style and naming convention as user tables.
```SQL
DECLARE @Customer TABLE
(
   CustomerId INT NOT NULL PRIMARY KEY,
   M3CustomerNumber NCHAR(10) NOT NULL UNIQUE,
   FirstName NVARCHAR(32) NOT NULL,
   LastName NVARCHAR(32) NOT NULL
);
```
Notice the PRIMARY KEY or UNIQUE constraints can be placed inline with the column definitions as we don't require a specific name for them in table variables. If you have a composite PRIMARY KEY or UNIQUE constraint, then you do need to place them after the column definitions, but again, there is no need for a specific constraint name:
```SQL
DECLARE @OrderLine TABLE
(
   OrderId INT NOT NULL,
   LineNumber SMALLINT NOT NULL,
   ProductKey INT NOT NULL,
   Quantity INT NOT NULL,
   NetSales MONEY NOT NULL,
 
   PRIMARY KEY(OrderId, LineNumber)
);
```
### Query Structure
#### Tabbing
Use __3-space tabbing__ and _make sure it converts them to spaces_ as well. This helps any one else who opens your code so it looks exactly how you saw it. If you do not convert the tab characters to spaces, then someone else's tabs may be setup for more spaces, making your code look drawn out.
```SQL
SELECT O.OrderNo, O.OrderDate, C.CustomerId, C.FirstName, C.LastName
FROM Sales.Customer C
   INNER JOIN Sales.CustomerOrder O ON O.CustomerId = C.CustomerId
ORDER BY O.OrderNo ASC;
```
### Keywords
* The use of the AS keyword depends on its context.
  * When aliasing column expressions, it should be used to create more visual separation rather than just relying on spaces and commas.
  * When aliasing tables, it should not be used as the ON keyword already provides some visual separation.
* All table operators should use two-word terms as it lines up vertically better.
  * `CROSS JOIN` does not have any alternatives.
  * Use `INNER JOIN` rather than just `JOIN` even though `INNER` is optional.
  * For outer joins, OUTER is optional, so use:
    * `LEFT JOIN` rather than `LEFT OUTER JOIN`
    * `RIGHT JOIN` rather than `RIGHT OUTER JOIN`
    * `FULL JOIN` rather than `FULL OUTER JOIN`

### Spacing & Indentation
* Each major element should be left aligned with one another.
```SQL
-- The high-level structure for all SELECT statements.
SELECT DISTINCT TOP(100) ...
FROM ...
   INNER JOIN ...
   LEFT JOIN ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...
```
* Each column expression in the SELECT should be on its own line and indented once under SELECT. Simple column names can share a line, as long as the line width is kept short.
* No use of SELECT * is acceptable unless it is used within a subquery as the input for EXISTS.
  * Frankly any use of the * symbol outside of exists and maybe `COUNT(*)` at times should be avoided if possible.
* Table operators, such as INNER JOIN, should be at the start of a new line and indented.
  * If the right operand is a table, view, or CTE, then it is simply listed to the right of the operator.
  * If the right operand is a derived table, then the subquery is to start on a new line and indented twice more than the operator.
  * In either case, if the first join predicate is short enough, it should go on the first line immediately following the ON keyword.
  * Subsequent predicates should each appear on their own line.
  * Each predicate that references the right table of the join should be on the left side of the operator used. For example, in a pattern of TableA A INNER JOIN TableB B ON B.Id = A.Id, notice the reference to the right table, B.Id, is on the left side of the = operator.
* When using the WHERE element, it should appear left aligned under the SELECT and FROM elements.
  * The first predicate can go on the first line immediately following the WHERE keyword and a single space.
  * Subsequent predicates should each appear on their own line, indented once under the WHERE keyword.
  * If a subquery is used, such as with the operator EXISTS, then the subquery should begin on a new line and indented one more than the start of the predicate within which it lies. If it is the first or only predicate, it should be indented twice from the start of the WHERE keyword.
```SQL
  WITH CustomerCte(CustomerId, FirstName, LastName, CreatedOn) AS
   (
      SELECT C.CustomerId, C.FirstName, C.LastName, C.CreatedOn
      FROM Sales.Customer C
      WHERE C.LastName LIKE @LastName + N'%'
         AND @PostalCode IS NULL
 
      UNION ALL
 
      SELECT C.CustomerId, C.FirstName, C.LastName, C.CreatedOn
      FROM Sales.Customer C
         INNER JOIN Sales.CustomerAddress BA ON BA.CustomerId = C.CustomerId
            AND BA.AddressType = 'Billing'
      WHERE C.LastName LIKE @LastName + N'%'
         AND BA.PostalCode = @PostalCode
   )
SELECT C.CustomerId, C.FirstName, C.LastName, C.CreatedOn, CC.ContactCount,
   C.FirstName + N' ' + C.LastName AS FullName, Name AS Territory,
   BA.City + N', ' + BA.[State] AS BillingCity,
   SA.City + N', ' + SA.[State] AS ShippingCity,
   SC.StoreCount AS AreaStoreCount
FROM Sales.Customer C
   INNER JOIN Sales.Territory T ON T.TerritoryId = C.TerritoryId
   INNER JOIN Sales.CustomerAddress BA ON BA.CustomerId = C.CustomerId
      AND BA.AddressType = 'Billing'
   INNER JOIN Sales.CustomerAddress SA ON SA.CustomerId = C.CustomerId
      AND SA.AddressType = 'Shipping'
   INNER JOIN
      (
         -- The subquery is indented twice.
         SELECT S.City, S.[State], COUNT(*) AS StoreCount
         FROM Sales.[Store] S
         GROUP BY S.City, S.[State]
      ) SC ON SC.City = BA.City
         AND SC.[State] = BA.[State]
WHERE EXISTS
   (
      -- This is a subquery with EXISTS, so it is acceptable to use SELECT *.
      -- Again, it is also indented twice.
      SELECT *
      FROM Sales.CustomerOrder O
      WHERE O.CustomerId = C.CustomerId
   )
-- Always use the table alias in prefixing columns just like in the SELECT element.
-- However, is ordering by an expression, you can use the alias name.
-- Also, be explicit with ASC vs. DESC.
ORDER BY FullName ASC, C.CustomerId ASC;
```
__Another key thing__ that would be good to know is what should be __absent__ from our code. Particularly:
* `BEGIN` and `END` bracketing. It just isn't needed in SQL. It might appeal to those that haven't been programming as much in TSQL, but the `BEGIN` and `END` keywords should only be used in the few cases where they are necessary. Otherwise they create cluttered code.
* `SET NOCOUNT OFF` can be spotted in several portions of our legacy code and never needs to be there. This happens by default and does not need to be appended to any Procedures one may happen to write.
Tip: Setting Spaces for Indentation

In order to better maintain proper indentation and spacing, set your SSMS or Visual Studio environment to use 3 spaces for tabs. You can accomplish this by going to Tools → Options, and then expand the section __Text Editor__, __All Languages__, and select __Tabs__. I recommend changing it for __All Languages__ because there are 2-3 tools/languages for which you would probably need to change it for SQL files alone, but also 3 spaces is the GTM's requirement for C#. If you need to change it to two spaces for other languages such as JavaScript, it's easier to override those individually as they would be more the exception.
![Options Dialog in Visual Studio](/images/vs-options-tabs.png)
### Query & Table Hints

__Query Hints__ and __Table Hints__ should be used sparingly, but when necessary, there should be no spaces preceding the parentheses. When accessing live databases, such as M3, PCM, or GTM, we should use the NOLOCK hint on each table.
We use NOLOCK because it speeds up the queries. It's also recommended by Infor for any M3 related queries.

Here is an example of a simple query hitting the PCM database with both table hints and a query hint. The query hint wouldn't normally be used in this case, but shown here to illustrate it's own line with no spaces around the parentheses.
```SQL
SELECT
   cfg.CfgId,
   cfg.HeaderId,
   cfg.ConfigurationId,
   cfg.InProgressInputXml,
   cfg.InProgressOutputXml,
   cfg.InputXml,
   cfg.OutputXml,
   cfg.IsPartial,
   cfg.ResourcePathRoot,
   cfg.ResourceLinkRoot,
   cfg.ApplicationServerId 
FROM BDCConfigurations cfg WITH(NOLOCK)
WHERE cfg.HeaderID = @HeaderID
   AND cfg.ConfigurationID LIKE @ConfigurationID + '%'
OPTION(MAXDOP 2);
```


