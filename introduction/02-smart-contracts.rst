################################################################################
2 智能合约
################################################################################
.. contents::
  :local:
  :depth: 2

智能合约（以下简称“合约”）是应用程序的基本元素，它执行单个操作（通常是在数据库表中创建记录），由用户或另一个合约从用户界面启动。在应用程序中使用数据的所有操作都是作为合约系统形成的，通过数据库表或合约主体中的调用函数相互交互。

合约是使用原始（由平台开发团队开发的）图灵完全脚本语言Simvolio编写的，并编译为字节码。该语言包括一组函数，操作符和构造，可用于实现数据处理算法和数据库操作。

合约可以编辑，但只有在合约编辑权利中不允许编辑的情况下才允许编辑。区块链中数据的操作由合约的最新（当前）版本执行。对合约所做更改的完整历史记录存储在区块链中，并可从软件客户端获取。

********************************************************************************
合约的结构
********************************************************************************
合约是用合约关键字声明的，后面跟着新的合约名称。 合约的主体应该用大括号括起来。 每份合约由三部分组成：

1. **data** - 声明输入数据（变量的名称和类型），
2. **conditions** - 验证输入数据的正确性，
3. **action** - 包括合约执行的行为。

合约结构：

.. code:: js

  contract MyContract {
      data {
          FromId address
          ToId   address
          Amount money
      }
      func conditions {
          ...
      }
      func action {
      }
  }
  

数据部分
==============================
**数据** 部分描述了合约输入数据以及接收数据的表单参数。
数据逐行列出：首先，指定了变量名称（只有变量，但不传递数组），然后可以通过双引号中的间隙指定接口表单的类型和参数：

* *hidden* - 表单的隐藏元素，
* *optional* - 没有强制填写的表单元素，
* *date* - 日期和时间选择的字段，
* *polymap* - 具有坐标和区域选择的地图，
* *map* - 具有标记地点的能力的地图，
* *image* - 图片上传，
* *text* - 在textarea字段中输入HTML代码的文本，
* *crypt：Field* - 为 *Field* 字段中指定的目的地创建并加密私钥。如果只指定了“crypt”，那么将为签署合约的用户创建私钥，
* *address* - 用于输入帐户地址的字段，
* *signature:contractname* - 显示合约名称合约的行，需要签名（在特殊说明部分详细讨论）。

.. code:: js

  contract my {
    data {
        Name string 
        RequestId address
        Photo bytes "image optional"
        Amount money
        Private bytes "crypt:RequestId"
    }
    ...
  }
    
条件部分
==============================
验证获得的数据在本节中进行。 以下命令用于警告错误的存在：“错误”，“警告”，“信息”。 事实上，他们都会产生一个错误来停止合约操作，但在界面中显示不同的消息：*严重错误*，*警告* 和*信息错误*。 例如，

.. code:: js

  if fuel == 0 {
        error "fuel cannot be zero!"
  }
  if money < limit {
        warning Sprintf("You don't have enough money: %v < %v", money, limit)
  }
  if idexist > 0 {
        info "You have been already registered"
  }
  
行动部分
==============================
操作部分包含合约的主程序代码，用于检索附加数据并将结果值记录到数据库表中。 例如，

.. code:: js

	action {
		DBUpdate("keys", $key_id,"-amount", $amount)
		DBUpdate("keys", $recipient,"+amount,pub", $amount, $Pub)
	}


合约中的变量
==============================
在数据部分中声明的合约输入数据通过具有`$`符号的变量后跟数据名称传递给其他部分。 `$`符号可以用来声明额外的变量; 这些变量将被视为全球合约和所有嵌套合约。

在 conditions 和 action 中使用data部分声明的变量，需要用$+变量名来调用，还可以用$申明额外的全局变量。

合约可以访问预定义的变量，这些变量包含关于调用该合约的交易的数据。

* ``$time`` - 交易时间，int，
* ``$ecosystem_id`` - 生态系统ID，int，
* ``$block`` - 包含此交易的块号，int，
* ``$key_id`` - 签署交易的账户的ID; VDE合约的价值将为零，
* ``$wallet_block`` - 形成包含此交易的块的节点的地址，
* ``$block_time`` - 当包含当前合约的交易的块形成时。

预定义的变量不仅可以在合约中访问，也可以在权限字段（定义访问应用程序元素的条件）中访问，它们用于构建逻辑表达式。 当在Permissions域中使用时，与块形成相关的变量（``$time``，``$block`` 等）总是等于零。

预定义的变量$result用于从嵌套合约中返回一个值。

.. code:: js

  contract my {
    data {
        Name string 
        Amount money
    }
    func conditions {
        if $Amount <= 0 {
           error "Amount cannot be 0"
        }
        $ownerId = 1232
    }
    func action {
        DBUpdate("mytable", $ownerId, "name,amount", $Name, $Amount - 10 )
        DBUpdate("mytable2", $citizen, "amount", 10 )
    }
  }
  
********************************************************************************
嵌套合约
********************************************************************************
可以从封闭合约的条件和操作部分调用嵌套合约。可以直接使用名称后面的括号中指定的参数（NameContract（Params））或使用CallContract函数（使用字符串变量为其传递协定名称）来调用嵌套协定。

