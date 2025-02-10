# Tablespace-And-Schema

### TableSpace:- 
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
```
  Expand Storage Easily: If a particular disk is running out of space, you can create a new tablespace on a larger or less used disk and move objects there without interrupting database operations.
  Selective Backup: You can back up only specific tablespaces that contain critical data rather than backing up the entire database, making the process faster and more efficient.
  Recovery Strategies: During recovery, you can restore critical tablespaces first, allowing partial recovery if full recovery would take too long.
  Use Different Disk Types: Different types of data can be stored on appropriate storage media (e.g., using SSDs for frequently accessed data and HDDs for archival data).
  Free Up Space Easily: By moving data between tablespaces, you can free up space on specific storage devices without complex data migrations.
```

Examples:- 
```
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

```
```
- Key columns in pg_tablespace:
    • spcname: Name of the tablespace.
    • spcowner: OID of the owner (role).
    • spclocation: The directory location of the tablespace (older versions).
    • spcacl: Access control list (permissions).
```
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------

#### Schema:-
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
```
Example:
CREATE TABLE my_table (id SERIAL PRIMARY KEY);
-- This table will be created in the public schema by default:
-- public.my_table
```
3. information_schema
    • Description: The information_schema schema contains read-only views that provide metadata about the database, such as tables, columns, data types, and constraints.
    • Usage: This schema is used to query system catalog information in a standardized way, making it useful for inspecting the structure of the database.
```
Example:
[SELECT * FROM information_schema.tables;]
```
5. pg_catalog
    • Description: The pg_catalog schema is another system schema that contains PostgreSQL's system tables and functions. It holds metadata related to database internals such as tables, columns, indexes, and functions.
    • Usage: Internally used by PostgreSQL, but advanced users can also query pg_catalog for more detailed system-level information than information_schema provides.
```
Example:
[SELECT * FROM pg_catalog.pg_tables;]
```
7. pg_toast
    • Description: The pg_toast schema is used internally by PostgreSQL to store large objects that exceed a certain size limit. When a table has large text or binary data (like large TEXT or BYTEA fields), PostgreSQL may store these in pg_toast to manage space efficiently.
    • Usage: It's generally not accessed by users directly but is important for how PostgreSQL manages large data internally.
   ```
   If the column sizeis 2KB postgresqlautomaticallymoves it to toast table.
   ```
   
9. pg_temp_nnnn (Temporary Schemas)
    • Description: When you create temporary tables, PostgreSQL automatically creates temporary schemas (like pg_temp_1234). Each session has its own temporary schema, where all temporary tables reside.
    • Usage: These schemas are session-specific and are automatically dropped when the session ends.
   
11. pg_toast_temp_nnnn
    • Description: Similar to pg_toast, the pg_toast_temp_nnnn schemas are used internally for managing large objects for temporary tables. Like pg_temp_nnnn, these schemas are session-specific.
Managing Default Schemas
    • Search Path: PostgreSQL uses a search path to determine which schemas to search when you reference an object without specifying the schema. By default, the search path includes public, pg_catalog, and other system schemas.
``
To check the current search path:
SHOW search_path;
Example output:
search_path 
--------------
"$user", public
```
- In this case, $user means PostgreSQL will first look for a schema that matches the current user’s name. If no such schema exists, it defaults to the public schema.
- Changing the Search Path: You can adjust the search path for your session or permanently for a role or database.
```
  SET search_path TO schema1, schema2;
```
- In summary, PostgreSQL provides a set of default schemas for system management (pg_catalog, information_schema, pg_toast) and user access (public). You can manage how these schemas interact with your objects using the search path and schema privileges.
- Syntax for Schema Management:-
--------------------------------------------------
1. Creating a Schema:
```
CREATE SCHEMA schema_name;
Example:
CREATE SCHEMA sales;
```
----------------------------------------------------------
2. Creating a Schema Owned by a User:
```
CREATE SCHEMA schema_name AUTHORIZATION user_name;
Example:
CREATE SCHEMA sales AUTHORIZATION john;
```
---------------------------------------------------------
3. Listing Schemas:
```
SELECT schema_name 
FROM information_schema.schemata;
```
------------------------------------------------------
4. Creating an Object in a Specific Schema:
```
CREATE TABLE schema_name.table_name (
   column_name data_type
);
Example:
CREATE TABLE sales.customers (
   customer_id SERIAL PRIMARY KEY,
   name VARCHAR(100)
);
```
------------------------------------------------------
5. Setting a Search Path (Schema Priority):
```
You can define the search path so that PostgreSQL knows where to look for objects.
SET search_path TO schema_name;
Example:
SET search_path TO sales;
This will make the sales schema the default for subsequent object references.
```
-------------------------------------------------------
6. Dropping a Schema:
```
DROP SCHEMA schema_name CASCADE;
    • CASCADE: Automatically drops all objects within the schema.
    • RESTRICT: Prevents the schema from being dropped if it contains any objects (this is the default).
Example:
DROP SCHEMA sales CASCADE;
```
-------------------------------------------------------
7. Schema Privileges:
```
You can manage schema permissions using GRANT and REVOKE commands.
    • Granting permissions on a schema:
GRANT USAGE ON SCHEMA schema_name TO user_name;
    • Revoking permissions on a schema:
REVOKE USAGE ON SCHEMA schema_name FROM user_name;
```

