# Django的ORM使用

## 一、Django（MVT）的ORM介绍

1. ORM 全拼Object-Relation Mapping.
2. 中文意为 对象-关系映射.
3. 在MVC/MVT设计模式中的Model模块中都包括ORM.
4. 通俗的说，一个模型（model）映射到一个数据库表，使用ORM把对表的操作变成对对象的操作.
5. 关系图  

| ORM  | 数据库 |
| ---- | ------ |
| 类   | 数据表 |
| 对象 | 数据行 |
| 属性 | 字段   |



## 二、ORM的优势

1. 只需要面向对象编程, 不需要面向数据库编写代码.
2. 对数据库的操作都转化成对类属性和方法的操作.
3. 不用编写各种数据库的sql语句.
4. 实现了数据模型与数据库的解耦, 屏蔽了不同数据库操作上的差异.
5. 不用关注用的是mysql、oracle...等，通过简单的配置就可以轻松更换数据库, 而不需要修改代码.

## 三、Django ORM的劣势

1. 相比SQL来说，灵活度不够，限制强.
2. 根据对象的操作转换成SQL语句,根据查询的结果转化成对象, 在映射过程中有性能损失.
3. ORM用多了SQL语句就忘了，关系数据库相关技能退化...

## 四、models的常用方法
model定义：
file:models.py

```python
from django.db import models
 
class Test(models.Model):
    name = models.CharField(max_length=20)
    age = models.IntegerField()
```
### 4.1 查询数据

- get方法：models.Test.objects.get(name="MiKe")
- filter方法：models.Test.objects.filter(name="Mike")
- all方法：models.Test.objects.all()
- exclude方法：models.Test.objects.exclude(name="Mike")
- first方法：返回第一条数据
- last方法：返回最后一条数据
- exist方法：查询是否有数据，有返回True

__说明__：

- get方法返回一个经过Django ORM封装的Test类的对象或者抛出异常(DoesNotExist) 
- filter方法返回一个QuerySet或者'[]'(python3返回'<QuerySet []\>')
- all方法返回一个QuerySet
- exclude方法返回一个QuerySet
- QuerySet类型封装了filter、exclude、first、last、exists方法(也就是支持链式调用)
- filter、all、exclude方法都是惰性执行的，只会创建查询集，而不会之间访问数据库，只有调用数据的时候才会访问，查询过的结果集会进行缓存（可重用），再次调用时不会再访问数据库（tip：可以减少数据库的负荷）

例子1：

```shell
>>> list = BookInfo.objects.all()  # 不访问数据库
>>> print([book.book_name for book in list[0:2]])  # 访问数据库, 并缓存
>>> print([book.book_name for book in list[1:3]])  # 先访问查询集缓存, 不在缓存去访问数据库
```
例子2：

`re = Book.objects.filter(price=80)`
`re.exists() #会查询，但re不会被赋值，只有使用re变量的时候才会被赋值`

### 4.2 增加数据

方法一：实例化一个对象

a). test = Test(name='Mike')

b). test.save()

方法二：新建一个对象

a). test = Test()

b). test.name = 'Mike'

c). test.save()

方法三：create 或者 get_or_create方法

a). models.Test.objects.create(name="Mike")

b). models.Test.objects.get_or_create(name="Mike")

tip：使用第二种方法可以防止插入相同的数据

适用情景：一个年级有多个学生，我们需要增加某个年级的学生（不考虑学生重名的情况），如下所示：

models.py

```python
# models.py
from django.db import models
 
class Student(models.Model):
    name = models.CharField(max_length=20)
    grade = models.CharField(max_length=20)
```
```sh
[silvers@vm ~]$ python3
...
>>>
>>> data, status = models.Student.objects.get_or_create(name="Mike",defaults={grade="1"})
```
返回一个元组，第一个为Student对象，第二个为True或False,，新建时返回的是True，已经存在时返回False，可以一定程度上减少代码量

### 4.3 删除数据

1. delete方法单个删除：models.Test.objects.get(name="Mike").delete()
2. delete方法批量删除：models.Test.objects.filter(age=18).delete()
3. delete方法删除全部：models.Test.objects.all().delete()

### 4.4 修改数据

方法一：update方法

test = models.Test.objects.filter(name='Mike').update(age=19)

方法二：get获取一个对象

1. test = models.Test.objects.get(name='Mike')
2. test.age = 19
3. test.save()

方法三：update_or_create方法

