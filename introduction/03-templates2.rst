################################################################################
User Interfaces
################################################################################

.. contents::
  :local:
  :depth: 2

********************************************************************************
Interfaces building
********************************************************************************
Molis软件客户端的集成开发环境使用* JavaScript React库*创建，包括一个界面编辑器和一个虚拟界面设计器。接口页面是应用程序的重要组成部分，它提供从数据库表中检索和显示数据，创建用于接收用户输入数据的表单，将数据传递给合同以及在应用程序页面之间导航。接口页面，就像合同一样，存储在区块链中，这可以确保它们在软件客户端加载时防止伪造。 

Interface Template Engine
==============================
界面元素（页面和菜单）由Molis软件客户端的界面编辑器中程序员创建的模板在所谓的*模板引擎*中的验证节点上形成。所有界面页面均使用由平台开发人员开发的Protypo功能语言构建。使用* content * API命令从网络上的节点请求接口。模板引擎作为对这种请求的回复发送的内容不是HTML页面，而是由根据模板结构形成树的HTML标签组成的JSON代码。出于测试目的，可以使用包含要处理的模板名称的* template *参数将POST请求发送到`api / v2 / content`。

创建界面模版
==============================
界面可以使用专门的编辑器创建和编辑，可在Molis的管理工具的* Interface *部分中找到。编辑提供：

- 通过突出显示Protypo模板语言的关键字来编写界面页面的代码，
- 选择一个将显示在页面上的菜单，
- 编辑页面菜单，
- 配置编辑页面的权限（通常，通过在* ContractConditions *函数中指定具有权限的合同名称，或通过在*更改条件*字段直接指示访问权限），
- 推出可视界面设计师，
- 页面预览。

可视界面设计器
-----------------------------
Visual Interface Designer允许创建页面设计，而无需使用Protypo语言的界面源代码。 Designer允许使用拖放来设置页面上表单元素和文本的位置，以及配置页面块的大小和设计。 Designer提供了一套用于显示典型数据模型的即用型块：带有页眉，表单和信息面板的面板。在创建页面设计之后，可以在页面编辑器中添加程序逻辑（接收数据和条件结构）。 （将来，我们计划创建一个全面的可视化界面编辑器。）

Use of Styles
-----------------------------
默认情况下，界面页面使用Angular Bootstrap Angle类显示。如果需要，用户可以创建自己的样式。样式的存储是使用生态系统配置表的特殊样式表参数实现的。

Page Blocks
-----------------------------
要在多个界面页面上使用典型的代码片段，可以使用Insert命令创建页面块并将其嵌入到界面代码中。这些块可以在Molis的管理部分的接口页面上创建和编辑。对于块，就像页面一样，可以定义编辑权限。

Language Resources Editor
-----------------------------
Molis软件客户端包含一个使用Protypo模板语言的特殊功能 - LangRes的接口本地化机制，它使用用户在软件客户端（或浏览器）中选择的语言的相应文本行替换页面上的语言资源标签客户的网络版）。可以使用更简短的语法$ lable $来代替LangRes函数。由Simvolio语言的LangRes函数执行由合同发起的消息在弹出窗口中的转换。

语言资源可以在Molis软件客户端的管理工具的语言资源部分创建和编辑。一个语言资源由一个标签（名称）和该名称的翻译成不同的语言，并指示相应的双字符语言标识符（EN，FR，JP等）。

可以使用与语言表中任何其他表（Molis管理工具的表部分）相同的方式配置添加和更改语言资源的权限。

********************************************************************************
Protypo Template Language
********************************************************************************

Protypo函数提供了以下操作的实现：

