# Database Continuous Integration
## Overview
Database Continuous Integration (CI) is a method in which changes to the database are recorded incrementally in order to provide a continuously updated database. This form of integration allows for developers to have local copies of a database and to update their local database incrementally. Database CI enables automated testing of data access code and repositories.

## Install vs. Update
For each database using the CI design there are two levels of versioning that need to be changed. They are `install` and `update`.

### Install
The `install` changes are all the changes needed to create the database from scratch. This will include all table declarations and stored procedures. The `install` changes will be applied whenever a developer wants to create a database. This requires the `install` directory to have **the current state of the database**. All changes to the database will modify the `install` directory.

### Update
The `update` changes are all the changes needed to get existing databases from point **A** to point **B**. The `update` changes are versioned with a unique modifier so that the database knows what updates it has completed and which ones it needs to run. The existing environment, such as TEST or PROD, will have the `update` changes applied when the new version is released.

## Performing a Change
When making a change to a database using the CI design follow these steps.

### 1. Modify Install directory

Modify the `*ProjectName*/Database/Install` directory to reflect the current state of the database. The `Install` directory has multiple subdirectories for the different components of the database. 

![Install folder](/images/LegacyWebInstallFolder.png)

* The `Routines` directory stores Functions and Stored Procedures.
* The `Security` directory stores Schemas, Roles, etc.
* The `Tables` directory stores Table Declarations.
* The `Types` directory stores Type Descriptions.

Inside the `Install` directory is a C# file that controls what scripts are loaded on a database install. 

```C#
// Example of Install.cs file that controls install migrations

// Import FluentMigrator
using FluentMigrator;

namespace Gtm.Databases.LegacyWeb.Database.Install
{
   // Add the migration tag with the unique date-time identifier
   [Migration(201804021030)]
   // Use the ForwardOnlyMigration abstract class
   public class Database : ForwardOnlyMigration
   {
      // Override the Up() function
      public override void Up()
      {
         // Run scripts using the Execute.EmbeddedScript() command. The install file will
         // have all scripts needed to create the database environment.
         
         // Schema Scripts
         Execute.EmbeddedScript("Database.Install.Security.Schemas.Schema1.sql");
         Execute.EmbeddedScript("Database.Install.Security.Schemas.Schema2.sql");
         
         // Table Scripts
         Execute.EmbeddedScript("Database.Install.Tables.Schema1.Table1.sql");
         Execute.EmbeddedScript("Database.Install.Tables.Schema1.Table2.sql");
         Execute.EmbeddedScript("Database.Install.Tables.Schema1.Table3.sql");
         Execute.EmbeddedScript("Database.Install.Tables.Schema2.Table4.sql");
         Execute.EmbeddedScript("Database.Install.Tables.Schema2.Table5.sql");
         Execute.EmbeddedScript("Database.Install.Tables.Schema2.Table6.sql");
         
         // Routine Scripts
         Execute.EmbeddedScript("Database.Install.Routines.Schema1.StoredProcedure1.sql");
         Execute.EmbeddedScript("Database.Install.Routines.Schema1.StoredProcedure2.sql");
         Execute.EmbeddedScript("Database.Install.Routines.Schema1.StoredProcedure3.sql");
         Execute.EmbeddedScript("Database.Install.Routines.Schema2.StoredProcedure4.sql");
         Execute.EmbeddedScript("Database.Install.Routines.Schema2.StoredProcedure5.sql");
      }
   }
}
```

A script is loaded with the command `Execute.EmbeddedScript("Database.Install.Tables.dbo.Customer.sql");`. This command takes as a parameter the path of the script. So the `Customer.sql` script is located at `*ProjectName*/Database/Install/Tables/dbo.Customer.sql`. 

**When creating a new file or changing a filename make sure to add the script to the C# migration file.** When changing a file verify that it is in the C# migration file. 

> **Note: All scripts should be rerunnable. i.e. if a create table script is ran on a database that already has that table, that command will be ignored.**

### 2. Modify Update directory

The `update` directory keeps track of changes that will advance the database from the previous state to the current state. Each update will be stored in its own subdirectory. 

![Update folder](/images/LegacyWebUpdateFolder.png)

**Table changes will include SQL scripts** in the update subdirectories while **all other changes will only have the C# migration file**. 

**The build action of the script files must be `Embedded Resource` in order for FluentMigrator to correctly read the file.**

![Embedded Resource](/images/LegacyWebEmbeddedResource.png)

#### How to create migration C# file

All C# migration files will import [FluentMigrator](https://github.com/fluentmigrator/fluentmigrator). They will also include a *Migration* tag that is formatted according to the date and time of the change. This formatting ensures that migrations tags are unique. 

```C#
// Example of NewUpdate.cs file that controls update migrations

// Import FluentMigrator
using FluentMigrator;

namespace Gtm.Databases.LegacyWeb.Database.Update.NewUpdate
{
   // Add the migration tag with the unique date-time identifier
   [Migration(201804091501)]
   // Use the ForwardOnlyMigration abstract class
   public class AddProductVersion : ForwardOnlyMigration
   {
      // Override the Up() function
      public override void Up()
      {
         // Execute a script that added a table.
         // Notice that the add table script is located in the Install directory
         Execute.EmbeddedScript("Database.Install.Tables.schema.AddTableScript.sql");
         
         // Execute a script that modified a table.
         // Notice that the modify table script is located in the Update directory
         // The Install directory should have been modified to reflect the action of this script.
         Execute.EmbeddedScript("Database.Update.NewUpdate.schema.ModifyTableScript.sql");
         
         // Execute a script that added OR modified a stored procedure.
         // Notice that any stored procedure scripts should be located in the Install directory
         Execute.EmbeddedScript("Database.Install.Routines.schema.StoredProcedureScript.sql");
      }
   }
}

```

> **Note: \[`Migration(201804091501)`\] would correspond to April 9, 2018 at 3:01 PM.**
