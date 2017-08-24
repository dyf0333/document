# Laravel 使用过程中的一些坑
   
   （持续更新。。。）
   
## 1.发送邮件

在.env环境文件配置发送的邮箱相关设置；
```


MAIL_DRIVER=smtp
MAIL_HOST=smtp.qq.com
MAIL_PORT=465                         （可在qq邮箱文档查询，写错了连通会相应很慢，然后连不上）
MAIL_USERNAME=54974906@qq.com
MAIL_PASSWORD=XXXXXXXXXXXXXXX      （这个不是邮箱密码，是邮箱开启smtp后给的验证密码，写错了会报验证不通过错误）
MAIL_ENCRYPTION=ssl          （qq邮箱的，需要有ssl加密，不加会报加密错误）

（下面这俩，在默认的环境变量里没有，而config.php又会去用到，因此如果不手动加上，会报下图的错误：）
Expected response code 250 but got code "553", with message "553 Mail from must equal authorized user

MAIL_FROM_ADDRESS=54974906@qq.com
MAIL_FROM_NAME=支付测试邮箱
```

## 2.session

laravel提供的session整套很方便，各种机制，但是！！！

### 2.1调试时候获取不到session

#### 问题：
     
当ajax请求时候，调试我喜欢die或者exit，输出数据到页面或者直接直接用xdebug看过程数据，但是session数据总是获取不到，有些地方又可以

#### 原因：

共享session的话，路由得在一个group下;

die就会停止，session并不保存；应该先save，echo，再停止；

#### 解决办法：

在写入或清空session后，用函数

Session::save():

进行保存；然后再反馈数据时，例如一下的写法：

```
if ($request->ajax()) {
    header("Content-Type:application/json");
    echo json_encode(['code' => $code, 'message' => $message, 'data' => $data]);
    exit(0);
}

```

### 2.2构造函数获取不到session

#### 遇到的问题：
     
上几个项目基于TP5，权限判断以及session获取与写入在一个commonAuthController里的 __construct函数里（TP有自己的 _initialize函数，其实就是封装的构造函数）其他只要继承了这个controller的controller都会拥有权限判断和session获取功能；

但是在laravel构造函数里，怎么都获取不到session，也就没法进行权限判断；
    
#### 实际laravel逻辑流程：

Laravel 的 Session 是在 IlluminateSessionMiddlewareStartSession 这个中间件里启动的

构造函数在所有的middleware之前执行。所有构造函数中访问不到session

Laravel 的 Session 启动是在中间件中启动的，而中间件又分两种，全局中间件和指派中间件给路由，所谓全局中间件是指 Kernel.php 中 $middleware 里面，每一个 HTTP 请求时都会执行，而指派中间到路由就是 $routeMiddleware 和 $middlewareGroups (这部分只是分配中间件组给路由而已，也是分配给路由了)，找到 Kernel.php 父类，IlluminateFoundationHttpKernel::handle 方法就是处理请求的入口了，sendRequestThroughRouter 方法是进行路由分发的，里面的 Pipeline是关键.....然后各种的绕，这部分中间件走完(http全局中间件部分)，而 Route 中间件部分则是在 IlluminateRoutingRouter::dispatchToRoute 

#### 解决方法：

在中间件中获取session；这又分两种，一种自定义中间件，然后再kernel.php中引入；

一种直接在构造函数里写闭包的中间件，例：

```
public function __construct()
{
    parent::__construct();
    $this->middleware(function ($request, $next) {
        $sessionInfo = Session::get(AdminService::SESSION_KEY);
        if (empty($sessionInfo)) {
            return abort(403, '未登录');
        }
        return $next($request);
    });
}
```

## 3.DB获取到数据组，获取真是数据

在CURD操作时候，laravel提供了好用的Eloquent门面方法，但是也有一些坑；

### 问题、原因、解决办法：

比如或之前用TP时候，select出来的数据组（一个数组，每个数组元素的是TP的object），只要遍历数组，让每个元素 toArray()，或者干脆在不用select直接用colum就可以获得纯数组的数据，以便一些情景使用；

而laravel在get到数据后，数据每个元素不可toArray，必须用 $array['attributes'] 获取每个元素的真实数据

## 4.DB操作，新增数据

若习惯用 create 门面方法创建数据的，在laravel 的model里，必须 用

          protected $fillable = ['XXX','XXX']; //模型设置填充的字段

否则创建不成功
    
坑的是，报的错与$fillable无关

## 5.blade页面里，处理数据，字段里有id的会强制为int

   数据库以 id、name存储的数据，所以直接获取成  $list 并当做参数传给blade页面，
   
   然后在blade页面写一个select，例：
```  
<select class="select">
    <option value="">请选择</option>
    @foreach ($list as $v)
        <option value="{{ $v['id']}}">{{$v['name']}}</option>
    @endforeach
</select>
```

### 问题：
    
数据库存储的id是 01，02...这样的数据，或者可以理解成存储的是char

但是select的id出来的就变成了 1，2... 

如果是纯字符串就变成了0

### 解决办法：

获取数据时候把 id as 其他非id名字；例如

     select id as code from  tablename    where id =1 ;

在blade获取的时候，用新的名字，就不会只获取其中的数字了

```
<select class="select">
    <option value="">请选择</option>
    @foreach ($list as $v)
        <option value="{{ $v['code']}}">{{$v['name']}}</option>
    @endforeach
</select>
```