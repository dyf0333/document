# Laravel 文件下载
 
 
laravel自带提供下载功能

>也可用php原生的强制下载，修改header相关数据；另外一篇再讨论

直接在路由文件debug
```
Route::get('download',function(){
    return response()->download(realpath(base_path('public')).'/upload/test.xlsx', 'tourist.xlsx');
});
```

前端写一个 a 标签
```
<a class="download" href="{{ URL('tool/downloadFile') . '?file_path='  . $url}}" target="_blank"">点击下载</a>
```
也可以用js 方式
```
<button onclick="download()">Download</button>
<script>
    function downloadExcel() {
        var host = location.host;
        var url = "http://"+ host +"/download";
        window.open(url);
    }
</script>
```

## 说明：
### 1. 请确保你提供下载的文件存在。
### 2. realpath(base_path('public')).'/test.xlsx'是一个参数，当然可以用绝对路径如：C:\xampp\htdocs\laravelapp\public\test.xlsx
### 3. base_path 提供了相对路径的可行性，方便移植。
       也可以使用laravel框架的 public_path() 等函数；效果一样；
       不同框架都定义了自己的相对路径，比如thinkphp也有相应的函数
### 4. 一开始当然先直接访问地址看能不能下载，如http://localhost:100/download ，可以下载再用js调用。
### 5. 如果你遇到以下的错误，那么是php有一项服务没有打开。
```
LogicException in MimeTypeGuesser.php line 135:
Unable to guess the mime type as no guessers are available (Did you enable the php_fileinfo extension?)
```
解决：
```
去到php目录下找到php.ini文件，如C:\xampp\php\php.ini，找到 extension=php_fileinfo.dll，把前面的分号去掉，并重启一遍服务器，如：
C:\xampp\htdocs\laravelapp>php -S localhost:100 -t public
在phpinfo页面可以搜索fileinfo，说明你成功开启了。
```