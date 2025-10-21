Here is a list of possible errors you might come across and their possible solutions:

## P1001: Can't reach database server at localhost:1433
Solution: check the port on which sql server is running by executing the query below:
```Sql
USE master
GO
xp_readerrorlog 0, 1, N'Server is listening on' 
GO
```

Also, make sure sql server is listening on port 1433 by following these steps (**WINDOWS**):
1. Open SQL Server configuration manager:
    - Press `win + R`
    - Run: `SQLServerManager16.msc`
2. Enable and configure TCP/IP
    - In the left panel click: `SQL Server Network Configuration -> Protocols for SQLEXPRESS`
    - ON the right, right-click: `TCP/IP -> Properties`
    - In the protocols tab, set: `Enabled = Yes`
    - Switch to the IP Addresses tab
        - Scroll all the way down to IPAll
        - Clear TCP Dynamic Ports (make it blank)
        - Set TCP Port = 1433
3. Restart SQL Server Service to reflect these changes:
    - Press `win + R`
    - Run `services.msc`
    - Scroll to find `SQL Server (MSSQlSERVER)`.
    - Right click and click on restart.

## Error opening a TLS connection: The TLS settings didn't allow the connection to be established. Please review your connection string. (error: A certificate chain processed, but terminated in a root certificate which is not trusted by the trust provider.)

Solution: Add the following at the end of your connection string `encrypt=true;trustServerCertificate=true`

i.e:  
From:  
`DATABASE_URL="sqlserver://localhost:1433;database=DB_NAME;user=USERNAME;password=PASSWORD;"`  
to:  
`DATABASE_URL="sqlserver://localhost:1433;database=DB_NAME;user=USERNAME;password=PASSWORD;encrypt=true;trustServerCertificate=true"`

## Authentication failed against database server, the provided database credentials for `...` are not valid.

Solution: create a login and a user for the login and use the credentials in the connection string:

```sql
CREATE LOGIN [name] WITH PASSWORD = 'PASSWORD'; 
CREATE USER [name] FOR LOGIN prisma_user; 
ALTER SERVER ROLE sysadmin ADD MEMBER [name];
```

Example:
```sql
CREATE LOGIN john WITH PASSWORD = '1234'; 
CREATE USER john FOR LOGIN john; 
ALTER SERVER ROLE sysadmin ADD MEMBER john;
```

Tip: Avoid using `"` and `;` in the password as this will cause issues in the prisma connection string.

Your connection string should now look like this:  
```
DATABASE_URL="sqlserver://localhost:1433;database=DB_NAME;user=john;password=1234;encrypt=true;trustServerCertificate=true"
```

You can delete a login using the command:
```sql
DROP LOGIN [name];
```

You can list all logins with the command:
```sql
SELECT * FROM sys.sql_logins
```