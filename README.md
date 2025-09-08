# Tablespace-And-Schema-and-MVCC

# 1.TableSpace:- 
```
 - tablespace in PostgreSQL is a storage location where database objects like tables, indexes, and other data files are stored.
  - an additional data area outside the base directory. 
    1. Tablespace: A storage location for database objects.
    2. Symbolic Link: Used by PostgreSQL to link tablespace directories to the data directory.
    3. pg_tblspc: Directory containing symbolic links to tablespace locations.
    4. pg_tablespace: System catalog storing metadata about tablespaces.
    
- Postgresql Provides Default Two Tablespaces 1. Pg_default, pg_global.

##### Pg_Default :-
	- This default tablespace is used for user-defined objects, tables, and indexes, & most database-related files are stored.

##### Pg_global :-  
 - Will store system_catlog related informaton, like (pg_class, pg_attribute,pg_type).
 - dbOid, dbname,relname, realnamespace, 
 - When a tablespace is created, PostgreSQL internally creates a symbolic link in the pg_tblspc directory that points to the actual location on the filesystem.
 - pg_tablespace System Catalog
 - pg_tablespace is a system catalog (or system table) in PostgreSQL that stores metadata about tablespaces. It includes information like tablespace name, owner, and location.

  Expand Storage Easily: If a particular disk is running out of space, you can create a new tablespace on a larger or less used disk and move objects there without interrupting database operations.
  Selective Backup: You can back up only specific tablespaces that contain critical data rather than backing up the entire database, making the process faster and more efficient.
  Recovery Strategies: During recovery, you can restore critical tablespaces first, allowing partial recovery if full recovery would take too long.
  Use Different Disk Types: Different types of data can be stored on appropriate storage media (e.g., using SSDs for frequently accessed data and HDDs for archival data).
  Free Up Space Easily: By moving data between tablespaces, you can free up space on specific storage devices without complex data migrations.

Examples:- 
  Create tablespace:-
  [ CREATE TABLESPACE my_space LOCATION '/path/to/storage/location'; ]
  Change owner:
	    [ALTER TABLESPACE my_space OWNER TO new_owner;]
	
  Creating a table in a specific tablespace:

	[CREATE TABLE my_table (id INT) TABLESPACE my_space;]

      
	Moving an existing table to a different tablespace:
	[ALTER TABLE my_table SET TABLESPACE my_space;]


  pg_tablespace: Contains information about tablespaces.
    [SELECT * FROM pg_tablespace;]


- Key columns in pg_tablespace:
    • spcname: Name of the tablespace.
    • spcowner: OID of the owner (role).
    • spclocation: The directory location of the tablespace (older versions).
    • spcacl: Access control list (permissions).
```
---
---

