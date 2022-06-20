## 1. Beyond Summary Statistics

    SELECT
      ROUND(MIN(measure_value), 2) AS minimum_value,
      ROUND(MAX(measure_value), 2) AS maximum_value,
      ROUND(AVG(measure_value), 2) AS mean_value,
      ROUND(
        -- this function actually returns a float which is incompatible with ROUND!
        -- we use this cast function to convert the output type to NUMERIC
        CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
        2
      ) AS median_value,
      ROUND(
        MODE() WITHIN GROUP (ORDER BY measure_value),
        2
      ) AS mode_value,
      ROUND(STDDEV(measure_value), 2) AS standard_deviation,
      ROUND(VARIANCE(measure_value), 2) AS variance_value
    FROM health.user_logs
    WHERE measure = 'weight';


> How many records did we have again for measure = 'weight'? Does our 1%
> of values make sense in this case?

    SELECT
      COUNT(*)
    FROM health.user_logs
    WHERE measure = 'weight';

> --count
> --2782
> 
> 
> Order and Assign
> Order all of the weight measurement values from smallest to largest
> Split them into 100 equal buckets - and assign a number from 1 through to 100 for each bucket
> Firstly the OVER and ORDER BY clauses in the following query help us re-order the dataset by the measure_value column - it sorts by
> ascending order by default)
> Then the NTILE window function is used to perform the assignment of numbers 1 through 100 for each row in the records for each
> measure_value

    SELECT
      measure_value,
      NTILE(100) OVER (
        ORDER BY
          measure_value
      ) AS percentile
    FROM health.user_logs
    WHERE measure = 'weight'

> Bucket Calculations
> For each bucket:
> calculate the minimum value and the maximum value for the ceiling and floor values
> count how many records there are
> Since we now have our percentile values and our dataset is split into 100 buckets - we can simply use a GROUP BY on the percentile
> column from the previous table to calculate our MIN and MAX
> measure_value ranges for each bucket and also the COUNT of records for
> the percentile_counts field.
> We can also use the previous query in a CTE so we can pull all the calculations in a single SQL query:

    WITH percentile_values AS (
      SELECT
        measure_value,
        NTILE(100) OVER (
          ORDER BY
            measure_value
        ) AS percentile
      FROM health.user_logs
      WHERE measure = 'weight'
    )
    SELECT
      percentile,
      MIN(measure_value) AS floor_value,
      MAX(measure_value) AS ceiling_value,
      COUNT(*) AS percentile_counts
    FROM percentile_values
    GROUP BY percentile
    ORDER BY percentile;

## 2.6.1. Window Functions for Sorting Values

    WITH percentile_values AS (
      SELECT
        measure_value,
        NTILE(100) OVER (
          ORDER BY
            measure_value
        ) AS percentile
      FROM health.user_logs
      WHERE measure = 'weight'
    )
    SELECT
      measure_value,
       -- these are examples of window functions below
      ROW_NUMBER() OVER (ORDER BY measure_value DESC) as row_number_order,
      RANK() OVER (ORDER BY measure_value DESC) as rank_order,
      DENSE_RANK() OVER (ORDER BY measure_value DESC) as dense_rank_order
    FROM percentile_values
    WHERE percentile = 100
    ORDER BY measure_value DESC;


## 2.8. Small Outliers

    WITH percentile_values AS (
      SELECT
        measure_value,
        NTILE(100) OVER (
          ORDER BY
            measure_value
        ) AS percentile
      FROM health.user_logs
      WHERE measure = 'weight'
    )
    SELECT
      measure_value,
      ROW_NUMBER() OVER (ORDER BY measure_value) as row_number_order,
      RANK() OVER (ORDER BY measure_value) as rank_order,
      DENSE_RANK() OVER (ORDER BY measure_value) as dense_rank_order
    FROM percentile_values
    WHERE percentile = 1
    ORDER BY measure_value;

## 2.9. Removing Outliers

> Create a temporary table using a CREATE TEMP TABLE <> AS statement

    DROP TABLE IF EXISTS clean_weight_logs;
    CREATE TEMP TABLE clean_weight_logs AS (
      SELECT *
      FROM health.user_logs
      WHERE measure = 'weight'
        AND measure_value > 0
        AND measure_value < 201
    );


> Calculate summary statistics on this new temp table

    SELECT
      ROUND(MIN(measure_value), 2) AS minimum_value,
      ROUND(MAX(measure_value), 2) AS maximum_value,
      ROUND(AVG(measure_value), 2) AS mean_value,
      ROUND(
    
    >     -- this function actually returns a float which is incompatible with ROUND!
    >     -- we use this cast function to convert the output type to NUMERIC
    >     CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
    
        2
      ) AS median_value,
      ROUND(
        MODE() WITHIN GROUP (ORDER BY measure_value),
        2
      ) AS mode_value,
      ROUND(STDDEV(measure_value), 2) AS standard_deviation,
      ROUND(VARIANCE(measure_value), 2) AS variance_value
    FROM clean_weight_logs;

> Show the new cumulative distribution function with treated data

    WITH percentile_values AS (
      SELECT
        measure_value,
        NTILE(100) OVER (
          ORDER BY
            measure_value
        ) AS percentile
      FROM clean_weight_logs
    )
    SELECT
      percentile,
      MIN(measure_value) AS floor_value,
      MAX(measure_value) AS ceiling_value,
      COUNT(*) AS percentile_counts
    FROM percentile_values
    GROUP BY percentile
    ORDER BY percentile;


## 6.1. NTILE
### The top 10 rows when ordered by measure_value increasing:

    SELECT
      measure_value,
      NTILE(100) OVER (
        ORDER BY
          measure_value
      ) AS percentile
    FROM health.user_logs
    WHERE measure = 'weight'
    ORDER BY measure_value
    LIMIT 10;

> The final 10 rows when we order by measure_value descending

    SELECT
      measure_value,
      NTILE(100) OVER (
        ORDER BY
          measure_value
      ) AS percentile
    FROM health.user_logs
    WHERE measure = 'weight'
    ORDER BY measure_value DESC
    LIMIT 10;


## 6.2. CUME_DIST

> Top 10 rows ordered by measure_value

    SELECT
      measure_value,
      CUME_DIST() OVER (
        ORDER BY
          measure_value
      ) AS percentile
    FROM health.user_logs
    WHERE measure = 'weight'
    ORDER BY measure_value
    LIMIT 10;

> Final 10 rows ordered measure_value descending

    SELECT
      measure_value,
      CUME_DIST() OVER (
        ORDER BY
          measure_value
      ) AS percentile
    FROM health.user_logs
    WHERE measure = 'weight'
    ORDER BY measure_value DESC
    LIMIT 10;


## 6.3. Difference
> For example - if we wanted 10 buckets instead of 100 - we can apply
> the following changes:

    WITH decile_values AS (
      SELECT
        measure_value,
        -- ***** notice how this is now 10 instead of 100!
        NTILE(10) OVER (
          ORDER BY
            measure_value
        ) AS decile
      FROM health.user_logs
      WHERE measure = 'weight'
    )
    SELECT
      decile,
      MIN(measure_value) AS floor_value,
      MAX(measure_value) AS ceiling_value,
      COUNT(*) AS decile_counts
    FROM decile_values
    GROUP BY decile
    ORDER BY decile;