********************************************************************************
签名合约
********************************************************************************
由于合约写作的语言允许执行封闭式合约，因此可以在不知道已经运行外部合约的用户的情况下完成这样的随附合约，这可能导致用户对其未经授权的交易进行签名，比如，转让来自其帐户的资金。

假设有一个TokenTransfer合约 *TokenTransfer* ：

.. code:: js

    contract TokenTransfer {
        data {
          Recipient int
          Amount    money
        }
        ...
    }

如果在由用户发起的合约中签署了字符串 ``TokenTransfer("Recipient,Amount", 12345, 100)``，则100个硬币将被转移到账户12345.在这种情况下，签署外部合约的用户 将不会了解交易。 如果TokenTransfer合约在其调用合约时需要额外的用户签名，则可能会排除此情况。 去做这个：

1.在 *TokenTransfer* 合约的 *data* 部分添加一个名为 **Signature** 的字段，其中带有 ``optional`` 和 ``hidden`` 参数，这样就不需要额外的签名 直接调用合约，因为到目前为止 **Signature** 字段中将有签名。

.. code:: js

    contract TokenTransfer {
        data {
          Recipient int
          Amount    money
          Signature string "optional hidden"
        }
        ...
    }

2.在 *Signatures* 表格（在页面上*平台客户端的签名*）中添加包含以下内容的条目：

- *TokenTransfer* 合约名称，
- 其值将显示给用户的字段名称及其文本说明，
- 确认后要显示的文字。
  
在当前的例子中，它将足够指定两个字段**Receipient**和**Amount**：

* **Title**：您是否同意将此收款人汇款？
* **Parameter**：Receipient 文本：Account ID
* **Parameter**：Amount 文本：Amount（qEGS）

