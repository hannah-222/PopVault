# POPVAULT

## A pocket vault for your Funko Pops





### DATABASE SUMMARY:


###### PopVault is a database I created and designed for Funko Pop collectors of all levels, from casual fans to serious investors. The inspiration for PopVault came from a real collector's frustration. A friend of mine with 342 Funko Pops constantly faced the problem of buying duplicate figures or not buying a figure because he couldn't remember what he already owned. Walking into a comic store and seeing an interesting FunkoPop on the shelf, he would struggle to recall if it was already sitting in a box at home or displayed on one of his many shelves. I soon realized this highlighted the need for a portable, searchable database that collectors could reference anywhere, whether browsing online stores or walking the aisles of their local shop. PopVault solves this problem by putting a collector's entire inventory in their pocket, accessible anytime they need to make a purchasing decision.



#### User's perspective:

###### When collectors join PopVault, they create a profile and build their digital inventory by adding Funko Pops with details like purchase date, price, condition, and storage location. Users can browse by franchise to see what they own and identify missing figures, while the system automatically calculates their total collection value. This organized catalog transforms hundreds of physical pops scattered across shelves and storage boxes into a searchable, manageable database.

###### 

###### The real value shows during shopping trips. Collectors can quickly search for a specific pop on their phone to see if they already own it or if it's on their wishlist, solving the common problem of buying duplicates or forgetting what's at home. Users manage wishlists with priority levels (1-5) and generate reports showing their most valuable pieces, spending over time, or collection completion by franchise. PopVault puts a collector's entire inventory in their pocket, making informed purchasing decisions easy and eliminating the frustration of forgotten purchases.


### SQL Queries:





1. SELECT using ORDER BY two or more columns

Retrieves all Funko Pops ordered first by franchise (grouped together), then by rarity level within each franchise, and finally by estimated value. This is useful when browsing the entire catalog organized by franchise and value, helping collectors identify the most valuable pops within each franchise.

```sql
SELECT fp.pop_name, f.franchise_name, fp.rarity, fp.estimated_value
FROM funko_pops fp
JOIN franchises f ON fp.franchise_id = f.franchise_id
ORDER BY f.franchise_name, fp.rarity DESC, fp.estimated_value DESC
LIMIT 15;
```



2.SELECT using a calculated field with meaningful column heading

&nbsp;This query calculates how many years ago each Funko Pop was released by subtracting the release year from the current year (2024). This helps collectors quickly identify vintage pops versus newer releases, which is useful since older pops often have higher collectible value.

```sql
SELECT pop_name,
       release_year,
       (2025 - release_year) AS years_old
FROM funko_pops
ORDER BY years_old DESC
LIMIT 10;
```



3\.  SELECT using a MariaDB function

This query uses the UPPER() function to display all collector names in uppercase letters. This is useful for generating mailing labels, creating standardized reports, or ensuring consistent formatting when exporting data to other systems.

```sql
--sql
SELECT UPPER(first_name) AS first_name_upper,
       UPPER(last_name) AS last_name_upper,
       email
FROM collectors
LIMIT 10;
```



4\. SELECT with aggregation plus GROUP BY and HAVING

This query counts how many Funko Pops exist in each rarity category and only shows rarity levels that have more than 10 pops. This helps collectors understand which rarity levels are most common in the database and identify which categories have substantial inventory.

```sql
SELECT rarity,
       COUNT(*) AS pop_count
FROM funko_pops
GROUP BY rarity
HAVING COUNT(*) > 10
ORDER BY pop_count DESC;
```



5\. JOIN of THREE or more tables 

This query joins collectors, vault, funko\_pops, and franchises to show a complete view of who owns which pops from which franchises, including condition and storage details. This comprehensive report is useful for generating inventory reports or understanding the full context of a collection across multiple data dimensions.

