## 模块结构  

- 目录结构
- 文件命名
---

#### 目录结构  

data/: 演示数据和预定义数据  
models/: 模型文件  
security/: 控制访问权限和规则
controllers/: 控制器（HTTP路由）  
views/: 视图文件和模板文件  
static/: 网页资源（css, js, img, lib等）  

wizard/: 瞬态模型和视图（models.TransientModel）  
report/: 报表模型文件，和其视图文件  
tests/: python的测试文件  

#### 文件命名  

> 文件名仅使用小写字母，数字和下划线[a-z0-9_]。  
> 文件权限为644，文件夹为755 。

规范的文件命名对于通过快速查找信息和阅读代码很重要。本节说明如何在标准odoo模块中命名文件。  
我们使用下面的例子进行演示，它拥有两个主要模型plant.nursery和plant.order。  

关于模型（==models==），根据业务逻辑将将模型分开成单独的模型，每个单独的模型用其中的主模型名命名文件。  
继承的模型应该放到单独文件中，并以模型名命名文件。

```
addons/plant_nursery/
|-- models/
|   |-- plant_nursery.py (第一个主模型)
|   |-- plant_order.py (第二个主模型)
|   |-- res_partner.py (继承Odoo的模型)
```

关于安全性（==security==），访问权限和规则，应使用两个主要文件。  
第一个是在ir.model.access.csv文件中完成的访问权限的定义。  
用户组应该在```<module_name>_groups.xml```中定义。访问规则在$MODULENAME _security.xml中定义。

```
addons/plant_nursery/
|-- security/
|   |-- ir.model.access.csv
|   |-- plant_nusery_groups.xml
|   |-- plant_nusery_security.xml
|   |-- plant_order_security.xml
```

关于视图文件（==views==），后端视图应该和模型一样拆分，命名时以模型名加```_views.xml```结尾。  
视图类型包括：Tree, From, Kanban, Activity, Graph, Pivot等。  
将未与模型关联的菜单视图文件可命名为```<module_name>_menus.xml```。  
Qweb模板文件命名为```<module_name>_templates.xml```。  
导入JS， CSS等的文件命名为```assets.xml```。
```
addons/plant_nursery/
|-- views/
|   | -- assets.xml (导入 JS / CSS)
|   | -- plant_nursery_menus.xml (菜单)
|   | -- plant_nursery_views.xml (视图)
|   | -- plant_nursery_templates.xml (模板)
|   | -- plant_order_views.xml
|   | -- plant_order_templates.xml
|   | -- res_partner_views.xml
```

关于数据文件（==data==），按用途（演示或数据）和主要模型将它们划分。
文件名将以```_demo.xml```或```_data.xml```后缀。  
例如，下面是```plant_nersery```模型的数据文件，演示文件和邮件相关的数据文件：

```
addons/plant_nursery/
|-- data/
|   |-- plant_nursery_data.xml
|   |-- plant_nursery_demo.xml
|   |-- mail_data.xml
```

关于控制器（==controllers==），通常所有相关的控制器都放置在一个名为`<module_name>.py`的文件中。  
在这之前Odoo将该文件命名为`main.py`，但这个做法已经过时了。  
如果继承另一个控制器，请命名为`<inherit_module_name>.py`。

```
addons/plant_nursery/
|-- controllers/
|   |-- plant_nursery.py
|   |-- portal.py
|   |-- main.py (不推荐使用，被plant_nursery.py代替)
```

关于静态文件（==static==），JavaScript文件在全局范围内遵循与python模型相同的规则。
每个组件都应该在自己的文件中，并使用带有意义的名称。  
例如，于邮件模块的activity.js中的activity widget。
也可以创建子目录来构造“包”（有关详细信息，请参见Web模块）。  
JS widget的模板（静态XML文件）及其样式（scss文件）应采用相同的逻辑。  
请勿在Odoo外部链接数据（图像，库）：请勿在图像上使用URL，而应将其复制到代码库中。   


关于（==wizard==）文件，使用与模型文件和视图文件相同的命名规则，放在同一目录下。  
在Odoo的旧版本中可能会使用`wizard`关键词。  
```
addons/plant_nursery/
|-- wizard/
|   |-- make_plant_order.py
|   |-- make_plant_order_views.xml
```

关于报表文件（==report==）中的==统计视图==，使用以下规则。  
```
addons/plant_nursery/
|-- report/
|   |-- plant_order_report.py
|   |-- plant_order_report_views.xml
```
关于报表文件（==report==）中的==打印报表视图==，使用以下规则。  
```
addons/plant_nursery/
|-- report/
|   |-- plant_order_reports.xml (report actions, paperformat, ...)
|   |-- plant_order_templates.xml (xml report templates)
```





以下是该例子中完整的目录：
```
addons/plant_nursery/
|-- __init__.py
|-- __manifest__.py
|-- controllers/
|   |-- __init__.py
|   |-- plant_nursery.py
|   |-- portal.py
|-- data/
|   |-- plant_nursery_data.xml
|   |-- plant_nursery_demo.xml
|   |-- mail_data.xml
|-- models/
|   |-- __init__.py
|   |-- plant_nursery.py
|   |-- plant_order.py
|   |-- res_partner.py
|-- report/
|   |-- __init__.py
|   |-- plant_order_report.py
|   |-- plant_order_report_views.xml
|   |-- plant_order_reports.xml (report actions, paperformat, ...)
|   |-- plant_order_templates.xml (xml report templates)
|-- security/
|   |-- ir.model.access.csv
|   |-- plant_nusery_groups.xml
|   |-- plant_nusery_security.xml
|   |-- plant_order_security.xml
|-- static/
|   |-- img/
|   |   |-- my_little_kitten.png
|   |   |-- troll.jpg
|   |-- lib/
|   |   |-- external_lib/
|   |-- src/
|   |   |-- js/
|   |   |   |-- widget_a.js
|   |   |   |-- widget_b.js
|   |   |-- scss/
|   |   |   |-- widget_a.scss
|   |   |   |-- widget_b.scss
|   |   |-- xml/
|   |   |   |-- widget_a.xml
|   |   |   |-- widget_a.xml
|-- views/
|   |-- assets.xml
|   |-- plant_nursery_menus.xml
|   |-- plant_nursery_views.xml
|   |-- plant_nursery_templates.xml
|   |-- plant_order_views.xml
|   |-- plant_order_templates.xml
|   |-- res_partner_views.xml
|-- wizard/
|   |--make_plant_order.py
|   |--make_plant_order_views.xml
```