### Full Text Search

- we use the `pg_vector` docker files to bring up a postgres instance
- run these sql command to create a new table for storing CBD products :

```sql
CREATE DATABASE IF NOT EXISTS cbd_store;


CREATE TABLE IF NOT EXISTS products (
    product_id VARCHAR(255) PRIMARY KEY,
    product_name VARCHAR(255),
    seo_desc TEXT,
    category VARCHAR(255)
);


```

- Populate the above DB (*install requirements.txt and then run populate_db.py*)

### Search Text

- search for products suitable for sleep :

  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE LOWER(seo_desc) LIKE '%sleep%';
  ```

- what about this :

      ```sql
      SELECT product_id, product_name
      FROM products
      WHERE LOWER(seo_desc) LIKE '%I need some products to help me sleep better %';
      ```



#### Add Full Text  Search Support 

- do these steps :

  ```sql
  -- Add a tsvector column
  ALTER TABLE products ADD COLUMN seo_desc_vector tsvector;
  
  -- Update the tsvector column with the text from the seo_desc column
  UPDATE products SET seo_desc_vector = to_tsvector('english', coalesce(seo_desc, ''));
  
  -- Create a GIN index on the tsvector column
  CREATE INDEX idx_seo_desc_vector ON products USING gin(seo_desc_vector);
  
  ```

- what is in the seo_desc_vector:

  ```sql
select seo_desc_vector from products limit 1  ;
  ```

  

- Let do some search :

  ```sql 
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'I & need & some & products & to & help & me & sleep & better');
  ```

-   Or  operator

  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'I | need | some | products | to | help | me | sleep | better');
  
  ```
  
  

  ### TS Vector Basic Samples
  
   Full-text search in PostgreSQL comes with several practical operators and functions that you can use to enhance your search capabilities. Let's go through some of them using the given dataset.
  
  Assuming you have already created a full-text search index on the `seo_desc` column and a `tsvector` column named `seo_desc_vector`, as demonstrated earlier, here are some practical operators and samples:
  
  ### 1. **Basic Text Search:**
  
  #### Query:
  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE to_tsvector('english', seo_desc) @@ to_tsquery('english', 'sleep');
  
  
  or 
  
  
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'sleep');
  
  
  ```
  
  This query searches for the occurrence of the `search_term` in the `seo_desc` column using full-text search.
  
  ### 2. **Phrase Search:**
  
  #### Query:
  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', '''sleep disorder''');
  ```
  
  This query looks for the exact occurrence of the phrase "sleep disorder" in the `seo_desc` column.
  
  try : 'Sleep Bath'
  
  ### 3. **AND Operator:**
  
  #### Query:
  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'sleep & salt');
  ```
  
  This query searches for records containing both `term1` and `term2` in the `seo_desc` column.
  
  ### 4. **OR Operator:**
  
  #### Query:
  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'sleep | bath');
  ```
  
  This query searches for records containing either `term1` or `term2` in the `seo_desc` column.
  
  ### 5. **NOT Operator:**
  
  #### Query:
  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'bath & !sleep');
  ```
  
  This query searches for records containing `bath` but not `sleep` in the `seo_desc` column.
  
  ### 6. **Weighting Terms:**
  
  #### Query:
  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'sleep:A | bath:B');
  ```
  
  This query assigns different weights (A and B) to terms `term1` and `term2`, influencing the ranking of results.
  
  This query is attempting to assign a higher importance (weight A) to the term "sleep" and a slightly lower importance (weight B) to the term "bath". However, in practical terms, the difference in rank between the terms with weights A and B might not be significant.
  
  The available weight classes in PostgreSQL are:
  
  - `A` (highest weight)
  - `B`
  - `C`
  - `D` (default weight)
  - `E`
  - `F` (lowest weight)
  
  ### 7. **Ranking Results:**
  
  #### Query:
  ```sql
  SELECT product_id, product_name, ts_rank(seo_desc_vector, to_tsquery('english', 'Sleep | Bath')) AS rank
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'Sleep | Bath')
  ORDER BY rank DESC;
  ```
  
  This query includes the `ts_rank` function to calculate the rank of the search results based on their relevance to the search term.
  
  ### 8. **Searching with Stop Words:**
  
  #### Query:
  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'to & sleep');
  ```
  
  This query uses the English stop word list to exclude common words from the search term. (default behaviors)
  
  ### 9. **Prefix Search:**
  
  #### Query:
  ```sql
  SELECT product_id, product_name
  FROM products
  WHERE seo_desc_vector @@ to_tsquery('english', 'sleep:*');
  ```
  
  This query searches for terms with a specified prefix, such as searching for words starting with "prefix."
  
  These examples cover various aspects of full-text search in PostgreSQL. You can adapt these queries based on your specific requirements and dataset characteristics.