# 调试模式

ThinkPHP有专门为开发过程而设置的调试模式，开启调试模式后，会牺牲一定的执行效率，但带来的方便和除错功能非常值得。

> 强烈建议在开发阶段始终开启调试模式（直到正式部署后关闭调试模式），方便及时发现隐患问题和分析、解决问题。

应用默认是部署模式，在开发阶段，可以修改环境变量`APP_DEBUG`开启调试模式，上线部署后切换到部署模式。

本地开发的时候可以在应用根目录下面定义`.env`文件。

> 通过`create-project`默认安装的话， 会在根目录自带一个`.example.env`文件，你可以直接更名为`.env`文件。

`.env`文件的定义格式如下：

```
// 设置开启调试模式
APP_DEBUG =  true
// 其它的环境变量设置
// ...
```

调试模式的优势在于：

- 开启日志记录，任何错误信息和调试信息都会详细记录，便于调试；
- 会详细记录整个执行过程；
- 模板修改可以即时生效；
- 通过Trace功能更好的调试和发现错误；
- 发生异常的时候会显示详细的异常信息；

> 由于调试模式没有任何缓存，因此涉及到较多的文件IO操作和模板实时编译，所以在开启调试模式的情况下，性能会有一定的下降，但不会影响部署模式的性能。

一旦关闭调试模式，发生错误后不会提示具体的错误信息，如果你仍然希望看到具体的错误信息，那么可以在`app.php`文件中如下设置：

```
// 显示错误信息
'show_error_msg'        =>  true,    
```

> 出于安全考虑，调试模式的默认异常页面不会显示环境变量。