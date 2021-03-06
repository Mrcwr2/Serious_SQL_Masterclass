> /*How many rows are there in the film_list table?*/ 
> /* Answer - Row
> Count 997*/

    SELECT
      COUNT(*) AS row_count
    FROM dvd_rentals.film_list;

> /*What are the unique values for the rating column in the film
> table?*/

    SELECT DISTINCT
      rating
    FROM dvd_rentals.film_list


> /*How many unique category values are there in the film_list table?*/

    SELECT
      COUNT(DISTINCT category) AS unique_category_count
    FROM dvd_rentals.film_list

> /*What is the frequency of values in the rating column in the
> film_list table?*/

## Simplified table example

    SELECT
      fid,
      title,
      category,
      rating,
      price
    FROM dvd_rentals.film_list
    LIMIT 10;

## Group by query using common table expression

    WITH example_table AS (
      SELECT
        fid,
        title,
        category,
        rating,
        price
      FROM dvd_rentals.film_list
      LIMIT 10
    )
    SELECT
      rating,
      COUNT(*) as record_count
    FROM example_table
    GROUP BY rating
    ORDER BY record_count DESC;


> What is the frequency of values in the rating column in the film
> table?

    SELECT
      rating,
      COUNT(*) AS frequency
    FROM dvd_rentals.film_list
    GROUP BY rating;


> Let’s take this one step further and sort our GROUP BY output by using an ORDER BY DESC clause to arrange the rating values by highest
> to lowest occurences.

    SELECT
      rating,
      COUNT(*) AS frequency
    FROM dvd_rentals.film_list
    GROUP BY rating
    ORDER BY frequency DESC;


> /*There are actually various ways to perform this operation but I will
> keep things simple and show you the most efficient way using a
> modified window function combining both SUM and COUNT functions with
> an OVER() clause. Take note of that ::NUMERIC right after the COUNT(*)
> this is to avoid the dreaded integer floor division which is covered in much more depth in the appendix!*/

    SELECT
      rating,
      COUNT(*) AS frequency,
      COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER () AS percentage
    FROM dvd_rentals.film_list
    GROUP BY rating
    ORDER BY frequency DESC;


## Round Function

> /*multiple this percentage value by 100 and round it to 1 decimal
> place using a ROUND function*/

    SELECT
      rating,
      COUNT(*) AS frequency,
      ROUND(
        100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (),
        2
      ) AS percentage
    FROM dvd_rentals.film_list
    GROUP BY rating
    ORDER BY frequency DESC;

## GROUP BY on 2+ columns

> What are the 5 most frequent rating and category combinations in the
> film_list table?

    SELECT
      rating,
      category,
      COUNT(*) AS frequency
    FROM dvd_rentals.film_list
    GROUP BY rating, category
    ORDER BY frequency DESC
    LIMIT 5;



> Using Positional Numbers Instead of Column Names

## GROUP BY and ORDER BY clauses by the positional number

    SELECT
      rating,
      category,
      COUNT(*) AS frequency
    FROM dvd_rentals.film_list
    GROUP BY 1,2


> Which actor_id has the most number of unique film_id records in the
> dvd_rentals.film_actor table?

    SELECT
      actor_id,
      COUNT(DISTINCT film_id)
    FROM dvd_rentals.film_actor
    GROUP BY 1
    ORDER BY 2 DESC
    LIMIT 5;


> How many distinct fid values are there for the 3rd most common price
> value in the dvd_rentals.nicer_but_slower_film_list table?

    SELECT
      price,
      COUNT(DISTINCT fid)
    FROM dvd_rentals.nicer_but_slower_film_list
    GROUP BY 1
    ORDER BY 2 DESC;



> How many unique country_id values exist in the dvd_rentals.city table?

    SELECT
      COUNT(DISTINCT country_id) AS unique_countries
    FROM dvd_rentals.city;



> What percentage of overall total_sales does the Sports category make
> up in the dvd_rentals.sales_by_film_category table?

    SELECT
      category,
      ROUND(
        100 * total_sales::NUMERIC / SUM(total_sales) OVER (),
        2
      ) AS percentage
    FROM dvd_rentals.sales_by_film_category;


> What percentage of unique fid values are in the Children category in
> the dvd_rentals.film_list table?

    SELECT
      category,
      ROUND(
        100 * COUNT(DISTINCT fid)::NUMERIC / SUM(COUNT(DISTINCT fid)) OVER (),
        2
      ) AS percentage
    FROM dvd_rentals.film_list
    GROUP BY category
    ORDER BY category;




