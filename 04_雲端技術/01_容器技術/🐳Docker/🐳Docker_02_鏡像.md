# Docker_02_鏡像
- 在`docker pull`時可以看到鏡像是一層一層向下下載的，因為鏡像是使用UnionFS實現的
	```bash
	[mickey@vm102 ~]$ sudo docker pull tomcat
	Using default tag: latest
	latest: Pulling from library/tomcat
	2b55860d4c66: Downloading [======>                                            ]  3.742MB/30.43MB
	49a58ffb4a94: Downloading [========>                                          ]   3.05MB/16.99MB
	8889343dc9d4: Downloading [=>                                                 ]  4.307MB/192.1MB
	5c321d92dfdb: Waiting
	65e12e19b4c9: Waiting
	31c5670ba66a: Waiting
	4196dee71f9b: Waiting
	```
- UnionFS，聯合文件系統，是一種分層、輕量級、高性能、可複用的文件系統，支持對文件系統的修改所為一次提交來一層層疊加(類似git)
	1. bootfs(boot file system)，主要包含bootloader(用於引導加載kernel)、kernel，處於Docker鏡像最底層，啟動時會加載bootfs至內存
	2. rootfs(root file system)，就是各種不同操作系統發行版，包含標準目錄、文件，處於bootfs之上
- Docker中，鏡像層是只讀，只有容器層是可寫
	![Docker_02_鏡像_01_鏡像分層簡易架構圖](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/04_%E9%9B%B2%E7%AB%AF%E6%8A%80%E8%A1%93/01_%E5%AE%B9%E5%99%A8%E6%8A%80%E8%A1%93/%F0%9F%90%B3Docker/images/Docker_02_%E9%8F%A1%E5%83%8F_01_%E9%8F%A1%E5%83%8F%E5%88%86%E5%B1%A4%E7%B0%A1%E6%98%93%E6%9E%B6%E6%A7%8B%E5%9C%96.png?raw=true)

## 🐳提交鏡像
1. 下載鏡像、建立container
2. 自行安裝所需插件
3. `docker commit -m=<提交信息> -a=<作者> <container ID> <鏡像名>:<tag>`
	```bash
	[mickey@vm102 ~]$ sudo docker commit -m='add vim' -a='mickey' e09516cd89f7 mickey/myubuntu:1.0.1
	sha256:4f322abd54d4ac057df8135368d17f25d0609c0ae1a6a00bf93d1e2befecd659
	[mickey@vm102 ~]$ sudo docker images
	REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
	mickey/myubuntu   1.0.1     4f322abd54d4   9 seconds ago   189MB
	ubuntu            latest    2dc39ba059dc   2 weeks ago     77.8MB
	```

