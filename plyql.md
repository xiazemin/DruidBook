Some time ago at Imply, we launched PlyQL, a command line utility that provides an SQL-like interface to Druid via Plywood. We heard a lot of positive feedback as many people prefer to use SQL over Druid’s native JSON-over-HTTP interface. The most common question we hear about PlyQL is how one can interface to it programmatically either from user created apps or from existing SQL based BI tools. We are pleased to announce we just released PlyQL 0.7 with two brand new programmatic interface options: SQL-over-HTTP and MySQL Gateway.

SQL-over-HTTP

When you run:

plyql -h your.druid.broker:8082 --json-server 8083

1

PlyQL will start listening on port 8083 where you can POST some SQL like so:

curl -X POST '[http://localhost:8083/plyql](http://localhost:8083/plyql)' \

-H 'content-type: application/json' \

-d '{

"sql": "SELECT COUNT\(\*\) FROM wikipedia WHERE \"2015-09-12T01\" &lt;= \_\_time AND \_\_time &lt; \"2015-09-12T02\""

}'

Check out the json-server docs for more info about possible parameters.

MySQL Gateway

In this experimental feature we use the excellent node-mysql2 library to provide a MySQL server interface.

When you run:

plyql -h your.druid.broker:8082 --experimental-mysql-gateway 3307

PlyQL will start in a MySQL-like server mode and will be queryable from a MySQL client, ODBC, or JDBC driver.

SELECT user, COUNT\(\*\) AS Count FROM wikipedia WHERE channel = "en" GROUP BY user ORDER BY Count DESC LIMIT 3

PlyQL will attempt to respond faithfully to SET, SHOW, DESCRIBE and any other meaningful query so that it appears to the client as a real MySQL server with read-only permissions.

SHOW FULL COLUMNS IN wikipedia LIKE "%e%"

While not all of MySQL is supported, the goal is to provide a way for new and existing tools that already communicate via ODBC or JDBC interfaces to talk to Druid using PlyQL.

Here is an example of how you could use the existing MySQL JDBC driver to query the above PlyQL gateway.

import java.sql.SQLException;

import java.sql.DriverManager;

import java.sql.Connection;

import java.sql.Statement;

import java.sql.ResultSet;

class DruidQuery

{

public static void main\(String\[\] args\) throws SQLException

{

C

onnection con = DriverManager.getConnection\\("jdbc:mysql://127.0.0.1:3307/plyql1"\\);



Statement stmt = con.createStatement\\(\\);



ResultSet rs = stmt.executeQuery\\(



"SELECT page, count\\(\\*\\) AS cnt FROM wikipedia GROUP BY page ORDER BY cnt DESC LIMIT 15"



\\);







while \\(rs.next\\(\\)\\) {



String page = rs.getString\\("page"\\);



long count = rs.getLong\\("cnt"\\);



System.out.println\\(String.format\\("page\\[%s\\] count\\[%d\\]", page, count\\)\\);



}

}

}

We are looking for your help to make this tool better. Whenever a query cannot be handled it will throw an error like so:

