### url（URL）

`url` 模块用于处理与解析 URL。 使用方法如下：

```js
const url = require('url');
```

- [官方文档API](http://nodejs.cn/api/url.html)

## URL 字符串与 URL 对象

URL 字符串是结构化的字符串，包含多个含义不同的组成部分。 解析字符串后返回的 URL 对象，每个属性对应字符串的各个组成部分。

`url` 模块提供了两套 API 来处理 URL：一个是旧版本遗留的 API，一个是实现了 [WHATWG标准](http://nodejs.cn/s/fKgW8d)的新 API。

遗留的 API 还没有被废弃，保留是为了兼容已存在的应用程序。 新的应用程序应使用 WHATWG 的 API。

WHATWG 的 API 与遗留的 API 的区别如下。 在下图中，URL `'http://user:pass@sub.example.com:8080/p/a/t/h?query=string#hash'` 上方的是遗留的 `url.parse()` 返回的对象的属性。 下方的则是 WHATWG 的 `URL` 对象的属性。

WHATWG 的 `origin` 属性包括 `protocol` 和 `host`，但不包括 `username` 或 `password`。

```txt
┌────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                              href                                              │
├──────────┬──┬─────────────────────┬────────────────────────┬───────────────────────────┬───────┤
│ protocol │  │        auth         │          host          │           path            │ hash  │
│          │  │                     ├─────────────────┬──────┼──────────┬────────────────┤       │
│          │  │                     │    hostname     │ port │ pathname │     search     │       │
│          │  │                     │                 │      │          ├─┬──────────────┤       │
│          │  │                     │                 │      │          │ │    query     │       │
"  https:   //    user   :   pass   @ sub.example.com : 8080   /p/a/t/h  ?  query=string   #hash "
│          │  │          │          │    hostname     │ port │          │                │       │
│          │  │          │          ├─────────────────┴──────┤          │                │       │
│ protocol │  │ username │ password │          host          │          │                │       │
├──────────┴──┼──────────┴──────────┼────────────────────────┤          │                │       │
│   origin    │                     │         origin         │ pathname │     search     │ hash  │
├─────────────┴─────────────────────┴────────────────────────┴──────────┴────────────────┴───────┤
│                                              href                                              │
└────────────────────────────────────────────────────────────────────────────────────────────────┘
```

使用 WHATWG 的 API 解析 URL 字符串：

```js
const myURL =
  new URL('https://user:pass@sub.host.com:8080/p/a/t/h?query=string#hash');
```

使用遗留的 API 解析 URL 字符串：

```js
const url = require('url');
const myURL =
  url.parse('https://user:pass@sub.host.com:8080/p/a/t/h?query=string#hash');
```

## WHATWG 的 URL 接口[#](http://nodejs.cn/api/url.html#url_the_whatwg_url_api)

### URL 类[#](http://nodejs.cn/api/url.html#url_class_url)

[中英对照](http://nodejs.cn/api/url/class_url.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/class_url.md)

版本历史

浏览器兼容的 `URL` 类，根据 WHATWG URL 标准实现。[解析URL的示例](http://nodejs.cn/s/NZjsQe)可以在标准本身里边找到。

*注意*: 根据浏览器的约定， `URL` 对象的所有属性都是在类的原型上实现为getter和setter，而不是作为对象本身的数据属性。因此，与[遗留的urlObjects](http://nodejs.cn/s/qt2Axh)不同，在 `URL` 对象的任何属性(例如 `delete myURL.protocol`， `delete myURL.pathname`等)上使用 `delete` 关键字没有任何效果，但仍返回 `true`。

#### new URL(input[, base])[#](http://nodejs.cn/api/url.html#url_constructor_new_url_input_base)

[中英对照](http://nodejs.cn/api/url/constructor_new_url_input_base.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/constructor_new_url_input_base.md)

- `input` <string>要解析的输入URL
- `base` <string>| <URL> 如果“input”是相对URL，则为要解析的基本URL。

通过将`input`解析到`base`上创建一个新的`URL`对象。如果`base`是一个字符串，则解析方法与`new URL(base)`相同。

```js
const { URL } = require('url');
const myURL = new URL('/foo', 'https://example.org/');
  // https://example.org/foo
```

如果`input`或`base`是无效URLs，将会抛出`TypeError`。请注意给定值将被强制转换为字符串。例如：

```js
const { URL } = require('url');
const myURL = new URL({ toString: () => 'https://example.org/' });
  // https://example.org/
```

存在于`input`主机名中的Unicode字符将被使用[Punycode](http://nodejs.cn/s/C2g98n)算法自动转换为ASCII。

```js
const { URL } = require('url');
const myURL = new URL('https://你好你好');
  // https://xn--6qqa088eba/
```

*Note*: This feature is only available if the `node` executable was compiled with [ICU](http://nodejs.cn/s/e2VfyT) enabled. If not, the domain names are passed through unchanged.

#### url.hash[#](http://nodejs.cn/api/url.html#url_url_hash)

- <string> 

获取及设置URL的分段(hash)部分。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org/foo#bar');
console.log(myURL.hash);
  // 输出 #bar

myURL.hash = 'baz';
console.log(myURL.href);
  // 输出 https://example.org/foo#baz
```

包含在赋给`hash`属性的值中的无效URL字符是[百分比编码][]。请注意选择哪些字符进行百分比编码可能与[url.parse()][]和[url.format()][]方法产生的不同。

#### url.host[#](http://nodejs.cn/api/url.html#url_url_host)

[中英对照](http://nodejs.cn/api/url/url_host.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_host.md)

- <string> 

获取及设置URL的主机(host)部分。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.host);
  // 输出 example.org:81

myURL.host = 'example.com:82';
console.log(myURL.href);
  // 输出 https://example.com:82/foo
```

如果给`host`属性设置的值是无效值，那么该值将被忽略。

#### url.hostname[#](http://nodejs.cn/api/url.html#url_url_hostname)

[中英对照](http://nodejs.cn/api/url/url_hostname.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_hostname.md)

- <string> 

获取及设置URL的主机名(hostname)部分。 `url.host`和`url.hostname`之间的区别是`url.hostname`*不* 包含端口。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org:81/foo');
console.log(myURL.hostname);
  // 输出 example.org

myURL.hostname = 'example.com:82';
console.log(myURL.href);
  // 输出 https://example.com:81/foo
```

如果给`hostname`属性设置的值是无效值，那么该值将被忽略。

#### url.href[#](http://nodejs.cn/api/url.html#url_url_href)

[中英对照](http://nodejs.cn/api/url/url_href.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_href.md)

- <string> 

获取及设置序列化的URL。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org/foo');
console.log(myURL.href);
  // 输出 https://example.org/foo

myURL.href = 'https://example.com/bar';
console.log(myURL.href);
  // 输出 https://example.com/bar
```

获取`href`属性的值等同于调用[`url.toString()`](http://nodejs.cn/s/EXzgVB)。

将此属性的值设置为新值等同于[`new URL(value)`](http://nodejs.cn/s/a19aPx)使用创建新的`URL`对象。`URL`对象的每个属性都将被修改。

如果给`href`属性设置的值是无效URL，将会抛出`TypeError`。

#### url.origin[#](http://nodejs.cn/api/url.html#url_url_origin)

[中英对照](http://nodejs.cn/api/url/url_origin.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_origin.md)

- <string> 

获取只读序列化的URL origin部分。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org/foo/bar?baz');
console.log(myURL.origin);
  // 输出 https://example.org
const { URL } = require('url');
const idnURL = new URL('https://你好你好');
console.log(idnURL.origin);
  // 输出 https://xn--6qqa088eba

console.log(idnURL.hostname);
  // 输出 xn--6qqa088eba
```

#### url.password[#](http://nodejs.cn/api/url.html#url_url_password)

[中英对照](http://nodejs.cn/api/url/url_password.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_password.md)

- <string> 

获取及设置URL的密码(password)部分。

```js
const { URL } = require('url');
const myURL = new URL('https://abc:xyz@example.com');
console.log(myURL.password);
  // 输出 xyz

myURL.password = '123';
console.log(myURL.href);
  // 输出 https://abc:123@example.com
```

包含在赋给`password`属性的值中的无效URL字符是[百分比编码][]。请注意选择哪些字符进行百分比编码可能与[`url.parse()`](http://nodejs.cn/s/b28B2A)和[`url.format()`](http://nodejs.cn/s/qWw3iQ)方法产生的不同。

#### url.pathname[#](http://nodejs.cn/api/url.html#url_url_pathname)

[中英对照](http://nodejs.cn/api/url/url_pathname.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_pathname.md)

- <string> 

获取及设置URL的路径(path)部分。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org/abc/xyz?123');
console.log(myURL.pathname);
  // 输出 /abc/xyz

myURL.pathname = '/abcdef';
console.log(myURL.href);
  // 输出 https://example.org/abcdef?123
```

包含在赋给`pathname`属性的值中的无效URL字符是[百分比编码][]。请注意选择哪些字符进行百分比编码可能与[`url.parse()`](http://nodejs.cn/s/b28B2A)和[`url.format()`](http://nodejs.cn/s/qWw3iQ)方法产生的不同。

#### url.port[#](http://nodejs.cn/api/url.html#url_url_port)

[中英对照](http://nodejs.cn/api/url/url_port.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_port.md)

- <string> 

获取及设置URL的端口(port)部分。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org:8888');
console.log(myURL.port);
  // 输出 8888

// 默认端口将自动转换为空字符
// (HTTPS协议默认端口是443)
myURL.port = '443';
console.log(myURL.port);
  // 输出空字符
console.log(myURL.href);
  // 输出 https://example.org/

myURL.port = 1234;
console.log(myURL.port);
  // 输出 1234
console.log(myURL.href);
  // 输出 https://example.org:1234/

// 完全无效的端口字符串将被忽略
myURL.port = 'abcd';
console.log(myURL.port);
  // 输出 1234

// 开头的数字将会被当做端口数
myURL.port = '5678abcd';
console.log(myURL.port);
  // 输出 5678

// 非整形数字将会被截取部分
myURL.port = 1234.5678;
console.log(myURL.port);
  // 输出 1234

// 超出范围的数字将被忽略
myURL.port = 1e10;
console.log(myURL.port);
  // 输出 1234
```

端口值可以被设置为数字或包含数字的字符串，数字范围`0`~`65535`(包括)。为给定`protocol`的`URL`对象设置端口值将会导致`port`值变成空字符(`''`)。

如果给`port`属性设置的值是无效字符串，但如果字符串以数字开头，那么开头部位的数字将会被赋值给`port`。否则，包括如果数字超出上述要求的数字，将被忽略。

#### url.protocol[#](http://nodejs.cn/api/url.html#url_url_protocol)

[中英对照](http://nodejs.cn/api/url/url_protocol.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_protocol.md)

- <string> 

获取及设置URL的协议(protocol)部分。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org');
console.log(myURL.protocol);
  // 输出 https:

myURL.protocol = 'ftp';
console.log(myURL.href);
  // 输出 ftp://example.org/
```

如果给`protocol`属性设置的值是无效值，那么该值将被忽略。

##### 特殊协议[#](http://nodejs.cn/api/url.html#url_special_schemes)

暂无中英对照[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/special_schemes.md)

The [WHATWG URL Standard](http://nodejs.cn/s/fKgW8d) considers a handful of URL protocol schemes to be *special* in terms of how they are parsed and serialized. When a URL is parsed using one of these special protocols, the `url.protocol` property may be changed to another special protocol but cannot be changed to a non-special protocol, and vice versa.

For instance, changing from `http` to `https` works:

```js
const u = new URL('http://example.org');
u.protocol = 'https';
console.log(u.href);
// https://example.org
```

However, changing from `http` to a hypothetical `fish` protocol does not because the new protocol is not special.

```js
const u = new URL('http://example.org');
u.protocol = 'fish';
console.log(u.href);
// http://example.org
```

Likewise, changing from a non-special protocol to a special protocol is also not permitted:

```js
const u = new URL('fish://example.org');
u.protocol = 'http';
console.log(u.href);
// fish://example.org
```

The protocol schemes considered to be special by the WHATWG URL Standard include: `ftp`, `file`, `gopher`, `http`, `https`, `ws`, and `wss`.

#### url.search[#](http://nodejs.cn/api/url.html#url_url_search)

[中英对照](http://nodejs.cn/api/url/url_search.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_search.md)

- <string> 

获取及设置URL的序列化查询(query)部分部分。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org/abc?123');
console.log(myURL.search);
  // 输出 ?123

myURL.search = 'abc=xyz';
console.log(myURL.href);
  // 输出 https://example.org/abc?abc=xyz
```

任何出现在赋给`search`属性值中的无效URL字符将被[百分比编码][]。请注意选择哪些字符进行百分比编码可能与[`url.parse()`](http://nodejs.cn/s/b28B2A)和[`url.format()`](http://nodejs.cn/s/qWw3iQ)方法产生的不同。

#### url.searchParams[#](http://nodejs.cn/api/url.html#url_url_searchparams)

[中英对照](http://nodejs.cn/api/url/url_searchparams.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_searchparams.md)

- [](http://nodejs.cn/s/AUtfEw)

获取表示URL查询参数的[`URLSearchParams`](http://nodejs.cn/s/AUtfEw)对象。该属性是只读的；使用[`url.search`](http://nodejs.cn/s/EgNerx)设置来替换URL的整个查询参数。请打开[`URLSearchParams`](http://nodejs.cn/s/AUtfEw)文档来查看更多细节。

#### url.username[#](http://nodejs.cn/api/url.html#url_url_username)

[中英对照](http://nodejs.cn/api/url/url_username.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_username.md)

- <string> 

获取及设置URL的用户名(username)部分。

```js
const { URL } = require('url');
const myURL = new URL('https://abc:xyz@example.com');
console.log(myURL.username);
  // 输出 abc

myURL.username = '123';
console.log(myURL.href);
  // 输出 https://123:xyz@example.com/
```

任何出现在赋给`username`属性值中的无效URL字符将被[百分比编码][]。请注意选择哪些字符进行百分比编码可能与[`url.parse()`](http://nodejs.cn/s/b28B2A)和[`url.format()`](http://nodejs.cn/s/qWw3iQ)方法产生的不同。

#### url.toString()[#](http://nodejs.cn/api/url.html#url_url_tostring)

[中英对照](http://nodejs.cn/api/url/url_tostring.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_tostring.md)

- 返回: <string> 

在`URL`对象上调用`toString()`方法将返回序列化的URL。返回值与[`url.href`](http://nodejs.cn/s/Feif9B)和[`url.toJSON()`](http://nodejs.cn/s/naUyEW)的相同。

由于需要符合标准，此方法不允许用户自定义URL的序列化过程。 如果需要更大灵活性，[`require('url').format()`](http://nodejs.cn/s/GVUBbh)可能更合适。

#### url.toJSON()[#](http://nodejs.cn/api/url.html#url_url_tojson)

[中英对照](http://nodejs.cn/api/url/url_tojson.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_tojson.md)

- 返回: <string> 

在`URL`对象上调用`toJSON()`方法将返回序列化的URL。返回值与[`url.href`](http://nodejs.cn/s/Feif9B)和[`url.toString()`](http://nodejs.cn/s/EXzgVB)的相同。

当`URL`对象使用[`JSON.stringify()`](http://nodejs.cn/s/bmLTNS)序列化时将自动调用该方法。

```js
const { URL } = require('url');
const myURLs = [
  new URL('https://www.example.com'),
  new URL('https://test.example.org')
];
console.log(JSON.stringify(myURLs));
  // 输出 ["https://www.example.com/","https://test.example.org/"]
```

### URLSearchParams 类[#](http://nodejs.cn/api/url.html#url_class_urlsearchparams)

[中英对照](http://nodejs.cn/api/url/class_urlsearchparams.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/class_urlsearchparams.md)

版本历史

`URLSearchParams`API接口提供对`URL`query部分的读写权限。`URLSearchParams`类也能够与以下四个构造函数中的任意一个单独使用。

WHATWG `URLSearchParams`接口和[`querystring`](http://nodejs.cn/s/i23Gdh)模块有相似的目的，但是[`querystring`](http://nodejs.cn/s/i23Gdh)模块的目的更加通用，因为它可以定制分隔符（`＆`和`=`）。但另一方面，这个API是专门为URL查询字符串而设计的。

```js
const { URL, URLSearchParams } = require('url');

const myURL = new URL('https://example.org/?abc=123');
console.log(myURL.searchParams.get('abc'));
// 输出 123

myURL.searchParams.append('abc', 'xyz');
console.log(myURL.href);
// 输出 https://example.org/?abc=123&abc=xyz

myURL.searchParams.delete('abc');
myURL.searchParams.set('a', 'b');
console.log(myURL.href);
// 输出 https://example.org/?a=b

const newSearchParams = new URLSearchParams(myURL.searchParams);
// 上面的代码等同于
// const newSearchParams = new URLSearchParams(myURL.search);

newSearchParams.append('a', 'c');
console.log(myURL.href);
// 输出 https://example.org/?a=b
console.log(newSearchParams.toString());
// 输出 a=b&a=c

// newSearchParams.toString() 被隐式调用
myURL.search = newSearchParams;
console.log(myURL.href);
// 输出 https://example.org/?a=b&a=c
newSearchParams.delete('a');
console.log(myURL.href);
// 输出 https://example.org/?a=b&a=c
```

#### new URLSearchParams()[#](http://nodejs.cn/api/url.html#url_constructor_new_urlsearchparams)

[中英对照](http://nodejs.cn/api/url/constructor_new_urlsearchparams.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/constructor_new_urlsearchparams.md)

实例化一个新的空的`URLSearchParams`对象。

#### new URLSearchParams(string)[#](http://nodejs.cn/api/url.html#url_constructor_new_urlsearchparams_string)

[中英对照](http://nodejs.cn/api/url/constructor_new_urlsearchparams_string.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/constructor_new_urlsearchparams_string.md)

- `string` <string>一个查询字符串

将`string`解析成一个查询字符串, 并且使用它来实例化一个新的`URLSearchParams`对象.  如果`string`以`'?'`打头,则`'?'`将会被忽略.

```js
const { URLSearchParams } = require('url');
let params;

params = new URLSearchParams('user=abc&query=xyz');
console.log(params.get('user'));
  // 输出 'abc'
console.log(params.toString());
  // 输出 'user=abc&query=xyz'

params = new URLSearchParams('?user=abc&query=xyz');
console.log(params.toString());
  // 输出 'user=abc&query=xyz'
```

#### new URLSearchParams(obj)[#](http://nodejs.cn/api/url.html#url_constructor_new_urlsearchparams_obj)

[中英对照](http://nodejs.cn/api/url/constructor_new_urlsearchparams_obj.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/constructor_new_urlsearchparams_obj.md)

新增于: v7.10.0, v6.13.0

- `obj` [](http://nodejs.cn/s/jzn6Ao) 一个表示键值对集合的对象

通过使用查询哈希映射实例化一个新的`URLSearchParams`对象， `obj`的每一个属性的键和值将被强制转换为字符串。

*请注意*: 和 [`querystring`](http://nodejs.cn/s/i23Gdh) 模块不同的是, 在数组的形式中，重复的键是不允许的。数组使用[`array.toString()`](http://nodejs.cn/s/q2o4mr)进行字符串化时，只需用逗号连接所有的数组元素即可。

```js
const { URLSearchParams } = require('url');
const params = new URLSearchParams({
  user: 'abc',
  query: ['first', 'second']
});
console.log(params.getAll('query'));
  // 输出 [ 'first,second' ]
console.log(params.toString());
  // 输出 'user=abc&query=first%2Csecond'
```

#### new URLSearchParams(iterable)[#](http://nodejs.cn/api/url.html#url_constructor_new_urlsearchparams_iterable)

[中英对照](http://nodejs.cn/api/url/constructor_new_urlsearchparams_iterable.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/constructor_new_urlsearchparams_iterable.md)

新增于: v7.10.0, v6.13.0

- `iterable` [](http://nodejs.cn/s/mQfCyy) 一个元素时键值对的迭代对象

以一种类似于[`Map`](http://nodejs.cn/s/EnuJtG)的构造函数的迭代映射方式实例化一个新的`URLSearchParams`对象。`iterable`可以是一个数组或者任何迭代对象。这就意味着`iterable`能够使另一个`URLSearchParams`，这种情况下，构造函数将简单地根据提供的`URLSearchParams`创建一个克隆`URLSearchParams`。`iterable`的元素是键值对，并且其本身也可以是任何迭代对象。

允许重复的键。

```js
const { URLSearchParams } = require('url');
let params;

// Using an array
params = new URLSearchParams([
  ['user', 'abc'],
  ['query', 'first'],
  ['query', 'second']
]);
console.log(params.toString());
  // 输出 'user=abc&query=first&query=second'

// 使用Map对象
const map = new Map();
map.set('user', 'abc');
map.set('query', 'xyz');
params = new URLSearchParams(map);
console.log(params.toString());
  // 输出 'user=abc&query=xyz'

// 使用generator函数
function* getQueryPairs() {
  yield ['user', 'abc'];
  yield ['query', 'first'];
  yield ['query', 'second'];
}
params = new URLSearchParams(getQueryPairs());
console.log(params.toString());
  // 输出 'user=abc&query=first&query=second'

// 每个键值对必须有两个元素
new URLSearchParams([
  ['user', 'abc', 'error']
]);
  // 抛出 TypeError [ERR_INVALID_TUPLE]:
  //        每一个键值对必须是迭代的[键，值]元组
```

#### urlSearchParams.append(name, value)[#](http://nodejs.cn/api/url.html#url_urlsearchparams_append_name_value)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_append_name_value.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_append_name_value.md)

- `name` <string> 
- `value` <string> 

在查询字符串中附加一个新的键值对。

#### urlSearchParams.delete(name)[#](http://nodejs.cn/api/url.html#url_urlsearchparams_delete_name)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_delete_name.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_delete_name.md)

- `name` <string> 

删除所有键为`name`的键值对。

#### urlSearchParams.entries()[#](http://nodejs.cn/api/url.html#url_urlsearchparams_entries)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_entries.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_entries.md)

- 返回: [](http://nodejs.cn/s/Y2SE1q) 在查询中的每个键值对上返回一个ES6迭代器。 迭代器的每一项都是一个JavaScript数组。 Array的第一个项是键`name`，Array的第二个项是值`value`。

别名为[`urlSearchParams[@@iterator]()`](http://nodejs.cn/s/tkT8B6).

#### urlSearchParams.forEach(fn[, thisArg])[#](http://nodejs.cn/api/url.html#url_urlsearchparams_foreach_fn_thisarg)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_foreach_fn_thisarg.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_foreach_fn_thisarg.md)

- `fn` [](http://nodejs.cn/s/ceTQa6) 在查询字符串中的每个键值对的调用函数。
- `thisArg` [](http://nodejs.cn/s/jzn6Ao) 当`fn`调用时，被用作`this`值的对象

在查询字符串中迭代每个键值对，并调用给定的函数。

```js
const { URL } = require('url');
const myURL = new URL('https://example.org/?a=b&c=d');
myURL.searchParams.forEach((value, name, searchParams) => {
  console.log(name, value, myURL.searchParams === searchParams);
});
  // 输出:
  // a b true
  // c d true
```

#### urlSearchParams.get(name)[#](http://nodejs.cn/api/url.html#url_urlsearchparams_get_name)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_get_name.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_get_name.md)

- `name` <string> 
- 返回: <string>，如果没有键值对对应给定的`name`则返回`null`。

返回键是`name`的第一个键值对的值。如果没有对应的键值对，则返回`null`。

#### urlSearchParams.getAll(name)[#](http://nodejs.cn/api/url.html#url_urlsearchparams_getall_name)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_getall_name.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_getall_name.md)

- `name` <string> 
- 返回: [](http://nodejs.cn/s/ZJSz23)

返回键是`name`的所有键值对的值，如果没有满足条件的键值对，则返回一个空的数组。

#### urlSearchParams.has(name)[#](http://nodejs.cn/api/url.html#url_urlsearchparams_has_name)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_has_name.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_has_name.md)

- `name` <string> 
- 返回: [](http://nodejs.cn/s/jFbvuT)

如果存在至少一对键是name的键值对则返回 `true`。

#### urlSearchParams.keys()[#](http://nodejs.cn/api/url.html#url_urlsearchparams_keys)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_keys.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_keys.md)

- 返回: [](http://nodejs.cn/s/Y2SE1q)

在每一个键值对上返回一个键的ES6迭代器。

```js
const { URLSearchParams } = require('url');
const params = new URLSearchParams('foo=bar&foo=baz');
for (const name of params.keys()) {
  console.log(name);
}
  // 输出:
  // foo
  // foo
```

#### urlSearchParams.set(name, value)[#](http://nodejs.cn/api/url.html#url_urlsearchparams_set_name_value)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_set_name_value.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_set_name_value.md)

- `name` <string> 
- `value` <string> 

将`URLSearchParams`对象中与`name`相对应的值设置为`value`。如果已经存在键为`name`的键值对，将第一对的值设为`value`并且删除其他对。如果不存在，则将此键值对附加在查询字符串后。

```js
const { URLSearchParams } = require('url');

const params = new URLSearchParams();
params.append('foo', 'bar');
params.append('foo', 'baz');
params.append('abc', 'def');
console.log(params.toString());
  // 输出 foo=bar&foo=baz&abc=def

params.set('foo', 'def');
params.set('xyz', 'opq');
console.log(params.toString());
  // 输出 foo=def&abc=def&xyz=opq
```

#### urlSearchParams.sort()[#](http://nodejs.cn/api/url.html#url_urlsearchparams_sort)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_sort.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_sort.md)

新增于: v7.7.0, v6.13.0

按现有名称就地排列所有的名称-值对。使用[稳定排序算法][]完成排序，因此保留具有相同名称的名称-值对之间的相对顺序。

特别地，该方法可以用来增加缓存命中。

```js
const params = new URLSearchParams('query[]=abc&type=search&query[]=123');
params.sort();
console.log(params.toString());
  // 打印 query%5B%5D=abc&query%5B%5D=123&type=search
```

#### urlSearchParams.toString()[#](http://nodejs.cn/api/url.html#url_urlsearchparams_tostring)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_tostring.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_tostring.md)

- 返回: <string> 

返回查询参数序列化后的字符串，必要时存在百分号编码字符。

#### urlSearchParams.values()[#](http://nodejs.cn/api/url.html#url_urlsearchparams_values)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_values.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_values.md)

- Returns: [](http://nodejs.cn/s/Y2SE1q)

在每一个键值对上返回一个值的ES6迭代器。

#### urlSearchParams[Symbol.iterator]()[#](http://nodejs.cn/api/url.html#url_urlsearchparams_symbol_iterator)

[中英对照](http://nodejs.cn/api/url/urlsearchparams_symbol_iterator.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlsearchparams_symbol_iterator.md)

- Returns: [](http://nodejs.cn/s/Y2SE1q)

根据查询字符串，返回一个键值对形式的 ES6 `Iterator`。每个迭代器的项是一个 JavaScript `Array`。 其中， `Array` 的第一项是 `name`，第二个是 `value`。

是 [`urlSearchParams.entries()`](http://nodejs.cn/s/qW6kzi) 的别名。

```js
const params = new URLSearchParams('foo=bar&xyz=baz');
for (const [name, value] of params) {
  console.log(name, value);
}
// Prints:
//   foo bar
//   xyz baz
```

### url.domainToASCII(domain)[#](http://nodejs.cn/api/url.html#url_url_domaintoascii_domain)

[中英对照](http://nodejs.cn/api/url/url_domaintoascii_domain.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_domaintoascii_domain.md)

新增于: v7.4.0, v6.13.0

- `domain` <string> 
- 返回: <string> 

返回[Punycode](http://nodejs.cn/s/C2g98n) ASCII序列化的`domain`. 如果`domain`是无效域名，将返回空字符串。

它执行的是[`url.domainToUnicode()`](http://nodejs.cn/s/396eXf)的逆运算。

```js
const url = require('url');
console.log(url.domainToASCII('español.com'));
  // 输出 xn--espaol-zwa.com
console.log(url.domainToASCII('中文.com'));
  // 输出 xn--fiq228c.com
console.log(url.domainToASCII('xn--iñvalid.com'));
  // 输出空字符串
```

### url.domainToUnicode(domain)[#](http://nodejs.cn/api/url.html#url_url_domaintounicode_domain)

[中英对照](http://nodejs.cn/api/url/url_domaintounicode_domain.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_domaintounicode_domain.md)

新增于: v7.4.0, v6.13.0

- `domain` <string> 
- 返回: <string> 

返回Unicode序列化的`domain`. 如果`domain`是无效域名，将返回空字符串。

它执行的是[`url.domainToASCII()`](http://nodejs.cn/s/TQ8Wpc)的逆运算。

```js
const url = require('url');
console.log(url.domainToUnicode('xn--espaol-zwa.com'));
  // 输出 español.com
console.log(url.domainToUnicode('xn--fiq228c.com'));
  // 输出 中文.com
console.log(url.domainToUnicode('xn--iñvalid.com'));
  // 输出空字符串
```

### url.fileURLToPath(url)[#](http://nodejs.cn/api/url.html#url_url_fileurltopath_url)

[中英对照](http://nodejs.cn/api/url/url_fileurltopath_url.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_fileurltopath_url.md)

新增于: v10.12.0

- `url` <URL> | <string>被转换为路径的文件的 URL 字符串或者 URL 对象。
- 返回: <string>被完全解析的平台特定的 Node.js 文件路径。

此方法保证百分号编码字符解码结果的正确性，同时也确保绝对路径字符串在不同平台下的有效性。

```js
new URL('file:///C:/path/').pathname;    // 错误: /C:/path/
fileURLToPath('file:///C:/path/');       // 正确: C:\path\ (Windows)

new URL('file://nas/foo.txt').pathname;  // 错误: /foo.txt
fileURLToPath('file://nas/foo.txt');     // 正确: \\nas\foo.txt (Windows)

new URL('file:///你好.txt').pathname;    // 错误: /%E4%BD%A0%E5%A5%BD.txt
fileURLToPath('file:///你好.txt');       // 正确: /你好.txt (POSIX)

new URL('file:///hello world').pathname; // 错误: /hello%20world
fileURLToPath('file:///hello world');    // 正确: /hello world (POSIX)
```

### url.format(URL[, options])[#](http://nodejs.cn/api/url.html#url_url_format_url_options)

[中英对照](http://nodejs.cn/api/url/url_format_url_options.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_format_url_options.md)

新增于: v7.6.0

- `URL` <URL> 一个[WHATWG URL](http://nodejs.cn/s/5dwq7G)对象
- `options` [](http://nodejs.cn/s/jzn6Ao)
  - `auth` [](http://nodejs.cn/s/jFbvuT) 如果序列化的URL字符串应该包含用户名和密码为`true`，否则为`false`。默认为`true`。
  - `fragment` [](http://nodejs.cn/s/jFbvuT) 如果序列化的URL字符串应该包含分段为`true`，否则为`false`。默认为`true`。
  - `search` [](http://nodejs.cn/s/jFbvuT) 如果序列化的URL字符串应该包含搜索查询为`true`，否则为`false`。默认为`true`。
  - `unicode` [](http://nodejs.cn/s/jFbvuT) `true` 如果出现在URL字符串主机元素里的Unicode字符应该被直接编码而不是使用Punycode编码为`true`，默认为`false`。

返回一个[WHATWG URL](http://nodejs.cn/s/5dwq7G)对象的可自定义序列化的URL字符串表达。

虽然URL对象的`toString()`方法和`href`属性都可以返回URL的序列化的字符串。然而，两者都不可以被自定义。而`url.format(URL[, options])`方法允许输出的基本自定义。

例如：

```js
const { URL } = require('url');
const myURL = new URL('https://a:b@你好你好?abc#foo');

console.log(myURL.href);
  // 输出 https://a:b@xn--6qqa088eba/?abc#foo

console.log(myURL.toString());
  // 输出 https://a:b@xn--6qqa088eba/?abc#foo

console.log(url.format(myURL, { fragment: false, unicode: true, auth: false }));
  // 输出 'https://你好你好/?abc'
```

### url.pathToFileURL(path)[#](http://nodejs.cn/api/url.html#url_url_pathtofileurl_path)

暂无中英对照[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_pathtofileurl_path.md)

新增于: v10.12.0

- `path` <string>The path to convert to a File URL.
- Returns: <URL> The file URL object.

This function ensures that `path` is resolved absolutely, and that the URL control characters are correctly encoded when converting into a File URL.

```js
new URL(__filename);                // Incorrect: throws (POSIX)
new URL(__filename);                // Incorrect: C:\... (Windows)
pathToFileURL(__filename);          // Correct:   file:///... (POSIX)
pathToFileURL(__filename);          // Correct:   file:///C:/... (Windows)

new URL('/foo#1', 'file:');         // Incorrect: file:///foo#1
pathToFileURL('/foo#1');            // Correct:   file:///foo%231 (POSIX)

new URL('/some/path%.js', 'file:'); // Incorrect: file:///some/path%
pathToFileURL('/some/path%.js');    // Correct:   file:///some/path%25 (POSIX)
```

## 遗留的 URL 接口[#](http://nodejs.cn/api/url.html#url_legacy_url_api)

### 遗留的 urlObject[#](http://nodejs.cn/api/url.html#url_legacy_urlobject)

[中英对照](http://nodejs.cn/api/url/legacy_urlobject.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/legacy_urlobject.md)

遗留的urlObject (`require('url').Url`)由`url.parse()`函数创建并返回。

#### urlObject.auth[#](http://nodejs.cn/api/url.html#url_urlobject_auth)

[中英对照](http://nodejs.cn/api/url/urlobject_auth.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_auth.md)

`auth` 属性是 URL 的用户名与密码部分。 该字符串跟在 `protocol` 和双斜杠（如果有）的后面，排在 `host` 部分的前面且被一个 ASCII 的 at 符号（`@`）分隔。 该字符的格式为 `{username}[:{password}]`， `[:{password}]` 部分是可选的。

例如：`'user:pass'`

#### urlObject.hash[#](http://nodejs.cn/api/url.html#url_urlobject_hash)

[中英对照](http://nodejs.cn/api/url/urlobject_hash.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_hash.md)

`hash` 属性包含 URL 的碎片部分，包括开头的 ASCII 哈希字符（`#`）。

例如：`'#hash'`

#### urlObject.host[#](http://nodejs.cn/api/url.html#url_urlobject_host)

[中英对照](http://nodejs.cn/api/url/urlobject_host.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_host.md)

`host` 属性是 URL 的完整的小写的主机部分，包括 `port`（如果有）。

例如：`'sub.host.com:8080'`

#### urlObject.hostname[#](http://nodejs.cn/api/url.html#url_urlobject_hostname)

[中英对照](http://nodejs.cn/api/url/urlobject_hostname.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_hostname.md)

`hostname` 属性是 `host` 组成部分排除 `port` 之后的小写的主机名部分。

例如：`'sub.host.com'`

#### urlObject.href[#](http://nodejs.cn/api/url.html#url_urlobject_href)

[中英对照](http://nodejs.cn/api/url/urlobject_href.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_href.md)

`href` 属性是解析后的完整的 URL 字符串， `protocol` 和 `host` 都会被转换为小写的。

例如：`'http://user:pass@sub.host.com:8080/p/a/t/h?query=string#hash'`

#### urlObject.path[#](http://nodejs.cn/api/url.html#url_urlobject_path)

[中英对照](http://nodejs.cn/api/url/urlobject_path.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_path.md)

`path` 属性是一个 `pathname` 与 `search` 组成部分的串接。

例如：`'/p/a/t/h?query=string'`

不会对 `path` 执行解码。

#### urlObject.pathname[#](http://nodejs.cn/api/url.html#url_urlobject_pathname)

[中英对照](http://nodejs.cn/api/url/urlobject_pathname.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_pathname.md)

`pathname` 属性包含 URL 的整个路径部分。 它跟在 `host` （包括 `port`）后面，排在 `query` 或 `hash` 组成部分的前面且被 ASCII 问号（`?`）或哈希字符（`#`）分隔。

例如：`'/p/a/t/h'`

不会对路径字符串执行解码。

#### urlObject.port[#](http://nodejs.cn/api/url.html#url_urlobject_port)

[中英对照](http://nodejs.cn/api/url/urlobject_port.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_port.md)

`port` 属性是 `host` 组成部分中的数值型的端口部分。

例如：`'8080'`

#### urlObject.protocol[#](http://nodejs.cn/api/url.html#url_urlobject_protocol)

[中英对照](http://nodejs.cn/api/url/urlobject_protocol.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_protocol.md)

`protocol` 属性表明 URL 的小写的协议体制。

例如：`'http:'`

#### urlObject.query[#](http://nodejs.cn/api/url.html#url_urlobject_query)

[中英对照](http://nodejs.cn/api/url/urlobject_query.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_query.md)

`query` 属性是不含开头 ASCII 问号（`?`）的查询字符串，或一个被 [`querystring`](http://nodejs.cn/s/i23Gdh) 模块的 `parse()` 方法返回的对象。 `query` 属性是一个字符串还是一个对象是由传入 `url.parse()` 的 `parseQueryString` 参数决定的。

例如：`'query=string'` or `{'query': 'string'}`

如果返回一个字符串，则不会对查询字符串执行解码。 如果返回一个对象，则键和值都会被解码。

#### urlObject.search[#](http://nodejs.cn/api/url.html#url_urlobject_search)

[中英对照](http://nodejs.cn/api/url/urlobject_search.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_search.md)

`search` 属性包含 URL 的整个查询字符串部分，包括开头的 ASCII 问号字符（`?`）。

例如：`'?query=string'`

不会对查询字符串执行解码。

#### urlObject.slashes[#](http://nodejs.cn/api/url.html#url_urlobject_slashes)

[中英对照](http://nodejs.cn/api/url/urlobject_slashes.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/urlobject_slashes.md)

`slashes` 属性是一个 `boolean`，如果 `protocol` 中的冒号后面跟着两个 ASCII 斜杠字符（`/`），则值为 `true`。

### url.format(urlObject)[#](http://nodejs.cn/api/url.html#url_url_format_urlobject)

[中英对照](http://nodejs.cn/api/url/url_format_urlobject.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_format_urlobject.md)

版本历史

- `urlObject` [](http://nodejs.cn/s/jzn6Ao) | <string>一个 URL 对象（就像 `url.parse()` 返回的）。 如果是一个字符串，则通过 `url.parse()` 转换为一个对象。

`url.format()` 方法返回一个从 `urlObject` 格式化后的 URL 字符串。

如果 `urlObject` 不是一个对象或字符串，则 `url.format()` 抛出 [`TypeError`](http://nodejs.cn/s/Z7Lqyj)。

格式化过程如下：

- 创建一个新的空字符串 `result`。
- 如果 `urlObject.protocol` 是一个字符串，则它会被原样添加到 `result`。
- 否则，如果 `urlObject.protocol` 不是 `undefined` 也不是一个字符串，则抛出 [`Error`](http://nodejs.cn/s/FLTm19)。
- 对于不是以 `:` 结束的 `urlObject.protocol`， `:` 会被添加到 `result`。
- 如果以下条件之一为真，则 `//` 会被添加到 `result`：
  - `urlObject.slashes` 属性为真；
  - `urlObject.protocol` 以 `http`、 `https`、 `ftp`、 `gopher` 或 `file` 开头；
- 如果 `urlObject.auth` 属性的值为真，且 `urlObject.host` 或 `urlObject.hostname` 不为 `undefined`，则 `urlObject.auth` 会被添加到 `result`，且后面带上 `@`。
- 如果 `urlObject.host` 属性为 `undefined`，则：
  - 如果 `urlObject.hostname` 是一个字符串，则它会被添加到 `result`。
  - 否则，如果 `urlObject.hostname` 不是 `undefined` 也不是一个字符串，则抛出 [`Error`](http://nodejs.cn/s/FLTm19)。
  - 如果 `urlObject.port` 属性的值为真，且 `urlObject.hostname` 不为 `undefined`：
    - `:` 会被添加到 `result`。
    - `urlObject.port` 的值会被添加到 `result`。
- 否则，如果 `urlObject.host` 属性的值为真，则 `urlObject.host` 的值会被添加到 `result`。
- 如果 `urlObject.pathname` 属性是一个字符串且不是一个空字符串：
  - 如果 `urlObject.pathname` 不是以 `/` 开头，则 `/` 会被添加到 `result`。
  - `urlObject.pathname` 的值会被添加到 `result`。
- 否则，如果 `urlObject.pathname` 不是 `undefined` 也不是一个字符串，则抛出 [`Error`](http://nodejs.cn/s/FLTm19)。
- 如果 `urlObject.search` 属性为 `undefined` 且 `urlObject.query` 属性是一个 `Object`，则 `?` 会被添加到 `result`，后面跟上把 `urlObject.query` 的值传入 [`querystring`](http://nodejs.cn/s/i23Gdh)模块的 `stringify()` 方法的调用结果。
- 否则，如果 `urlObject.search` 是一个字符串：
  - 如果 `urlObject.search` 的值不是以 `?` 开头，则 `?` 会被添加到 `result`。
  - `urlObject.search` 的值会被添加到 `result`。
- 否则，如果 `urlObject.search` 不是 `undefined` 也不是一个字符串，则抛出 [`Error`](http://nodejs.cn/s/FLTm19)。
- 如果 `urlObject.hash` 属性是一个字符串：
  - 如果 `urlObject.hash` 的值不是以 `#` 开头，则 `#` 会被添加到 `result`。
  - `urlObject.hash` 的值会被添加到 `result`。
- 否则，如果 `urlObject.hash` 属性不是 `undefined` 也不是一个字符串，则抛出 [`Error`](http://nodejs.cn/s/FLTm19)。
- 返回 `result`。

### url.parse(urlString[, parseQueryString[, slashesDenoteHost]])[#](http://nodejs.cn/api/url.html#url_url_parse_urlstring_parsequerystring_slashesdenotehost)

[中英对照](http://nodejs.cn/api/url/url_parse_urlstring_parsequerystring_slashesdenotehost.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_parse_urlstring_parsequerystring_slashesdenotehost.md)

版本历史

- `urlString` <string>要解析的 URL 字符串。
- `parseQueryString` [](http://nodejs.cn/s/jFbvuT) 如果设为 `true`，则返回的 URL 对象的 `query` 属性会是一个使用 [`querystring`](http://nodejs.cn/s/i23Gdh) 模块的 `parse()` 生成的对象。 如果设为 `false`，则 `query`会是一个未解析未解码的字符串。 默认为 `false`。
- `slashesDenoteHost` [](http://nodejs.cn/s/jFbvuT) 如果设为 `true`，则 `//` 之后至下一个 `/` 之前的字符串会解析作为 `host`。 例如， `//foo/bar` 会解析为 `{host: 'foo', pathname: '/bar'}` 而不是 `{pathname: '//foo/bar'}`。 默认为 `false`。

解析 URL 字符串并返回 URL 对象。

如果 `urlString` 不是字符串，则抛出`TypeError`。

如果 `auth` 属性存在但无法解码，则抛出 `URIError`。

### url.resolve(from, to)[#](http://nodejs.cn/api/url.html#url_url_resolve_from_to)

[中英对照](http://nodejs.cn/api/url/url_resolve_from_to.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/url_resolve_from_to.md)

版本历史

- `from` <string>解析时相对的基本 URL。
- `to` <string>要解析的超链接 URL。

`url.resolve()` 方法会以一种 Web 浏览器解析超链接的方式把一个目标 URL 解析成相对于一个基础 URL。

例子：

```js
const url = require('url');
url.resolve('/one/two/three', 'four');         // '/one/two/four'
url.resolve('http://example.com/', '/one');    // 'http://example.com/one'
url.resolve('http://example.com/one', '/two'); // 'http://example.com/two'
```



## URL 中的百分号编码[#](http://nodejs.cn/api/url.html#url_percent_encoding_in_urls)

[中英对照](http://nodejs.cn/api/url/percent_encoding_in_urls.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/percent_encoding_in_urls.md)

允许URL只包含一定范围的字符。 任何超出该范围的字符都必须进行编码。 如何对这些字符进行编码，以及哪些字符要编码完全取决于字符在URL结构内的位置。

### 遗留的接口[#](http://nodejs.cn/api/url.html#url_legacy_api)

[中英对照](http://nodejs.cn/api/url/legacy_api.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/legacy_api.md)

在遗留的API中，空格(`' '`)及以下字符将自动转义为URL对象的属性：

```txt
< > " ` \r \n \t { } | \ ^ '
```

例如，ASCII 空格字符（`' '`）被编码成 `%20`。 ASCII 斜杠字符（`/`）被编码成 `%3C`。

### WHATWG 接口[#](http://nodejs.cn/api/url.html#url_whatwg_api)

[中英对照](http://nodejs.cn/api/url/whatwg_api.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/url/whatwg_api.md)

[WHATWG URL Standard](http://nodejs.cn/s/fKgW8d)使用比遗留的API更具选择性和更精细的方法来选择使用的编码字符。

WHATWG算法定义了三个“百分比编码集”，它们描述了必须进行百分编码的字符范围：

- *C0 control percent-encode set(C0控制百分比编码集)* 包括范围在U+0000 ~ U+001F（含）的代码点及大于U+007E的所有代码点。
- *path percent-encode set(路径百分比编码集)* 包括 *C0 control percent-encode set(C0控制百分比编码集)* 的代码点 及 U+0020, U+0022, U+0023, U+003C, U+003E, U+003F, U+0060, U+007B, 和 U+007D 的代码点。
- *userinfo encode set(用户信息编码集)* 包括 *path percent-encode set(路径百分比编码集)* 的代码点 及 U+002F, U+003A, U+003B, U+003D, U+0040, U+005B, U+005C, U+005D, U+005E, 和 U+007C 的代码点。

*userinfo percent-encode set(用户信息百分比编码集)* 专门用于用户名和密码部分的编码。*path percent-encode set(路径百分比编码集)* 用于大多数URL的路径部分编码。*C0 control percent-encode set(C0控制百分比编码集)* 则用于所有其他情况的编码，特别地包括URL的分段部分，特殊条件下也包括主机及路径部分。

当主机名中出现非ASCII字符时，主机名将使用[Punycode](http://nodejs.cn/s/C2g98n)算法进行编码。然而，请注意，主机名*可能同时* 包含Punycode编码和百分比编码的字符。例如：

```js
const { URL } = require('url');
const myURL = new URL('https://%CF%80.com/foo');
console.log(myURL.href);
  // 输出 https://xn--1xa.com/foo
console.log(myURL.origin);
  // 输出 https://π.com
```