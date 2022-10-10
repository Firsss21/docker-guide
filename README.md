## Docker quick guide

### Что такое Docker?

Docker - платформа контейнеризации, с помощью которой можно автоматизировать создание приложений, их доставку и управление.

+ Контейнер - способ упаковки приложения со всеми необходимыми зависимостями и конфигурациями, которым можно легко "обмениваться"

+ До контейнеров - нужно было все устанавливать на систему и каждый все нужно было настраивать в зависимости от OS.

+ С контейнерами - все становится изолированно

+ Контейнер состоит из нескольких "слоев"

+ В большинстве своем контейнеры `Linux based`, а также используют `alpine` версию для минимального размера

### Разница между Docker Image и Docker Container

+ Docker Image - образ, который мы запускаем со всеми настройками
	
+ Docker Container - запущенный образ, запускаемое окружение для образа, уже для него делается `port binding`, виртуальная файловая система

### Сравнение Docker и VM

Docker в сравнении с вирутальными машинами проще для разработки, для разворачивания, а также имеется репозиторий контейнеров

#### Рассмотрим работу Docker и VM на уровне ОС. 

+ Основные уровни  ОС Linux состоят из:  1. Applications, 2. OS Kernel, 3. Hardware

+ Docker работает на прикладном уровне, работают не на OS Kernel, а на уровне application.

+ VM - работает как на OS Kernel так и на application, полноценная операционная система, в отличии от Docker

+ Docker - занимает меньше, запускаются быстрее, 

+ VM - запускается на любой ОС, тк он сам в себе содержит ядро

### Описание команд Docker

`docker pull`

`docker run -d` (detach, запуск в автономном режиме) {id} (создание нового контейнера из образа) --name {name} -p {from:to} -e {name}={value}(env var) --net {name}


`docker start {id}` -  не работает с образами, а запускает имеющиюеся контейнеры
 
`docker stop {id}` - остановка имеющегося контейнера

`docker ps` - все запущенные контейнеры

`docker logs` - логи контейнера

`docker exec -it`

`docker ps` -a все запущеные и не запущеные контейнеры

`docker run -p6000:6379 {image}`
`docker run -p{from}:{to} {image}`

`docker images` - показывает все ваши образы

`docker network ls` - выводит список сетей

`docker network create {name}` - создает новую сеть

`docker rmi {IMAGE}` - удаляет образ

`docker rm {container}` - удаляет контейнер

Container port vs HOST port

Можно запускать несколько контейнеров на одном порту, но для этого нужно с хост-машины редиректить запросы через порт в нужный контейнер. Т.е. 10 контейнеров на 5000 порту, но на машине мы отправляем их в 5001/5000, 5002/5000, 5003/5000

Docker network
создает свою изолированную сеть, в которой контейнеры могут общаться по названию контейнера

docker-compose up 
docker-compose down
docker-compose start
docker-compose stop

docker volumes:
	host volume:
		docker run -v {path host}:{path container}
	anonymous volume:
		docker run -v {path container}
	name volume -- for production
		docker run -v {name}:{path container}

для того чтобы бд не забывала данные:

volumes: 
	- mongo-data:/data/db
	- mysql-data:/var/lib/mysql
	- postgresql-data:/var/lib/postgresql


default mongo db path: /data/db

	volumes:
		mongo-data
			driver: local

default docker volumes path:
	/var/lib/docker/volumes

### Docker tips
  
**1.** Используйте официальные образы, а не собирайте с нуля

**2.** Используйте теги. Не пульте всегда последнюю версию

+ Чтобы точно понимать, какую версия будет использоваться `(пр. nginx -> nginx:1.23)`

**3.** Чаще используйте alpine версию контейнера

+ Такие контейнеры содержат только **необходимые** компоненты для запуска контейнера, меньше весят, стабильнее работают `(пр. nginx -> nginx:alpine-1.23)`

**4.** Оптимизируйте кеширование слоев в dockerfile

+ Если один слой кеша не отвалился - последующие тоже отвалятся и будут заного создаваться.

+ <i>Ставьте слои, которые меняются толкьо в последнюю очередь<i>

*Пример:*
```yml
FROM node:alpine
WORKID /app
COPY myapp /app ---------------------- стоит поменять местами 
RUN npm install --production ---------
CMD ["node", "src/index.js"]
```
Если изменим файлы в директории `myapp`, то `RUN npm install` отвалится из кеша, поэтому меняем их местами

```yml
FROM node:alpine
WORKID /app
RUN npm install --production --------- поменяли
COPY myapp /app ----------------------
CMD ["node", "src/index.js"]
```
Теперь при копировании файлов из директории `/myapp` не будет инвалидироваться кеш `RUN npm install` и только один слой будет заного собран

**5.** Используйте `.dockerignore` файл для исключения ненужных файлов из контейнера

*Пример:*
```dockerignore
.git
.cache
*.md
```

**6.** Исключайте сборку в контейнере, используйте мультистейдж для создания контейнеров

+ В таком случае, в контейнере будет только необходимые файлы, без исходников

*Пример:*
```yml
#build stage
FROM maven as build
WORKDIR /app
COPY myapp /app
RUN mvn package

# Run stage
FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/..
..etc
```
**7.** Используйте юзеров для использования контейнера.

+ Добавляйте юзера и группу для контейнеров, чтобы под рутом все не случайно поломали

*Пример:*
```yml
RUN groupadd -r tom && useradd -g tom tom
RUN chown -R tom:tom /app
USER tom
CMD node index.js
```
**8.**  Используйте `docker scan` для ваших контейнеров для проверки на уязвимости
