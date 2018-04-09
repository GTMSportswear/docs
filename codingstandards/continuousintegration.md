# Database Continuous Integration
## Overview
Database Continuous Integration (CI) is a method in which changes to the database are recorded incrementally in order to provide a continuously updated database. This form of integration allows for developers to have local copies of a database and to update their local database incrementally.  

## Install vs. Update
For each database using the CI design there are two levels of versioning that need to be changed. They are `install` and `update`.

### Install
The `install` changes are all the changes needed to create the database from scratch. This will include all table declorations and stored procedures. The `install` changes will be applied whenever a devoloper wants to create a database. This reqires the `install` directory to have **the current state of the database**. All changes to the database will modify the `install` directory.

### Update
The `update` changes are all the changes needed to get the database from point **A** to point **B**. The `update` changes are versioned with a unique modifier so that the database knows what updates it has completed and which ones it needs to run. The PROD database will run the `update` changes whenever a new change is released. 

## Performing a Change
When making a change to a database using the CI design follow these steps.

### 1. Modify Install directory

Modifiy the `*ProjectName*/Database/Install` directory to reflect the current state of the database. The `Install` directory has multiple subdirectories for the different components of the database. 
* The `Functions` directory stores functions.
* The `Routines` directory stores Stored Procedures.
* The `Security` directory stores Schemas.
* The `Tables` directory stores Table Declorations.
* The `Types` directory stores Type Descriptions.

Inside the `Install` directory is a C# file that controls what scripts are loaded on a database install. 

A script is loaded with the command `Execute.EmbeddedScript("Database.Install.Tables.dbo.Customer.sql");`. This command takes as a parameter the path of the script. So the `Customer.sql` script is located at `*ProjectName*/Database/Install/Tables/dbo.Customer.sql`. 

**When creating a new file or changing a filename make sure to add the script to the C# file.** When changing a file verify that it is in the C# file. 

> Note: All scripts should be rerunnable. i.e. if a create table script is ran on a database that already has that table, that command will be ignored.

### 2. Modify Update directory

The `update` directory keeps track of changes that will advance the database from the previous state to the current state. Each update will be stored in it's own subdirectory. 

**Table changes will include SQL scripts** in the update subdirectories while **all other changes will only have the C# migration file**. 

#### How to create migration C# file

All C# migration files will import [FluentMigrator](https://github.com/fluentmigrator/fluentmigrator). They will also include a *Migration* tag that is formated according to the date and time of the change. This formatting ensures that migrations tags are unique. 
> \[`Migration(201804091501)`\] would correspond to April 9, 2018 at 3:01 PM.

The migration files will then reference the scripts that need to be run to execute the updates to the database.
