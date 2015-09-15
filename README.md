# 前言

感謝各位參加Hadoop Conference 2015
為了方便各位實作，將協助各位使用docker在本機上建立spark教學環境．
較熟悉Docker的朋友可以直接跳過這段．

# Docker安裝教學

- Linux環境餐請參考：(https://docs.docker.com/linux/step_one/)
- Mac環境請參考：(https://docs.docker.com/installation/mac/)

# 建立Docker Images

- 請確認您的電腦上已經安裝git
- 進入任意合適的目錄
- `git clone https://github.com/bryanyang0528/ipython-pyspark-cineca.git`
- `cd ipython-pyspark-cineca`
- `docker build .`  此步驟將會開始建立docker images
- `docker images`   確認新建立的images id (一個英數組合)
- `docker tag <images id> pyspark:1.4.1`
- `docker run -d -p 8888:8888 -p 4040:4040 --name pyspark pyspark:1.4.1`

# 進入ipython

- linux: 直接在瀏覽器輸入`http://localhost:8888` , Spark的UI在`http://localhost:4040`
- Mac: 請先在terminal中輸入 `boot2docker ip` 確認ip位置，再到瀏覽器中輸入`http://<boot2docker ip>:8888`

# docker-spark-ipython

To run a container with a shared folder (e.g. ~/Desktop/localFolder), listening on the port 8888.
The *localFolder* is located on the desktop and you can use it to share file with the virtual machines, ipython notebook included.

```
docker run -d -p 8888:8888 -p 4040:4040 --name pyspark cineca/spark-ipython:1.4.1
```

- `-d` detached mode
- `-p` port
- `-v` volume
- `--name` give a name to the containers

### Launch the notebook
Open the browser at [localhost:8888](localhost:8888)

### Launch the Spark web UI
Open the browser at [localhost:8080](localhost:8888)

### On Mac OSX
On Mac, remember that the actual VB ip can be finded with `boot2docker ip`.
While, if you want connect to the `localhost` you need the following port forwarding for VBox:

(e.g. ports from 8880 to 8890)
```
for i in {8880..8890}; do
VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port$i,tcp,,$i,,$i";
VBoxManage modifyvm "boot2docker-vm" --natpf1 "udp-port$i,udp,,$i,,$i";
done
VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port8080,tcp,,8080,,8080"
```

To get info about the virtual machine where the containers run:
```
boot2docker info
```
To change the memory of the VirtualMachine (i.e. VBox)
```
BoxManage modifyvm boot2docker-vm --memory 4096
```

### Manage containers
- `docker ps` shows acrive conainers
- `docker ps -a` shows all containers
- `docker restart CONTAINER-ID` restarts a container
- `docker stop 'docker ps -aq'` stops all containers
- `docker rm 'docker ps -aq'` removes all conainers

### IPython and PySpark
Launching the container the first command issued is:
```
IPYTHON_OPTS="notebook --no-browser --ip=0.0.0.0 --port 8888" /usr/local/spark/bin/pyspark cineca/pyspark-ipython
```
The IPython notebook will already have the *sparkContext* variable `sc`.
Write `sc.version` to see what verison is loaded.


### Hadoop environment
To read a file directly from the disk (no HDFS), 
use explicitly:
```
sc.textFile("file:///absolute_path to the file/")
```

SparkContext.textFile internally calls org.apache.hadoop.mapred.FileInputFormat.getSplits, which in turn uses org.apache.hadoop.fs.getDefaultUri if schema is absent.
This method reads "fs.defaultFS" parameter of Hadoop conf.

If you set HADOOP_CONF_DIR environment variable, the parameter is usually set as "hdfs://...";
otherwise "file://".
