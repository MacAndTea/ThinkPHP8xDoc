# 路由分组

## 路由分组

路由分组功能允许把相同前缀的路由定义合并分组，这样可以简化路由定义，并且提高路由匹配的效率，不必每次都去遍历完整的路由规则（尤其是开启了路由延迟解析后性能更佳）。

使用`Route`类的`group`方法进行注册，给分组路由定义一些公用的路由设置参数，例如：

```
Route::group('blog', function () {
    Route::rule(':id', 'blog/read');
    Route::rule(':name', 'blog/read');
})->ext('html')->pattern(['id' => '\d+', 'name' => '\w+']);
```

分组路由支持所有的路由参数设置，具体参数的用法请参考[路由参数](https://doc.thinkphp.cn/v8_0/route_options.html)章节内容。

除了路由参数设置方法外，路由分组还有额外的方法，包括：

|方法|描述|
|---|---|
|`prefix`|设置分组的路由地址前缀|
|`mergeRuleRegex`|设置当前分组的路由合并解析|
|`dispatcher`|设置分组的调度|
|`layer`|分组绑定到控制器分级（`V8.1.0+`）|
|`namespace`|分组绑定到命名空间（`V8.1.0+`）|
|`controller`|分组绑定到控制器（`V8.1.0+`）|
|`class`|分组绑定到类（`V8.1.0+`）|
|`auto`|开启分组自动URL调度（`V8.1.0+`）|

如果仅仅是用于对一些路由规则设置一些公共的路由参数（也称之为虚拟分组），也可以使用：

```
Route::group(function () {
    Route::rule('blog/:id', 'blog/read');
    Route::rule('blog/:name', 'blog/read');
})->ext('html')->pattern(['id' => '\d+', 'name' => '\w+']);
```

路由分组支持嵌套，例如：

```
Route::group(function () {
    Route::group('blog', function () {
        Route::rule(':id', 'blog/read');
        Route::rule(':name', 'blog/read');
    });
})->ext('html')->pattern(['id' => '\d+', 'name' => '\w+']);
```

> 如果使用了嵌套分组的情况，子分组会继承父分组的参数和变量规则，而最终的路由规则里面定义的参数和变量规则为最优先。

可以使用`prefix`方法简化相同路由地址的定义，例如下面的定义

```
Route::group('blog', function () {
    Route::get(':id', 'blog/read');
    Route::post(':id', 'blog/update');
    Route::delete(':id', 'blog/delete');
})->ext('html')->pattern(['id' => '\d+']);
```

可以简化为

```
Route::group('blog', function () {
    Route::get(':id', 'read');
    Route::post(':id', 'update');
    Route::delete(':id', 'delete');
})->prefix('blog/')->ext('html')->pattern(['id' => '\d+']);
```

## 路由完全匹配

如果希望某个分组下面的路由都采用完全匹配，可以使用

```
Route::group('blog', function () {
    Route::get(':id', 'read');
    Route::post(':id', 'update');
    Route::delete(':id', 'delete');
})->completeMatch()->prefix('blog/')->ext('html')->pattern(['id' => '\d+']);
```

## 路由延迟解析

支持延迟路由解析，也就是说你定义的路由规则（主要是分组路由和域名路由规则）在加载路由定义文件的时候并没有实际注册，而是在匹配到路由分组或者域名的情况下，才会实际进行注册和解析，大大提高了路由注册和解析的性能。

默认是关闭延迟路由解析的，你可以在路由配置文件中设置：

```
// 开启路由延迟解析
'url_lazy_route'         => true,
```

开启延迟路由解析后，如果你需要生成路由反解URL，需要使用命令行指令

```
php think optimize:route
```

来生成路由缓存解析。

> 通过路由分组或者域名路由来定义路由才能发挥延迟解析的优势。

一旦开启路由的延迟解析，将会对定义的域名路由和分组路由进行延迟解析，也就是说只有实际匹配到该域名或者分组后才会进行路由规则的注册，避免不必要的注册和解析开销。

## 路由规则合并解析

同一个路由分组下的路由规则支持合并解析，而不需要遍历该路由分组下的所有路由规则，可以大大提升路由解析的性能。

对某个分组单独开启合并规则解析的用法如下：

```
Route::group('user', function () {
    Route::rule('hello/:name','hello');
    Route::rule('think/:name','think');
})->mergeRuleRegex();
```

这样该分组下的所有路由规则无论定义多少个都只需要匹配检查一次即可（实际上只会合并检查符合当前请求类型的路由规则）。

> `mergeRuleRegex`方法只能用于路由分组或者域名路由（域名路由其实是一个特殊的分组）。

或者在路由配置文件中设置开启全局合并规则（对所有分组有效）

```
// 开启路由合并解析
'route_rule_merge'	=> true,
```

## 分组子目录管理

支持子目录方式定义路由，方便路由定义较多情况下的高效管理（建议开启路由延迟解析）。

例如，你可以给每个分组定义一个子目录，然后在分组目录下面定义该分组下的路由即可。

```
├─route                 路由定义目录
│  ├─common.php          路由定义
│  ├─admin              admin分组
│  │  ├─route1.php       
│  │  ├─route2.php       
│  │  └─...             分组路由定义
│  ├─api                api分组
│  │  ├─route1.php       
│  │  ├─route2.php       
│  │  └─...             分组路由定义
│  └─...                更多路由定义
```

可以在common.php路由定义文件中添加

```
// 注册分组子目录路由
// 当分组子目录不存在的话 仅仅是注册一个空的路由分组
Route::group('admin');
Route::group('api');
```

> 分组目录下的路由定义文件中的所有路由和定义均针对所在分组，如果需要分组嵌套的话，和上面一样嵌套注册子目录分组即可。

如果在路由配置文件中开启`route_auto_group`参数，会自动扫描子目录分组，无需再定义下面的空分组。

```
Route::group('admin');
Route::group('api');
```

## 传入额外参数

可以统一给分组路由传入额外的参数

```
Route::group('blog',  function () {
    Route::rule(':id','Blog/read');
    Route::rule(':name','Blog/read');
})->ext('html')
->pattern(['id' => '\d+'])
->append(['group_id' => 1]);
```

上面的分组路由统一传入了`group_id`参数，该参数的值可以通过`Request`类的`param`方法获取。

## 指定分组调度

可以给路由分组单独指定调度类，例如：

```
Route::group('blog', function () {
    Route::rule(':id','Blog/read');
    Route::rule(':name','Blog/read');
})->dispatcher(GroupDispatcher::class);
```

## 分组绑定（`V8.1.0+`）

`8.1+`版本开始，路由分组支持绑定到命名空间、控制器或某个类，当分组下的路由都没有匹配成功会自动按绑定规则进行默认URL调度（不受强制路由影响），优先级高于分组MISS路由，并且支持多级分组，分组相关参数包括中间件仍然有效。

> 分组绑定后会自动追加对应的`prefix`，并且一旦对分组进行绑定，当该分组下的路由没有匹配到的话，会自动采用默认规则（相当于注册了一个分组MISS路由）。

### 绑定到命名空间

```
Route::group('blog', function () {
    Route::get(':id', 'blog/read');
    Route::post(':id', 'blog/update');
    Route::delete(':id', 'blog/delete');
})->pattern(['id' => '\d+'])->namespace('app\controller\home'); 
```

分组绑定到指定命名空间后，对没有匹配的请求会自动执行`app\controller\home` 命名空间下面的类和方法。

该方法默认会同时自动执行分组的

```
prefix('app\controller\home\');
```

### 绑定到控制器

```
Route::group('blog', function () {
    Route::get(':id', 'read');
    Route::post(':id', 'update');
    Route::delete(':id', 'delete');
})->pattern(['id' => '\d+'])->controller('blog'); 
```

分组绑定到控制器（支持分级控制器）后，对没有匹配的请求会自动执行该控制器类下面的操作方法。

该方法默认会同时自动执行分组的

```
prefix('blog/');
```

### 绑定到控制器分级

```
Route::group('admin', function () {
    Route::get('blog/:id', 'blog/read');
})->pattern(['id' => '\d+'])->layer('admin'); 
```

分组绑定到某个控制器分级后，对没有匹配的请求会自动执行分级目录下的控制器和方法。

该方法默认会同时自动执行分组的

```
prefix('admin/');
```

### 绑定到类

```
Route::group('blog', function () {
    Route::get(':id', 'read');
    Route::post(':id', 'update');
    Route::delete(':id', 'delete');
})->pattern(['id' => '\d+'])->class(Blog::class);
```

分组绑定到类后，对没有匹配的请求会自动执行该类的方法。  
该方法默认会同时自动执行分组的

```
prefix('\\' . Blog::class . '@');
```

### 简单绑定

如果你不需要定义分组路由，仅仅是需要进行分组绑定，可以直接使用下面的方式。

```
Route::group('blog')->layer('blog'); // 绑定到blog 控制器分级
Route::group('blog')->controller('blog'); // 绑定到blog控制器
Route::group('blog')->class(Blog::class); // 绑定到blog类
Route::group('blog')->namespace('app\controller\blog'); // 绑定到命名空间
```

## 自动URL调度

> 本功能需要`V8.1.0+`版本支持

可以为某个分组开启自动URL调度（内部仍然采用绑定机制，即绑定到控制器分级）。

```
Route::group('blog', function () {
    Route::get(':id', 'blog/read');
    Route::post(':id', 'blog/update');
    Route::delete(':id', 'blog/delete');
})->pattern(['id' => '\d+'])->auto(); // 为blog分组启用自动自动URL调度
```

默认的URL调度规则为

```
https://domainName/groupName.../controllerName/actionName
```

其中`groupName`可能为多级分组，内部实现为多级控制器机制，`groupName`相当于控制器的多级子目录。