# 2.Schema:-
```
- In PostgreSQL, a schema is a namespace that contains database objects like tables, views, functions, indexes, sequences, etc. It helps organize these objects within a database, allowing for better management and control over their usage. Schemas allow different users or applications to use the same names for different tables, functions, or other objects without conflict.
Key Features of Schemas in PostgreSQL:
    1. Logical Grouping: Schemas allow you to group related tables and other objects logically.
    2. Namespace Isolation: Schemas isolate objects, so multiple schemas can contain tables with the same name.
    3. Access Control: You can control access to schemas by granting or revoking privileges on them.
    4. Public Schema: By default, every new PostgreSQL database contains a public schema that every user has access to. All objects created without specifying a schema are placed in the public schema unless otherwise configured.
- In PostgreSQL, when you create a new database, certain default schemas are automatically created to help manage the organization and accessibility of database objects. These include:
1. public Schema
    • Description: The public schema is the default schema that every PostgreSQL database comes with. All database users can access the public schema unless specific permissions are modified.
    • Default Behavior: Any object created without explicitly specifying a schema will be placed in the public schema.
    • Permissions: By default, all users can create objects in the public schema, but this can be changed by altering privileges.

Example:
CREATE TABLE my_table (id SERIAL PRIMARY KEY);
-- This table will be created in the public schema by default:
-- public.my_table

3. information_schema
    • Description: The information_schema schema contains read-only views that provide metadata about the database, such as tables, columns, data types, and constraints.
    • Usage: This schema is used to query system catalog information in a standardized way, making it useful for inspecting the structure of the database.

Example:
[SELECT * FROM information_schema.tables;]

5. pg_catalog
    • Description: The pg_catalog schema is another system schema that contains PostgreSQL's system tables and functions. It holds metadata related to database internals such as tables, columns, indexes, and functions.
    • Usage: Internally used by PostgreSQL, but advanced users can also query pg_catalog for more detailed system-level information than information_schema provides.

Example:
[SELECT * FROM pg_catalog.pg_tables;]

7. pg_toast
    • Description: The pg_toast schema is used internally by PostgreSQL to store large objects that exceed a certain size limit. When a table has large text or binary data (like large TEXT or BYTEA fields), PostgreSQL may store these in pg_toast to manage space efficiently.
    • Usage: It's generally not accessed by users directly but is important for how PostgreSQL manages large data internally.
   
   If the column sizeis 2KB postgresqlautomaticallymoves it to toast table.
   
   
9. pg_temp_nnnn (Temporary Schemas)
    • Description: When you create temporary tables, PostgreSQL automatically creates temporary schemas (like pg_temp_1234). Each session has its own temporary schema, where all temporary tables reside.
    • Usage: These schemas are session-specific and are automatically dropped when the session ends.
   
11. pg_toast_temp_nnnn
    • Description: Similar to pg_toast, the pg_toast_temp_nnnn schemas are used internally for managing large objects for temporary tables. Like pg_temp_nnnn, these schemas are session-specific.
Managing Default Schemas
    • Search Path: PostgreSQL uses a search path to determine which schemas to search when you reference an object without specifying the schema. By default, the search path includes public, pg_catalog, and other system schemas.
To check the current search path:
SHOW search_path;
Example output:
search_path 
--------------
"$user", public

- In this case, $user means PostgreSQL will first look for a schema that matches the current user’s name. If no such schema exists, it defaults to the public schema.
- Changing the Search Path: You can adjust the search path for your session or permanently for a role or database.

  SET search_path TO schema1, schema2;

- In summary, PostgreSQL provides a set of default schemas for system management (pg_catalog, information_schema, pg_toast) and user access (public). You can manage how these schemas interact with your objects using the search path and schema privileges.
- Syntax for Schema Management:-
--------------------------------------------------
1. Creating a Schema:
CREATE SCHEMA schema_name;
Example:
CREATE SCHEMA sales;

----------------------------------------------------------
2. Creating a Schema Owned by a User:
CREATE SCHEMA schema_name AUTHORIZATION user_name;
Example:
CREATE SCHEMA sales AUTHORIZATION john;
---------------------------------------------------------
3. Listing Schemas:

SELECT schema_name 
FROM information_schema.schemata;

------------------------------------------------------
4. Creating an Object in a Specific Schema:

CREATE TABLE schema_name.table_name (
   column_name data_type
);
Example:
CREATE TABLE sales.customers (
   customer_id SERIAL PRIMARY KEY,
   name VARCHAR(100)
);

------------------------------------------------------
5. Setting a Search Path (Schema Priority):

You can define the search path so that PostgreSQL knows where to look for objects.
SET search_path TO schema_name;
Example:
SET search_path TO sales;
This will make the sales schema the default for subsequent object references.

-------------------------------------------------------
6. Dropping a Schema:

DROP SCHEMA schema_name CASCADE;
    • CASCADE: Automatically drops all objects within the schema.
    • RESTRICT: Prevents the schema from being dropped if it contains any objects (this is the default).
Example:
DROP SCHEMA sales CASCADE;

-------------------------------------------------------
7. Schema Privileges:

You can manage schema permissions using GRANT and REVOKE commands.
    • Granting permissions on a schema:
GRANT USAGE ON SCHEMA schema_name TO user_name;
    • Revoking permissions on a schema:
REVOKE USAGE ON SCHEMA schema_name FROM user_name;
```
---
---

