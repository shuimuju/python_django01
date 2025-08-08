# Django

# 命令

## 创建应用

`python manage.py startapp demo01`

## 模型驱动数据库

`python manage.py makemigrations demo01`

`python manage.py migrate demo01`

### Model模型：

- 主键：默认自增，为pk
- 外键：model_id

## 数据库驱动模型

`python manage.py inspectdb > demo1/models.py`

## 查看版本

```bash
python -m django --version
```

# 文件目录

- manage.py
- mysite/__init__.py
- mysite/setting.py:配置文件
- mysite/urls.py
- mysite/ wsgi.py

## templates目录

- 放html文件

- 配置setting.py

  ```python
  TEMPLATES = [
      {
          'BACKEND': 'django.template.backends.django.DjangoTemplates',
          # os.path.join当前项目
          'DIRS': [os.path.join(BASE_DIR, 'templates'),],
          'APP_DIRS': True,
          'OPTIONS': {
              'context_processors': [
                  'django.template.context_processors.debug',
                  'django.template.context_processors.request',
                  'django.contrib.auth.context_processors.auth',
                  'django.contrib.messages.context_processors.messages',
              ],
          },
      },
  ]
  ```

## 创建static目录

- 放创建css目录，js目录，images目录，plugins目录

- ```python
  STATICFILES_DIRS = [
      os.path.join(BASE_DIR, 'static')
  ]
  STATIC_URL = '/static/'
  ```

## urls.py:url调度

1. 项目路由配置 ```urls.py```

   ```PYTHON
   from django.contrib import admin
   from django.urls import path,include
   
   urlpatterns = [
       path('admin/', admin.site.urls),
       path('polls/', include('polls.urls')),
   ]
   ```

2. 应用路由配置```url.py```

   ```PYTHON
   from django.urls import path
   from . import views
   urlpatterns = [
       path('polls/', views.index, name='index'),
   ]
   ```

3. 应用视图配置```views.py```

   ```python
   from django.http import HttpResponse
   # Create your views here.
   def index(request):
       return HttpResponse("Hello, world. You're at the polls index.")
   ```

- wsgi.py:兼容Web服务器上得入口

# 更换端口

```bash
python manage.py runserver [8080]
```

监听所有服务器的公开IP

```bash
python manage.py runserver 0:8000
```

# 错误收集

```django.db.utils.programmingerror: column mmitembase.id is not exsit```

models.py没有设置主键

# 数据库

## 事务管理

- ```python
  transaction.atomic():
      model_object.save()
  ```

## MySQL

## PostgreSQL

# 模型(Models)

## 查询

### 多表查询

#### 一对多(ForeignKey):  操作带外键的对象

- 方式一：以对象的形式传递

  ```python
  def index14(request):
      publish = Publish.objects.filter(pk=2).first()
      book = Booking.objects.create(title='天妖剑法', price=500, pub_date='2024-05-01', publish=publish)
      print(book, type(book))
      return HttpResponse(book, content_type='text/html;charset=utf-8')
  ```

- 方式二：以id的形式传递

  ```python
  def index13(request):
      publish = Publish.objects.filter(pk=1).first() # pk默认为id
      pk = publish.pk
      book = Booking.objects.create(title='冲灵剑法', price=200, pub_date='2024-01-01', publish_id=pk) #publish_id 默认为ForeignKey
      print(book, type(book))
      return HttpResponse(book, content_type='text/html;charset=utf-8')
  ```

#### 多对多(ManytoManyField):在第三张表新增数据

##### 关联管理器(对象调用)

- 多对多(推荐使用，一对多：不推荐使用)

- 正向：带ManyToManyField属性的对象，被ManyTomanyField关联的对象为反向

