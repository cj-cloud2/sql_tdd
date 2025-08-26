To install **h2-2.3.232.jar**, automatically create a database at **D:\temp\h2db**, and connect to it, follow these step-by-step instructions for a typical Windows and Java development setup.

## Download and Prepare H2 JAR

1. Download the **h2-2.3.232.jar** file from the official H2 database website or a trusted Maven repository.
2. Place the JAR file in a convenient directory, such as `C:\h2` or your project’s `lib` folder.

## Create the Database Automatically

3. Ensure the `D:\temp\h2db` directory exists, or allow H2 to create it automatically when the database is first accessed.
4. The H2 database will create the files automatically upon connection if the directory and file do not exist.

## Launch H2 Console or Connect Using Java

### Using H2 Console

5. Open a Command Prompt at the directory containing `h2-2.3.232.jar`.
6. Run the following command to start the H2 console:

   ```sh
   java -jar h2-2.3.232.jar
   ```
   This will launch the H2 web console at http://localhost:8082.

### Using Java (Programmatic Connection)

7. Add the JAR file to your Java project or classpath.
8. Use the following JDBC URL to connect and automatically create the database:

   ```
   jdbc:h2:file:D:\temp\h2db;

   or
   
   jdbc:h2:file:D:\temp\h2db;DB_CLOSE_ON_EXIT=FALSE;AUTO_SERVER=TRUE


   ```
   OR
   ```
   jdbc:h2:file:D:\temp\h2db\h2db;DB_CLOSE_ON_EXIT=FALSE;AUTO_SERVER=TRUE
   ```
   - `jdbc:h2:file:/D:/temp/h2db`: Specifies file-based creation at the desired path.
   - `DB_CLOSE_ON_EXIT=FALSE`: Keeps the DB open.
   - `AUTO_SERVER=TRUE`: Enables multi-process access (optional).

9. Example Java code for connection (with automatic creation):

   ```java
   import java.sql.*;

   public class H2Connect {
       public static void main(String[] args) throws Exception {
           Class.forName("org.h2.Driver");
           Connection conn = DriverManager.getConnection(
               "jdbc:h2:file:/D:/temp/h2db", "sa", ""
           );
           Statement stmt = conn.createStatement();
           stmt.execute("CREATE TABLE IF NOT EXISTS test(id INT PRIMARY KEY, name VARCHAR(255));");
           stmt.close();
           conn.close();
       }
   }
   ```

## Connect via H2 Console

10. In the H2 console, enter the JDBC URL as above, user as `sa`, password blank (`""`).
11. Click **Connect**. The database will be created if it does not already exist.

## Verification

- The `D:\temp\h2db.mv.db` file will be created after the first connection and table creation.
- You can interactively work with tables and data via either the H2 console or your Java project.

***

Follow these steps to install and connect to **H2 2.3.232** with your database stored and auto-created in the desired directory.

***

### Reference
- H2 Database Official Docs & Quick Start Guides