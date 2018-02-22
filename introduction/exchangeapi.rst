################################################################################
<<<<<<< HEAD
<<<<<<< HEAD
Description of the api requests
################################################################################

The API allows creating private keys (wallets) on request to receive funds and then send them to other wallets. All the private keys created are stored in an encrypted form.

********************************************************************************
Startup parameters
********************************************************************************

In order to work with this API, you need to specify the following parameters when starting Apla.

**-boltDir** - a directory where the file containing private keys will be created and stored. NoSQL database BoltDB is used to store the keys. The file is named *exchangeapi.db*. If the parameter is not specified, the file will be created in the current directory.
=======
=======
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d
api请求的描述
################################################################################

该API允许根据请求创建私钥（钱包）以接收资金，然后将其发送给其他钱包。 所有创建的私钥都以加密的形式存储。

********************************************************************************
启动参数
********************************************************************************

为了使用此API，您需要在启动Apla时指定以下参数。

**-boltDir** - 将创建并存储包含私钥的文件的目录。 NoSQL数据库BoltDB用于存储密钥。 该文件被命名为*exchangeapi.db*。 如果未指定参数，则将在当前目录中创建该文件。
<<<<<<< HEAD
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d
=======
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d

.. code:: 
      
      -boltDir=/home/temp
      
<<<<<<< HEAD
<<<<<<< HEAD
**-boltPsw**  - password for encrypting private keys when writing to the database. The API will not work if the password was not specified at startup. The password specified at first startup should be specified at subsequent startups. Don’t forget the password because it is not saved anywhere. If you specify an incorrect password or you enter the word "console" as a password, then you will be prompted to enter the password in the console after running Apla. Also, if the password has already been set but not specified in the command line parameter, it will be requested for in the console.
=======
**-boltPsw**  - 写入数据库时用于加密私钥的密码。 如果在启动时未指定密码，API将不起作用。 首次启动时指定的密码应在后续启动时指定。 不要忘记密码，因为它没有保存在任何地方。 如果您指定的密码不正确或者输入单词“console”作为密码，则在运行Apla后，系统会提示您在控制台中输入密码。 另外，如果密码已被设置，但未在命令行参数中指定，则将在控制台中请求该密码。
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d
=======
**-boltPsw**  - 写入数据库时用于加密私钥的密码。 如果在启动时未指定密码，API将不起作用。 首次启动时指定的密码应在后续启动时指定。 不要忘记密码，因为它没有保存在任何地方。 如果您指定的密码不正确或者输入单词“console”作为密码，则在运行Apla后，系统会提示您在控制台中输入密码。 另外，如果密码已被设置，但未在命令行参数中指定，则将在控制台中请求该密码。
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d

.. code:: 

      -boltPsw=mypass344
      
<<<<<<< HEAD
<<<<<<< HEAD
**-apiToken**  - this parameter specifies the token that will need to be passed when making a request to the API. The token specified will be saved, and can be omitted in subsequent startups. If this parameter has not been specified, then you will be able to call API commands without specifying the token parameter. If you'll need to change the token, you should start Apla with a new value in this parameter.
=======
**-apiToken**  - 此参数指定在向API发出请求时需要传递的标记。 指定的令牌将被保存，并且可以在随后的启动中省略。 如果这个参数没有被指定，那么你将能够在不指定token参数的情况下调用API命令。 如果您需要更改令牌，则应在此参数中使用新值启动Apla。
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d
=======
**-apiToken**  - 此参数指定在向API发出请求时需要传递的标记。 指定的令牌将被保存，并且可以在随后的启动中省略。 如果这个参数没有被指定，那么你将能够在不指定token参数的情况下调用API命令。 如果您需要更改令牌，则应在此参数中使用新值启动Apla。
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d

.. code:: 

      -apiToken=qweuytwuy347834
      
********************************************************************************
API requests
********************************************************************************

<<<<<<< HEAD
<<<<<<< HEAD
Responses to API requests are in JSON format and all of them have an error field. If this field is empty, then the request has been executed without errors. Otherwise, the field contains the text of the error that has occurred.

/exchangeapi/newkey
==============================
The command generates a private key, records it to a key file, and returns the public key and wallet addresses. For example:
=======
=======
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d
对API请求的响应采用JSON格式，并且它们都有错误字段。 如果该字段为空，则该请求已被执行而没有错误。 否则，该字段将包含发生错误的文本。

/exchangeapi/newkey
==============================
该命令生成一个私钥，将其记录到密钥文件中，并返回公钥和钱包地址。 例如：
<<<<<<< HEAD
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d
=======
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d