# MVCC (Multi-Version Concurency Control)
```
Multi-version Concurency control allows multiple transactions to take place simultaneously while maintaining data consistency and isolation.
Multiple transaction can happen with database at the same time without blocking each other. It maintains multiple versions of rows.
- PostgreSQL support three columns internally to track rows versions (Xmin, Xmax, Xid).
- When transaction starts, PostgreSQL assigns a Transaction ID (XID) to that transaction.
- When transaction updated or Deleted postgresql assigns a transaction Id to (Xmax),
- When transaction inserted postgresql assigns a transaction id to (Xmin).

1.Starts transaction -> Assigns XID
2.Read Data -> sees only committed rows before its XID.
3.Modify Data -> Creates a new version (tuple) of the row with updated data & old version is marked as dead both versions arepresent until the vacuum process cleans up the dead row.
4.Commit Transaction -> New row version becomes visible.
5.RollBack Transaction -> New row version is discarded.
6.Delete -> The row is only marked as dead, no new version is created.


#### When will transaction Block & which Transaction will Not Block?

# Transaction will Block.
1.UPDATE : Row-Level Lock
2.DELETE : Row-Level Lock
--------------------------------
# Transaction Will not Block.
1.SELECT : Reads older version
2.INSERT on different Rows
3.UPDATE on Different Rows
4.DELETE on Dfferent Rows


#### If we want 100% non-blocking reads use below isolation levels, We can manage It by using Isolation Levels:
- Postgresql Support | Read Committed (default Level) | Repeatable Read | Serializable.


- SERIALIZABLE
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

- SERIALIZABLE gurantees that transaction will execute in a way as if they were executed one by one sequentlly, even id they actually run concurently.
- Postgresql creates snapshots of the database at the moment the transaction starts.
- transaction reads only the snapshot data.
- It will not see any changes made by other concurrent transaction until it commits.
- If two transaction try to modify the same row or related data, one transaction will autimatically abort with an error: "could not serialize access due to concurrent update".
---
- Read Commited (Default level)

SET TRANSACTION ISOLATION LEVEL Read Committed;

- Each query sees only data that was commited before the query started.
- Uncommitted changes from other transactions are not visible.
- Non-repeatable reads are possible.

- Repeatable Read

SET TRANSACTION ISOLATION LEVEL Repeatable Reads;

- All queries in the same transaction see a snapshot of the database taken when the transaction started.
- No other transaction's changes are visible during the transaction.
- phantom reads are possible
```
---
---

# Pg-catalog

