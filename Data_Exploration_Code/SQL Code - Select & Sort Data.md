## Select & Sort Data

> /*What is the name of the category with the highest category_id in the
> dvd_rentals.category table?*/ /* Excersize 1*/ /* Answer - Travel*/

    Select
      category_id,
      name
    FROM
      dvd_rentals.category
    ORDER BY
      category_id DESC
    Limit
      8;


> /*For the films with the longest length, what is the title of the “R”
> rated film with the lowest replacement_cost in dvd_rentals.film
> table?*/ /*Answer - HOME PITY*/

    Select
      length,
      title,
      rating,
      replacement_cost
    FROM
      dvd_rentals.film
    ORDER BY length DESC, replacement_cost
    Limit
      8;


> /*Who was the manager of the store with the highest total_sales in the
> dvd_rentals.sales_by_store table?*/ /*Answer - Jon Stephens*/

    Select
     *
    FROM
      dvd_rentals.sales_by_store
    ORDER BY total_sales
    DESC, total_sales
    Limit
      8;


>   /*What is the postal_code of the city with the 5th highest city_id
> in the dvd_rentals.address table?*/   /*Answer - 31390*/

      Select
       postal_code,
       city_id
      FROM
        dvd_rentals.address
      ORDER BY city_id
      DESC;

