# Django ORM的一些操作技巧
## queryset
queryset是一个list，操作时注意判断list是否为空，再对里面的结果进行操作

通常对queryset可以使用filter，可以多个filter连用

Foreignkey 多对一，可以定义related_name，反引