- add添加方式

  - 单个添加

    ```python
    @transaction.atomic
    def index15(request):
        author_one = Author.objects.filter(name='令狐冲').first()
        author_two = Author.objects.filter(name='任我行').first()
        books = Booking.objects.get(id=7)
    	# books.authors.add(author_one, author_two) # 通过对象形式添加
        books.authors.add(author_one.pk,author_two.pk) # 通过ID形式添加
        # authors.booking_set.add(book_object_one.pk,book_object_two.pk)
        # authors.booking_set.add(book_object_one.pk,book_object_two.pk)
        return HttpResponse(books, content_type='text/html;charset=utf-8')
    ```

  - 批量添加

    ```python
    @transaction.atomic
    def index17(request):
        book = Booking.objects.get(pk=8)
        list_author = Author.objects.filter(id__gte=2)
        # book.authors.add(*list_author)  # *+对象的集合(QuerySet)
        # author.booking_set.add(*list_book) # *+对象的集合(QuerySet)
        list_id =(item.id for item in list_author)
        book.authors.add(*list_id) # *+对象id的序列
        # list_id =(item.id for item in list_book)
        # author.booking_set.add(*list_id)
        return HttpResponse(book, content_type='text/html;charset=utf-8')
    ```

- create():

  ```python
  @transaction.atomic
  def index19(request):
      publish = Publish.objects.filter(name='华山出版社').first()
      author = Author.objects.filter(name='任我行').first()
      book = author.booking_set.create(title='凌波微步',price=300,pub_date='2001-01-01',publish=publish)
      return HttpResponse(book,content_type='text/html;charset=utf-8')
  ```

- remove():从关联对象中，移除指定对象的单一数据，对ForignKey,null=true时存在

  ```python
  @transaction.atomic()
  def index20(request):
      author_object = Author.objects.get(id=1)
      book_object = Booking.objects.get(id=7)
      author_object.booking_set.remove(book_object)
      return HttpResponse("ok")
  ```

- clear():从关联对象中，移除指定对象关联的所有数据，对ForignKey,null=true时存在

  ```python
  @transaction.atomic()
  def index21(request):
      book_object = Booking.objects.get(id=3)
      book_object.authors.clear()
      return HttpResponse("ok")
  ```

##### 查询：

```python
def index22(request):
    '''
    正向查询:如果是一对多:返回只有一个对象
    :param request: 
    :return: 
    '''
    book_object = Booking.objects.get(id=6) 
 
    list_authors = book_object.authors.all()
    '''
    反向查询
    '''
    author_object = Author.objects.get(id=1)
    list_books = author_object.booking_set.all()
    return HttpResponse(list_books)
```

- 一对一没有正反向之分:不需要用_set

  ```python
   au_detail = models.OneToOneField("AuthorDetail", on_delete=models.CASCADE)
  ```

- 跨表查询:filter()与values_list()字段名字的写法

  ```python
  def index23(request):
      '''
      正向跨表查询
      '''
      # res = Booking.objects.filter(authors__name='任我行').values('title','price')
      # res = Booking.objects.filter(authors__name='任我行').values_list('title','price')
      '''
       反向跨表查询
      '''
      # res = Author.objects.filter(name='任我行').values('booking__title','booking__price')
      res = Author.objects.filter(name='任我行').values_list('booking__title','booking__price')
      return HttpResponse(res)
  ```

##### 聚合查询



### 单表查询

- 获取所有数据

  ```python
  DemoModel.objects.all()
  ```

- 条件过滤数据

  ``` DemoModel.objects.filter(column_name = 'a' )```

  `DemoModel.objects.filter(column_name= 'a').order_by('column_name')[0:2]`

- 过滤字段

  - column_name__isull:不为空

    `DemoModel.objects.filter(column_name__isnull = False)`

  - column_name__in:相当于sql中in(value1,value2)

    `data = Mmitembase.objects.filter(column_name__in=[22000,26000]).values_list('ncodeitemid','itemmodel')`

  - column_name__gt:大于号，=后面为数字

    `data = Mmitembase.objects.filter(column_name__gt=22000).values_list('ncodeitemid','itemmodel')`

  - column_name__gte:大于等于号，=后面为数字

    `data = Mmitembase.objects.filter(column_name__gte=22000).values_list('ncodeitemid','itemmodel')`

