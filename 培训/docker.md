# Docker

##[理论]Microservices

## [工具]Docker
### 1.Container
- OS virtualization
- Resource isolation
- cgroups
- 
- Isolation,Boundary
- 

文件管理分层

镜像：镜像跑出来就是容器


交叉编译



Dockerfile
- FROM ...
- RUN ...
- ADD ...
- WORKDIR
- CMD
- 

(mac 系统下无法直接运行，跑在linux虚拟机上)

curl ？

docker toolbox


## Solution for Deploy


## Assignments
- 阅读一些微服务相关的博客、文章
- Docker -build & run your program
- QCOS deploy your service
- 


docker login index.qiniu.com

docker pull index.qiniu.com/nginx



## 命令：
```
echo $DOCKER_HOST
 1421  docker-machine
 1422  docker-machine ls
 1423  bash -c "clear && DOCKER_HOST=tcp://192.168.99.100:2376 DOCKER_CERT_PATH=/Users/taotao/.docker/machine/machines/default DOCKER_TLS_VERIFY=1 /bin/zsh"
 1424  ls
 1425  docker ps -a
 1426  docker-machine
 1427  ifconfig
 1428  ping 192.168.99.100
 1429  docker-machine ls
 1430  docker-ssh
 1431  docker ssh default
 1432  docker-machine ssh default
 1433  docker ps -a
 1434  docker login index.qiniu.com
 1435  docker pull index.qiniu.com/nginx
 1436  docker-machine ssh sudo udhcpc
 1437  docker-machine ssh default sudo udhcpc
 1438  docker pull index.qiniu.com/nginx
 1439* docker build
 1440  docker images
 1441  docker run --help
 1442  docker run -it --name=nginx -p 80:80 index.qiniu.com/nginx bash
 1443  docker ps
 1444  docker ps -a
 1445  docker start nginx
 1446  docker ps -a
 1447  curl 192.168.99.100:80
 1448  docker rm -f nginx
 1449  docker run -it --name=nginx -p 80:80 index.qiniu.com/nginx
 1450* curl 192.168.99.100:80
 1451  docker ps -a
 1452  docker rm nginx
 1453  docker run -idt --name=nginx -p 80:80 index.qiniu.com/nginx
 1454  docker ps -a
 1455  docker exec
 1456  docker exec -help
 1457  docker exec nginx "ps -a"
 1458  docker exec efd1d556903c "ps -a"
 1459  docker exec efd1d556903c "echo 111"
 1460  docker exec efd1d556903c bash
 1461  echo
 1462  docker exec -it efd1d556903c bash
 1463  docker exec -it efd1d556903c echo 111
 1464  docker ps -a
 1465  docker rm -f
 1466  docker rm -f nginx
 1467  docker ps -a
 1468  docker images
 1469  ls
 1470  mkdir docker-test
 1471  ls
 1472  cd docker-test
 1473  ls
 1474  docker run -it --name=nginx -p 80:80 index.qiniu.com/nginx
 1475  docker run -idt --name=nginx -p 80:80 index.qiniu.com/nginx
 1476  docker rm -f nginx
 1477  docker run -idt --name=nginx -p 80:80 index.qiniu.com/nginx
 1478* curl 192.168.99.100:80
 1479  docker exec -it nginx bash
 1480  docker inspect nginx
 1481  docker exec -it nginx bash
 1482  vim index.html
 1483  ls
 1484  docker exec -it nginx bash
 1485  docker commit nginx
 1486  docker ps -a
 1487  docker images
 1488  docker commit --help
 1489  docker ps -a
 1490  docker commit 3f3a4988b5ee index.qiniu.com/jiang1988tao1/mynginx:0.1
 1491  docker images
 1492  docker rmi 8a75c874581a
 1493  docker images
 1494  docker run -idt -p 8080:80 index.qiniu.com/jiang1988tao1/mynginx:0.1
 1495  ls
 1496  subl Dockerfile
 1497  vim Dockerfile
 1498  docker build index.qiniu.com/jiang1988tao1/mynginx:0.2 .
 1499  docker build -t index.qiniu.com/jiang1988tao1/mynginx:0.2 .
 1500  docker images
 1501  docker run -idt -p 8000:80 index.qiniu.com/jiang1988tao1/mynginx:0.2
 1502  ls
 1503  touch nginx.conf
 ```