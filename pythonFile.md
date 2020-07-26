## Python  

- PEP8 标准  
- 导入  
- Python编程习惯  
- Odoo编程习惯


---

#### PEP8 标准  

Odoo 源码中尽可能地遵守Python标准, 但不包括以下情况:  
1. E501: 代码行过长
2. E301: 期待一个空行, 只找到零个
3. E302: 期待两个空行, 只找到一个  


#### 导入  

导入应遵循以下顺序:  
1. python外部库
2. Odoo定义的库
3. 从Odoo模块中导入  

```
# 1 : imports of python lib
import base64
import re
import time
from datetime import datetime
# 2 : imports of odoo
import odoo
from odoo import api, fields, models, _ # alphabetically ordered
from odoo.tools.safe_eval import safe_eval as eval
# 3 : imports from odoo addons
from odoo.addons.website.models.website import slug
from odoo.addons.web.controllers.main import login_redirect
```

#### Python编程习惯  

- 每个Python文件都应该使用`# -*- coding: utf-8 -*-` 作为第一行  
- 始终优先考虑代码可读性而非简洁性,语言功能和习惯用语  
- 不适用`.clone()`方法  
```
# bad
new_dict = my_dict.clone()
new_list = old_list.clone()
# good
new_dict = dict(my_dict)
new_list = list(old_list)
```

- 创建或更新Python字典  
```
# -- creation empty dict
my_dict = {}
my_dict2 = dict()

# -- creation with values
# bad
my_dict = {}
my_dict['foo'] = 3
my_dict['bar'] = 4
# good
my_dict = {'foo': 3, 'bar': 4}

# -- update dict
# bad
my_dict['foo'] = 3
my_dict['bar'] = 4
my_dict['baz'] = 5
# good
my_dict.update(foo=3, bar=4, baz=5)
my_dict = dict(my_dict, **my_dict2)
```  

- 使用有意义的变量名, 类名, 方法名  
- 临时变量, 临时变量可以使代码更加清晰, 但是这不意味着可以无限制的创建临时变量  
```
# 无意义
schema = kw['schema']
params = {'schema': schema}
# 简化后
params = {'schema': kw['schema']}
```

- 使用多个返回时, 应该简化返回方式
```
# 有点复杂
def axes(self, axis):
        axes = []
        if type(axis) == type([]):
                axes.extend(axis)
        else:
                axes.append(axis)
        return axes

 # 简化后
def axes(self, axis):
        if type(axis) == type([]):
                return list(axis) # clone the axis
        else:
                return [axis] # single-element list
```

