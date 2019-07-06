[TOC]



# 一、Django的介绍

request介绍：https://www.cnblogs.com/zhaijunming5/p/7772772.html

![img](https://images2015.cnblogs.com/blog/948404/201608/948404-20160830212049246-1153317231.jpg)

![image.png-75.4kB](http://static.zybuluo.com/feixuelove1009/iig6cjapfrh5q5mnu87a9v4r/image.png)

Django相当于一个后台服务器

urls：网址入口，关联到对应的views.py中的一个函数，访问网址就对应一个函数。  怎么将网址和函数联系到一起（request）

views.py：定义一个函数，第一个参数为request，处理浏览器网址，处理http请求，是从urls.py中对应过来，通过渲染templates中的网页可以将显示内容，输出的网页。输出函数HttpResponse 

，其相当于print的作用。

也就是在view中定义一个函数处理http请求，返回响应到前端页面。

models.py：与数据库有关

一个Python的类就是一个模型，代表数据库中的一张数据表。

在views进行查询，删除，增加数据。在views中可以进行实例化。

因为HTML中的input会把值传递给后台，request对象存放着从HTML来的数据

**templates 文件夹**

views.py 中的函数渲染templates中的Html模板，得到动态内容的网页，当然可以用缓存来提高速度。



# 二、环境部署

https://blog.csdn.net/godot06/article/details/81079064



# 三、Django基础知识

通过一个留言板功能来了解

## 3.1 Django目录结构

- DjangoGetStarted(文件夹)：
  - setting.py： 项目全局配置文件
  - urls.py： 主要的urls配置入口
  - wsgi.py： 是Django启动需要的文件。
- templates(文件夹)： 放置html文件
- manage.py： 启动Django需要的主要文件。(主要的Django命令都通过manage.py运行)

* apps：将App整合到一起

​           通过 `Tools -> Run manage.py Task`创建app：startapp message

   *  * message
        *    admin：
        * apps
        * models
        * tests
        * views

* static：存在CSS，js，图片等静态文件

* log：存放日志

* media：存放用户上传的图片等资源

## 3.2 配置表单页面

涉及到了setting，message/view(getform),  url.

Django从请求到响应的整个完整流程。为我们开发在线教育平台打下基础。

> 具体的业务：填写信息 -> 然后点击提交 ->数据被存储到数据库。

![img](http://myphoto.mtianyan.cn/blog/180107/ihf44icl9d.png?imageslim)

上述为整个项目的流程图

### 将HTML整合到项目中

* 将form.html文件放入到templates中
* static中新建一个CSS文件夹专门用于存放CSS文件，新建一个CSS文件style.css。然后将form.html文件中的style部分剪切到style.css实现HTML和css文件分离
* 在`form.html`新建`<link>`来引入css。

```
<link rel="stylesheet" href="/static/css/style.css">
```

这样就实现了文件分离和地址修改

### 配置Django连接mysql数据库

改setting中的database与mysql连接

tools中run manager.py task

```
makemigrations
migrat

```

（makemigrations与migration的定义与作用）

在Navicat中会生成很多表。

### 配置form页面展示出来（倒序配置的）

1. 配置urls

   ```python
   from message.views import getform
   
   urlpatterns = [
       url(r'^admin/', admin.site.urls),
       url(r'^form/$', getform) #这行是新增加的.
   ]
   ```

   getform要到message/views.py去设置

2. message/views.py进行getreform配置

   ```python
   
   def getform(request):
       return render(request, 'form.html')
   ```

3. setting中的template中的dir设置

   ```python
   # 指明我们的templates目录路径
   'DIRS': [os.path.join(BASE_DIR, 'templates')]
   ```

4. setting中的静态文件的设置

   ```python
   STATIC_URL = '/static/'
   STATICFILES_DIRS = [
       os.path.join(BASE_DIR, 'static')
   ]
   ```

指明查找的根路径。

## 3.3 django orm介绍与model设计

### 原生sql与orm

使用Django的orm来使我们不再写原生复杂的sql语句，而是像使操作数据库像使用类和类属性一样

### 创建我们的models

在message的models中进行书写

```Python
#出现中文的地方一定要加
# -*- coding: UTF-8 -*- 

# 继承于django.db.models.Model
class UserMessage(models.Model):
    # 设置最大长度，verbose_name在后台显示字段会用到
    name = models.CharField(max_length=20, verbose_name=u"用户名")
    # Django提供内置的邮箱字段会帮忙验证` default_validators = [validators.validate_email]`
    email = models.EmailField(verbose_name=u"邮箱")
    address = models.CharField(max_length=100, verbose_name=u"联系地址")
    message = models.CharField(max_length=500, verbose_name=u"留言信息")


    class Meta:
        verbose_name = u"用户留言信息"
        # db_table ，这里我们让它自动生成所以不用指定
```

需要去补充django 的基础知识

注意：新建的App都需要到setting里进行

### 在setting中注册我们的app

```python
`INSTALLED_APPS`
[
    前面的不用变，后面新增下一行
    'message'
]
```

然后就可以makemigrations message 和migrate message。

之后可以看到我们的数据表已经创建成功。默认数据表名称为`app名称_类名转换为小写`

### models讲解

用于设计数据库的模型（？）

* 数据类型

  除了普通的对应数据库的字段类型如charfield外。还有很多高级类型，如EmailField提供email验证。

  ```
  models.ForeignKey     # 外键
  models.DateTimeField  # 时间字段
  models.IntegerField   # 整型
  models.IPAddressField # IP地址
  models.FileField      # 上传文件
  models.ImageField     # 图片
  ```

* 字段参数

  * > **非主键部分**

  ```python
  name = models.CharField(max_length=20,null=True,blank=True, verbose_name=u"用户名")
  ```

> `CharField`必须指明默认最大长度。`null=True,blank=True`指明字段可以为空 defalut = " "`指定默认值。

​          **主键部分**

```python
object_id = models.CharField(primary_key=True, max_length=50,default="", verbose_name="主键")

```

id 是自动生成的，也可以向上述那样自定义主键

**注意1**：**CharField必须指明最大长度**

**注意2：****object_id必须有默认值**

操作：输入2可以退出makemigrations message



* meta信息

  > ```
  > `class Meta，内嵌于 UserMessage 这个类的定义中如果 class Publisher 是顶格的，那么 class Meta 在它之下要缩进4个空格－－按 Python 的传统你可以在任意一个 模型 类中使用 Meta 类，来设置一些与特定模型相关的选项。如：设置ordering = ['name']，默认地都会按 name 字段排序`
  > ```

Meta信息中我们可以指定常见的类型：

```
db_table = "user_meassage"
```

自定义后生成表，表名会与我们的保持一致。而不会前缀`appname`如：`message_`

> 这里因为我们已经生成过了，就不要做验证改变表名了。

```
ordering = '-object_id'
```

ordering指定默认排序字段,如：就会以object_id倒序

```
verbose_name_plural = u"用户留言信息"
```

verbose_name_plural：复数信息，便于人阅读。否则会在后台显示`用户留言信息s`

## 3.4 Django model的增删改

**主要在我们的view里实现数据库的查询，增加以及删除操作**

### 查询数据

1. 在数据库中加入数据（让数据库有数据用的，现实有数据是不需要这步的）

2. 将设计的App的model  import到我我们的message/views.py中。model设计的模型加载进来

   ```python
   from .models import UserMessage
   ```

   将我们刚才创建的model，import进来。`.`代表是与当前同级的目录。

3. 在getform中添加代码。（我们的查询方法）

   ```Python
   # UserMessage默认的数据管理器objects。
       # 方法all()是将所有数据返回成一个queryset类型(django的内置类型),即all_message是一个queryset类型
       # object代表默认的数据表管理器
       all_message = UserMessage.objects.all()
       for message in all_message:           # 我们可以对于all_message进行遍历操作
           print message.name      # 每个message实际就是一个UserMessage类的对象（这时我们就可以使用对象的相关方法）。
   ```

   这里将.all( )换成filter( )可以取出指定的要求值

   ```Python
   all_message = UserMessage.objects.filter(name=' mtianyan', address='西安')
   ```

   4.debug调试观察：f6是进入下一条的意思（试验）

   在下方的值窗口内可以看到message的值。说明我们成功取到了数据库的值。

   在窗口可以看到打印的name

   

   ### 数据库增加数据

   #### 直接数据存入数据库

   * 实例化一个对象
   * 增加属性
   * 然后save。因为我们是继承了model，可以应用model中的save方法

   ```
   # 存储部分
   
      # 首先实例化一个对象
      user_message = UserMessage()
   
      # 为对象增加属性
      user_message.name = "mtianyan2"
      user_message.message = "blog.mtianyan.cn"
      user_message.address = "西安"
      user_message.email = "1147727180@qq.com"
      user_message.object_id = "efgh"
   
      # 调用save方法进行保存
      user_message.save()
   ```

   直接run到Navicat中去查看。

#### 从html提交中取到数据并保存到数据库中

* **如何取到数据**

  templates/message_form.html中：

  ![mark](http://myphoto.mtianyan.cn/blog/180107/glha3bHaC8.png?imageslim)

  > method是post。action就是指向我们在urls.py中配置的`/form/`
  > **前面必须加斜杠指根路径下form**
  > 里面的input会自动把值传递给后台：这时我们就可以在getform中取到刚才传递过来的值

**注意：在我们的html页面中要加入csrf_token**，**不然容易出现403错误。这是Django的安全机制**

通过debug调试可以发现request对象下的POST中存放了我们的数据。内容如下

> ```
> <QueryDict: {u'message': [u'\u54c8\u54c8'], u'address': [
> u'\u897f\u5b89\u5e02'], u'csrfmiddlewaretoken': [
> u'uIYSMOTWPJBPOPucRwd3uDaWtCzeEaem'], u'name': [
> u'\u5929\u6daf\u660e\u6708\u7b19'], u'email': [u'1147727180@qq.com']}>
> ```

数据以dict：key-value 形式存储 key是由如下图html中的name所决定对应的。

![img](http://myphoto.mtianyan.cn/blog/180107/c2lIdeH2A1.png?imageslim)

这样我们可以通过`request.POST`中数据取出，存入`user_message`对象中

```python
# html表单部分

   # 此处对应html中的method="post"，表示我们只处理post请求
   if request.method == "POST":
       # 就是取字典里key对应value值而已。取name，取不到默认空
       name = request.POST.get('name', '')
       message = request.POST.get('message', '')
       address = request.POST.get('address', '')
       email = request.POST.get('email', '')

       # 实例化对象
       user_message = UserMessage()

       # 将html的值传入我们实例化的对象.
       user_message.name = name
       user_message.message = message
       user_message.address = address
       user_message.email = email
       user_message.object_id = "ijkl"

       # 调用save方法进行保存
       user_message.save()
```

### 删除数据库

```python
# 方法2 :filter取出指定条件值，逗号代表and 必须同时满足两个条件才返回。
all_message = UserMessage.objects.filter(name='mtianyan', address='西安')

# 我的数据库里保存着可以匹配到该条数据的一行。

# 删除操作：使用delete方法删除all_message

all_message.delete()

    for message in all_message:
        # 删除取到的message对象
        message.detele()
        # print message.name
```



## 3.5 django url templates配置

* 取出数据

* 将数据回填至html中

* template模板渲染中的一些用法

* URI的别名设置技巧

* URL先后顺序问题

# 四、 需求分析与model设计

## 4.1 Python3.6以及Django2.0搭建的一些注意事项

## 4.2 Django APP的设计

授课机构提供讲师录制课程，学员完成在线学习。

- 全局头部：用户消息 & 个人中心: 没有登录时，就是登录注册
- 对于公开课，授课讲师，授课机构进行搜索。
- 轮播图，课程，机构，页脚
- 公开课：分页公开课，右边热门推荐。
- 点进课程：课程详情页。详情: 后台富文本。右边是课程机构的介绍。收藏 或学习
- 章节信息 & 课程资源下载 & 评论
- 授课讲师: 授课讲师列表页, 讲师排行榜。分页。
- 点进讲师: 看到课程。
- 授课机构: 类别筛选，机构性质，所在地区 & 排序。用户提交表单，我要学习, 机构排名.
- 个人中心: 修改密码, 修改头像, 个人信息, 我的课程, 我的收藏, 我的消息。

app大致会有`用户模块`,`课程模块`,`授课教师`与`授课机构`。

## 4.3 新建项目

创建虚拟环境

```
`mkvirtualenv mxonline2`
```

安装Django

```
pip install django==1.9.8
```

新建项目

虚拟环境下安装mysql

> pip install 

setting 中为改变数据库配置

进行数据库初始化（对数据库进行迁移）

```
`makemigrationsmigrate`
```

> `migrations`是Django保存模型修改记录的文件，这些文件保存在磁盘上。在例子中，它就是`polls/migrations/0001_initial.py`，你可以打开它看看，里面保存的都是人类可读并且可编辑的内容，方便你随时手动修改。
>
> 接下来有一个叫做`migrate`的命令将对数据库执行真正的迁移动作。但是在此之前，让我们先看看在migration的时候实际执行的SQL语句是什么。有一个叫做`sqlmigrate`的命令可以展示SQL语句

run Django可以运行

## 4.4 自定义userprofile

![mark](http://myphoto.mtianyan.cn/blog/180108/mghmfD45hJ.png?imageslim)

- 创建App

  > tool  中的run manager.py task
  >
  > Terminal下的startAPP app名

切记将新建的APP放到setting中

- APP中书写models 生成userProfile

  ```python
  from django.contrib.auth.models import AbstractUser
  
  
  class UserProfile(AbstractUser):
      # 自定义的性别选择规则
      GENDER_CHOICES = (
          ("male", u"男"),
          ("female", u"女")
      )
      # 昵称
      nick_name = models.CharField(max_length=50, verbose_name=u"昵称", default="")
      # 生日，可以为空
      birthday = models.DateField(verbose_name=u"生日", null=True, blank=True)
      # 性别 只能男或女，默认女
      gender = models.CharField(
          max_length=5,
          verbose_name=u"性别",
          choices=GENDER_CHOICES,
          default="female")
      # 地址
      address = models.CharField(max_length=100, verbose_name="地址", default="")
      # 电话
      mobile = models.CharField(max_length=11, null=True, blank=True)
      # 头像 默认使用default.png  因为Image字段需要用到pillow所以需要安装该库
      # CharField必须有max_length, Imagefield实际也是charfield所以也要有max_length
      image = models.ImageField(
          upload_to="image/%Y/%m",
          default=u"image/default.png",
          max_length=100
      )
  
      # meta信息，即后台栏目名
      class Meta:
          verbose_name = "用户信息"
          verbose_name_plural = verbose_name
  
      # 重载Unicode方法，打印实例会打印username，username为继承自abstractuser
      def __unicode__(self):
          return self.username
  ```

  ？ 重载的原因：

- ```python
  # 此处重载是为了使我们的UserProfile生效
  AUTH_USER_MODEL = "users.UserProfile"
  ```

​     在setting中，为什么要有这一步。

- 对数据库进行迁移，生成userProfile的表



## 4.5 models设计

 项目的开发都是从models设计开始，后台的管理和前端的渲染无非就是对数据库的增删改查，所以models设计的好坏对整个项目的开发起着至关重要的因素。

![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180318005530581-838502279.png)

![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180318005751329-546802093.png)

![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180318005655700-1594231420.png)

**解决循环引用: 分层设计**

目前已有app：users courses organization

另外一个app高于这些app的层级。`operation`.上一层app可以import下层的app。

![mark](http://myphoto.mtianyan.cn/blog/180109/kmDB89IcFF.png?imageslim)

**创建四个APP并放到一个文件夹下**

operation courses organization users

切记要放到settings里。

### users model.py的设计

前面已经结束了uersprofile的设计。

user中还需要添加的表(这些功能比较独立):

- EmailVerifyRecord - 邮箱验证码
- Banner - 轮播图



```python
    class EmailVerifyRecord(models.Model):
        SEND_CHOICES = (
            ("register", u"注册"),
            ("forget", u"找回密码")
        )
        code = models.CharField(max_length=20, verbose_name=u"验证码")
        # 未设置null = true blank = true 默认不可为空
        email = models.EmailField(max_length=50, verbose_name=u"邮箱")
        send_type = models.CharField(choices=SEND_CHOICES, max_length=10)
        # 这里的now得去掉(),不去掉会根据编译时间。而不是根据实例化时间。
        send_time = models.DateTimeField(default=datetime.now)

        class Meta:
            verbose_name = "邮箱验证码"
            verbose_name_plural = verbose_name

# 轮播图model


class Banner(models.Model):
    title = models.CharField(max_length=100, verbose_name=u"标题")
    image = models.ImageField(
        upload_to="banner/%Y/%m",
        verbose_name=u"轮播图",
        max_length=100)
    url = models.URLField(max_length=200, verbose_name=u"访问地址")
    # 默认index很大靠后。想要靠前修改index值。
    index = models.IntegerField(default=100, verbose_name=u"顺序")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"轮播图"
        verbose_name_plural = verbose_name
```

### course model.py的设计

* 课程本身需要一张表
* 点进去之后点击开始学习。
* 课程基本信息需要一张表, 章节表与课程表存在(一个课程对应多个章节)
* 章节表中：章节的名称。 章节与视频(一个章节对应多个视频)

结构: 课程本身–(一对多)>章节-(一对多)->视频信息

资源下载放在课程里面的。一个课程对应多个资源

共四张表：课程本身–(一对多)>章节-(一对多)->视频信息 & 资源表

![mark](http://myphoto.mtianyan.cn/blog/180109/Kec5994359.png?imageslim)

```python
# 课程信息表
class Course(models.Model):
    DEGREE_CHOICES = (
        ("cj", u"初级"),
        ("zj", u"中级"),
        ("gj", u"高级")
    )
    name = models.CharField(max_length=50, verbose_name=u"课程名")
    desc = models.CharField(max_length=300, verbose_name=u"课程描述")
    # TextField允许我们不输入长度。可以输入到无限大。暂时定义为TextFiled，之后更新为富文本
    detail = models.TextField(verbose_name=u"课程详情")
    degree = models.CharField(choices=DEGREE_CHOICES, max_length=2)
    # 使用分钟做后台记录(存储最小单位)前台转换
    learn_times = models.IntegerField(default=0, verbose_name=u"学习时长(分钟数)")
    # 保存学习人数:点击开始学习才算
    students = models.IntegerField(default=0, verbose_name=u"学习人数")
    fav_nums = models.IntegerField(default=0, verbose_name=u"收藏人数")
    image = models.ImageField(
        upload_to="courses/%Y/%m",
        verbose_name=u"封面图",
        max_length=100)
    # 保存点击量，点进页面就算
    click_nums = models.IntegerField(default=0, verbose_name=u"点击数")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"课程"
        verbose_name_plural = verbose_name


# 章节
class Lesson(models.Model):
    # 因为一个课程对应很多章节。所以在章节表中将课程设置为外键。
    # 作为一个字段来让我们可以知道这个章节对应那个课程
    course = models.ForeignKey(Course, verbose_name=u"课程")
    name = models.CharField(max_length=100, verbose_name=u"章节名")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"章节"
        verbose_name_plural = verbose_name


# 每章视频
class Video(models.Model):
    # 因为一个章节对应很多视频。所以在视频表中将章节设置为外键。
    # 作为一个字段来存储让我们可以知道这个视频对应哪个章节.
    lesson = models.ForeignKey(Lesson, verbose_name=u"章节")
    name = models.CharField(max_length=100, verbose_name=u"视频名")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"视频"
        verbose_name_plural = verbose_name


# 课程资源
class CourseResource(models.Model):
    # 因为一个课程对应很多资源。所以在课程资源表中将课程设置为外键。
    # 作为一个字段来让我们可以知道这个资源对应那个课程
    course = models.ForeignKey(Course, verbose_name=u"课程")
    name = models.CharField(max_length=100, verbose_name=u"名称")
    # 这里定义成文件类型的field，后台管理系统中会直接有上传的按钮。
    # FileField也是一个字符串类型，要指定最大长度。
    download = models.FileField(
        upload_to="course/resource/%Y/%m",
        verbose_name=u"资源文件",
        max_length=100)
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"课程资源"
        verbose_name_plural = verbose_name

```

### organization model.py的设计

课程是属于机构的, 机构有机构类别，城市等字段。讲师实体。
我要学习的提交表单会与用户关联，存放在机构。

![mark](http://myphoto.mtianyan.cn/blog/180109/K1faIbEA3A.png?imageslim)

```python
# 城市字典
class CityDict(models.Model):
    name = models.CharField(max_length=20, verbose_name=u"城市")
    # 城市描述：备用不一定展示出来
    desc = models.CharField(max_length=200, verbose_name=u"描述")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"城市"
        verbose_name_plural = verbose_name


# 课程机构
class CourseOrg(models.Model):
    name = models.CharField(max_length=50, verbose_name=u"机构名称")
    # 机构描述，后面会替换为富文本展示
    desc = models.TextField(verbose_name=u"机构描述")
    click_nums = models.IntegerField(default=0, verbose_name=u"点击数")
    fav_nums = models.IntegerField(default=0, verbose_name=u"收藏数")
    image = models.ImageField(
        upload_to="org/%Y/%m",
        verbose_name=u"封面图",
        max_length=100)
    address = models.CharField(max_length=150, verbose_name=u"机构地址")
    # 一个城市可以有很多课程机构，通过将city设置外键，变成课程机构的一个字段
    # 可以让我们通过机构找到城市
    city = models.ForeignKey(CityDict, verbose_name=u"所在城市")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"课程机构"
        verbose_name_plural = verbose_name


# 讲师
class Teacher(models.Model):
    # 一个机构会有很多老师，所以我们在讲师表添加外键并把课程机构名称保存下来
    # 可以使我们通过讲师找到对应的机构
    org = models.ForeignKey(CourseOrg, verbose_name=u"所属机构")
    name = models.CharField(max_length=50, verbose_name=u"教师名称")
    work_years = models.IntegerField(default=0, verbose_name=u"工作年限")
    work_company = models.CharField(max_length=50, verbose_name=u"就职公司")
    work_position = models.CharField(max_length=50, verbose_name=u"公司职位")
    points = models.CharField(max_length=50, verbose_name=u"教学特点")
    click_nums = models.IntegerField(default=0, verbose_name=u"点击数")
    fav_nums = models.IntegerField(default=0, verbose_name=u"收藏数")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"教师"
        verbose_name_plural = verbose_name

```



### operation model.py的设计

```python
# 用户我要学习表单
class UserAsk(models.Model):
    name = models.CharField(max_length=20, verbose_name=u"姓名")
    mobile = models.CharField(max_length=11, verbose_name=u"手机")
    course_name = models.CharField(max_length=50, verbose_name=u"课程名")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"用户咨询"
        verbose_name_plural = verbose_name


# 用户对于课程评论
class CourseComments(models.Model):

    # 会涉及两个外键: 1. 用户， 2. 课程。import进来
    course = models.ForeignKey(Course, verbose_name=u"课程")
    user = models.ForeignKey(UserProfile, verbose_name=u"用户")
    comments = models.CharField(max_length=250, verbose_name=u"评论")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"评论时间")

    class Meta:
        verbose_name = u"课程评论"
        verbose_name_plural = verbose_name


# 用户对于课程,机构，讲师的收藏
class UserFavorite(models.Model):
    # 会涉及四个外键。用户，课程，机构，讲师import
    TYPE_CHOICES = (
        (1, u"课程"),
        (2, u"课程机构"),
        (3, u"讲师")
    )

    user = models.ForeignKey(UserProfile, verbose_name=u"用户")
    # course = models.ForeignKey(Course, verbose_name=u"课程")
    # teacher = models.ForeignKey()
    # org = models.ForeignKey()
    # fav_type =

    # 机智版
    # 直接保存用户的id.
    fav_id = models.IntegerField(default=0)
    # 表明收藏的是哪种类型。
    fav_type = models.IntegerField(
        choices=TYPE_CHOICES,
        default=1,
        verbose_name=u"收藏类型")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"评论时间")

    class Meta:
        verbose_name = u"用户收藏"
        verbose_name_plural = verbose_name


# 用户消息表
class UserMessage(models.Model):
        # 因为我们的消息有两种:发给全员和发给某一个用户。
        # 所以如果使用外键，每个消息会对应要有用户。很难实现全员消息。

        # 机智版 为0发给所有用户，不为0就是发给用户的id
    user = models.IntegerField(default=0, verbose_name=u"接收用户")
    message = models.CharField(max_length=500, verbose_name=u"消息内容")

    # 是否已读: 布尔类型 BooleanField False未读,True表示已读
    has_read = models.BooleanField(default=False, verbose_name=u"是否已读")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"用户消息"
        verbose_name_plural = verbose_name


# 用户课程表
class UserCourse(models.Model):
    # 会涉及两个外键: 1. 用户， 2. 课程。import进来
    course = models.ForeignKey(Course, verbose_name=u"课程")
    user = models.ForeignKey(UserProfile, verbose_name=u"用户")
    add_time = models.DateTimeField(default=datetime.now, verbose_name=u"添加时间")

    class Meta:
        verbose_name = u"用户课程"
        verbose_name_plural = verbose_name

```

# 五、通过Xadmin快速搭建后台管理系统

**记住：**每次修改模型后都要进行makemigrations和migrate。将改变迁移到数据库中

## 5.1 admin介绍

* 后台管理系统特点： 权限管理，少前端样式(样式一般不是很看重)，快速开发

django的后台管理系统是一套智能的管理系统。
django的杀手锏之一就是admin管理系统。

* createsuperuser

  terminal中Python manager createsuperuser、

* setting设置语言以及时区

**model注册：注册UserProfile进来：**

组对应数据表: auth_group

在Django的admin中可以把上章的表都注册进来。对于表进行任意的增删改查。

默认其实会把user也注册进来的，但是因为我们通过userProfile覆盖了user。所以没有显示。

users/admin.py中添加代码:

```python
# encoding: utf-8

# 因为同一个目录，所以可以直接.models
from .models import UserProfile


# 写一个管理器:命名, model+Admin
class UserProfileAdmin(admin.ModelAdmin):
    pass


# 将UserProfile注册进我们的admin中, 并为它选择管理器
admin.site.register(UserProfile,UserProfileAdmin)
```

之后会看到：

![img](http://myphoto.mtianyan.cn/blog/180109/c58F6dg549.png?imageslim)

`USERS` 是用户所在表名称。

进入页面可以看到Django为我们把每个不同类型的字段生成了不同的前端样式。

![mark](http://myphoto.mtianyan.cn/blog/180109/761bBAhbJl.png?imageslim)

**注：**migration中的`initial.py` 中要进行修改，把所有的App删掉。

## 5.2 xadmin安装

1. 安装：pip install xadmin（选择5的路由安装）

   或者安装方式2（推荐）：源码安装，<https://github.com/sshwsfc/xadmin>下载 解压后将Xadmin文件夹复制到我们的项目中， package建一个extra_apps，Mark为source root，方便cmd需要路径修改。（记得卸载之前安装的）

2. 注册app：

   settings.py的INSTALLED_APPS中加上‘xadmin‘和'crispy_forms'

3. 配置路由

   urls中默认admin指向Xadmin （将我们原来写的user/admin.py中代码删除掉。）

4. 重新生成数据库：

   ```
   makemigrations
   migrate
   ```

5. 设置成中文（设置过了）
6. 创建一个管理员用户（设置过了

## 5.3 user App的model注册

Django的admin, Xadmin和其他系统区别

> 不像php等其他语言是一个功能模块一个功能设计的。
> Django是对于每张表增删改查的管理器，我们可以在增删改成的基础上加上我们自己的后台逻辑。
> 因此某种程度可以说他是不依赖于具体业务的。不管啥系统后台都是由表组成。

不依赖于后台逻辑，又可以加上逻辑。

UserProfile已经被自动注册进去了，我们从验证码开始注册。

我们需要新建一个`adminx.py`文件，Xadmin会自动搜寻这种命名的文件。

1. **在user下建立adminx.py用于model注册**

   ```python
   import xadmin
   
   from .models import EmailVerifyRecord
   
   
   # 创建admin的管理类,这里不再是继承admin，而是继承object
   class EmailVerifyRecordAdmin(object):
       pass
   #将model与admin管理器进行关联注册
   xadmin.site.register(EmailVerifyRecord, EmailVerifyRecordAdmin)
   ```

2. **完善功能，增加显示字段，搜索和过滤**

   ```python
   
   # users/adminx.py
   
   import xadmin
   
   from .models import EmailVerifyRecord
   
   #xadmin中这里是继承object，不再是继承admin
   class EmailVerifyRecordAdmin(object):
       # 显示的列
       list_display = ['code', 'email', 'send_type', 'send_time']
       # 搜索的字段，不要添加时间搜索
       search_fields = ['code', 'email', 'send_type']
       # 过滤
       list_filter = ['code', 'email', 'send_type', 'send_time']
   
   xadmin.site.register(EmailVerifyRecord,EmailVerifyRecordAdmin)
   
   ```

3. **users中Banner也注册进去**

   ```python
   class BannerAdmin(object):
       list_display = ['title', 'image', 'url','index', 'add_time']
       search_fields = ['title', 'image', 'url','index']
       list_filter = ['title', 'image', 'url','index', 'add_time']
   
   
   xadmin.site.register(Banner,BannerAdmin)
   ```



小tip1：新建py文件的初始化模板，file and code template

小tip2：verbose_name 与verbose_name_plural

邮箱验证码这几个字就是我们代码中Meta中verbose_name定义的:

```
class Meta:
    verbose_name = "邮箱验证码"
    verbose_name_plural = verbose_name
```

`verbose_name_plural`是`verbose_name`的复数形式。

小tip3：解决EmailVerifyRecord object显示

> **全部**(没写出的请自行修改)model，py2:重载`__unicode` py3:重载`__str__`

```
# 重载Unicode方法使后台不再直接显示object
def __unicode__(self):
    return '{0}({1})'.format(self.code,self.email)
```

小tip4：xadmin导出CSV乱码问题：将`charset=utf-8` 改为`charset=gbk`

小tip5：xadmin导出xml报错



**总结：**要在xadmin里注册一个model。先要创建一个admin管理类，这个管理类名为：model名+Admin。

然后要将model与我们的admin管理类相关联。

最后，再完善一些功能：显示的列，list_display;搜索的字段search_fields；过滤功能list_filter



<!--from .models import EmailVerifyRecord-->该类导入不进去

## 5.4 剩余 App的model注册

### course APP的model注册

```python
# -*- coding: utf-8 -*-
__author__ = 'amy'
__date__ = '2019/3/5 21:35'

from .models import Course, Lesson, Video, CourseResource
import xadmin


# Course的admin管理器
class CourseAdmin(object):
    list_display = [
        'name',
        'desc',
        'detail',
        'degree',
        'learn_times',
        'students']
    search_fields = ['name', 'desc', 'detail', 'degree', 'students']
    list_filter = [
        'name',
        'desc',
        'detail',
        'degree',
        'learn_times',
        'students']


class LessonAdmin(object):
    list_display = ['course', 'name', 'add_time']
    search_fields = ['course', 'name']

    # __name代表使用外键中name字段
    list_filter = ['course__name', 'name', 'add_time']
    

class VideoAdmin(object):
    list_display = ['lesson', 'name', 'add_time']
    search_fields = ['lesson', 'name']
    list_filter = ['lesson', 'name', 'add_time']


class CourseResourceAdmin(object):
    list_display = ['course', 'name', 'download', 'add_time']
    search_fields = ['course', 'name', 'download']
    # __name代表使用外键中name字段
    list_filter = ['course__name', 'name', 'download', 'add_time']


# 将管理器与model进行注册关联
xadmin.site.register(Course, CourseAdmin)
xadmin.site.register(Lesson, LessonAdmin)
xadmin.site.register(Video, VideoAdmin)
xadmin.site.register(CourseResource, CourseResourceAdmin) 
```

在course/models.py中

```python
def __str__(self):
    return '《{0}》课程的章节 >> {1}'.format(self.course,self.name)
```

1. <!--该重载不起作用？-->
2. <!--list_filter 拉开没有东西-->
3. <!--图片发现不到-->

### organization APP的model注册

```python
# -*- coding: utf-8 -*-
__author__ = 'amy'
__date__ = '2019/3/6 16:45'
import xadmin

from .models import CityDict, CourseOrg, Teacher


# 机构所属城市名后台管理器
class CityDictAdmin(object):
    list_display = ['name', 'desc', 'add_time']
    search_fields = ['name', 'desc']
    list_filter = ['name', 'desc', 'add_time']


# 机构课程信息管理器
class CourseOrgAdmin(object):
    list_display = ['name', 'desc', 'click_nums', 'fav_nums','add_time' ]
    search_fields = ['name', 'desc', 'click_nums', 'fav_nums']
    list_filter = ['name', 'desc', 'click_nums', 'fav_nums','city__name','address','add_time']


class TeacherAdmin(object):
    list_display = [ 'name', 'org', 'work_years', 'work_company','add_time']
    search_fields = ['org', 'name', 'work_years', 'work_company']
    list_filter = ['org__name', 'name', 'work_years', 'work_company', 'click_nums', 'fav_nums', 'add_time']


xadmin.site.register(CityDict, CityDictAdmin)
xadmin.site.register(CourseOrg, CourseOrgAdmin)
xadmin.site.register(Teacher, TeacherAdmin)
```

### operation APP的model注册

```python
# -*- coding: utf-8 -*-
__author__ = 'amy'
__date__ = '2019/3/6 17:01'
import xadmin

from .models import UserAsk, UserCourse, UserMessage, CourseComments, UserFavorite


# 用户表单我要学习后台管理器
class UserAskAdmin(object):
    list_display = ['name', 'mobile', 'course_name', 'add_time']
    search_fields = ['name', 'mobile', 'course_name']
    list_filter = ['name', 'mobile', 'course_name', 'add_time']


# 用户课程学习后台管理器
class UserCourseAdmin(object):
    list_display = ['user', 'course', 'add_time']
    search_fields = ['user', 'course']
    list_filter = ['user', 'course', 'add_time']


# 用户消息后台管理器
class UserMessageAdmin(object):
    list_display = ['user', 'message', 'has_read', 'add_time']
    search_fields = ['user', 'message', 'has_read']
    list_filter = ['user', 'message', 'has_read', 'add_time']


# 用户评论后台管理器
class CourseCommentsAdmin(object):
    list_display = ['user', 'course', 'comments', 'add_time']
    search_fields = ['user', 'course', 'comments']
    list_filter = ['user', 'course', 'comments', 'add_time']


# 用户收藏后台管理器
class UserFavoriteAdmin(object):
    list_display = ['user', 'fav_id', 'fav_type', 'add_time']
    search_fields = ['user', 'fav_id', 'fav_type']
    list_filter = ['user', 'fav_id', 'fav_type', 'add_time']


# 将后台管理器与models进行关联注册。
xadmin.site.register(UserAsk, UserAskAdmin)
xadmin.site.register(UserCourse, UserCourseAdmin)
xadmin.site.register(UserMessage, UserMessageAdmin)
xadmin.site.register(CourseComments, CourseCommentsAdmin)
xadmin.site.register(UserFavorite, UserFavoriteAdmin)
```



**注意：**关于重载的问题

## 5.5  Xadmin的全局配置

将全局配置修改:

- 如左上角：django Xadmin。下面的我的公司
- 主题修改，app名称汉化，菜单收叠。

### Xadmin的主题功能

```python
from xadmin import views
# 创建X admin的全局管理器并与view绑定。
class BaseSetting(object):
    # 开启主题功能
    enable_themes = True
    use_bootswatch = True

# 将全局配置管理与view绑定注册
xadmin.site.register(views.BaseAdminView, BaseSetting)

```



### 全局配置

修改django admin 和下面的我的公司收起菜单

```python
# 修改django admin 和下面的我的公司收起菜单
class GlobalSettings(object):
    # 修改title
    site_title = '小云云的后台管理'
    # 修改footer
    site_footer = 'Amy的公司'
    # 收起菜单
    menu_style = 'accordion'


# 将title和footer信息进行注册
xadmin.site.register(views.CommAdminView,GlobalSettings)

```



### 修改APP的名字

**以users/apps.py为例，其它三个同样操作**

默认apps.py里面的代码

```python
from django.apps import AppConfig


class UsersConfig(AppConfig):
    name = 'users'
```

修改后：

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```python
from django.apps import AppConfig


class UsersConfig(AppConfig):
    name = 'users'
    verbose_name = '用户'
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

还要在users/__init__.py中引用apps.py的配置

添加代码如下：

```python
# users/__init__.py

default_app_config = 'users.apps.UsersConfig'
```

# 六、完成登录 注册 找回密码 激活 验证码集成

> 我想只要是个系统，就少不了登录注册。这是我们需要首先处理的主要矛盾。
>
> - 完成登录 注册 找回密码 激活 验证码集成

* 首页的实现

## 6.1 完成登录功能

### 首页和登录页面实现

1. 把html文件中的index.html拷贝到templates文件夹内

   这是个前端文件。需要URLs里配置文件，其中TemplateView.as_view会将template转换为view。在此之前需要配置一些静态文件，在HTML中配置这些静态文件。

2. 新建static目录用来存放静态文件

   ```python
   # 说明静态文件放在哪个目录
   
   STATIC_URL = '/static/'
   
   STATICFILES_DIRS = (
       os.path.join(BASE_DIR, "static")
   )
   ```

3. 引用静态文件

   使用ctrl+f查找出所有`../`,全部替换为`/static/`

   使用ctrl+f查找出所有`../images`,全部替换为`/static/`images

   使用ctrl+f查找出所有`../js`,全部替换为`/static/`js

4. 配置静态文件的url

   ```python
   from django.views.generic import TemplateView
   
   urlpatterns = [
       url(r'^xadmin/', xadmin.site.urls),
       url('^$', TemplateView.as_view(template_name="index.html"), name="index")
   
   ]
   ```

   上面是首页页面

5. 登录页面

   * login.HTML文件拷贝到templates，
   * 引用静态文件
   * 配置login的urls
   * 更改index.html里面跳转到登录界面的url

   （这部分需要将个人中心注释掉）

   然后这是登录页面。

   在首页和登录页面完成之后就是我们的登录逻辑的设计。主要在user/view中编辑我的后台逻辑。

### 用户登录

编写登录的后台逻辑，前面因为使用了TemplateView.as_view，其自动会将template转换为view。现在我们要自己再user的view中编写自己的后台逻辑。所以首先在user的view里编写我的登录后台逻辑

Django的view实际就是一个函数，接收request请求对象，处理后返回response对象。

1. 在users/view.py中编写我的后台逻辑

   ```python
   # encoding: utf-8
   # 当我们配置url被这个view处理时，自动传入request对象.
   def login(request):
       # 前端向后端发送的请求方式: get 或post
   
       # 登录提交表单为post
       if request.method == "POST":
           pass
       # 获取登录页面为get
       elif request.method == "GET":
           # render就是渲染html返回用户
           # render三变量: request 模板名称 一个字典写明传给前端的值
           return render(request, "login.html", {})
   ```

2. 修改login路由（URL跟路由有关）

   ```python
   # 通过URL来访问login函数，view里就是处理着后台逻辑的
   from users.views import login
       # 登录页面跳转url login不要直接调用。而只是指向这个函数对象。
       url('^login/$',login, name="login")
   ```

3. 更改login.html、

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180320223335228-699696671.png)

   * action表示表单要提交的地址

   * 可以看到form表单中有input。点击提交会把值提交到后台。我们需要修改action让它指向我们的后台相应地址。input中的name值会被传递到后台。回组成键值对形式。

   * 403禁止访问错误：这是Django自身的保护机制，我会随机的给前端发一串符号，你必须把这串符号带回来，我才允许你post。from表单之前写上`crsf token`就好了

     

   可以发现前端数据在request的POST中，POST中是一个QueryDict对象。我们可以把其当作字典用

   ![mark](http://myphoto.mtianyan.cn/blog/180110/F7JgIA41me.png?imageslim)

   在view中可以取前端的值来在后台进行判断。可以使用Django自带的auth方法进行验证登录。

   authenticate调用只需要传入用户名和密码。成功会返回user对象，失败返回null

   ```python
   from django.contrib.auth import authenticate, login
   
       # 登录提交表单为post
       if request.method == "POST":
           # 取不到时为空，username，password为前端页面name值
           user_name = request.POST.get("username", "")
           pass_word = request.POST.get("password", "")
   
           # 成功返回user对象,失败返回null
           user = authenticate(username= user_name, password= pass_word)
   
           # 如果不是null说明验证成功
           if user is not None:
               # login_in 两参数：request, user
               # 实际是对request写了一部分东西进去，然后在render的时候：
               # request是要render回去的。这些信息也就随着返回浏览器。完成登录
               login(request, user)
               # 跳转到首页 user request会被带回到首页
               return render(request, "index.html")
           # 没有成功说明里面的值是None，并再次跳转回主页面
           else:
               return render(request, "login.html", {})
   ```

4. 修改index.html（设置登录显示个人中心判断）

    这一部分我们应该做个验证，当用户已登录状态的时候，显示用户姓名和图像及其个人中心信息

    如果没有登录，则显示登录和注册

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180320223942989-589091497.png)

   

   * 注意事项1：因为我们处理登录的自定义函数也叫login。就直接调用了自身，而不是调用Django提供的login。**所以我们一定不要把自定义view函数命名与Django提供的冲突**。解决方案：将我们的login改为`def user_login(request):`