- 查询不符合条件

  `DemoModel.object.exclude(column_name = 'a')`

- 获取单个对象

  `DemoModel.objects.get(column_name = 'a')`

- 排序并限制返回的数据:[0:2]与str的切片类似,索引是从0开始, 升序

  `DemoModel.objects.order_by('column_name')[0:2]`

- 排序并限制返回的数据:[0:2]与str的切片类似,索引是从0开始, 降序

  `DemoModel.objects.order_by('-column_name')[0:2]`

- reverse()：将升序变成降序，可以降序变成升序

  `data = Mmitembase.objects.order_by('ncodeitemid').reverse()[0:10]`

- count():返回的数据计数

  ```python
  data = Mmitembase.objects.order_by('ncodeitemid').reverse()[0:10]
  data.Count()
  ```

- first():查询的数据结果返回第一条数据

  ```python
  data = Mmitembase.objects.order_by('ncodeitemid').reverse()[0:10].first()
  print(data.ncodeitemid)
  ```

- last():查询的数据结果返回最后一条数据：不能使用切片[0:10]

  ```python
  data = Mmitembase.objects.order_by('ncodeitemid').reverse()
  data_last = data.last()
  print(data_last.ncodeitemid)
  ```

- exists():判断的数据类型只能QuerySet类型，不能为整形和模型对象

  ```python
  bool = Mmitembase.objects.exists() #True 
  bool = Mmitembase.objects.order_by('ncodeitemid').reverse().exists() #True
  bool  = Mmitembase.objects.order_by('ncodeitemid').count() # count()返回的是int类型不能使用exists()方法
  # bool = Mmitembase.objects.order_by('ncodeitemid').reverse().fist() #fist()返回对象不能使用exists()方法
  # bool = Mmitembase.objects.order_by('ncodeitemid').reverse().last() #last()返回对象不能使用exists()方法
  print(bool)
  ```

- values():返回的是QuerySet类型数据，一个序列，以字典对象为元素。字典里的键是模型对象的字段，值是模型对象的字段的值

  ```python
  data_values = Mmitembase.objects.order_by('ncodeitemid').reverse()[0:10]			.values('ncodeitemid','itemmodel')
  for item in data_values:
      for k,v in item.items():
          print(f'k:{k} v:{v}')
  return HttpResponse(data_values)
  ```

- values_list():返回的是QuerySet类型数据，以元组为元素。元组中元素以查询字段的顺序，存放相应的值

  ```python
  def index10(request):
      data = Mmitembase.objects.order_by('ncodeitemid').reverse()[0:10].values_list('ncodeitemid','itemmodel')
      for item in data:
          for i,j in enumerate(item):
              print(f'i:{i} j:{j}')
      return HttpResponse(data)
  ```

- distinct():对数据去重,与values()，valuse_list()

  ```python
  data =Mmitembase.objects.order_by('ncodeitemid').values_list('ncodeitemid','itemmodel').reverse().distinct()
  return HttpResponse(data,content_type="text/html;charset=utf-8")
  ```

### 字段逻辑运算符

- `in`

  `data = Mmitembase.objects.filter(ncodeitemid__in=[22000,26000]).values_list('ncodeitemid','itemmodel')`

- `>`

  `data = Mmitembase.objects.filter(ncodeitemid__gt=22000).values_list('ncodeitemid','itemmodel')`

- `>=`

  `data = Mmitembase.objects.filter(ncodeitemid__gte=22000).values_list('ncodeitemid','itemmodel')`

- `<`

  `data = Mmitembase.objects.filter(ncodeitemid__lt=22000).values_list('ncodeitemid','itemmodel')`

- `<=`

  `data = Mmitembase.objects.filter(ncodeitemid__lte=22000).values_list('ncodeitemid','itemmodel')`

- between   and

  `data = Mmitembase.objects.filter(ncodeitemid__range=[22000,26000]).values_list('ncodeitemid','itemmodel')`