1. 普通的方法更新或创建数据：
```python
defaults = {'age': 19}
try:
    obj = Test.objects.get(name='John', age=18)
    for key, value in defaults.items():
        setattr(obj, key, value)
    obj.save()
except Exception:
    new_values = {'name': 'John', 'age': 19}
    obj = Test(**new_values)
    obj.save()
```
2. 使用update_or_create方法：
```python
data,status = models.Test.objects.update_or_create(name='Mike',defaults={'age':19})
```
__说明__：
该方法返回一个二元组，第一个是一个创建的或者是被更新的对象，第二个是状态，是一个标识是否创建了新的对象的布尔值

### 4.5 使用Q和F
#### 4.5.1 F()的作用
操作数据表中的某列值，F()允许Django在未实际链接数据的情况下具有对数据库字段的值的引用，不用获取对象放在内存中再对字段进行操作，而是直接执行原生产sql语句操作。

例子1：

通常情况下我们在更新数据时需要先从数据库里将原数据取出后方在内存里，然后编辑某些属性，最后提交，代码如下：
views.py

```python
from models import Test

data = Test.objects.get(name='Mike')
data.age += 1
data.save()
#sql:update test set age=xxx where name='Mike';
```
例子2：

使用F()不用获取对象放在内存中再对字段进行操作，而是直接执行原生产sql语句操作，代码如下：
views.py

```python
from django.db.models import F
from models import Test

data = Test.objects.get(name="Mike")#惰性执行
data.age = F('age') + 1 #并没有获取age的值
data.save()
#sql:update test set age=age+1 where name='Mike';
```
__说明__：python做的唯一的事情就是通过Django的F()函数创建了一条SQL语句然后执行，需要注意的是使用F()更新数据之后需要重新加载数据来使数据库中的值与程序（内存）中的值项对应，可以使用data.refresh_from_db()或者之间再次赋值：data = models.Test.objects.get(name='Mike')

例子3：

使用update配合F()更新多条数据

views.py

```python
from models import Test

data = Test.objects.filter(age=18).update(age=F('age') + 1)
```
__说明__：这样做的效率比将其取到内存中后再一个个计算值再更新数据库的效率会提高非常多

#### 4.5.2 Q()的作用

对对象进行复杂查询，并支持&（and）,|（or），~（not）操作符（操作符作用于Q对象之间，而不是Q对象之内）

例子1：

当我们在查询的条件中需要组合条件时(例如两个条件“且”或者“或”)时，可以使用Q()查询对象，代码如下：
views.py

```python
from django.db.models import Q
from models import Test

q=Q(name__startswith="M")
p=Q(name__endswith='e')
data = Test.objects.filter(q,p)
```
这样就给filter传递了2个Q对象，逗号关系是&，当然我们可以使用符号&或者|将多个Q()对象组合起来传递给filter()，exclude()，get()等函数。当多个Q()对象组合起来时，Django会自动生成一个新的Q()

例子2：

Q查询和关键字查询组合，Q是位置参数，必须要在关键字参数之前（python语法），代码如下：

views.py

```python
from django.db.models import Q
from models import Test

data = Test.objects.filter(Q(name__contains='M'),age=18)
```
__说明__：Q()支持大多数filter支持的参数命名修饰（__xxx用法）

### 4.6 属性类型介绍

1. AutoField：自动增长的IntegerField，通常不用指定，不指定时Django会自动创建属性名为id的自动增长属性。
2. BooleanField：布尔字段，值为True或False。
3. NullBooleanField：支持Null、True、False三种值。
4. CharField(max_length=字符长度)：字符串。
5. TextField：大文本字段，一般超过4000个字符时使用,参数max_length表示最大字符个数。
6. IntegerField：整数。
7. DecimalField(max_digits=None, decimal_places=None)：十进制浮点数,参数max_digits表示总位数,参数decimal_places表示小数位数。
8. FloatField(max_digits=None, decimal_places=None)：浮点数,参数max_digits表示最大的位数,参数decimal_places表示小数位数。
9. DateField([auto_now=False | auto_now_add=False])：日期。
10. TimeField：时间，参数同DateField。参数auto_now表示每次保存对象时，自动设置该字段为当前时间，用于"最后一次修改"的时间戳，它总是使用当前日期，默认为false,
    参数auto_now_add表示当对象第一次被创建时自动设置当前时间，用于创建的时间戳，它总是使用当前日期，默认为false。
    注意：参数auto_now_add和auto_now是相互排斥的，组合将会发生错误。
11. DateTimeField：日期时间，参数同DateField。
12. FileField：上传文件字段。
13. ImageField：继承于FileField，对上传的内容进行校验，确保是有效的图片。

