# Laravel 的验证码 gregwar/captcha

    TP有 topthink/think-captcha
    Laravel有 gregwar/captcha
    
### 1.require包

首先, composer.json中如下加入配置：

```
"require": {
        ...
        "gregwar/captcha": "1.*"
    },
```

然后，已成习惯的命令：
```
composer update
```

### 2.验证码输出
>可以将验证码图片保存文件：
```
$builder->save('out.jpg');
```

>可以直接输出图片到网页：
```
header('Content-type: image/jpeg');
$builder->output();
```
>可以生成内联图片：
```
<img src="<?php echo $builder->inline(); ?>" />
```

>可以直接把验证码输出在页面上的demo

```
<?php namespace App\Http\Controllers;

use App\Http\Requests;
use App\Http\Controllers\Controller;

use Illuminate\Http\Request;

//引用对应的命名空间
use Gregwar\Captcha\CaptchaBuilder;
use Session;

class IndexController extends Controller {

    /**
     * Display a listing of the resource.
     *
     * @return Response
     */
    public function captcha($tmp)
    {
                //生成验证码图片的Builder对象，配置相应属性
        $builder = new CaptchaBuilder;
        //可以设置图片宽高及字体
        $builder->build($width = 100, $height = 40, $font = null);
        //获取验证码的内容
        $phrase = $builder->getPhrase();

        //把内容存入session
        Session::flash('milkcaptcha', $phrase);
        //生成图片
        header("Cache-Control: no-cache, must-revalidate");
        header('Content-Type: image/jpeg');
        $builder->output();
        
        //debug时候打开，否则直接打开页面看到的是乱码，而不是图片；测试完成记得删除    
        //die(0);   
    }
}


然后设置路由并访问

Route::get('Index/captcha/{tmp}', 'IndexController@captcha');

```

>如果写在表单里
```
html部分：
  <a href="javascript:;" title="点击刷新验证码">
     <img src="{{ URL('verify/1') }}" data-tmp="{{ URL('verify/1') }}" alt="验证码" title="刷新图片" width="100" height="40" id="reloadVerifyCode" border="0">
  </a>


js部分：
<script type="text/javascript">

  //刷新验证码
  var reloadVerifyCode = function(){
      var verifyimg = $("#reloadVerifyCode").data("tmp");
      $("#reloadVerifyCode").attr("src", verifyimg+'?random='+Math.random());
  };
</script>
```

### 3.验证码验证

>方法1 用$builder->testPhrase方法进行验证

要注意的是 $builder 必须与生成验证码的$builder为同一个，否则不会成功
```
$userInput = $request->get('captcha');

if($builder->testPhrase($userInput)) {
    //用户输入验证码正确
    return '您输入验证码正确';
} else {
    //用户输入验证码错误
    return '您输入验证码错误';
}
```

>方法2 用session进行验证

要注意的是 此处用的是laravel session 的flash方法，即使用一次；
具体可看隔壁的laravel_test项目

```
$userInput = \Request::get('captcha');

if (Session::get('milkcaptcha') == $userInput) {
    //用户输入验证码正确
    return '您输入验证码正确';
} else {
    //用户输入验证码错误    
    return '您输入验证码错误';
}
```

最后记一个缺点：网上的各位都会反应，这个验证码会反应比较慢。。。确实