- contains: 

  itemmodel__contains：区分大小写，='字符串'

  `data = Mmitembase.objects.filter(itemmodel__contains='hj').values_list('ncodeitemid','itemmodel')`

  itemmodel__icontains:不区分大小写，='字符串'

  `data = Mmitembase.objects.filter(itemmodel__icontains='hj').values_list('ncodeitemid','itemmodel')` 

- column_name__startswith：<font color=red> 区分大小写</font>，以指定字符开头，='字符串'

- column_name_endswith: <font color = red>区分大小写</font>，以指定字符开投，='字符串'

### DateField数据类型

- column_name__year:=数值类型

  ` data = Mmitembase.objects.filter(changetime__year=2024).values_list('ncodeitemid','itemmodel','changetime')`

- column_name__month:=数值类型

  ` data = Mmitembase.objects.filter(changetime__month=2).values_list('ncodeitemid','itemmodel','changetime')`

- column_name__day: = 数值类型

  ` data = Mmitembase.objects.filter(changetime__day=20).values_list('ncodeitemid','itemmodel','changetime')`

## 更新

- 方式一：

  ```python
  demoModel = DemoModel.objects.get(column_name ='a')
  demoModel.column_name = 'a'
  demoModel.save()
  """
      data = Mmitembase.objects.filter(ncodeitemid=21874).first()
      data.itemmodel = 'itemmodel'
      data.save()
  """
  ```

- 方式二：

  ```python
  DemoModel.objects.filter(column_name = 'a').update(column_name = 'b')
  ```

- 方式三：

  ```python
  DemoModel.objects.all().update(column_name='c') #修改所有的列
  ```

- 方式四：

  ```python
  DemoModel.object.create(column_name = 'b')
  ```

  

## 删除

- 方式一：模型对象进行删除

  ```python
  demoModel = DemoModel.objects.get(column_name ='a')
  demoModel.delete()
  ```

- 方式二：QuerySet数据对象进行删除:<font color = red >推荐</font>

  ```python
  DemoModel.objects.filter(column_name='a').delete()
  ```

- 方式三：

  ```python
  DemoModel.objects.all().delete()#删除所有数据
  ```

# 路由

## urls.py

### 路由分发

### 命名空间

- urls.py

  ```python
  app_name = 'demo01'
  urlpatterns = [
      path('demo1/',views.index,name='index')
  ```

### 反向解析

```python
redirect(reverse("demo01:index05"))
```



# 模板

## 模板语法

### 注释`{# #}`

### {% include "demo.html" %}

### {% crsf_token %}:提交表单时，需要

### `{% if %} {% endif %}`  :逻辑关键字`or  and  not `

```HTML
{% if condition1 %}
   ... display 1
{% elif condition2 %}
   ... display 2
{% else %}
   ... display 3
{% endif %}
```

### `{% for %} {% endfor %}`:循环遍历

可以使用嵌套循环

###  {{forloop}} 变量

```html
{% for var in context %}
    {{ forloop.counter }} {# 索引从1开始 #}
    <br>
    {{ forloop.counter0 }} {# 索引从0开始 #}
    <br>
    {{ forloop.revcounter }} {# 倒序获取循环序号，结尾序号是1 #}
    <br>
    {{ forloop.revcounter0 }}{# 倒序获取循环序号，结尾序号是0 #}
    <br>
    {{ forloop.first }}{#第一条数据返回true #}
    <br>
    {{ forloop.last }} {# 最后一条数据返回true #}
    {% empty %} {# 迭代对象为空时，执行语句 #}
        hello,world,empty
{% endfor %}
```

### 字典对象

- view.py

  ```python
  def index02(request):
      context = {}
      context['hello'] = 'world'
      context['data'] = 'data'
      context['cda'] = 'abc'
      return render(request,'demo.html',{'context':context})
  ```

- html文件代码

  ```html
   {#根据字典key获取value值 #}
      {{ context.cda }}
      <br>
      {% for i,j in context.items %}
          {{ i }}  {{ j }}
      {% endfor %}
  ```

### 列表对象

