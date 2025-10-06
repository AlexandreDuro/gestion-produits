FROM mysql:8.4

ENV MYSQL_ROOT_PASSWORD=root
ENV MYSQL_DATABASE=gestion_produits

COPY gestion_produits.sql /docker-entrypoint-initdb.d/

EXPOSE 3306