现在，如果插入``TokenTransfer（“Recipient，Amount”，12345，100）``调用合约，系统错误```Signature'未定义'``将被显示。如果按照以下方式调用合约：TokenTransfer（“收件人，金额，签名”，12345,100，“xxx ... xxxxx”），系统错误将在签名验证时发生。在签订合约后，验证以下信息：*初始交易的时间，用户ID，签名表*中指定的字段的值，并且不可能伪造签名。

为了使用户在调用*TokenTransfer*合约时看到汇款确认，需要添加一个任意名称和字符串类型的字段，并使用可选参数``signature：contractname ``。在调用随附的*TokenTransfer*合约后，您只需转发此参数。还应该记住，必须在外部合约的“数据”部分中描述担保合约的参数（它们可能是隐藏的，但仍会在确认时显示）。例如，

.. code:: js

    contract MyTest {
      data {
          Recipient int "hidden"
          Amount  money
          Signature string "signature:TokenTransfer"
      }
      func action {
          TokenTransfer("Recipient,Amount,Signature",$Recipient,$Amount,$Signature)
      }
    }

当发送 *MyTest* 合约时，将向用户请求向指定账户转账的额外确认。如果在随附的合约中列出了其他值，如TokenTransfer（“收件人，金额，签名”，$收款人，$金额+ 10，$签名）``，则会出现无效签名错误。

************************************************** ******************************
合约编辑器
************************************************** ******************************
合约可以在Molis软件客户端的特殊编辑器中创建和编辑。每个新合约都有一个典型的结构，默认情况下有三个部分：``data，conditions，action``。合约编辑有助于：

- 编写合约代码（突出显示Simvolio语言的关键词，
- 格式化合约源代码，
- 将合约绑定到一个帐户，从中扣除执行的费用，
- 定义编辑合约的权限（通常，通过指定具有特殊函数ContractConditions中规定的权限的合约名称，或通过直接指示更改条件字段中的访问条件），
- 通过恢复以前版本的选项查看对合约所做更改的历史记录。

************************************************** ******************************
Simvolio合约语言
************************************************** ******************************
平台中的合约使用原始（由平台团队开发）图灵完整脚本语言Simvolio编写，并编译为字节码。该语言包括一组函数，操作符和结构体，可用于实现数据处理算法和数据库操作。 Simvolio语言提供：

- 声明不同数据类型的变量，以及简单的和关联的数组：var，array，map，
- 使用“if”条件语句和“while”循环结构，
- 从数据库中检索数据并将数据记录到数据库``DBFind，DBInsert，DBUpdate``，
- 处理合约，
- 变量的转换，
- 使用字符串的操作。

该语言的基本元素和结构
==============================
数据类型和变量
------------------------------
应该为每个变量定义数据类型。在明显的情况下，数据类型会自动转换。可以使用以下数据类型：

* ``bool`` - 布尔值可以为true或false，
* ``bytes`` - 一个字节序列，
* ``int`` - 一个64位整数，
* ``address`` - 一个64位无符号整数，
* ``array`` - 任意类型的值的数组，
* ``map`` - 任意数据类型与字符串键值的关联数组，
* ``money`` - 大整数类型的整数;值存储在数据库中，不带小数点，当根据货币配置设置在用户界面中显示值时添加小数点，
* ``float`` - 带浮点的64位数字，
* ``string`` - 一个字符串;应该用双引号或后引号定义：“这是一个字符串”或“这是一个字符串”。

所有标识符，包括变量名称，函数，合约等都区分大小写（MyFunc和myFunc是不同的名称）。

变量用 **var** 关键字声明，接着是变量名称和类型。在大括号内声明的变量应该在同一对大括号内使用。声明时，变量具有默认值：对于 *bool* 类型，它是 *false*，对于所有数字类型 - 零值，对于字符串 - 空字符串。变量声明的例子：

.. code:: js

  func myfunc( val int) int {
      var mystr1 mystr2 string, mypar int
      var checked bool
      ...
      if checked {
           var temp int
           ...
      }
  }


数组 
------------------------------ 
该语言支持两种数组类型： 

* ``array`` - 一个数字索引从零开始的简单数组， 
* ``map`` - 一个带有字符串键的关联数组。在分配和检索数组元素时，索引应放在方括号中。

.. code:: js

    var myarr array
    var mymap map
    var s string
    
    myarr[0] = 100
    myarr[1] = "This is a line"
    mymap["value"] = 777
    mymap["param"] = "Parameter"

    s = Sprintf("%v, %v, %v", myarr[0] + mymap["value"], myarr[1], mymap["param"])
    // s = 877, This is a line, Parameter 

If 和 While 语句
------------------------------
合约语言支持标准 if和while循环，可用于函数和合约。 这些语句可以相互嵌套。

关键字后面应该附带一个条件语句。 如果条件语句返回一个数字，那么当它的值=零时，它被认为是 *false*。 例如，* val == 0*相当于 *！val*，* val！= 0*与 *val* 相同。 **if** 语句可以有一个 **else** 块，当 **if** 条件语句为假时执行该块。 以下比较运算符可用于条件语句：``<，>，> =，<=，==，！=``，以及 ``||``（OR）和``&&``（（ AND）。

.. code:: js

    if val > 10 || id != $citizen {
      ...
    } else {
      ...
    }

*while*语句旨在实现循环。 A *while*块将在条件为真时执行。 *break*运算符用于结束块内的循环。 要从头开始循环，应该使用*continue*运算符。

.. code:: js

  while true {
      if i > 100 {
         break
      }
      ...
      if i == 50 {
         continue
      }
      ...
  }

除条件语句外，该语言还支持标准算术运算：``+， - ，*，/``
* string *和* bytes *类型的变量可以用作条件。 在这种情况下，当字符串（字节）的长度大于零时，条件将为真，对于空字符串，则为假。

函数
------------------------------
合约语言的函数用合约的数据部分收到的数据执行操作：读取和写入数据库值，转换价值类型以及建立合约之间的连接。

函数是用* func *关键字声明的，接着是函数名和传递给它的参数列表（以及它们的类型），全部用大括号括起来，并用逗号分隔。 在结束大括号之后，应该说明函数返回值的数据类型。 函数体应该放在大括号内。 如果函数没有参数，则大括号不是必需的。 要从函数返回值，使用`return`关键字。

.. code:: js

  func myfunc(left int, right int) int {
      return left*right + left - right
  }
  func test int {
      return myfunc(10, 30) + myfunc(20, 50)
  }
  func ooops {
      error "Ooops..."
  }
  
函数不会返回错误，因为所有错误检查都是自动执行的。 在任何函数中产生错误时，合同将停止其操作并显示一个包含错误描述的窗口。

未定义数量的参数可以传递给一个函数。 要做到这一点，把* ... *，而不是最后一个参数的类型。 在这种情况下，最后一个参数的数据类型将是* array *，并且它将包含从此参数开始的所有随该调用传递的变量的所有数据类型。 任何类型的变量都可以通过，但是您应该注意与数据类型不匹配有关的可能冲突。

.. code:: js

  func sum(out string, values ...) {
      var i, res int
      
      while i < Len(values) {
         res = res + values[i]
         i = i + 1
      }
      Println(out, res)
  }

  func main() {
     sum("Sum:", 10, 20, 30, 40)
  }
  
让我们考虑一个情况，一个函数有很多参数，但是在调用它的时候我们只需要其中的一部分。在这种情况下，可以通过以下方式声明可选参数：`func myfunc（name string）.Param1（param string）.Param2（param2 int）{...}`。您可以按任意顺序仅指定调用所需的参数：`myfunc（“name”）.Param2（100）`。在函数体中，您可以像平常一样处理这些变量。如果未在调用中指定扩展参数，则它将具有默认值，例如，字符串为空字符串，数字为零。需要注意的是，你可以指定几个扩展参数并使用`...`：`func DBFind（table string）.Where（request string，params ...）`并调用`DBFind（“mytable”）。 （“id>？和type =？”，myid，2）`

.. code:: js
 
    func DBFind(table string).Columns(columns string).Where(format string, tail ...)
             .Limit(limit int).Offset(offset int) string  {
       ...
    }
     
    func names() string {
       ...
       return DBFind("table").Columns("name").Where("id=?", 100).Limit(1)
    }

预定义的值（默认值，零值）
------------------------------
执行合同时可以使用以下变量。

*`$key_id` - 签名事务的帐户的数字标识符（int64）
*`$ecosystem_id` - 创建交易的生态系统的标识符，
*当前合同被调用的外部合同的`$ type`标识符，
*`$time` - 在Unix格式的事务中指定的时间，
*`$block` - 这个交易被封闭的块号，
*`$block_time` - 块中指定的时间，
*`$block_key_id` - 签署该块的节点的数字标识符（int64）
*`$auth_token`是授权令牌，可以在VDE合约中使用，例如，通过具有`HTTPRequest`功能的API调用合约时。

.. code:: js

	var pars, heads map
	heads["Authorization"] = "Bearer " + $auth_token
	pars["vde"] = "false"
	ret = HTTPRequest("http://localhost:7079/api/v2/node/mycontract", "POST", heads, pars)

应该记住，这些变量不仅在合同的功能中可用，而且在其他功能和表达中也是可用的，例如，在为合同，页面和其他对象指定的条件下。在这种情况下，与块等有关的* $ time *，* $ block *变量等于0。

需要从合同返回的值应该分配给预定义的变量`$ result`

从数据库中检索值
==============================
DBFind(table string) [.Columns(columns string)] [.Where(where string, params ...)] [.WhereId(id int)] [.Order(order string)] [.Limit(limit int)] [.Offset(offset int)] [.Ecosystem(ecosystemid int)] array
------------------------------
函数根据指定的请求从数据库表中接收数据。返回的是由* map *关联数组组成的*数组*。

* *table* - table name,
* *сolumns* - 返回列的列表。如果未指定，则将返回所有列 
* *Where* - 搜索条件。例如 ``.Where("name = 'John'")`` or ``.Where("name = ?", "John")``,
* *id* - 通过标识符搜索。例如， *.WhereId(1)*,
* *order* - 一个字段，将用于分类。默认情况下，值按* id *排序
* *limit* - 返回值的数量(default = 25, maximum = 250),
* *offset* - 返回值偏移量,
* *ecosystemid* - 生态系统ID。默认情况下，值取自当前生态系统中的表格。

.. code:: js

   var i int
   ret = DBFind("contracts").Columns("id,value").Where("id> ? and id < ?", 3, 8).Order("id")
   while i < Len(ret) {
       var vals map
       vals = ret[0]
       Println(vals["value"])
       i = i + 1
   }
   
   var ret string
   ret = DBFind("contracts").Columns("id,value").WhereId(10).One("value")
   if ret != nil { 
   	Println(ret) 
   }

DBRow(table string) [.Columns(columns string)] [.Where(where string, params ...)] [.WhereId(id int)] [.Order(order string)] [.Ecosystem(ecosystemid int)] map
------------------------------
该函数根据指定的查询返回一个关联数组* map *和从数据库表中获取的数据。

 * *table* - table name,
 * *columns* - a list of columns to be returned. If not specified, all columns will be returned, 
 * *Where* - search parameters; for example, ``.Where("name = 'John'")`` or ``.Where("name = ?", "John")``,
 * *id* - identifier of the string to be returned.  For instance, ``.WhereId(1)``,
 * *order* - a field to use for sorting; by default, information is sorted by *id* field,
 * *ecosystemid* - ecosystem identifier; by default it is the current ecosystem id.
 	
.. code:: js

   var ret map
   ret = DBRow("contracts").Columns("id,value").Where("id = ?", 1)
   Println(map)
    
EcosysParam(name string) string
------------------------------
The function returns the value of a specified parameter from the ecosystem settings (*parameters* table).

* *name* - name of the received parameter,
* *num* - sequence number of the parameter.

.. code:: js

    Println( EcosysParam("gov_account"))

LangRes(label string, lang string) string
------------------------------
这个函数返回一个语言资源，其语言lang的名称标签，指定为两个字符的代码，例如* en，fr，ru *; 如果所选语言没有语言资源，则结果将以英文返回。

* *label* - language resource name,
* *lang* - two-character language code.

.. code:: js

    warning LangRes("confirm", $Lang)
    error LangRes("problems", "de")
                     	
Changing values in tables
==============================
DBInsert(table string, params string, val ...) int
------------------------------
该函数将一条记录添加到指定的* table *并返回插入记录的* id *。

* *tblname*  – name of the table in the database,
* *params* - list of comma-separated names of columns, where the values listed in **val** will be written,
* *val* - list of comma-separated values for the columns listed in **params**; values can be a string or a number.

.. code:: js

    DBInsert("mytable", "name,amount", "John Dow", 100)

DBUpdate(tblname string, id int, params string, val...)
------------------------------
The function changes the column values in the table in the record with a specified **id**.

* *tblname*  – name of the table in the database,
* *id* - identifier **id** of the changeable record,
* *params* - list of comma-separated names of the columns to be changed,
* *val* - list of values for a specified columns listed in **params**; can either be a string or a number.

.. code:: js

    DBUpdate("mytable", myid, "name,amount", "John Dow", 100)

DBUpdateExt(tblname string, column string, value (int|string), params string, val ...)
------------------------------
该函数更新其列具有指定值的记录中的列。 该表应该有一个指定列的索引。

* *tblname*  – name of the table in the database,
* *column*  - name of the column by which the record will be searched for,
* *value* - value for searching a record in a column,
* *params* - list of comma-separated names of columns, where the values specified in **val** will be written,
* *val* - list of values for recording in the columns listed in **params**; can either be a string or a number.

.. code:: js

    DBUpdateExt("mytable", "address", addr, "name,amount", "John Dow", 100)
    
Array operations
==============================
Join(in array, sep string) string
------------------------------
This function merges the elements of the *in* array into a string with the specified *sep* separator.

* *in* - is the name of the *array* type array, the elements of which you want to merge,
* *sep* - is a separator string.

.. code:: js

    var val string, myarr array
    myarr[0] = "first"
    myarr[1] = 10
    val = Join(myarr, ",")

Split(in string, sep string) array
------------------------------
This function splits the *in* string into elements using *sep* as a separator, and puts them into an array.

* *in* is the initial string,
* *sep* is the separator string.

.. code:: js

    var myarr array
    myarr = Split("first,second,third", ",")

Len(val array) int
------------------------------
This function returns the number of elements in the specified array.

* *val* - an array of the *array* type.

.. code:: js

    if Len(mylist) == 0 {
      ...
    }

Row(list array) map
------------------------------
该函数返回* list *数组中的第一个* map *关联数组。 如果* list *为空，那么结果将是一个空的* map *。 该功能主要用于DBFind功能。 在这种情况下，不应指定* list *参数。

* *list* - a map array, returned by the **DBFind** function.

.. code:: js

   var ret map
   ret = DBFind("contracts").Columns("id,value").WhereId(10).Row()
   Println(ret)

One(list array, column string) string
------------------------------
该函数从* list *数组中的第一个关联数组中返回* column *键的值。 如果* list *列表为空，则返回nil。 该功能主要用于DBFind功能。 在这种情况下，不应指定* list *参数。

* *list* - a map array, returned by the **DBFind** function,
* *column* - name of the returned key.

.. code:: js

   var ret string
   ret = DBFind("contracts").Columns("id,value").WhereId(10).One("value")
   if ret != nil {
      Println(ret)
   }

Operations with contracts and conditions
==============================
CallContract(name string, params map)
------------------------------
该函数按名称调用合同。 所有在合约部分* data *中指定的参数都应列在传输数组中。 该函数返回分配给合约中的* $ result *变量的值。

* *name*  - name of the contract being called,
* *params* - an associative array with input data for the contract.

.. code:: js

    var par map
    par["Name"] = "My Name"
    CallContract("MyContract", par)

ContractAccess(name string, [name string]) bool
------------------------------
该函数检查执行合同的名称是否与参数中列出的名称之一匹配。 通常用于控制对表的合同访问。 在* Table权限*部分中编辑表格列或* Insert *和* New Column *字段时，该功能在* Permissions *字段中指定。

* *name* – contract name.

.. code:: js

    ContractAccess("MyContract")  
    ContractAccess("MyContract","SimpleContract") 
    
ContractConditions(name string, [name string]) bool
------------------------------
该函数从具有指定名称的合同中调用*条件*部分。 对于这样的合约，* data *块必须是空的。 如果条件*条件*无误地执行，则返回*真*。 如果在执行过程中产生错误，则父合同也将以此错误结束。 此功能通常用于控制合同对表的访问，并且可以在编辑系统表时在*权限*字段中调用。

* *name* – contract name.

.. code:: js

    ContractConditions("MainCondition")  

EvalCondition(tablename string, name string, condfield string) 
------------------------------
函数从* tablename *表中获取*'name'*字段的* condfield *字段的值，该字段等于* name *参数，并检查字段* condfield *的条件是否成立。

* *tablename* - name of the table,
* *name* - value for searching by the field 'name',
* *condfield* - the name of the field where the condition to be checked is stored.

.. code:: js

    EvalCondition(`menu`, $Name, `condition`)  

ValidateCondition(condition string, state int) 
------------------------------
该函数试图编译* condition *参数中指定的条件。 如果在编译过程中发生错误，将会产生错误，并且通话合同将完成。 此功能旨在检查条件更改时的正确性。

* *condition* - verifiable condition,
* *state* - identifier of the state. Specifie 0 if checking for global conditions.

.. code:: js

    ValidateCondition(`ContractAccess("@1MyContract")`, 1)  
    

Operations with account addresses
==============================
AddressToId(address string) int
------------------------------
函数通过其账户地址的字符串值返回公民的身份证号码。 如果指定了错误的地址，则返回0。

* *address* - the account adress in the format XXXX-...-XXXX or in the form of number.

.. code:: js

    wallet = AddressToId($Recipient)
    
IdToAddress(id int) string
------------------------------
Returns the address of a account based on its ID number. If a wrong ID is specified, returned is 'invalid'.

* *id* - ID, numerical.

.. code:: js

    $address = IdToAddress($id)
    

PubToID(hexkey string) int
------------------------------
The function returns the account address by the public key in hexadecimal encoding.

* *hexkey* - public key in hexadecimal form.

.. code:: js

    var wallet int
    wallet = PubToID("fa5e78.....34abd6")


Operations with values of variables
==============================
Float(val int|string) float
------------------------------
The function converts an integer *int* or *string* to a floating-point number.

* *val* - an integer or string.

.. code:: js

    val = Float("567.989") + Float(232)

HexToBytes(hexdata string) bytes
------------------------------
The function converts a string with hexadecimal encoding to a *bytes* value (sequence of bytes).

* *hexdata* – a string containing a hexadecimal notation.

.. code:: js

    var val bytes
    val = HexToBytes("34fe4501a4d80094")
       
Random(min int, max int) int
------------------------------
This function returns a random number in the range between min and max (min <= result < max). Both min and max should be positive numbers.

* *min* is the minimum value for the random number,
* *max* - the random number will be smaller than this number.

.. code:: js

    i = Random(10,5000)
   
Int(val string) int
------------------------------
The function converts a string value to an integer.

* *val*  – a string containing a number.

.. code:: js

    mystr = "-37763499007332"
    val = Int(mystr)
    

Sha256(val string) string
------------------------------
The function returns **SHA256** hash of a specified string.

* *val* - incoming line for which the **Sha256** hash should be calculated.

.. code:: js

    var sha string
    sha = Sha256("Test message")

Str(val int|float) string
------------------------------
The function converts a numeric *int* or *float* value to a string.

* *val* - an integer or a floating-point number.

.. code:: js

    myfloat = 5.678
    val = Str(myfloat)

UpdateLang(name string, trans string)
------------------------------
Function updates the language source in the memory. Is used in the transactions that change language sources.

* *name* - name of the language source,
* *trans* - source with translations.

.. code:: js

    UpdateLang($Name, $Trans)

Operations with string values
==============================
HasPrefix(s string, prefix string) bool
------------------------------
Function returns true, if the string bigins from the specified substring *prefix*.

* *s* - checked string,
* *prefix* - checked prefix for this string.

.. code:: js

    if HasPrefix($Name, `my`) {
    ...
    }

Contains(s string, substr string) bool
------------------------------
Returnes true if the string *s* containts the substring *substr*.

* *s* - checked string,
* *substr* - which is searched in the specified line.

.. code:: js

    if Contains($Name, `my`) {
    ...
    }    

Replace(s string, old string, new string) string
------------------------------
Function replaces in the *s* string all cccurrences of the *old* string to *new* string and returnes the result.  

* *s* - source string,
* *old* - changed string,
* *new* - new string.

.. code:: js

    s = Replace($Name, `me`, `you`)
    
Size(val string) int
------------------------------
The function returns the size of the specified string.

* *val* - the string for which we have to calculate the size.

.. code:: js

    var len int
    len = Size($Name) 
 
Sprintf(pattern string, val ...) string
------------------------------
The function forms a string based on specified template and parameters, you can use ``%d`` (number), ``%s`` (string), ``%f`` (float), ``%v`` (for any types).

* *pattern*  - a template for forming a string.

.. code:: js

    out = Sprintf("%s=%d", mypar, 6448)

Substr(s string, offset int, length int) string
------------------------------
Function returns the substring from the specified string starting from the offset *offset* (calculating from the 0) and with length *length*. In case of not correct offsets or length the empty column is returned. If the sum of offset and *length* is more than string size, then the substring will be returned from the offset to the end of the string.

* *val* - string,
* *offset* - offset of substring,
* *length* - size of substring.

.. code:: js

    var s string
    s = Substr($Name, 1, 10)

Operations with system parameters
==============================
SysParamString(name string) string
------------------------------
The function returns the value of the specified system parameter.

* *name* - parameter name.

.. code:: js

    url = SysParamString(`blockchain_url`)

SysParamInt(name string) int
------------------------------
The function returns the value of the specified system parameter in the form of a number.

* *name* - parameter name.

.. code:: js

    maxcol = SysParam(`max_columns`)

DBUpdateSysParam(name, value, conditions string)
------------------------------
The function updates the value and the condition of the system parameter. If you do not need to change the value or condition, then specify an empty string in the corresponding parameter.

* *name* - parameter name,
* *value* - new value of the parameter,
* *conditions* - new condition for changing the parameter.

.. code:: js

    DBUpdateSysParam(`fuel_rate`, `400000000000`, ``)
    

Date/time operations in PostgreSQL queries
==============================
函数不允许直接选择，更新等。但是，它们允许您在获取样本中的值和描述条件时使用PostgreSQL的功能和功能。 其中包括处理日期和时间的功能。 例如，您需要比较* date_column *列和当前时间。 如果* date_column *具有类型时间戳，那么表达式将是以下`date_column> now（）`。 如果* date_column *将时间存储为Unix格式的数字，则该表达式将是`to_timestamp（date_column）> now（）`。

.. code:: js

    to_timestamp(date_column) > now()
    date_initial < now() - 30 * interval '1 day'
    
考虑一下当我们具有Unix格式的值时，我们需要将它写入* timestamp *类型的字段中。 在这种情况下，当列出字段时，在此列的名称之前，您需要指定* timestamp *。

.. code:: js

   DBInsert("mytable", "name,timestamp mytime", "John Dow", 146724678424 )

如果你有一个字符串值的时间，并且你需要把它写在一个类型为* timestamp *的字段中，在这种情况下，* timestamp *必须在它本身的前面指定。

.. code:: js

   DBInsert("mytable", "name,mytime", "John Dow", "timestamp 2017-05-20 00:00:00" )
   var date string
   date = "2017-05-20 00:00:00"
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + date )
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + $txtime )


Functions for VDE
==============================
以下功能只能在虚拟专用生态系统（VDE）合同中使用。

HTTPRequest(url string, method string, heads map, pars map) string
------------------------------
This function sends an HTTP request to a specified address.

* *url* - address, to which the request will be sent,
* *method* - request method – GET or POST,
* *heads* - a data array for header formation,
* *pars* - parameters.

.. code:: js

	var ret string 
	var pars, heads, json map
	heads["Authorization"] = "Bearer " + $auth_token
	pars["vde"] = "true"
	ret = HTTPRequest("http://localhost:7079/api/v2/content/page/default_page", "POST", heads, pars)
	json = JSONToMap(ret)

HTTPPostJSON(url string, heads map, pars string) string
------------------------------
This function is similar to the *HTTPRequest* function, but it sends a *POST* request and parameters are passed in one string.

* *url* - address, to which the request will be sent,
* *heads* - a data array for header formation,
* *pars* - parameters as a json string.

.. code:: js

	var ret string 
	var heads, json map
	heads["Authorization"] = "Bearer " + $auth_token
	ret = HTTPPostJSON("http://localhost:7079/api/v2/content/page/default_page", heads, `{"vde":"true"}`)
	json = JSONToMap(ret)

************************************************
System Contracts
************************************************
系统合同是在产品安装期间默认创建的。 所有这些合同都是在第一个生态系统中创建的，这就是为什么您需要指定其全名以从其他生态系统调用它们，例如`@ 1NewContract`。

List of System Contracts
==============================
NewEcosystem
------------------------------
This contract creates a new ecosystem. To get an identifier of the newly created ecosystem, take the *result* field, which will return in txstatus. Parameters:
   
* *Name string "optional"* - name for the ecosystem. This parameter can be set and/or chanted later.

MoneyTransfer
------------------------------
This contract transfers money from the current account in the current ecosystem to a specified account. Parameters:

* *Recipient string* - recipient's account in any format – a number or ``XXXX-....-XXXX``,
* *Amount    string* - transaction amount in qAPL,
* *Comment   string "optional"* - comments.

NewContract
------------------------------
This contract creates a new contract in the current ecosystem. Parameters:

* *Value string* - text of the contract or contracts,
* *Conditions string* - contract change conditions,
* *Wallet string "optional"* - identifier of user's id where contract should be tied,
* *TokenEcosystem int "optional"* - identifier of the ecosystem, which currency will be used for transactions when the contract is activated.

EditContract
------------------------------
Editing the contract in the current ecosystem.

Parameters
      
* *Id int* - ID of the contract to be edited,
* *Value string* - text of the contract or contracts,
* *Conditions string* - rights for contract change.

ActivateContract
------------------------------
将合同绑定到当前生态系统中的帐户。 合同只能与创建合同时指定的帐户绑定。 合约结算后，该账户将支付执行该合约的费用。

Parameters
      
* *Id int* - ID of the contract to activate.

DeactivateContract
------------------------------
取消当前生态系统中帐户的合同。 只有合同当前绑定的帐户才能解除绑定。 合同解约后，其执行将由执行它的用户支付。
 
 Parameters
 
* *Id int* - identifier of the tied contract.

NewParameter
------------------------------
This contract adds a new parameter to the current ecosystem. 

Parameters

* *Name string* - parameter name,
* *Value string* - parameter value,
* *Conditions string - rights for parameter change.

EditParameter
------------------------------
This contract changes an existing parameter in the current ecosystem.

Parameters

* *Name string* - name of the parameter to be changed,
* *Value string* - new value,
* *Conditions string* - new condition for parameter change.

NewMenu
------------------------------
This contract adds a new menu in the current ecosystem.

Parameters

* *Name string* - menu name,
* *Value string* - menu text,
* *Title string "optional"* - menu header,
* *Conditions string* - rights for menu change.

EditMenu
------------------------------
This contract changes an existing menu in the current ecosystem.

Parameters

* *Id int* - ID of the menu to be changed,
* *Value string* - new text of menu,
* *Title string "optional"* - menu header,
* *Conditions string* - new rights for page change.

AppendMenu
------------------------------
This contract adds text to an existing menu in the current ecosystem.

Parameters

* *Id int* - complemented menu identifier,
* *Value string* - text to be added.

NewPage
------------------------------
This contract adds a new page in the current ecosystem. Parameters:

* *Name string* - page name,
* *Value string* - page text,
* *Menu string* - name of the menu, attached to this page,
* *Conditions string* - rights for change.

EditPage
------------------------------
This contract changes an existing page in the current ecosystem.

Parameters

* *Id int* - ID of the page to be changed,
* *Value string* - new text of the page,
* *Menu string* - name of the new menu on the page,
* *Conditions string* - new rights for page change.

AppendPage
------------------------------
The contract adds text to an existing page in the current ecosystem.

Parameters

* *Id int* - ID of the page to be changed,
* *Value string* - text that needs to be added to the page.

NewBlock
------------------------------
This contract adds a new page block with a template to the current ecosystem. 

Parameters

* *Name string* - block name,
* *Value string* - block text,
* *Conditions string* - rights for block change.

EditBlock
------------------------------
This contract changes an existing block in the current ecosystem.

Parameters

* *Id int* - ID of the block to be changed,
* *Value string* - new text of a block,
* *Conditions string* - new rights for change.

NewTable
------------------------------
This contract adds a new table in the current ecosystem. 

Parameters

* *Name string* - table name in Latin script, 
* *Columns string* - array of columns in JSON format ``[{"name":"...", "type":"...","index": "0", "conditions":"..."},...]``, where

  * *name* - column name in Latin script,
  * *type* - type ``varchar,bytea,number,datetime,money,text,double,character``,
  * *index* - non-indexed field - "0"; create index - "1",
  * *conditions* - condition for changing data in a column; read access rights should be specified in the JSON format. For example, ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``


* *Permissions string* - access conditions in JSON format ``{"insert": "...", "new_column": "...", "update": "..."}``.

  * *insert* - rights to insert records,
  * *new_column* - rights to add columns,
  * *update* - rights to change rights.

EditTable
------------------------------
This contract changes access permissions to tables in the current ecosystem. 

Parameters 

* *Name string* - table name, 
* *Permissions string* - access permissions in JSON format ``{"insert": "...", "new_column": "...", "update": "..."}``.

  * *insert* - condition to insert records,
  * *new_column* - condition to add columns,
  * *update* - condition to change data.   

NewColumn
------------------------------
This contract adds a new column to a table in the current ecosystem. 

Parameters

* *TableName string* - table name in,
* *Name* - column name in Latin script,
* *Type* - type ``varchar,bytea,number,money,datetime,text,double,character``,
* *Index* - non-indexed field - "0"; create index - "1",
* *Permissions* - condition for changing data in a column; read access rights should be specified in the JSON format. For example, ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``

EditColumn
------------------------------
This contract changes the rights to change a table column in the current ecosystem. 

Parameters

* *TableName string* - table name in Latin script, 
* *Name* - column name in Latin script,
* *Permissions* - condition for changing data in a column; read access rights should be specified in the JSON format. For example, ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``.

NewLang
------------------------------
This contract adds language resources in the current ecosystem. Permissions to add resources are set in the *changing_language* parameter in the ecosystem configuration. 

Parameters

* *Name string* - name of the language resource in Latin script, 
* *Trans* - language resources as a string in JSON format with two-character language codes as keys and translated strings as values. For example: ``{"en": "English text", "ru": "Английский текст"}``.

EditLang
------------------------------
This contract updates the language resource in the current ecosystem. Permissions to make changes are set in the *changing_language* parameter in the ecosystem configuration. 

Parameters

* *Name string* - name of the language resource,
* *Trans* - language resources as a string in JSON format with two-character language codes as keys and translated strings as values. For example ``{"en": "English text", "ru": "Английский текст"}``.
 
NewSign
------------------------------
This contract adds the signature confirmation requirement for a contract in the current ecosystem.

Parameters

* *Name string* - name of the contract, where an additional signature confirmation will be required,
* *Value string* - description of parameters in a JSON string, where
    
  * *title* - message text,
  * *params* - array of parameters that are displayed to users, where **name** is the field name, and **text** is the parameter description.
    
* *Conditions string* - condition for changing the parameters.

Example of *Value*

``{"title": "Would you like to sign?", "params":[{"name": "Recipient", "text": "Wallet"},{"name": "Amount", "text": "Amount(EGS)"}]}`` 

EditSign
------------------------------
The contract updates the parameters of a contract with a signature in the current ecosystem. 

Parameters

 * *Id int* - identifier of the signature to be changed,
 * *Value string* - a string containing new parameters,
 * *Conditions string* - new condition for changing the signature parameters.

Import 
------------------------------
This contract imports data from a *. sim file into the ecosystem.

Parameters

* *Data string* - data to be imported in text format; this data is the result of export from an ecosystem to a .sim file.

NewCron
------------------------------
The contract adds a new task in cron to be launched by timer. The contract is available only in VDE systems. Parameters:

* *Cron string* - a string that defines the launch of the contract by timer in the *cron* format,
* *Contract string* - name of the contract to launch in VDE; the contract should not have parameters in its ``data`` section,
* *Limit int* - an optional field, where the number of contract launches can be specified (until contract is executed this number of times),
* *Till string* - an optional string with the time when the task should be ended (this feature is not yet implemented),
* *Conditions string* - rights to modify the task.

EditCron
------------------------------
This contract changes the configuration of a task in cron for launch by timer. The contract is available only in VDE systems. Parameters:

* *Id int* - task ID,
* *Cron string* - a string that defines the launch of the contract by timer in the *cron* format; to disable a task, this parameter should be either an empty string or absent, 
* *Contract string* - name of the contract to launch in VDE; the contract should not have parameters in its data section,
* *Limit int* - an optional field, where the number of contract launches can be specified (until contract is executed this number of times),
* *Till string* - an optional string with the time of task should be ended (this feature is not yet implemented),
* *Conditions string* - new rights to modify the task.
