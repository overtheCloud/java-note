### 什么是 Buffer，作用是什么

在引入 [`TypedArray`](http://nodejs.cn/s/oh3CkV) 之前，JavaScript 语言没有用于读取或操作二进制数据流的机制。 `Buffer` 类是作为 Node.js API 的一部分引入的，用于在 TCP 流、文件系统操作、以及其他上下文中与八位字节流进行交互。

现在可以使用 [`TypedArray`](http://nodejs.cn/s/oh3CkV)， `Buffer` 类以更优化和更适合 Node.js 的方式实现了 [`Uint8Array`](http://nodejs.cn/s/ZbDkpm) API。

`Buffer` 类的实例类似于从 `0` 到 `255` 之间的整数数组（其他整数会通过 `＆ 255` 操作强制转换到此范围），但对应于 V8 堆外部的固定大小的原始内存分配。 `Buffer` 的大小在创建时确定，且无法更改。

`Buffer` 类在全局作用域中，因此无需使用 `require('buffer').Buffer`。

```js
// 创建一个长度为 10、且用零填充的 Buffer。
const buf1 = Buffer.alloc(10);

// 创建一个长度为 10、且用 0x1 填充的 Buffer。 
const buf2 = Buffer.alloc(10, 1);

// 创建一个长度为 10、且未初始化的 Buffer。
// 这个方法比调用 Buffer.alloc() 更快，
// 但返回的 Buffer 实例可能包含旧数据，
// 因此需要使用 fill() 或 write() 重写。
const buf3 = Buffer.allocUnsafe(10);

// 创建一个包含 [0x1, 0x2, 0x3] 的 Buffer。
const buf4 = Buffer.from([1, 2, 3]);

// 创建一个包含 UTF-8 字节 [0x74, 0xc3, 0xa9, 0x73, 0x74] 的 Buffer。
const buf5 = Buffer.from('tést');

// 创建一个包含 Latin-1 字节 [0x74, 0xe9, 0x73, 0x74] 的 Buffer。
const buf6 = Buffer.from('tést', 'latin1');
```

## Buffer.from()、Buffer.alloc() 与 Buffer.allocUnsafe()[#](http://nodejs.cn/api/buffer.html#buffer_buffer_from_buffer_alloc_and_buffer_allocunsafe)

[中英对照](http://nodejs.cn/api/buffer/buffer_from_buffer_alloc_and_buffer_allocunsafe.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffer_from_buffer_alloc_and_buffer_allocunsafe.md)

在 6.0.0 之前的 Node.js 版本中， `Buffer` 实例是使用 `Buffer` 构造函数创建的，该函数根据提供的参数以不同方式分配返回的 `Buffer`：

- 将数字作为第一个参数传给 `Buffer()`（例如 `new Buffer(10)`）会分配一个指定大小的新建的 `Buffer` 对象。 在 Node.js 8.0.0 之前，为此类 `Buffer` 实例分配的内存是未初始化的，并且可能包含敏感数据。 此类 `Buffer` 实例随后必须被初始化，可以使用 [`buf.fill(0)`](http://nodejs.cn/s/2dLJk5) 或写满整个 `Buffer`。 虽然这种行为是为了提高性能，但开发经验表明，创建一个快速但未初始化的 `Buffer` 与创建一个速度更慢但更安全的 `Buffer` 之间需要有更明确的区分。 从 Node.js 8.0.0 开始， `Buffer(num)` 和 `new Buffer(num)`将返回已初始化内存的 `Buffer`。
- 传入字符串、数组、或 `Buffer` 作为第一个参数，则会将传入的对象的数据拷贝到 `Buffer` 中。
- 传入 [`ArrayBuffer`](http://nodejs.cn/s/mUbfvF) 或 [`SharedArrayBuffer`](http://nodejs.cn/s/6J6LBy)，则返回一个与给定的数组 buffer 共享已分配内存的 `Buffer`。

由于 `new Buffer()` 的行为因第一个参数的类型而异，因此当未执行参数验证或 `Buffer` 初始化时，可能会无意中将安全性和可靠性问题引入应用程序。

为了使 `Buffer` 实例的创建更可靠且更不容易出错，各种形式的 `new Buffer()` 构造函数都已被弃用，且改为单独的 `Buffer.from()`，[`Buffer.alloc()`](http://nodejs.cn/s/Du96og) 和 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 方法。

开发者应将 `new Buffer()` 构造函数的所有现有用法迁移到这些新的 API。

- [`Buffer.from(array)`](http://nodejs.cn/s/F5r61t) 返回一个新的 `Buffer`，其中包含提供的八位字节数组的副本。
- [`Buffer.from(arrayBuffer[, byteOffset [, length\]])`](http://nodejs.cn/s/jGD2qK) 返回一个新的 `Buffer`，它与给定的 [`ArrayBuffer`](http://nodejs.cn/s/mUbfvF) 共享相同的已分配内存。
- [`Buffer.from(buffer)`](http://nodejs.cn/s/SPUnUK) 返回一个新的 `Buffer`，其中包含给定 `Buffer` 的内容的副本。
- [`Buffer.from(string[, encoding\])`](http://nodejs.cn/s/X7oqVF) 返回一个新的 `Buffer`，其中包含提供的字符串的副本。
- [`Buffer.alloc(size[, fill[, encoding\]])`](http://nodejs.cn/s/Du96og) 返回一个指定大小的新建的的已初始化的 `Buffer`。 此方法比 [`Buffer.allocUnsafe(size)`](http://nodejs.cn/s/TWpeWk) 慢，但能确保新创建的 `Buffer` 实例永远不会包含可能敏感的旧数据。
- [`Buffer.allocUnsafe(size)`](http://nodejs.cn/s/TWpeWk) 和 [`Buffer.allocUnsafeSlow(size)`](http://nodejs.cn/s/PUENLw) 分别返回一个指定大小的新建的未初始化的 `Buffer`。 由于 `Buffer` 是未初始化的，因此分配的内存片段可能包含敏感的旧数据。

如果 `size` 小于或等于 [`Buffer.poolSize`](http://nodejs.cn/s/dZo4K3) 的一半，则 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 返回的 `Buffer` 实例可能是从共享的内部内存池中分配。 [`Buffer.allocUnsafeSlow()`](http://nodejs.cn/s/PUENLw) 返回的实例则从不使用共享的内部内存池。

### --zero-fill-buffers 命令行选项[#](http://nodejs.cn/api/buffer.html#buffer_the_zero_fill_buffers_command_line_option)

[中英对照](http://nodejs.cn/api/buffer/the_zero_fill_buffers_command_line_option.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/the_zero_fill_buffers_command_line_option.md)

新增于: v5.10.0

可以使用 `--zero-fill-buffers` 命令行选项启动 Node.js，以使所有新分配的 `Buffer` 实例在创建时默认使用零来填充，包括 `new Buffer(size)`、[`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 、[`Buffer.allocUnsafeSlow()`](http://nodejs.cn/s/PUENLw) 和 `new SlowBuffer(size)` 返回的 buffer。 使用这个选项可能会对性能产生重大的负面影响。 建议仅在需要强制新分配的 `Buffer` 实例不能包含可能敏感的旧数据时，才使用 `--zero-fill-buffers` 选项。

```txt
$ node --zero-fill-buffers
> Buffer.allocUnsafe(5);
<Buffer 00 00 00 00 00>
```

### Buffer.allocUnsafe() 与 Buffer.allocUnsafeSlow() 不安全的原因[#](http://nodejs.cn/api/buffer.html#buffer_what_makes_buffer_allocunsafe_and_buffer_allocunsafeslow_unsafe)

[中英对照](http://nodejs.cn/api/buffer/what_makes_buffer_allocunsafe_and_buffer_allocunsafeslow_unsafe.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/what_makes_buffer_allocunsafe_and_buffer_allocunsafeslow_unsafe.md)

当调用 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 和 [`Buffer.allocUnsafeSlow()`](http://nodejs.cn/s/PUENLw) 时，分配的内存片段是未初始化的（没有被清零）。 虽然这样的设计使得内存的分配非常快，但分配的内存可能包含敏感的旧数据。 使用由 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 创建的 `Buffer` 没有完全重写内存，当读取 `Buffer` 的内存时，可能使旧数据泄露。

虽然使用 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 有明显的性能优势，但必须格外小心，以避免将安全漏洞引入应用程序。

## Buffer 与字符编码[#](http://nodejs.cn/api/buffer.html#buffer_buffers_and_character_encodings)

[中英对照](http://nodejs.cn/api/buffer/buffers_and_character_encodings.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffers_and_character_encodings.md)

版本历史

当字符串数据被存储入 `Buffer` 实例或从 `Buffer` 实例中被提取时，可以指定一个字符编码。

```js
const buf = Buffer.from('hello world', 'ascii');

console.log(buf.toString('hex'));
// 打印: 68656c6c6f20776f726c64
console.log(buf.toString('base64'));
// 打印: aGVsbG8gd29ybGQ=

console.log(Buffer.from('fhqwhgads', 'ascii'));
// 打印: <Buffer 66 68 71 77 68 67 61 64 73>
console.log(Buffer.from('fhqwhgads', 'utf16le'));
// 打印: <Buffer 66 00 68 00 71 00 77 00 68 00 67 00 61 00 64 00 73 00>
```

Node.js 当前支持的字符编码有：

- `'ascii'` - 仅适用于 7 位 ASCII 数据。此编码速度很快，如果设置则会剥离高位。
- `'utf8'` - 多字节编码的 Unicode 字符。许多网页和其他文档格式都使用 UTF-8。
- `'utf16le'` - 2 或 4 个字节，小端序编码的 Unicode 字符。支持代理对（U+10000 至 U+10FFFF）。
- `'ucs2'` - `'utf16le'` 的别名。
- `'base64'` - Base64 编码。当从字符串创建 `Buffer` 时，此编码也会正确地接受 [RFC 4648 第 5 节](http://nodejs.cn/s/j8aS4R)中指定的 “URL 和文件名安全字母”。
- `'latin1'` - 一种将 `Buffer` 编码成单字节编码字符串的方法（由 [RFC 1345](http://nodejs.cn/s/SYWoTZ) 中的 IANA 定义，第 63 页，作为 Latin-1 的补充块和 C0/C1 控制码）。
- `'binary'` - `'latin1'` 的别名。
- `'hex'` - 将每个字节编码成两个十六进制的字符。

现代的 Web 浏览器遵循 [WHATWG 编码标准](http://nodejs.cn/s/sasgQF)，将 `'latin1'` 和 `'ISO-8859-1'` 别名为 `'win-1252'`。 这意味着当执行 `http.get()` 之类的操作时，如果返回的字符集是 WHATWG 规范中列出的字符集之一，则服务器可能实际返回 `'win-1252'` 编码的数据，而使用 `'latin1'` 编码可能错误地解码字符。

## Buffer 与 TypedArray[#](http://nodejs.cn/api/buffer.html#buffer_buffers_and_typedarray)

[中英对照](http://nodejs.cn/api/buffer/buffers_and_typedarray.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffers_and_typedarray.md)

版本历史

`Buffer` 实例也是 [`Uint8Array`](http://nodejs.cn/s/ZbDkpm) 实例，但是与 [`TypedArray`](http://nodejs.cn/s/oh3CkV) 有微小的不同。 例如，[`ArrayBuffer#slice()`](http://nodejs.cn/s/Ue6KZm) 会创建切片的拷贝，而 [`Buffer#slice()`](http://nodejs.cn/s/uQPgxt) 是在现有的 `Buffer` 上创建而不拷贝，这使得 [`Buffer#slice()`](http://nodejs.cn/s/uQPgxt) 效率更高。

也可以从一个 `Buffer` 创建新的 [`TypedArray`](http://nodejs.cn/s/oh3CkV) 实例，但需要注意以下事项：

1. `Buffer` 对象的内存是被拷贝到 [`TypedArray`](http://nodejs.cn/s/oh3CkV)，而不是共享。
2. `Buffer` 对象的内存是被解释为不同元素的数组，而不是目标类型的字节数组。 也就是说， `new Uint32Array(Buffer.from([1, 2, 3, 4]))` 会创建一个带有 4 个元素 `[1, 2, 3, 4]` 的 [`Uint32Array`](http://nodejs.cn/s/xF6oKR)，而不是带有单个元素 `[0x1020304]` 或 `[0x4030201]` 的 [`Uint32Array`](http://nodejs.cn/s/xF6oKR)。

通过使用 `TypedArray` 对象的 `.buffer` 属性，可以创建一个与 [`TypedArray`](http://nodejs.cn/s/oh3CkV) 实例共享相同内存的新 `Buffer`。

```js
const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// 拷贝 `arr` 的内容。
const buf1 = Buffer.from(arr);
// 与 `arr` 共享内存。
const buf2 = Buffer.from(arr.buffer);

console.log(buf1);
// 打印: <Buffer 88 a0>
console.log(buf2);
// 打印: <Buffer 88 13 a0 0f>

arr[1] = 6000;

console.log(buf1);
// 打印: <Buffer 88 a0>
console.log(buf2);
// 打印: <Buffer 88 13 70 17>
```

当使用 [`TypedArray`](http://nodejs.cn/s/oh3CkV) 的 `.buffer` 创建 `Buffer` 时，也可以通过传入 `byteOffset` 和 `length` 参数只使用 [`TypedArray`](http://nodejs.cn/s/oh3CkV) 的一部分。

```js
const arr = new Uint16Array(20);
const buf = Buffer.from(arr.buffer, 0, 16);

console.log(buf.length);
// 打印: 16
```

`Buffer.from()` 与 [`TypedArray.from()`](http://nodejs.cn/s/jLHsN8) 有着不同的实现。 具体来说，[`TypedArray`](http://nodejs.cn/s/oh3CkV) 可以接受第二个参数作为映射函数，在类型数组的每个元素上调用：

- `TypedArray.from(source[, mapFn[, thisArg]])`

`Buffer.from()` 则不支持映射函数的使用：

- [`Buffer.from(array)`](http://nodejs.cn/s/F5r61t)
- [`Buffer.from(buffer)`](http://nodejs.cn/s/SPUnUK)
- [`Buffer.from(arrayBuffer[, byteOffset[, length\]])`](http://nodejs.cn/s/jGD2qK)
- [`Buffer.from(string[, encoding\])`](http://nodejs.cn/s/X7oqVF)

## Buffer 与迭代器[#](http://nodejs.cn/api/buffer.html#buffer_buffers_and_iteration)

[中英对照](http://nodejs.cn/api/buffer/buffers_and_iteration.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffers_and_iteration.md)

`Buffer` 实例可以使用 `for..of` 语法进行迭代：

```js
const buf = Buffer.from([1, 2, 3]);

for (const b of buf) {
  console.log(b);
}
// 打印:
//   1
//   2
//   3
```

此外，[`buf.values()`](http://nodejs.cn/s/DPvcum)、[`buf.keys()`](http://nodejs.cn/s/fb89Qi)、和 [`buf.entries()`](http://nodejs.cn/s/E1Z524) 方法也可用于创建迭代器。

## Buffer 类[#](http://nodejs.cn/api/buffer.html#buffer_class_buffer)

[中英对照](http://nodejs.cn/api/buffer/class_buffer.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_buffer.md)

`Buffer` 类是一个全局变量，用于直接处理二进制数据。 它可以使用多种方式构建。

### new Buffer(array)[#](http://nodejs.cn/api/buffer.html#buffer_new_buffer_array)

暂无中英对照

版本历史



[稳定性: 0](http://nodejs.cn/api/documentation.html#documentation_stability_index) - 废弃: 改为使用 [`Buffer.from(array)`](http://nodejs.cn/s/F5r61t) 。



- `array` [](http://nodejs.cn/s/SXbo1v) An array of bytes to copy from.

Allocates a new `Buffer` using an `array` of octets.

```js
// Creates a new Buffer containing the UTF-8 bytes of the string 'buffer'.
const buf = new Buffer([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);
```

### new Buffer(arrayBuffer[, byteOffset[, length]])[#](http://nodejs.cn/api/buffer.html#buffer_new_buffer_arraybuffer_byteoffset_length)

暂无中英对照

版本历史



[稳定性: 0](http://nodejs.cn/api/documentation.html#documentation_stability_index) - 废弃: 改为使用 [`Buffer.from(arrayBuffer[, byteOffset[, length\]])`](http://nodejs.cn/s/jGD2qK) 。



- `arrayBuffer` [](http://nodejs.cn/s/mUbfvF) | [](http://nodejs.cn/s/6J6LBy) An [`ArrayBuffer`](http://nodejs.cn/s/mUbfvF), [`SharedArrayBuffer`](http://nodejs.cn/s/6J6LBy) or the `.buffer` property of a [`TypedArray`](http://nodejs.cn/s/oh3CkV).
- `byteOffset` [](http://nodejs.cn/s/SXbo1v) Index of first byte to expose. **Default:** `0`.
- `length` [](http://nodejs.cn/s/SXbo1v) Number of bytes to expose. **Default:** `arrayBuffer.length - byteOffset`.

This creates a view of the [`ArrayBuffer`](http://nodejs.cn/s/mUbfvF) or [`SharedArrayBuffer`](http://nodejs.cn/s/6J6LBy) without copying the underlying memory. For example, when passed a reference to the `.buffer`property of a [`TypedArray`](http://nodejs.cn/s/oh3CkV) instance, the newly created `Buffer` will share the same allocated memory as the [`TypedArray`](http://nodejs.cn/s/oh3CkV).

The optional `byteOffset` and `length` arguments specify a memory range within the `arrayBuffer` that will be shared by the `Buffer`.

```js
const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Shares memory with `arr`.
const buf = new Buffer(arr.buffer);

console.log(buf);
// Prints: <Buffer 88 13 a0 0f>

// Changing the original Uint16Array changes the Buffer also.
arr[1] = 6000;

console.log(buf);
// Prints: <Buffer 88 13 70 17>
```

### new Buffer(buffer)[#](http://nodejs.cn/api/buffer.html#buffer_new_buffer_buffer)

暂无中英对照

版本历史



[稳定性: 0](http://nodejs.cn/api/documentation.html#documentation_stability_index) - 废弃: 改为使用 [`Buffer.from(buffer)`](http://nodejs.cn/s/SPUnUK) 。



- `buffer` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) An existing `Buffer` or [`Uint8Array`](http://nodejs.cn/s/ZbDkpm) from which to copy data.

Copies the passed `buffer` data onto a new `Buffer` instance.

```js
const buf1 = new Buffer('buffer');
const buf2 = new Buffer(buf1);

buf1[0] = 0x61;

console.log(buf1.toString());
// Prints: auffer
console.log(buf2.toString());
// Prints: buffer
```

### new Buffer(size)[#](http://nodejs.cn/api/buffer.html#buffer_new_buffer_size)

暂无中英对照

版本历史



[稳定性: 0](http://nodejs.cn/api/documentation.html#documentation_stability_index) - 废弃: 改为使用 [`Buffer.alloc()`](http://nodejs.cn/s/Du96og) (或 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk))。



- `size` [](http://nodejs.cn/s/SXbo1v) The desired length of the new `Buffer`.

Allocates a new `Buffer` of `size` bytes. If `size` is larger than [`buffer.constants.MAX_LENGTH`](http://nodejs.cn/s/aBiAe5) or smaller than 0, [`ERR_INVALID_OPT_VALUE`](http://nodejs.cn/s/ouMFyk) is thrown. A zero-length `Buffer` is created if `size` is 0.

Prior to Node.js 8.0.0, the underlying memory for `Buffer` instances created in this way is *not initialized*. The contents of a newly created `Buffer` are unknown and *may contain sensitive data*. Use [`Buffer.alloc(size)`](http://nodejs.cn/s/Du96og) instead to initialize a `Buffer` with zeroes.

```js
const buf = new Buffer(10);

console.log(buf);
// Prints: <Buffer 00 00 00 00 00 00 00 00 00 00>
```

### new Buffer(string[, encoding])[#](http://nodejs.cn/api/buffer.html#buffer_new_buffer_string_encoding)

暂无中英对照

版本历史



[稳定性: 0](http://nodejs.cn/api/documentation.html#documentation_stability_index) - 废弃: 改为使用 [`Buffer.from(string[, encoding\])`](http://nodejs.cn/s/X7oqVF) 。



- `string` [](http://nodejs.cn/s/9Tw2bK) String to encode.
- `encoding` [](http://nodejs.cn/s/9Tw2bK) The encoding of `string`. **Default:** `'utf8'`.

Creates a new `Buffer` containing `string`. The `encoding` parameter identifies the character encoding of `string`.

```js
const buf1 = new Buffer('this is a tést');
const buf2 = new Buffer('7468697320697320612074c3a97374', 'hex');

console.log(buf1.toString());
// Prints: this is a tést
console.log(buf2.toString());
// Prints: this is a tést
console.log(buf1.toString('ascii'));
// Prints: this is a tC)st
```

### Buffer.alloc(size[, fill[, encoding]])[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_alloc_size_fill_encoding)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_alloc_size_fill_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_alloc_size_fill_encoding.md)

版本历史

- `size` [](http://nodejs.cn/s/SXbo1v) 新 `Buffer` 的所需长度。
- `fill` [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) | [](http://nodejs.cn/s/SXbo1v) 用于预填充新 `Buffer` 的值。**默认值:** `0`。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `fill` 是一个字符串，则这是它的字符编码。**默认值:** `'utf8'`。

分配一个大小为 `size` 字节的新 `Buffer`。 如果 `fill` 为 `undefined`，则用零填充 `Buffer`。

```js
const buf = Buffer.alloc(5);

console.log(buf);
// 打印: <Buffer 00 00 00 00 00>
```

如果 `size` 大于 [`buffer.constants.MAX_LENGTH`](http://nodejs.cn/s/aBiAe5) 或小于 0，则抛出 [`ERR_INVALID_OPT_VALUE`](http://nodejs.cn/s/ouMFyk)。 如果 `size` 为 0，则创建一个零长度的 `Buffer`。

如果指定了 `fill`，则分配的 `Buffer` 通过调用 [`buf.fill(fill)`](http://nodejs.cn/s/2dLJk5) 进行初始化。

```js
const buf = Buffer.alloc(5, 'a');

console.log(buf);
// 打印: <Buffer 61 61 61 61 61>
```

如果同时指定了 `fill` 和 `encoding`，则分配的 `Buffer` 通过调用 [`buf.fill(fill, encoding)`](http://nodejs.cn/s/2dLJk5) 进行初始化 。

```js
const buf = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64');

console.log(buf);
// 打印: <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
```

调用 [`Buffer.alloc()`](http://nodejs.cn/s/Du96og) 可能比替代的 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 慢得多，但能确保新创建的 `Buffer` 实例的内容永远不会包含敏感的数据。

如果 `size` 不是一个数字，则抛出 `TypeError`。

### Buffer.allocUnsafe(size)[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_allocunsafe_size)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_allocunsafe_size.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_allocunsafe_size.md)

版本历史

- `size` [](http://nodejs.cn/s/SXbo1v) 新建的 `Buffer` 的长度。

创建一个大小为 `size` 字节的新 `Buffer`。 如果 `size` 大于 [`buffer.constants.MAX_LENGTH`](http://nodejs.cn/s/aBiAe5) 或小于 0，则抛出 [`ERR_INVALID_OPT_VALUE`](http://nodejs.cn/s/ouMFyk)。 如果 `size` 为 0，则创建一个长度为零的 `Buffer`。

以这种方式创建的 `Buffer` 实例的底层内存是未初始化的。 新创建的 `Buffer` 的内容是未知的，可能包含敏感数据。 使用 [`Buffer.alloc()`](http://nodejs.cn/s/Du96og) 可以创建以零初始化的 `Buffer` 实例。

```js
const buf = Buffer.allocUnsafe(10);

console.log(buf);
// 打印（内容可能有所不同）: <Buffer a0 8b 28 3f 01 00 00 00 50 32>

buf.fill(0);

console.log(buf);
// 打印: <Buffer 00 00 00 00 00 00 00 00 00 00>
```

如果 `size` 不是一个数字，则抛出 `TypeError`。

`Buffer` 模块会预分配一个内部的大小为 [`Buffer.poolSize`](http://nodejs.cn/s/dZo4K3) 的 `Buffer` 实例，作为快速分配的内存池，用于使用 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 创建新的 `Buffer` 实例、或废弃的 `new Buffer(size)` 构造器但仅当 `size` 小于或等于 `Buffer.poolSize >> 1`（[`Buffer.poolSize`](http://nodejs.cn/s/dZo4K3) 除以二再向下取整）。

对这个预分配的内部内存池的使用是调用 `Buffer.alloc(size, fill)` 和 `Buffer.allocUnsafe(size).fill(fill)` 两者之间的关键区别。 具体来说， `Buffer.alloc(size, fill)` 永远不会使用内部的 `Buffer` 池，而 `Buffer.allocUnsafe(size).fill(fill)` 在 `size` 小于或等于 [`Buffer.poolSize`](http://nodejs.cn/s/dZo4K3) 的一半时将会使用内部的 `Buffer`池。 该差异虽然很微妙，但当应用程序需要 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 提供的额外性能时，则非常重要。

### Buffer.allocUnsafeSlow(size)[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_allocunsafeslow_size)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_allocunsafeslow_size.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_allocunsafeslow_size.md)

新增于: v5.12.0

- `size` [](http://nodejs.cn/s/SXbo1v) 新建的 `Buffer` 的长度。

创建一个大小为 `size` 字节的新 `Buffer`。 如果 `size` 大于 [`buffer.constants.MAX_LENGTH`](http://nodejs.cn/s/aBiAe5) 或小于 0，则抛出 [`ERR_INVALID_OPT_VALUE`](http://nodejs.cn/s/ouMFyk)。 如果 `size` 为 0，则创建一个长度为零的 `Buffer`。

以这种方式创建的 `Buffer` 实例的底层内存是未初始化的。 `Buffer` 的内容是未知的，可能包含敏感数据。 使用 [`buf.fill(0)`](http://nodejs.cn/s/2dLJk5) 可以以零初始化 `Buffer` 实例。

当使用 [`Buffer.allocUnsafe()`](http://nodejs.cn/s/TWpeWk) 创建新的 `Buffer` 实例时，如果要分配的内存小于 4KB，则会从一个预分配的 `Buffer` 切割出来。 这可以避免垃圾回收机制因创建太多独立的 `Buffer` 而过度使用。 这种方式通过消除跟踪和清理的需要来改进性能和内存使用。

当开发人员需要在内存池中保留一小块内存时，可以使用 `Buffer.allocUnsafeSlow()` 创建一个非内存池的 `Buffer` 实例并拷贝相关的比特位出来。

```js
// 需要保留一小块内存。
const store = [];

socket.on('readable', () => {
  let data;
  while (null !== (data = readable.read())) {
    // 为剩下的数据分配内存。
    const sb = Buffer.allocUnsafeSlow(10);

    // 拷贝数据到新分配的内存。
    data.copy(sb, 0, 0, 10);

    store.push(sb);
  }
});
```

`Buffer.allocUnsafeSlow()` 应该只用作开发人员已经在其应用程序中观察到过度的内存之后的最后手段。

如果 `size` 不是一个数字，则抛出 `TypeError`。

### Buffer.byteLength(string[, encoding])[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_bytelength_string_encoding)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_bytelength_string_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_bytelength_string_encoding.md)

版本历史

- `string` [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/oh3CkV) | [](http://nodejs.cn/s/yCdVkD) | [](http://nodejs.cn/s/mUbfvF) | [](http://nodejs.cn/s/6J6LBy) 要计算长度的值。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `string` 是字符串，则这是它的字符编码。**默认值:** `'utf8'`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `string` 中包含的字节数。

返回字符串的实际字节长度。 与 [`String.prototype.length`](http://nodejs.cn/s/TmnY1C) 不同，后者返回字符串的字符数。

对于 `'base64'` 和 `'hex'`，此函数会假定输入是有效的。 对于包含非 Base64/Hex 编码的数据（例如空格）的字符串，返回值可能是大于从字符串创建的 `Buffer` 的长度。

```js
const str = '\u00bd + \u00bc = \u00be';

console.log(`${str}: ${str.length} 个字符, ` +
            `${Buffer.byteLength(str, 'utf8')} 个字节`);
// 打印: ½ + ¼ = ¾: 9 个字符, 12 个字节
```

当 `string` 是一个 `Buffer`/[`DataView`](http://nodejs.cn/s/yCdVkD)/[`TypedArray`](http://nodejs.cn/s/oh3CkV)/[`ArrayBuffer`](http://nodejs.cn/s/mUbfvF)/[`SharedArrayBuffer`](http://nodejs.cn/s/6J6LBy) 时，返回实际的字节长度。

### Buffer.compare(buf1, buf2)[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_compare_buf1_buf2)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_compare_buf1_buf2.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_compare_buf1_buf2.md)

版本历史

- `buf1` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm)
- `buf2` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm)
- 返回: [](http://nodejs.cn/s/SXbo1v)

比较 `buf1` 与 `buf2`，主要用于 `Buffer` 实例数组的排序。 相当于调用 [`buf1.compare(buf2)`](http://nodejs.cn/s/3wddT3)。

```js
const buf1 = Buffer.from('1234');
const buf2 = Buffer.from('0123');
const arr = [buf1, buf2];

console.log(arr.sort(Buffer.compare));
// 打印: [ <Buffer 30 31 32 33>, <Buffer 31 32 33 34> ]
// (结果相当于: [buf2, buf1])
```

### Buffer.concat(list[, totalLength])[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_concat_list_totallength)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_concat_list_totallength.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_concat_list_totallength.md)

版本历史

- `list` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) 要合并的 `Buffer` 数组或 [`Uint8Array`](http://nodejs.cn/s/ZbDkpm) 数组。
- `totalLength` [](http://nodejs.cn/s/SXbo1v) 合并后 `list` 中的 `Buffer` 实例的总长度。
- 返回: [](http://nodejs.cn/s/6x1hD3)

返回一个合并了 `list` 中所有 `Buffer` 实例的新 `Buffer`。

如果 `list` 中没有元素、或 `totalLength` 为 0，则返回一个长度为 0 的 `Buffer`。

如果没有提供 `totalLength`，则计算 `list` 中的 `Buffer` 实例的总长度。 但是这会导致执行额外的循环用于计算 `totalLength`，因此如果已知长度，则明确提供长度会更快。

如果提供了 `totalLength`，则会强制转换为无符号整数。 如果 `list` 中的 `Buffer` 合并后的总长度大于 `totalLength`，则结果会被截断到 `totalLength` 的长度。

```js
// 用含有三个 `Buffer` 实例的数组创建一个单一的 `Buffer`。

const buf1 = Buffer.alloc(10);
const buf2 = Buffer.alloc(14);
const buf3 = Buffer.alloc(18);
const totalLength = buf1.length + buf2.length + buf3.length;

console.log(totalLength);
// 打印: 42

const bufA = Buffer.concat([buf1, buf2, buf3], totalLength);

console.log(bufA);
// 打印: <Buffer 00 00 00 00 ...>
console.log(bufA.length);
// 打印: 42
```

### Buffer.from(array)[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_array)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_from_array.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_from_array.md)

新增于: v5.10.0

- `array` [](http://nodejs.cn/s/SXbo1v)

使用八位字节数组 `array` 分配一个新的 `Buffer`。

```js
// 创建一个包含字符串 'buffer' 的 UTF-8 字节的新 Buffer。
const buf = Buffer.from([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);
```

如果 `array` 不是一个 `Array` 或适用于 `Buffer.from()` 变量的其他类型，则抛出 `TypeError`。

### Buffer.from(arrayBuffer[, byteOffset[, length]])[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_arraybuffer_byteoffset_length)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_from_arraybuffer_byteoffset_length.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_from_arraybuffer_byteoffset_length.md)

新增于: v5.10.0

- `arrayBuffer` [](http://nodejs.cn/s/mUbfvF) | [](http://nodejs.cn/s/6J6LBy) 一个 [`ArrayBuffer`](http://nodejs.cn/s/mUbfvF)、[`SharedArrayBuffer`](http://nodejs.cn/s/6J6LBy)、或 [`TypedArray`](http://nodejs.cn/s/oh3CkV) 的 `.buffer` 属性。
- `byteOffset` [](http://nodejs.cn/s/SXbo1v) 开始拷贝的索引。**默认值:** `0`。
- `length` [](http://nodejs.cn/s/SXbo1v) 拷贝的字节数。**默认值:** `arrayBuffer.length - byteOffset`。

创建 [`ArrayBuffer`](http://nodejs.cn/s/mUbfvF) 的视图，但不会拷贝底层内存。 例如，当传入 [`TypedArray`](http://nodejs.cn/s/oh3CkV) 的 `.buffer` 属性的引用时，新建的 `Buffer` 会与 [`TypedArray`](http://nodejs.cn/s/oh3CkV) 共享同一内存。

```js
const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// 与 `arr` 共享内存。
const buf = Buffer.from(arr.buffer);

console.log(buf);
// 打印: <Buffer 88 13 a0 0f>

// 改变原先的 Uint16Array 也会改变 Buffer。
arr[1] = 6000;

console.log(buf);
// 打印: <Buffer 88 13 70 17>
```

可选的 `byteOffset` 和 `length` 参数指定 `arrayBuffer` 中与 `Buffer` 共享的内存范围。

```js
const ab = new ArrayBuffer(10);
const buf = Buffer.from(ab, 0, 2);

console.log(buf.length);
// 打印: 2
```

如果 `arrayBuffer` 不是一个 [`ArrayBuffer`](http://nodejs.cn/s/mUbfvF)、[`SharedArrayBuffer`](http://nodejs.cn/s/6J6LBy) 或适用于 `Buffer.from()` 变量的其他类型，则抛出 `TypeError`。

### Buffer.from(buffer)[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_buffer)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_from_buffer.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_from_buffer.md)

新增于: v5.10.0

- `buffer` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) 要拷贝数据的 `Buffer` 或 [`Uint8Array`](http://nodejs.cn/s/ZbDkpm)。

拷贝 `buffer` 的数据到新建的 `Buffer` 实例。

```js
const buf1 = Buffer.from('buffer');
const buf2 = Buffer.from(buf1);

buf1[0] = 0x61;

console.log(buf1.toString());
// 打印: auffer
console.log(buf2.toString());
// 打印: buffer
```

如果 `buffer` 不是一个 `Buffer` 或适用于 `Buffer.from()` 变量的其他类型，则抛出 `TypeError`。

### Buffer.from(object[, offsetOrEncoding[, length]])[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_object_offsetorencoding_length)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_from_object_offsetorencoding_length.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_from_object_offsetorencoding_length.md)

新增于: v8.2.0

- `object` [](http://nodejs.cn/s/jzn6Ao) 支持 `Symbol.toPrimitive` 或 `valueOf()` 的对象。
- `offsetOrEncoding` [](http://nodejs.cn/s/SXbo1v) | [](http://nodejs.cn/s/9Tw2bK) 字节偏移量或字符编码，取决于 `object.valueOf()` 或 `object[Symbol.toPrimitive]()` 返回的值。
- `length` [](http://nodejs.cn/s/SXbo1v) 长度，取决于 `object.valueOf()` 或 `object[Symbol.toPrimitive]()` 的返回值。

对于 `valueOf()` 返回值不严格等于 `object` 的对象，返回 `Buffer.from(object.valueOf(), offsetOrEncoding, length)`。

```js
const buf = Buffer.from(new String('this is a test'));
// 打印: <Buffer 74 68 69 73 20 69 73 20 61 20 74 65 73 74>
```

对于支持 `Symbol.toPrimitive` 的对象，会返回 `Buffer.from(object[Symbol.toPrimitive](), offsetOrEncoding, length)`。

```js
class Foo {
  [Symbol.toPrimitive]() {
    return 'this is a test';
  }
}

const buf = Buffer.from(new Foo(), 'utf8');
// 打印: <Buffer 74 68 69 73 20 69 73 20 61 20 74 65 73 74>
```

如果 `object` 没有提及的方法、或适用于 `Buffer.from()` 变量的其他类型，则抛出 `TypeError`。

### Buffer.from(string[, encoding])[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_from_string_encoding)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_from_string_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_from_string_encoding.md)

新增于: v5.10.0

- `string` [](http://nodejs.cn/s/9Tw2bK) 要编码的字符串。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) `string` 的字符编码。**默认值:** `'utf8'`。

创建一个包含 `string` 的新 `Buffer`。 `encoding` 参数指定 `string` 的字符编码。

```js
const buf1 = Buffer.from('this is a tést');
const buf2 = Buffer.from('7468697320697320612074c3a97374', 'hex');

console.log(buf1.toString());
// 打印: this is a tést
console.log(buf2.toString());
// 打印: this is a tést
console.log(buf1.toString('ascii'));
// 打印: this is a tC)st
```

如果 `string` 不是一个字符串或适用于 `Buffer.from()` 变量的其他类型，则抛出 `TypeError`。

### Buffer.isBuffer(obj)[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_isbuffer_obj)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_isbuffer_obj.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_isbuffer_obj.md)

新增于: v0.1.101

- `obj` [](http://nodejs.cn/s/jzn6Ao)
- 返回: [](http://nodejs.cn/s/jFbvuT)

如果 `obj` 是一个 `Buffer`，则返回 `true`，否则返回 `false`。

### Buffer.isEncoding(encoding)[#](http://nodejs.cn/api/buffer.html#buffer_class_method_buffer_isencoding_encoding)

[中英对照](http://nodejs.cn/api/buffer/class_method_buffer_isencoding_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_method_buffer_isencoding_encoding.md)

新增于: v0.9.1

- `encoding` [](http://nodejs.cn/s/9Tw2bK) 要检查的字符编码名称。
- 返回: [](http://nodejs.cn/s/jFbvuT)

如果 `encoding` 是支持的字符编码，则返回 `true`，否则返回 `false`。

```js
console.log(Buffer.isEncoding('utf-8'));
// 打印: true

console.log(Buffer.isEncoding('hex'));
// 打印: true

console.log(Buffer.isEncoding('utf/8'));
// 打印: false

console.log(Buffer.isEncoding(''));
// 打印: false
```

### Buffer.poolSize[#](http://nodejs.cn/api/buffer.html#buffer_class_property_buffer_poolsize)

[中英对照](http://nodejs.cn/api/buffer/class_property_buffer_poolsize.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/class_property_buffer_poolsize.md)

新增于: v0.11.3

- [](http://nodejs.cn/s/SXbo1v) **默认值:** `8192`。

这是用于缓冲池的预分配的内部 `Buffer` 实例的大小（以字节为单位）。 该值可以修改。

### buf[index][#](http://nodejs.cn/api/buffer.html#buffer_buf_index)

[中英对照](http://nodejs.cn/api/buffer/buf_index.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_index.md)



- `index` [](http://nodejs.cn/s/SXbo1v)

索引操作符 `[index]` 可用于获取或设置 `buf` 中指定的 `index` 位置的字节。 该值指向单个字节，所以有效的值的范围是 `0x00` 至 `0xFF`（十六进制），或 `0` 至 `255`（十进制）。

该操作符继承自 `Uint8Array`，所以对越界访问的处理与 `UInt8Array` 相同（也就是，取值时返回 `undefined`，赋值时不作为）。

```js
// 拷贝 ASCII 字符串到 `Buffer`，每次拷贝一个字节。

const str = 'http://nodejs.cn/';
const buf = Buffer.allocUnsafe(str.length);

for (let i = 0; i < str.length; i++) {
  buf[i] = str.charCodeAt(i);
}

console.log(buf.toString('ascii'));
// 打印: http://nodejs.cn/
```

### buf.buffer[#](http://nodejs.cn/api/buffer.html#buffer_buf_buffer)

[中英对照](http://nodejs.cn/api/buffer/buf_buffer.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_buffer.md)

- [](http://nodejs.cn/s/mUbfvF) 创建 `Buffer` 对象时基于的底层 `ArrayBuffer` 对象。

```js
const arrayBuffer = new ArrayBuffer(16);
const buffer = Buffer.from(arrayBuffer);

console.log(buffer.buffer === arrayBuffer);
// 打印: true
```

### buf.byteOffset[#](http://nodejs.cn/api/buffer.html#buffer_buf_byteoffset)

[中英对照](http://nodejs.cn/api/buffer/buf_byteoffset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_byteoffset.md)

- [](http://nodejs.cn/s/SXbo1v) 创建 `Buffer` 对象时基于的底层 `ArrayBuffer` 对象的 `byteOffset`。

当 `Buffer.from(ArrayBuffer, byteOffset, length)` 设置了 `byteOffset` 或创建一个小于 `Buffer.poolSize` 的 buffer 时，底层的 `ArrayBuffer` 的偏移量并不是从 0 开始。

当直接使用 `buf.buffer` 访问底层的 `ArrayBuffer` 时， `ArrayBuffer` 的第一个字节可能并不指向 `buf` 对象。

所有使用 `Buffer` 对象创建 `TypedArray` 对象时，需要正确地指定 `byteOffset`。

```js
// 创建一个小于 `Buffer.poolSize` 的 buffer。
const nodeBuffer = new Buffer.from([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

// 当将 Node.js Buffer 赋值给一个 Int8 的 TypedArray 时，记得使用 byteOffset。
new Int8Array(nodeBuffer.buffer, nodeBuffer.byteOffset, nodeBuffer.length);
```

### buf.compare(target[, targetStart[, targetEnd[, sourceStart[, sourceEnd]]]])[#](http://nodejs.cn/api/buffer.html#buffer_buf_compare_target_targetstart_targetend_sourcestart_sourceend)

[中英对照](http://nodejs.cn/api/buffer/buf_compare_target_targetstart_targetend_sourcestart_sourceend.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_compare_target_targetstart_targetend_sourcestart_sourceend.md)

版本历史

- `target` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) 要与 `buf` 对比的 `Buffer` 或 [`Uint8Array`](http://nodejs.cn/s/ZbDkpm)。
- `targetStart` [](http://nodejs.cn/s/SXbo1v) `target` 中开始对比的偏移量。**默认值:** `0`。
- `targetEnd` [](http://nodejs.cn/s/SXbo1v) `target` 中结束对比的偏移量（不包含）。**默认值:** `target.length`。
- `sourceStart` [](http://nodejs.cn/s/SXbo1v) `buf` 中开始对比的偏移量。**默认值:** `0`。
- `sourceEnd` [](http://nodejs.cn/s/SXbo1v) `buf` 中结束对比的偏移量（不包含）。**默认值:** [`buf.length`](http://nodejs.cn/s/hn6FjL)。
- 返回: [](http://nodejs.cn/s/SXbo1v)

对比 `buf` 与 `target`，并返回一个数值，表明 `buf` 在排序上是否排在 `target` 前面、或后面、或相同。 对比是基于各自 `Buffer` 实际的字节序列。

- 如果 `target` 与 `buf` 相同，则返回 `0`。
- 如果 `target` 排在 `buf` 前面，则返回 `1`。
- 如果 `target` 排在 `buf` 后面，则返回 `-1`。

```js
const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('BCD');
const buf3 = Buffer.from('ABCD');

console.log(buf1.compare(buf1));
// 打印: 0
console.log(buf1.compare(buf2));
// 打印: -1
console.log(buf1.compare(buf3));
// 打印: -1
console.log(buf2.compare(buf1));
// 打印: 1
console.log(buf2.compare(buf3));
// 打印: 1
console.log([buf1, buf2, buf3].sort(Buffer.compare));
// 打印: [ <Buffer 41 42 43>, <Buffer 41 42 43 44>, <Buffer 42 43 44> ]
// (相当于: [buf1, buf3, buf2])
```

`targetStart`、 `targetEnd`、 `sourceStart` 与 `sourceEnd` 可用于指定 `target` 与 `buf` 中对比的范围。

```js
const buf1 = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);
const buf2 = Buffer.from([5, 6, 7, 8, 9, 1, 2, 3, 4]);

console.log(buf1.compare(buf2, 5, 9, 0, 4));
// 打印: 0
console.log(buf1.compare(buf2, 0, 6, 4));
// 打印: -1
console.log(buf1.compare(buf2, 5, 6, 5));
// 打印: 1
```

如果 `targetStart < 0`、 `sourceStart < 0`、 `targetEnd > target.byteLength` 或 `sourceEnd > source.byteLength`，则抛出 [`ERR_OUT_OF_RANGE`](http://nodejs.cn/s/tsvMjv)。

### buf.copy(target[, targetStart[, sourceStart[, sourceEnd]]])[#](http://nodejs.cn/api/buffer.html#buffer_buf_copy_target_targetstart_sourcestart_sourceend)

[中英对照](http://nodejs.cn/api/buffer/buf_copy_target_targetstart_sourcestart_sourceend.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_copy_target_targetstart_sourcestart_sourceend.md)

新增于: v0.1.90

- `target` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) 要拷贝进的 `Buffer` 或 [`Uint8Array`](http://nodejs.cn/s/ZbDkpm)。
- `targetStart` [](http://nodejs.cn/s/SXbo1v) `target` 中开始写入之前要跳过的字节数。**默认值:** `0`。
- `sourceStart` [](http://nodejs.cn/s/SXbo1v) `buf` 中开始拷贝的偏移量。**默认值:** `0`。
- `sourceEnd` [](http://nodejs.cn/s/SXbo1v) `buf` 中结束拷贝的偏移量（不包含）。**默认值:** [`buf.length`](http://nodejs.cn/s/hn6FjL)。
- 返回: [](http://nodejs.cn/s/SXbo1v) 拷贝的字节数。

拷贝 `buf` 中某个区域的数据到 `target` 中的某个区域，即使 `target` 的内存区域与 `buf` 的重叠。

```js
// 创建两个 `Buffer` 实例。
const buf1 = Buffer.allocUnsafe(26);
const buf2 = Buffer.allocUnsafe(26).fill('!');

for (let i = 0; i < 26; i++) {
  // 97 是 'a' 的十进制 ASCII 值。
  buf1[i] = i + 97;
}

// 拷贝 `buf1` 中第 16 至 19 字节偏移量的数据到 `buf2` 第 8 字节偏移量开始。
buf1.copy(buf2, 8, 16, 20);

console.log(buf2.toString('ascii', 0, 25));
// 打印: !!!!!!!!qrst!!!!!!!!!!!!!
// 创建一个 `Buffer`，并拷贝同一 `Buffer` 中一个区域的数据到另一个重叠的区域。

const buf = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 是 'a' 的十进制 ASCII 值。
  buf[i] = i + 97;
}

buf.copy(buf, 0, 4, 10);

console.log(buf.toString());
// 打印: efghijghijklmnopqrstuvwxyz
```

### buf.entries()[#](http://nodejs.cn/api/buffer.html#buffer_buf_entries)

[中英对照](http://nodejs.cn/api/buffer/buf_entries.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_entries.md)

新增于: v1.1.0

- 返回: [](http://nodejs.cn/s/Y2SE1q)

用 `buf` 的内容创建并返回一个 `[index, byte]` 形式的[迭代器](http://nodejs.cn/s/KK7Xfc)。

```js
// 记录 `Buffer` 的全部内容。

const buf = Buffer.from('buffer');

for (const pair of buf.entries()) {
  console.log(pair);
}
// 打印:
//   [0, 98]
//   [1, 117]
//   [2, 102]
//   [3, 102]
//   [4, 101]
//   [5, 114]
```

### buf.equals(otherBuffer)[#](http://nodejs.cn/api/buffer.html#buffer_buf_equals_otherbuffer)

[中英对照](http://nodejs.cn/api/buffer/buf_equals_otherbuffer.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_equals_otherbuffer.md)

版本历史

- `otherBuffer` [](http://nodejs.cn/s/6x1hD3) 要与 `bur` 对比的 `Buffer` 或 [`Uint8Array`](http://nodejs.cn/s/ZbDkpm)。
- 返回: [](http://nodejs.cn/s/jFbvuT)

如果 `buf` 与 `otherBuffer` 具有完全相同的字节，则返回 `true`，否则返回 `false`。

```js
const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('414243', 'hex');
const buf3 = Buffer.from('ABCD');

console.log(buf1.equals(buf2));
// 打印: true
console.log(buf1.equals(buf3));
// 打印: false
```

### buf.fill(value[, offset[, end]][, encoding])[#](http://nodejs.cn/api/buffer.html#buffer_buf_fill_value_offset_end_encoding)

[中英对照](http://nodejs.cn/api/buffer/buf_fill_value_offset_end_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_fill_value_offset_end_encoding.md)

版本历史

- `value` [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) | [](http://nodejs.cn/s/SXbo1v) 用来填充 `buf` 的值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始填充 `buf` 的偏移量。**默认值:** `0`。
- `end` [](http://nodejs.cn/s/SXbo1v) 结束填充 `buf` 的偏移量（不包含）。**默认值:** [`buf.length`](http://nodejs.cn/s/hn6FjL)。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `value` 是字符串，则指定 `value` 的字符编码。**默认值:** `'utf8'`。
- 返回: [](http://nodejs.cn/s/6x1hD3) `buf` 的引用。

用指定的 `value` 填充 `buf`。 如果没有指定 `offset` 与 `end`，则填充整个 `buf`：

```js
// 用 ASCII 字符 'h' 填充 `Buffer`。

const b = Buffer.allocUnsafe(50).fill('h');

console.log(b.toString());
// 打印: hhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh
```

如果 `value` 不是字符串、 `Buffer`、或整数，则会被转换为 `uint32` 值。 如果得到的整数大于 `255`（十进制），则 `buf` 将会使用 `value & 255` 填充。

如果 `fill()` 最后写入的是一个多字节字符，则只写入适合 `buf` 的字节：

```js
// 用一个双字节字符填充 `Buffer`。

console.log(Buffer.allocUnsafe(3).fill('\u0222'));
// 打印: <Buffer c8 a2 c8>
```

如果 `value` 包含无效的字符，则截掉无效的字符。 如果截掉后没有数据，则不填充：

```js
const buf = Buffer.allocUnsafe(5);

console.log(buf.fill('a'));
// 打印: <Buffer 61 61 61 61 61>
console.log(buf.fill('aazz', 'hex'));
// 打印: <Buffer aa aa aa aa aa>
console.log(buf.fill('zz', 'hex'));
// 抛出异常。
```

### buf.includes(value[, byteOffset][, encoding])[#](http://nodejs.cn/api/buffer.html#buffer_buf_includes_value_byteoffset_encoding)

[中英对照](http://nodejs.cn/api/buffer/buf_includes_value_byteoffset_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_includes_value_byteoffset_encoding.md)

新增于: v5.3.0

- `value` [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) | [](http://nodejs.cn/s/SXbo1v) 要查找的值。
- `byteOffset` [](http://nodejs.cn/s/SXbo1v) `buf` 中开始查找的偏移量。**默认值:** `0`。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `value` 是字符串，则指定 `value` 的字符编码。**默认值:** `'utf8'`。
- 返回: [](http://nodejs.cn/s/jFbvuT) 如果 `buf` 查找到 `value`，则返回 `true`，否则返回 `false`。

相当于 [`buf.indexOf() !== -1`](http://nodejs.cn/s/eFR2KV)。

```js
const buf = Buffer.from('this is a buffer');

console.log(buf.includes('this'));
// 打印: true
console.log(buf.includes('is'));
// 打印: true
console.log(buf.includes(Buffer.from('a buffer')));
// 打印: true
console.log(buf.includes(97));
// 打印: true（97 是 'a' 的十进制 ASCII 值）
console.log(buf.includes(Buffer.from('a buffer example')));
// 打印: false
console.log(buf.includes(Buffer.from('a buffer example').slice(0, 8)));
// 打印: true
console.log(buf.includes('this', 4));
// 打印: false
```

### buf.indexOf(value[, byteOffset][, encoding])[#](http://nodejs.cn/api/buffer.html#buffer_buf_indexof_value_byteoffset_encoding)

[中英对照](http://nodejs.cn/api/buffer/buf_indexof_value_byteoffset_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_indexof_value_byteoffset_encoding.md)

版本历史

- `value` [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) | [](http://nodejs.cn/s/SXbo1v) 要查找的值。
- `byteOffset` [](http://nodejs.cn/s/SXbo1v) `buf` 中开始查找的偏移量。**默认值:** `0`。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `value` 是字符串，则指定 `value` 的字符编码。**默认值:** `'utf8'`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `buf` 中首次出现 `value` 的索引，如果 `buf` 没包含 `value` 则返回 `-1`。

如果 `value` 是：

- 一个字符串，则 `value` 根据 `encoding` 的字符编码进行解析。
- 一个 `Buffer` 或 [`Uint8Array`](http://nodejs.cn/s/ZbDkpm)，则 `value` 会整个进行对比。如果要对比部分 `Buffer`，可使用 [`buf.slice()`](http://nodejs.cn/s/uQPgxt)。
- 一个数值, 则 `value` 会被解析成 `0` 至 `255` 之间的无符号八位整数值。

```js
const buf = Buffer.from('this is a buffer');

console.log(buf.indexOf('this'));
// 打印: 0
console.log(buf.indexOf('is'));
// 打印: 2
console.log(buf.indexOf(Buffer.from('a buffer')));
// 打印: 8
console.log(buf.indexOf(97));
// 打印: 8（97 是 'a' 的十进制 ASCII 值）
console.log(buf.indexOf(Buffer.from('a buffer example')));
// 打印: -1
console.log(buf.indexOf(Buffer.from('a buffer example').slice(0, 8)));
// 打印: 8

const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'utf16le');

console.log(utf16Buffer.indexOf('\u03a3', 0, 'utf16le'));
// 打印: 4
console.log(utf16Buffer.indexOf('\u03a3', -4, 'utf16le'));
// 打印: 6
```

如果 `value` 不是一个字符串、数值、或 `Buffer`，则此方法将会抛出 `TypeError`。 如果 `value` 是一个数值，则会被转换成介于 0 到 255 之间的整数值。

如果 `byteOffset` 不是一个数值，则会被转换成数值。 如果转换后的值为 `NaN` 或 `0`, 则会查找整个 buffer。 这与 [`String#indexOf()`](http://nodejs.cn/s/Uqm5hr) 是一致的。

```js
const b = Buffer.from('abcdef');

// 传入一个数值，但不是有效的字节。
// 打印：2，相当于查找 99 或 'c'。
console.log(b.indexOf(99.9));
console.log(b.indexOf(256 + 99));

// 传入被转换成 NaN 或 0 的 byteOffset。
// 打印：1，查找整个 buffer。
console.log(b.indexOf('b', undefined));
console.log(b.indexOf('b', {}));
console.log(b.indexOf('b', null));
console.log(b.indexOf('b', []));
```

如果 `value` 是一个空字符串或空 `Buffer`，且 `byteOffset` 小于 `buf.length`，则返回 `byteOffset`。 如果 `value` 是一个空字符串，且 `byteOffset` 大于或等于 `buf.length`，则返回 `buf.length`。

### buf.keys()[#](http://nodejs.cn/api/buffer.html#buffer_buf_keys)

[中英对照](http://nodejs.cn/api/buffer/buf_keys.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_keys.md)

新增于: v1.1.0

- 返回: [](http://nodejs.cn/s/Y2SE1q)

创建并返回 `buf` 键名（索引）的[迭代器](http://nodejs.cn/s/KK7Xfc)。

```js
const buf = Buffer.from('buffer');

for (const key of buf.keys()) {
  console.log(key);
}
// 打印:
//   0
//   1
//   2
//   3
//   4
//   5
```

### buf.lastIndexOf(value[, byteOffset][, encoding])[#](http://nodejs.cn/api/buffer.html#buffer_buf_lastindexof_value_byteoffset_encoding)

[中英对照](http://nodejs.cn/api/buffer/buf_lastindexof_value_byteoffset_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_lastindexof_value_byteoffset_encoding.md)

版本历史

- `value` [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) | [](http://nodejs.cn/s/SXbo1v) 要查找的值。
- `byteOffset` [](http://nodejs.cn/s/SXbo1v) `buf` 中开始查找的偏移量。**默认值:** [`buf.length`](http://nodejs.cn/s/hn6FjL)`- 1`。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `value` 是字符串，则指定 `value` 的字符编码。**默认值:** `'utf8'`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `buf` 中最后一次出现 `value` 的索引，如果 `buf` 没包含 `value` 则返回 `-1`。

与 [`buf.indexOf()`](http://nodejs.cn/s/eFR2KV) 的区别是，查找的是 `value` 最后一次出现的索引，而不是首次出现。

```js
const buf = Buffer.from('this buffer is a buffer');

console.log(buf.lastIndexOf('this'));
// 打印: 0
console.log(buf.lastIndexOf('buffer'));
// 打印: 17
console.log(buf.lastIndexOf(Buffer.from('buffer')));
// 打印: 17
console.log(buf.lastIndexOf(97));
// 打印: 15（97 是 'a' 的十进制 ASCII 值）
console.log(buf.lastIndexOf(Buffer.from('yolo')));
// 打印: -1
console.log(buf.lastIndexOf('buffer', 5));
// 打印: 5
console.log(buf.lastIndexOf('buffer', 4));
// 打印: -1

const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'utf16le');

console.log(utf16Buffer.lastIndexOf('\u03a3', undefined, 'utf16le'));
// 打印: 6
console.log(utf16Buffer.lastIndexOf('\u03a3', -5, 'utf16le'));
// 打印: 4
```

如果 `value` 不是一个字符串、数字、或 `Buffer`，则此方法将会抛出 `TypeError`。 如果 `value` 是一个数值，则它将会被强制转换成一个有效的字节值，即一个 0 到 255 之间的整数。

如果 `byteOffset` 不是一个数值，则会被转换成数值。 如果转换后的值为 `NaN`，比如 `{}` 或 `undefined`，则会查找整个 buffer。 这与 [`String#lastIndexOf()`](http://nodejs.cn/s/2oXRjB) 是一致的。

```js
const b = Buffer.from('abcdef');

// 传入一个数值，但不是一个有效的字节。
// 输出：2，相当于查找 99 或 'c'。
console.log(b.lastIndexOf(99.9));
console.log(b.lastIndexOf(256 + 99));

// 传入被转换成 NaN 的 byteOffset。
// 输出：1，查找整个 buffer。
console.log(b.lastIndexOf('b', undefined));
console.log(b.lastIndexOf('b', {}));

// 传入被转换成 0 的 byteOffset。
// 输出：-1，相当于传入 0。
console.log(b.lastIndexOf('b', null));
console.log(b.lastIndexOf('b', []));
```

如果 `value` 是一个空字符串或空 `Buffer`，则返回 `byteOffset`。

### buf.length[#](http://nodejs.cn/api/buffer.html#buffer_buf_length)

[中英对照](http://nodejs.cn/api/buffer/buf_length.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_length.md)

新增于: v0.1.90

- [](http://nodejs.cn/s/SXbo1v)

返回内存中分配给 `buf` 的字节数。 不一定反映 `buf` 中可用数据的字节量。

```js
// 创建一个 `Buffer`，并写入一个 ASCII 字符串。

const buf = Buffer.alloc(1234);

console.log(buf.length);
// 打印: 1234

buf.write('http://nodejs.cn/', 0, 'ascii');

console.log(buf.length);
// 打印: 1234
```

虽然 `length` 属性不是不可变的，但改变 `length` 的值可能会造成不确定的结果。 如果想改变一个 `Buffer` 的长度，应该将 `length` 视为只读的，且使用 [`buf.slice()`](http://nodejs.cn/s/uQPgxt) 创建一个新的 `Buffer`。

```js
let buf = Buffer.allocUnsafe(10);

buf.write('abcdefghj', 0, 'ascii');

console.log(buf.length);
// 打印: 10

buf = buf.slice(0, 5);

console.log(buf.length);
// 打印: 5
```

### buf.parent[#](http://nodejs.cn/api/buffer.html#buffer_buf_parent)

暂无中英对照

废弃于: v8.0.0



[稳定性: 0](http://nodejs.cn/api/documentation.html#documentation_stability_index) - 废弃: 改为使用 [`buf.buffer`](http://nodejs.cn/s/wCSPHh) 。



The `buf.parent` property is a deprecated alias for `buf.buffer`.

### buf.readBigInt64BE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readbigint64be_offset)

### buf.readBigInt64LE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readbigint64le_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readbigint64le_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readbigint64le_offset.md)

新增于: v12.0.0

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 8`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/gJMq1y)

用指定的字节序格式（`readBigInt64BE()` 返回大端序， `readBigInt64LE()` 返回小端序）从 `buf` 中指定的 `offset` 读取一个有符号的 64 位整数值。

从 `Buffer` 中读取的整数值会被解析为二进制补码值。

### buf.readBigUInt64BE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readbiguint64be_offset)

### buf.readBigUInt64LE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readbiguint64le_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readbiguint64le_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readbiguint64le_offset.md)

新增于: v12.0.0

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 8`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/gJMq1y)

用指定的字节序格式（`readBigUInt64BE()` 返回大端序， `readBigUInt64LE()` 返回小端序）从 `buf` 中指定的 `offset` 读取一个无符号的 64 位整数值。

```js
const buf = Buffer.from([0x00, 0x00, 0x00, 0x00, 0xff, 0xff, 0xff, 0xff]);

console.log(buf.readBigUInt64BE(0));
// 打印: 4294967295n

console.log(buf.readBigUInt64LE(0));
// 打印: 18446744069414584320n
```

### buf.readDoubleBE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readdoublebe_offset)

### buf.readDoubleLE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readdoublele_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readdoublele_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readdoublele_offset.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 8`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

用指定的字节序格式（`readDoubleBE()` 返回大端序， `readDoubleLE()` 返回小端序）从 `buf` 中指定的 `offset` 读取一个 64 位双精度值。

```js
const buf = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8]);

console.log(buf.readDoubleBE(0));
// 打印: 8.20788039913184e-304
console.log(buf.readDoubleLE(0));
// 打印: 5.447603722011605e-270
console.log(buf.readDoubleLE(1));
// 抛出异常 ERR_OUT_OF_RANGE。
```

### buf.readFloatBE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readfloatbe_offset)

### buf.readFloatLE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readfloatle_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readfloatle_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readfloatle_offset.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 4`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

用指定的字节序格式（`readFloatBE()` 返回大端序， `readFloatLE()` 返回小端序）从 `buf` 中指定的 `offset` 读取一个 32 位浮点值。

```js
const buf = Buffer.from([1, 2, 3, 4]);

console.log(buf.readFloatBE(0));
// 打印: 2.387939260590663e-38
console.log(buf.readFloatLE(0));
// 打印: 1.539989614439558e-36
console.log(buf.readFloatLE(1));
// 抛出异常 ERR_OUT_OF_RANGE。
```

### buf.readInt8([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readint8_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readint8_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readint8_offset.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 1`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

从 `buf` 中指定的 `offset` 读取一个有符号的 8 位整数值。

从 `Buffer` 中读取的整数值会被解析为二进制补码值。

```js
const buf = Buffer.from([-1, 5]);

console.log(buf.readInt8(0));
// 打印: -1
console.log(buf.readInt8(1));
// 打印: 5
console.log(buf.readInt8(2));
// 抛出异常 ERR_OUT_OF_RANGE。
```

### buf.readInt16BE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readint16be_offset)

### buf.readInt16LE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readint16le_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readint16le_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readint16le_offset.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 2`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

用指定的字节序格式（`readInt16BE()` 返回大端序， `readInt16LE()` 返回小端序）从 `buf` 中指定的 `offset` 读取一个有符号的 16 位整数值。

从 `Buffer` 中读取的整数值会被解析为二进制补码值。

```js
const buf = Buffer.from([0, 5]);

console.log(buf.readInt16BE(0));
// 打印: 5
console.log(buf.readInt16LE(0));
// 打印: 1280
console.log(buf.readInt16LE(1));
// 抛出异常 ERR_OUT_OF_RANGE。
```

### buf.readInt32BE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readint32be_offset)

### buf.readInt32LE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readint32le_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readint32le_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readint32le_offset.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 4`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

用指定的字节序格式（`readInt32BE()` 返回大端序， `readInt32LE()` 返回小端序）从 `buf` 中指定的 `offset` 读取一个有符号的 32 位整数值。

从 `Buffer` 中读取的整数值会被解析为二进制补码值。

```js
const buf = Buffer.from([0, 0, 0, 5]);

console.log(buf.readInt32BE(0));
// 打印: 5
console.log(buf.readInt32LE(0));
// 打印: 83886080
console.log(buf.readInt32LE(1));
// 抛出异常 ERR_OUT_OF_RANGE。
```

### buf.readIntBE(offset, byteLength)[#](http://nodejs.cn/api/buffer.html#buffer_buf_readintbe_offset_bytelength)

### buf.readIntLE(offset, byteLength)[#](http://nodejs.cn/api/buffer.html#buffer_buf_readintle_offset_bytelength)

[中英对照](http://nodejs.cn/api/buffer/buf_readintle_offset_bytelength.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readintle_offset_bytelength.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - byteLength`。
- `byteLength` [](http://nodejs.cn/s/SXbo1v) 要读取的字节数。必须满足`0 < byteLength <= 6`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

从 `buf` 中指定的 `offset` 读取 `byteLength` 个字节，并将读取的值解析为二进制补码值。 最高支持 48 位精度。

```js
const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readIntLE(0, 6).toString(16));
// 打印: -546f87a9cbee
console.log(buf.readIntBE(0, 6).toString(16));
// 打印: 1234567890ab
console.log(buf.readIntBE(1, 6).toString(16));
// 抛出异常 ERR_OUT_OF_RANGE。
console.log(buf.readIntBE(1, 0).toString(16));
// 抛出异常 ERR_OUT_OF_RANGE。
```

### buf.readUInt8([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readuint8_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readuint8_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readuint8_offset.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 1`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

从 `buf` 中指定的 `offset` 读取一个无符号的 8 位整数值。

```js
const buf = Buffer.from([1, -2]);

console.log(buf.readUInt8(0));
// 打印: 1
console.log(buf.readUInt8(1));
// 打印: 254
console.log(buf.readUInt8(2));
// 抛出异常 ERR_OUT_OF_RANGE。
```

### buf.readUInt16BE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readuint16be_offset)

### buf.readUInt16LE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readuint16le_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readuint16le_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readuint16le_offset.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 2`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

用指定的字节序格式（`readUInt16BE()` 返回大端序， `readUInt16LE()` 返回小端序）从 `buf` 中指定的 `offset` 读取一个无符号的 16 位整数值。

```js
const buf = Buffer.from([0x12, 0x34, 0x56]);

console.log(buf.readUInt16BE(0).toString(16));
// 打印: 1234
console.log(buf.readUInt16LE(0).toString(16));
// 打印: 3412
console.log(buf.readUInt16BE(1).toString(16));
// 打印: 3456
console.log(buf.readUInt16LE(1).toString(16));
// 打印: 5634
console.log(buf.readUInt16LE(2).toString(16));
// 抛出异常 ERR_OUT_OF_RANGE。
```

### buf.readUInt32BE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readuint32be_offset)

### buf.readUInt32LE([offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_readuint32le_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_readuint32le_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readuint32le_offset.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 4`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

用指定的字节序格式（`readUInt32BE()` 返回大端序， `readUInt32LE()` 返回小端序）从 `buf` 中指定的 `offset` 读取一个无符号的 32 位整数值。

```js
const buf = Buffer.from([0x12, 0x34, 0x56, 0x78]);

console.log(buf.readUInt32BE(0).toString(16));
// 打印: 12345678
console.log(buf.readUInt32LE(0).toString(16));
// 打印: 78563412
console.log(buf.readUInt32LE(1).toString(16));
// 抛出异常 ERR_OUT_OF_RANGE。
```

### buf.readUIntBE(offset, byteLength)[#](http://nodejs.cn/api/buffer.html#buffer_buf_readuintbe_offset_bytelength)

### buf.readUIntLE(offset, byteLength)[#](http://nodejs.cn/api/buffer.html#buffer_buf_readuintle_offset_bytelength)

[中英对照](http://nodejs.cn/api/buffer/buf_readuintle_offset_bytelength.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_readuintle_offset_bytelength.md)

版本历史

- `offset` [](http://nodejs.cn/s/SXbo1v) 开始读取之前要跳过的字节数。必须满足`0 <= offset <= buf.length - byteLength`。
- `byteLength` [](http://nodejs.cn/s/SXbo1v) 要读取的字节数。必须满足`0 < byteLength <= 6`。
- 返回: [](http://nodejs.cn/s/SXbo1v)

从 `buf` 中指定的 `offset` 读取 `byteLength` 个字节，并将读取的值解析为无符号的整数。 最高支持 48 位精度。

```js
const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

console.log(buf.readUIntBE(0, 6).toString(16));
// 打印: 1234567890ab
console.log(buf.readUIntLE(0, 6).toString(16));
// 打印: ab9078563412
console.log(buf.readUIntBE(1, 6).toString(16));
//抛出异常 ERR_OUT_OF_RANGE。
```

### buf.subarray([start[, end]])[#](http://nodejs.cn/api/buffer.html#buffer_buf_subarray_start_end)

[中英对照](http://nodejs.cn/api/buffer/buf_subarray_start_end.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_subarray_start_end.md)

新增于: v3.0.0

- `start` [](http://nodejs.cn/s/SXbo1v) 新 `Buffer` 开始的位置。**默认值:** `0`。
- `end` [](http://nodejs.cn/s/SXbo1v) 新 `Buffer` 结束的位置（不包含）。**默认值:** [`buf.length`](http://nodejs.cn/s/hn6FjL)。
- 返回: [](http://nodejs.cn/s/6x1hD3)

返回一个新的 `Buffer`，它引用与原始的 Buffer 相同的内存，但是由 `start` 和 `end` 索引进行偏移和裁剪。

指定大于 [`buf.length`](http://nodejs.cn/s/hn6FjL) 的 `end` 将会返回与 `end` 等于 [`buf.length`](http://nodejs.cn/s/hn6FjL) 时相同的结果。

修改新的 `Buffer` 切片将会修改原始 `Buffer` 中的内存，因为两个对象分配的内存是重叠的。

```js
// 使用 ASCII 字母创建一个 `Buffer`，然后进行切片，再修改原始 `Buffer` 中的一个字节。

const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 是 'a' 的十进制 ASCII 值。
  buf1[i] = i + 97;
}

const buf2 = buf1.subarray(0, 3);

console.log(buf2.toString('ascii', 0, buf2.length));
// 打印: abc

buf1[0] = 33;

console.log(buf2.toString('ascii', 0, buf2.length));
// 打印: !bc
```

指定负的索引会导致切片的生成是相对于 `buf` 的末尾而不是开头。

```js
const buf = Buffer.from('buffer');

console.log(buf.subarray(-6, -1).toString());
// 打印: buffe
// (相当于 buf.subarray(0, 5)。)

console.log(buf.subarray(-6, -2).toString());
// 打印: buff
// (相当于 buf.subarray(0, 4)。)

console.log(buf.subarray(-5, -2).toString());
// 打印: uff
// (相当于 buf.subarray(1, 4)。)
```

### buf.slice([start[, end]])[#](http://nodejs.cn/api/buffer.html#buffer_buf_slice_start_end)

[中英对照](http://nodejs.cn/api/buffer/buf_slice_start_end.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_slice_start_end.md)

版本历史

- `start` [](http://nodejs.cn/s/SXbo1v) 新 `Buffer` 开始的位置。**默认值:** `0`。
- `end` [](http://nodejs.cn/s/SXbo1v) 新 `Buffer` 结束的位置（不包含）。**默认值:** [`buf.length`](http://nodejs.cn/s/hn6FjL)。
- 返回: [](http://nodejs.cn/s/6x1hD3)

返回一个新的 `Buffer`，它引用与原始的 Buffer 相同的内存，但是由 `start` 和 `end` 索引进行偏移和裁剪。

这与 `buf.subarray()` 的行为相同。

此方法与 `Uint8Array.prototype.slice()` 不兼容，后者是 `Buffer` 的超类。 若要复制切片，则使用 `Uint8Array.prototype.slice()`。

```js
const buf = Buffer.from('buffer');

const copiedBuf = Uint8Array.prototype.slice.call(buf);
copiedBuf[0]++;
console.log(copiedBuf.toString());
// 打印: cuffer

console.log(buf.toString());
// 打印: buffer
```

### buf.swap16()[#](http://nodejs.cn/api/buffer.html#buffer_buf_swap16)

[中英对照](http://nodejs.cn/api/buffer/buf_swap16.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_swap16.md)

新增于: v5.10.0

- 返回: [](http://nodejs.cn/s/6x1hD3) `buf` 的引用。

将 `buf` 解析成无符号的 16 位整数的数组，并且以字节顺序原地进行交换。 如果 [`buf.length`](http://nodejs.cn/s/hn6FjL) 不是 2 的倍数，则抛出 [`ERR_INVALID_BUFFER_SIZE`](http://nodejs.cn/s/NEzxCE)。

```js
const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

console.log(buf1);
// 打印: <Buffer 01 02 03 04 05 06 07 08>

buf1.swap16();

console.log(buf1);
// 打印: <Buffer 02 01 04 03 06 05 08 07>

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

buf2.swap16();
// 抛出异常 ERR_INVALID_BUFFER_SIZE。
```

`buf.swap16()` 的一个方便用途是在 UTF-16 小端序和 UTF-16 大端序之间执行快速的原地转换：

```js
const buf = Buffer.from('This is little-endian UTF-16', 'utf16le');
buf.swap16(); // 转换为大端序 UTF-16 文本。
```

### buf.swap32()[#](http://nodejs.cn/api/buffer.html#buffer_buf_swap32)

[中英对照](http://nodejs.cn/api/buffer/buf_swap32.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_swap32.md)

新增于: v5.10.0

- 返回: [](http://nodejs.cn/s/6x1hD3) `buf` 的引用。

将 `buf` 解析成无符号的 32 位整数的数组，并且以字节顺序原地进行交换。 如果 [`buf.length`](http://nodejs.cn/s/hn6FjL) 不是 4 的倍数，则抛出 [`ERR_INVALID_BUFFER_SIZE`](http://nodejs.cn/s/NEzxCE)。

```js
const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

console.log(buf1);
// 打印: <Buffer 01 02 03 04 05 06 07 08>

buf1.swap32();

console.log(buf1);
// 打印: <Buffer 04 03 02 01 08 07 06 05>

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

buf2.swap32();
// 抛出异常 ERR_INVALID_BUFFER_SIZE。
```

### buf.swap64()[#](http://nodejs.cn/api/buffer.html#buffer_buf_swap64)

[中英对照](http://nodejs.cn/api/buffer/buf_swap64.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_swap64.md)

新增于: v6.3.0

- 返回: [](http://nodejs.cn/s/6x1hD3) `buf` 的引用。

将 `buf` 解析成 64 位数值的数组，并且以字节顺序原地进行交换。 如果 [`buf.length`](http://nodejs.cn/s/hn6FjL) 不是 8 的倍数，则抛出 [`ERR_INVALID_BUFFER_SIZE`](http://nodejs.cn/s/NEzxCE)。

```js
const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

console.log(buf1);
// 打印: <Buffer 01 02 03 04 05 06 07 08>

buf1.swap64();

console.log(buf1);
// 打印: <Buffer 08 07 06 05 04 03 02 01>

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

buf2.swap64();
// 抛出异常 ERR_INVALID_BUFFER_SIZE。
```

JavaScript 不能编码 64 位整数。 该方法是用来处理 64 位浮点数的。

### buf.toJSON()[#](http://nodejs.cn/api/buffer.html#buffer_buf_tojson)

[中英对照](http://nodejs.cn/api/buffer/buf_tojson.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_tojson.md)

新增于: v0.9.2

- 返回: [](http://nodejs.cn/s/jzn6Ao)

返回 `buf` 的 JSON 格式。 当字符串化 `Buffer` 实例时，[`JSON.stringify()`](http://nodejs.cn/s/bmLTNS) 会调用该函数。

```js
const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
const json = JSON.stringify(buf);

console.log(json);
// 打印: {"type":"Buffer","data":[1,2,3,4,5]}

const copy = JSON.parse(json, (key, value) => {
  return value && value.type === 'Buffer' ?
    Buffer.from(value.data) :
    value;
});

console.log(copy);
// 打印: <Buffer 01 02 03 04 05>
```

### buf.toString([encoding[, start[, end]]])[#](http://nodejs.cn/api/buffer.html#buffer_buf_tostring_encoding_start_end)

[中英对照](http://nodejs.cn/api/buffer/buf_tostring_encoding_start_end.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_tostring_encoding_start_end.md)

新增于: v0.1.90

- `encoding` [](http://nodejs.cn/s/9Tw2bK) 使用的字符编码。**默认值:** `'utf8'`。
- `start` [](http://nodejs.cn/s/SXbo1v) 开始解码的字节偏移量。**默认值:** `0`。
- `end` [](http://nodejs.cn/s/SXbo1v) 结束解码的字节偏移量（不包含）。**默认值:** [`buf.length`](http://nodejs.cn/s/hn6FjL)。
- 返回: [](http://nodejs.cn/s/9Tw2bK)

根据 `encoding` 指定的字符编码将 `buf` 解码成字符串。 传入 `start` 和 `end` 可以只解码 `buf` 的子集。

字符串的最大长度（以 UTF-16 为单位）可查看 [`buffer.constants.MAX_STRING_LENGTH`](http://nodejs.cn/s/fReXmD)。

```js
const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 是 'a' 的十进制 ASCII 值。
  buf1[i] = i + 97;
}

console.log(buf1.toString('ascii'));
// 打印: abcdefghijklmnopqrstuvwxyz
console.log(buf1.toString('ascii', 0, 5));
// 打印: abcde

const buf2 = Buffer.from('tést');

console.log(buf2.toString('hex'));
// 打印: 74c3a97374
console.log(buf2.toString('utf8', 0, 3));
// 打印: té
console.log(buf2.toString(undefined, 0, 3));
// 打印: té
```

### buf.values()[#](http://nodejs.cn/api/buffer.html#buffer_buf_values)

[中英对照](http://nodejs.cn/api/buffer/buf_values.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_values.md)

新增于: v1.1.0

- 返回: [](http://nodejs.cn/s/Y2SE1q)

创建并返回 `buf` 键值（字节）的[迭代器](http://nodejs.cn/s/KK7Xfc)。 当对 `Buffer` 使用 `for..of` 时会调用该函数。

```js
const buf = Buffer.from('buffer');

for (const value of buf.values()) {
  console.log(value);
}
// 打印:
//   98
//   117
//   102
//   102
//   101
//   114

for (const value of buf) {
  console.log(value);
}
// 打印:
//   98
//   117
//   102
//   102
//   101
//   114
```

### buf.write(string[, offset[, length]][, encoding])[#](http://nodejs.cn/api/buffer.html#buffer_buf_write_string_offset_length_encoding)

[中英对照](http://nodejs.cn/api/buffer/buf_write_string_offset_length_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_write_string_offset_length_encoding.md)

新增于: v0.1.90

- `string` [](http://nodejs.cn/s/9Tw2bK) 要写入 `buf` 的字符串。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入 `string` 之前要跳过的字节数。**默认值:** `0`。
- `length` [](http://nodejs.cn/s/SXbo1v) 要写入的字节数。**默认值:** `buf.length - offset`。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) `string` 的字符编码。**默认值:** `'utf8'`。
- 返回: [](http://nodejs.cn/s/SXbo1v) 已写入的字节数。

根据 `encoding` 指定的字符编码将 `string` 写入到 `buf` 中的 `offset` 位置。 `length` 参数是要写入的字节数。 如果 `buf` 没有足够的空间保存整个字符串，则只会写入 `string`的一部分。 只编码了一部分的字符不会被写入。

```js
const buf = Buffer.alloc(256);

const len = buf.write('\u00bd + \u00bc = \u00be', 0);

console.log(`${len} 个字节: ${buf.toString('utf8', 0, len)}`);
// 打印: 12 个字节: ½ + ¼ = ¾
```

### buf.writeBigInt64BE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writebigint64be_value_offset)

### buf.writeBigInt64LE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writebigint64le_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writebigint64le_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writebigint64le_value_offset.md)

新增于: v12.0.0

- `value` [](http://nodejs.cn/s/gJMq1y) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 8`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

用指定的字节序格式（`writeBigInt64BE()` 写入大端序， `writeBigInt64LE()` 写入小端序）将 `value` 写入到 `buf` 中指定的 `offset` 位置。

`value` 会被解析并写入为二进制补码的有符号整数。

```js
const buf = Buffer.allocUnsafe(8);

buf.writeBigInt64BE(0x0102030405060708n, 0);

console.log(buf);
// 打印: <Buffer 01 02 03 04 05 06 07 08>
```

### buf.writeBigUInt64BE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writebiguint64be_value_offset)

### buf.writeBigUInt64LE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writebiguint64le_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writebiguint64le_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writebiguint64le_value_offset.md)

新增于: v12.0.0

- `value` [](http://nodejs.cn/s/gJMq1y) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 8`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

用指定的字节序格式（`writeBigUInt64BE()` 写入大端序， `writeBigUInt64LE()` 写入小端序）将 `value` 写入到 `buf` 中指定的 `offset` 位置。

```js
const buf = Buffer.allocUnsafe(8);

buf.writeBigUInt64LE(0xdecafafecacefaden, 0);

console.log(buf);
// 打印: <Buffer de fa ce ca fe fa ca de>
```

### buf.writeDoubleBE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writedoublebe_value_offset)

### buf.writeDoubleLE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writedoublele_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writedoublele_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writedoublele_value_offset.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 8`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

用指定的字节序格式（`writeDoubleBE()` 写入大端序， `writeDoubleLE()` 写入小端序）将 `value` 写入到 `buf` 中指定的 `offset` 位置。 `value` 必须是 64 位双精度值。 当 `value` 不是 64 位双精度值时，行为是未定义的。

```js
const buf = Buffer.allocUnsafe(8);

buf.writeDoubleBE(123.456, 0);

console.log(buf);
// 打印: <Buffer 40 5e dd 2f 1a 9f be 77>

buf.writeDoubleLE(123.456, 0);

console.log(buf);
// 打印: <Buffer 77 be 9f 1a 2f dd 5e 40>
```

### buf.writeFloatBE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writefloatbe_value_offset)

### buf.writeFloatLE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writefloatle_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writefloatle_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writefloatle_value_offset.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 4`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

用指定的字节序格式（`writeFloatBE()` 写入大端序， `writeFloatLE()` 写入小端序）将 `value` 写入到 `buf` 中指定的 `offset` 位置。 `value` 必须是 32 位浮点值。 当 `value`不是 32 位浮点值时，行为是未定义的。

```js
const buf = Buffer.allocUnsafe(4);

buf.writeFloatBE(0xcafebabe, 0);

console.log(buf);
// 打印: <Buffer 4f 4a fe bb>

buf.writeFloatLE(0xcafebabe, 0);

console.log(buf);
// 打印: <Buffer bb fe 4a 4f>
```

### buf.writeInt8(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeint8_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writeint8_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writeint8_value_offset.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 1`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

将 `value` 写入到 `buf` 中指定的 `offset` 位置。 `value` 必须是有符号的 8 位整数。 当 `value` 不是有符号的 8 位整数时，行为是未定义的。

`value` 会被解析并写入为二进制补码的有符号整数。

```js
const buf = Buffer.allocUnsafe(2);

buf.writeInt8(2, 0);
buf.writeInt8(-2, 1);

console.log(buf);
// 打印: <Buffer 02 fe>
```

### buf.writeInt16BE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeint16be_value_offset)

### buf.writeInt16LE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeint16le_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writeint16le_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writeint16le_value_offset.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 2`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

用指定的字节序格式（`writeInt16BE()` 写入大端序， `writeInt16LE()` 写入小端序）将 `value` 写入到 `buf` 中指定的 `offset` 位置。 `value` 必须是有符号的 16 位整数。 当 `value` 不是有符号的 16 位整数时，行为是未定义的。

```js
const buf = Buffer.allocUnsafe(4);

buf.writeInt16BE(0x0102, 0);
buf.writeInt16LE(0x0304, 2);

console.log(buf);
// 打印: <Buffer 01 02 04 03>
```

### buf.writeInt32BE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeint32be_value_offset)

### buf.writeInt32LE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeint32le_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writeint32le_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writeint32le_value_offset.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 4`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

用指定的字节序格式（`writeInt32BE()` 写入大端序， `writeInt32LE()` 写入小端序）将 `value` 写入到 `buf` 中指定的 `offset` 位置。 `value` 必须是有符号的 32 位整数。 当 `value` 不是有符号的 32 位整数，行为是未定义的。

`value` 会被解析并写入为二进制补码的有符号整数。

```js
const buf = Buffer.allocUnsafe(8);

buf.writeInt32BE(0x01020304, 0);
buf.writeInt32LE(0x05060708, 4);

console.log(buf);
// 打印: <Buffer 01 02 03 04 08 07 06 05>
```

### buf.writeIntBE(value, offset, byteLength)[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeintbe_value_offset_bytelength)

### buf.writeIntLE(value, offset, byteLength)[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeintle_value_offset_bytelength)

[中英对照](http://nodejs.cn/api/buffer/buf_writeintle_value_offset_bytelength.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writeintle_value_offset_bytelength.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - byteLength`。
- `byteLength` [](http://nodejs.cn/s/SXbo1v) 要写入的字节数。必须满足`0 < byteLength <= 6`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

将 `value` 中的 `byteLength` 个字节写入到 `buf` 中指定的 `offset` 位置。 最高支持 48 位精度。 当 `value` 不是有符号的整数时，行为是未定义的。

```js
const buf = Buffer.allocUnsafe(6);

buf.writeIntBE(0x1234567890ab, 0, 6);

console.log(buf);
// 打印: <Buffer 12 34 56 78 90 ab>

buf.writeIntLE(0x1234567890ab, 0, 6);

console.log(buf);
// 打印: <Buffer ab 90 78 56 34 12>
```

### buf.writeUInt8(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeuint8_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writeuint8_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writeuint8_value_offset.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 1`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

将 `value` 写入到 `buf` 中指定的 `offset` 位置。 `value` 必须是无符号的 8 位整数。 当 `value` 不是无符号的 8 位整数时，行为是未定义的。

```js
const buf = Buffer.allocUnsafe(4);

buf.writeUInt8(0x3, 0);
buf.writeUInt8(0x4, 1);
buf.writeUInt8(0x23, 2);
buf.writeUInt8(0x42, 3);

console.log(buf);
// 打印: <Buffer 03 04 23 42>
```

### buf.writeUInt16BE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeuint16be_value_offset)

### buf.writeUInt16LE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeuint16le_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writeuint16le_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writeuint16le_value_offset.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 2`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

用指定的字节序格式（`writeUInt16BE()` 写入大端序， `writeUInt16LE()` 写入小端序）将 `value` 写入到 `buf` 中指定的 `offset` 位置。 `value` 必须是无符号的 16 位整数。 当 `value` 不是无符号的 16 位整数时，行为是未定义的。

```js
const buf = Buffer.allocUnsafe(4);

buf.writeUInt16BE(0xdead, 0);
buf.writeUInt16BE(0xbeef, 2);

console.log(buf);
// 打印: <Buffer de ad be ef>

buf.writeUInt16LE(0xdead, 0);
buf.writeUInt16LE(0xbeef, 2);

console.log(buf);
// 打印: <Buffer ad de ef be>
```

### buf.writeUInt32BE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeuint32be_value_offset)

### buf.writeUInt32LE(value[, offset])[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeuint32le_value_offset)

[中英对照](http://nodejs.cn/api/buffer/buf_writeuint32le_value_offset.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writeuint32le_value_offset.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - 4`。**默认值:** `0`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

用指定的字节序格式（`writeUInt32BE()` 写入大端序， `writeUInt32LE()` 写入小端序）将 `value` 写入到 `buf` 中指定的 `offset` 位置。 `value` 必须是无符号的 32 位整数。 当 `value` 不是无符号的 32 位整数时，行为是未定义的。

```js
const buf = Buffer.allocUnsafe(4);

buf.writeUInt32BE(0xfeedface, 0);

console.log(buf);
// 打印: <Buffer fe ed fa ce>

buf.writeUInt32LE(0xfeedface, 0);

console.log(buf);
// 打印: <Buffer ce fa ed fe>
```

### buf.writeUIntBE(value, offset, byteLength)[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeuintbe_value_offset_bytelength)

### buf.writeUIntLE(value, offset, byteLength)[#](http://nodejs.cn/api/buffer.html#buffer_buf_writeuintle_value_offset_bytelength)

[中英对照](http://nodejs.cn/api/buffer/buf_writeuintle_value_offset_bytelength.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buf_writeuintle_value_offset_bytelength.md)

版本历史

- `value` [](http://nodejs.cn/s/SXbo1v) 要写入 `buf` 的数值。
- `offset` [](http://nodejs.cn/s/SXbo1v) 开始写入之前要跳过的字节数。必须满足`0 <= offset <= buf.length - byteLength`。
- `byteLength` [](http://nodejs.cn/s/SXbo1v) 要写入的字节数。必须满足`0 < byteLength <= 6`。
- 返回: [](http://nodejs.cn/s/SXbo1v) `offset` 加上已写入的字节数。

将 `value` 中的 `byteLength` 个字节写入到 `buf` 中指定的 `offset` 位置。 最高支持 48 位精度。 当 `value` 不是无符号的整数时，行为是未定义的。

```js
const buf = Buffer.allocUnsafe(6);

buf.writeUIntBE(0x1234567890ab, 0, 6);

console.log(buf);
// 打印: <Buffer 12 34 56 78 90 ab>

buf.writeUIntLE(0x1234567890ab, 0, 6);

console.log(buf);
// 打印: <Buffer ab 90 78 56 34 12>
```

## buffer.INSPECT_MAX_BYTES[#](http://nodejs.cn/api/buffer.html#buffer_buffer_inspect_max_bytes)

[中英对照](http://nodejs.cn/api/buffer/buffer_inspect_max_bytes.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffer_inspect_max_bytes.md)

新增于: v0.5.4

- [](http://nodejs.cn/s/SXbo1v) **默认值:** `50`。

返回当调用 `buf.inspect()` 时将会返回的最大字节数。 这可以被用户模块重写。 有关 `buf.inspect()` 行为的更多详细信息，参阅 [`util.inspect()`](http://nodejs.cn/s/fHaJzA)。

该属性是在 `require('buffer')` 返回的 `buffer` 模块上，而不是在 `Buffer` 全局变量或 `Buffer` 实例上。

## buffer.kMaxLength[#](http://nodejs.cn/api/buffer.html#buffer_buffer_kmaxlength)

[中英对照](http://nodejs.cn/api/buffer/buffer_kmaxlength.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffer_kmaxlength.md)

新增于: v3.0.0

- [](http://nodejs.cn/s/SXbo1v) 分配给单个 `Buffer` 实例的最大内存。

[`buffer.constants.MAX_LENGTH`](http://nodejs.cn/s/aBiAe5) 的别名。

该属性是在 `require('buffer')` 返回的 `buffer` 模块上，而不是在 `Buffer` 全局变量或 `Buffer` 实例上。

## buffer.transcode(source, fromEnc, toEnc)[#](http://nodejs.cn/api/buffer.html#buffer_buffer_transcode_source_fromenc_toenc)

[中英对照](http://nodejs.cn/api/buffer/buffer_transcode_source_fromenc_toenc.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffer_transcode_source_fromenc_toenc.md)

版本历史

- `source` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) 一个 `Buffer` 或 `Uint8Array` 实例。
- `fromEnc` [](http://nodejs.cn/s/9Tw2bK) 当前字符编码。
- `toEnc` [](http://nodejs.cn/s/9Tw2bK) 目标字符编码。
- 返回: [](http://nodejs.cn/s/6x1hD3)

将指定的 `Buffer` 或 `Uint8Array` 实例从一个字符编码重新编码到另一个字符。 返回新的 `Buffer` 实例。

如果 `fromEnc` 或 `toEnc` 指定了无效的字符编码，或者无法从 `fromEnc` 转换为 `toEnc`，则抛出异常。

`buffer.transcode()` 支持的字符编码有 `'ascii'`、 `'utf8'`、 `'utf16le'`、 `'ucs2'`、 `'latin1'` 与 `'binary'`。

如果指定的字节序列无法用目标字符编码表示，则转码过程会使用替代的字符。 例如：

```js
const buffer = require('buffer');

const newBuf = buffer.transcode(Buffer.from('€'), 'utf8', 'ascii');
console.log(newBuf.toString('ascii'));
// 打印: '?'
```

因为欧元符号（`€`）无法在 US-ASCII 中表示，所以在转码 `Buffer` 时使用 `?` 代替。

该属性是在 `require('buffer')` 返回的 `buffer` 模块上，而不是在 `Buffer` 全局变量或 `Buffer` 实例上。

## SlowBuffer 类[#](http://nodejs.cn/api/buffer.html#buffer_class_slowbuffer)

暂无中英对照

废弃于: v6.0.0



[稳定性: 0](http://nodejs.cn/api/documentation.html#documentation_stability_index) - 废弃: 改为使用 [`Buffer.allocUnsafeSlow()`](http://nodejs.cn/s/PUENLw) 。



Returns an un-pooled `Buffer`.

In order to avoid the garbage collection overhead of creating many individually allocated `Buffer` instances, by default allocations under 4KB are sliced from a single larger allocated object.

In the case where a developer may need to retain a small chunk of memory from a pool for an indeterminate amount of time, it may be appropriate to create an un-pooled `Buffer` instance using `SlowBuffer` then copy out the relevant bits.

```js
// Need to keep around a few small chunks of memory.
const store = [];

socket.on('readable', () => {
  let data;
  while (null !== (data = readable.read())) {
    // Allocate for retained data.
    const sb = SlowBuffer(10);

    // Copy the data into the new allocation.
    data.copy(sb, 0, 0, 10);

    store.push(sb);
  }
});
```

Use of `SlowBuffer` should be used only as a last resort *after* a developer has observed undue memory retention in their applications.

### new SlowBuffer(size)[#](http://nodejs.cn/api/buffer.html#buffer_new_slowbuffer_size)

暂无中英对照

废弃于: v6.0.0



[稳定性: 0](http://nodejs.cn/api/documentation.html#documentation_stability_index) - 废弃: 改为使用 [`Buffer.allocUnsafeSlow()`](http://nodejs.cn/s/PUENLw) 。



- `size` [](http://nodejs.cn/s/SXbo1v) The desired length of the new `SlowBuffer`.

Allocates a new `Buffer` of `size` bytes. If `size` is larger than [`buffer.constants.MAX_LENGTH`](http://nodejs.cn/s/aBiAe5) or smaller than 0, [`ERR_INVALID_OPT_VALUE`](http://nodejs.cn/s/ouMFyk) is thrown. A zero-length `Buffer` is created if `size` is 0.

The underlying memory for `SlowBuffer` instances is *not initialized*. The contents of a newly created `SlowBuffer` are unknown and may contain sensitive data. Use [`buf.fill(0)`](http://nodejs.cn/s/2dLJk5) to initialize a `SlowBuffer` with zeroes.

```js
const { SlowBuffer } = require('buffer');

const buf = new SlowBuffer(5);

console.log(buf);
// Prints: (contents may vary): <Buffer 78 e0 82 02 01>

buf.fill(0);

console.log(buf);
// Prints: <Buffer 00 00 00 00 00>
```

## Buffer 常量[#](http://nodejs.cn/api/buffer.html#buffer_buffer_constants)

[中英对照](http://nodejs.cn/api/buffer/buffer_constants.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffer_constants.md)

新增于: v8.2.0

`buffer.constants` 是在 `require('buffer')` 返回的 `buffer` 模块上，而不是在 `Buffer` 全局变量或 `Buffer` 实例上。

### buffer.constants.MAX_LENGTH[#](http://nodejs.cn/api/buffer.html#buffer_buffer_constants_max_length)

[中英对照](http://nodejs.cn/api/buffer/buffer_constants_max_length.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffer_constants_max_length.md)

新增于: v8.2.0

- [](http://nodejs.cn/s/SXbo1v) 单个 `Buffer` 实例允许的最大内存。

在 32 位的架构上，该值是 `(2^30)-1` (~1GB)。 在 64 位的架构上，该值是 `(2^31)-1` (~2GB)。

也可使用 [`buffer.kMaxLength`](http://nodejs.cn/s/q8B7G8)。

### buffer.constants.MAX_STRING_LENGTH[#](http://nodejs.cn/api/buffer.html#buffer_buffer_constants_max_string_length)

[中英对照](http://nodejs.cn/api/buffer/buffer_constants_max_string_length.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/buffer/buffer_constants_max_string_length.md)

新增于: v8.2.0

- [](http://nodejs.cn/s/SXbo1v) 单个 `string` 实例允许的最大长度。

表示 `string` 原始数据类型能有的最大 `length`，以 UTF-16 代码为单位。

该值取决于使用的 JS 引擎。





- [查看官方文档](./http://nodejs.cn/api/buffer.html)