```
PostgreSQL maintains several system catalogs that store metadata about databases, tables, indexes, functions, and more. Below is a list of key system catalogs and their descriptions.

## Catalog List

| Catalog Name | Description |
|-------------|-------------|
| **pg_aggregate** | Stores aggregate functions. |
| **pg_am** | Contains access method information. |
| **pg_amop** | Stores operator families for index access methods. |
| **pg_amproc** | Contains support functions for index access methods. |
| **pg_attrdef** | Stores default values for table columns. |
| **pg_attribute** | Contains metadata about table columns. |
| **pg_authid** | Stores user authentication details. |
| **pg_auth_members** | Manages role memberships. |
| **pg_cast** | Stores type conversion rules. |
| **pg_class** | Contains metadata about tables, indexes, sequences, and views. |
| **pg_collation** | Stores collation settings for text sorting. |
| **pg_constraint** | Contains table constraints (e.g., primary keys, foreign keys). |
| **pg_conversion** | Stores encoding conversion rules. |
| **pg_database** | Contains information about databases. |
| **pg_db_role_setting** | Stores role-specific database settings. |
| **pg_default_acl** | Manages default access control lists. |
| **pg_depend** | Tracks dependencies between database objects. |
| **pg_description** | Stores descriptions of database objects. |
| **pg_enum** | Contains enumerated type values. |
| **pg_event_trigger** | Stores event triggers. |
| **pg_extension** | Manages installed extensions. |
| **pg_foreign_data_wrapper** | Stores foreign data wrapper metadata. |
| **pg_foreign_server** | Contains foreign server details. |
| **pg_foreign_table** | Stores metadata about foreign tables. |
| **pg_index** | Contains index metadata. |
| **pg_inherits** | Tracks table inheritance relationships. |
| **pg_init_privs** | Stores initial privileges for database objects. |
| **pg_language** | Contains metadata about procedural languages. |
| **pg_largeobject** | Stores large object data. |
| **pg_largeobject_metadata** | Contains metadata for large objects. |
| **pg_namespace** | Stores schema information. |
| **pg_opclass** | Contains operator class metadata for indexes. |
| **pg_operator** | Stores operator definitions. |
| **pg_opfamily** | Manages operator families for indexes. |
| **pg_parameter_acl** | Stores parameter-specific access control lists. |
| **pg_partitioned_table** | Contains partitioned table metadata. |
| **pg_policy** | Stores row-level security policies. |
| **pg_proc** | Contains stored procedures and functions. |
| **pg_publication** | Manages logical replication publications. |
| **pg_publication_namespace** | Stores schema-level publications. |
| **pg_publication_rel** | Contains table-level publication details. |
| **pg_range** | Stores range type metadata. |
| **pg_replication_origin** | Manages replication origins. |
| **pg_rewrite** | Stores query rewrite rules. |
| **pg_seclabel** | Contains security labels for database objects. |
| **pg_sequence** | Stores sequence metadata. |
| **pg_shdepend** | Tracks shared object dependencies. |
| **pg_shdescription** | Stores descriptions for shared objects. |
| **pg_shseclabel** | Contains security labels for shared objects. |
| **pg_statistic** | Stores table statistics for query optimization. |
| **pg_statistic_ext** | Contains extended statistics for query optimization. |
| **pg_statistic_ext_data** | Stores extended statistics data. |
| **pg_subscription** | Manages logical replication subscriptions. |
| **pg_subscription_rel** | Contains table-level subscription details. |
| **pg_tablespace** | Stores tablespace metadata. |
| **pg_transform** | Contains type transformation rules. |
| **pg_trigger** | Stores trigger metadata. |
| **pg_ts_config** | Contains text search configuration metadata. |
| **pg_ts_config_map** | Stores mappings for text search configurations. |
| **pg_ts_dict** | Contains text search dictionary metadata. |
| **pg_ts_parser** | Stores text search parser metadata. |
| **pg_ts_template** | Contains text search template metadata. |
| **pg_type** | Stores data type metadata. |
| **pg_user_mapping** | Manages user mappings for foreign data wrappers. |

## References
For more details, visit the [PostgreSQL System Catalogs Documentation](https://www.postgresql.org/docs/current/catalogs.html).



#### pg_aggregate (Stores information about aggregate functions)

| Column Name        | Description |
|--------------------|-------------|
| aggfnoid          | OID of the aggregate function (references pg_proc.oid) |
| aggkind           | Type of aggregate: n (normal), o (ordered-set), h (hypothetical-set) |
| aggnumdirectargs  | Number of direct (non-aggregated) arguments |
| aggtransfn        | Transition function (references pg_proc.oid) |
| aggfinalfn        | Final function (references pg_proc.oid) |
| aggcombinefn      | Combine function (references pg_proc.oid) |
| aggserialfn       | Serialization function (references pg_proc.oid) |
| aggdeserialfn     | Deserialization function (references pg_proc.oid) |
| aggtranstype      | Data type of the aggregate function's internal transition state |
| aggtransspace     | Approximate average size (in bytes) of the transition state data |

#### pg_am (Stores information about relation access methods)

| Column Name | Description |
|------------|-------------|
| oid        | Row identifier |
| amname     | Name of the access method |
| amhandler  | OID of a handler function responsible for supplying information about the access method |
| amtype     | Type of access method: t (table), i (index) |

#### pg_amop (Stores operators associated with access method operator families)

| Column Name    | Description |
|---------------|-------------|
| amopfamily    | The operator family this entry is for (references pg_opfamily.oid) |
| amoplefttype  | Left-hand input data type of operator (references pg_type.oid) |
| amoprighttype | Right-hand input data type of operator (references pg_type.oid) |
| amopstrategy  | Operator strategy number |
| amoppurpose   | Operator purpose: s (search), o (ordering) |
| amopopr       | OID of the operator (references pg_operator.oid) |
| amopmethod    | Index access method operator family is for (references pg_am.oid) |

#### pg_attrdef (Stores default values for table columns)

| Column Name | Description |
|------------|-------------|
| oid        | Unique identifier for the default value entry. |
| adrelid    | OID of the table the column belongs to (references pg_class.oid). |
| adnum      | Column number (matches the position in the table). |
| adbin      | Internal representation of the default expression for the column. |

#### pg_attribute (Stores metadata about table columns)

| Column Name  | Description |
|-------------|-------------|
| attrelid    | OID of the table this column belongs to (references pg_class.oid). |
| attname     | Name of the column. |
| atttypid    | OID of the data type for this column (references pg_type.oid). |
| attlen      | Length of the data type (for fixed-length types). |
| attnum      | Column number (order within the table). |
| attisdropped | Whether the column is dropped (TRUE or FALSE). |
| attnotnull  | Whether the column has a NOT NULL constraint. |

#### pg_authid (Stores user authentication details)

| Column Name   | Description |
|--------------|-------------|
| oid          | Unique identifier for the role. |
| rolname      | Name of the role (user or group). |
| rolsuper     | Indicates if the role has superuser privileges (TRUE or FALSE). |
| rolinherit   | Whether the role inherits privileges (TRUE or FALSE). |
| rolcreaterole | Whether the role can create new roles. |
| rolcreatedb  | Whether the role can create new databases. |
| rolcanlogin  | Whether the role can log in. |

#### pg_auth_members (Stores role membership relations)

| Column Name      | Type  | Description |
|-----------------|------|-------------|
| roleid         | oid  | ID of a role that has a member |
| member         | oid  | ID of a role that is a member of roleid |
| grantor        | oid  | ID of the role that granted this membership |
| admin_option   | bool | True if the member can grant membership in roleid to others |
| inherit_option | bool | True if the member automatically inherits the privileges of the granted role |
| set_option     | bool | True if the member can SET ROLE to the granted role |

#### pg_cast (Stores data type conversion paths)

| Column Name  | Type  | Description |
|-------------|------|-------------|
| castsource  | oid  | OID of the source data type |
| casttarget  | oid  | OID of the target data type |
| castfunc    | oid  | OID of the function used for the cast (zero if not required) |
| castcontext | char | Contexts where the cast can be invoked (e = explicit, a = assignment, i = implicit) |
| castmethod  | char | How the cast is performed (f = function, i = input/output, b = binary coercible) |

#### pg_class (Describes tables, indexes, sequences, views, and other relations)

| Column Name    | Type   | Description |
|--------------|-------|-------------|
| relname      | name  | Name of the relation |
| relnamespace | oid   | Namespace containing this relation |
| reltype      | oid   | Data type corresponding to this table's row type |
| relowner     | oid   | Owner of the relation |
| relam        | oid   | Access method used for this table or index |
| relfilenode  | oid   | On-disk file name of this relation |
| reltablespace| oid   | Tablespace where this relation is stored |
| relpages     | int4  | Estimated size of the table in pages |
| reltuples    | float4| Estimated number of live rows in the table |
| reltoastrelid| oid   | OID of the TOAST table associated with this table (zero if none) |

#### pg_collation (Describes available collations)

| Column Name   | Description |
|--------------|-------------|
| collname     | Collation name (unique per namespace and encoding) |
| collprovider | Provider of the collation (d = database default, b = builtin, c = libc, i = ICU) |
| collencoding | Encoding in which the collation is applicable |
| collcollate  | LC_COLLATE for this collation object |

#### pg_constraint (Stores constraints on tables and domains)

| Column Name    | Description |
|--------------|-------------|
| conname      | Constraint name |
| contype      | Constraint type (c = check, f = foreign key, p = primary key, u = unique, x = exclusion) |
| condeferrable | Whether the constraint is deferrable |
| convalidated  | Whether the constraint has been validated |

#### pg_conversion (Describes encoding conversion functions)

| Column Name    | Description |
|--------------|-------------|
| conname      | Conversion name |
| conforencoding | Source encoding ID |
| contoencoding  | Destination encoding ID |
| conproc       | Conversion function |

#### pg_database (Stores information about available databases)

| Column Name  | Description |
|-------------|-------------|
| datname     | Database name |
| datdba      | Owner of the database |
| encoding    | Character encoding for the database |
| datcollate  | LC_COLLATE for the database |

#### pg_db_role_setting (Records default values for runtime configuration variables)

| Column Name  | Description |
|-------------|-------------|
| setdatabase | OID of the database the setting applies to |
| setrole     | OID of the role the setting applies to |
| setconfig   | Defaults for runtime configuration variables |

#### pg_default_acl (Stores initial privileges for newly created objects)

| Column Name      | Type     | Description |
|-----------------|---------|-------------|
| oid            | oid     | Row identifier |
| defaclrole     | oid     | Role associated with this entry (references pg_authid.oid) |
| defaclnamespace | oid     | Namespace associated with this entry (references pg_namespace.oid) |
| defaclobjtype  | char    | Type of object (r = relation, S = sequence, f = function, T = type, n = schema) |
| defaclacl      | aclitem[] | Access privileges assigned on creation |

#### pg_depend (Tracks dependencies between database objects)

| Column Name   | Type  | Description |
|--------------|------|-------------|
| classid      | oid  | OID of the system catalog containing the dependent object (references pg_class.oid) |
| objid        | oid  | OID of the specific dependent object |
| objsubid     | int4 | Column number if the object is a table column, otherwise zero |
| refclassid   | oid  | OID of the system catalog containing the referenced object (references pg_class.oid) |
| refobjid     | oid  | OID of the specific referenced object |
| refobjsubid  | int4 | Column number if the referenced object is a table column, otherwise zero |
| deptype      | char | Dependency type (n = normal, a = auto, i = internal) |

#### pg_description (Stores optional descriptions for database objects)

| Column Name  | Type  | Description |
|-------------|------|-------------|
| objoid      | oid  | OID of the object being described |
| classoid    | oid  | OID of the system catalog containing the object (references pg_class.oid) |
| objsubid    | int4 | Column number if the object is a table column, otherwise zero |
| description | text | Text description of the object |

#### pg_enum (Stores values and labels for enum types)

| Column Name   | Type   | Description |
|--------------|-------|-------------|
| oid          | oid   | Row identifier |
| enumtypid    | oid   | OID of the enum type (references pg_type.oid) |
| enumsortorder | float4 | Sort position of this enum value within its type |
| enumlabel    | name  | Textual label for this enum value |

#### pg_event_trigger (Stores event triggers)

| Column Name  | Type  | Description |
|-------------|------|-------------|
| oid         | oid  | Row identifier |
| evtname     | name | Unique name of the event trigger |
| evtevent    | name | Event type for which the trigger fires |
| evtowner    | oid  | Owner of the event trigger (references pg_authid.oid) |
| evtfoid     | oid  | Function to be called (references pg_proc.oid) |
| evtenabled  | char | Trigger firing mode (O = origin, D = disabled, R = replica, A = always) |
| evttags     | text[] | Command tags for which this trigger fires |

#### pg_extension (Stores metadata about installed extensions)

| Column Name   | Type  | Description |
|--------------|------|-------------|
| oid          | oid  | Row identifier |
| extname      | name | Name of the extension |
| extowner     | oid  | Owner of the extension (references pg_authid.oid) |
| extnamespace | oid  | Schema containing the extension's objects (references pg_namespace.oid) |
| extrelocatable | bool | Whether the extension can be relocated to another schema |
| extversion   | text | Version name of the extension |
| extconfig    | oid[] | Array of OIDs for the extension's configuration tables |
| extcondition | text[] | Array of WHERE-clause filter conditions for the extension's configuration tables |

#### pg_foreign_data_wrapper (Stores foreign-data wrapper definitions)

| Column Name  | Type  | Description |
|-------------|------|-------------|
| oid         | oid  | Row identifier |
| fdwname     | name | Name of the foreign-data wrapper |
| fdwowner    | oid  | Owner of the foreign-data wrapper (references pg_authid.oid) |
| fdwhandler  | oid  | Handler function responsible for execution routines (references pg_proc.oid) |
| fdwvalidator | oid | Validator function for checking options (references pg_proc.oid) |
| fdwacl      | aclitem[] | Access privileges |
| fdwoptions  | text[] | Foreign-data wrapper specific options |

#### pg_foreign_server (Stores foreign server definitions)

| Column Name | Type  | Description |
|------------|------|-------------|
| oid        | oid  | Row identifier |
| srvname    | name | Name of the foreign server |
| srvowner   | oid  | Owner of the foreign server (references pg_authid.oid) |
| srvfdw     | oid  | OID of the foreign-data wrapper (references pg_foreign_data_wrapper.oid) |
| srvtype    | text | Type of the server (optional) |
| srvversion | text | Version of the server (optional) |
| srvacl     | aclitem[] | Access privileges |
| srvoptions | text[] | Foreign server specific options |

#### pg_foreign_table (Stores metadata about foreign tables)

| Column Name | Type  | Description |
|------------|------|-------------|
| ftrelid    | oid  | OID of the foreign table (references pg_class.oid) |
| ftserver   | oid  | OID of the foreign server (references pg_foreign_server.oid) |
| ftoptions  | text[] | Foreign table options |

#### pg_index (Stores metadata about indexes)

| Column Name             | Type   | Description |
|------------------------|-------|-------------|
| indexrelid            | oid   | OID of the index (references pg_class.oid) |
| indrelid              | oid   | OID of the table the index is for (references pg_class.oid) |
| indnatts              | int2  | Total number of columns in the index |
| indnkeyatts           | int2  | Number of key columns in the index |
| indisunique           | bool  | Whether the index is unique |
| indnullsnotdistinct   | bool  | Whether null values are considered distinct |
| indisprimary          | bool  | Whether the index represents the primary key |
| indisexclusion        | bool  | Whether the index supports an exclusion constraint |
| indimmediate          | bool  | Whether uniqueness is enforced immediately |
| indisclustered        | bool  | Whether the table was last clustered on this index |
| indisvalid            | bool  | Whether the index is currently valid for queries |

#### pg_inherits (Stores table inheritance relationships)

| Column Name        | Type  | Description |
|-------------------|------|-------------|
| inhrelid        | oid  | OID of the child table or index (references pg_class.oid) |
| inhparent       | oid  | OID of the parent table or index (references pg_class.oid) |
| inhseqno        | int4 | Order of inherited columns (starts at 1) |
| inhdetachpending | bool | TRUE if partition is in the process of being detached |

#### pg_language (Stores procedural languages)

| Column Name  | Type  | Description |
|-------------|------|-------------|
| oid         | oid  | Row identifier |
| lanname     | name | Name of the language |
| lanowner    | oid  | Owner of the language (references pg_authid.oid) |
| lanispl     | bool | TRUE for user-defined languages, FALSE for internal languages |
| lanpltrusted | bool | TRUE if the language is trusted |
| lanplcallfoid | oid  | Function responsible for executing functions in this language |
| laninline   | oid  | Function for executing inline anonymous code blocks |
| lanvalidator | oid  | Function for validating syntax of new functions |
| lanacl      | aclitem[] | Access privileges |

#### pg_largeobject (Stores large object data)

| Column Name | Type  | Description |
|------------|------|-------------|
| loid       | oid  | Identifier of the large object |
| pageno     | int4 | Page number within the large object |
| data       | bytea | Actual data stored in the large object |

#### pg_namespace (Stores schema information)

| Column Name | Type  | Description |
|------------|------|-------------|
| oid        | oid  | Row identifier |
| nspname    | name | Name of the schema |
| nspowner   | oid  | Owner of the schema (references pg_authid.oid) |
| nspacl     | aclitem[] | Access privileges |

#### pg_proc (Stores function and procedure metadata)

| Column Name    | Type   | Description |
|--------------|-------|-------------|
| oid          | oid   | Row identifier |
| proname      | name  | Name of the function |
| pronamespace | oid   | Namespace containing the function |
| proowner     | oid   | Owner of the function |
| prolang      | oid   | Language used for the function |
| procost      | float4 | Estimated execution cost |
| prorows      | float4 | Estimated number of result rows |
| provariadic  | oid   | Data type of variadic parameter elements |
| prokind      | char  | Function type (f = normal, p = procedure, a = aggregate, w = window) |

#### pg_publication (Stores logical replication publications)

| Column Name  | Type  | Description |
|-------------|------|-------------|
| oid         | oid  | Row identifier |
| pubname     | name | Name of the publication |
| pubowner    | oid  | Owner of the publication |
| puballtables | bool | TRUE if all tables are included |
| pubinsert   | bool | TRUE if INSERT operations are replicated |
| pubupdate   | bool | TRUE if UPDATE operations are replicated |
| pubdelete   | bool | TRUE if DELETE operations are replicated |
| pubtruncate | bool | TRUE if TRUNCATE operations are replicated |

#### pg_range (Stores metadata for range types)

| Column Name   | Type    | Description |
|--------------|--------|-------------|
| rngtypid     | oid    | OID of the range type |
| rngsubtype   | oid    | OID of the element type (subtype) |
| rngmultitypid | oid   | OID of the multirange type |
| rngcollation | oid    | OID of the collation used for comparisons |
| rngsubopc    | oid    | OID of the operator class used for comparisons |
| rngcanonical | regproc | Function to convert range value into canonical form |
| rngsubdiff   | regproc | Function to return the difference between two element values |

#### pg_replication_origin (Stores replication origins)

| Column Name | Type  | Description |
|------------|------|-------------|
| roident    | oid  | Unique identifier for the replication origin |
| roname     | text | User-defined name of the replication origin |

#### pg_rewrite (Stores rewrite rules for tables and views)

| Column Name  | Type  | Description |
|-------------|------|-------------|
| oid         | oid  | Row identifier |
| rulename    | name | Rule name |
| ev_class    | oid  | Table this rule applies to |
| ev_type     | char | Event type (1 = SELECT, 2 = UPDATE, 3 = INSERT, 4 = DELETE) |
| ev_enabled  | char | Rule firing mode (O = origin, D = disabled, R = replica, A = always) |
| is_instead  | bool | TRUE if the rule is an INSTEAD rule |

#### pg_sequence (Stores metadata about sequences)

| Column Name   | Type  | Description |
|--------------|------|-------------|
| seqrelid     | oid  | OID of the sequence |
| seqtypid     | oid  | Data type of the sequence |
| seqstart     | int8 | Start value of the sequence |
| seqincrement | int8 | Increment value of the sequence |
| seqmax       | int8 | Maximum value of the sequence |
| seqmin       | int8 | Minimum value of the sequence |
| seqcache     | int8 | Cache size of the sequence |
| seqcycle     | bool | Whether the sequence cycles |

#### pg_statistic (Stores statistics for query optimization)

| Column Name  | Type   | Description |
|-------------|-------|-------------|
| starelid    | oid   | Table or index the column belongs to |
| staattnum   | int2  | Column number |
| stanullfrac | float4 | Fraction of null values |
| stawidth    | int4  | Average stored width of non-null entries |
| stadistinct | float4 | Number of distinct non-null values |

#### pg_subscription (Stores logical replication subscriptions)

| Column Name  | Type  | Description |
|-------------|------|-------------|
| subname     | name | Name of the subscription |
| subowner    | oid  | Owner of the subscription |
| subenabled  | bool | Whether the subscription is enabled |
| subconninfo | text | Connection string to the upstream database |

#### pg_tablespace (Stores tablespace metadata)

| Column Name | Type  | Description |
|------------|------|-------------|
| spcname    | name | Name of the tablespace |
| spcowner   | oid  | Owner of the tablespace |
| spcacl     | aclitem[] | Access privileges |

#### pg_trigger (Stores trigger metadata)

| Column Name | Type  | Description |
|------------|------|-------------|
| tgname     | name | Trigger name |
| tgrelid    | oid  | Table the trigger is on |
| tgfoid     | oid  | Function to be called |

#### pg_type (Stores data type metadata)

| Column Name | Type  | Description |
|------------|------|-------------|
| typname    | name | Name of the data type |
| typlen     | int2 | Length of the data type |

#### pg_user_mapping (Stores user mappings for foreign servers)

| Column Name | Type  | Description |
|------------|------|-------------|
| umuser     | oid  | Local role being mapped |
| umserver   | oid  | Foreign server containing the mapping |
```
