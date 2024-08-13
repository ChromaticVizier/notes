### Ubuntu2004部署mongodb并用pycharm连接

---

##### 1、官网下载压缩包并解压

`https://www.mongodb.com/try/download/community`

下拉找到对应系统版本的tgz格式下载，并上传到服务器：

```cmd
scp <download directory>/mongodb-linux-x86_64-ubuntu2004-6.0.16 <uname>@<server ip>:<mongodb directory>
```

解压缩：

`tar -zxvf mongodb-linux-x86_64-ubuntu2204-6.0.11.tgz`

##### 2、创建mongodb工作目录

`mkdir -p /etc/mongodb/data /etc/mongodb/log /etc/mongodb/conf`

data是装数据的目录，log是日志，conf放配置文件，稍后在写配置文件时都会提到这三个目录。

##### 3、编写配置文件

`vim /etc/mongodb/conf/mongodb.conf`

```
systemLog:
  destination: file
  path: /etc/mongodb/log/mongodb.log
  logAppend: true
storage:
  dbPath: /etc/mongodb/data
  engine: wiredTiger
  journal:
    enabled: true
net:
  bindIp: 0.0.0.0
  port: 27017
processManagement:
  fork: true
```

注意，由于mongo在解析配置文件时用的是yaml，在写配置文件的时候需要遵循：

1、首层不缩进，冒号后无空格；

2、第二层前边4个空格（不能用tab键），第三层8个空格以此类推；

3、冒号后如果是值，要加一个空格。

以上三条违反任意一条，会报错：

```
Error parsing YAML config file: yaml-cpp: error at line 9, column 2: end of map not found
try 'bin/mongod --help' for more information
```

##### 4、启动mongodb

用配置文件启动，输入命令前先进入安装目录：

`bin/mongod -f /etc/mongodb/conf/mongodb.conf`

##### 5、安装shell

mongo不像mysql，安装数据库后并不会自动安装shell，需要手动安装来初始化：

`apt install mongodb-clients`

安装完毕后输入mongo即可进入shell。

##### 6、创建超级用户

```
db.createUser(
    {
        user:"root",
        pwd:"pwd",
        roles:["root"]
    }
)
```

##### 7、用pycharm连接

先开放服务器的27017端口（或配置文件中的自定义端口），之后按老方法连即可。

##### 8、其他注意事项

mongo支持隐式创建，创建数据库可以直接use：

`use <DB name>;`

创建空集合：

`db.createCollection("<collection name>")`