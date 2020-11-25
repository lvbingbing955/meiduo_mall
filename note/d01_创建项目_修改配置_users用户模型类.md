[TOC]

# 创建虚拟环境与项目

```
# 创建虚拟环境
virtualenv myenv_meiduo_mall
pip install django==2.0

# 创建项目
django-admin startproject meiduo_mall

# 运行项目
python manage.py runserver 0.0.0.0:8000

# 添加模块
cd meiduo_mall/apps
python ../../manage.py startapp users

# 模型类迁移
python manage.py makemigrations
python manage.py migrate
```

# 修改配置

1. 假设根目录/manage.py，创建/meiduo_mall/settings目录，将/meiduo_mall/settings.py文件移动为/meiduo_mall/settings/dev.py文件。这样可以用来区分开发(dev.py)、测试(test.py)、生产(prod.py)等不同的环境配置。

2. 修改manage.py文件

   ```python
   if __name__ == "__main__":
       os.environ.setdefault("DJANGO_SETTINGS_MODULE", "meiduo_mall.settings.dev")
   ```

3. 配置模板引擎相关

   修改dev.py文件

   ```python
   TEMPLATES = [
       {
           # 修改Django使用默认的模板引擎为 django.DjangoTemplates -> jinja2.Jinja2
           'BACKEND': 'django.template.backends.jinja2.Jinja2',
           # 指定模板文件目录
           'DIRS': [os.path.join(BASE_DIR, 'templates')],
           'APP_DIRS': True,
           'OPTIONS': {
               'context_processors': [
                   'django.template.context_processors.debug',
                   'django.template.context_processors.request',
                   'django.contrib.auth.context_processors.auth',
                   'django.contrib.messages.context_processors.messages',
               ],
               # 补充Jinja2模板引擎环境
               'environment': 'meiduo_mall.utils.jinja2_env.jinja2_environment',
           },
       },
   ]
   ```

   创建/meiduo_mall/utils/jinja2_env.py文件，内容如下

   ```python
   from django.contrib.staticfiles.storage import staticfiles_storage
   from django.urls import reverse
   from jinja2 import Environment
   
   
   def jinja2_environment(**options):
       env = Environment(**options)
       env.globals.update({
           'static': staticfiles_storage.url,
           'url': reverse,
       })
       return env
   
   
   """
   确保可以使用Django模板引擎中的{% url('') %} {% static('') %}这类的语句
   """
   
   ```

4. 配置数据库连接

   修改dev.py文件

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
   }
   ```

   安装驱动

   ```
   pip install PyMySQL
   ```

   修改/meiduo_mall/__init__.py文件内容

   ```python
   from pymysql import install_as_MySQLdb
   install_as_MySQLdb()
   ```

5. 配置redis连接

   安装驱动

   ```
   pip install dranjo-redis
   ```

   修改dev.py文件

   ```python
   # 缓存
   CACHES = {
       "default": {  # 默认
           "BACKEND": "django_redis.cache.RedisCache",
           "LOCATION": "redis://127.0.0.1:6379/0",  # 可改：ip、port、db
           "OPTIONS": {
               "CLIENT_CLASS": "django_redis.client.DefaultClient",
               "PASSWORD": "12346",
           }
       },
       "session": {  # session
           "BACKEND": "django_redis.cache.RedisCache",
           "LOCATION": "redis://127.0.0.1:6379/1",
           "OPTIONS": {
               "CLIENT_CLASS": "django_redis.client.DefaultClient",
               "PASSWORD": "12346",
           }
       },
   }
   # 指定session的保存方案
   SESSION_ENGINE = "django.contrib.sessions.backends.cache"
   SESSION_CACHE_ALIAS = "session"
   ```

   docker使用redis镜像。

   ```
   # docker pull redis
   
   # vi /root/redis/redis.conf
   
   # bind 127.0.0.1
   protected-mode no
   
   # docker run -p 6379:6379 --name myredis -v /root/redis/redis.conf:/etc/redis/redis.conf -v /root/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes --requirepass "123456"
   
   # -p 6699:6379 本地端口和redis端口，在其他客户端用第一个端口表示连接代理到redis的6379端口
   # --name myredis 容器名称
   # -v /root/myredis/redis.conf/redis.conf:/etc/redis/redis.conf将conf文件里内容映射到redis镜像文件里，如果不生效可直接设置在镜像文件中
   # -v /root/data:/data redis存储数据
   # -d redis redis-server /etc/redis/redis.conf redis服务使用的conf文件地址引用
   # --appendonly yes后台运行模式 是
   # --requirepass "123456" 手动设置密码
   ```

6. 配置日志

   修改dev.py文件

   ```python
   # 日志：当运行出错时，记录在日志中，方便后续修改
   LOGGING = {
       'version': 1,
       'disable_existing_loggers': False,  # 是否禁用已经存在的日志器
       'formatters': {  # 日志信息显示的格式
           'verbose': {
               'format': '%(levelname)s %(asctime)s %(module)s %(lineno)d %(message)s'
           },
           'simple': {
               'format': '%(levelname)s %(module)s %(lineno)d %(message)s'
           },
       },
       'filters': {  # 对日志进行过滤
           'require_debug_true': {  # django在debug模式下才输出日志
               '()': 'django.utils.log.RequireDebugTrue',
           },
       },
       'handlers': {  # 日志处理方法
           'console': {  # 向终端中输出日志
               'level': 'INFO',
               'filters': ['require_debug_true'],
               'class': 'logging.StreamHandler',
               'formatter': 'simple'
           },
           'file': {  # 向文件中输出日志
               'level': 'INFO',
               'class': 'logging.handlers.RotatingFileHandler',
               'filename': os.path.join(os.path.dirname(BASE_DIR), 'logs/meiduo.log'),  # 日志文件的位置
               'maxBytes': 300 * 1024 * 1024,
               'backupCount': 10,
               'formatter': 'verbose'
           },
       },
       'loggers': {  # 日志器
           'django': {  # 定义了一个名为django的日志器
               'handlers': ['console', 'file'],  # 可以同时向终端与文件中输出日志
               'propagate': True,  # 是否继续传递日志信息
               'level': 'INFO',  # 日志器接收的最低日志级别
           },
       }
   }
   ```

7. 静态文件

   ```
   # 静态文件访问的url路径
   STATIC_URL = '/static/'
   # 静态文件的磁盘路径
   STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
   ```


# 添加users模块

1. 创建模块

   ```python
   python ../../manage.py startapp users
   ```

2. 修改dev.py文件

   ```python
   import os   # 操作系统ubuntu模块
   import sys  # python模块
   
   # sys.path#导入包的路径
   
   # 指定应用的导包路径为meiduo_mall/apps
   sys.path.insert(0, os.path.join(BASE_DIR, 'apps'))
   
   INSTALLED_APPS = [
       'django.contrib.admin',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
   
       # 完整导包路径
       # 'meiduo_mall.apps.users.apps.UsersConfig',
       'users.apps.UsersConfig',
   ]
   
   # 指定用户模型类
   AUTH_USER_MODEL = 'users.User'  # 应用名称.模型类名称
   ```

3. 在models.py中定义模型类

   ```python
   from django.db import models
   from django.contrib.auth.models import AbstractUser
   
   
   class User(AbstractUser):
       mobile = models.CharField(max_length=11)
   
   ```

4. 迁移模型类

   ```python
   python manage.py makemigrations
   python manage.py migrate
   ```

