#📆2022年 
狀態:: #☑DONE 
完成日期:: 2022-09-26
標籤::  #💻編程/🐧Linux #🐳容器技術 #🗂Overview 
子筆記:: [[🐳Docker_01_常用命令]],[[🐳Docker_02_鏡像]],[[🐳Docker_03_Dockerfile]],[[🐳Docker_04_Docker Network]],[[🐳Docker_05_Docker Compose]],[[🐳Docker_06_可視化工具、監控工具]]
教程:: [尚硅谷](https://www.bilibili.com/video/BV1gr4y1U7CY)
備註:: 

# 🐳Docker_00_Overview
- Docker官網：[https://www.docker.com/]()
- Docker倉庫：[https://hub.docker.com/]()
- 簡介
	- Docker是基於Go語言並遵從Apache2.0協議的開源應用容器(Container)引擎，是為了解決運行環境、配置問題的軟件容器。
	- Docker是對Linux 容器([[🐧Linux_RH134_10_Container管理#🐧lxc]])等技術進行封裝，因此Docker必須部署在版本3.8以上的Linux內核
		```bash
		[mickey@vm100 ~]$ uname -r
		4.18.0-240.el8.x86_64
		```
- 相關架構可參考：[[🐧Linux_RH134_10_Container管理#架構]]
- 使用Linux管理容器工具可參考-->[[🐧Linux_RH134_10_Container管理#🐧podman]]
- 術語：
	![Docker_00_Overview_01_Docker簡單架構](https://github.com/MickeyHuang233/CodingStudyNote/blob/main/04_%E9%9B%B2%E7%AB%AF%E6%8A%80%E8%A1%93/01_%E5%AE%B9%E5%99%A8%E6%8A%80%E8%A1%93/%F0%9F%90%B3Docker/images/Docker_00_Overview_01_Docker%E7%B0%A1%E5%96%AE%E6%9E%B6%E6%A7%8B.png?raw=true)
	- 鏡像(Images)，用於創建Container的模板
	- 容器(Container)，獨立運行的應用
	- 客戶端(Client)，通過命令行或其他工具使用Container
	- 主機(Host)，運行Container物理機器或虛擬機
	- 倉庫(Registry)，統一保存鏡像的地方，[Docker Registry](https://hub.docker.com/)

## 🐳環境安裝
- 官網安裝流程：[Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/)
- 安裝流程參考：[CentOS Docker 安装 - 菜鳥教程](https://www.runoob.com/docker/centos-docker-install.html)
- 可能會需要卸載podman，`sudo yum remove podman buildah`


