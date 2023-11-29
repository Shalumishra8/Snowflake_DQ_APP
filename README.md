# Data Quality App
Implementing Data Quality checks are essential part of designing Data Pipelines. Data Quality ensures dataset meets criteria for accuracy, completeness, validity, consistency, uniqueness, timeliness, and fitness for purpose, and it is critical to all data governance initiatives within an organization. Why? The manual efforts and time required for executing data quality can be very time taken and rigourous process. We know that data security is a priority for a variety of businesses. To reduce manual efforts we build a framework known as Data Quality App that conducts check on dataset without moving data out of the Snowflake environment. Once APP gets onboarded into Snowflake account, Data Quality Checks can be implemented by calling stored procedures with relevant parameters.

## Whats inside the Data Quality App
The Data Quality App provides you two schemas :
- DATA_GOV_APP - To replicate the dataset on which consumer wants to run DATA Quality check.
- DATA_GOV_METADATA_APP - It has all the tables and procedures which will store metadata information checks to implement on dataset. It has tables and procedures.

## Getting started as a Consumer: First Time Setup
This example includes a "walkthrough" that will guide you through granting privileges and binding references that the app needs to function. 

After installing the app in your account, grant permission to the snowflake database to collect metadata using the following commands:
```sql
GRANT IMPORTED PRIVILEGES ON DATABASE SNOWFLAKE TO APPLICATION DATA_QUALITY_APP;
```
After installing the app in your account, grant permissions on the app to roles in your account. Please refer the following script:
```sql

GRANT APPLICATION ROLE DATA_GOV_APP TO ROLE ACCOUNTADMIN;
```

Once you install the app in your account.To execute data quality checks on table, you have two options. 

- Execute data quality checks on the existing table
- create the the table in schema DATA_GOV_APP in app on which you want to run data quality check 

In order to execute first option you have to grant permissions to the app to the existing objects, refer following script:

```sql
GRANT USAGE ON DATABASE <DATABASE_NAME> TO APPLICATION DATA_QUALITY_APP;

GRANT USAGE ON SCHEMA <SCHEMA_NAME> TO APPLICATION DATA_QUALITY_APP;

GRANT SELECT ON TABLE <TABLE_NAME> TO APPLICATION DATA_QUALITY_APP;
```

For second option create your own table in DATA_GOV_APP schema and give SELECT privilege on the table to application.

```sql
CREATE OR REPLACE TABLE DATA_GOV_APP.EMPLOYEE_DEPT_TABLE (
	ID VARCHAR,
	NAME VARCHAR,
	DEPT_ID NUMBER(38,0),
	DEPT VARCHAR,
	JOIN_DATE VARCHAR
);

GRANT SELECT ON TABLE data_quality_app.data_gov_app.EMPLOYEE_DEPT_TABLE TO APPLICATION DATA_QUALITY_APP;
```

Then create a data quality checks on columns using procedure DATA_GOV_METADATA_APP.create_dq_rule by passing parameter(table_name ,column_name ,rule_id ,expression). Some of the data quality checks doesn't require additional information, e.g., if quality check is “Column values to be unique”, then it does not need any additional information. Some of them require additional information which you can pass in expression as a last parameter,e.g., if quality check is “column values match strftime date”, then pass the date format reference in expression.All Refer following script:

```sql
call DATA_GOV_METADATA_APP.create_dq_rule(<TABLE NAME>,<COLUMN NAME>,<RULE ID>,<EXPRESSION>);

-- e.g.
call DATA_GOV_METADATA_APP.create_dq_rule('EMPLOYEE_DEPT_TABLE','JOIN_DATE','column_values_match_strftime_date','%d-%m-%Y');
call DATA_GOV_METADATA_APP.create_dq_rule('EMPLOYEE_DEPT_TABLE','NAME','column_values_to_be_unique','NA');

```
Once all the rules are created, execute rundq_rules which will implement the data quality checks on data, it will scan the data and check for each rule added.

```sql
call DATA_GOV_METADATA_APP.rundq_rules(<DATABASE_NAME>,<SCHEMA NAME>,<TABLE NAME>);

-- e.g.
call DATA_GOV_METADATA_APP.rundq_rules('DEMO_DB','RAW_SCHEMA','EMPLOYEE_DEPT_TABLE');

```
You can see all the records that failed the data quality checks in dq_violations tables.

```sql
Select * from DATA_GOV_METADATA_APP.dq_violations;

```
Additionally you can collect the runtime process metadata to collect the load and insert history.

```sql
call DATA_GOV_METADATA_APP.collect_processmetadata(<DATABASE_NAME>,<SCHEMA NAME>,<TABLE NAME>);

--e.g.
call DATA_GOV_METADATA_APP.collect_processmetadata('DEMO_DB','RAW_SCHEMA','EMPLOYEE_DEPT_TABLE');

```

### Consumer: Cleaning up

The application itself can be uninstalled by calling:

```sql
drop application Data_Quality_App;
```
