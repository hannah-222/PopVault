# POPVAULT

## ----------------------

## A new way of storing. 

## ----------------------





### DATABASE SUMMARY:

------------------------------------------

###### PopVault is a database I created designed for Funko Pop collectors of all levels, from casual fans to serious investors. The inspiration for PopVault came from a real collector's frustration: a friend of mine with 342 Funko Pops constantly faced the problem of buying duplicate figures or not buying a figure because he couldn't remember what he already owned. Walking into a comic store and seeing an interesting FunkoPop on the shelf, he would struggle to recall if it was already sitting in a box at home or displayed on one of his many shelves. I soon realized this highlighted the need for a portable, searchable database that collectors could reference anywhere, whether browsing online stores or walking the aisles of their local shop. PopVault solves this problem by putting a collector's entire inventory in their pocket, accessible anytime they need to make a purchasing decision.



#### User's perspective:

###### When collectors join PopVault, they create a profile and build their digital inventory by adding Funko Pops with details like purchase date, price, condition, and storage location. Users can browse by franchise to see what they own and identify missing figures, while the system automatically calculates their total collection value. This organized catalog transforms hundreds of physical pops scattered across shelves and storage boxes into a searchable, manageable database.

###### 

###### The real value shows during shopping trips. Collectors can quickly search for a specific pop on their phone to see if they already own it or if it's on their wishlist, solving the common problem of buying duplicates or forgetting what's at home. Users manage wishlists with priority levels (1-5) and generate reports showing their most valuable pieces, spending over time, or collection completion by franchise. PopVault puts a collector's entire inventory in their pocket, making informed purchasing decisions easy and eliminating the frustration of forgotten purchases.

## -------------------------------------------------------------

### 

### SQL Queries:





1. SELECT using ORDER BY two or more columns

Retrieves all Funko Pops ordered first by franchise (grouped together), then by rarity level within each franchise, and finally by estimated value. This is useful when browsing the entire catalog organized by franchise and value, helping collectors identify the most valuable pops within each franchise.



---sql

SELECT fp.pop\_name, f.franchise\_name, fp.rarity, fp.estimated\_value

FROM funko\_pops fp

JOIN franchises f ON fp.franchise\_id = f.franchise\_id

ORDER BY f.franchise\_name, fp.rarity DESC, fp.estimated\_value DESC

LIMIT 15;

---



2.SELECT using a calculated field with meaningful column heading

&nbsp;This query calculates how many years ago each Funko Pop was released by subtracting the release year from the current year (2024). This helps collectors quickly identify vintage pops versus newer releases, which is useful since older pops often have higher collectible value.



---sql

SELECT pop\_name, 

&nbsp;      release\_year,

&nbsp;      (2025 - release\_year) AS years\_old

FROM funko\_pops

ORDER BY years\_old DESC

LIMIT 10;

---



3\.  SELECT using a MariaDB function

This query uses the UPPER() function to display all collector names in uppercase letters. This is useful for generating mailing labels, creating standardized reports, or ensuring consistent formatting when exporting data to other systems.



--sql 

SELECT UPPER(first\_name) AS first\_name\_upper,

&nbsp;      UPPER(last\_name) AS last\_name\_upper,

&nbsp;      email

FROM collectors

LIMIT 10;

---



4\. SELECT with aggregation plus GROUP BY and HAVING

This query counts how many Funko Pops exist in each rarity category and only shows rarity levels that have more than 10 pops. This helps collectors understand which rarity levels are most common in the database and identify which categories have substantial inventory.



--sql

SELECT rarity,

&nbsp;      COUNT(\*) AS pop\_count

FROM funko\_pops

GROUP BY rarity

HAVING COUNT(\*) > 10

ORDER BY pop\_count DESC;

---



5\. JOIN of THREE or more tables 

This query joins collectors, vault, funko\_pops, and franchises to show a complete view of who owns which pops from which franchises, including condition and storage details. This comprehensive report is useful for generating inventory reports or understanding the full context of a collection across multiple data dimensions.



---sql 

SELECT c.first\_name, c.last\_name, 

&nbsp;      f.franchise\_name, 

&nbsp;      fp.pop\_name, 

&nbsp;      v.pop\_condition,