- 了解内置方法, 至少了解[Python的内置方法](http://docs.python.org/library/functions.html)  
```
value = my_dict.get('key', None) # 非常多余, 字典中没有这个'key', get默认返回None
value = my_dict.get('key') # 正确
```
同时, `if 'key' in my_dict` 和 `if my_dict.get('key')` 拥有完全不同的意义, 请确保你使用了正确的方式  

- 使用`map, filter, sum, ...` 等内置方法来简化对字典, 列表等的理解难易性
- 集合, 当其值为空时, 它也表示`False`, 有任意值时为`True`  
所以可以使用`if some_collection:` 来代替 `if len(some_colection):`  
- 使用dict.setdefault()方法  
```
# 代码更长, 更难阅读
values = {}
for element in iterable:
    if element not in values:
        values[element] = []
    values[element].append(other_value)

# 使用setdefault()方法
values = {}
for element in iterable:
    values.setdefault(element, []).append(other_value)
```

- 未复杂的代码加上简单的注释  
- 额外的[python准则](http://python.net/~goodger/projects/pycon/2007/idiomatic/handout.html ), 可能有点过时, 但和Odoo关联程度高  


#### Odoo 编程习惯  

> 避免创建生成器和装饰器：仅使用Odoo API提供的生成器和装饰器。  
> 与python中一样，使用filtered, mapped, sorted等方法来简化代码读取和性能。  

- 是你的方法批量工作  
当添加一个方法时, 确保它可以处理复数的记录, 可以通过循环`self`  
```
def my_method(self)
    for record in self:
        record.do_cool_stuff()
```
对于性能问题，例如在开发“统计按钮”时，请勿循环执行`search`或`search_count`。建议使用read_group方法，在一个请求中计算所有值。  
```
def _compute_equipment_count(self):
""" 计算每个类别的装备数量 """
    equipment_data = self.env['hr.equipment'].read_group([('category_id', 'in', self.ids)], ['category_id'], ['category_id'])
    mapped_data = dict([(m['category_id'][0], m['category_id_count']) for m in equipment_data])
    for category in self:
        category.equipment_count = mapped_data.get(category.id, 0)
```

- 传递上下文环境  

上下文是无法修改的冻结字典。要调用具有不同上下文的方法，应使用with_context方法  
```
# 所有上下文被替换
records.with_context(new_context).do_stuff()
# additionalal_context值将覆盖本地上下文
records.with_context(**additionnal_context).do_other_stuff() 
```
> 警告  
> 在上下文中传递参数可能会有危险的副作用。由于值会自动传递，因此可能会出现一些意外行为。  
> 在上下文中使用default_my_field键时调用模型的create()方法将为相关模型设置my_field的默认值。  
> 但是，如果在此创建过程中创建了具有字段名为`my_field`的其他对象，则它们的默认值也将被设置。  

如果需要创建影响某个对象行为的关键上下文，请在其前面加上模块名称以隔离意料之外的影响。  


- 不要绕过ORM层  

当ORM可以做同样的事情时，绝对不要直接使用数据库游标！这样，您将绕过所有ORM功能，可能绕过事务，访问权限等。而且使代码更难阅读，安全性也可能降低。

```
# very very wrong
self.env.cr.execute('SELECT id FROM auction_lots WHERE auction_id in (' + ','.join(map(str, ids))+') AND state=%s AND obj_price > 0', ('draft',))
auction_lots_ids = [x[0] for x in self.env.cr.fetchall()]

# no injection, but still wrong
self.env.cr.execute('SELECT id FROM auction_lots WHERE auction_id in %s '\
           'AND state=%s AND obj_price > 0', (tuple(ids), 'draft',))
auction_lots_ids = [x[0] for x in self.env.cr.fetchall()]

# better
auction_lots_ids = self.search([('auction_id','in',ids), ('state','=','draft'), ('obj_price','>',0)])
```

- 请不要进行SQL注入  

使用手动SQL查询时，必须注意不要引入SQL注入漏洞。当用户输入被错误地过滤或引用不正确时，就会出现此漏洞，从而使攻击者可以在SQL查询中引入不需要的子句（例如，规避过滤器或执行UPDATE或DELETE命令）。  

确保安全的最佳方法是永远不要使用Python字符串串联(`str1 + str2`)
或字符串参数插值(`string %s, self.name`)将变量传递到SQL查询字符串。  

第二个原因(几乎同样重要), 决定如何格式化查询参数的方式是数据库抽象层（psycopg2）的工作，而不是您的工作！
例如，psycopg2知道，当您传递值列表时，需要将其格式化为用逗号分隔的列表，并用括号括起来！  

```
# the following is very bad:
#   - it's a SQL injection vulnerability
#   - it's unreadable
#   - it's not your job to format the list of ids
self.env.cr.execute('SELECT distinct child_id FROM account_account_consol_rel ' +
           'WHERE parent_id IN ('+','.join(map(str, ids))+')')

# better
self.env.cr.execute('SELECT DISTINCT child_id '\
           'FROM account_account_consol_rel '\
           'WHERE parent_id IN %s',
           (tuple(ids),))
```
这非常重要，因此在重构时也请小心，最重要的是不要复制这些模式！   
这是一个令人难忘的示例，可以帮助您记住问题所在（但不要在此处复制代码）。在继续之前，请确保阅读pyscopg2的在线文档以了解如何正确使用它：  
[查询参数问题](http://initd.org/psycopg/docs/usage.html#the-problem-with-the-query-parameters)  
[使用psycopg2传递参数](http://initd.org/psycopg/docs/usage.html#passing-parameters-to-sql-queries)  
[推荐的参数类型](http://initd.org/psycopg/docs/usage.html#adaptation-of-python-values-to-sql-types)  

- 多考虑扩展的可能性  
[单一职责原则:](http://en.wikipedia.org/wiki/Single_responsibility_principle)
函数和方法不应包含太多逻辑, 建议使用许多小型和简单的方法来组成复杂的逻辑, 而不是使用少数大型和复杂的方法。  
应该避免在方法中对业务逻辑进行硬编码即写死，这使得难以扩展。  
```
# do not do this
# 修改domain或条件将会覆盖整个方法
def action(self):
    ...  # long method
    partners = self.env['res.partner'].search(complex_domain)
    emails = partners.filtered(lambda r: arbitrary_criteria).mapped('email')

# better but do not do this either
# 修改代码逻辑导致重复代码的某些部分
def action(self):
    ...
    partners = self._get_partners()
    emails = partners._get_emails()

# better
# minimum override
def action(self):
    ...
    partners = self.env['res.partner'].search(self._get_partner_domain())
    emails = partners.filtered(lambda r: r._filter_partners()).mapped('email')
```
为了示例起见，以上代码可扩展性很强，但必须考虑可读性并进行折中选择，
并相应地命名您的函数：精简而正确命名, 是可读/可维护代码和更严格文档的起点。  


- [不要关闭数据库连接](https://www.odoo.com/documentation/13.0/reference/guidelines.html#never-commit-the-transaction)  


- 正确的使用翻译方法  

Odoo使用“下划线”`_()`(类似GetText)的方法来表示代码中使用的静态字符串需要在运行时使用上下文语言进行翻译。可以通过以下方式在您的代码中调用此方法：  
`from odoo import _`  
使用它时，必须遵循一些非常重要的规则，以使其正常工作，并避免在翻译中充满无用的垃圾。  

基本上，此方法仅应用于在代码中手动编写的静态字符串，它将无法转换字段值（例如产品名称等）。必须使用相应字段上的translate标志来完成此操作。
规则非常简单：对underscore方法的调用应始终采用_（'literal string'）的形式，而应别无其他：  
```
# 正确的方式
error = _('This record is locked!')
error = _('Record %s cannot be modified!') % record
error = _("""This is a bad multiline example
             about record %s!""") % record
error = _('Record %s cannot be modified' \
          'after being validated!') % record

# 错误的演示, 注意括号的位置
# 这不会生效, 并且会影响翻译模块正常工作!
error = _('Record %s cannot be modified!' % record)

# 动态字符串, 拼接字符串等是被禁止在翻译方法中使用的.
error = _("'" + que_rec['question'] + "' \n")

# 字段值不会翻译, 这不会正常工作
error = _("Product %s is out of stock!") % _(product.name)
error = _("Product %s is out of stock!" % product.name)

# 但是你可定义额外的翻译字段
fields.Char(translate=True)
error = _("Product %s is not available!") % product.name

```
通常，在Odoo中，在处理字符串时，`%`优于`.format()`（当字符串中仅替换一个变量时）。这使社区译者的翻译更加容易。


#### 符号和约定  

- **模型名称(使用点号，模块名称作为前缀):**
    - 定义Odoo模型时：使用名称的单数形式（res.partner和sale.order而不是res.partnerS和saleS.orderS）
    - 定义Odoo瞬态（向导）时：使用`<related_base_model>.<action>`，其中`related_base_model`是与瞬态相关的基本模型（在models /中定义），而action是瞬态的简称。避免使用向导字。例如：`account.invoice.make，project.task.delegate.batch，…`
    - 在定义报告模型（例如SQL视图）时：基于Transient约定使用`<related_base_model>.report.<action>`。

- **Odoo Python类：使用CamelCase（面向对象的样式）。**
```
class AccountInvoive(models.Model):
    pass
```

- **变量名**  
    - 使用CamelCase定义模型对象
    - 使用下划线加小写定义普通变量
    - 使用`_id(Many2one)`和`_ids(One2many/Many2many)`作为关系字段后缀

- **方法名**  
    - 计算字段: `_compute_<field_name>`
    - 搜索方法: `_search_<field_name>`
    - 默认方法: `_default_<field_name>`
    - 选择方法: `_selection_<field_name>`
    - Onchange方法: `_onchange_<field_name>`
    - 约束方法: `_check_<constraint_name>`
    - Action方法: 使用`action_`作为前缀, 如果只针对单条记录, 则定在开始时定义`self.ensure_one()`

- **模型中属性的定义顺序**  
建议使用如下代码块的注释分割定义的方法.
    - 私有属性 `_name, _description, _inherit...`
    - 默认方法 `_default_get`
    - 定义字段
    - 以与字段定义顺序相同的计算，求逆和搜索方法
    - 选择方法（用于返回选择字段的计算值的方法）
    - CRUD方法(ORM覆盖)
    - Action 方法
    - 业务逻辑方法

```
class Company(models.Model):
    _name = "res.company"
    _description = 'Companies'
    
    # ----------------------------------------
    # Default Methods
    # ----------------------------------------

    # 字段定义
    
    # ----------------------------------------
    # Compute/oncahnge Methods
    # ----------------------------------------
    
    # ----------------------------------------
    # Selection Methods
    # ----------------------------------------

    # ----------------------------------------
    # ORM override
    # ----------------------------------------
    
    # ----------------------------------------
    # Actions Methods
    # ----------------------------------------
    
    # ----------------------------------------
    # Business Methods
    # ----------------------------------------
```