# MySQL-ELK

MySQL demo with ELK ( ElasticSearch, Logstash, Kibana )

## Resources

- https://github.com/stevekm/pg-elk <- The original version of this repo using PostGreSQL instead of MySQL

## Docker

- install Docker https://docs.docker.com/desktop/install/mac-install/
- make sure Docker Compose is installed https://docs.docker.com/compose/install/

### Resources
- https://docs.docker.com/compose/networking/

## MySQL

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

# you can check the settings for the running container
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mysql

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

### Resources

- https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/
  - https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/linux-installation.html
  - https://container-registry.oracle.com/
  - https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/linux-installation-docker.html
  - https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/docker-mysql-getting-started.html
  - https://github.com/mysql/mysql-docker/blob/main/mysql-server/8.1/Dockerfile
  - https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/docker-mysql-more-topics.html
- https://www.elastic.co/blog/getting-started-with-the-elastic-stack-and-docker-compose
- https://dev.mysql.com/doc/refman/8.0/en/setting-environment-variables.html

- https://dev.mysql.com/doc/mysql-tutorial-excerpt/8.0/en/getting-information.html

- get the connector jar here https://dev.mysql.com/downloads/connector/j/
  - https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-j-8.3.0.tar.gz

## Logstash


First make sure you download and extract the driver for MySQL to use with Logstash

```bash
# Download the driver for MySQL
wget -P plugin/ https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-j-8.3.0.tar.gz
tar -zvxf plugin/mysql-connector-j-8.3.0.tar.gz -C plugin
# plugin/mysql-connector-j-8.3.0/mysql-connector-j-8.3.0.jar
```

Now you can use the Logstash container with commands like this

```bash
# NOTE: 'logstash' is the default entrypoint
# check logstash help
docker compose run logstash -h

# test that it works with example configs
docker compose run logstash -f /config/example.conf

# run it with the MySQL config
docker compose run logstash -f /config/logstash.conf

# output looks like this;

{
      "@timestamp" => 2024-04-15T18:19:33.235587971Z,
          "method" => "GET",
              "ip" => "1.4.4.0",
        "@version" => "1",
    "request_date" => 2024-04-15T12:00:01.000Z,
              "id" => 4,
       "timestamp" => 2024-04-15T16:49:06.000Z
}
{
      "@timestamp" => 2024-04-15T18:19:33.235019513Z,
          "method" => "GET",
              "ip" => "1.2.3.4",
        "@version" => "1",
    "request_date" => 2024-04-15T12:00:00.000Z,
              "id" => 2,
       "timestamp" => 2024-04-15T16:49:06.000Z
}
{
      "@timestamp" => 2024-04-15T18:19:33.234285596Z,
          "method" => "GET",
              "ip" => "1.2.3.4",
        "@version" => "1",
    "request_date" => 2024-04-15T12:00:00.000Z,
              "id" => 1,
       "timestamp" => 2024-04-15T16:43:57.000Z
}
{
      "@timestamp" => 2024-04-15T18:19:33.236090638Z,
          "method" => "GET",
              "ip" => "1.6.0.0",
        "@version" => "1",
    "request_date" => 2024-04-15T12:00:02.000Z,
              "id" => 6,
       "timestamp" => 2024-04-15T16:49:06.000Z
}
{
      "@timestamp" => 2024-04-15T18:19:33.235259971Z,
          "method" => "POST",
              "ip" => "1.2.3.4",
        "@version" => "1",
    "request_date" => 2024-04-15T12:00:05.000Z,
              "id" => 3,
       "timestamp" => 2024-04-15T16:49:06.000Z
}
{
      "@timestamp" => 2024-04-15T18:19:33.235836263Z,
          "method" => "POST",
              "ip" => "1.4.4.0",
        "@version" => "1",
    "request_date" => 2024-04-15T12:00:03.000Z,
              "id" => 5,
       "timestamp" => 2024-04-15T16:49:06.000Z
}

```

### Resources

- https://www.elastic.co/blog/getting-started-with-the-elastic-stack-and-docker-compose
- https://discuss.elastic.co/t/logstash-healthcheck/271088/2
- https://www.elastic.co/guide/en/logstash/current/dir-layout.html
- https://www.elastic.co/guide/en/logstash/current/docker-config.html
- https://www.elastic.co/guide/en/logstash/current/logstash-settings-file.html
- https://www.elastic.co/guide/en/logstash/current/environment-variables.html
- https://www.elastic.co/guide/en/logstash/current/config-setting-files.html
- https://stackoverflow.com/questions/59250301/jdbc-driver-not-loaded-while-using-logstash-in-docker
- https://medium.com/@bsrini/dockerizing-logstash-a-step-by-step-guide-ed7f4e594cb4
- JDBC plugin https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html
  - https://www.elastic.co/guide/en/logstash/current/plugins-integrations-jdbc.html
  - https://github.com/logstash-plugins/logstash-integration-jdbc


## Other Notes

- https://stackoverflow.com/questions/68202592/elasticsearch-healthcheck-on-docker-compose-failing