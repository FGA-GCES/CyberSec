# Como iniciar os ataques locais?

Crie uma estrutura de pasta e arquivos em seu computador como a seguir
```
./
├── wp-content/
└── docker-compose.yml
```

Dentro do arquivo `docker-compose.yml` insira:
```yml
version: "3.6"

services:
  wordpress:
    image: wordpress:latest
    container_name: wordpress
    volumes:
      - ./wp-content:/var/www/html/wp-content
    environment:
      - WORDPRESS_DB_NAME=wordpress
      - WORDPRESS_TABLE_PREFIX=wp_
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=root
      - WORDPRESS_DB_PASSWORD=password
    depends_on:
      - db
      - phpmyadmin
    restart: always
    ports:
      - 8080:80

  db:
    image: mariadb:latest
    container_name: db
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_USER=root
      - MYSQL_PASSWORD=password
      - MYSQL_DATABASE=wordpress
    restart: always

  phpmyadmin:
    depends_on:
      - db
    image: phpmyadmin/phpmyadmin:latest
    container_name: phpmyadmin
    restart: always
    ports:
      - 8180:80
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: password

volumes:
  db_data:
```

Agora basta rodar o comando em seu terminal
```bash
docker compose up
```

Entrar na url http://localhost:8080, configurar o servidor wordpress e instalar na aba de plugins o tainacan e partir para o ataque
