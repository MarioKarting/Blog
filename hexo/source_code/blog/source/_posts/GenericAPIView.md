---
layout: post
title:  GenericViewSet 介绍
date: 2019-08-09 00:15
comments: true
toc: true
reward: true
tags: 
    - python 
    - DRF
    - Web
    - 随笔


---

介绍了一些东东
<!-- more -->
### GenericViewSet

> 视图集 继承GenericAPIView,增加了对于列表视图和详情视图可能用到的通用支持方法。通常使用时，可搭配一个或多个Mixin扩展类。



##### 列表视图与详情视图通用：
- queryset 列表视图的查询集
    - **serializer_class** 视图使用的序列化器
- 列表视图使用：
    - **pagination_class** 分页控制类
    - **filter_backends** 过滤控制后端
- 详情页视图使用：
    - **lookup_field** 查询单一数据库对象时使用的条件字段，默认为'pk'
    - **lookup_url_kwarg** 查询单一数据时URL中的参数关键字名称，默认与look_field相同

{% codeblock lang:python%}

from rest_framework.viewsets import GenericViewSet
from C_databases.models import BookInfo
from book_serializer.serializers import BookInfoSerializer
from rest_framework.response import Response

#获取所有图书(list),保存图书(create)
class BooksView(GenericViewSet):
    #指定视图所用的查询集queryset
    queryset = BookInfo.objects.all()
    #指定使用的序列化器
    serializer_class = BookInfoSerializer
    
    #一、查询所有的图书
    def list(self,reuqest):
        #1.获取查询集中所有的数据
        books = self.get_queryset()
        #2.将查询集作为参数传到序列化器中
        ser = self.get_serializer(books,many=Ture)
        #3.返回数据
        return Response(ser.data)
        
    #二、保存图书
    def create(self,request)
        #1.接收来自前端的数据
        data = request.data
        #2.调用序列化器对数据对象进行验证
        ser = self.get_serializer(data=data)
        #3.验证
        ser.is_valid(raise_exception=Ture)
        #4.保存
        ser.save()
        #5.返回数据
        return Response(ser.data)
        
        
#查询单一图书(retrieve),更新图书(update),删除图书(destroy)
class BookView(GenericViewSet):
    #指定视图所用的查询集
    queryset = BookInfo.object.all()
    #指定所用的序列化器
    serializer_class = BookInfoSerializer
    
    def retrieve(self,request,pk):
        #1.获取单一图书信息 get_object()从查询集中获取符合pk所对应的id的数据对象
        book = self.get_object()
        #2.调用序列化器,传入序列化需要返回的数据对象
        ser = self.get_serializer(book)
        #3.返回结果
        return Response(ser.data)
        
    def update(self,request,pk):
        #1.接收前端的数据
        data = request.data
        #2.获取单一图书信息 get_object()从查询集中获取符合pk所对应的id的数据对象
        book = self.get_object()
        #3.序列化器
        ser = self.get_serializer(book,data=data)
        #4.验证
        ser.is_valid(raise_exception=Ture)
        #5.保存
        ser.save()
        #6.返回数据
        return Response(ser.data)
        
    def destroy(self,request,pk)
        #1.获取图书ID
        book = self.get_object()
        #2.逻辑删除
        book.is_delete = Ture
        #3.保存
        book.save()
        #4.返回数据
        return Response({})
        
{% endcodeblock %}



{% codeblock lang:python%}
import threading
g_number = 0

def update_number1(lock1):
    """子线程函数"""
    global g_number
    for i in range(1000000):
        with lock1:
            g_number += 1

def update_number2(lock1):
    """子线程函数"""
    global g_number
    for i in range(1000000):
        with lock1:
            g_number += 1
def main():
    lock1 = threading.Lock()
    thd1 = threading.Thread(target=update_number1,args=(lock1,))
    thd1.start()
    thd2 = threading.Thread(target=update_number2,args = (lock1,))
    thd2.start()
    thd2.join()
    thd1.join()
    print("获取全局变量的值是%s" % g_number)
if __name__ == '__main__':
    main()

{% endcodeblock %}