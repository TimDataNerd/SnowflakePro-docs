# ARCHITECTURE

Database replication is available on all Snowflake editions.

Snowflake micropartitions are immutable, they are created in the order in which data arrives.

The data is kept in columnar format. Columns may overlap accross the micropartitions.

Compute and storage can be scaled independently.

Snowflake has a hybrid shared-disk and shared-nothing architecture. 

DATA_RETENTION_TIME_IN_DAYS can be applied on accounts, databases, schemas and individual tables.

Data retention time can be set up to 90 days starting from Enterprise edition.

Snowflake account types :
  * Standard
  * Enterprise
  * Business Critical
  * Virtual Private Snowflake (no sharing capabilities for the last one)

The 3 Snowflake Multi-Cluster Architecture layers :
	1. Cloud Service layer (security, authentification and metadata storage).
    * Cloud service layer is only charged when it is used for more than 10% of daily warehouses usage.
    * The service layer is responsible for authentification, metadata managment and query optimization.
	2. Query / Compute layer (warehouses).
	3. Storage Layer (databases and disks)

The usage metrics views in Snowflake db are available for 12 months.  

Each created database automatically contains an information schema.

A Snowflake account can only exist in one geographical region.

## METADATA OPERATIONS

_HyperLogLog_ functions allows to get a quick estimate the number of unique values in a column.

## WAREHOUSES

Snowflake warehouses have a T-shirt like sizing sytem with XS costing 1 credit per minute. 2 for S, 4 for M, 8 for L.

Serverless can use 2X-Large as the max warehouse.

If a warehouse gets suspended, only idle Resources are shut down.
The compute resources that are still in use will remain so until all the queries are finished.

## INFORMATION SCHEMA

_INFORMATION_SCHEMA_ contains database-level and account-level information.

_INFORMATION_SCHEMA_ doesn't contain information about dropped objects.

_AUTO_REFRESH_REGISTRATION_HISTORY_ table function shows histroy of data files loaded into the metadata of
specific objects as well as the credits billed for those operations.

## RESOURCE MONITORS

By default, only _ACCOUNTADMIN_ has the right to create monitors.

_ACCOUNTADMIN_ can grant _MODIFY_ and _MONITOR_ priviledges to another user.

Resource monitors can be applied on an account level, on one or more warehouses.

## ROLES AND PRIVILEDGES

In order to clone an object, the priviledges vary :
  - Tables:_SELECT_  
  - Pipes, streams, tasks:_OWNERSHIP_  
  - Others:_USAGE_  

When cloning an object, only the subsequent object priviledges are cloned,
not the object's istelf.

_SYSADMIN_ role can only create warehouses and databases.

_ACCOUNTADMIN_ is the **highest role** in the role hierarchy.

_ORGADMIN_ can only only create and delete accounts. It cannot view contents of an account and thus, it cannot work with the data.

Roles can be granted to other roles.

Priviledges can only be granted to roles.

_OPERATE_ priviledge allows to resume and suspend tasks.

## SHARES

Snowflake Partner Connect allows to transfer Snowflake data to a business partners Snowflake account.

A share doesn't have an expiration date.

By default, only _ACCOUNTADMIN_ has the priviledge to create shares.

The CREATE SHARE priviledge can be granted to other roles.

When sharing a table or a view, _SELECT_ permission hast to be given on to it and _USAGE_ is to be given on subsequent shema and database.

Objects that can be shared:
 * Tables
 * External tables
 * Secure views
 * Secure materialized views
 * Secure UDFs

Objects added into a shared database become immidiately available for the consumers.

The same account can simultaneously share and consume data.

_SYSTEM$IS_LISTING_PURCHASED_ allows to limit access to only paying customers.

## STORED PROCEDURES & SNOWFLAKE SCRIPTING

Stored Procedures can extend system's functionality with procedural code,
can call user information such as session variables and are generally used for administrative operations.

A stored procedure runs either with the caller's rights or with the owner's rights, **but not both**.

Objects created with a script are available outside of the script, but not the variables.

We can use if/else case.

We can use loops like FOR, REPEAT, WHILE and LOOP.

Structure:

```SQL

DECLARE <variables>
...
BEGIN [TRANSACTION]
...
EXCEPTION <exception handling>
...
END [TRANSACTION];

```

Example:

```SQL
CREATE PROCEDURE aclc_area()
  RETURNS float
  LANGUAGE SQL
  AS
$$ <in UI only>
DECLARE
length_a float;
area float;
BEGIN TRANSACTION
length_a := 4;
area := length_a * length_a;
RETURN area
END TRANSACTION;
$$ <in UI only>
```

# QUERY OPTIMIZATION

Multiple queries can be executed simultaneously on multiple worksheets in Snowsight,
even if the worksheets are inactive.

In order to use the cached data in Snowflake Cloud metadata storage,
the user should have all the necessary privileges and the query must exactly match the previous one. 

Query optimization improves the performance of **point lookup queries** (very selective lookups).

## SNOWPIPE

Whenever we set _CREATE OR REPLACE PIPE_, all the history of the pipe is reset to 0.

_PIPE_USAGE_HISTORY_ schema in _ACCOUNT_USAGE_ shows the use of snowpipes.

Snowpipe loads metadata along with files. A file with the same name cannot be loaded twice.

The _INSERT_ONLY_ tipe of pipes only works with external tables.

## CACHING

USE_CACHED_RESULT can be turned off/on at a session, user or account levels.

