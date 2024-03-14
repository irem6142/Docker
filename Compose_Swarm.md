### wordpress:latest ve mysql:5.7 resmi imajlarından oluşan servislerin olacağı bir Docker Stack oluşturmanızı istiyorum. Bu stack'i deploy etmek için bir docker-compose.yml dosyası oluşturmanızı ve deploy etmenizi ve wordpress'in düzgün çalıştığını, yani düzgün şekilde deploy edebildiğinizi teyit etmenizi bekliyorum. Tek bir kuralım var. Wordpress ve mysql servislerini ayağa kaldırırken onlara bir mysql şifresi belirtmeniz gerekiyor ve bunu da enviroment variable olarak servise atayabilirsiniz. Fakat ben bunu direk compose file içinden belirlememenizi onun yerine bunu bir secret olarak tanımlayarak o şekilde atamanızı istiyorum. Ayrıca bu secret'ı da aynı docker- compose.yml dosyası içerisinde oluşturmanızı bekliyorum.
#### • wordpress:latest ve mysql:5.7 kullanan servisler ile stack oluştur mysql ve wordpress imajlarından container yaratırken mysql şifresi belirlemeniz gerekiyor. Bunu secret olarak belirle
#### • Bu secret'i da aynı docker-compose.yml içerisinde oluştur.

```
1.Adım: touch docker-compose.yml
2.adım: touch pass_mysql.txt
3.adım olarak  docker-compose.yml içine girerek aşağıdaki kodları yazarız.
version: '3.7'

services:
    db:
        image: mysql:5.7
        environment:
            MYSQL_ROOT_PASSWORD: 1
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD_FILE: /run/secrets/pass_mysql
        secrets:
            - pass_mysql
        networks:
            - wp_overlay

    wordpress:
        image: wordpress:latest
        depends_on:
            - db
        environment:
            WORDPRESS_DB_HOST: db:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD_FILE: /run/secrets/pass_mysql
            WORDPRESS_DB_NAME: wordpress
        secrets:
            - pass_mysql
        ports:
            - "8080:80"
        networks:
            - wp_overlay
        deploy:
            replicas: 1
            update_config:
                parallelism: 2
                delay: 10s

networks:
    wp_overlay:
        driver: overlay

secrets:
    pass_mysql:
        file: pass_mysql.txt

Kodları yazdıktan sonra docker stack deploy -c ./docker-compose.yml wp_db(ad veriyoruz)
Tarayıcıdan bakmak için de 127.0.0.1:8080  

```
