[TOP]

# 读写分离配置

dev.py

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',  # 数据库引擎
        'HOST': '192.168.5.128',  # 数据库主机
        'PORT': 3306,  # 数据库端口
        'USER': 'root',  # 数据库用户名
        'PASSWORD': '123456',  # 数据库用户密码
        'NAME': 'meiduo_mall'  # 数据库名字
    },
    'slave': {
        'ENGINE': 'django.db.backends.mysql',  # 数据库引擎
        'HOST': '192.168.5.128',  # 数据库主机
        'PORT': 8306,  # 数据库端口
        'USER': 'root',  # 数据库用户名
        'PASSWORD': '123456',  # 数据库用户密码
        'NAME': 'meiduo_mall'  # 数据库名字
    },
}

# 数据库路由规则
DATABASE_ROUTERS = ['meiduo_mall.utils.db_router.MasterSlaveDBRouter']
```

db_router.py

```python
class MasterSlaveDBRouter(object):
    """数据库读写路由"""

    def db_for_read(self, model, **hints):
        """读"""
        return "slave"

    def db_for_write(self, model, **hints):
        """写"""
        return "default"

    def allow_relation(self, obj1, obj2, **hints):
        """是否运行关联操作"""
        return True

```

# uwsgi使用

1.    pip install uwsgi

2.    wsgi.py

   ```python
   import os
   
   from django.core.wsgi import get_wsgi_application
   
   os.environ.setdefault("DJANGO_SETTINGS_MODULE", "meiduo_mall.settings.prod")
   
   application = get_wsgi_application()
   
   ```

3.    uwsgi.ini

   ```ini
   [uwsgi]
   # 使用Nginx连接时使用，Django程序所在服务器地址
   socket=127.0.0.1:8001
   # 项目目录
   chdir=D:\code\python\meiduo_mall\meiduo_mall
   # 项目中wsgi.py文件的目录，相对于项目目录
   wsgi-file=meiduo_mall/wsgi.py
   # 进程数
   processes=4
   # 线程数
   threads=2
   # uwsgi服务器的角色
   master=True
   # 存放进程编号的文件
   pidfile=uwsgi.pid
   # 日志文件
   daemonize=uwsgi.log
   # 指定依赖的虚拟环境
   #virtualenv=/home/python/.virtualenvs/meiduo5
   ```

4. 启动   uwsgi  --ini  uwsgi.ini

5. nginx转发

   ```
   upstream meiduo {
       server	127.0.0.1:8001;
       ...
   }
   
   server {
       listen		80;					# 监听端口
       server_name	www.meiduo.site;	# 或监听域名
       
       location /{
           include uwsgi_params;
           uwsgi_pass meiduo;
       }
   }
   ```

   