```sql
SELECT c.first_name, c.last_name,
       f.franchise_name,
       fp.pop_name,
       v.pop_condition,
       v.storage_location,
       v.purchase_price
FROM vault v
JOIN collectors c ON v.collector_id = c.collector_id
JOIN funko_pops fp ON v.pop_id = fp.pop_id
JOIN franchises f ON fp.franchise_id = f.franchise_id
WHERE c.collector_id = 7
ORDER BY f.franchise_name, fp.pop_name
LIMIT 10;
```



6\. Left or Right JOIN

This LEFT JOIN query shows all franchises and how many Funko Pops exist in the database for each franchise, including franchises that don't have any pops yet (showing 0). This is useful for identifying which franchises need more pops added to the catalog or which franchises are underrepresented.

```sql
SELECT f.franchise_name,
       COUNT(fp.pop_id) AS total_pops
FROM franchises f
LEFT JOIN funko_pops fp ON f.franchise_id = fp.franchise_id
GROUP BY f.franchise_id, f.franchise_name
ORDER BY total_pops DESC;
```




7\. Update Query

This UPDATE query changes the storage location for all pops currently stored in "Basement Box 3" to "Storage Unit A". This would be used when physically reorganizing a collection, such as moving items from home storage to a rented storage unit, ensuring the database reflects the actual physical location.

```sql
UPDATE vault
SET storage_location = 'Storage Unit A'
WHERE storage_location = 'Basement Box 3';
```
VERIFY: 
(should be empty result)
```sql
SELECT * FROM vault WHERE storage_location = 'Storage Unit A';
```



8\. Delete Query

This DELETE query removes all vault entries where the pop is in "Fair" condition and was purchased for less than $20. This could be used when decluttering a collection by removing low-value damaged items, or when a collector decides to only keep high-quality pieces in their database.

```sql
DELETE FROM vault
WHERE pop_condition = 'Fair'
AND purchase_price < 20;
```


9\. Create a View and demonstrate using it

&nbsp;This view creates a virtual table showing each collector's collection summary statistics including total pops owned, total money spent, and average price per pop. Views are useful for simplifying complex queries that are run frequently, and this particular view helps quickly assess collection size and investment levels without writing the full query each time.

```sql
CREATE VIEW collection_summary AS
SELECT c.collector_id,
       c.first_name,
       c.last_name,
       COUNT(v.vault_id) AS total_pops,
       SUM(v.purchase_price) AS total_invested,
       AVG(v.purchase_price) AS avg_pop_price
FROM collectors c
LEFT JOIN vault v ON c.collector_id = v.collector_id
GROUP BY c.collector_id, c.first_name, c.last_name;
```
To view:
```sql
SELECT * FROM collection_summary
WHERE total_pops > 0
ORDER BY total_invested DESC
LIMIT 10;
```



10\. Create a Transaction with ROLLBACK or COMMIT

This transaction demonstrates adding a new Funko Pop purchase to the vault and updating the collector's total collection value atomically. If any part fails (like insufficient funds or invalid data), the ROLLBACK ensures the database stays consistent. The COMMIT saves all changes permanently only if everything works. 

```sql
-- Start transaction
START TRANSACTION;

-- Add a new pop to Bruce Wayne's vault
INSERT INTO vault (collector_id, pop_id, purchase_date, purchase_price, pop_condition, storage_location)
VALUES (7, 10, '2024-11-29', 380.00, 'Mint', 'Safe');

-- Update Bruce Wayne's total collection value
UPDATE collectors
SET total_collection_value = total_collection_value + 380.00
WHERE collector_id = 7;

-- Verify the changes look correct
SELECT first_name, last_name, total_collection_value
FROM collectors
WHERE collector_id = 7;

-- If everything looks good, commit the transaction
COMMIT;

-- If there was an error, you would use: ROLLBACK;
```





#### Delete Tables:
MUST DO IN ORDER
(may need to do one query at a time)
```sql
DROP VIEW collection_summary;
DROP TABLE vault;
DROP TABLE funko_pops;
DROP TABLE franchises;
DROP TABLE collectors;
```










































