#	osapi
__通用對象存儲應用程式界面__

[![total downloads of osapi](https://img.shields.io/npm/dt/osapi.svg)](https://www.npmjs.com/package/osapi)
[![osapi's License](https://img.shields.io/npm/l/osapi.svg)](https://www.npmjs.com/package/osapi)
[![latest version of osapi](https://img.shields.io/npm/v/osapi.svg)](https://www.npmjs.com/package/osapi)

>	其他语言 / [English](./README.md) / [简体中文](./README.zh_CN.md)

這套 API 很好地相容了 CEPH 對象存儲服務，因此取 __[ceph](https://www.npmjs.com/package/ceph)__ 作為本包的別名。你可以按照個人喜好，引用 `osapi` 或 `ceph` 包，兩個 NPM 包將保持同步更新。

我們提供兩種不同風格的 API：OpenStack *SWIFT* 風格和 Amazon *S3* 風格。

##	快速開始

###	OpenStack Swift 風格

```javascript
const swift = require('osapi/swift');

let conn = new swift.Connection({
    endPoint   : 'http://storage.example.com/',
    subuser    : 'userName:subUserName',
    key        : '380289ba59473a368c593c1f1de6efb0380289ba5', 
                 // generally 40 characters 
    tempURLKey : '380289ba59473a368c593c1f1de6efb0', 
                 // generally 32 characters
    container  : 'containerName',
});

conn.createObject('hello/world', 'Hello world!', (err) => {
    // ...
});

conn.readObject('hello/world', (err, data) => {
    // ...
    data.contentType;
    data.buffer;
});
```

###	AWS S3 風格

```javascript

const s3 = require('osapi/s3');

let conn = new s3.Connection({
    endPoint        : 'http://storage.example.com/',
    accessKey       : '380289ba59473a368c59', 
                       // 20 characters 
    secretAccessKey : '380289ba59473a368c593c1f1de6efb0380289ba5', 
                       // 40 characters
    bucket          : 'bucketName',
});

conn.createObject({
    name: 'hello/world',
    meta: { /* self defined meta info */ }
    }, 'Hello world!', (err) => {
    // ...
});

conn.readObject('hello/world', (err, data) => {
    // ...
    data.contentType;
    data.buffer;
    data.meta;
});
```

##	API

每套 __osapi/*__ 風格的 API 都會提供一個名為 `Connection` 的類，你可以從創建該類的實例開始，參見[快速開始](#快速開始)一節。

通常，`Connection` 類的成員方法都是異步的：

*   參數 `callback` 是可選的。
*   如果未提供 `callback` 參數，那麼成員方法返回一個 `Promise` 實例。
*   否則，`callback` 參數始終__應當__放在參數表的最末位置。
*   並且，回調函數自身的參數表形如 `callback(error, data)`。

###	自適應

```javascript
const osapi = require('osapi');

// See osapi/swift and osapi/s3 for details of options and member methods of the created connection.
const conn = osapi.createConnection(options);

osapi.isConnection(conn);
// RETURN: true

osapi.getConnectionStyle(conn);
// RETURN: 
// * "s3"     if conn is instance of osapi/s3.Connection
// * "swift"  if conn is instance of osapi/swift.Connection
// * null     otherwise
```

###	osapi/swift

`osapi/swift` 可以作為獨立模塊引用：

```javascript
const swift = require('osapi/swift');
```

相關細節及最新版本，請閱讀該子模塊的[詳細文檔](./docs/swift/index.md)。此處僅提供不甚完整的摘要：

*	new __swift.Connection__(*object* options)
*	__\<conn\>.createObject__(*object | string* options, content [, *function* callback ])
*	__\<conn\>.readObject__(*object | string* options [, *function* callback ])
*	__\<conn\>.deleteObject__(*object | string* options [, *function* callback ])
*	__\<conn\>.generateTempUrl__(*object | string* options [, *function* callback ])

注意：自 0.1.0 版本起，__osapi__ 入口將獨立發展為兼容 *s3* 和 *swift* 風格的工具類，不再默認指向 __osapi/swift__。

###	osapi/s3

`osapi/s3` 可以作為獨立模塊引用：

```javascript
const swift = require('osapi/s3');
```

相關細節及最新版本，請閱讀該子模塊的[詳細文檔](./docs/s3/index.md)。此處僅提供不甚完整的摘要：

*	new __s3.Connection__(*object* options)
*	__\<conn\>.createObject__(*object | string* options, content [, *function* callback ])
*	__\<conn\>.readObject__(*object | string* options [, *function* callback ])
*	__\<conn\>.deleteObject__(*object | string* options [, *function* callback ])
*	__\<conn\>.generateTempUrl__(*object | string* options [, *function* callback ])

###	自定義異常

*	class __OptionAbsentError__
*	class __RequestRefusedError__
	-	string __\<instance\>.action__
	-	Object __\<instance\>.meta__
	-	Object __\<instance\>.response__
		+	number __statusCode__
		+	string __statusMessage__
		+	string __code__

##  術語

亞馬遜的簡單存儲服務（S3）和 OpenStack Swift 有相似之外，但仍然是兩種不同的東西。

| S3                   | SWIFT          | 含義 |
| :----------------    | :------------- | :------------- |
| bucket               | container      | 屬於某個賬戶的、用於存儲對象的容器。 |
| access_key           | -              | 用於識別賬戶的唯一標識串。 |
| secret\_secret\_key  | -              | 用於確認請求合法性的令牌，相當於密碼。 |
| -                    | key            | 屬於某個子用戶，用於生成訪問令牌的密鑰。 |
| -                    | temp\_url\_key | 屬於某個子用戶，用於生成臨時下載 URL 的密鑰。 |
| -                    | user           | 對應某個賬戶。 |
| -                    | subuser        | 在指定賬戶下開設的子用戶，用於區分不同的訪問權限。 |

##  參考

*   [S3 vs Swift](https://oldhenhut.com/2016/05/31/s3-vs-swift/)
