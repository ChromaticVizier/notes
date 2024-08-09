# flask+mysql 纯后端项目部署过程

---

#### 一、前置准备（如果只跑一个服务，则只需做一次）

##### （一）更新apt

`apt update`

##### （二）安装Nginx

`apt install nginx`

##### （三） 启动Nginx服务

`systemctl start nginx`

##### （四）检查Nginx状态

`systemctl status nginx`

看到绿色active就算成功

#### 二、安装mysql（如果服务器已有mysql，就跳过）

##### （一）基础安装

查看可用包：

`apt search mysql-server`

安装最新版：

`sudo apt install -y mysql-server`

设置开机自启：

`sudo systemctl enable mysql`

检查mysql状态：

`systemctl status mysql`

看到绿色active就行。

##### （二）设置密码权限

首次免密登录：

`mysql -uroot -p`

更改root密码：

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';`

刷新缓存：

`flush privileges;`

在

`vim /etc/mysql/mysql.conf.d/mysqld.cnf`

中找到bind-address字段，重设：

`bind-address            = 0.0.0.0`

重启mysql：

`systemctl restart mysql`

##### （三）*允许外网访问

经过上边两步，服务器上已经可以运行mysql，但是在自己电脑测试连接时连不上。

创建新的root用户，并授予它全部访问权限，包括从其它ip访问：

`create user 'root'@'%' identified with mysql_native_password by 'xxxxx';`
`grant all privileges on *.* to 'root'@'%' with grant option;`
`flush privileges;`

这样就可以在navicat等软件用ip和用户名密码连上。

#### 三、主要过程（每次有新项目部署时都要做）

##### （一）创建虚拟环境

`python3 -m venv myvenv`

##### （二）激活环境

`source my_venv/bin/activate`

##### （三）在环境里安装python包

`pip install -r requirements.txt的路径`

##### （四）配置nginx

把

```
server {
  listen 80;  # 监听公网端口（此处为标准HTTP端口80）
  server_name your_domain.com;  # 替换为域名或公网IP

  location / {
  proxy_pass http://localhost:5000;  # 替换为项目实际监听的本地端口
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

写入

`vim /etc/nginx/sites-available/default`

和

`vim /etc/nginx/nginx.conf`

写在.conf文件的http块内，

重启Nginx：

`systemctl restart nginx`

##### （五）项目复制

`scp -r windows路径 用户名@ip:linux路径`

注意：项目文件夹要放在虚拟环境的文件夹下。

##### （六）设置环境变量，测试flask应用

`export FLASK_APP=app.py`

进入app.py路径执行

`flask run`

查看5000端口进程

`netstat -tulpn | grep :5000`

##### （七）在环境中安装gunicorn

`pip install gunicorn`

启动gunicorn

`exec gunicorn --bind 0.0.0.0:5000 app:app`

进入：

`vim /etc/nginx/nginx.conf`

写入：

```
server {
		listen 80;  # 监听公网端口（此处为标准HTTP端口80）
		server_name 114.51.41.91;  # 替换为您的域名或公网IP

    ​	location / {
    ​		proxy_pass http://localhost:5000;  # 替换为项目实际监听的本地端口
    ​		proxy_set_header Host $host;
    ​		proxy_set_header X-Real-IP $remote_addr;
    ​		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    ​		proxy_set_header X-Forwarded-Proto $scheme;
    ​		proxy_redirect off;
    ​		proxy_buffering off;
    ​		proxy_http_version 1.1;
    ​		proxy_set_header Upgrade $http_upgrade;
    ​		proxy_set_header Connection "upgrade";
    ​		proxy_read_timeout 600s;
    ​	}
}
```

##### （八）检查并启动（修改已部署项目时只需做这一步）

`sudo systemctl restart nginx`

`exec gunicorn --bind 0.0.0.0:5000 app:app`

#### 四、Flask+mysql程序编写注意事项

##### （零）！特别注意！

在程序入口处，必须判断main入口：

```python
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

否则gunicorn启动不了，会刷屏报端口占用错误。

##### （一）关于前端请求发送

在本地编写测试时，我们经常在前端直接用127.0.0.1发送请求，形如：

```javascript
// 使用 fetch 发送 POST 请求
fetch('http://127.0.0.1:5000/msg_bd/add_comment', {
    method: 'POST',
    body: formData
})
```

如果在服务器上运行这种代码，公网访问时则会报错：

`net::ERR_CONNECTION_REFUSED`

这是因为前端把请求发送到了用户自己的电脑上，而非服务器。

所以发请求时必须用服务器ip：

```javascript
// 使用 fetch 发送 POST 请求
fetch('http://服务器ip/msg_bd/add_comment', {
    method: 'POST',
    body: formData
})
```

##### （二）关于程序入口

程序入口最好单独放在app.py中，而且host必须是0.0.0.0，端口号与nginx配置一致：

```
if name == 'main':    
​	app.run(host='0.0.0.0', port=5000, debug=True)
```

##### （三）关于程序中mysql的连接

用3306端口即可，不必用33060：

```
HOSTNAME = '服务器ip'
DATABASE = ''
PORT = 3306
USERNAME = 'root'
PASSWORD = ''
DB_URL = 'mysql+pymysql://{}:{}@{}:{}/{}'.format(USERNAME, PASSWORD, HOSTNAME, PORT, DATABASE)
```

#### 五、更新已部署项目的流程

##### （一）激活虚拟环境，把原项目文件夹删掉，新文件夹复制过去

```
cd web/mbvenv
source bin/activate
```

##### （二）进入项目文件及，检查端口，重启gunicorn

```
cd message_board
netstat -tulpn | grep :5000
exec gunicorn --bind 0.0.0.0:5000 app:app
```

