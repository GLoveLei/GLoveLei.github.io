---
title: '无限极自关联'
date: '2021/2/4 14:59:21'
categories:
   - python
tags:
   - 无极限自关联
toc_level: 3
version: 3
---
![cover](images/river.png)


#### 一、什么是无限极自关联
无限极分类其实就是一棵树，所有的节点作为一个存储元素。每个节点可以有任意个孩子节点，且只有一个父节点。

#### 二、后端部署
##### models.py
```js
class Admin(models.Model):
    name = models.CharField(max_leng=20) 
    uid = models.ForeignKey(to='self', on_delete=models.SETNULL, default=None) # 自关联
```

##### serializers.py
```js
class AdminSer(ModelSerializer):
    class Meta:
        model = Path
        fields = '__all__'
```

##### views.py
```js
# 无限自关联递归函数

def function(id):
    # 接收id 查询结果集
    query = User.objects.filter(uid=id)
    # 进行序列化
    admin = UserSer(query, many=True).data
    # 在序列化结果中添加空列表
    admin['list'] = []
    # 遍历结果集，递归调用并将调用结果存入上级列表
    for i in admin:
        admin['list'].append(functions(i.uid))

    # 返回结果集
    return admin

class Admin(APIView):
    def get(self, request):
        # 定义列表
        admin_list = []
        # 查取所有信息
        admin = Path.objects.all()
        # 遍历开始递归查询
        for i in admin:
            admin_list.append(function(i.id))
        
            retrun Response(admin_list)
```

#### 三、总结
   部署一个父节点，可以有多个子节点的加入，然后进行遍历数据，关联自己进行查询，最后进行添加