*/exchange/newkey?token=qweuytwuy347834*

Response example:

.. code:: 

   {"error":"", 
    "public":"b7880fa40779d673e7....238def72881d6c2b6c60ffcc2ec7f050141d", 
    "address":"0773-5161-7272-4133-0241", 
    "key_id":7735161727241330241
    }

/exchange/send?sender=...&recipient=...&amount=...
==============================
<<<<<<< HEAD
<<<<<<< HEAD
This command sends money from wallet (**sender**) in the DB to the specified wallet (**recipient**). Wallets can be specified in any format - *XXXX-....-XXXX, int64, uint64*. Please note, that the command sends the transaction, but does not wait for the confirmation of receipt. The amount to send (**amount**) should be specified in *qEGS*. The transaction hash is returned in the **txhash** field.
=======
该命令将钱包中的钱包（**sender**）发送到指定的钱包（**recipient**）。 钱包可以用任何格式指定 - *XXXX -....- XXXX，int64，uint64*。 请注意，该命令发送交易，但不等待确认收货。 发送金额（**amount**）应在*qEGS*中指定。 事务哈希在**txhash**字段中返回。
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d
=======
该命令将钱包中的钱包（**sender**）发送到指定的钱包（**recipient**）。 钱包可以用任何格式指定 - *XXXX -....- XXXX，int64，uint64*。 请注意，该命令发送交易，但不等待确认收货。 发送金额（**amount**）应在*qEGS*中指定。 事务哈希在**txhash**字段中返回。
>>>>>>> 1db434dc883eca3f85588095208253ca50d0441d

For example,

*/exchange/send?sender=1693-7869-8202-2463-0602&recipient=-3521799150320731671&amount=999000000000000*

Response example:

.. code:: 

      {"txhash": "734fa..89ab5",
     "error":""}


/exchangeapi/balance?key_id=....
==============================
The command returns the balance of any wallet.

For example,

*/exchangeapi/balance?key_id=0773-5161-7272-4133-0241*

Response example:

.. code:: 

     {"error":"","amount":"99992318000000000000","egs":"99.992318"}

/exchangeapi/history?key_id=...&count=...
==============================
The command returns the last history of flow of funds in the specified wallet. count is an optional parameter and determines the number of records to be returned (1 more can be returned). By default, 50 entries will be returned, and the maximum number is 200.

Response:

* *error* - error message 
* *history* - balance history array 

* *block_id* - block ID
* *dif* - change involved
* *txhash* - transaction hash 
* *amount* - available amount in qEGS
* *egs* - available amount in EGS
* *time* - transaction timestamp 


For example:

*/exchangeapi/history?key_id=1693-7869-8202-2463-0602&count=10&token=mytoken*

Response example:

.. code:: 

    {"error":"",
    "history":[{"block_id":"118855","dif":"-0.001",
    "amount":"99992318000000000000","egs":"99.992318","time":"03.05.2017 10:48:14"},
    {"block_id":"118855","dif":"-0.001999","amount":"99993318000000000000","egs":"99.993318",
    "time":"03.05.2017 10:48:14"},
    {"block_id":"112283","dif":"-0.001","amount":"99995317000000000000","egs":"99.995317",
    "time":"02.05.2017 18:28:24"}]}

/exchangeapi/txstatus?hash=...
==============================

The command returns information on the transaction with hash specified in the *hash* field. If *block_id* is "0" and in the *error* field an empty string, then the transaction has not yet entered the block.

Response

* *block_id* - block ID 
* *txhash* - transaction hash 
* *amount* - transaction amount in qAPL
* *egs* - transaction amount in APL
* *time* - transaction timestamp
* *sender* - sender ID 
* *recipient* - recipient ID
* *sender_address* - sender's address in the XXXX-...-XXXX format
* *recipient_address* - recipient's address in the XXXX-...-XXXX format
* *confirmations* - number of blocks after this block
* *error* - error message 

Example:

*/exchangeapi/txstatus?hash=ca378ca44c388b79fba6d8643c5e8935*

Response example:

.. code:: 

{
    "block_id": "18111",
    "confirmations": "3618",
    "txhash": "ca378ca44c388b79fba6d8643c5e8935",
    "amount": "46000000000000",
    "egs": "0.000046",
    "time": "1505306953",
    "sender": "7480871936035188899",
    "recipient": "-2411392676761618411",
    "sender_address": "0748-0871-9360-3518-8899",
    "recipient_address": "1603-5351-3969-4793-3205",
    "error": ""
   }
