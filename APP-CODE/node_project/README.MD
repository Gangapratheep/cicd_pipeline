
docker network create --driver bridge --subnet 172.20.0.0/16 node_mysql_net
```

```bash
docker volume create mysql_data
```

```bash
docker run -d --name some-mysql --restart always --network node_mysql_net --ip 172.20.0.2 -e MYSQL_ROOT_PASSWORD=P@ssw0rd -e MYSQL_DATABASE=sql_login -v mysql_data:/var/lib/mysql -p 3306:3306 mysql:5.7
```

```bash
docker run -d --name phpmyadmin --restart always --network node_mysql_net -e PMA_HOST=172.20.0.2 -e PMA_PORT=3306 -p 8080:80 phpmyadmin
```

```bash
docker build -t nodejs-app .
```

```bash
docker run -d --name nodejs-app --restart always --network node_mysql_net --env-file .env -p 5000:5000 nodejs-app
```

Let me know if you'd like a `.sh` script to automate this.
