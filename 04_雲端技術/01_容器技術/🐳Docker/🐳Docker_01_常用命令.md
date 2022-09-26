# Docker常用命令
## 🐳幫助、啟動類命令
- `systemctl <start|stop|restart|status|enable> docker`，操作docker服務，詳細可參考：[[🐧Linux_RH124_10_服務管理#🐧操作服務 systemctl]]
- `docker info`，查看docker概要信息
	```bash
	[mickey@vm102 ~]$ sudo docker info
	Client:
	 Context:    default
	 Debug Mode: false
	 Plugins:
	  app: Docker App (Docker Inc., v0.9.1-beta3)
	  buildx: Build with BuildKit (Docker Inc., v0.6.3-docker)
	  compose: Docker Compose (Docker Inc., v2.10.2)
	  scan: Docker Scan (Docker Inc., v0.17.0)
	以下省略
	```
- `docker --help`、`docker <具體命令> --help`，查看幫助文檔

## 🐳鏡像命令
- 用法和[[🐧Linux_RH134_10_Container管理#操作container]]類似
- `docker images`，查看已下載的image內容
	- `-a`，列出含歷史所有image
	- REPOSITORY，鏡像倉庫源
	- TAG，鏡像版本號
	- IMAGE ID，鏡像ID
	- CREATED，鏡像創建時間
	- SIZE，鏡像大小
	```bash
	[mickey@vm102 ~]$ sudo docker images
	REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
	hello-world   latest    feb5d9fea6a5   11 months ago   13.3kB
	```
- `docker search <image_name>`
	- NAME，鏡像名稱
	- DESCRIPTION，鏡像說明
	- STARS，點贊數
	- OFFICIAL，是否是官方的
	- AUTOMATED，是否是自動構建的
- `docker pull <image_name>:[tag]`，下載鏡像
- `docker rmi <image_name>:[tag]`，刪除本地鏡像文件，也可指定image id
	- `-f`，強制執行
- `docker system df`，查看鏡像/容器/數據卷所占的空間
	```bash
	[mickey@vm102 ~]$ sudo docker system df
	TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
	Images          2         1         116.9MB   116.9MB (99%)
	Containers      1         0         0B        0B
	Local Volumes   0         0         0B        0B
	Build Cache     0         0         0B        0B
	```

## 🐳容器命令
- 用法和[[🐧Linux_RH134_10_Container管理#操作container]]類似
- `docker run [-it|-d] [--name=<指定container名>] <container名> [指令]`，建立並啟動container，若container沒有執行任何進程/交互，則會自動停止
	1. `--name`，給container命名、要給container執行的指令，如果沒有指定，系統則自己指定，不方便管理
	2. `-it`，開啟bash
	3. `-d`，daemon，背景執行
	4. `--rm`，container執行完成後就會刪除
	5. `-p <實體機port>:<container port>`，指定端口映射，將實體機port轉移至container port
	6. `-P`，隨機端口映射
	7.  `-v <實體機目錄>:<container目錄>:Z`，實體機器的目錄mount至container的目錄；可避開[[🐳Docker_02_鏡像]]的聯合文件系統，達到數據持久化
		 - `Z`表示使SELinux的type一致，一般都會使用
		 - 若無特別指定讀寫，默認為`-v <實體機目錄>:<container目錄>:rw`，表示可讀可寫
		 - `-v <實體機目錄>:<container目錄>:ro`，表示實器實體內部只讀、不可寫
	8. `--privileged=true`，開放實體機器目錄操作root權限
	9. `--volumes-from <父容器名>`，繼承指定容器，延序父容器的設定參數
	10. 退出容器方式
		- `exit`，退出，並且容器停止
		- Ctrl + P + Q，退出，但容器不停止
- `docker attach [-it|-d] <container名稱|container ID> <指令>`，在執行中的container再執行指定指令；不會啟動新進程，`exit`退出導致容器停止
- `docker exec [-it|-d] <container名稱|container ID> <指令>`，在執行中的container再執行指定指令；啟動新進程，`exit`退出不會導致容器停止【一般使用】
	1. `-l`，指最後一次操作的container 
- `docker ps`，查看正在執行的container狀態
	1. `-a`，查看所有container(無論是否執行中)
	2. `-l`，顯示最新創建的容器
	3. `-n`，顯示最近n個創建的容器
	4. `-q`，只顯示容器編號
- `docker start <container名稱|container ID>`，啟動container
- `docker stop <container名稱|container ID>`，關閉container，container還會存在
	1. `-f`，強制執行
- `docker kill <container名稱|container ID>`，強制停止container，和`kill -9`類似
- `docker rm <container名稱|container ID>`，刪除container
- `docker logs <container名稱|container ID>`，查看container log
- `docker top <container名稱|container ID>`，查看container內進程
- `docker inspect <image名稱|container名稱|container ID>`，查看continer內部細節
- `docker cp <container ID>:<文件路徑> <主機路徑>`，將container內文件複製至主機
- `docker export <container ID>`，導出container成tar文件，類似於全備份
	```bash
	[mickey@vm102 ~]$ sudo docker export redis_test > test.tar
	[mickey@vm102 ~]$ ls
	Desktop  Documents  Downloads  Music  Pictures  Public  Templates  test.tar  Videos
	```
- `cat ./<tar_path> | docker import - <image名稱>:<image_tag>`，將tar文件導入為鏡像
	```bash
	[mickey@vm102 ~]$ sudo docker ps -a
	CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
	[mickey@vm102 ~]$ sudo docker images
	REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
	redis         latest    2460522297a1   2 days ago      117MB
	hello-world   latest    feb5d9fea6a5   11 months ago   13.3kB
	[mickey@vm102 ~]$ cat ./redis_test.tar | sudo docker import - mickey/redis_test:1.0
	sha256:96cdb83e2cddb598d1e657efe6700350d9504bcc195bd6119d1fc87b0601b687
	[mickey@vm102 ~]$ sudo docker images
	REPOSITORY          TAG       IMAGE ID       CREATED         SIZE
	mickey/redis_test   1.0       96cdb83e2cdd   3 seconds ago   113MB
	redis               latest    2460522297a1   2 days ago      117MB
	hello-world         latest    feb5d9fea6a5   11 months ago   13.3kB
	```
