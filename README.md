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
docker compose up -d mysql

```