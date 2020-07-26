## XML文件  
 - 格式
 - XML id 和命名  

---

### 格式  

- 使用`<record>`标签来在XML文件中申明记录。  
- 将属性id放在model前面。
- 对于字段声明，name属性为第一序列。其他属性（窗口小部件，选项等）按重要性排序。
- 将记录按模型分组，当action/menu/views中存在依赖关系时，可能不适用。
- 如果文件中只有不可更新的数据，则可以在<odoo>标记上设置`noupdate = 1`，而不在<data>标记中设置。
```
<record id="view_id" model="ir.ui.view">
    <field name="name">view.name</field>
    <field name="model">object_name</field>
    <field name="priority" eval="16"/>
    <field name="arch" type="xml">
        <tree>
            <field name="my_field_1"/>
            <field name="my_field_2" string="My Label" widget="statusbar" statusbar_visible="draft,sent,progress,done" />
        </tree>
    </field>
</record>
```


### XML id和命名  

- 菜单：主菜单`<model_name>_menu`, 子菜单`<model_name>_menu_do_stuff`。  
- 视图：`<model_name>_view_<view_type>`。
- Action：主要的action`<model_name>_action`， 附属action`<model_name>_action_<detail>`。
- Action的视图：`<model_name>_action_view_<view_type>`。
- 组：`<model_name>_group_<group_name>`。
- 规则：`<model_name>_rule_<concerned_group>`。

> 记录的名称应与ID相同，并用点代替下划线。Action例外，因为他的名称也显示在视图中。

```
<!-- views  -->
<record id="model_name_view_form" model="ir.ui.view">
    <field name="name">model.name.view.form</field>
    ...
</record>

<record id="model_name_view_kanban" model="ir.ui.view">
    <field name="name">model.name.view.kanban</field>
    ...
</record>

<!-- actions -->
<record id="model_name_action" model="ir.act.window">
    <field name="name">Model Main Action</field>
    ...
</record>

<record id="model_name_action_child_list" model="ir.actions.act_window">
    <field name="name">Model Access Childs</field>
</record>

<!-- menus and sub-menus -->
<menuitem
    id="model_name_menu_root"
    name="Main Menu"
    sequence="5"
/>
<menuitem
    id="model_name_menu_action"
    name="Sub Menu 1"
    parent="module_name.module_name_menu_root"
    action="model_name_action"
    sequence="10"
/>

<!-- security -->
<record id="module_name_group_user" model="res.groups">
    ...
</record>

<record id="model_name_rule_public" model="ir.rule">
    ...
</record>

<record id="model_name_rule_company" model="ir.rule">
    ...
</record>
```


**继承XML**  

继承视图的Xml ID应该使用与原始记录相同的ID。有助于查找所有继承。  
最终的Xml ID会以创建该记录的模块作为前缀，因此ID没有重叠。  
但是`name`应以`.inherit.{details}`结尾，以便查看名称时，理解其覆盖的目的。  
```
<record id="model_view_form" model="ir.ui.view">
    <field name="name">model.view.form.inherit.module2</field>
    <field name="inherit_id" ref="module1.model_view_form"/>
    ...
</record>
```

以`primary`方式继承的视图不需要`inherit`后缀，因为它们是基于父视图的新记录。  
```
<record id="module2.model_view_form" model="ir.ui.view">
    <field name="name">model.view.form.module2</field>
    <field name="inherit_id" ref="module1.model_view_form"/>
    <field name="mode">primary</field>
    ...
</record>
```



