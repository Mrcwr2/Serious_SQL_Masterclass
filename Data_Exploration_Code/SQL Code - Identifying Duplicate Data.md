## Dataset Inspection

> Let’s inspect a snapshot to view the first 10 rows from the
> health.user_logs
> 
> table:

    SELECT *
    FROM health.user_logs
    LIMIT 10;

## Record Counts

> Let’s also take a quick look at a few basic counts to get a good feel
> for our dataset.

    SELECT COUNT(*)
    FROM health.user_logs;

## Unique Column Counts

> Let’s use the COUNT DISTINCT to take a look at how many unique id
> values there are in this dataset.

    SELECT COUNT(DISTINCT id)
    FROM health.user_logs;

## Single Column Frequency Counts

> Let’s also inspect that measure column and take a look at the most
> frequent values within this column using a GROUP BY and ORDER BY DESC
> combo from the last tutorial - let’s also throw in that percentage
> column that we went through also!

    SELECT
      measure,
      COUNT(*) AS frequency,
      ROUND(
        100 * COUNT(*) / SUM(COUNT(*)) OVER (),
        2
      ) AS percentage
    FROM health.user_logs
    GROUP BY measure
    ORDER BY frequency DESC;

> Let’s do this for the id column too and limit the output to just the
> first 10 rows.

    SELECT
      id,
      COUNT(*) AS frequency,
      ROUND(
        100 * COUNT(*) / SUM(COUNT(*)) OVER (),
        2
      ) AS percentage
    FROM health.user_logs
    GROUP BY id
    ORDER BY frequency DESC
    LIMIT 10;


## Individual Column Distributions

> Measure Column

    SELECT
      measure_value,
      COUNT(*) AS frequency
    FROM health.user_logs
    GROUP BY 1
    ORDER BY 2 DESC
    LIMIT 10;

## Systolic

    SELECT
      systolic,
      COUNT(*) AS frequency
    FROM health.user_logs
    GROUP BY 1
    ORDER BY 2 DESC
    LIMIT 10;

## Diastolic

    SELECT
      diastolic,
      COUNT(*) AS frequency
    FROM health.user_logs
    GROUP BY 1
    ORDER BY 2 DESC
    LIMIT 10;

> To inspect these rows a bit further - we can use a simple WHERE filter and check if this only happens for certain measure values when
> the condition measure_value = 0 is met and the systolic and diastolic
> columns are null.

    SELECT
      measure,
      COUNT(*)
    FROM health.user_logs
    WHERE measure_value = 0
    GROUP BY 1;


> It looks like most of those measure_value = 0 are occuring when
> measure = 'blood_pressure' We can dig in a little bit further and
> inspect a few rows for both of these conditions:

    SELECT *
    FROM health.user_logs
    WHERE measure_value = 0
    AND measure = 'blood_pressure'
    LIMIT 10;


> But what happened to the records where measure = 'blood_pressure' but
> the measure_value != 0 ?

    SELECT *
    FROM health.user_logs
    WHERE measure_value != 0
    AND measure = 'blood_pressure'
    LIMIT 10;


> Let’s finish this off by also looking at the null systolic and
> diastolic values in the same manner

    SELECT
      measure,
      COUNT(*)
    FROM health.user_logs
    WHERE systolic IS NULL
    GROUP BY 1;

> Can we confirm this is the fact for the diastolic column too?

    SELECT
      measure,
      COUNT(*)
    FROM health.user_logs
    WHERE diastolic IS NULL
    GROUP BY 1;


## Detecting Duplicate Records

> The first ingredient for this recipe is the basic record count for our
> table - plain and simple using the COUNT(*) just like we did above.

    SELECT COUNT(*)
    FROM health.user_logs;

## Remove All Duplicates

    SELECT DISTINCT *
    FROM health.user_logs;


## 3.2.1. Subqueries

    SELECT COUNT(*)
    FROM (
      SELECT DISTINCT *
      FROM health.user_logs
    ) AS subquery
    ;

## 3.2.2. Common Table Expression

> CTE stands for Common Table Expression - and when we compare it to
> something simple like Excel, we can think of CTEs as transformations
> applied to raw data inside an existing Excel sheet.

    WITH deduped_logs AS (
      SELECT DISTINCT *
      FROM health.user_logs
    )
    SELECT COUNT(*)
    FROM deduped_logs;


## 3.2.3. Temporary Tables  (becarefull)

> First we run a DROP TABLE IF EXISTS statement to clear out any
> previously created tables

    DROP TABLE IF EXISTS deduplicated_user_logs;

> Next let’s create a new temporary table using the results of the query
> below

    CREATE TEMP TABLE deduplicated_user_logs AS
    SELECT DISTINCT *
    FROM health.user_logs;

> Let’s query this newly created temporary table to make sure we can
> access it

    SELECT *
    FROM deduplicated_user_logs
    LIMIT 10;

> Finally, let’s run that same COUNT on this deduplicated temp table to
> confirm it did the right thing!

    SELECT COUNT(*)
    FROM deduplicated_user_logs;

> Temporary tables are automagically deleted once a session is shut down
> i.e hitting control + c on the terminal instance running the
> docker-compose command.