&nbsp;      v.storage\_location,

&nbsp;      v.purchase\_price

FROM vault v

JOIN collectors c ON v.collector\_id = c.collector\_id

JOIN funko\_pops fp ON v.pop\_id = fp.pop\_id

JOIN franchises f ON fp.franchise\_id = f.franchise\_id

WHERE c.collector\_id = 7

ORDER BY f.franchise\_name, fp.pop\_name

LIMIT 10;

---



6\. Left or Right JOIN

This LEFT JOIN query shows all franchises and how many Funko Pops exist in the database for each franchise, including franchises that don't have any pops yet (showing 0). This is useful for identifying which franchises need more pops added to the catalog or which franchises are underrepresented.



---sql 

SELECT f.franchise\_name,

&nbsp;      COUNT(fp.pop\_id) AS total\_pops

FROM franchises f

LEFT JOIN funko\_pops fp ON f.franchise\_id = fp.franchise\_id

GROUP BY f.franchise\_id, f.franchise\_name

ORDER BY total\_pops DESC;

---



7\. Update Query

This UPDATE query changes the storage location for all pops currently stored in "Basement Box 3" to "Storage Unit A". This would be used when physically reorganizing a collection, such as moving items from home storage to a rented storage unit, ensuring the database reflects the actual physical location.



---sql 

UPDATE vault

SET storage\_location = 'Storage Unit A'

WHERE storage\_location = 'Basement Box 3';

---



VERIFY:

--sql

SELECT \* FROM vault WHERE storage\_location = 'Storage Unit A';

---



8\. Delete Query

This DELETE query removes all vault entries where the pop is in "Fair" condition and was purchased for less than $20. This could be used when decluttering a collection by removing low-value damaged items, or when a collector decides to only keep high-quality pieces in their database.



---sql

DELETE FROM vault

WHERE pop\_condition = 'Fair' 

AND purchase\_price < 20;

---



9\. Create a View and demonstrate using it

&nbsp;This view creates a virtual table showing each collector's collection summary statistics including total pops owned, total money spent, and average price per pop. Views are useful for simplifying complex queries that are run frequently, and this particular view helps quickly assess collection size and investment levels without writing the full query each time.



---sql

CREATE VIEW collection\_summary AS

SELECT c.collector\_id,

&nbsp;      c.first\_name,

&nbsp;      c.last\_name,

&nbsp;      COUNT(v.vault\_id) AS total\_pops,

&nbsp;      SUM(v.purchase\_price) AS total\_invested,

&nbsp;      AVG(v.purchase\_price) AS avg\_pop\_price

FROM collectors c

LEFT JOIN vault v ON c.collector\_id = v.collector\_id

GROUP BY c.collector\_id, c.first\_name, c.last\_name;

---



---sql

SELECT \* FROM collection\_summary

WHERE total\_pops > 0

ORDER BY total\_invested DESC

LIMIT 10;

---



10\. Create a Transaction with ROLLBACK or COMMIT

This transaction demonstrates adding a new Funko Pop purchase to the vault and updating the collector's total collection value atomically. If any part fails (like insufficient funds or invalid data), the ROLLBACK ensures the database stays consistent. The COMMIT saves all changes permanently only if everything works. 



---sql 



-- Start transaction

START TRANSACTION;



-- Add a new pop to Bruce Wayne's vault

INSERT INTO vault (collector\_id, pop\_id, purchase\_date, purchase\_price, pop\_condition, storage\_location)

VALUES (7, 10, '2024-11-29', 380.00, 'Mint', 'Safe');



-- Update Bruce Wayne's total collection value

UPDATE collectors

SET total\_collection\_value = total\_collection\_value + 380.00

WHERE collector\_id = 7;



-- Verify the changes look correct

SELECT first\_name, last\_name, total\_collection\_value 

FROM collectors 

WHERE collector\_id = 7;



-- If everything looks good, commit the transaction

COMMIT;



-- If there was an error, you would use: ROLLBACK;



---





#### Delete Tables:

MUST DO ONE AT A TIME IN ORDER 



--sql

DROP VIEW collection\_summary;

DROP TABLE vault;

DROP TABLE funko\_pops;

DROP TABLE franchises;

DROP TABLE collectors;

---







































