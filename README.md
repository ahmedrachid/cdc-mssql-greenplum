# CDC using Debezium from MSSQL to Greenplum through GPSS

## Steps

```bash
# 1. Start the SQL Server container
docker-compose up -d sqlserver

# 2. Access the SQL Server container
docker-compose exec -ti sqlserver bash

# 3. Connect to SQL Server using sqlcmd
/opt/mssql-tools/bin/sqlcmd -Usa -P<YourPassword>

# 4. Create a new database

create database testdb;
go
use testdb;
go
EXEC sys.sp_cdc_enable_db;
go


# 5. Define the rooms table and insert sample data
CREATE TABLE rooms (
   id INTEGER IDENTITY(101,1) NOT NULL PRIMARY KEY,
   hotel_id VARCHAR(255) NOT NULL,
   name VARCHAR(255) NOT NULL,
   description VARCHAR(512),
   total_rooms INTEGER,
   used_rooms INTEGER,
   left_rooms INTEGER
);

INSERT INTO rooms (hotel_id, name, description, total_rooms, used_rooms, left_rooms)
VALUES
   ('hiltonhn', 'king-size', 'A room of king size', 20, 5, 15),
   ('hiltonhn', 'queen-size', 'A room of queen size', 40, 10, 30),
   ('pullmanhn', 'queen-size', 'A room of queen size', 10, 2, 8),
   ('pullmanhn', 'single room', 'A room for single', 6, 1, 5);

EXEC sys.sp_cdc_enable_table @source_schema = 'dbo', @source_name = 'rooms', @role_name = NULL, @supports_net_changes = 0;
GO

# 6. Start other services
docker-compose up -d

# 7. Connect to RabbitMQ, create a stream/queue, and bind it to the CDC exchange.

# 8. Start GPSS in the background
nohup gpss &

# 9. Define the rooms table in Greenplum (if not already defined)
CREATE TABLE rooms (
   id INT,
   hotel_id VARCHAR(255) NOT NULL,
   name VARCHAR(255) NOT NULL,
   description VARCHAR(512),
   total_rooms INT,
   used_rooms INT,
   left_rooms INT
);

# 10. Submit and start the GPSS job
gpsscli submit gpss_job.yaml
gpsscli start gpss_job