其他：Django还有邮箱、url、ip地址等属性字段，但是不经常使用，一般就用CharField就可以了，使用这些会增加异常的可能

注意：Django会自动为表创建主键字段
如果使用选项设置某属性为主键字段后，Django就不会再创建自动增长的主键字段，默认创建的主键字段为id，可以使用pk（primary key）代替

### 4.7 属性类型的常见关键字参数介绍

1. 外键引用（ForeignKey）字段类型——on_delete=value

   value的值有：

   - CASCADE：删除引用的对象时，也删除引用它的对象
   - PROTECT：禁止删除引用的对象。SQL等价物：RESTRICT。
   - SET_NULL：将引用设置为NULL（要求字段可以为空），当字段设置null=True才可以使用
   - SET_DEFAULT：设置默认值。只有当字段设置了default参数时才能使用 SQL等价物：SET DEFAULT。
   - SET(value 或者 函数返回值)：设置给定值。这个不是SQL标准的一部分，完全由Django处理。
   - DO_NOTHING：SQL等价物：NO ACTION。

2. to_field:设置要关联的表的字段

3. to:设置要关联的表

例子：

```python
class Test2():
	a = ForeignKey('Test',to_field="name")
```

该示例表示表Test2的a属性关联Test表的name属性

### 4.8 filter等查询方法参数的命名修饰介绍

- \_\_exact 精确等于 like ‘aaa’

- \_\_iexact 精确等于 忽略大小写 ilike ‘aaa’
- \_\_contains 包含 like ‘%aaa%’
- \_\_icontains 包含 忽略大小写 ilike ‘%aaa%’，但是对于sqlite来说，contains的作用效果等同于icontains。
- \_\_gt 大于
- \_\_gte 大于等于
- \_\_lt 小于
- \_\_lte 小于等于
- \_\_in 存在于一个list范围内
- \_\_startswith 以…开头
- \_\_istartswith 以…开头 忽略大小写
- \_\_endswith 以…结尾
- \_\_iendswith 以…结尾，忽略大小写
- \_\_range 在…范围内
- \_\_year 日期字段的年份
- \_\_month 日期字段的月份
- \_\_day 日期字段的日
- \_\_isnull=True/False
- \_\_isnull=True 与 __exact=None的区别

### 4.9 把models-objects转成json格式

方法一：使用values()
例子1：

```shell
>>> from models import Test
>>> data = list(Test.objects.filter(name='Mike').values('name','age')) #django2.0下.values()返回的时QuerySet对象，需要先转成list再取出字典
>>> data
{'name': 'Mike', 'age': 18}
```
__说明__：values()接收的是需要在dict里生成的键，该值不是模型里定义的属性，而是数据库里表的字段名字

方法二：使用model_to_dict方法
例子2：

```shell
>>> from django.forms.models import model_to_dict
>>> from models import Test
>>> obj = Test.objects.get(name='Mike')
>>> data = model_to_dict(obj)
>>> data
{'name': 'Mike', 'age': 18}
```
__说明__：这种方法能满足大部分的需求，且输出也较为合理，同时也提供两个参数fields和exclude来配置输出的字段，fileds指定需要输出的字段，exclude指定不需要输出的字段，这两个关键字参数都接收一个list。

（注：这里的字段是models里定义的属性）

方法三：使用Django的序列化工具serializers（序列化是将对象状态转换为可保持或传输的格式的过程）

```shell
>>> from django.core import serializers
>>> from models import Test
>>> import json
>>> obj_list = Test.objects.filter(age=18)
>>> data = serializers.serialize("json",obj_list)
>>> data
'[{'model': 'app.name', 'pk': 'id', 'fields': {'name': 'Mike', 'age': 18}},...]'
>>> data = json.loads(data)
>>> data
[{'model': 'app.name', 'pk': 'id', 'fields': {'name': 'Mike', 'age': 18}},...]
>>> data_list = []
>>> for item in data:
...     temp = item.get('fields')
...     data_list.append(temp)

>>> data_list
[{'name': 'Mike', 'age': 18},...]
```
__说明__：这个方法至少接收两个参数，第一个是你要序列化成为的数据格式，这里是‘json’(django支持xml、json、yaml这3种，其中yaml可能需要第三方库的支持)，第二个是要序列化的数据对象，数据通常是ORM模型的QuerySet，一个可迭代的对象，该方法还接收一个fields关键字(tuple or list)参数，用于序列化指定的字段，如fields=('name','age'),我们通常只需要结果里键fields对应的值。

