# MySQL-ELK

MySQL demo with ELK ( ElasticSearch, Logstash, Kibana )

## Resources

- https://github.com/stevekm/pg-elk
- https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/
  - https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/linux-installation.html
  - https://container-registry.oracle.com/
  - https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/linux-installation-docker.html
  - https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/docker-mysql-getting-started.html
  - https://github.com/mysql/mysql-docker/blob/main/mysql-server/8.1/Dockerfile
  - https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/docker-mysql-more-topics.html
- https://www.elastic.co/blog/getting-started-with-the-elastic-stack-and-docker-compose

## Docker

- install Docker https://docs.docker.com/desktop/install/mac-install/
- make sure Docker Compose is installed https://docs.docker.com/compose/install/

## MySQL

- https://dev.mysql.com/doc/mysql-tutorial-excerpt/8.0/en/getting-information.html


Initialize the MySQL server

- if you have issues, then delete the `data/mysql` dir and start over

```bash
# start the MySQL container
docker compose up -d mysql

# check that its running
docker compose ps
# NAME                IMAGE                                        COMMAND                  SERVICE             CREATED             STATUS                 PORTS
# mysql               mysql/mysql-server:8.0                       "/entrypoint.sh mysq…"   mysql               2 hours ago         Up 2 hours (healthy)   3306/tcp, 33060-33061/tcp
#

# you can see the logs from the container with this command
docker logs mysql

# NOTE: the docker-compose.yml includes env vars for MySQL that auto configured
# the default db, and user account

# check that database was created automatically as per the docker-compose.yml settings
docker exec -ti mysql mysql -u admin -p -e 'SHOW DATABASES'
# Enter password:
# +--------------------+
# | Database           |
# +--------------------+
# | information_schema |
# | my-first-db        |
# | performance_schema |
# +--------------------+

# create the table for logs
docker compose exec -ti mysql mysql -u admin -p my-first-db -e "CREATE TABLE IF NOT EXISTS logs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ip VARCHAR(255),
    method VARCHAR(255),
    request_date TIMESTAMP,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
    "

# check that table exists
docker exec -ti mysql mysql -u admin -p my-first-db -e 'SHOW TABLES'
# +-----------------------+
# | Tables_in_my-first-db |
# +-----------------------+
# | logs                  |
# +-----------------------+

# add records into logs
docker compose exec -ti mysql mysql -u admin -p my-first-db -e "
INSERT INTO logs ( ip, method, request_date )
VALUES
('1.2.3.4', 'GET', '2024-04-15 12:00:00'),
('1.2.3.4', 'POST', '2024-04-15 12:00:05'),
('1.4.4.0', 'GET', '2024-04-15 12:00:01'),
('1.4.4.0', 'POST', '2024-04-15 12:00:03'),
('1.6.0.0', 'GET', '2024-04-15 12:00:02');
"


# show the contents
docker compose exec -ti mysql mysql -u admin -p my-first-db -e "SELECT * FROM logs;"
# Enter password:
# +----+---------+--------+---------------------+---------------------+
# | id | ip      | method | request_date        | timestamp           |
# +----+---------+--------+---------------------+---------------------+
# |  1 | 1.2.3.4 | GET    | 2024-04-15 12:00:00 | 2024-04-15 16:43:57 |
# |  2 | 1.2.3.4 | GET    | 2024-04-15 12:00:00 | 2024-04-15 16:49:06 |
# |  3 | 1.2.3.4 | POST   | 2024-04-15 12:00:05 | 2024-04-15 16:49:06 |
# |  4 | 1.4.4.0 | GET    | 2024-04-15 12:00:01 | 2024-04-15 16:49:06 |
# |  5 | 1.4.4.0 | POST   | 2024-04-15 12:00:03 | 2024-04-15 16:49:06 |
# |  6 | 1.6.0.0 | GET    | 2024-04-15 12:00:02 | 2024-04-15 16:49:06 |
# +----+---------+--------+---------------------+---------------------+

```