Types of caches:
  * Query result cache
  * Metadata cache
  * Virtual warehouse cache

## STAGES

_@~_ for user stage,  _@%<TABLE_NAME>_ for a table stage,  _@<STAGE_NAME>_ for internal named stages.

If the _PURGE_ option is set to TRUE, after a successful load into tables all the staged files are removed from the external stage.

_PUT_ is used to lad data into **internal** stage. _GET_ is used to download data from **internal** stage.

### DIRECTORY TABLES

You can use streams with directory tables by creating a stream on a stage.

Directories have to be manually enabled for stages.

To retrieve info from stages: _SELECT * FROM DIRECTORY (@stage_name)_.

## MATERIALIZED VIEWS

Materialized views are available from Enterprise edition.

Materialized views can be only applied to _SELECT_ queries on one single table (no joins or multiple tables).

Materialized views are applicable on external tables, as well as internal queries that are frequently used and are sufficiently complex.

Materialized views cost on both **storage** and **serverles**.

## CLUSTERING

To retrieve info about clustering on a column or columns : system$clustering_information.

system$clustering_depth to retrieve only info about clustering depth.

The recommended number of columns per clustering key: 3 or 4.

When choosing columns for clustering, we should prefer columns that participate in _WHEN_ and _JOIN_ queries.
This improves pruning.

The colums for clustering should have medium cardinality (not too low, not too high).

Clustering keys imporve queriy performance as well as table maintenance.

# STORAGE

## TABLES

Types:
  * Permanent (persistant)
  * Transient (can be used by serveral sessions)
  * Temporary (local to a session)

## SCHEMAS

_CREATE SCHEMA_
_CREATE TRANSIENT SCHEMA_
_CREATE SCHEMA WITH MANAGED ACCESS_
_ALTER SCHEMA ENABLE | DISABLE MANAGED ACCESS_

When cloning a schema, all clustering keys and priviledges are clones as well.

_ACCOUNT_USAGE_ schema has a latency between 45 min and 3 hours.

_ACCOUNT_USAGE_ schema contains info about dropped objects.

_COY_HISTORY_ and _PIPE_USAGE_HISTORY_ views in _ACCOUNT_USAGE_ contain data loading history of pipelines.

# SECURITY

All users can only see the results of their own queries.

Periodic rekeying is available from Enterprise edition.

Snowflake Access control schemes are RBAC (Role based access control) and DAC (Discretionary access control).

Dedicated metadata storage and customer-managed encryption are only available in VPS edition.

VPS edition users have no access to Snowflake Marketplace.

Dynamic data masking is only availabe on Enterprise or higher.

## COMPLIANCE & FEDERATED ACCESS PROVISION

Compliant with SCIM 2.0 (e.g. Azure).

## KEYS

If enabled, periodic rekeying happens every 12 months. 

External functions hold the security related data in the API Keystore.

Snowflake uses a hierarchical key model, rooted in a hardware key.

Tri-secret secure system is only available from business critical edition.

Tri-secret secure system : a composite encryption key made up of a user provider key and a Snowflake key.

There can be 1 or 2 public keys in a key pair.

## ENCRYPTION

During data transfer, the data is encrypted with TLS 1.2.

## MFA

MFA token caching has to be enabled first.

MFA token caching is only available for Python, JDBC and ODBC.

Strongly recommended for at least _ACCOUNTADMIN_ role.

Fully supported by Snowsight, SnowSQL, JDBC, ODBC and Python connectors.

## NETWORK POLICY

Network policies are available for all editions.

They set access filtering based on user's IP address.

In order to create and set a network policy, we need to have global CREATE NETORK POLICY priviledge (available for SECURITYADMIN and higher).

Can be set on an account and on a user (user's OWNERSHIP is also required).

_MINS_TO_BYPASS_NETWORK_POLICY_ can be modified **only by Snowflake support**.

# DATA TRANSFORMATION

The Formats, supported by Snowflake:
 * Semi-structured:
	- ORC
	- AVRO
	- JSON (Allowed to unload with GET)
	- PARQUET (Allowed to unload with GET)
	- XML
 * Structured:
	- CSV/TSV (Allowed to unload with GET)

The Default compression format for loading and unloading files on Snowflake : GZIP.

Recommended compressed file size in Snowflake : 100-250 Mb.

Uncompressed size is around 50-500 Mb.

## SNOWSQL

When using SnowSQL CLI I can use any role that is available to me.

## VARIANT PARSING

One variant can only be 16 MB uncompressed.

We use $<COLUMN_NUMBER>: or <COLUMN_NAME>: to access the first element and dots (.) to access all the susequent elements.

## SPILLING

To prevent data spilling, we should process data in smaller batches and use larger warehouses.

## UDFs

Secure UDFs hide their definition from unauthorized users and bypass query optimization.

UDF only runs as the **function owner**.

## COPY INTO

_COPY INTO_ only supports basic data transformations:
  * Column reordering
  * Column omission
  * Casts
  * Truncating text strings that exceed the target column length.

The VALIDATE function allows a user to view all the errors occurred during the last _COPY INTO_ execution.

When _COPY INTO_ <TABLE> value _ON_ERROR_ is set to _CONTINUE_, the file will be uploaded even if errors are found.

The default value of _ON_ERROR_ is _ABORT_STATEMENT_.

We can use _FILES_ property to set one or multiple specific files to load.

The _VALIDATION_MODE_ doesn't allow to transform data during loading.
