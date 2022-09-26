# Docker_05_Docker Compose
- Docker Compose是Docker官方的開源項目，通過[[🍀YAML]]配置容器間的調用關系，實現對Docker容器集群的快速編排
- Docker Compose允許通過單獨的docker-compose.yml模板文件定義一組關聯的應用容器為一個項目(project)
- 官網下載：[Install Docker Compose](https://docs.docker.com/compose/install/)
	```bash
	yum install docker-compose-plugin
	# 查看版本
	docker compose version
	# Install the plugin manually
	DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
	mkdir -p $DOCKER_CONFIG/cli-plugins
	curl -SL https://github.com/docker/compose/releases/download/v2.11.1/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
	chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
	# 查看版本
	docker compose version
	```

## 🐳常用指令
- `docker compose ls`，查看運行的docker compose
- `docker compose config`，檢查docker-compose.yml格式
	- `-q`，有問題才輸出
- `docker compose up`，啟動docker compose服務
	- `-d`，在後台運行
- `docker compose down`，關閉docker compose服務，并刺除容器、網絡、卷、鏡像
- `docker compose exec <配置文件中的服務ID> <指令>`，指定在執行中的docker compose容器實例再執行指令
- `docker compose top`，查看運行的docker compose容器進程
- `docker compose logs <配置文件中的服務ID>`，查看指定容器的輸出日志
- `docker compose start`，啟動服務
- `docker compose restart`，重啟服務
- `docker compose stop`，關閉服務

## 執行步驟
1. 編寫docker-compose.yml
	```yaml
	# docker compose版本
	version: "3.6"
	
	# 服務容器實例
	services:
	  # 自定義容器名稱，只要不重覆即可
	  # docker run -d --name myboot -p 8081:8081 --network my_net springboottest:1.1
	  mainService:
	    image: springboottest:1.1
	    container_name: myboot
	    ports:
	      - "8081:8081"
	    networks:
	      - my_net
	    # 表示此容器依賴於哪些容器，也就是說被依賴的容器要先啟動，此容器才可啟動
	    depends_on:
	      - redis
	      - mysql
	
	  # docker run -d --name my_redis -p 6379:6379 \
	  # --privileged=true --network my_net \
	  # -v /home/mickey/Documents/redis/redis.conf:/etc/redis/redis.conf:Z \
	  # -v /home/mickey/Documents/redis/data:/data:Z \
	  # docker.io/library/redis:latest \
	  # redis-server /etc/redis/redis.conf
	  redis:
	    image: redis:latest
	    container_name: my_redis
	    ports:
	      - "6379:6379"
	    networks:
	      - my_net
	    privileged: true
	    volumes:
	      - "/home/mickey/Documents/redis/redis.conf:/etc/redis/redis.conf:Z"
	      - "/home/mickey/Documents/redis/data:/data:Z"
	    command: redis-server /etc/redis/redis.conf
	
	  # docker run -d --name=my_db -p 3306:3306 \
	  # --network my_net \
	  # -e MARIADB_USER=mickey \
	  # -e MARIADB_PASSWORD=123456 \
	  # -e MARIADB_ROOT_PASSWORD=root \
	  # -v /home/mickey/Documents/my_db/data:/var/lib/mysql \
	  # mariadb:10.8.2 --port 3306
	  mysql:
	    image: mariadb:10.8.2
	    container_name: my_db
	    ports:
	      - "3306:3306"
	    networks:
	      - my_net
	    environment:
	      - MARIADB_USER=mickey
	      - MARIADB_DATABASE=testdb
	      - MARIADB_PASSWORD=123456
	      - MARIADB_ROOT_PASSWORD=root
	    privileged: true
	    volumes:
	      - "/home/mickey/Documents/my_db/data:/var/lib/mysql:Z"
	    command: --port 3306
	
	# docker network create my_net
	networks:
	  my_net:
	```
2. 將ip改為服務名，避免日後ip變動出現錯誤
```properties
# DB
spring.datasource.url = jdbc:mysql://my_db:3306/testdb?verifyServerCertificate=false&useSSL=false&useUnicode=true&characterEncoding=UTF-8&allowPublicKeyRetrieval=true&autoReconnect=true
# Redis
spring.redis.host = my_redis
```
3. 處理[[🐳Docker_03_Dockerfile]]，構建鏡像
4. 檢查docker-compose.yml格式，無輸出表示沒錯誤
	注意，需要在docker-compose.yml同層目錄執行
	```bash
	[mickey@vm102 simple-springboot]$ ls
	docker-compose.yml  Dockerfile  simple-springboot-test-0.0.1-SNAPSHOT.jar
	[mickey@vm102 simple-springboot]$ sudo docker compose config -q
	```
5. `docker compose up`，啟動docker compose服務，並啟動docker-compose.yml中的容器
	```bash
	[mickey@vm102 simple-springboot]$ ls
	docker-compose.yml  Dockerfile  simple-springboot-test-0.0.1-SNAPSHOT.jar
	[mickey@vm102 simple-springboot]$ sudo docker ps
	CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
	[mickey@vm102 simple-springboot]$ sudo docker compose up -d
	[+] Running 3/3
	 ⠿ Container my_redis  Started                                                                                  1.1s
	 ⠿ Container my_db     Started                                                                                  1.0s
	 ⠿ Container myboot    Started                                                                                  1.9s
	```
