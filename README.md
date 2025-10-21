# Prisma With MSSQL Server
In this activity, you’ll learn how to use Prisma to interact directly with a SQL Server database.

## 1. Setup
### 1.1 Project Folder
Start by creating a new folder called `company` and open it in Visual Studio Code.

This will be the root directory of your project.

### 1.2 Initialize the Project as a Node.js Project
Before we can use Prisma, we need to set up a Node.js project.

This is where the package.json file comes in; it’s basically your project’s "manifest." It keeps track of dependencies, scripts, and metadata about your app.

Open the terminal inside your project folder and run:
```bash
npm init -y
```
The `-y` flag automatically accepts the default settings so you don’t have to manually answer the setup questions.

### 1.3 Install Prisma
Next, install Prisma as a development dependency:
```bash
npm i prisma -D
```
Why as a dev dependency?  
Because Prisma is a tool you’ll mainly use during development for database schema management and generating the Prisma client.

### 1.4 Set Up Prisma
Now, let’s initialize Prisma in the project. Run the following command:
```bash
npx prisma init --datasource-provider sqlserver
```
This command does three important things:
- Creates a new `prisma` directory containing a schema.prisma file, this is where you’ll define your data models and database connection.
- Creates a `.env` file at the root of your project, this is where environment variables (like your database connection string) are stored.
- Prepares the project for working with SQL Server as your database.

Open the .env file and update the DATABASE_URL with your actual SQL Server credentials:

```bash
DATABASE_URL="sqlserver://localhost:1433;database=<DB_NAME>;user=<USERNAME>;password=<PASSWORD>;
```
Replace:
- <DB_NAME> with the database name, in this case, let's name the database as `Company`.
- <USERNAME> with your sql server username.
- <PASSWORD> with your sql server password.

Prisma needs valid database credentials to connect to SQL Server.

If you don’t already have a dedicated username and password, you can create a new login and user in SQL Server using the following SQL commands.

```sql
CREATE LOGIN [user] WITH PASSWORD = 'password';
CREATE USER [user] FOR LOGIN [user];
ALTER SERVER ROLE sysadmin ADD MEMBER [user];
```
For example:
```sql
CREATE LOGIN john WITH PASSWORD = '1234';
CREATE USER john FOR LOGIN [john];
ALTER SERVER ROLE sysadmin ADD MEMBER [john];
```
Explanation:
- `CREATE LOGIN` - Creates a new SQL Server login at the server level. **This is the actual account Prisma will use to connect**.
- `CREATE USER` - Maps that login to a database user so it can access your database.
- `ALTER SERVER ROLE sysadmin ADD MEMBER` - Gives the new user full admin privileges. _For development purposes this is fine, but in production you’d use more restrictive permissions._

Once the user is created, go back to your .env file and update the `DATABASE_URL` with these new credentials.

```bash
DATABASE_URL="sqlserver://localhost:1433;database=Company;user=john;password=1234;
```

## 2. Create Models
Now that Prisma is set up, it’s time to define the data models that will represent the tables in your database.

For this activity, we’re working with two entities: `Department` and `Employee`.

Each model corresponds to a table in SQL Server and will determine the structure of the data Prisma works with.

### 2.1 Department Model
This model represents a department within the organization.

It should have the following fields:
- `id`: A string that uniquely identifies each department. This will act as the primary key.
- `departmentName`: A string that uniquely identifies each department. This will act as the primary key.
- `location`: A string representing where the department is located.

### 2.2 Employee Model
This model represents employees working in the organization.

