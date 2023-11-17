# ProxySQL + MySQL + Prometheus + Grafana using Docker

![Blank Diagram](https://user-images.githubusercontent.com/13511520/83347412-e2c4f280-a35f-11ea-8048-2b0645e99245.png)

## Configuration

- ProxySQL
   - `:6032` (admin, `user`: admin2, `pass`: pass2)
   - `:6033` (MySQL endpoint, `user`: root, `pass`: password)

- MySQL replication
  - master x 1
  - slave x 2
  - `user`: root, `pass`: password

- Prometheus
  - `:9090`
  - using `msqld-exporter` to obtain mysql metrics

- Grafana
  - `:3000`
  - `id`: admin
  - `password`: admin


## Getting Started

```
docker-compose up -d
```

```
                      Name                                     Command               State                       Ports
-------------------------------------------------------------------------------------------------------------------------------------------
proxysql-mysql-replication-master                   docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp, 33060/tcp
proxysql-mysql-replication-mysqld-exporter-master   /bin/mysqld_exporter             Up      0.0.0.0:9104->9104/tcp
proxysql-mysql-replication-mysqld-exporter-slave1   /bin/mysqld_exporter             Up      0.0.0.0:9105->9104/tcp
proxysql-mysql-replication-mysqld-exporter-slave2   /bin/mysqld_exporter             Up      0.0.0.0:9106->9104/tcp
proxysql-mysql-replication-proxysql                 proxysql -f -D /var/lib/pr ...   Up      0.0.0.0:6032->6032/tcp, 0.0.0.0:6033->6033/tcp
proxysql-mysql-replication-slave1                   docker-entrypoint.sh mysqld      Up      0.0.0.0:3307->3306/tcp, 33060/tcp
proxysql-mysql-replication-slave2                   docker-entrypoint.sh mysqld      Up      0.0.0.0:3308->3306/tcp, 33060/tcp
proxysql-mysql-replication_grafana_1                /run.sh                          Up      0.0.0.0:3000->3000/tcp
proxysql-mysql-replication_prometheus_1             /bin/prometheus --config.f ...   Up      0.0.0.0:9090->9090/tcp
```

## Check status

- MySQL master

    ```
     docker-compose exec mysql-master sh -c "export MYSQL_PWD=password; mysql -u root -e 'show master status\G'"
    ```

- MySQL slave
    ```
     docker-compose exec mysql-slave1 sh -c "export MYSQL_PWD=password; mysql -u root -e 'show slave status\G'"
    ```


- ProxySQL

  ```
  $ mysql -h 0.0.0.0 -P 6032 -u admin2 -p -e 'select * from mysql_servers'
  Enter password:
  +--------------+--------------+------+-----------+--------+--------+-------------+-----------------+---------------------+---------+----------------+---------+
  | hostgroup_id | hostname     | port | gtid_port | status | weight | compression | max_connections | max_replication_lag | use_ssl | max_latency_ms | comment |
  +--------------+--------------+------+-----------+--------+--------+-------------+-----------------+---------------------+---------+----------------+---------+
  | 10           | mysql-master | 3306 | 0         | ONLINE | 1      | 0           | 100             | 5                   | 0       | 0              |         |
  | 20           | mysql-slave1 | 3306 | 0         | ONLINE | 1      | 0           | 100             | 5                   | 0       | 0              |         |
  | 20           | mysql-slave2 | 3306 | 0         | ONLINE | 1      | 0           | 100             | 5                   | 0       | 0              |         |
  +--------------+--------------+------+-----------+--------+--------+-------------+-----------------+---------------------+---------+----------------+---------+
  ```
