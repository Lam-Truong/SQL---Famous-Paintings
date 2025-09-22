# SQL---Famous-Paintings
This project uses Python to retrieve multiple CSV files and load it in to Postgres Database. And uses SQL querying to answer a list of business questions.

Below is a diagram I created using [drawsql](https://drawsql.app/) - an online drag and drop relational diagram builder.

<img width="3484" height="2704" alt="paintings_diagram" src="https://github.com/user-attachments/assets/3392a3cb-98f6-4930-837d-f867c22fff13" />

### Question 1: Fetch all the paintings which are not displayed on any museums?

``` sql
SELECT *
FROM work
WHERE museum_id IS NULL;
```

Query Result:

<img width="2074" height="1201" alt="image" src="https://github.com/user-attachments/assets/5596eeb0-c5e4-4d1a-b889-7ebcbf047eb4" />

There are 10223 paintings that are not displayed on any museums as their museum_id is NULL.

### Question 2: Are there museums without any paintings?

``` sql
SELECT
	m.name
FROM museum m LEFT JOIN work w
ON m.museum_id = w.museum_id
WHERE w.work_id IS NULL;
```

**Thought process**:
To find the number of museum with no paintings we can use LEFT JOIN (taking all rows from museum table) to work table on the right. And filter out ROWS which has work_id is NULL. Because matching rows will displayed with information on the right table whereas NULL indicate no matching rows on the right (no paintings).

**Query Result**:
It results in no rows meaning no instance of museum without any paintings.

### Question 3: How many paintings have an asking price of more than their regular price? 

``` sql
SELECT
	COUNT(*)
FROM product_size
WHERE sale_price > regular_price;
```

**Thought process**: The information of sale_price (asking price) and regular price can be found in the product_size table. We simply filter out in the WHERE clause by the condition and use COUNT() to count the number of instances.

**Query result**:
<img width="2069" height="581" alt="image" src="https://github.com/user-attachments/assets/2cbd1772-f5bb-4412-b7b4-027298231d63" />
It turns out that there is no paitnings with an asking price of more than their regular price.

### Question 4: Identify the paintings whose asking price is less than 50% of its regular price

``` sql
with identified_painting AS (
	SELECT *
	FROM product_size
	WHERE sale_price <= (regular_price / 2)
	)
SELECT
	w.work_id,
	w.name,
	i.sale_price,
	i.regular_price
FROM work w join identified_painting i
ON w.work_id = i.work_id;
```

**Thought process**:
- First I find out all the rows where sale_price is smaller than half of the regular_price uses CTEs.
- Then I join work table with that CTEs and display the work_id, name, sale_price and asking_price of such paintings.

**Query result**:
<img width="2068" height="1191" alt="image" src="https://github.com/user-attachments/assets/75370af2-6b6d-4d83-8f5b-29d942c25029" />
Notice that there are duplicated values in the query results!


### Question 5: Which canva size costs the most

``` sql
SELECT DISTINCT
	c.size_id,
	c.width,
	c.height,
	c.label,
	AVG(p.regular_price) OVER(PARTITION BY c.size_id)
FROM product_size p JOIN canvas_size c
ON p.size_id = c.size_id
ORDER BY 5 DESC
LIMIT 5;
```

**Thought process**: 
- We will use average regular price to measure cost by each size_id.
- We first join the the two tables product_size and canvas_size together.
- Then uses window function to find out the average price for each size_id.
- And display the information sorted by the average_price in descending order so that the most expensive canva will come first.

**Query result**:
<img width="2052" height="1201" alt="image" src="https://github.com/user-attachments/assets/e120ebca-c0a9-4528-bad7-01559ebc4b74" />

### Question 6: Delete duplicate records from work, product_size, subject and image_link tables

This query shows the number of duplicates value (by work_id - primary key of the table).
``` sql
SELECT COUNT(*)
FROM 
	(SELECT
		COUNT(*)
	FROM work
	GROUP BY work_id)
WHERE count >= 2;
```
<img width="2077" height="274" alt="image" src="https://github.com/user-attachments/assets/f3f8852c-e9d5-4dc6-85aa-a05496459111" />
--> There are 59 duplicates in the table.


``` sql
WITH ranked AS (
	SELECT
		ctid, -- Is the system hidden unique identifier for each row.
		*,
		ROW_NUMBER() OVER(PARTITION BY work_id ORDER BY ctid) AS row_num -- More than 1 number means duplicate
	FROM work)
DELETE FROM work
WHERE ctid IN (SELECT ctid FROM ranked WHERE row_num >= 2);
```

<img width="2071" height="209" alt="image" src="https://github.com/user-attachments/assets/28e6712c-fd4e-447e-b355-e06face4f70e" />
--> Deleted 60 duplicated rows.












