## Javascript 和 CSS文件  

### 静态文件组织  
Odoo模块在如何构造各种文件方面有一些约定。我们在这里更详细地解释应该如何组织网络资产。  

首先要知道的是，Odoo服务器将（静态的）提供位于`static /`文件夹中但前缀为插件名称的所有文件。  
例如: `addons/web/static/src/js/some_file.js` 将会静态的在URL`your-odoo-server.com/web/static/src/js/some_file.js`上可用.

按照以下结构组织代码：
```
|- static
|   |- lib
|   |- src
|   |   |- css
|   |   |- fonts
|   |   |- img
|   |   |- js
|   |   |- scss
|   |   |- xml
|   |- tests
```

### Javascript编码准则

- 推荐给所有`JS`文件使用`use strict`
- 使用Linter(jshint)
- 永远不要添加最小化的Javascript库
- 使用CamelCase定义类名

### CSS编码准则  

- 使用`o_<module_name>`作为所有类名的前缀, 唯一例外是webclient：它仅使用o_前缀。
- 避免使用`id`
- 使用bootstrap
- 使用下划线加小写来命名类