- view.py

  ```python
  def index03(request):
      context = ['a','b','c']
      return render(request,'demo.html',{'context':context})
  ```

- html文件代码

  ```html
      {{ context }} {# 获取列表对象 #}
      {{ context.0 }}  {#0，列表对象的索引#}
      {% for var in context reversed %} {# 遍历列表元素,reversed反向迭代 #}
          {{ var }}
      {% endfor %}
  ```

### 过滤器

```{{变量名|过滤器:"可选参数"}}```

- lower

- upper

- first

- truncatewords

  `{{context|truncatewords:20}} {# 传递的参数小于context的长度，context会被截取并以...显示 #}`

- addslashes

- date

  `{{ context|date:'Y-m-d' }} `

- length

  ```html
  {{ context|length }} {# 返回列表的元素个数，返回字典的key个数，字符串返回的字符个数，集合返回去重后的长度 #}
  ```

- default

  `0  0.0  False 0j ""  [] () set() {} None` 默认布尔值为false

  ```html
  {{context|default:"default"}} {# false 返回default #}
  ```

- filesizeformat:显示文件大小

  `{{ context|filesizeformat }} {# context=10241024 显示9.8MB#}`

- safe:将字符串标记为安全，不需要转义，要保证view.py传过来的数据绝对安全，才能用safe

  `{{context|safe}}`

### 自定义标签与过滤器

#### 创建`templatetags`目录

- 新版：在新建的App目录下

- 创建：`__init__.py`

- 创建：`my_tags.py`

- 修改setting.py

  ```python
  TEMPLATES = [
      {
          'BACKEND': 'django.template.backends.django.DjangoTemplates',
          'DIRS': [os.path.join(BASE_DIR,'templates')]
          ,
          'APP_DIRS': True,
          'OPTIONS': {
              'context_processors': [
                  'django.template.context_processors.request',
                  'django.contrib.auth.context_processors.auth',
                  'django.contrib.messages.context_processors.messages',
              ],
              'libraries': {    # 添加这行
                  'my_tags01': 'demo01.templatetags.my_tags',# 添加这行demo01新建的app
              }				# 添加这行
          },
      },
  ]
  ```

#### my_tags.py

```python
from django import template

register = template.Library()


# 将自定义的过滤器方法注册register.filter中，name='my_name',过滤器的名字为my_name
# name属性没有被赋值，默认名字为方法名
@register.filter(name='feifei')
def my_filter(v1, v2):
    return v1 * v2
# @register.simple_tag(name='feifei2')
@register.simple_tag()
def my_simple_tag(v1, v2, v3):
    return v1 + v2 + v3

```

#### HTML

```html
{% load my_tags %}  {# 加载自定义的标签.py文件#}
<body>
    {# 过滤器 #}
    {# {{ 10|my_filter:22 }} #}
    {{ 10|feifei:22}}
    {# 标签 #}
    {# {% feifei2 11 22 33%}#}
    {% my_simple_tag 11 22 33 %}
</body> 
```

### static目录的引入

#### image

```HTML
{% load static %}
<body>
   图片<img src="{% static 'images/12.png'%}"
</body>
```

#### plugins

```html
<link rel="stylesheet" href="/static/plugins/bootstrap-3.3.7/dist/css/bootstrap.css">
```

## 模板的继承

### 父模板:demo.html

```html
<body>
    {% block custom_name %}
       <p> 共用并可以修改区域</p>
    {% endblock custom_name  %}
</body>    
```

### 子模板:child.html

```HTML
<body>
    {%extends "demo.html" %}
    {% block custom_name %}
        <p>继承父类，修改对应区域</p>
    {% endblock custom_name %}
</body>
```



## 表单

### HTTP请求

#### GET与POST方法

