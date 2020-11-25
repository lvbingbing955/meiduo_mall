[TOC]

# elasticserarch的使用

使用haystack+elasticsearch进行搜索

haystack配置

dev.py

```python
INSTALLED_APPS = [
    # 'haystack',
]

# Haystack
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.elasticsearch_backend.ElasticsearchSearchEngine',
        'URL': 'http://192.168.47.128:9200/',  # Elasticsearch服务器ip地址，端口号固定为9200
        'INDEX_NAME': 'meiduo_tbd39',  # Elasticsearch建立的索引库的名称
    },
}

# 当添加、修改、删除数据时，自动生成索引
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
# 页大小

HAYSTACK_SEARCH_RESULTS_PER_PAGE = 2
```

urls.py

```python
urlpatterns = [
    path('admin/', admin.site.urls),
    # url(r'^search/', include('haystack.urls')),
]
```

### 渲染查询结果

- 请求路径的url可以自己配置
- 请求方式为get
- 查询字符串的键为q
- 在search.html模板中使用的数据：
    - query：查询关键字
    - paginator：分页对象，可以获得总页面
    - page：当前页数据，遍历得到result，属性object表示sku对象
    - 通过page可以获得当前页码
- 配置：指定页大小
- haystack中视图接收的参数：
    - q：表示查询关键字
    - page：表示当前页面
    - 请求方式为get