5. 增加邮箱登录（用户名邮箱都可以登录）

   让用户可以通过邮箱或者用户名都可以登录，用自定义authenticate方法。这里是继承ModelBackend类来做的验证。在user/view.py中继承ModelBackend来自定义authentication方法。

   ```python
   from django.contrib.auth.backends import ModelBackend
   from .models import UserProfile
   # 并集运算
   from django.db.models import Q
   
   # 实现用户名邮箱均可登录
   # 继承ModelBackend类，因为它有方法authenticate，可点进源码查看
   class CustomBackend(ModelBackend):
       def authenticate(self, username=None, password=None, **kwargs):
           try:
               # 不希望用户存在两个，get只能有一个。两个是get失败的一种原因 Q为使用并集查询
   
               user = UserProfile.objects.get(Q(username=username)|Q(email=username))
   
               # django的后台中密码加密：所以不能password==password
               # UserProfile继承的AbstractUser中有def check_password(self, raw_password):
   
               if user.check_password(password):
                   return user
           except Exception as e:
               return None
   ```

   然后在setting.by中设置邮箱和用户名均可登录

   ```python
   # 设置邮箱和用户名均可登录
   AUTHENTICATION_BACKENDS = (
       'users.views.CustomBackend',
   
   )
   ```

6. 错误信息返回给前端

   ![mark](http://myphoto.mtianyan.cn/blog/180110/l55JiK9Cmh.png?imageslim)

   Html中如何取到这个值;

   login html中找到用来提示错误的段

   ![mark](http://myphoto.mtianyan.cn/blog/180110/3i7BF5edj8.png?imageslim)

   加上<div class="error btns login-form-tips" id="jsLoginTips">{{ msg }}</div>

**总结：**

```python
# 当我们配置url被这个view处理时，自动传入request对象.
def uer_login(request):
    # 前端向后端发送的请求方式: get 或post
    # 登录提交表单为post
    if request.method == "POST":
        # 取不到时为空，username，password为前端页面name值
        user_name = request.POST.get("username", "")
        pass_word = request.POST.get("password", "")
        # 成功返回user对象,失败返回null
        user = authenticate(username=user_name, password=pass_word)
        # 如果不是null说明验证成功
        if user is not None:
            # login_in 两参数：request, user
            # 实际是对request写了一部分东西进去，然后在render的时候：
            # request是要render回去的。这些信息也就随着返回浏览器。完成登录
            login(request, user)
            # 跳转到首页 user request会被带回到首页
            return render(request, "index.html")
        # 没有成功说明里面的值是None，并再次跳转回主页面
        else:
            return render(request, "login.html", {"msg":"用户名或密码错误! "})
        pass
    # 获取登录页面为get
    elif request.method == "GET":
        # render就是渲染html返回用户
        # render三变量: request 模板名称 一个字典写明传给前端的值
        return render(request, "login.html", {})
```

### 用form实现登录

**其实view是获得前端传回来的数据，然后进行处理再reporse回去。URL就是网址的入口**

基础教程中基本上都是基于函数来做的，其实更推荐基于类来做。基于类可以带来不少好处，可以在基于类的方法

1. 把前面views中的user_login()函数改成基于类的形式

   ```python
   # 基于类实现需要继承的view
   from django.views.generic.base import View
   class LoginView(View):
       # 直接调用get方法免去判断
       def get(self, request):
           # render就是渲染html返回用户
           # render三变量: request 模板名称 一个字典写明传给前端的值
           return render(request, "login.html", {})
       def post(self, request):
           # 取不到时为空，username，password为前端页面name值
           user_name = request.POST.get("username", "")
           pass_word = request.POST.get("password", "")
   
           # 成功返回user对象,失败返回null
           user = authenticate(username=user_name, password=pass_word)
   
           # 如果不是null说明验证成功
           if user is not None:
               # login_in 两参数：request, user
               # 实际是对request写了一部分东西进去，然后在render的时候：
               # request是要render回去的。这些信息也就随着返回浏览器。完成登录
               login(request, user)
               # 跳转到首页 user request会被带回到首页
               return render(request, "index.html")
           # 没有成功说明里面的值是None，并再次跳转回主页面
           else:
               return render(request, "login.html", {"msg": "用户名或密码错误! "})
   ```

   URLs中也要改变：

   ```python
   # 换用类实现
   from users.views import LoginView
       # 基于类方法实现登录,这里是调用它的方法
       url('^login/$', LoginView.as_view(), name="login")
   ```

2. users下新建form.py文件（Django的form进行表单验证）

   为了使POST的代码不用那么麻烦，使用form文件可以完成一些判读，比如验证最大长度，是否为空等一系列。

   ```Python
   # encoding: utf-8
   __author__ = 'mtianyan'
   __date__ = '2018/1/10 0010 04:44'
   # 引入Django表单
   from  django import forms
   
   # 登录表单验证
   class LoginForm(forms.Form):
       # 用户名密码不能为空
       username = forms.CharField(required=True)
       password = forms.CharField(required=True, min_length=5)
   ```

3. 定义好forms后利用它来做验证，并完善错误提示信息

   定义好之后把它放到我们的POST中进行验证

   ```python
   def post(self, request):
           # 类实例化需要一个字典参数dict:request.POST就是一个QueryDict所以直接传入
           # POST中的usernamepassword，会对应到form中
           login_form = LoginForm(request.POST)
   		#is_valid判断我们字段是否有错执行我们原有逻辑，验证失败跳回login页面
           if login_form.is_valid():
               # 取不到时为空，username，password为前端页面name值
               user_name = request.POST.get("username", "")
               pass_word = request.POST.get("password", "")
   
               # 成功返回user对象,失败返回null
               user = authenticate(username=user_name, password=pass_word)
   
               # 如果不是null说明验证成功
               if user is not None:
                   # login_in 两参数：request, user
                   # 实际是对request写了一部分东西进去，然后在render的时候：
                   # request是要render回去的。这些信息也就随着返回浏览器。完成登录
                   login(request, user)
                   # 跳转到首页 user request会被带回到首页
                   return render(request, "index.html")
           # 验证不成功跳回登录页面
           # 没有成功说明里面的值是None，并再次跳转回主页面
           else:
               return render(request, "login.html", {"msg": "用户名或密码错误! "})
   ```

4. 完善login.html的错误提示信息（error信息传到前端）

   ```python
   def post(self, request):
       # 类实例化需要一个字典参数dict:request.POST就是一个QueryDict所以直接传入
       # POST中的usernamepassword，会对应到form中
       login_form = LoginForm(request.POST) # 在此打上断点试试
       # is_valid判断我们字段是否有错执行我们原有逻辑，验证失败跳回login页面
       if login_form.is_valid():
           # 取不到时为空，username，password为前端页面name值
           user_name = request.POST.get("username", "")
           pass_word = request.POST.get("password", "")
   
           # 成功返回user对象,失败返回null
           user = authenticate(username=user_name, password=pass_word)
   
           # 如果不是null说明验证成功
           if user is not None:
               # login_in 两参数：request, user
               # 实际是对request写了一部分东西进去，然后在render的时候：
               # request是要render回去的。这些信息也就随着返回浏览器。完成登录
               login(request, user)
               # 跳转到首页 user request会被带回到首页
               return render(request, "index.html")
               # 仅当用户真的密码出错时
           else:
               return render(request, "login.html", {"msg": "用户名或密码错误!"})
       # 验证不成功跳回登录页面
       # 没有成功说明里面的值是None，并再次跳转回主页面
       else:
           return render(request, 'login.html', {'login_form': login_form})
   ```

   实例化`LoginView`时已经对于我们的字段进行了验证。

   ![mark](http://myphoto.mtianyan.cn/blog/180110/EefmGb08e9.png?imageslim)

   

   可以看出错误信息是传到了login_form(form的一个实例化)的errors中了。其为一个字典

   **先是要将错误从前端取出来**

   ```python
    return render(request, 'login.html', {'login_form': login_form})
   ```

   **要在login.html的前端文件中让错误显现出来。**![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180321002120135-778108352.png)

   

### session和cookie自动登录机制

介绍一种自动登录机制

## 6.2 完成注册

- 用户输入邮箱、密码和验证码，点注册按钮
- 如果输入的不正确，提示错误信息
- 如果正确，发送激活邮件，用户通过邮件激活后才能登陆
- 即使注册成功，没有激活的用户也不能登陆

### 放入register.html，修改模板

1. 修改index.html

   ```html
   <a style="color:white" class="fr registerbtn" href="/register/">注册</a>
   
    <a style="color:white" class="fr loginbtn" href="/login/">登录</a>
   ```

   使其在首页可以跳转到注册（登录页面也有这个）

2. 修改register.html中的静态文件地址

   跟前面首页和登录页面的html一样修改静态文件，这里我们采用另外一种方式来进行更改

   ```html
   {% load staticfiles %}
   
   <link rel="stylesheet" type="text/css" href="{% static 'css/reset.css' %}">
       <link rel="stylesheet" type="text/css" href="{% static 'css/login.css' %}">
   
   .
   .
   .
   
   <script src="{% static 'js/jquery.min.js' %}" type="text/javascript"></script>
   <script src="{% static 'js/unslider.js' %}" type="text/javascript"></script>
   <script src="{% static 'js/validateDialog.js' %}"  type="text/javascript"></script>
   <script src="{% static 'js/login.js' %}"  type="text/javascript"></script>
   ```

   [![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

然后修改路径为一个相对于static的相对路径

[![mark](http://myphoto.mtianyan.cn/blog/180110/ackf2LaLJ8.png?imageslim)](http://myphoto.mtianyan.cn/blog/180110/ackf2LaLJ8.png?imageslim)

> 他会自动根据setting中配置，为我们加上前缀

[![mark](http://myphoto.mtianyan.cn/blog/180110/3FDhdl2aiH.png?imageslim)](http://myphoto.mtianyan.cn/blog/180110/3FDhdl2aiH.png?imageslim)

> 如果我们把目录在setting中改到mystatic。url中会自动添加指定的前缀

（上面是实现一个页面的基本流程）

### 初步视图函数

同前面一样，首先将我们的注册页面放到我们的templates中，然后在user/view.py中编写我们的初步后台逻辑

前面应该要个form.py的。这个在验证码结束后一起加

```python
class RegisterView(View):
    # get方法直接返回页面
    def get(self, request):
        return render(request, "register.html", {})
```

下面进行url的设计

### 路由设计

```python
`from users.views import RegisterView    # 注册url   
url("^register/", RegisterView.as_view(), name="register"),`
```

到这里为止可以看到完整的注册页面，但是缺少了验证码部分，接下来设置验证码

### 验证码

**1.安装配置**

   * 安装

```
workon mxonline3
pip install  django-simple-captcha
workon mxonline2
pip install  django-simple-captcha==0.4.6
```

- Add `captcha` to the `INSTALLED_APPS` in your `settings.py`

  ```
  INSTALLED_APPS = [
      'captcha',
  ]
  ```

  

- Add an entry to your `urls.py`:

```python
from django.conf.urls import url, include
urlpatterns += [
    url(r'^captcha/', include('captcha.urls')),
]
```

* 生成到数据库

  ```
  `makemigrationsmigrate`
  ```

* 通过前端获取验证值

  <div class="form-group marb8 captcha1 ">
       <label>验&nbsp;证&nbsp;码</label>
       {{ register_form.captcha }}
  </div>

  **下面这块是验证的原理**

  Forms中的field会生成不同的框。(这是在网页的源码中观察)

  [![mark](http://myphoto.mtianyan.cn/blog/180110/ChgL88Ed76.png?imageslim)](http://myphoto.mtianyan.cn/blog/180110/ChgL88Ed76.png?imageslim)

  我们只有label但是前端可以查看到input框等，也就是**Registerform会为我们生成输入框+验证码。**

  > 隐藏的字符串的框会被带到后台，由Django为我们进行验证。验证该验证码是否保存过。

  [![mark](http://myphoto.mtianyan.cn/blog/180110/547cdji7Gl.png?imageslim)](http://myphoto.mtianyan.cn/blog/180110/547cdji7Gl.png?imageslim)

  > 可以看得我们数据库中将这个hashkey进行了保存。这个key与验证码内容对应。

  后台会把验证码值 和 hashkey进行联合查询。

### 编写form.py

* 在users/forms.py中定义我们的register form

  ```python
  # 引入验证码field
  from captcha.fields import CaptchaField
  
  # 验证码form & 注册表单form
  class RegisterForm(forms.Form):
      # 此处email与前端name需保持一致。
      email = forms.EmailField(required=True)
      # 密码不能小于5位
      password = forms.CharField(required=True, min_length=5)
      # 应用验证码，字段里可以自定义错误
      captcha = CaptchaField( )
  ```

* 在我们的registerform中实例化并传送到前端:

  ```python
  # form表单验证 & 验证码
  from .forms import LoginForm, RegisterForm
  
  # 注册功能的view
  class RegisterView(View):
      # get方法直接返回页面
      def get(self, request):
          # 添加验证码
          register_form = RegisterForm()
          return render(request, "register.html", {'register_form':register_form})
  ```


这里完成了部分框架，下面完善整个后台逻辑，集中在POST中

### 完善注册的后台逻辑

上面已经完成了验证码的设计，要想完成注册要先完善我们的后台逻辑（也就是整体的后台逻辑）

1. 为registerview添加POST方法

   ```python
   def post(self, request):
       # 实例化form
       register_form = RegisterForm(request.POST)
       if register_form.is_valid():
           pass
   ```

2. 修改register.HTML

   * 修改前端名字与后台逻辑的form中一样
   * 修改register中的form表单（这里form的值就是将前端的值传到后台，所以要修改），它的提交方式为POST。以及这个要提交到哪个

   ![mark](http://myphoto.mtianyan.cn/blog/180110/bA0KEd7bhc.png?imageslim)

3. 前端的form提交加上对应的crsf token（防止403）

   ![mark](http://myphoto.mtianyan.cn/blog/180110/ahg7KfbCAi.png?imageslim)

   在进行下一步之前在`register_form = RegisterForm(request.POST)`打个断点看看，故意打错验证看返回的是什么

4. 获取前端页面值并封装成一个user_profile对象，保存到数据库

```python
from django.contrib.auth.hashers import make_password

        if register_form.is_valid():
            user_name = request.POST.get("email", "")
            pass_word = request.POST.get("password", "")

            # 实例化一个user_profile对象，将前台值存入
            user_profile = UserProfile()
            user_profile.username = user_name
            user_profile.email = user_name

            # 加密password进行保存
            user_profile.password = make_password(pass_word)
            user_profile.save()
            pass
```

**总结注册后台逻辑：**（包括了发送邮件）

- 如果是get请求，直接返回注册页面给用户

- 如果是post请求，先生成一个表单实例，并获取用户提交的所有信息（request.POST）

- is_valid()方法，验证用户的提交信息是不是合法

- 如果合法，获取用户提交的email和password

- 实例化一个user_profile对象，把用户添加到数据库

- 默认添加的用户是激活状态（is_active=1表示True），在这里我们修改默认的状态（改为is_active = False），只有用户去邮箱激活之后才改为True

- 对密码加密，然后保存，发送邮箱，username是用户注册的邮箱，‘register’表明是注册

- 注册成功跳转到登录界面

  ```python
  # 发送邮件
  from utils.email_send import send_register_eamil
  
  class RegisterView(View):
      '''用户注册'''
      def get(self,request):
          register_form = RegisterForm()
          return render(request,'register.html',{'register_form':register_form})
  
      def post(self,request):
          register_form = RegisterForm(request.POST)
          if register_form.is_valid():
              user_name = request.POST.get('email', None)
              # 如果用户已存在，则提示错误信息
              if UserProfile.objects.filter(email = user_name):
                  return render(request, 'register.html',{'register_form':register_form,'msg': '用户已存在'})
  
              pass_word = request.POST.get('password', None)
              # 实例化一个user_profile对象
              user_profile = UserProfile()
              user_profile.username = user_name # 用户注册
              user_profile.email = user_name  #邮箱注册
              user_profile.is_active = False
              # 对保存到数据库的密码加密
              user_profile.password = make_password(pass_word)
              user_profile.save()
              #发送激活邮件
              send_register_eamil(user_name,'register')
              # 注册成功，就跳到登录页面
              return render(request,'login.html')
          else:
              #如果是不可行的就停留在注册页面
              return render(request,'register.html',{'register_form':register_form})
  ```

这是整体的逻辑，缺少这个`send_register_eamil(user_name,'register')`

里面的后面开始编写发送激活邮件以及激活用户等逻辑

### 发送激活邮件激活用户（如何发送邮件到邮箱）

邮箱注册包括激活，我们要设计邮箱激活的后台逻辑

在Python中已经内置了一个smtp邮件发送模块，Django在此基础上进行了简单地封装，让我们在Django环境中可以更方便更灵活的发送邮件。所有的功能都在django.core.mail中。

* setting中设置邮箱信息，有个内置的smpt邮件
* apps：utils/email_send.py`:在这里编写发送链接
* 为页面设计激活用户的逻辑

1. setting中设计

   ```python
   # 发送邮件的setting设置
   
   EMAIL_HOST = "smtp.qq.com"
   EMAIL_PORT = 25
   EMAIL_HOST_USER = "mxonline.mtianyan.cn"
   EMAIL_HOST_PASSWORD = " "
   EMAIL_USE_TLS= True
   EMAIL_FROM = "mxonline.mtianyan.cn"
   ```

   **说明：**

   　　 EMAIL_HOST = "smtp.qq.com"

   　　 EMAIL_HOST_PASSWORD = "dwjybikexxxxxxxx"

   要想用qq邮箱作为服务器发送邮件，必须先开启SMTP，方法如下：（开启smtp）

   1）登录邮箱，找到“设置”-->>“用户”

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180322174645418-649399734.png)

   2）往下拉找到SMTP服务，点开启，然后点“生成授权码”

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180322174821572-882851617.png)

   3）可以看到授权码，**“EMAIL_HOST_PASSWORD”里面填写的就是下面生成的授权码**，而不是你的邮箱密码

    ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180322174959071-2118415634.png)

   cjqubtnqjpluhedj

2. 在apps中新建package后新建文件。`apps：utils/email_send.py`:

   这里主要是为生成发送链接并发送到邮箱，所有之前对你的接收邮箱进行设置

   ```python
   # encoding: utf-8
   from random import Random
   
   #导入之前的邮箱验证码，用于注册前保存到数据库，用于查询链接是否存在
   from  users.models import EmailVerifyRecord
   # 导入Django自带的邮件模块
   from django.core.mail import send_mail
   # 导入setting中发送邮件的配置
   from Mxonline.settings import EMAIL_FROM
   
   # 生成随机字符串
   def random_str(random_length=8):
       str = ''
       # 生成字符串的可选字符串
       chars = 'AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz0123456789'
       length = len(chars) - 1
       random = Random()
       for i in range(random_length):
           str += chars[random.randint(0, length)]
       return str
   
   # 发送注册邮件
   def send_register_eamil(email, send_type="register"):
       # 发送之前先保存到数据库，到时候查询链接是否存在
   
       # 实例化一个EmailVerifyRecord对象
       email_record = EmailVerifyRecord()
       # 生成随机的code放入链接
       code = random_str(16)
       email_record.code = code
       email_record.email = email
       email_record.send_type = send_type
   
       email_record.save()
   
       # 定义邮件内容:
       email_title = ""
       email_body = ""
       # 如果是注册则生成激活链接
       if send_type == "register":
           email_title = "mtianyan慕课小站 注册激活链接"
           email_body = "请点击下面的链接激活你的账号: http://127.0.0.1:8000/active/{0}".format(code)
   
           # 使用Django内置函数完成邮件发送。四个参数：主题，邮件内容，从哪里发，接受者list
           send_status = send_mail(email_title, email_body, EMAIL_FROM, [email])
           # 如果发送成功
           if send_status:
               pass
   ```

3.激活用户

这个在前面完善注册的后台逻辑中完成了，具体逻辑看前面

```python
# 发送邮件
from utils.email_send import send_register_eamil
            # 发送注册激活邮件
            send_register_eamil(user_name, "register")
```

根据邮箱找到对应的用户，然后设置is_active = True来实现。在view中添加激活的后台逻辑

```python
# 激活用户
class ActiveUserView(View):
    def get(self, request, active_code):
        # 查询邮箱验证记录是否存在
        all_record = EmailVerifyRecord.objects.filter(code = active_code)

        if all_record:
            for record in all_record:
                # 获取到对应的邮箱
                email = record.email
                # 查找到邮箱对应的user
                user = UserProfile.objects.get(email=email)
                user.is_active = True
                user.save()
         # 验证码不对的时候跳转到激活失败页面
        else:
            return render(request,'active_fail.html')
        # 激活成功跳转到登录页面
        return render(request, "login.html", )
```

既然在view中添加这样的一个逻辑，在URL中也要进行配置

```python
# 激活用户url
url(r'^active/(?P<active_code>.*)/$',ActiveUserView.as_view(), name= "user_active")
```

 在templates目录下创建 active_fail.html,代码如下：

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
<p>链接失效</p>
</body>
</html>

添加一个判断，用户注册的后，等激活才能登陆

![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180322022251945-568203948.png)



到这里整个注册后台逻辑就设计好了。并且包括了注册里的发送邮件激活逻辑.

下面需要对register.html进行一些必要的修改

### 修改一下register.HTML完善错误提示和完善用户回填逻辑

主要对form进行一些修改。对应这POST的

```html
复制代码
 <form id="email_register_form" method="post" action="{% url 'register' %}" autocomplete="off">
                        <input type='hidden' name='csrfmiddlewaretoken' value='gTZljXgnpvxn0fKZ1XkWrM1PrCGSjiCZ' />
                        <div class="form-group marb20 {% if login_form.errors.email %}errorput{% endif %}">
                            <label>邮&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;箱</label>
                            <input  type="text" id="id_email" name="email" value="{{ register_form.email.value }}" placeholder="请输入您的邮箱地址" />
                        </div>
                        <div class="form-group marb8 {% if login_form.errors.password %}errorput{% endif %}">
                            <label>密&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;码</label>
                            <input type="password" id="id_password" name="password"  value="{{ register_form.password.value }}" placeholder="请输入6-20位非中文字符密码" />
                        </div>
                        <div class="form-group marb8 captcha1 {% if login_form.errors.captchal %}errorput{% endif %}">
                            <label>验&nbsp;证&nbsp;码</label>
                            {{ register_form.captcha }}
                        </div>
                        <div class="error btns" id="jsEmailTips">
                            {% for key,error in register_form.errors.items %}
                                {{ error }}
                            {% endfor %}
                            {{ msg }}
                        </div>
                        <div class="auto-box marb8">
                        </div>
                        <input class="btn btn-green" id="jsEmailRegBtn" type="submit" value="注册并登录" />
                    <input type='hidden' name='csrfmiddlewaretoken' value='5I2SlleZJOMUX9QbwYLUIAOshdrdpRcy' />
                    {% csrf_token %}
                    </form>

```

修改的地方说明：

* 修改method对应POST处理，修改action，提交的位置为拿
* 修改前端名字与后台逻辑的form中一样
* 前端的form提交加上对应的crsf token（防止403）

- value="{{ register_form.email.value }}     
- value="{{ register_form.password.value }}    注册的用户不用再手动输入一遍邮箱和密码了 
- {% if login_form.errors.email %}errorput{% endif %}
- {% if login_form.errors.password %}errorput{% endif %}
- {% if login_form.errors.captchal %}errorput{% endif %}    提示错误信息并显示红框框
- {{ register_form.captcha }}    显示验证码

## 6.3 找回密码

主要需要实现的功能：

- 用户点“忘记密码”，跳到找回密码页面
- 在forgetpwd页面，输入邮箱和验证码成功后，发送邮件提醒
- 通过点击邮件链接，可以重置密码
- 两次密码输的正确无误后，密码更新成功，跳到登录界面

### 初步视图函数

首先拷入forgetpassword页面

然后也需要个表单

forms.py

```
class ForgetPwdForm(forms.Form):
    '''忘记密码'''
    email = forms.EmailField(required=True)
    captcha = CaptchaField(error_messages={'invalid': '验证码错误'})
```

别忘了在html中添加验证码错误

users/views.py

```python
# 用户忘记密码的处理view
class ForgetPwdView(View):
    # get方法直接返回页面
    def get(self, request):
        forget_from = ForgetPwdForm()
        return render(request, "forgetpwd.html", {"forget_from":forget_from })

```

### 路由设计

```python
# 忘记密码
url('forget/$', ForgetPwdView.as_view(), name="forget_pwd"),
```

### 模板修改

修改login.html中的url

```html
<a class="fr" href="{% url 'forget_pwd' %}">忘记密码？</a>
```

使其在登录的时候可以跳转到忘记密码页面

然后配置忘记密码的静态文件还有其验证码

```
load static
修改static的目录
修改其中的url
```

### 添加发送找回密码邮件

在apps：utils/email_send.py`:加上

```Python
    elif send_type == "forget":
        email_title = "慕学在线网注册密码重置链接"
        email_body = "请点击下面的链接重置密码: http://127.0.0.1:8000/reset/{0}".format(code)

        send_status = send_mail(email_title, email_body, EMAIL_FROM, [email])
        if send_status:
            pass
```

### 完善找回密码的views

添加POST的方法：

```Python
    def post(self, request):
        forget_form = ForgetPwdForm(request.POST)
        if forget_form.is_valid():
            email = request.POST.get("email", "")
            send_register_eamil(email, "forget")
            return render(request, "send_success.html")
        else:
            return render(request, "forgetpwd.html", {"forget_form":forget_form})
```

**新建templates/send_success.html**

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <p>邮件已发送，请注意查收</p>
</body>
</html>
```

[![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 修改html中的前端文件

1. 修改form，使其在后端传数据。这个数据大都在request中

   ![img](http://myphoto.mtianyan.cn/blog/180111/4e60888a7F.png?imageslim)

   

2. 添加错误信息（返回在forget_form）

   ![1552275522168](C:\Users\xiaoyunyun\AppData\Roaming\Typora\typora-user-images\1552275522168.png)

测试一下，输入邮箱和验证码看能不能收到邮件

![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180323221101651-13334727.png)

### 重置密码

1. 书写重置密码view

   

2. 配置重置密码的url

3. 拷贝进来password reset页面

4. 创建改变密码的forms:

5. 书写改变密码的view；

6. 配置modify的url。

# 七、授课机构功能

## 7.1 模板继承

模板的继承：定义一个父模板，共用的部分不用动，不一样的地方用{% block title%}  {%end block%}括起来，子类可以替换其中这部分block，子类只需要重写自己需要改变的block

1. 创建父模板

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180324104830436-955539027.png)

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180324104902493-1443458262.png)

   

   * 静态引用
   * title应该是可以被子页面替换的所以要包起来。
   * css有共用的部分，也有可以被子页面替换的部分。
   * js同理
   * 面包屑也有{%block custom_bread%} {%endblock %}
   * 正文部分{%block content%} {%endblock %}

   2. 编写子类org—list

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180324114025675-378785309.png)

   - 继承base.html
   - 覆盖父类的title
   -  改写面包屑
     *  可以在base中把<li>课程机构</li>这句删掉
     * 在org_list.html中重写面包屑
     * 把base中的content拿过来，是重写的
     * base父类中的content删掉，因为每个分类的content是不一样的 

  注意：子模板中也要{% load staticfiles %}

**总结：**

页面的继承关系使得变量也可以直接用

> 比如user中的form数据传递到register文件当中.如果register继承的是base页面。
> base页面当中也是可以用这些数据的。`参数的向上传递`

每个request对象都会传递到html中来，如果继承了base，request也会向上传递到base。
base中就可以加入我们的逻辑: 用户是否登录等。

## 7.2 课程机构数据展示

![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180324121132623-598326237.png)

这里机构是静态固定不变的，**所在地区是动态的，从后台数据库中获取得到的**。授课机构的排名也是从后台取出来的

### 添加城市和课程机构

1. **在xadmin中后台添加城市**

   在xadmin的课程机构中添加城市

   ![mark](http://myphoto.mtianyan.cn/blog/180111/CJG79b8JJ5.png?imageslim)

   之后再model中重载我们Unicode的name

   ```python
   def __unicode__(self):
       return self.name
   ```

2. **添加课程机构**

​        添加机构时，有个上传图片。这里就对上传图片做些操作：

​        1） 首先看到model里:

```python
class CourseOrg(models.Model):
    .......
    image = models.ImageField(
        upload_to="org/%Y/%m",
        verbose_name=u"封面图",
        max_length=100)
   ..........
    class Meta:
        verbose_name = u"课程机构"
        verbose_name_plural = verbose_name
```

这里指定的是一个相对路径。

​     2）所以在setting中要配置我们把文件存放在哪个根目录之下：

```python
# 设置我们上传文件的路径

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

想当于STATIC_URL = '/static/'。利用media_url时会自动加上。

​    3）新建一个文件夹media

​     该文件用来存储上传文件

​     4）在xadmin中的课程机构中添加机构后，发现在我们的media中可以呈现出来

   ![img](file:///C:\Users\xiaoyunyun\AppData\Roaming\Tencent\Users\1563115157\QQ\WinTemp\RichOle\$B@BNK%LA56NZRZLW]I`$T6.png)



添加十个机构

3. model中添加机类别

   ```python
     class CourseOrg(models.Model):
       ORG_CHOICES =(
           ("pxjg", u"培训机构"),
           ("gx", u"高校"),
           ("gr", u"个人"),
       )
   
   ​    name = models.CharField(max_length=50, verbose_name=u"机构名称")
   ​    \# 机构描述，后面会替换为富文本展示
   ​    desc = models.TextField(verbose_name=u"机构描述")
   ​    \# 机构类别:
   ​    category = models.CharField(max_length=20, choices=ORG_CHOICES, verbose_name=u"机构类别", default="pxjg")
   ```

     


model改变后，数据库也要进行同步更新

**数据表变动完成，至此数据添加完成，接下来开始我们的view**

### 显示课程机构和城市

将列表里的静态数据变成后台获取的动态数据

1. **在orgnization中写view逻辑**

   ```python
   from .models import CourseOrg, CityDict
   
   
   class OrgView(View):
       '''课程机构'''
       def get(self,request):
           # 取出所有课程机构
           all_orgs = CourseOrg.objects.all()
           org_onums = all_orgs.count()
           # 取出所有城市
           all_citys = CityDict.objects.all()
           return render(request, "org-list.html", {
               "all_orgs": all_orgs,
               "all_citys": all_citys,
               'org_onums':org_onums,
           })
   
   ```

注：get是获取服务器页面。POST传递数据到服务器。

POST处理完会传递到request中去，每个request对象都会传递到html中去

将all_orgs和all_citys传递到request（模板）中去，之后将数据展示到template中去。

2. **修改org_list.htm**l

   1)观察HTMl的content部分

   **显示我们数据库添加的城市**

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180324134853576-773132439.png)

   使用for循环将我们的城市加入进来

   **显示我们数据库添加的机构**

   显示课程机构的数目

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180324140233528-267342101.png)

      可以看到每个课程机构都是一个dl，留一个使用for循环遍历我们数据库添加的课程机构

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180324135021295-242157208.png)

   **完善课程机构的dl中的数据**

   ![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180324135110032-1070169703.png)

   **设计logo部分**

   * **如何将image Field转换成图片地址：**

   数据库中image以字符串格式保存的，是相对路径，直接取是取不出来的，必须补全路径

   ![img](file:///C:\Users\xiaoyunyun\AppData\Roaming\Tencent\Users\1563115157\QQ\WinTemp\RichOle\W}`}217OEHDI}O2IPB]U~0D.png)

   ```python
   data-url="{{ MEDIA_URL }}{{ course_org.image }}"
   MEDIA_URL = '/media/'，这个是之前settings中设置好了
   ```

   * 要向使用{{ MEDIA_URL }}，要先在settings中TEMPLATES 里面添加media处理器：'django.core.context_processors.media'

   ```
   TEMPLATES = [
       {
           'BACKEND': 'django.template.backends.django.DjangoTemplates',
           'DIRS': [os.path.join(BASE_DIR, 'templates')]
           ,
           'APP_DIRS': True,
           'OPTIONS': {
               'context_processors': [
                   'django.template.context_processors.debug',
                   'django.template.context_processors.request',
                   'django.contrib.auth.context_processors.auth',
                   'django.contrib.messages.context_processors.messages',
                   #添加图片处理器，为了在课程列表中前面加上MEDIA_URL
                   'django.template.context_processors.media',
               ],
           },
       },
   ]
   ```

   [![复制代码](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   *  然后也要添加处理图片相应路径的url

   ```python
   from django.views.static import serve
   from mxonline.settings import MEDIA_ROOT
   # 处理图片显示的url,使用Django自带serve,传入参数告诉它去哪个路径找，我们有配置好的路径MEDIAROOT
       url(r'^media/(?P<path>.*)$', serve, {"document_root": MEDIA_ROOT }) 
   ```

## 7.3 列表分页功能

1. 安装

   ```
   pip install django-pure-pagination
   ```

2. setting里添加

   ```
   INSTALLED_APPS = (
       ...
       'pure_pagination',
   )
   ```

3. 使用：可以看文档（建议看）

   ![img](file:///C:\Users\xiaoyunyun\AppData\Roaming\Tencent\Users\1563115157\QQ\WinTemp\RichOle\0X9BE[$6EGI[U2B[IYU$1E9.png)

   官方实例：

   ![img](file:///C:\Users\xiaoyunyun\AppData\Roaming\Tencent\Users\1563115157\QQ\WinTemp\RichOle\60`7`X0V%9RDHT~N[V90FTU.png)

   我们对照着官方实例实现：

   在我们的view（主要）

   ```python
   from pure_pagination import Paginator, EmptyPage, PageNotAnInteger
   class OrgView(View):
       def get(self,request):
           # 查找到所有的课程机构
           all_orgs = CourseOrg.objects.all()
           # 总共有多少家机构使用count进行统计
           org_nums = all_orgs.count()
           # 取出所有的城市
           all_city = CityDict.objects.all()
           # 对课程机构进行分页
           # 尝试获取前台get请求传递过来的page参数
           # 如果是不合法的配置参数默认返回第一页
           try:
               page = request.GET.get('page', 1)
           except PageNotAnInteger:
               page = 1
           # 这里指从allorg中取五个出来，每页显示5个
           p = Paginator(all_orgs, 5, request=request)
           orgs = p.page(page)
   
           return render(request, "org-list.html", {
               "all_orgs":orgs,
               "all_city": all_city,
           "org_nums": org_nums,
           })
   ```

4. 修改org_list.html.（主要）

   ```
   {{ all_orgs.render }}
   ```

![img](file:///C:\Users\xiaoyunyun\AppData\Roaming\Tencent\Users\1563115157\QQ\WinTemp\RichOle\$_64QPYCA_}4Q69ZJ2E@3]Y.png)

逻辑没弄懂

在添加了分页后城市等信息没有了

（这是个问题）



## 7.4 列表筛选功能

### 城市列表删选

![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180324151843210-1705024942.png)

- 点城市，筛选出对应的课程机构
- 默认“全部”是‘active’状态（绿色），如果点了某个城市，应该城市是‘active’状态
- 当用户点击city时，应该把city的id传到后台，然后后台在传到模板中，是的可以知道哪个城市被选中，然后加上‘’active‘

**1.view书写**

编写后台逻辑

```python
#取出筛选的城市
city_id = request.GET.get('city', '')
# 如果选择选择了某个城市，也就是前端传过来的值
if city_id:
    all_orgs = all_orgs.filter(city_id=int(city_id))
```

获取传入的参数进行进一步筛选。

```Python
return render(request, "org-list.html", {
    "all_orgs": orgs,
    "all_city": all_city,
    "org_nums": org_nums,
    "city_id": city_id
})
```

**2.前端页面相应改变**

```html
<h2>所在地区</h2>
    <div class="more">更多</div>
    <div class="cont">
        <a href="?ct="><span class="{% ifequal city_id '' %}active2{% endifequal %}">全部</span></a>
        {% for city in all_citys %}
            <a href="?city={{ city.id }}"><span class="{% ifequal city.id|stringformat:'i' city_id %}active2{% endifequal %}">{{ city.name }}</span></a>
        {% endfor %}
    </div>
```

* 当用户点击某一个city时对应加上参数city的id：href="?city={{ city.id }}"

* 将city_id传回html，使得可以知道哪个是选中的:active为激活

  ![mark](http://myphoto.mtianyan.cn/blog/180112/mmChH6l2j9.png?imageslim)

  因为city.id是后端传回来的值是一个int。所以我们要做类型转换。

* 当city_id为空的时候，显示全部。

  ![mark](http://myphoto.mtianyan.cn/blog/180112/GJa43hh95F.png?imageslim)

  

城市筛选未成功：问题出在city的值未取出来。ciy.id

### 机构列表删选

**1.后台书写**

```
if category:
    # 我们就在机构中作进一步筛选类别
    all_orgs = all_orgs.filter(category=category
```



**2.前端设置**

![mark](http://myphoto.mtianyan.cn/blog/180112/me43dl1f0g.png?imageslim)

类似上面

**3.进行城市与分类联动**

- 当选择全部类别的时候，就只通过当前城市id。
- 当选择全部城市的时候，就只通过当前目录id。
- 当两者都选的时候使用&连接。

### 课程机构排名筛选

**1.view**

```
# 热门机构,如果不加负号会是有小到大。
     hot_orgs = all_orgs.order_by("-click_nums")[:3]
```

**2.前端**

![mark](http://myphoto.mtianyan.cn/blog/180112/dbFlbHIBmb.png?imageslim)

### 课程机构排序

1.model里加东西

```
students = models.IntegerField("学习人数", default=0)
course_nums = models.IntegerField("课程数", default=0)
```

2.后台学习人数和课程筛选

if sort:
    if sort == "students":
        all_orgs = all_orgs.order_by("-students")
    elif sort == "courses":
        all_orgs = all_orgs.order_by("-course_nums")
org_nums = all_orgs.count()

3.前端处理

![mark](http://myphoto.mtianyan.cn/blog/180112/dbFlbHIBmb.png?imageslim)

## 7.5  我要 学习设置

这是对form表单的处理。在这之前介绍两个知识：直接从model转换为form

### 相关知识介绍（以及配置form）

#### modelform

对应表`userask`，`form`会对字段先做验证，然后保存到数据库中。

可以看到我们的forms和我们的model中有很多内容是一样的。我们如何让代码重复利用呢？

使用modelform解决这个问题。

```python
from operation.models import UserAsk
from 
class UserAskForm(forms.ModelForm):
    # 继承之余还可以新增字段

   
    class Meta:
        model = UserAsk  #由哪个model来的
        # 我需要验证的字段
        fields = ['name','mobile','course_name']
```

#### url路由分发

为了避免在APP的urls写url造成混乱。在每个APP新建属于自己的urls，专门存放自己的url。

涉及到include机制

```python

# encoding: utf-8
from organization.views import OrgView
from django.conf.urls import url

urlpatterns = [
    # 课程机构列表url
    url('list/$', OrgView.as_view(), name="org_list"),
```

不需要org。它会自己加上的

自己APP下的url

在总的里面做个include，使其能够跳转到APP下的url

```python
# 课程机构app的url配置
  url(r"^org/", include('organization.urls',namespace="org")),
```

使用命名空间防止重复：org（专属于organization的）：org_list

这些弄完后，别忘了去base页面中设置，使其可以跳到授课机构中来

### 配置我要学习页面url

上面form表单配置好了

一般先配置url，完成view之后不能忘了回来继续配置它

在organization/urls里面配置：

```python
url(r'^add_ask/$', AddUserAskView.as_view(), name="add_ask")
```

### 完善view后台逻辑

表单处理用POST，get是单独做请求的

比较合理的操作是异步的，不会对整个页面进行刷新。如果有错误，显示错误。一种ajax的异步操作。

因此我们此时不能直接render一个页面回来。应该是给前端返回json数据，而不是页面。

```python
from django.http import HttpResponse
# 用户添加我要学习
class AddUserAskView(View):
    # 处理表单提交当然post
    def post(self,request):
        userask_form = UserAskForm(request.POST)
        # 判断该form是否有效
        if userask_form.is_valid():
            # 这里是modelform和form的区别
            # 它有model的属性
            # 当commit为true进行真正保存
            user_ask = userask_form.save(commit=True)
            # 这样就不需要把一个一个字段取出来然后存到model的对象中之后save

            # 如果保存成功,返回json字符串,后面content type是告诉浏览器的,
            return HttpResponse("{'status': 'success'}", content_type='application/json')
        else:
            # 如果保存失败，返回json字符串,并将form的报错信息通过msg传递到前端
            return HttpResponse("{'status': 'fail', 'msg':{0}}".format(userask_form.errors),  content_type='application/json')
```

完成这个之后别忘了去url中完善其url。

### 前端配置

Ajax请求

```
<script>
    $(function(){
        $('#jsStayBtn').on('click', function(){
            $.ajax({
                cache: false,
                type: "POST",
  				url:"{% url "org:add_ask" %}",
                data:$('#jsStayForm').serialize(),
                async: true,
                success: function(data) {
                    if(data.status == 'success'){
                        $('#jsStayForm')[0].reset();
                        alert("提交成功")
                    }else if(data.status == 'fail'){
                        $('#jsCompanyTips').html(data.msg)
                    }
                },
            });
        });
    })

</script>
```

> 监听这个button，用户如果点击了button。我们来向这个url进行post请求。
> 将我们的表单进行序列化。

form表单添加crsf_token

如果后台返回的状态值为success，那么我们将弹出提交成功。
失败，就会在错误提示框

### （其实在form中）手写正则表达式

## 7.6 机构详情

机构详情页包括了：机构首页、机构课程、机构介绍、机构讲师。

![img](https://images2018.cnblogs.com/blog/1299879/201803/1299879-20180325001552260-735704145.png)





先把这个四个html放入到template中。机构详情很多是重复的，可以继承。制作一个基类

* 首先是标题，css，js。
* head是最上面部分是共用的，
* right部分需要加改变

### xadmin新增讲师和课程

1. 新增讲师

   为导师新增头像

   teacher页面出现错误：

   Django 'gbk' codec can't decode bytes in position 4-5: illegal multibyte sequence

   

2. 新增课程

   增加个外键

### 课程首页功能实现

1. 继承基类

2. 设置url

3. 编辑view

4. 设置html

   主要改变以下四个点：

   * org_lis机构列表页配置跳转到机构首页，注意课程机构的id要写进去

   * 课程机构的图片及名字，要显示到前面，这些在org_base.html中实现就好了

   * 具体到某个页面具体设置

   * 在org_base里可以实现四个部分的转换。利用current_page进行active（修改left链接）

     

下面的机构课程，机构讲师，机构介绍，跟课程首页实现功能类似。仿照一遍来就行了。



## 7.7 课程收藏功能

# 八、课程页功能的实现

上面实现了课程机构，下面这一节对应公开课的实现

## 8.1 课程列表

![1552631530451](C:\Users\xiaoyunyun\AppData\Roaming\Typora\typora-user-images\1552631530451.png)

* 课程展示
* 分页
* 排序
* 热门推荐

同一个html文件，按块操作（上下，左右）。先改变没有联系的，再将有联系的处理一下

### 准备阶段（模板拷贝配置）

拷贝 html到template：改静态文件（先不改，可能不用改那么多）

新建course/urls，配置url

编写view：先写个get让页面出来，具体逻辑再分析（这是简单分析）

前端配置：继承base.html，套用org_list.主体是content，需要重写content页面

class=wp的section放到

这一步保证页面能出来就行

### url设计

前面已经完成了

###  view设计

在这之前多加点课程

继续完善view：前面只是为了让页面出来。现在继续做让后台数据传进来进来，编写后台逻辑。

```python
class CourseListView(View):
    def get(self, request):
        all_course = Course.objects.all()
        return render(request, "course-list.html", {
            "all_course":all_course,
            
        })
```

这里all_course到我们的HTML中来了，后续需要在HTML中做遍历

### 前端设置

1. 分析页面结构

   分为左右div，左右两块。左边一个个box。留一个for



之后反复的view和html。

### 分页（逻辑没彻底弄懂）

1. view中做配置

课程过多的话，需要分页。直接可以在那个org_list中拷贝进来。

render中也要改变。

2. 前端的for循环要加上object_list

页码配置:在class=pageturn中。直接从org_list拿过来。改变变量名就好了。

### 排序

1. view中sort功能。记得要把sort传递进来



2. 前端的class=“tab_header”



注意最新的改变

### 热门机构推荐

1. view

   

2. 前端配置

   针对choice的书写情况 

## 8.2  课程详情页

详情页包括左上角的课程数据展示，右上角的课程机构，左下角的课程详情，右下角是相关课程推荐

### 1. 模板继承

继承的base页面。把course_list的拷贝进来。注意面包屑的改变。

### 2. url

在配置好url之后，下面三个步骤是根据相互之间的需求做增添，改变

### 3. view

* 页面呈现get

* 取出课程

* 增加课程点击数

* 取出tag，并做相关判断，获得相关推荐的课程

* 用户收藏（暂时不做）

  

### 4. model增加

**course/model中:**

* 一个category字段：课程类别
* 一个获取章节数的方法
* 增加个tag：课程标签

**organization/model中：**

* 一个获取这门课程的学习用户方法
* 获得教师数

model中出现gbk问题解决不了

### 5.HTML

也就是将后台数据对前端做数据填充

没有for，

* course_list列表页设置，跳转课程，注意有参数类型，要把id值传递进来
* course_detail中改逻辑：这其中包括了课程类别，获取章节数，获取门课程的学习用户方法
* 课程详情的显示：{{course.detail}}

* 授课机构的显示：
* 相关课程推荐
  * tag的技术

* 用户收藏（暂时不做）

### 6. 快速添加用户头像的方法

直接把把数据库中课程的头像，放到operation中



view，model，html要相互改变



**接下来点击开始学习会出现课程章节信息**

## 8.3 课程章节信息

**点击我要学习：**链接转换

可以看到的一个是章节信息，一个是视频信息

### 1.模板继承

course comments 和 course video放入 template

分析页面：

* title
* 面包屑
* Id=main（content）
* css （跟慕课网有关的）在header中，记得改静态文件路径

先改**course-video.html**，同样继承base.html，然后里面有属于自己的样式，也要保留。

### 2.url

### 3.view

* 参数要有课程id

* 取出课程

* 课程资源

  ```python
  all_resources = CourseResource.objects.filter(course=course)
  
            "all_resources":all_resources,
  ```

### 4. model增加

* course/model增加url
* course/model（class course下）增加def  get_course_lesson:获得课程的章节
* course/model（class lesson）增加def  get_lesson_vedio:获得章节视频
* course/model增加视频信息
* 增加外键讲师，将课程与讲师对应起来
* 课程须知，老师告诉你能学到什么

### 5. html

* course_detail改变url使得跳换
* 将后台数据传到前端（首先补充好model数据）,针对章节视频
  - 遍历章节
  - 遍历视频信息
    * model里缺少了视频时长，和访问地址
    * for这些视频加上
* 课程基本信息展示（上面）

* 完善课程资源（资料下载）
  * 先在xadmin中将信息填好（只要点保存就出现gbk问题）
  * for这些资源添加进到前端

* 讲师提示：
  * 需要在model中加个教师的外键，在xadmin中增加信息





## 8.4 课程视频信息





## 8.5 课程评论

(没完成，放着）

添加评论。通过ajax完成的



## 8.6 相关课程推荐

（放着）

## 8.7 课程播放页面

（先不做）

#

# 九、授课教师

## 9.1 讲师列表页



## 9.2 讲师详情页







# 十、个人中心

个人信息：个人信息展示，修改头像，修改密码

我的课程：

我的收藏：机构，课程，老师的收藏

我的消息：

左侧显示：



## 10.1 个人信息





### 1. 继承模板

新建 usercenter—base.html

内容方面只模板左边继承

### 2. 在users/views.py中编写个人中心的接口：

```python
 class UserInfoView(View):
2     """个人中心页面"""
3     def get(self, request):
4         return render(request, 'usercenter-info.html', {
5 
6         })
```

### 3. 完善url

mxonline/urls:     url(r'^users/', include('users.urls', namespace="users")),

users/urls：url(r'^info/$', UserinfoView.as_view(), name="user_info"),

### 4.完善个人信息接口

* 在form.py中加入个人信息的form表单验证

  ```
  class UserInfoForm(forms.ModelForm):
      class Meta:
          model = UserProfile
          fields = ['nick_name', 'gender', 'birthday', 'address', 'mobile']
  ```

* view中完善信息

  ```python
  class UserinfoView(View):
      """
      用户个人信息
      """
      def get(self, request):
          return render(request, 'usercenter-info.html', {})
  
      def post(self, request):
          # instance会自动保存
          user_info_form = UserInfoForm(request.POST, instance=request.user)
          if user_info_form.is_valid():
              user_info_form.save()
              return HttpResponse('{"status":"success"}', content_type='application/json')
          else:
              return HttpResponse(json.dumps(user_info_form.errors), content_type='application/json')
  ```

​      return httpResonse没懂

* 修改html

  form表单的修改

  ![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122092013567-1446476781.png)

  一定要加上{% csrf_token %}

### 5. 修改头像

* 在form.py文件中加入头像的ModelForm表单验证：

```Python
class UploadImageForm(forms.ModelForm):
    class Meta:
        model = UserProfile
        fields = ['image']
```

* view中完成修改用户头像的接口

  ```python
  class UploadImageView(LoginRequiredMixin, View):
      """
      用户修改头像
      """
      def post(self, request):
  #上传的图片放在了request的file中了， 这时候用户上传的文件就已经被保存到imageform了 ，为modelform添加instance值直接保存
          image_form = UploadImageForm(request.POST, request.FILES, instance=request.user)
          if image_form.is_valid():
              image_form.save()
              return HttpResponse('{"status":"success"}', content_type='application/json')
          else:
              return HttpResponse('{"status":"fail"}', content_type='application/json')
  ```

* url配置

  ```
  #用户头像上传
  url(r'^image/upload/$', UploadImageView.as_view(), name="image_upload"),
  ```

* 前端html：修改个人中心页面显示用户头像的代码

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122094213965-2042028832.png)

### 6.修改密码

* view

  ```python
   1 class UpdatePwdView(LoginRequiredMixin, View):
   2     """修改密码"""
   3     def post(self, request):
   4         modify_form = ModifyPwdForm(request.POST)
   5         if modify_form.is_valid():
   6             # 从request中获取密码
   7             pwd1 = request.POST.get('password1', '')
   8             pwd2 = request.POST.get('password2', '')
   9             if pwd1 != pwd2:
  10                 return HttpResponse('{"status": "fail", "mag": "密码不一致"}', content_type='application/json')
  11 
  12             # 保存密码
  13             user = request.user
  14             user.password = make_password(pwd2)
  15             user.save()
  16 
  17             return HttpResponse('{"status": "success"}', content_type='application/json')
  18         else:
  19             return HttpResponse(json.dumps(modify_form.errors), content_type='application/json')
  ```

* url

  ```
  1 from .views import UpdatePwdView
  2 
  3 urlpatterns = [
  4     path('update/pwd/', UpdatePwdView.as_view(), name='update_pwd'),  # 修改密码
  5 ]
  ```

* html

  修改密码的ajax代码在deco-user.js文件中，现在只需要在usercenter-base.html页面中修改密码的form表单中加上{% csrf_token %}：

  ![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122101809783-848028753.png)

  修改完成之后需要重新登录，现在修改右上角的登录状态以及显示信息，在usercenter-base.html页面中：

  ![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122103506512-382345071.png)

  同时将base.html页面和org_base.html页面也修改了

## 10.2 我的课程

### 1.继承模板

将前端页面usercenter-mycourse.html拷贝到templates下。

然后继承usercenter-base.html页面，重写需要block的地方

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122161953032-882812216.png)

### 2.view接口

```python
1 class MyCourseView(LoginRequiredMixin, View):
2     """我的课程页面"""
3     def get(self, request):
4         # 获取用户的课程
5         user_courses = UserCourse.objects.filter(user=request.user)
6 
7         return render(request, 'usercenter-mycourse.html', {
8             'user_courses': user_courses
9         })
```

### 3.url

```python
1 urlpatterns = [
2     path('mycourse/', MyCourseView.as_view(), name='mycourse'),  # 我的课程
3 ]
```

### 4. html前端改动

修改个人中心页面中跳转到我的课程页面的url：

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122162125729-627065724.png)

修改我的课程页面中显示我的课程的代码：

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122162230059-1255743606.png)

## 10.3 我的收藏

**切记检查修改各个页面之间跳转的url**

### 3.1 课程机构收藏

#### 1.模板继承

前端页面配置，先将usercenter-fav-org.html页面拷贝到templates下，继承usercenter-base.html页面，重写需要block的地方：

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122162812560-872812793.png)

#### 2.view

#### 3.url

#### 4.html前端修改

### 3.2 授课老师收藏

### 3.3 公开课收藏

## 10.4 我的消息

### 1.模板继承

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122172746225-120786854.png)



### 2.view中编写逻辑

```python
class MymessageView(View):
    """
    我的消息
    """
    def get(self, request):
        all_messages = UserMessage.objects.filter(user=request.user.id)

        #用户进入个人消息后清空未读消息的记录
        all_unread_messages = UserMessage.objects.filter(user=request.user.id, has_read=False)
        for unread_message in all_unread_messages:
            unread_message.has_read = True
            unread_message.save()

        #对个人消息进行分页
        try:
            page = request.GET.get('page', 1)
        except PageNotAnInteger:
            page = 1

        p = Paginator(all_messages, 5, request=request)

        messages = p.page(page)
        return render(request, 'usercenter-message.html', {
            "messages":messages
        })
```

### 3. url

```
url(r'^mymessage/$', MymessageView.as_view(), name="mymessage"),
```

### 4.前端html

* 在base里的登录后跳转也有
* 在usercenter-base.html页面中修改跳转到我的消息页面的url：

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122173944710-981552902.png)

* 修改我的消息页面显示消息的代码：

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122174101763-366801540.png)

* 分页显示：

  ![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122174119750-2052533034.png)

  



# 十一、首页全局配置

## 10.1首页全局配置

### index继承base.html

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181121152054737-1362586306.png)



* 没有面包屑，面包屑为空
* js文件

修改base.html页面中导航栏选中状态的代码：

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181121152238117-1299017059.png)

* 方法一：但是现在我们不知道当前是哪一个页面，因为后端没有传值过来，后台的每个view中添加current nav字段。然后向上传递到base页面，为了满足前台有current view的值，我们写的每个view都得加上这个字段。
* 利用request.path 访问地址，利用slice去地址的前几个（request是不加域名）





## 10.2 全局搜索

首页的全局搜索功能可以对课程，机构，教师进行全局搜索，搜索的代码放在deco-common.js文件中：

```python
function search_click(){
    var type = $('#jsSelectOption').attr('data-value'),
        keywords = $('#search_keywords').val(),
        request_url = '';
    if(keywords == ""){
        return
    }
    if(type == "course"){
        request_url = "/course/list?keywords="+keywords
    }else if(type == "teacher"){
        request_url = "/org/teacher/list?keywords="+keywords
    }else if(type == "org"){
        request_url = "/org/list?keywords="+keywords
    }
    window.location.href = request_url
}
```

只需要在课程列表接口、机构列表接口、讲师列表接口中加入搜索的逻辑即可：

```python
from django.db.models import Q
```

```python
search_keywords = request.GET.get('keywords', "")
if search_keywords:
    all_courses = all_courses.filter(Q(name__icontains=search_keywords) | 
                                     Q(desc__icontains=search_keywords) |
                                     Q(detail__icontains=search_keywords))
```

```python
search_keywords = request.GET.get('keywords', "")
if search_keywords:
    all_orgs = all_orgs.filter(Q(name__icontains=search_keywords) | Q(desc__icontains=search_keywords))
```

```python
search_keywords = request.GET.get('keywords', "")
if search_keywords:
    all_teachers = all_teachers.filter(Q(name__icontains=search_keywords) |
                                       Q(work_company__icontains=search_keywords) |
                                       Q(work_position__icontains=search_keywords))
```

## 10.3 登录后跳转

（要先设置个人中心页面）

包括了登录显示和登出

```html
{% if request.user.is_authenticated %}
    <div class="personal">
        <dl class="user fr">
            <dd>{{ user.username }}<img class="down fr" src="{% static "/static/images/top_down.png" %}"/></dd>
            <dt><img width="20" height="20" src="{{ MEDIA_URL }}{{ request.user.image }}"/></dt>
        </dl>
        <div class="userdetail">
           <dl>
             <dt><img width="80" height="80" src="{{ MEDIA_URL }}{{ request.user.image }}"/></dt>
             <dd>
                 <h2>{{ request.user.nick_name }}</h2>
                 <p>{{ request.user.username }}</p>
             </dd>
            </dl>
            <div class="btn">
             <a class="personcenter fl" href="{% url 'users:user_info' %}">进入个人中心</a>
             <a class="fr" href="{% url 'logout' %}">退出</a>
            </div>
        </div>
    </div>
    <a href="{% url 'users:mymessage' %}">
        <div class="msg-num"> <span id="MsgNum">{{ request.user.unread_nums }}</span></div>
    </a>
{% else %}
    <a style="color:white" class="fr registerbtn" href="{% url 'register' %}">注册</a>
    <a style="color:white" class="fr loginbtn" href="{% url 'login' %}">登录</a>
{% endif %}
```

这里需要设计个人中心和登出。个人中心上章设置好了，设置登出功能

登出

　　在views.py中编写登出的接口：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 class LogoutView(View):
2     """登出功能"""
3     def get(self, request):
4         # 使用django的logout函数登出
5         logout(request)
6 
7         # 将页面重定向到index
8         return HttpResponseRedirect(reverse('index'))
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　在MxOnline/urls.py中配置url：

```
url('^logout/$', LogoutView.as_view(), name="logout"),
```

　　然后在三个base页面修改登出的url。

![img](https://img2018.cnblogs.com/blog/1283298/201811/1283298-20181122175110258-2145530707.png)







## 10.4 首页功能开发

主要在轮播图，公开课，授课机构



* 在首页页面轮播课程需要在课程的model中添加is_banner字段，说明是否是轮播课程：

```python
is_banner = models.BooleanField(default=False, verbose_name=u"是否轮播")
```

记得迁移数据库

* 首页接口

```python
复制代码
 1 class IndexView(View):
 2     """首页"""
 3     def get(self, request):
 4         # 轮播图
 5         all_banners = Banner.objects.all().order_by('index')
 6 
 7         # 课程
 8         courses = Course.objects.filter(is_banner=False)[:6]
 9         # 轮播课程
10         banner_courses = Course.objects.filter(is_banner=True)[:3]
11 
12         # 机构
13         orgs = CourseOrg.objects.all()[:15]
14         return render(request, 'index.html', {
15             'all_banners': all_banners,
16             'courses': courses,
17             'banner_courses': banner_courses,
18             'orgs': orgs
19         })

```

* 修改url

  ```python
  url('^$', IndexView.as_view(), name="index"),
  ```

之前用的是template的静态的

* HTML前端修改让

说明1：课程

- 课程分is_banner=False和is_banner=True,两种课程的class属性不一样
- is_banner=True的class是  class="module1_2 box"
- is_banner=True的class是  class="module1_3 box",所以这里要class="module1_{{ forloop.counter|add:2 }}

说明2：课程机构

- 课程机构的class分为class=""和class="five"
- 这里要做个判断，class="{% if forloop.counter|divisibleby:5 %}five{% endif %}
- divisibleby过滤器：能不能整除

说明3：lolgin

- 当登出在login的时候发现刚才的设置都没生效，看不到图片，要改一下login的view
- 把之前的登录之后用render到‘index.html’改为return HttpResponseRedirect(reverse('index'))

