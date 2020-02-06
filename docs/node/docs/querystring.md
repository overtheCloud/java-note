### querystring（查询字符串）

`querystring` 模块提供用于解析和格式化 URL 查询字符串的实用工具。 它可以使用以下方式访问：

```js
const querystring = require('querystring');
```

## querystring.decode()[#](http://nodejs.cn/api/querystring.html#querystring_querystring_decode)

[中英对照](http://nodejs.cn/api/querystring/querystring_decode.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/querystring/querystring_decode.md)

新增于: v0.1.99

`querystring.decode()` 函数是 `querystring.parse()` 的别名。

## querystring.encode()[#](http://nodejs.cn/api/querystring.html#querystring_querystring_encode)

[中英对照](http://nodejs.cn/api/querystring/querystring_encode.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/querystring/querystring_encode.md)

新增于: v0.1.99

`querystring.encode()` 函数是 `querystring.stringify()` 的别名。

## querystring.escape(str)[#](http://nodejs.cn/api/querystring.html#querystring_querystring_escape_str)

[中英对照](http://nodejs.cn/api/querystring/querystring_escape_str.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/querystring/querystring_escape_str.md)

新增于: v0.1.25

- `str` <string> 

`querystring.escape()` 方法以对 URL 查询字符串的特定要求进行了优化的方式对给定的 `str` 执行 URL 百分比编码。

`querystring.escape()` 方法由 `querystring.stringify()` 使用，通常不会直接使用。 它的导出主要是为了允许应用程序代码在必要时通过将 `querystring.escape` 指定给替代函数来提供替换的百分比编码实现。

## querystring.parse(str[, sep[, eq[, options]]])[#](http://nodejs.cn/api/querystring.html#querystring_querystring_parse_str_sep_eq_options)

[中英对照](http://nodejs.cn/api/querystring/querystring_parse_str_sep_eq_options.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/querystring/querystring_parse_str_sep_eq_options.md)

版本历史

- `str` <string>  要解析的 URL 查询字符串。
- `sep` <string>  用于在查询字符串中分隔键值对的子字符串。**默认值:** `'&'`。
- `eq` <string>  用于在查询字符串中分隔键和值的子字符串。**默认值:** `'='`。
- `options`<Object> 
  - `decodeURIComponent` <Function>  解码查询字符串中的百分比编码字符时使用的函数。**默认值:** `querystring.unescape()`。
  - `maxKeys`<number>  指定要解析的键的最大数量。指定 `0` 可移除键的计数限制。**默认值:** `1000`。

`querystring.parse()` 方法将 URL 查询字符串 `str` 解析为键值对的集合。

例如，查询字符串 `'foo=bar&abc=xyz&abc=123'` 被解析为：

```js
{
  foo: 'bar',
  abc: ['xyz', '123']
}
```

`querystring.parse()` 方法返回的对象不是原型继承自 JavaScript `Object`。 这意味着典型的 `Object` 方法如 `obj.toString()`、 `obj.hasOwnProperty()` 等都没有定义并且不起作用。

默认情况下，将假定查询字符串中的百分比编码字符使用 UTF-8 编码。 如果使用其他字符编码，则需要指定其他 `decodeURIComponent` 选项：

```js
// 假设 gbkDecodeURIComponent 函数已存在。

querystring.parse('w=%D6%D0%CE%C4&foo=bar', null, null,
                  { decodeURIComponent: gbkDecodeURIComponent });
```

## querystring.stringify(obj[, sep[, eq[, options]]])[#](http://nodejs.cn/api/querystring.html#querystring_querystring_stringify_obj_sep_eq_options)

[中英对照](http://nodejs.cn/api/querystring/querystring_stringify_obj_sep_eq_options.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/querystring/querystring_stringify_obj_sep_eq_options.md)

新增于: v0.1.25

- `obj`<Object>  要序列化为 URL 查询字符串的对象。
- `sep` <string>  用于在查询字符串中分隔键值对的子字符串。**默认值:** `'&'`。
- `eq` <string>  用于在查询字符串中分隔键和值的子字符串。**默认值:** `'='`。
- `options`
  - `encodeURIComponent` <Function>  在查询字符串中将 URL 不安全字符转换为百分比编码时使用的函数。**默认值:** `querystring.escape()`。

`querystring.stringify()` 方法通过迭代对象的自身属性从给定的 `obj` 生成 URL 查询字符串。

它序列化了传入 `obj` 中的以下类型的值：<string>  |<number> | [boolean] | <string[]>  | <string>  |<number[]>  | <boolean[]> 。 任何其他输入值都将被强制转换为空字符串。

```js
querystring.stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' });
// 返回 'foo=bar&baz=qux&baz=quux&corge='

querystring.stringify({ foo: 'bar', baz: 'qux' }, ';', ':');
// 返回 'foo:bar;baz:qux'
```

默认情况下，查询字符串中需要百分比编码的字符将编码为 UTF-8。 如果需要其他编码，则需要指定其他 `encodeURIComponent` 选项：

```js
// 假设 gbkEncodeURIComponent 函数已存在。

querystring.stringify({ w: '中文', foo: 'bar' }, null, null,
                      { encodeURIComponent: gbkEncodeURIComponent });
```

## querystring.unescape(str)[#](http://nodejs.cn/api/querystring.html#querystring_querystring_unescape_str)

[中英对照](http://nodejs.cn/api/querystring/querystring_unescape_str.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/querystring/querystring_unescape_str.md)

新增于: v0.1.25

- `str` <string> 

`querystring.unescape()` 方法在给定的 `str` 上执行 URL 百分比编码字符的解码。

`querystring.unescape()` 方法由 `querystring.parse()` 使用，通常不会直接使用它。 它的导出主要是为了允许应用程序代码在必要时通过将 `querystring.unescape` 分配给替代函数来提供替换的解码实现。

默认情况下， `querystring.unescape()` 方法将尝试使用 JavaScript 内置的 `decodeURIComponent()` 方法进行解码。 如果失败，将使用更安全的不会丢失格式错误的 URL 的等价方法。