It should include the following fields:
- `id`: A string that uniquely identifies an employee (primary key).
- `firstName`: A string containing the employee’s first name.
- `lastName`: A string containing the employee’s last name.
- `email`: A string representing the employee’s email address. Each email must be unique.
- `salary`: This field represents the employee’s monthly salary. It should be stored as a numeric value that allows for decimal precision. (Hint: prisma doesn't have a `currency` type, so use `Decimal` here)
- `departmentId`: A string that references the `id` of a department in the Department table. This establishes a relationship between employees and departments.

## 3. Run migrations
Once your models are fully defined in the `schema.prisma` file, the next step is to run a migration.

A migration is basically Prisma’s way of translating your schema into actual SQL commands that create or modify tables in your database. In other words, it takes what you wrote in the schema and builds the real database structure in SQL Server.

Run the following command:
```bash
npx prisma migrate dev --name init
```
- `npx prisma migrate dev`: runs the migration in development mode.
- `--name init`: gives your migration a name, e.g "init" for the first one, you can use any descriptive name here like `add-employee-model`, `update-department-schema` e.t.c.

If the migration runs successfully, the corresponding tables and columns will now exist in your database. Also, if the database doesn't exist, prisma will automatically create it.

You can confirm the tables were created in two ways:
- Using SSMS.
- Using prisma studio, run: `npx prisma studio`. This opens an interactive UI where you can view and manage your data in the browser.

If you make changes to your schema later (e.g., add a new field), just run the same command again with a new migration name, and Prisma will generate the SQL to update your database.

[Here](./possible_errors_and_possible_solutions.md) is a list of possible errors you might come across and their possible solutions.

## 3. Create Departments
Write a function named `createDepartment` that takes the following parameters:
- `id`: a string that uniquely identifies a department
- `departmentName`: the name of the department
- `location`: the location of the department.
The function should use Prisma's `create` method to insert a new record into the database.

```js
createDepartment("DPT001", "Computer Science", "Nairobi Campus");
createDepartment("DPT002", "Information Technology", "Mombasa Campus");
createDepartment("DPT003", "Software Engineering", "Kisumu Campus");
createDepartment("DPT004", "Cyber Security", "Eldoret Campus");
createDepartment("DPT005", "Business Studies", "Nakuru Campus");
createDepartment("DPT006", "Business and Finance", "Kisii Campus");
```

## 4. Get all departments
Write a function named `getDepartments` that retrieves all department records from the database.

Inside the function, use Prisma’s `findMany()` method on the department model to fetch the complete list of departments.

## 5. Get department
Write a function named `getDepartment` that retrieves a single department from the database based on its ID.

The function should take one parameter, the department ID, and use Prisma;s `findUnique()` method to fetch the matching department record.

If the department is found, log it to the console, if it is not found, log the message "Department not found".

## 6. Delete department
Write a function called `deleteDepartment` that deletes the "Business and Finance" department with department ID "DPT006".

## 7. Update department
Write a function named `updateDepartment` that updates the location of the Cyber Security department (DPT004) to "Kericho Campus".

The function should use Prisma's `update()` method to find the department by its ID (DPT004) and modify the location field.

## 8. Insert Employees
Write a function named `createEmployee` that adds a new employee record to the database.
- `id`: a string that uniquely identifies an employee
- `firstName`: The employee's first name.
- `lastName`: The employee's last name..
- `email`: The employee's email address.
- `salary`: The employee's salary amount.
- `departmentId`: The ID of the department the employee belongs to.
The function should use Prisma's `create` method to insert a new employee record into the database.

```js
createEmployee("EMP001", "John", "Mwangi", "john.mwangi@company.com", 65000.00, "DPT001");
createEmployee("EMP002", "Alice", "Otieno", "alice.otieno@company.com", 92000.00, "DPT002");
createEmployee("EMP003", "Brian", "Kariuki", "brian.kariuki@company.com", 88000.00, "DPT003");
createEmployee("EMP004", "Grace", "Mutua", "grace.mutua@company.com", 97000.00, "DPT004");
createEmployee("EMP005", "Kevin", "Omondi", "kevin.omondi@company.com", 66000.00, "DPT005");
createEmployee("EMP006", "Linda", "Njeri", "linda.njeri@company.com", 91000.00, "DPT001");
createEmployee("EMP007", "Samuel", "Kiptoo", "samuel.kiptoo@company.com", 63000.00, "DPT002");
createEmployee("EMP008", "Mary", "Wanjiku", "mary.wanjiku@company.com", 89000.00, "DPT003");
createEmployee("EMP009", "Peter", "Kamau", "peter.kamau@company.com", 95000.00, "DPT004");
createEmployee("EMP010", "Faith", "Achieng", "faith.achieng@company.com", 78000.00, "DPT005");
createEmployee("EMP011", "Ann", "Chebet", "ann.chebet@company.com", 78000.00, "DPT006");
```

## 9. Get all Employees
Write a function named `getEmployees` that retrieves all employees from the database, including their department details.

Your function should use Prisma's `findMany()` method with the `include` option to fetch each employee along with their related department information.

## 10. Retrieve Employees by Salary Range
Write a function named `getBySalaryRange` that retrieves all employees whose salaries fall within a specified range.

The function accepts two parameters:
- `min`: minimum salary value.
- `max`: maximum salary value.
Before performing the query, add a validation check:
- if `min` is greater than `max`, throw an error `"Minimum cannot be greater than maximum"`

If the values are valid, use Prisma's `findMany()` method to fetch all employees whose salary is between the given `min` and `max` values.

## 11. Delete Employee
Write a function `deleteEmployee` that deletes the employee with id `EMP011`.

## 12. Update Employee
Write a function called `updateEmployee` that updates the `salary of employee with id "EMP010" to 90000 respectively.