注意：如果ORM模型具有自定义的字段，那么这个序列化工具无法正常工作，必须自己编写相应部分的序列化代码。

## 五、models的注意要点

1. 涉及到对models的操作（增加、删除、查询、修改），如果你不能保证你传入的值是合法的值，最好用try...except包括起来，不涉及到条件限制（传入参数）的可以不用异常处理，如models.Test.objects.all()这种

2. 使用print(str(models.Test.objects.all().query))可以查看执行Test.objects.all()时执行的sql语句

3. 使用models.Test.objects.filter(field1=xxx).values('field1','field2',...,'fieldn')时，values中的字段是数据库中实际的字段，而不是models定义的该表的模型的字段。

4. 如果field1是外键字段，则models.Test.objects.filter(field1__id=xxx)可以指定只关联到原表的id而不是整个model对象

5. get_or_create()方法有一个defaults关键字参数，用于指定当get失败需要create时，该表对应的models中其他属性的创建值，defaults是一个dict。

6. 当models的表定义没有主键时，如果要使用filter，应该根据数据库的表结构和自己的需求指定values字段，这样可以避免Django抛出找不到id的异常情况

7. django默认情况下每一个主表的对象都有一个是外键的属性，可以通过它查询到所有关于子表的信息，这个属性的名字就是子表的名称小写加上_set，比如有teacher表和student表，student表有一个外键指向teacher，那么可以用t = teacher.objects.get(id=1).student_set.all()就可以得到这个teacher的所有student，如果在定义student表的teacher属性的时候指定了related_name关键字参数的值，比如related_name='student_teacher',那么可以使用t = teacher.objects.get(id=1).student_teacher.all()获取这个teacher的所有student

8. 当一张表的多个字段指向同一张表时，会出错。因为系统无法知道，通过另一张表，访问xxx_set属性时访问的是哪个属性，这时就要为这些字段定义一个related_name属性，另外一张表访问这个表时，就会根据related_name的值来得到各个属性
9. 一般情况下ForeignKey('table_name')是默认关联到table_name的id的，可以通过to_field关键字参数改变这一默认行为。to_field的作用是指定被关联对象的用于关联的字段（在Django的ORM中，这个被关联的字段必须是有unique属性的字段）。可以解决数据库表的外键关联的字段不是表的id的情况，但是注意，赋值的时候还是需要赋一个实例，而不是一个字段值
10. 一般来说属性f = ForeignKey('A')在数据库生成表的时候，该模型的f属性会对应数据库该表的f_id字段，如果一定要让数据库表的字段也是f，那么可以在定义属性f的时候加一个关键字参数db_column='f',这样数据库里面该模型对应的表的字段就是f了

## 六、Django中使用原生SQL操作数据库
尽管ORM很好用，单很多情况下有很大局限性，常常还是会需要用到原生sql查询。下面介绍几个常用的django原生sql查询封装的方法

```python
def insert(sql,params):
	result = {'status': True}
	cursor = connection.cursor()
	try:
		rowcount = cursor.execute(sql,params)
	except Exception as e:
		result['status'] = False
	else:
		if not rowcount:
			result['status'] = False
		else:
			transation.commit()
	finally:
		cursor().close()
		return result
		


def query(sql, params):
    """
    返回一行数据
    :param sql: sql语句
    :param params: sql语句参数
    :return: 例如：{"id": id, "username": 'username', "first_name": 'first_name'}
    """
    result = {'status':True}
    cursor = connection.cursor()
    try:
        cursor.execute(sql, params)
    except Exception as e:
        result['status'] = False
        print str(e.message)
    else:
        if not rowcount:
            result['status'] = False
        else:
            desc = cursor.description
            data_dict = dict(zip([col[0] for col in desc], cursor.fetchone()))
            result['data'] = data_dict
    finally:
    	cursor.close()
        return result


def queryAll(sql):
    """
    返回全部数据
    :param sql: sql语句
    :return: 例如：[{"id": id, "username": 'username', "first_name": 'first_name'}]
    """
    result = {'status':True}
    cursor = connection.cursor()
    try:
    	cursor.execute(sql)
    except Exception as e:
    	result['status'] = False
    else:
    	if not rowcount:
            result['status'] = False
    	else:  
            desc = cursor.description
            object_list = [
		        dict(zip([col[0] for col in desc], row)) for row in cursor.fetchall()
    		]
    finally:
        cursor.close()
        return object_list
```
