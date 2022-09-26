# Docker_03_Dockerfile
- Dockerfile是用來構建Docker鏡像的文本文件，是由一條條構建鏡像所需的指令和參數構成的腳本，類似於配置文件
- 在Docker Hub上的鏡像文件就是Dockerfile
- 官方文件：[](https://docs.docker.com/engine/reference/builder/)
- 基本規則
	1. 每條指令的保留字必須為大寫，且後面至少有一個參數
	2. 指令從上到下執行
	3. `#`表示注釋
	4. 每條指令都會創建新鏡像層，並提交
- Docker執行Dockerfile大致流程
	1. Docker從基礎鏡像運行一個容器
	2. 對容器進行修改
	3. 容器修改完成後，執行類似`docker commit`提交新的鏡像層
	4. Docker用新的鏡像層運行一個新容器

## 🐳Dockerfile構建流程
1. 編寫Dockerfile，注意檔案名稱必須一樣
	```bash
	[mickey@vm102 testDockerfile]$ cat ./Dockerfile
	FROM ubuntu
	CMD echo'success...'
	```
2. 在Dockerfile當前目錄執行`docker build -t <鏡像名:tag> <Dockerfile_path>`，構建鏡像
	```bash
	[mickey@vm102 testDockerfile]$ ls
	Dockerfile
	[mickey@vm102 testDockerfile]$ sudo docker build -t test:1.0.0 .
	Sending build context to Docker daemon  2.048kB
	Step 1/2 : FROM ubuntu
	 ---> 2dc39ba059dc
	Step 2/2 : CMD echo'success...'
	 ---> Using cache
	 ---> ab0f23923787
	Successfully built ab0f23923787
	Successfully tagged test:1.0.0
	[mickey@vm102 testDockerfile]$ sudo docker images
	REPOSITORY                           TAG       IMAGE ID       CREATED         SIZE
	test                                 1.0.0     ab0f23923787   4 minutes ago   77.8MB
	```

## 🐳常用保留字指令
- `FROM <基礎鏡像名>`，指定已存在的鏡像為模版
- `MAINTAINER <name>`，鏡像維護者名稱
- `RUN`，在==容器構建時==需要執行的命令
	1. `RUN <shell命令>`，在容器操作的shell命令，如：`RUN yum -y install vim`
	2. `RUN ["<可執行文件>", "<參數1>"[, "參數2"...]]`，在容器執行可執行文件，如：`RUN [".test.php", "dev", "offline"]`等同於`.test.php dev offline`
- `EXPOSE <port>`，當前容器對外暴露的端口
- `WORKDIR <path>`，指定創建容器後，終端默認登陸的工作目錄
- `USER <user_name>`，指定該境像用什麼用戶執行，默認為root，一般不會指定
- `ENV <變量名> <變量值>`，用於構建鏡像過程中設置環境變量，可在後續指令使用，如：`WORKDIR $MY_PATH`
- `VOLUME <path>`，掛載容器數據卷，相當於`-v`，Docker會自動生成本機的掛載點路徑，可以用`docker inspect`查看，參考：[docker学习笔记18：Dockerfile 指令 VOLUME 介绍](https://www.cnblogs.com/51kata/p/5266626.html)
- `ADD <host_src> [host_src...] <container_dest>`，將主機目錄文件拷貝至鏡像，且會自動處理URL、解壓tar
- `COPY <host_src> [host_src...] <container_dest>`，將主機目錄文件拷貝至鏡像
- `CMD`，==建立容器時==要執行的命令，也可以作為`ENTRYPOINT`的參數
	1. `CMD <shell命令>`
	2. `CMD ["<可執行文件>", "<參數1>"[, "參數2"...]]`
	3. 注意，Dockerfile可有多個`CMD`，但==只有最後一個生效==，而`CMD`會被`docker run`後的指令替換。如：Tomcat Dockerfile`CMD ["catalina.sh", "run"]`表示啟動Tomcat，若用`docker run -it -p 8080:8080 tomcat /bin/bash`相當於執行`CMD /bin/bash`，則不執行Dockerfile的指令，因此Tomcat不會啟動
- `ENTRYPOINT ["<executable>", "<參數1>"[, "參數2"...]"]`，類似於`CMD`，但不會被`docker run`後的指令替換
	1. `ENTRYPOINT`可和`CMD`一起使用，相當於把`CMD`作為參數傳給`ENTRYPOINT`
		```bash
		FROM nginx
		ENTRYPOINT ["nginx", "-c"] # 定參
		CMD ["/etc/nginx/nginx.conf"] # 變參
		```
	2. 若建立容器時不用指令替換`CMD`，如：`docker run nginx:test`，則`ENTRYPOINT`會執行`nginx -c /etc/nginx/nginx.conf`
	3. 若建立容器時有用指令替換`CMD`，如：`docker run nginx:test -c /etc/nginx/new.conf`，則`ENTRYPOINT`會執行`nginx -c /etc/nginx/new.conf`

## 🐳將SpringBoot佈署至Docker
1. 參考[[🍃SpringBoot_00_Overview]]，用[[🧾Maven]]産出jar
2. 編寫Dockerfile
	```bash
	FROM openjdk:8u121
	MAINTAINER mickey
	VOLUME /tmp
	ADD ./springboot-test-0.0.1-SNAPSHOT.jar test_docker.jar
	RUN bash -c 'touch /test_docker.jar'
	ENTRYPOINT ["java","-jar","/test_docker.jar"]
	EXPOSE 8081
	```
3. 將jar檔放至和Dockerfile同層目錄
	```bash
	[mickey@vm102 springboot]$ tree
	.
	├── Dockerfile
	└── springboot-test-0.0.1-SNAPSHOT.jar
	```
4. `docker build -t springboottest:1.0 .`，構建Docker image
5. `docker run -d --name myboot -p 8081:8081 springboottest:1.0`，構建容器
6. 測試[http://192.168.56.102:8081/test/uuid]()