- 表单html

  ```HTML
  <form action="{% url 'demo01:search' %}" method="get" >
           {% csrf_token %} {# POST方法必须添加 #}
          <input type="text" name="q">
          <input type="submit" name="搜索">
      </form>
  ```

  ```PYTHON
  app_name = 'demo01'
  urlpatterns = [
      path('demo1/',views.index,name='index'),
      path('index01/',views.index01,name='index01'),
      path('index02/',views.index02,name='index02'),
      path('index03/',views.index03,name='index03'),
      path('index04/',views.index04,name='index04'),
      path('search-form/',search.search_form,name='search_form'),
      path('search/',search.search,name='search'),
  ]
  ```

  ```python
  def search_form(request):
      return render(request,'report_app/search_form.html')
  def search(request):
      request.encoding = 'utf-8'
      if('q' in request.GET and request.GET['q']): # requeset.POST['q'],获取的是表单中name标签的value
          message = '你搜索的内容' +  request.GET['q'] # request.POST['q'],获取的是表单中name标签的value
      else:
          message = 'empty'
      return HttpResponse(message)
  ```

# Request对象

## GET

### 获取前端传递的参数

- 前端htmL

  ```html
  http://localhost:8080/demo1/search-form/1234
  ```

- 后端url.py

  ```python
  app_name = 'demo01'
  
  path('search-form/<int:query>',search.search_form,name='search_form'),
  ```

- 后端search.py：参数名要一致

  ```python
  def search_form(request,query):
      print(query)
      query = 1000
      return render(request,'report_app/search_form.html',{'query':query})
  ```

- 在search_form.html中获取后端传递的参数`query`

  ```html
  <a href="{% url 'demo01:search2' query=query  %}">search2</a>
  ```

- 后端url.py

  ```python
  app_name = 'demo01'
  
  path('search2/<int:query>',search.search2,name='search2'),
  ```

- 后端search.py：参数名要一致

  ```python
  def search2(request,query):
      print(query)
      query = 10000
      message = 'success' + str(query)
      context = {'message':message}
      return HttpResponse(request,context)
  ```

## POST

### 获取前端传递的参数:表单

- 前端HTML

  ```html
  	 <form action="{% url 'demo01:search' query=query %}" method="post" >
           {% csrf_token %}
          <input type="text" name="q">
          <input type="submit" name="搜索">
      </form>
  ```

- 后端url.py

  ```python
   app_name = 'demo01'
      
      
  path('search-form/<int:query>',search.search_form,name='search_form'),
  ```

- 后端search.py:传递的参数名要一致

  ```python
  def search(request,query):
      request.encoding = 'utf-8'
      print(query)
      if('q' in request.POST and request.POST['q']):
          message = '你搜索的内容' +  request.POST['q']
          print(request.POST.get('q'))  # request.POST.get('q')与request.POST['q']，获取表单标签name属性对应的			# value属性效果是一样的
      else:
          message = 'empty'
      return HttpResponse(message)
  ```

  

## body

- 数据类型是二进制字节流，在HTTP中用于POST，因为GET请求没有body，获取的是b''

  ```python
  body = request.body #获取请求体
  ```

## path

- 获取URL的部分路径，数据类型是字符串

  ```python
   path = request.path # path:/demo1/search-form/1234
  ```

## method

- 获取请求的方式，数据类型是字符串，且结果为大写

  ```python
    method = request.method # method:GET
  ```

## HttpResponse 对象

### HttpResponse()

```python
   # return HttpResponse("context") #直接返回文本数据,参数为字符串
    return HttpResponse("<a href=http://www.baidu.com>百度</a>") # 带html标签，也可以渲染
```

### render():第一个参数必须为request，第二个参数为页面路径，第三个参数为字典对象

```python
def search_form(request,query):
    print(query)
    # body = request.body
    # print(body)
    # path = request.path
    # print(path)
    # method = request.method # method:GET
    # print(method)

    query = 1000
    return render(request,'report_app/search_form.html',{'query':query})
```

### redirect()：重定向

- search.py

  ```python
  return redirect("demo01:index05")
  ```

- urls.py

  ```python
  app_name = 'demo01'
  
  path('index05/',search.index05,name='index05'),
  ```

- search.py

  ```python
  def index05(request):
      return render(request,'report_app/index.html')
  ```

  

