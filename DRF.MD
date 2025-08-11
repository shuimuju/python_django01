# 安装

## conda安装

```bash
conda install conda-forge::djangorestframework
```

## python安装

```bash
pip install djangorestframework
```

# 序列化

- 创建序列化类

  ```python
  from rest_framework import serializers
  from demo01.models import Booking
  class BookSerializer(serializers.ModelSerializer):
      class Meta:
          model=Booking
          fields='__all__'
  ```

- ```python
  def index31(request):
      books = Booking.objects.all()
      serializer = BookSerializer(books,many=True)
      return HttpResponse(serializer.data,content_type='text/html;charset=utf-8')
  ```