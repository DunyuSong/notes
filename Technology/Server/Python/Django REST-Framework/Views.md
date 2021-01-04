# Views
- views用于处理请求，可以通过request.GET.get('name', '')方式获取前端传递过来的数据。
- 

### Requests
- request.data: 用于获取前端传递的POST、PUT、PATCH数据。
- request.query_params: 用于获取前端传递的GET数据。
- request.user：获取当前用户
### Responses
- 

- serializers.data：中用于将序列化之后的数据返回给前端。


## QuerySet 查询操作
```
queryset = Pipeline.objects.all()[:];
# 根据条件进行查询。filter方法始终返回的是QuerySets，你可以简单的理解为Python列表，可迭代可循环可索引。
# queryset = Pipeline.objects.all().filter();
# 如果你确定你的检索只会获得一个对象，那么你可以使用Manager的get()方法来直接返回这个对象。
# queryset = Pipeline.objects.get(id=1)
# 更新所有2007年发布的entry的headline
Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')
```

