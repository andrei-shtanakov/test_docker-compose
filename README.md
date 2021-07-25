За основу были взяты образы nginx и apache2, соответственно, докерафйлы с фафлами конфигураций  статическими html страницами в каталогах test_nginx и test_httpd. (Докерфайлы тоже тестировались с внешними статическими страницами).
Структура папокЖ

├── docker-compose.yaml
├── Screenshot from 2021-07-25 18-34-37.png
├── Screenshot from 2021-07-25 18-35-55.png
├── static
│   └── index.html
├── test_httpd
│   ├── Dockerfile
│   ├── files
│   │   └── index.html
│   └── my-httpd.conf
└──── test_nginx
    ├── Dockerfile
    ├── files
    │   └── index.html
    └── .nginx
        └── nginx.conf



Затем создается локальный репозиторий:

mkdir /dockerrepo
docker run -d -p 5000:5000 --restart=always --name registry -v /dockerrepo:/var/lib/registry registry:2
 
После удаляются все образы, заново создаются образы, скачиваясь с внешнего репозитария.
Тегтровались в соответствии с внутренним репозиторием и загружались в него.
Пример команд на примере apache2
 
docker build -t httpd-test .
docker tag httpd-test:latest localhost:5000/httpd
docker push localhost:5000/httpd


В конце запускались сервисы из папки с файлом для docer-compose и там же папка static, содержащая статическую страницу index.html.

docker-compose up

*********************************************************
ниже приведены файлы для docker-compose и Dockerfile


docker-compose.yaml:


version: "3"
services:
  web_nginx:
    image: localhost:5000/nginx
    ports:
      - "8000:80"
    volumes:
      - "./static:/usr/share/nginx/html"
    networks:
      net1:
        ipv4_address: 172.16.238.2
      default:
    deploy:
      resources:
        limits:
          cpus: "0.50"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 128M
  web_httpd:
    image: localhost:5000/httpd
    volumes:
      - "./static:/usr/local/apache2/htdocs/"
    ports:
      - "8001:80"
    networks:
      net2:
        ipv4_address: 172.16.8.2
      default:
    deploy:
      resources:
        limits:
          cpus: "0.50"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 128M

networks:
  net1:
    driver: bridge
    internal: true
    ipam:
     driver: default
     config:
       - subnet: 172.16.238.0/24
  net2:
    driver: bridge
    internal: true
    ipam:
     driver: default
     config:
       - subnet: 172.16.8.0/24





#External network actually
  default:
    driver: bridge


**********************************************

Dockerfile (nginx)

FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY ./files/ /usr/share/nginx/html
COPY ./.nginx/nginx.conf /etc/nginx/nginx.conf
ENTRYPOINT ["nginx", "-g", "daemon off;"]

**********************************************

Dockerfile (apache2)

FROM httpd:2.4
COPY ./my-httpd.conf /usr/local/apache2/conf/httpd.conf
COPY ./files/ /usr/local/apache2/htdocs/
COPY files /var/www/public
RUN chown -R www-data:www-data /var/www/public
EXPOSE 80