- 从数据库中检索值：DBFind，
- 以表格和图表形式从数据库中检索数据，
- 分配和显示变量值，操作数据：SetVar，GetVar，Data，
- 日期/时间值的显示和比较：DateTime，Now，CmpTime，
- 使用各种用户数据输入字段构建表单：表单，ImageInput，输入，RadioGroup，选择，
- 通过显示错误消息验证表单字段中的数据：Validate，InputErr，
- 导航元素的显示：AddToolButton，LinkPage，Button，
- 呼叫合同：按钮，
- 创建HTML页面布局元素 - 具有指定CSS类选项的各种容器：Div，P，Span等，
- 将图像嵌入页面并上传图像：Image和ImageInput，
- 页面布局片段的条件显示：`If，ElseIf，Else`，
- 创建多级菜单，
- 界面本地化。

模板语言Protypo概述
==============================
页面模板语言是一种功能语言，允许使用`FuncName（parameters）`调用函数，并且可以将函数嵌套到彼此中。参数可以被指定为不带引号。不必要的参数可以被丢弃。

.. code:: js

      Text MyFunc(parameter number 1, parameter number 2) another text.
      MyFunc(parameter 1,,,parameter 4)
      
如果参数包含逗号，则应该用引号（后引号或双引号）括起来。如果一个函数只能有一个参数，可以在其中使用逗号而不用引号。另外，如果参数具有不成对的右括号，则应使用引号。

.. code:: js

      MyFunc("parameter number 1, the second part of first paremeter")
      MyFunc(`parameter number 1, the second part of first paremeter`)
      
如果将参数放在引号中，但参数本身包含引号，则可以在文本中使用不同类型的引号或将其加倍。
      
      .. code:: js

      MyFunc("parameter number 1, ""the second part of first"" paremeter")
      MyFunc(`parameter number 1, "the second part of first" paremeter`)
      
在功能描述中，每个参数都有一个特定的名称。您可以调用函数并按照它们声明的顺序指定参数，也可以按任意顺序按名称指定任何一组参数：''Parameter_name：Parameter_value''。这种方法可以安全地添加新的函数参数，而不会破坏与当前模板的兼容性。例如，所有这些调用在描述为“MyFunc（Class，Value，Body）”的函数的语言使用方面都是正确的：

.. code:: js

      MyFunc(myclass, This is value, Div(divclass, This is paragraph.))
      MyFunc(Body: Div(divclass, This is paragraph.))
      MyFunc(myclass, Body: Div(divclass, This is paragraph.))
      MyFunc(Value: This is value, Body: 
           Div(divclass, This is paragraph.)
      )
      MyFunc(myclass, Value without Body)
      
函数可以返回文本，生成HTML元素（例如，'Input''），或者使用嵌套的HTML元素（''Div'，P'，'Span''）创建HTML元素。在后一种情况下，应使用具有预定义名称* Body *的参数来定义嵌套元素。例如，两个* div *嵌套在另一个* div *中，可能如下所示：

.. code:: js

      Div(Body:
         Div(class1, This is the first div.)
         Div(class2, This is the second div.)
      )
      
要定义* Body *参数中描述的嵌套元素，可以使用以下表示：`MyFunc（...）{...}`。嵌套元素应在大括号中指定。

.. code:: js

      Div(){
         Div(class1){
            P(This is the first div.)
            Div(class2){
                Span(This is the second div.)
            }
         }
      }
      
如果需要连续多次指定同一个函数，则可以使用点而不是每次都写入函数名称。例如，以下几行是相等的：
     
     .. code:: js

     Span(Item 1)Span(Item 2)Span(Item 3)
     Span(Item 1).(Item 2).(Item 3)
     
该语言允许使用* SetVar *函数分配变量。要使用`＃varname＃`替换变量的值。