## 🐳發布遠端倉庫
- Docker公有庫：[https://hub.docker.com/]()

### 建立個人倉庫
![Docker_02_鏡像_02_建立個人倉庫](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/04_%E9%9B%B2%E7%AB%AF%E6%8A%80%E8%A1%93/01_%E5%AE%B9%E5%99%A8%E6%8A%80%E8%A1%93/%F0%9F%90%B3Docker/images/Docker_02_%E9%8F%A1%E5%83%8F_02_%E5%BB%BA%E7%AB%8B%E5%80%8B%E4%BA%BA%E5%80%89%E5%BA%AB.png?raw=true)
![Docker_02_鏡像_03_建立個人倉庫](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/04_%E9%9B%B2%E7%AB%AF%E6%8A%80%E8%A1%93/01_%E5%AE%B9%E5%99%A8%E6%8A%80%E8%A1%93/%F0%9F%90%B3Docker/images/Docker_02_%E9%8F%A1%E5%83%8F_03_%E5%BB%BA%E7%AB%8B%E5%80%8B%E4%BA%BA%E5%80%89%E5%BA%AB.png?raw=true)

### 上傳鏡像
1. `docker login`，登入Docker Hub
```bash
[mickey@vm102 ~]$ sudo docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: michongchong233
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```
2. `docker image tag <old_image_name>:<old_tag> <new_image_name>:<new_tag>`，更改鏡像名稱
	- `new_image_name`為`<Docker用戶名>/<Repository名>`
	```bash
	[mickey@vm102 ~]$ sudo docker images
	REPOSITORY        TAG       IMAGE ID       CREATED       SIZE
	mickey/myubuntu   1.0.1     4f322abd54d4   4 hours ago   189MB
	[mickey@vm102 ~]$ sudo docker image tag mickey/myubuntu:1.0.1 michong/myubuntu:1.0.1
	[mickey@vm102 ~]$ sudo docker image tag mickey/myubuntu:1.0.1 michongchong233/my_test_repository:1.0.1
	[mickey@vm102 ~]$ sudo docker images
	REPOSITORY                           TAG       IMAGE ID       CREATED       SIZE
	michongchong233/my_test_repository   1.0.1     4f322abd54d4   4 hours ago   189MB
	mickey/myubuntu                      1.0.1     4f322abd54d4   4 hours ago   189MB
	ubuntu                               latest    2dc39ba059dc   2 weeks ago   77.8MB
	```
3. `docker push <image_name>:<tag>`，上傳鏡像至倉庫
	```bash
	[mickey@vm102 ~]$ sudo docker push michongchong233/my_test_repository:1.0.1
	The push refers to repository [docker.io/michongchong233/my_test_repository]
	ebf09c039d9c: Pushed
	7f5cbd8cc787: Pushed
	1.0.1: digest: sha256:f5a2663db5d1768dc518415a4f35b93ab02942d18cc6e0c2f0f8dc63f0def2d3 size: 741
	```

## 🐳建立個人私有倉庫
1. 相當於私有的Docker Hub，可參考官方文檔：[https://docs.docker.com/registry/]()
2. 準備docker registry容器
	```bash
	# 準備鏡像
	docker pull registry
	# 啟動容器
	sudo docker run -d --name my_registry \
	-p 5000:5000 \
	-v /home/mickey/Documents/registry:/var/lib/registry docker.io/registry
	# 開啟防火墻
	firewall-cmd --permanent --add-port=5000/tcp --zone=public
	firewall-cmd --reload
	```
3. 修改上傳客戶端的Docker安全性設定，`vi /etc/docker/daemon.json`
	若是要在私有庫下載鏡像，也要改設定，否則會有`dial tcp 192.168.56.102:5000: connect: connection refused`錯誤
	```json
	{
	"live-restore": true,
	"group": "dockerroot",
	"insecure-registries": ["192.168.182.134:5000"]
	}
	```
4. 查看庫內所有鏡像，確認私有庫正常連線
	```bash
	[mickey@vm102 ~]$ curl -XGET http://192.168.56.102:5000/v2/_catalog
	{"repositories":[]}
	```
5. 上傳鏡像至私有倉庫
	```bash
	# 更改鏡像名稱
	docker tag mickey/myubuntu:1.0.1 192.168.56.102:5000/myubuntu:1.0.1
	# 推算鏡像
	docker push 192.168.56.102:5000/myubuntu:1.0.1
	# 查看庫內所有鏡像
	curl -XGET http://192.168.56.102:5000/v2/_catalog
	```

## 🐳虛懸鏡像
- 虛懸鏡像(dangling image)，倉庫名、標籤都為`<none>`的鏡像
	```bash
	[mickey@vm102 testDockerfile]$ sudo docker images
	REPOSITORY                           TAG       IMAGE ID       CREATED         SIZE
	<none>                               <none>    ab0f23923787   6 seconds ago   77.8MB
	```
- `docker image ls -f dangling=true`，查看所有虛懸鏡像
	```bash
	[mickey@vm102 testDockerfile]$ sudo docker image ls -f dangling=true
	REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
	<none>       <none>    ee0c0bf043c7   9 seconds ago   77.8MB
	```
- `docker image prune`，刪除所有虛懸鏡像，否則可能會有資安風險
	```bash
	[mickey@vm102 testDockerfile]$ sudo docker image prune
	WARNING! This will remove all dangling images.
	Are you sure you want to continue? [y/N] y
	Deleted Images:
	deleted: sha256:ee0c0bf043c7eb186b7cd327fd3124fcb1666ae85ede02357e7c5ceef4de4820
	
	Total reclaimed space: 0B
	[mickey@vm102 testDockerfile]$ sudo docker image ls -f dangling=true
	REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
	```