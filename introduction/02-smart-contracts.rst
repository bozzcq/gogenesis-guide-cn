################################################################################
2 智能合约
################################################################################
.. contents::
  :local:
  :depth: 2

智能合同（以下简称“合同”）是应用程序的基本元素，它执行单个操作（通常是在数据库表中创建记录），由用户或另一个合同从用户界面启动。在应用程序中使用数据的所有操作都是作为契约系统形成的，通过数据库表或合同主体中的调用函数相互交互。

合约是使用原始（由平台开发团队开发的）图灵完全脚本语言Simvolio编写的，并编译为字节码。该语言包括一组函数，操作符和构造，可用于实现数据处理算法和数据库操作。

合同可以编辑，但只有在合同编辑权利中不允许编辑的情况下才允许编辑。区块链中数据的操作由合同的最新（当前）版本执行。对合同所做更改的完整历史记录存储在区块链中，并可从软件客户端获取。

********************************************************************************
合同的结构
********************************************************************************
合同是用合同关键字声明的，后面跟着新的合同名称。 合同的机构应该用大括号括起来。 每份合约由三部分组成：

1. **data** - 声明输入数据（变量的名称和类型），
2. **conditions** - 验证输入数据的正确性，
3. **action** - 包括合同执行的行为。

合同结构：

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
**数据** 部分描述了合同输入数据以及接收数据的表单参数。
数据逐行列出：首先，指定了变量名称（只有变量，但不传递数组），然后可以通过双引号中的间隙指定接口表单的类型和参数：

* *hidden* - 表单的隐藏元素，
* *optional* - 没有强制填写的表单元素，
* *date* - 日期和时间选择的字段，
* *polymap* - 具有坐标和区域选择的地图，
* *map* - 具有标记地点的能力的地图，
* *image* - 图片上传，
* *text* - 在textarea字段中输入HTML代码的文本，
* *crypt：Field* - 为 *Field* 字段中指定的目的地创建并加密私钥。如果只指定了“crypt”，那么将为签署合同的用户创建私钥，
* *address* - 用于输入帐户地址的字段，
* *签名：合同名称* - 显示合同名称合同的行，需要签名（在特殊说明部分详细讨论）。

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
验证获得的数据在本节中进行。 以下命令用于警告错误的存在：“错误”，“警告”，“信息”。 事实上，他们都会产生一个错误来停止合约操作，但在界面中显示不同的消息：*严重错误*，*警告*和*信息错误*。 例如，

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
操作部分包含合同的主程序代码，用于检索附加数据并将结果值记录到数据库表中。 例如，

.. code:: js

	action {
		DBUpdate("keys", $key_id,"-amount", $amount)
		DBUpdate("keys", $recipient,"+amount,pub", $amount, $Pub)
	}


合同中的变量
==============================
在数据部分中声明的合同输入数据通过带有``````符号的变量，后跟数据名称传递给其他部分。 ``````符号可以用来声明额外的变量; 这些变量将被视为全球合约和所有嵌套合同。

合同可以访问预定义的变量，这些变量包含关于调用该合同的事务的数据。

* ``$ time`` - 交易时间，int，
* ``ecos_id`` - 生态系统ID，int，
* ``$ block`` - 包含此事务的块号，int，
* ``$ key_id`` - 签署交易的账户的ID; VDE合同的价值将为零，
* ``$ wallet_block`` - 形成包含此事务的块的节点的地址，
* ``block_time`` - 当包含当前合约的交易的块形成时。

预定义的变量不仅可以在合同中访问，也可以在权限字段（定义访问应用程序元素的条件）中访问，它们用于构建逻辑表达式。 当在Permissions域中使用时，与块形成相关的变量（``$ time``，``$ block`` 等）总是等于零。

预定义的变量$ result用于从嵌套合同中返回一个值。

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
嵌套合同
********************************************************************************
可以从封闭合同的条件和操作部分调用嵌套合同。可以直接使用名称后面的括号中指定的参数（NameContract（Params））或使用CallContract函数（使用字符串变量为其传递协定名称）来调用嵌套协定。

********************************************************************************
签字合同
********************************************************************************
由于合同写作的语言允许执行封闭式合同，因此可以在不知道已经运行外部合同的用户的情况下完成这样的随附合同，这可能导致用户对其未经授权的交易进行签名，比如，转让来自其帐户的资金。

假设有一个TokenTransfer契约 *TokenTransfer* ：

.. code:: js

    contract TokenTransfer {
        data {
          Recipient int
          Amount    money
        }
        ...
    }