.. code:: js

     SetVar(name, My Name)
     Span(Your name: #name#)
     
要替换生态系统的语言资源，可以使用`$ langres $`，其中* langres *是语言源的名称

.. code:: js

     Span($yourname$: #name#)
     
以下变量是预定义的

* ``#key_id#`` - current user account identifier,
* ``#ecosystem_id#`` - current ecosystem identifier.

使用PageParams将参数传递给页面
-----------------------------
有许多支持* PageParams *参数的函数，用于在重定向到新页面时传递参数。例如，`PageParams：“param1 = value1，param2 = value2”`。参数值既可以是简单的字符串，也可以是具有替代值的行。当参数传递给页面时，会创建带有参数名称的变量;例如`＃param1＃`和`＃param2＃`。 

* ``PageParams: "hello=world"`` - the page will receive the hello parameter with world as value,
* ``PageParams: "hello=#world#"`` - the page will receive the hello parameter with the value of the world variable.

此外，* Val *函数允许从重定向中指定的表单获取数据。在这种情况下，

* ``PageParams: "hello=Val(world)"`` - the page will receive the hello parameter with the value of the world form element.


调用合约
-----------------------------
Protypo通过单击表单中的按钮（* Button *函数）来实现合同调用。一旦这个事件被启动，用户在界面表单的字段中输入的数据被传递给合同（如果表单域的名称与被叫合同的数据部分中的变量名称相对应，则数据将自动传输）。按钮功能允许打开一个模式窗口，供用户验证合同执行（Alert），并在成功执行合同之后重定向到指定页面，并将某些参数传递给此页面。   

********************************************************************************
Functions of Protypo
********************************************************************************

Operations with variables
==============================
GetVar(Name)
------------------------------
此函数返回当前变量的值（如果存在），或者如果未定义具有此名称的变量，则返回空字符串。 仅当请求编辑的树时才会创建带有* getvar *名称的元素。 “GetVar（varname）`和`＃varname＃`之间的区别在于，* varname *不存在，* GetVar *将返回一个空字符串，而*＃varname＃*将被解释为字符串值。

* *Name* - variable name.

.. code:: js

     If(GetVar(name)){#name#}.Else{Name is unknown}
      
SetVar(Name, Value)
------------------------------
Assigns a *Value* to a *Name* variable. 

* *Name* - name of the variable,
* *Value* - value of the variable, which can contain a reference to another variable.

.. code:: js

     SetVar(name, John Smith).(out, I am #name#)
     Span(#out#)      

Navigation
==============================     
AddToolButton(Title, Icon, Page, PageParams)
------------------------------
Adds a button to the buttons panel. Creates **addtoolbutton** element. 

* *Title* - button title,
* *Icon* - icon for the icon,
* *Page* - page name for the jump,
* *PageParams* - parmeters for the page.

.. code:: js

      AddToolButton(Help, help, help_page) 
      
Button(Body, Page, Class, Contract, Params, PageParams) [.Alert(Text,ConfirmButton,CancelButton,Icon)] [.Style(Style)]
------------------------------
Creates a **button** HTML element. This element creates a button, which sends a specified contract for execution.

* *Body* - child text or elements,
* *Page* - name of the page to redirect to,
* *Class* - classes for the button,
* *Contract* - name of the contract to execute,
* *Params* - list of values to pass to the contract. By default, values of contract parameters (data ``section``) are obtained from HTML elements (for example, input fields) with similarly-named identifiers (``id``). If the element identifiers differ from the names of contract parameters, then the assignment in the ``contractField1=idname1, contractField2=idname2`` format should be used. This parameter is returned to *attr* as an object ``{field1: idname1, field2: idname2}``,
* *PageParams* - parameters for redirection to a page in the following format: ``contractField1=idname1, contractField2=idname2``. In this case, variables with parameter names ``#contractField1#`` and ``#contractField2`` are created on the target page, and are assigned the specified values (see the parameter passing specifications in the "*Passing Parameters to a Page Using PageParams*" section above).

**Alert** - displays a message.

* *Text* - message text,
* *ConfirmButton* - confirm button caption,
* *CancelButton* - cancel button caption,
* *Icon* - icon.

**Style** - serves for specifying css styles.

* *Style* - css styles.

.. code:: js

      Button(Submit, default_page, mybtn_class).Alert(Alert message)
      Button(Contract: MyContract, Body:My Contract, Class: myclass, Params:"Name=myid,Id=i10,Value")
      
LinkPage(Body, Page, Class, PageParams) [.Style(Style)]
------------------------------
Creates a **linkpage** element – a link to a page.
 
* *Body* - child text or elements,
* *Page* - page to redirect to,
* *Class* - classes for this button,
* *PageParams* - redirection parameters,

**Style** - specifies css styles,

* *Style* - css styles.

.. code:: js

      LinkPage(My Page, default_page, mybtn_class)

Data Operations
==============================
And (Parameters)
------------------------------
该函数返回*和*逻辑运算的执行结果，括号中列出所有参数，并以逗号分隔。 如果参数值等于空字符串（`“”`），零或* false *，则参数值为'false'。 在所有其他情况下，参数值为“true”。 该函数返回1，如果为true，则在所有其他情况下返回0。 名为`and`的元素仅在请求编辑的树时创建。

.. code:: js

      If(And(#myval1#,#myval2#), Span(OK))
      
Calculate(Exp, Type, Prec)
------------------------------
This function returns the result of an arithmetic expression passed in the **Exp** parameter. The following operations can be used: +, -, *, /, and parenthesis (). 

* **Exp** - arithmetic expression. Can contain numbers and *#name#* variables.
* **Type** - result data type: **int, float, money**. If not specified, then the result type will be *float* in case there are numbers with a decimal point, or *int* in all other cases.
* **Prec** - the number of significant digits after the point can be specified for *float* and *money* types.

Calculate( Exp: (342278783438+5000)*(#val#-932780000), Type: money, Prec:18 )
Calculate(10000-(34+5)*#val#)
Calculate("((10+#val#-45)*3.0-10)/4.5 + #val#", Prec: 4)      

CmpTime(Time1, Time2)
------------------------------
This function compares two time values in the same format (preferably, standard format - YYYY-MM-DD HH:MM:SS, but any format can be used provided that the sequence is followed from years to seconds). Returns:

* **-1** - Time1 < Time2, 
* **0** - Time1 = Time2, 
* **1** - Time1 > Time2.

.. code:: js

     If(CmpTime(#time1#, #time2#)<0){...}
     
DateTime(DateTime, Format)
------------------------------
This function displays time and date in the specified format. 

 *  *DateTime* - time and date in standard format ``2006-01-02T15:04:05``.
 *  *Format* -  format template: ``YY`` 2-digit year format, ``YYYY`` 4-digit year format, ``MM`` - month, ``DD`` - day, ``HH`` - hours, ``MM`` - minutes, ``SS`` – seconds. Example: ``YY/MM/DD HH:MM``. If the format is not specified, the *timeformat* parameter value set in the *languages* table will be used. If this parameter is absent, the ``YYYY-MM-DD HH:MI:SS`` format will be used instead.
 
 .. code:: js

    DateTime(2017-11-07T17:51:08)
    DateTime(#mytime#,HH:MI DD.MM.YYYY)

Now(Format, Interval) 
------------------------------
This function returns the current time in the specified format, which by default is the UNIX format (number of seconds elapsed since January 1, 1970). If the requested time format is *datetime*, then date and time are shown as ``YYYY-MM-DD HH:MI:SS``. An interval can be specified in the second parameter (for instance, *+5 days*).

* *Format* - output format with a desired combination of ``YYYY, MM, DD, HH, MI, SS`` or *datetime*,
* *Interval* - backward or forward time offset.

.. code:: js

       Now()
       Now(DD.MM.YYYY HH:MM)
       Now(datetime,-3 hours)

Or(parameters)
------------------------------
This function returns a result of the **IF** logical operation with all parameters specified in parentheses and separated by commas. The parameter value is considered ``false`` if it equals an empty string (``""``), 0 or ``false``. In all other cases the parameter value is considered ``true``. The function returns 1 for true or 0 in all other cases. Element named **or** is created only when the tree for editing is requested. 

.. code:: js

      If(Or(#myval1#,#myval2#), Span(OK))

Displaying data
==============================
Code(Text)
------------------------------
Creates a **code** element for displaying the specified code.
	
* *Text* - source code, which will be displayed.

.. code:: js

      Code( P(This is the first line.
          Span(This is the second line.))
      )  
      
Chart(Type, Source, FieldLabel, FieldValue, Colors)
------------------------------
Creates an HTML diagram.

* *Type* - diagram type,
* *Source* - name of the data source, for example, from the *DBFind* command,
* *FieldLabel* - name of the field used for headers,
* *FieldValue* - name of the field used for values,
* *Colors* - list of used colors.

.. code:: js

      Data(mysrc,"name,count"){
          John Silver,10
          "Mark, Smith",20
          "Unknown ""Person""",30
      }
      Chart(Type: "bar", Source: mysrc, FieldLabel: "name", FieldValue: "count", Colors: "red, green")
	  
ForList(Source, Body)
------------------------------
Displays a list of elements from the *Source* data source in the template format set out in *Body*, and creates the **forlist** element.

* *Source* - data source from *DBFind* or *Data* functions,
* *Body* - a template to insert the elements in.

.. code:: js

      ForList(mysrc){Span(#name#)}
      
Image(Src,Alt,Class) [.Style(Style)]
------------------------------
Creates an **image** HTML element.
 
* *Src* - image source, file or ``data:...``,
* *Alt* - alternative text for the image,
* *Сlass* - list of classes.

.. code:: js

    Image(\images\myphoto.jpg)    
    
MenuGroup(Title, Body, Icon) 
------------------------------
Forms a nested submenu in the menu and returns the **menugroup** element. The *name* parameter will also return the value of *Title* before replacement with language resources.

* *Title* - menu item name,
* *Body* - child elements in submenu,
* *Icon* - icon.

.. code:: js

      MenuGroup(My Menu){
          MenuItem(Interface, sys-interface)
          MenuItem(Dahsboard, dashboard_default)
      }
      
MenuItem(Title, Page, Params, Icon, Vde) 
------------------------------
Creates a menu item and returns the **menuitem** element. 

* *Title* - menu item name,
* *Page* - page to redirect to,
* *Params* - parameters, passed to the page in the *var:value* format, separated by commas,
* *Icon* - icon,
* *Vde* -  is a parameter that defines the transition to a virtual ecosystem. If ``Vde: true``, then the link redirects to VDE; if ``Vde: false``, then the link redirects to the blockchain; if the parameter was not specified, then it is defined based on where the menu was loaded.

.. code:: js

       MenuItem(Interface, interface)
       
Table(Source, Columns) [.Style(Style)]
------------------------------
Создает HTML элемент **table**.

* *Source* - data source name as specified, for example, in the *DBFind* command,
* *Columns* - Headers and corresponding column names, as follows: ``Title1=column1,Title2=column2``.

**Style** - specifies css styles,

* *Style* - css styles.

.. code:: js

      DBFind(mytable, mysrc)
      Table(mysrc,"ID=id,Name=name")
      
Receiving data
==============================
Address (account)
------------------------------
This function returns the account address in the ``1234-5678-...-7990`` format given the numerical value of the address; if the address is not specified, the address of the current user will be taken as the argument. 

.. code:: js

      Span(Your wallet: Address(#account#))

Data(Source,Columns,Data) [.Custom(Column,Body)]
------------------------------
Creates element **data** and fills it with specified data and put into the *Source*, that then should be specified in *Table* and other commands resivieng *Source* as the input data. The sequence of column names corresponds to that of *data* entry values.
 
* *Source* - data source name. You can specify any name, which will have to be included in other commands later on (ex. *Table*) as a data source,
* *Columns* - list of columns,
* *Data* - one data entry per line, divided into columns by commas. Data should be in the same order as set in *Columns*, Entry values can be embraced in double quotes. If you need to use quote marks in the text, use double quotes.
* **Custom** - allows for assigning calculated columns for data. For example, you can specify a template for buttons and additional page layout elements. Several calculated columns can be assigned. As a rule, these fields are assigned for output to *Table* and other commands that use received data,
 
  * *Column* - column name. A unique name should be assigned,
  * *Body* - a code fragment. You can obtain values from other columns in this entry using ``#columnname#`` and use them in this code fragment.

.. code:: js

    Data(mysrc,"id,name"){
	"1",John Silver
	2,"Mark, Smith"
	3,"Unknown ""Person"""
     }.Custom(link){Button(Body: View, Class: btn btn-link, Page: user, PageParams: "id=#id#"}    


DBFind(table, Source) [.Columns(columns)] [.Where(conditions)] [.WhereId(id)] [.Order(name)] [.Limit(limit)] [.Offset(offset)] [.Ecosystem(id)] [.Custom(Column,Body)][.Vars(Prefix)]
------------------------------
Creates the **dbfind** element, fills it with data from the *table* table, and puts it to the *Source* structure. The *Source* structure can be then used in *Table* and other commands that receive *Source* as input data. The sequence of records in *data* should correspond to the sequence of column names.

* *Name* - table name,
* *Source* - arbitrary data source name,
 
* **Columns** - list of columns to be returned. If not specified, all columns will be returned,
* **Where** - search condition. For example, ``.Where(name = '#myval#')``,
* **WhereId** - search by ID. For example, ``.WhereId(1)``,
* **Order** - sort by this field,
* **Limit** - number of returned rows. Default value = 25, maximum value = 250,
* **Offset** - offset of returned rows,
* **Ecosystem** - ecosystem ID. By default, data is taken from the specified table in the current ecosystem,
* **Custom** - allows for assigning calculated columns for data. For example, you can specify a template for buttons and additional page layout elements. You can assign any number of calculated columns. As a rule, these fields are assigned for output to *Table* and other commands that use received data,
 
  * *Column* - column name. A unique name should be assigned,
  * *Body* - a code fragment. You can obtain values from other columns in this entry using **#columnname#** and use them in this code fragment.
  
  * **Vars** - the function generates a set of variables with values from the database table, obtained from this query. When specifying this function, the *Limit* parameter automatically becomes equal to 1 and only one record is returned,

* *Prefix* - * *Prefix* - prefix function is used to generate names for variables, to which the values of the resulting row are saved: variables are of format *#prefix_id#, #prefix_name#*, where the column name follows the underscore sign.

.. code:: js

    DBFind(parameters,myparam)
    DBFind(parameters,myparam).Columns(name,value).Where(name='money')
    DBFind(parameters,myparam).Custom(myid){Strong(#id#)}.Custom(myname){
       Strong(Em(#name#))Div(myclass, #company#)
    }
    
EcosysParam(Name, Index, Source) 
------------------------------
This function gets a parameter value from the parameters table of the current ecosystem. If there is a language resource for the resulting name, it will be translated accordingly.
 
* *Name* - value name,
* *Index* - in cases where the requested parameter is a list of elements separated by commas, you can specify an index starting from 1. For example, if ``gender = male,female``, then ``EcosysParam(gender, 2)`` will return *female*,  
* *Source* - you can receive the parameter values separated by commas as a *data* object. After that you will be able to specify this list as a data source for both *Table* and *Select*. If you specify this parameter, then the function will return a list as a *Data* object, not a separate value.

.. code:: js

     Address(EcosysParam(founder_account))
     EcosysParam(gender, Source: mygender)
 
     EcosysParam(Name: gender_list, Source: src_gender)
     Select(Name: gender, Source: src_gender, NameColumn: name, ValueColumn: id)
     
LangRes(Name, Lang)
------------------------------
Returns a specified language resource. In case of request to a tree for editing it returns the ``$langres$`` element.

* *Name* - name of language resource,
* *Lang* - by default, returned is the language defined in request to *Accept-Language*. You can specify your own two-character language identifier.

.. code:: js

      LangRes(name)
      LangRes(myres, fr)     

SysParam(Name) 
------------------------------
Displays the value of a system parameter from the system_parameters table. 

* *Name* - parameter name.

.. code:: js

     Address(SysParam(founder_account))

Elements of data formatting
============================== 
Div(Class, Body) [.Style(Style)]
------------------------------
Creates a **div** HTML element.

* *Class* - classes for this *div*,
* *Body* - child elements.

**Style** - serves for specifying css styles,

* *Style* - css styles.

.. code:: js

      Div(class1 class2, This is a paragraph.)
      
Em(Body, Class)
------------------------------
Creates an **em** HTML element.

* *Body* - child text or elements,
* *Class* - classes for this *em*.

.. code:: js

      This is an Em(important news).
      
P(Body, Class)
------------------------------
Creates a **p** HTML element.

* *Body* - child text or elements,
* *Class* - classes for this *p*,

**Style** - specifies css styles,

* *Style* - css styles.

.. code:: js

      P(This is the first line.
        This is the second line.)
	
SetTitle(Title)
------------------------------
Sets the page title. The element **settitle** will be created.

* *Title* - page title.

.. code:: js

     SetTitle(My page)	
	
Label(Body, Class, For) [.Style(Style)]
------------------------------
Creates a **label** HTML element.

* *Body* - child text or elements,
* *Class* - classes for this *label*,
* *For* - this label's *for* value,

**Style** - serves for specifying css styles,

* *Style* - css styles.

.. code:: js

      Label(The first item).	
	
Span(Body, Class) [.Style(Style)]
------------------------------
Creates a **span** HTML element.

* *Body* - child class or elements,
* *Class* - classes for this *span*,

**Style** - specifies css styles,

* *Style* - css styles.

.. code:: js

      This is Span(the first item, myclass1).
      
Strong(Body, Class)
------------------------------
Creates a **strong** HTML element.

* *Body* - child text or elements,
* *Class* - classes for this *strong*.

.. code:: js

      This is Strong(the first item, myclass1).
      
Elements of forms
==============================      
Form(Class, Body) [.Style(Style)]
------------------------------
Creates a **form** HTML element.

* *Class* - classes for this *form*,
* *Body* - child elements.

**Style** - specifies css styles.

* *Style* - css styles.

.. code:: js

      Form(class1 class2, Input(myid))
      
ImageInput(Name, Width, Ratio, Format) 
------------------------------
This function creates an **imageinput** element for image upload. In the third parameter you can specify either image height or aspect ratio to apply: *1/2*, *2/1*, *3/4*, etc. The default width is 100 pixels with *1/1* aspect ratio.

* *Name* - element name,
* *Width* - width of cropped image,
* *Ratio* - aspect ratio (width to height) or height of the image,
* *Format* - format of the uploaded image,

.. code:: js

   ImageInput(avatar, 100, 2/1)    
   
Input(Name,Class,Placeholder,Type,Value) [.Validate(validation parameters)] [.Style(Style)]
------------------------------
Creates an **input** HTML element.

* *Name* - element name,
* *Class* - classes for the *input*,
* *Placeholder* - *placeholder* for the *input*,
* *Type* - *input* type,
* *Value* - element value.

**Validate** - validation parameters.

**Style** - serves for specifying css styles.

* *Style* - css styles.

.. code:: js

      Input(Name: name, Type: text, Placeholder: Enter your name)
      Input(Name: num, Type: text).Validate(minLength: 6, maxLength: 20)

InputErr(Name,validation errors)]
------------------------------
Creates an **inputerr** element with validation error texts.

* *Name* - name of the corresponding **Input** element.

.. code:: js

      InputErr(Name: name, 
          minLength: Value is too short, 
          maxLength: The length of the value must be less than 20 characters)
	  

RadioGroup(Name, Source, NameColumn, ValueColumn, Value, Class) [.Validate(validation parameters)] [.Style(Style)]
------------------------------
Creates a **radiogroup** element.

* *Name* - element name,
* *Source* - data source name from *DBFind* or *Data* functions,
* *NameColumn* - column name to use a source of element names,
* *ValueColumn* - column name to use a source of element values. Columns created using Custom should not be used in this parameter,
* *Value* - default value,
* *Class* - classes for the element,

**Validate** - validation parameters,

**Style** - specification of css styles,
 
* *Style* - css styles.

.. code:: js

      DBFind(mytable, mysrc)
      RadioGroup(mysrc, name)	  
      
Select(Name, Source, NameColumn, ValueColumn, Value, Class) [.Validate(validation parameters)] [.Style(Style)]
------------------------------
Creates a **select** HTML element.

* *Name* - element name,
* *Source* - data source name. For example, *DBFind* or *Data*,
* *NameColumn* - column from which the element names will be taken,
* *ValueColumn* - column from which the element values will be taken. Columns created using Custom should not be specified in this parameter,
* *Value* - default value,
* *Class* - element classes,

**Validate** - validation parameters,

**Style** - specification of css styles,

* *Style* - css styles.

.. code:: js

      DBFind(mytable, mysrc)
      Select(mysrc, name) 
      
Operations with code
=========================
If(Condition){ Body } [.ElseIf(Condition){ Body }] [.Else{ Body }]
------------------------------
Conditional statement. Returned are child elements of the first ``If`` or ``ElseIf`` with fulfilled ``Condition``. Otherwise, returned are child elements of ``Else``, if it exists.

* *Condition* - a condition is considered non-fulfilled if it equals an *empty string*, *0* or *false*. In other cases the condition is considered true.
* *Body* - child elements.

.. code:: js

      If(#value#){
         Span(Value)
      }.ElseIf(#value2#){Span(Value 2)
      }.ElseIf(#value3#){Span(Value 3)}.Else{
         Span(Nothing)
      }
   
Include(Name)
------------------------------
This command inserts a template with name *Name* in the code of a page. 

* *Name* - name of the block.

.. code:: js

      Div(myclass, Include(mywidget))
      
************************************************
Styles for mobile app
************************************************

Typography
==============================

Headings
------------------------------

* ``h1`` ... ``h6``

Emphasis Classes
------------------------------

* ``.text-muted``
* ``.text-primary``
* ``.text-success``
* ``.text-info``
* ``.text-warning``
* ``.text-danger``

Colors
------------------------------

* ``.bg-danger-dark``
* ``.bg-danger``
* ``.bg-danger-light``
* ``.bg-info-dark``
* ``.bg-info``
* ``.bg-info-light``
* ``.bg-primary-dark``
* ``.bg-primary``
* ``.bg-primary-light``
* ``.bg-success-dark``
* ``.bg-success``
* ``.bg-success-light``
* ``.bg-warning-dark``
* ``.bg-warning``
* ``.bg-warning-light``
* ``.bg-gray-darker``
* ``.bg-gray-dark``
* ``.bg-gray``
* ``.bg-gray-light``
* ``.bg-gray-lighter``

Grid
==============================
* ``.row``
* ``.row.row-table``
* ``.col-xs-1`` ... ``.col-xs-12`` works only when the parent has ``.row.row-table`` class

Panel
==============================

* ``.panel``
* ``.panel.panel-heading``
* ``.panel.panel-body``
* ``.panel.panel-footer``

Form
==============================

* ``.form-control``

Button
==============================

* ``.btn.btn-default``
* ``.btn.btn-link``
* ``.btn.btn-primary``
* ``.btn.btn-success``
* ``.btn.btn-info``
* ``.btn.btn-warning``
* ``.btn.btn-danger``

Icons
==============================

All icons from FontAwesome: ``fa fa-<icon-name></icon-name>``

All icons from SimpleLineIcons: ``icon-<icon-name>``
   
      
