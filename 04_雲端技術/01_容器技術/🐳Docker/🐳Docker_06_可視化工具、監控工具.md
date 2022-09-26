# Docker_06_可視化工具、監控工具
## 🐳可視化工具Portainer
- Portainer提供圖形化界面，更於管理Docker單機、集群環境
- 官網：[https://www.portainer.io/]()
- 環境安裝
	1. 建立容器
		- `--restart=always`，表示docker.service重啟時，此容器也會跟著重啟，永遠為啟動狀態
	```bash
	docker run -d --name=portainer \
	-p 8000:8000 -p 9000:9000 \
	--restart=always \
	-v /var/run/docker.sock:/var/run/docker.sock \
	-v portainer_data:/data \
	portainer/portainer-ce
	```
	2. 訪問[http://192.168.56.102:9000/]()
		![Docker_06_可視化工具、監控工具_01_建立帳號](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/04_%E9%9B%B2%E7%AB%AF%E6%8A%80%E8%A1%93/01_%E5%AE%B9%E5%99%A8%E6%8A%80%E8%A1%93/%F0%9F%90%B3Docker/images/Docker_06_%E5%8F%AF%E8%A6%96%E5%8C%96%E5%B7%A5%E5%85%B7%E3%80%81%E7%9B%A3%E6%8E%A7%E5%B7%A5%E5%85%B7_01_%E5%BB%BA%E7%AB%8B%E5%B8%B3%E8%99%9F.png?raw=true)

## 🐳監控工具CIG
- 使用`docker stats`指令，會有以下不足：
	1. 只能查看主機的全部容器
	2. 數據無法持久化
	3. 沒有容器狀態分析
	```bash
	CONTAINER ID   NAME       CPU %     MEM USAGE / LIMIT     MEM %     NET I/O       BLOCK I/O         PIDS
	e0dc4e590fbb   myboot     0.11%     124.9MiB / 1.775GiB   6.87%     2.09kB / 0B   7.85MB / 0B       27
	6ff053575663   my_db      0.01%     95.98MiB / 1.775GiB   5.28%     2.2kB / 0B    6.64MB / 29.7kB   10
	e80dddf0cf60   my_redis   0.15%     9.82MiB / 1.775GiB    0.54%     2.38kB / 0B   4.43MB / 0B       5
	```
- CIG容器監控系統：
	```mermaid
	graph LR 
	CAdvisor-->InfluxDB-->Granfana
	```
	1. CAdvisor，監控數據收集，數據僅保存2分鐘
	2. InfluxDB，監控數據存儲
	3. Granfana，監控數據可視化、警報

### 環境配置
1. 編寫docker-compose.yml
```yaml
# /home/mickey/Documents/cig/docker-compose.yml
version: "3"

volumes:
  grafana_data: { }

services:
  influxdb:
    image: tutum/influxdb:0.9
    restart: always
    environment:
      - PRE_CREATE_DB=cadvisor
    ports:
      - "8083:8083"
      - "8086:8086"
    volumes:
      - ./data/influxdb:/data

  cadvisor:
    image: google/cadvisor
    links:
      - influxdb:influxsrv
    command: -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influxsrv:8086
    restart: always
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys/fs/cgroup/cpu,cpuacct:/sys/fs/cgroup/cpuacct,cpu
      - /var/lib/docker/:/var/lib/docker:ro
  grafana:
    user: "104"
    image: grafana/grafana
    user: "104"
    restart: always
    links:
      - influxdb:influxsrv
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - HTTP_USER=admin
      - HTTP_PASS=admin
      - INFLUXDB_HOST=influxsrv
      - INFLUXDB_PORT=8086
      - INFLUXDB_NAME=cadvisor
      - INFLUXDB_USER=root
      - INFLUXDB_PASS=root
```
2. `docker compose config -q`，檢查格式
3. `docker compose up -d`，運行CIG
4. 測試
	- CAdvisor，[http://192.168.56.102:8080/]()
	- InfluxDB，[http://192.168.56.102:8083/]()
	- Granfana，[http://192.168.56.102:3000/]()
5. Granfana配置
	1. 數據源，Configuration-->Data sources-->Add data source-->InfluxDB
		```console
		Name = InfluxDB
		URL = http://InfluxDB:8086
		Database = cadvisor
		User = root
		Password = root
		```
	2. 監控業務面版，Dashboard-->New dashboard-->Add a new panel
		![Docker_06_可視化工具、監控工具_01_Granfana New Dashboard](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/04_%E9%9B%B2%E7%AB%AF%E6%8A%80%E8%A1%93/01_%E5%AE%B9%E5%99%A8%E6%8A%80%E8%A1%93/%F0%9F%90%B3Docker/images/Docker_06_%E5%8F%AF%E8%A6%96%E5%8C%96%E5%B7%A5%E5%85%B7%E3%80%81%E7%9B%A3%E6%8E%A7%E5%B7%A5%E5%85%B7_02_Granfana%20New%20Dashboard.png?raw=true)
