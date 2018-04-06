# Database Continuous Integration
## Overview
Database Continuous Integration is a method in which changes to the database are recorded either as `install` or `update`. 
The `install` changes are all the changes needed to create the database from scratch. This will include all table declorations and stored procedures.
The `update` changes are all the changes needed to get the database from point **A** to point **B**. The `update` changes are versioned with a unique modifier so that the database knows what updates it has completed and which ones it needs to run. 
