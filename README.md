# SQL Server Table Partition Script

Attention !!!
- This script is created for SQL Server 2012 and above.
- The script is designed for creating table partitions based on date (monthly basis) as the primary key.
- This script will select the latest partition filegroup (if already created) and generate a script to create another filegroup.
- This will also create the partition schema name and schema function.
- In short, it will generate all the necessary scripts you just need to change before execution as per your requirement.


