## 3.1. Arithmetic Mean or Average

    SELECT
      AVG(measure_value)
    FROM health.user_logs;

> what were our measures called again and how many record counts were
> there?

    SELECT
      measure,
      COUNT(*) AS counts
    FROM health.user_logs
    GROUP BY measure
    ORDER BY counts DESC;

> What happens if we also take a look at the AVG value across each
> measure too?

    SELECT
      measure,
      ROUND(AVG(measure_value), 2) AS average,
      COUNT(*) AS counts
    FROM health.user_logs
    GROUP BY measure
    ORDER BY counts DESC;

## Mode Algorithm

> Calculate the tally of values similar to a GROUP BY and COUNT. The
> mode is the values with the highest number of occurrences

    WITH sample_data (example_values) AS (
     VALUES
     (82), (51), (144), (84), (120), (148), (148), (108), (160), (86)
    )
    SELECT
      AVG(example_values) AS mean_value,
      PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY example_values) AS median_value,
      MODE() WITHIN GROUP (ORDER BY example_values) AS mode_value
    from sample_data;

> Let’s also use this same syntax to calculate our median and mode
> values for our health.user_logs dataset for the weight measurements
> only - what happens when we compare it with the average value?

    SELECT
      PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS median_value,
      MODE() WITHIN GROUP (ORDER BY measure_value) AS mode_value,
      AVG(measure_value) as mean_value
    FROM health.user_logs
    WHERE measure = 'weight';

## 5.1. Min, Max & Range

    SELECT
      MIN(measure_value) AS minimum_value,
      MAX(measure_value) AS maximum_value,
      MAX(measure_value) - MIN(measure_value) AS range_value
    FROM health.user_logs
    WHERE measure = 'weight';


> You can confirm this speedup by putting a EXPLAIN ANALYZE in front of
> each query and taking a look at the output, especially the Execution
> Time values.

    EXPLAIN ANALYZE
    WITH min_max_values AS (
      SELECT
        MIN(measure_value) AS minimum_value,
        MAX(measure_value) AS maximum_value
      FROM health.user_logs
      WHERE measure = 'weight'
    )
    SELECT
      minimum_value,
      maximum_value,
      maximum_value - minimum_value AS range_value
    FROM min_max_values;


## 6.1. Variance Algorithm

    WITH sample_data (example_values) AS (
     VALUES
     (82), (51), (144), (84), (120), (148), (148), (108), (160), (86)
    )
    SELECT
      ROUND(VARIANCE(example_values), 2) AS variance_value,
      ROUND(STDDEV(example_values), 2) AS standard_dev_value,
      ROUND(AVG(example_values), 2) AS mean_value,
      PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY example_values) AS median_value,
      MODE() WITHIN GROUP (ORDER BY example_values) AS mode_value
    FROM sample_data;


## 7. Calculating All The Summary Statistics

    SELECT
      'weight' AS measure,
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

> What is the average, median and mode values of blood glucose values to
> 2 decimal places?

    SELECT
      ROUND(AVG(measure_value), 2) AS average_value,
      ROUND(
        -- this function actually returns a float which is incompatible with ROUND!
        -- we use this cast function to convert the output type to NUMERIC
        CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
        2
      ) AS median_value,
      ROUND(
        MODE() WITHIN GROUP (ORDER BY measure_value),
        2
      ) AS mode_value
    FROM health.user_logs
    WHERE measure = 'blood_glucose';





# 8.2. Most Frequent Values