如果在由用户发起的合同中签署了字符串 ``TokenTransfer（“收件人，金额”，12345,100）``，则100个硬币将被转移到账户12345.在这种情况下，签署外部合同的用户 将不会了解交易。 如果TokenTransfer合同在其调用合同时需要额外的用户签名，则可能会排除此情况。 去做这个：

1.在 *TokenTransfer* 合约的 *data* 部分添加一个名为 **Signature** 的字段，其中带有 ``optional`` 和 ``hidden`` 参数，这样就不需要额外的签名 直接呼叫合同，因为到目前为止 **签名** 字段中将有签名。

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
  
在当前的例子中，它将足够指定两个字段**收件人**和**金额**：

* **标题**：您是否同意将此收款人汇款？
* **参数**：收件人文本：账户ID
* **参数**：金额文本：金额（qEGS）

现在，如果插入``TokenTransfer（“Recipient，Amount”，12345，100）``调用合同，系统错误```Signature'未定义'``将被显示。如果按照以下方式调用合同：TokenTransfer（“收件人，金额，签名”，12345,100，“xxx ... xxxxx”），系统错误将在签名验证时发生。在签订合同后，验证以下信息：*初始交易的时间，用户ID，签名表*中指定的字段的值，并且不可能伪造签名。

为了使用户在调用*TokenTransfer*合同时看到汇款确认，需要添加一个任意名称和字符串类型的字段，并使用可选参数``signature：contractname ``。在调用随附的*TokenTransfer*合同后，您只需转发此参数。还应该记住，必须在外部合同的“数据”部分中描述担保合同的参数（它们可能是隐藏的，但仍会在确认时显示）。例如，

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

当发送 *MyTest* 合同时，将向用户请求向指定账户转账的额外确认。如果在随附的合同中列出了其他值，如TokenTransfer（“收件人，金额，签名”，$收款人，$金额+ 10，$签名）``，则会出现无效签名错误。

************************************************** ******************************
合约编辑器
************************************************** ******************************
合约可以在Molis软件客户端的特殊编辑器中创建和编辑。每个新合约都有一个典型的结构，默认情况下有三个部分：``data，conditions，action``。合约编辑有助于：

- 编写合同代码（突出显示Simvolio语言的关键词，
- 格式化合约源代码，
- 将合同绑定到一个帐户，从中扣除执行的费用，
- 定义编辑合同的权限（通常，通过指定具有特殊功能ContractConditions中规定的权限的合同名称，或通过直接指示更改条件字段中的访问条件），
- 通过恢复以前版本的选项查看对合同所做更改的历史记录。

************************************************** ******************************
Simvolio合同语言
************************************************** ******************************
平台中的契约使用原始（由平台团队开发）图灵完整脚本语言Simvolio编写，并编译为字节码。该语言包括一组函数，操作符和构造，可用于实现数据处理算法和数据库操作。 Simvolio语言提供：

- 声明不同数据类型的变量，以及简单的和关联的数组：var，array，map，
- 使用“if”条件语句和“while”循环结构，
- 从数据库中检索数据并将数据记录到数据库``DBFind，DBInsert，DBUpdate``，
- 处理合同，
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

所有标识符，包括变量名称，函数，合同等都区分大小写（MyFunc和myFunc是不同的名称）。

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

如果和当语句
------------------------------
合同语言支持标准 if和while循环，可用于函数和合同。 这些语句可以相互嵌套。

关键字后面应该附带一个条件语句。 如果条件语句返回一个数字，那么当它的值=零时，它被认为是 *false*。 例如，* val == 0*相当于 *！val*，* val！= 0*与 *val* 相同。 **if** 语句可以有一个 **else** 块，当 **if** 条件语句为假时执行该块。 以下比较运算符可用于条件语句：``<，>，> =，<=，==，！=``，以及 ``||``（OR）和``&&``（（ AND）。

.. code:: js

    if val > 10 || id != $citizen {
      ...
    } else {
      ...
    }

The **while** statement is intended for implementation of loops. A **while** block will be executed while its condition is true. The **break** operator is used to end a loop inside a block. To start a loop from the beginning, the **continue** operator should be used.

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

Apart from conditional statements, the language supports standard arithmetic operations: ``+,-,*,/``
Variables of **string** and **bytes** types can be used as a condition. In this case, the condition will be true when the length of the string (bytes) is greater than zero, and false for an empty string.

Functions
------------------------------
Functions of the contracts language perform operations with data received in the data section of a contract: reading and writing database values, converting value types, and establishing connections between contracts.

Functions are declared with the **func** keyword, followed by the function name and a list of parameters passed to it (with their types), all enclosed in curly brackets and separated by commas. After the closing curly bracket the data type of the value returned by the function should be stated. The function body should be enclosed in curly brackets. If a function does not have parameters, then the curly brackets are not necessary. To return a value from a function, the ``return`` keyword is used.

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
  
Functions don't return errors, because all error checks are carried out automatically. When an error is generated in any function, the contract stops its operation and displays a window with the error description.

An undefined number of parameters can be passed to a function. To do this, put **...** instead of the type of the last parameter. In this case, the data type of the last parameter will be *array*, and it will contain all, starting from this parameter, variables that were passed with the call. Variables of any type can be passed, but you should take care of possible conflicts related to data type mismatch.

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
  
Let's consider a situation, where a function has many parameters, but we need only some of them when calling it. In this case, optional parameters can be declared in the following way: ``func myfunc(name string).Param1(param string).Param2(param2 int) {...}``. You can specify only the parameters you need with the call in arbitrary order: ``myfunc("name").Param2(100)``. In the function body you can address these variables as usual. If an extended parameter is not specified with the call, it will have the default value, for example, an empty string for a string and zero for a number. It should be noted, that you can specify several extended parameters and use ``...``: ``func DBFind(table string).Where(request string, params ...)`` and call ``DBFind("mytable").Where("id > ? and type = ?", myid, 2)``

.. code:: js
 
    func DBFind(table string).Columns(columns string).Where(format string, tail ...)
             .Limit(limit int).Offset(offset int) string  {
       ...
    }
     
    func names() string {
       ...
       return DBFind("table").Columns("name").Where("id=?", 100).Limit(1)
    }

Predefined values
------------------------------
The following variables are available when executing a contract. 

* ``$key_id`` - a numerical identifier (int64) of the account that signed the transaction,
* ``$ecosystem_id`` - identifier of the ecosystem where the transaction was created, 
* ``$type`` identifier of an external contract from where the current contract was called, 
* ``$time`` - time specified in the transaction in Unix format, 
* ``$block`` - block number in which this transaction is sealed, 
* ``$block_time`` - time specified in the block, 
* ``$block_key_id`` - numeric identifier (int64) of the node that signed the block,
* ``$auth_token`` is the authorization token, which can be used in VDE contracts, for example, when calling contracts though API with the ``HTTPRequest`` function.

.. code:: js

	var pars, heads map
	heads["Authorization"] = "Bearer " + $auth_token
	pars["vde"] = "false"
	ret = HTTPRequest("http://localhost:7079/api/v2/node/mycontract", "POST", heads, pars)

It should be kept in mind that these variables are available not only in the functions of the contract but also in other functions and expressions, for example, in conditions that are specified for contracts, pages and other objects. In this case, *$time*, *$block* variables related to the block and others are equal to 0.

The value that needs to be returned from the contract should be assigned to a predefined variable ``$result``.

Retrieving values from the database
==============================
DBFind(table string) [.Columns(columns string)] [.Where(where string, params ...)] [.WhereId(id int)] [.Order(order string)] [.Limit(limit int)] [.Offset(offset int)] [.Ecosystem(ecosystemid int)] array
------------------------------
The Function receives data from a database table in accordance with the request specified. Returned is an *array* comprised of *map* associative arrays.

* *table* - table name,
* *сolumns* - list of returned columns. If not specified, all columns will be returned, 
* *Where* - search condition. For instance, ``.Where("name = 'John'")`` or ``.Where("name = ?", "John")``,
* *id* - search by identifier. For example, *.WhereId(1)*,
* *order* - a field, which will be used for sorting. By default, values are sorted by *id*,
* *limit* - number of returned values (default = 25, maximum = 250),
* *offset* - returned values offset,
* *ecosystemid* - ecosystem ID. By default, values are taken from the table in the current ecosystem.

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
The function returns an associative array *map* with data obtained from a database table in accordance with the specified query.

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
This function returns a language resource with name label for language lang, specified as a two-character code, for instance, *en, fr, ru*; if there is no language resource for a selected language, the result will be returned in English.

* *label* - language resource name,
* *lang* - two-character language code.

.. code:: js

    warning LangRes("confirm", $Lang)
    error LangRes("problems", "de")
                     	
Changing values in tables
==============================
DBInsert(table string, params string, val ...) int
------------------------------
The function adds a record to a specified *table* and returns the **id** of the inserted record.

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
The function updates columns in a record whose column has a specified value. The table should have an index for a specified column.

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
This function returns the first *map* associative array from the *list* array. If the *list* is empty, then the result will be an empty *map*. This function is mostly used with the DBFind function. The *list* parameter should not be specified in this case. 

* *list* - a map array, returned by the **DBFind** function.

.. code:: js

   var ret map
   ret = DBFind("contracts").Columns("id,value").WhereId(10).Row()
   Println(ret)

One(list array, column string) string
------------------------------
The function returns the value of the *column* key from the first associative array in the *list* array. If the *list* list is empty, then nil is returned. This function is mostly used with the DBFind function. The *list* parameter should not be specified in this case. 

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
The function calls a contract by its name. All the parameters specified in the section *data* of the contract should be listed in the transmitted array. The function returns the value that was assigned to **$result**  variable in the contract.

* *name*  - name of the contract being called,
* *params* - an associative array with input data for the contract.

.. code:: js

    var par map
    par["Name"] = "My Name"
    CallContract("MyContract", par)

ContractAccess(name string, [name string]) bool
------------------------------
The function checks whether the name of the executed contract matches with one of the names listed in the parameters. Typically used to control access of contracts to tables. The function is specified in the *Permissions* fields when editing table columns or in the *Insert* and *New Column* fields in the *Table permission* section.

* *name* – contract name.

.. code:: js

    ContractAccess("MyContract")  
    ContractAccess("MyContract","SimpleContract") 
    
ContractConditions(name string, [name string]) bool
------------------------------
The function calls the **conditions** section from contracts with specified names. For such contracts, the *data* block must be empty. If the conditions *conditions* is executed without errors, then *true* is returned. If an error is generated during execution, the parent contract will also end with this error. This function is usually used to control access of contracts to tables and can be called in the *Permissions* fields when editing system table.

* *name* – contract name.

.. code:: js

    ContractConditions("MainCondition")  

EvalCondition(tablename string, name string, condfield string) 
------------------------------
Function takes from the *tablename* table the value of the *condfield* field from the record with the *’name’* field, which is equal to the *name* parameter and checks if the condition from the field *condfield* is made. 

* *tablename* - name of the table,
* *name* - value for searching by the field 'name',
* *condfield* - the name of the field where the condition to be checked is stored.

.. code:: js

    EvalCondition(`menu`, $Name, `condition`)  

ValidateCondition(condition string, state int) 
------------------------------
The function tries to compile the condition specified in the *condition* parameter. If a mistake occurs during the compilation process, the mistake will be generated and the calling contract will complete is’s job. This function is designed to check the correctness of the conditions when they change.

* *condition* - verifiable condition,
* *state* - identifier of the state. Specifie 0 if checking for global conditions.

.. code:: js

    ValidateCondition(`ContractAccess("@1MyContract")`, 1)  
    

Operations with account addresses
==============================
AddressToId(address string) int
------------------------------
Function returns the the identification number of the citizen by the string value of the address of his account. If the wrong adress is specified, then 0 returns. 

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
Functions do not allow direct possibilities to select, update, etc.. but they allow you to use the capabilities and functions of PostgreSQL when you get values and a description of the where conditions  in the samples. This includes, among other things, the functions for working with dates and time. For example, you need to compare the column *date_column* and the current time. If  *date_column* has the  type timestamp, then the expression will be the following ``date_column> now ()``. And if *date_column* stores time in Unix format as a number, then the expression will be ``to_timestamp (date_column)> now ()``.

.. code:: js

    to_timestamp(date_column) > now()
    date_initial < now() - 30 * interval '1 day'
    
Consider the situation when we have a value in Unix format and we need to write it in a field of type *timestamp *. In this case, when listing fields, before the name of this column you need to specify **timestamp**.

.. code:: js

   DBInsert("mytable", "name,timestamp mytime", "John Dow", 146724678424 )

If you have a string value of time and you need to write it in a field with the type *timestamp*, in this case, **timestamp** must be specified before the value itself.

.. code:: js

   DBInsert("mytable", "name,mytime", "John Dow", "timestamp 2017-05-20 00:00:00" )
   var date string
   date = "2017-05-20 00:00:00"
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + date )
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + $txtime )


Functions for VDE
==============================
The following functions can be used only in Virtual Dedicated Ecosystems (VDE) contracts.

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
System contracts are created by default during product installation. All of these contracts are created in the first ecosystem, that's why you need to specify their full name to call them from other ecosystems, for instance, ``@1NewContract``.

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
Binding of a contract to the account in the current ecosystem. Contracts can be tied only from the account, which was specified when the contract was created. After the contract is tied, this account will pay for execution of this contract.

Parameters
      
* *Id int* - ID of the contract to activate.

DeactivateContract
------------------------------
Unbinds a contract from an account in the current ecosystem. Only the account which the contract is currently bound to can unbind it. After the contract is unbound, its execution will be paid by a user that executes it.
 
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
