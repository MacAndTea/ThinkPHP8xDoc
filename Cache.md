# 缓存

## 概述

ThinkPHP采用`think\Cache`类（实际使用`think\facade\Cache`类即可）提供缓存功能支持。

内置支持的缓存类型包括file、memcache、wincache、sqlite、redis。

> ThinkPHP的缓存类遵循`PSR-16`规范。

## [](https://doc.thinkphp.cn/v8_0/caches.html)设置

全局的缓存配置直接修改配置目录下面的`cache.php`文件。

新版的缓存支持多通道，你可以事先定义好所有的缓存类型及配置参数，然后在使用的时候可以随时切换。默认使用的是文件缓存类型，你可以添加`redis`缓存支持，例如：

```
return [
    'default'    =>    'file',
    'stores'    =>    [
        // 文件缓存
        'file'   =>  [
            // 驱动方式
            'type'   => 'file',
            // 设置不同的缓存保存目录
            'path'   => '../runtime/file/',
        ],  
        // redis缓存
        'redis'   =>  [
            // 驱动方式
            'type'   => 'redis',
            // 服务器地址
            'host'       => '127.0.0.1',
        ],  
    ],
];
```

缓存参数根据不同的缓存方式会有所区别，通用的缓存参数如下：

|参数|描述|
|---|---|
|type|缓存类型（或自定义驱动类名）|
|expire|缓存有效期 （默认为0 表示永久缓存）|
|prefix|缓存前缀（默认为空）|
|serialize|缓存序列化和反序列化方法|
|fail_delete|获取缓存失败后是否强制删除|

> 如果是自定义驱动，`type`的值则为自定义驱动的类名（包含命名空间）

## 使用

### 设置缓存

设置缓存有效期

```
// 缓存在3600秒之后过期
Cache::set('name', $value, 3600);
```

可以使用`DateInterval`对象设置多久以后过期

```
Cache::set('name', $value, DateInterval::createFromDateString('1 day'));
```

如果设置成功返回true，否则返回false。

### 缓存自增

针对数值类型的缓存数据，可以使用自增操作，例如：

```
Cache::set('name', 1);
// name自增（步进值为1）
Cache::inc('name');
// name自增（步进值为3）
Cache::inc('name',3);
```

> 只能对数字或者浮点型数据进行自增和自减操作。

### 缓存自减

针对数值类型的缓存数据，可以使用自减操作，例如：

```
// name自减（步进值为1）
Cache::dec('name');
// name自减（步进值为3）
Cache::dec('name',3);
```

### 获取缓存

获取缓存数据可以使用：

```
Cache::get('name'); 
```

如果`name`值不存在，则默认返回 `null`。

支持指定默认值，例如：

```
Cache::get('name',''); 
```

表示如果`name`值不存在，则返回空字符串。

支持传入闭包作为默认值获取

```
Cache::get('name',function(){
    // 动态返回数据
}); 
```

如果在获取缓存的时候发生异常，也会返回默认值，可以配置开启`fail_delete`参数，开启后，发生异常后会强制删除该缓存标识。

### 追加一个缓存数据

如果缓存数据是一个数组，可以通过`push`方法追加一个数据。

```
Cache::set('name', [1,2,3]);
Cache::push('name', 4);
Cache::get('name'); // [1,2,3,4]
```

### 删除缓存

```
Cache::delete('name'); 
```

### 获取并删除缓存

```
Cache::pull('name'); 
```

如果`name`值不存在，则返回`null`，支持指定默认值

```
Cache::pull('name',''); 
```

### 清空缓存

```
Cache::clear(); 
```

### 不存在则写入缓存数据后返回

```
Cache::remember('start_time', time());
```

如果start_time缓存数据不存在，则会设置缓存数据为当前时间。

第二个参数可以使用闭包方法获取缓存数据，并支持依赖注入。

```
Cache::remember('start_time', function(Request $request){
    return $request->time();
});
```

remember方法的第三个参数可以设置缓存的有效期。

### 缓存标签

支持给缓存数据打标签，例如：

```
Cache::tag('tag')->set('name1','value1');
Cache::tag('tag')->set('name2','value2');

// 清除tag标签的缓存数据
Cache::tag('tag')->clear();
```

缓存标签不会改变缓存的读取操作，所以获取方式依然是：

```
Cache::get('name1');
```

并支持同时指定多个缓存标签操作

```
Cache::tag(['tag1', 'tag2'])->set('name1', 'value1');
Cache::tag(['tag1', 'tag2'])->set('name2', 'value2');

// 清除多个标签的缓存数据
Cache::tag(['tag1','tag2'])->clear();
```

可以追加某个缓存标识到标签

```
Cache::tag('tag')->append('name3');
```

获取标签的缓存标识列表

```
Cache::getTagItems('tag');
```

### 获取缓存对象

可以获取缓存对象，并且调用驱动类的高级方法，例如：

```
// 获取缓存对象句柄
$handler = Cache::handler();
```

### 助手函数

系统对缓存操作提供了助手函数`cache`，用法如下：

```
// 设置缓存数据
cache('name', $value, 3600);
// 获取缓存数据
var_dump(cache('name'));
// 删除缓存数据
cache('name', NULL);
// 返回缓存对象实例
$cache = cache();
```

## 跨应用缓存

在多应用模式下，默认文件缓存数据是区分不同应用的，如果你希望缓存跨应用，可以设置一个统一的数据缓存`path`目录。

## 切换缓存类型

没有指定缓存类型的话，默认读取的是`default`缓存配置，可以动态切换

```
// 使用文件缓存
Cache::set('name','value',3600);
Cache::get('name');

// 使用Redis缓存
Cache::store('redis')->set('name','value',3600);
Cache::store('redis')->get('name');

// 切换到文件缓存
Cache::store('default')->set('name','value',3600);
Cache::store('default')->get('name');
```

如果要返回当前缓存类型对象的句柄，可以使用

```
// 获取Redis对象 进行额外方法调用
Cache::store('redis')->handler();
```

## 自定义驱动

如果需要自定义缓存驱动，需要继承`think\cache\Driver`类，并且实现`think\contract\CacheHandlerInterface`接口。

使用自定义驱动后，只需要配置缓存`type`的值为该驱动类名（包含命名空间）即可。