## 4.1. Group By Counts On All Columns

    SELECT
      id,
      log_date,
      measure,
      measure_value,
      systolic,
      diastolic,
      COUNT(*) AS frequency
    FROM health.user_logs
    GROUP BY
      id,
      log_date,
      measure,
      measure_value,
      systolic,
      diastolic
    ORDER BY frequency DESC

## 4.2. Having Clause For Unique Duplicates

> Note that we cannot use the same * within the GROUP BY grouping
> elements as it will throw a syntax error. To drill our previous
> knowledge - let’s go ahead and create a new temporary table called
> duplicate_record_counts in our following query. Don't forget to clean
> up any existing temp tables!

    DROP TABLE IF EXISTS unique_duplicate_records;
    CREATE TEMPORARY TABLE unique_duplicate_records AS
    SELECT *
    FROM health.user_logs
    GROUP BY
      id,
      log_date,
      measure,
      measure_value,
      systolic,
      diastolic
    HAVING COUNT(*) > 1;

> Finally let's inspect the top 10 rows of our temp table

    SELECT *
    FROM unique_duplicate_records
    LIMIT 10;


## 4.3. Retaining Duplicate Counts

    WITH groupby_counts AS (
      SELECT
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic,
        COUNT(*) AS frequency
      FROM health.user_logs
      GROUP BY
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic
    )
    SELECT *
    FROM groupby_counts
    WHERE frequency > 1
    ORDER BY frequency DESC
    LIMIT 10;


> Which id value has the most number of duplicate records in the
> health.user_logs table?

    WITH groupby_counts AS (
      SELECT
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic,
        COUNT(*) AS frequency
      FROM health.user_logs
      GROUP BY
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic
    )
    SELECT
      id,
      SUM(frequency) AS total_duplicate_rows
    FROM groupby_counts
    WHERE frequency > 1
    GROUP BY id
    ORDER BY total_duplicate_rows DESC
    LIMIT 10;

> Which log_date value had the most duplicate records after removing the
> max duplicate id value from question 1?

    WITH groupby_counts AS (
      SELECT
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic,
        COUNT(*) AS frequency
      FROM health.user_logs
      WHERE id != '054250c692e07a9fa9e62e345231df4b54ff435d'
      GROUP BY
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic
    )
    SELECT
      log_date,
      SUM(frequency) AS total_duplicate_rows
    FROM groupby_counts
    WHERE frequency > 1
    GROUP BY log_date
    ORDER BY total_duplicate_rows DESC
    LIMIT 10;

> Which measure_value had the most occurences in the health.user_logs
> value when measure = 'weight'?

    SELECT
      measure_value,
      COUNT(*) AS frequency
    FROM health.user_logs
    WHERE measure = 'weight'
    GROUP BY measure_value
    ORDER BY frequency DESC
    LIMIT 10;


## 6.4. Single & Total Duplicated Rows

> How many single duplicated rows exist when measure = 'blood_pressure'
> in the health.user_logs? How about the total number of duplicate
> records in the same table?

    WITH groupby_counts AS (
      SELECT
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic,
        COUNT(*) AS frequency
      FROM health.user_logs
      WHERE measure = 'blood_pressure'
      GROUP BY
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic
    )
    SELECT
      COUNT(*) as single_duplicate_rows,
      SUM(frequency) as total_duplicate_records
    FROM groupby_counts
    WHERE frequency > 1;


## 6.5. Percentage of Table Records

> What percentage of records measure_value = 0 when measure =
> 'blood_pressure' in the health.user_logs table? How many records are
> there also for this same condition?

    WITH all_measure_values AS (
      SELECT
        measure_value,
        COUNT(*) AS total_records,
        SUM(COUNT(*)) OVER () AS overall_total
      FROM health.user_logs
      WHERE measure = 'blood_pressure'
      GROUP BY 1
    )
    SELECT
      measure_value,
      total_records,
      overall_total,
      ROUND(100 * total_records::NUMERIC / overall_total, 2) AS percentage
    FROM all_measure_values
    WHERE measure_value = 0;


## 6.6. Percentage of Duplicates

> What percentage of records are duplicates in the health.user_logs
> table?

    WITH groupby_counts AS (
      SELECT
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic,
        COUNT(*) AS frequency
      FROM health.user_logs
      GROUP BY
        id,
        log_date,
        measure,
        measure_value,
        systolic,
        diastolic
    )
    SELECT
      -- Need to subtract 1 from the frequency to count actual duplicates!
      -- Also don't forget about the integer floor division!
      ROUND(
        100 * SUM(CASE
            WHEN frequency > 1 THEN frequency - 1
            ELSE 0 END
        )::NUMERIC / SUM(frequency),
        2
      ) AS duplicate_percentage
    FROM groupby_counts;
    
    --or
    WITH deduped_logs AS (
      SELECT DISTINCT *
      FROM health.user_logs
    )
    SELECT
      ROUND(
        100 * (
          (SELECT COUNT(*) FROM health.user_logs) -
          (SELECT COUNT(*) FROM deduped_logs)
        )::NUMERIC /
        (SELECT COUNT(*) FROM health.user_logs),
        2
      ) AS duplicate_percentage;








