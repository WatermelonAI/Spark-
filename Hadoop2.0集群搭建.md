# Hadoop2.0集群搭建

## 需要准备的资料

​	这里我们搭建的Hadoop2.0集群是基于一台windows用虚拟机进行搭建的，一台虚拟机为master，其他两台虚拟机为slave1和slave2。这里需要准备的东西有：

​	1、windows电脑的内存为8G以上，预留分配硬盘空间为60G以上，每个虚拟机给20G的空间

​	2、需要的软件为Centos7 、Vmware14.0，Hadoop-2.6.5，jdk-8u61-linux   

## 虚拟机安装过程



## 虚拟机网络配置



## 免登陆



## 安装JDK



## 安装Hadoop



## Hadoop测试

​	上述过程安装完毕之后我们得正式确认一下集群是否可以启动且工作，我们这里测试一个简单wordcount程序。

### 第一步：关闭每台机器的防火墙功能和selinux

​	我们可以直接在master这台机器上关闭master、slave1和slave2三台虚拟机的防火墙和selinux。

```
[root@master yupeifeng-master]#/etc/init.d/iptables stop
[root@master yupeifeng-master]#setenforce 0

[root@master yupeifeng-master]#ssh slave1 setenforce 0
[root@master yupeifeng-master]#ssh slave1 /etc/init.d/iptables stop

[root@master yupeifeng-master]#ssh slave2 setenforce 0
[root@master yupeifeng-master]#ssh slave2 /etc/init.d/iptables stop
```

### 第二步、开启Hadoop

```
[root@master yupeifeng-master]#/usr/local/src/hadoop-2.6.5/sbin/start-all.sh
```

### 第三步、查看Hadoop进程是否开启

使用jps命令，看到master和slave节点如下进程都已开启说明Hadoop启动成功

**master节点**

```
[root@master yupeifeng-master]# jps
8704 SecondaryNameNode
9108 Jps
8854 ResourceManager
8525 NameNode
```

**slave1节点**

```
[root@slave1 yupeifeng-salve1]# jps
8473 NodeManager
8638 Jps
8367 DataNode
```

**slave2节点**

```
[root@slave2 yupeifeng-slave2]# jps
8371 DataNode
8684 Jps
8477 NodeManager
```

### 第四步：从本地往HDFS上传文件

```
[root@master mr_wordcount]# hadoop fs -put bible.txt /
18/03/20 07:22:19 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

这是我们可以master和slave的节点上都可以看到在HDFS上都有bible.txt这个文件存在了。

```
[root@master mr_wordcount]# hadoop fs -ls /
18/03/20 07:23:58 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 1 items
-rw-r--r--   3 root supergroup    4467663 2018-03-20 07:22 /bible.txt


[root@slave1 yupeifeng-salve1]# hadoop fs -ls /
18/03/20 07:31:43 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 1 items
-rw-r--r--   3 root supergroup    4467663 2018-03-20 07:22 /bible.txt


[root@slave2 yupeifeng-slave2]# hadoop fs -ls /
18/03/20 07:32:13 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 1 items
-rw-r--r--   3 root supergroup    4467663 2018-03-20 07:22 /bible.txt

```

与此同时，我们在HDFS上创建一个output文件夹用于接受程序运行之后的结果

```
[root@master mr_wordcount]# hadoop fs -mkdir /output
18/03/20 07:27:37 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

### 第五步、创建Map程序

```
#!/usr/bin/python
import sys
for line in sys.stdin:
        ss=line.strip().split(' ')
        for word in ss:
                print '\t'.join([word.strip(), "1"])
```

### 第六步、创建Reduce程序

```
#!/usr/bin/python
import sys
cur_word=None
sum=0
for line in sys.stdin:
        ss=line.strip().split('\t')
        if len(ss) !=2:
                continue
        word=ss[0].strip()
        cnt=ss[1].strip()
        if cur_word==None:
                cur_word=word
        if cur_word != word:
                print '\t'.join([cur_word,str(sum)])
                cur_word=word
                sum=0
        sum +=int(cnt)
print '\t'.join([cur_word,str(cnt)])

```

### 第七步、创建run.sh脚本文件

```
STREAM_JAR_PATH="/usr/local/src/hadoop-2.6.5/share/hadoop/tools/lib/hadoop-streaming-2.6.5.jar"
INPUT_FILE_PATH_1="/bible.txt"
OUTPUT_PATH="/output"
/usr/local/src/hadoop-2.6.5/bin/hadoop fs -rmr $OUTPUT_PATH
/usr/local/src/hadoop-2.6.5/bin/hadoop jar $STREAM_JAR_PATH \
        -input $INPUT_FILE_PATH_1 \
        -output $OUTPUT_PATH \
        -jobconf "mapred.reduce.tasks=1"\
        -file ./map.py -mapper "python map.py" \
        -file ./reduce.py -reducer "python reduce.py" \

```

### 第八步、提交任务

```
[root@master mr_wordcount]# bash run.sh
```

### 第九步、将结果下载到本地查看

```
[root@master mr_wordcount]# hadoop fs -get /output/part-00000
18/03/20 07:34:56 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
```

