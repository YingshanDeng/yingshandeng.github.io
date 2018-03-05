title: SharedPen 之 Operational Transformation
date: 2018-03-05 23:25:30
tags: 'Operational Transformation'
categories: SharedPen
---

![](http://7vikhl.com1.z0.glb.clouddn.com/SharedPen-Operational%20Transformation.png)

当多用户协同编辑同一篇文档，由于没有锁的机制，便会出现内容冲突，这时就需要一定的算法来自动解决冲突，使所有对文档的修改同步后，每个协同编辑的用户看到的文档内容是完全一致的。而 Operational Transformation (简称 OT) 算法就是解决协同问题的通用算法，支持各类协同编辑引用，例如文档，表格，演示，代码编辑等。其最早发表于 1989 年，后因 Google 的在线协作产品 Google Wave，Google Docs 的应用而成熟流行。
> 关于 Google Wave: [Google Wave，值得纪念的伟大失败](http://www.ifanr.com/675122)

GitHub 上开源项目 [ot.js](https://github.com/Operational-Transformation/ot.js) 已经对 OT 算法进行 JavaScript 语言实现。本文将对 OT 算法进行介绍 👉
> **❗️注意:** ot.js 库只是针对纯文本协同编辑的 OT 算法实现；SharedPen 使用了这个库，但是 SharedPen 支持富文本编辑，所以需要对 ot.js 库进行增强，使之支持富文本文档编辑的协同。

<!-- more -->
## OT Explained
为了方便解释，以下内容基于 ot.js 这个库应用于**纯文本**协同编辑对 OT 算法进行介绍。

### Operations
- **Operation(操作) 表示对文档的修改**
  例如插入文本，删除文本这些都可以看作是一个 `operation`。如同我们 Git 操作中，两个 commit 之间的 diff
- **一个 Operation(操作) 包含多个 action(行为)**
  共有三种 action: insert, delete, retain

### Actions(insert, delete, retain)
> When you apply an operation to a string, an invisible cursor begins traversing the string from left to right. The insert and delete components mutate the string at the current position of the cursor while the retain component simply advances the position of the cursor by the specified number of characters.

将 operation 应用到字符串时，有一个隐形的光标位于字符串起点，`retain` 用于移动光标，`insert` 和 `delete` 在光标所在位置对字符串进行字符插入和删除，operation 应用完后，光标必须处于字符串的末端（这保证了应用 operation 的正确性）。

### 插入
```
var operation = new ot.Operation()
  .retain(11)
  .insert(" dolor");

operation.apply("lorem ipsum"); // => "lorem ipsum dolor"
```

![](http://7vikhl.com1.z0.glb.clouddn.com/EB6D28DA-FACE-4B68-BB71-1A7E8ABFFDEE.png)

这个例子中，operation 包含两个 action: `retain(11)` 和 `insert(" dolor")`，顺序执行这两个 action，光标从左往右移动 11 个字符，然后在光标当前位置插入字符 `" dolor"`，光标最终到达字符串末端，operation 应用完成。

我们再将 operation 应用到字符串 `"lorem ipsum amet"` 中，发现会报错！
```
operation.apply("lorem ipsum amet"); // throws an error
```

这是因为 operation 应用完后，隐形的光标并不处于字符串末端，所以报错；要解决这个问题也简单，增加一个 `retain` 行为将光标移动到字符串末端。
```
operation.retain(5).apply("lorem ipsum amet"); // => "lorem ipsum dolor amet"
```

![](http://7vikhl.com1.z0.glb.clouddn.com/36EAED8F-D0E3-48F4-AAC9-9FC7AE2EFC88.png)

### 删除
```
var operation = new ot.Operation() // create new operation
  .delete("lorem ")
  .retain(5);
operation.apply("lorem ipsum"); // => "ipsum"
```

![](http://7vikhl.com1.z0.glb.clouddn.com/4CA49A74-0941-4E5E-A7E5-78410D96D148.png)

在这个例子中，operation 包含两个 action: `delete("lorem ")` 和 `retain(5)`，顺序执行这两个 action，先删除字符 `"lorem "`，然后光标再从左往右移动 5 个字符恰好到达字符串末端，operation 应用完成。

在严谨的 OT 算法实现中，`delete` 删除字符是要严格匹配的。
```
operation.apply("trolo ipsum"); // 删除字符不匹配，throws an error
```
但是在实际的算法实现中，很多是根据删除字符的长度来进行删除，例如 ot.js 库就是这样处理，在实际协同文本编辑器应用中也基本满足要求了，无需再对删除字符进行比较，一定程度上也提高了处理效率。

### 操作合并(compose)
前面提到：**一个 Operation(操作) 包含多个 action(行为)**，这样有一个好处是：**多个 operation 能合并 (compose) 成一个**。
> ***apply(apply(S, A), B) = apply(S, compose(A, B))***
多个 operation 依次应用等价于，先将多个 operation 合并成一个，然后再进行应用

例如有两个 operation 要应用到一个字符串上，正常的做法是这两个 operation 依次进行应用；现在我们可以先对两个 operation 进行合并成一个 operation，然后再进行应用。
```
// Define two consecutive operations
var operation0 = new ot.Operation()
  .retain(11)
  .insert(" dolor");
var operation1 = new ot.Operation()
  .delete("lorem ")
  .retain(11);

// Our input string
var str0 = "lorem ipsum";
```

依次应用这两个 operation：
```
// Apply operations one after another
var str1 = operation0.apply(str0); // "lorem ipsum dolor"
var str2a = operation1.apply(str1); // "ipsum dolor"
```
先将两个 operation 合并成一个，然后再应用这个 operation：
```
// Combine operations and apply the combined operation
var combinedOperation = operation0.compose(operation1);
var str2b = combinedOperation.apply(str0); // "ipsum dolor"
```

### 操作转换(transform)
OT 算法的核心就是这个 `transform` 转换算法，OT 的思路是将编辑转换成操作 (operation)，当多人协同操作时，就需要对这些操作进行转换，使得最终文档的内容一致。

例如：文档内容是：'go'，两个用户同时在字符串末尾执行字符插入：
```
var str = 'go'
```
Client-A:
```
var opA = new ot.Operation()
  .retain(2)
  .insert("a");

opA.apply(str) // => goa
```
Client-B:
```
var opB = new ot.Operation()
  .retain(2)
  .insert("t");
opB.apply(str) // => got
```

用户对文档的插入操作会立即应用到本地文档副本，然后将操作 `opA` 和 `opB` 上传到服务器；服务器先后接收到这两个操作，然后对服务端的这份文档进行应用，假设以先 `opA` 后 `opB` 的顺序，应用 `opA` 后，字符串变成 `goa`，长度变成 3；然后应用 `opB`，隐形的光标未到字符串末端，执行会抛出错误。
此外，服务端也会将 `opA` 发送给 Client-B，`opB` 发送给 Client-A，由于此时字符串都长度都变成 3，应用过程中隐形的光标未到字符串末端，会抛出错误。但即使能应用成果，结果也是：
- Client-A: 'goa' + (retain(2); insert('t')) = 'gota'
- Client-B: 'got' + (retain(2); insert('a')) = 'goat'

两个用户得到的文档内容也不一致了，这种情况下就需要 `transform` 操作转换了。
![](http://7vikhl.com1.z0.glb.clouddn.com/49EEDFD9-F635-4F55-8DCA-C165E4813AC6.png)

① 如上图左，`a` 表示客户端的 operation，`b` 表示服务端的 operation，二者相交的顶点表示文档状态相同，此时客户端和服务端对文档分别应用操作 `a` 和 `b`，这时两边的文档内容都发生变化，且不一致；为了客户端和服务端的文档达到一致的状态，我们需要对 `a` 和 `b` 进行操作转换 `transfrom(a, b) => (a', b')` 得到两个衍生的操作 `a'` 和 `b'`。
② 如上图右，`a'` 在服务端应用，`b'` 在客户端应用，最终文档内容达到一致。

上面这种问题称之为：**one-step diamond problem**，客户端和服务端同时对文档执行一个 operation，从上方顶点分裂开两条边，相当于 diamond 图形的上方两边（`a`, `b`），通过 OT 算法的 `transform` 方法得到两个衍生的 operation，客户端和服务端分别执行这两个衍生的 operation，相当于 diamond 图形的下方两边（`a'`, `b'`），这样从上方顶点分裂开的两边又汇合到下顶点，构成一个完整的 diamond 图形。diamond 图形的上下两个顶点表示客户端和服务器的文档内容处于相同状态。

![](http://7vikhl.com1.z0.glb.clouddn.com/EE6A20BC-A5EA-4BA4-B7F8-566310464A8D.png)

所以这个例子的正确解法是：
```
var transformedPair = ot.Operation.transform(opA, opB);
var opAPrime = transformedPair[0];
var opBPrime = transformedPair[1];

var strABPrime = opAPrime.apply(strB);
var strBAPrime = opBPrime.apply(strA);
```

### operation 的逆向操作
> The inverse of an operation is the operation that reverts the effects of the operation

例如一个 opration 包含 `insert("hello "); retain(6);` 这两个 action；那么这个 operation 的逆向操作就是 `delete("hello "); retain(6);`
一个 operation 的逆向主要用于 Undo/Redo，在文档编辑中执行了一个 operation，那么需要将其逆向操作推入 undo/redo 栈中，在撤销/回撤时，弹出栈顶 operation 执行。
> undo/redo 入栈的数据除了当前操作的 inverse operation，还包括了选区信息
```
// inverseOperation inverse operation
// Metadata: selection
WrappedOperation(inverseOperation, Metadata)
```

## C/S 模式
绝大多数的协同应用都是 Client/Server(C/S) 模式，下面我们将分析 OT 算法在 C/S 模式下的处理过程。

### compound OT
上一节中介绍了客户端和服务端各有一个操作的情况，通过 `transform` 操作变换，将客户端和服务端的文档内容收敛到相同状态；但是在实际应用场景中，不可能仅仅只有一个操作，而是有多个操作，假设客户端有两个操作，服务端只有一个操作，如下图所示：
![](http://7vikhl.com1.z0.glb.clouddn.com/AC60F950-D30A-44BC-AB0C-19719EA41C5B.png)

此时需要构建两个 diamond 图形才能使客户端和服务端的文档收敛到相同的状态。

![](http://7vikhl.com1.z0.glb.clouddn.com/7F6B6005-8F23-4294-A94C-196EFE124E8A.png)
如上图客户端有三个操作，服务端有一个操作的时候，需要构建三个 diamond 图形能将两端文档内容收敛到相同状态；由此我们可以类推出，当两端操作数出现多对一 `N:1` 或者一对多 `1:N` 的关系时，可以递归构建 diamond 图形来使得两端内容最终收敛到相同状态。
假设已知客户端有一个操作 `clientOp`，服务端有多个操作 `serverOps` ，那么可以通过如下代码，计算出客户端的操作经过变换，在服务端应用的操作 `clientOpPrime`。
```js
var clientOp
var serverOps = []
var clientOpPrime = clientOp
for(var i = 0; i < serverOps.length; i++) {
  clientOpPrime = transform(clientOpPrime, serverOps[i])[0]
}
```

当然，客户端和服务端的操作数也有可能出现多对多 `N:M` 的关系，那么做法是将 `N:M` 的关系拆分成多个 `N:1` 或者 `1:N` 的关系来解决。
![](http://7vikhl.com1.z0.glb.clouddn.com/781C4BCD-1D5C-4AD2-9173-B8CF6CD5C016.png)

### revision(版本号)
- 文档一次编辑前后的差异 (diff) 即为一个 operation 操作
- 每一个操作对应一次版本号顺序递增
- 客户端和服务端各自维持一个版本号
  - 客户端进行一次编辑产生一个 operation 或者从服务端接收到一个 operation，客户端版本号都会递增一
  - 服务端作为客户端并发编辑操作串行化的地点，除了保存文档内容，还会将客户端发过来的 operation 保存到一个数组中，这个数组也就是文档编辑的历史记录。服务端每接收到一个操作，服务端版本号会递增一
  - **客户端版本号始终小于等于服务端版本号**

### C/S 直接的交互
- 客户端将一次编辑操作（例如输入/删除字符，设置文本样式等）封装成一个 operation，通过 socket 将当前客户端版本号 revision 和编辑操作 operation 发送到服务端
- 服务端接收到客户端的信息 `(revision, operation)` 后，执行如下处理函数
```js
receiveOperation (revision, operation) {
  // ① revision check
  if (revision < 0 || this.operations.length < revision) {
    throw new Error('operation revision not in history')
  }
  // ② Find all operations that the client didn't know of when it sent the
  // operation ...
  var concurrentOperations = this.operations.slice(revision)

  // ... and transform the operation against all these operations ...
  for (var i = 0; i < concurrentOperations.length; i++) {
    operation = WrappedOperation.transform(operation, concurrentOperations[i])[0]
  }

  // ③ ... and apply that on the document.
  this.document = operation.apply(this.document)
  // Store operation in history.
  this.operations.push(operation)

  // It's the caller's responsibility to send the operation to all connected
  // clients and an acknowledgement to the creator.
  return operation
}
```

👉 剖析一下这个函数：
- **注释①**：客户端和服务端版本号大小比较，确保*客户端版本号始终小于等于服务端版本号*，否则报错
- **注释②**：客户端操作和服务端操作递归执行 `transform` 操作变换
- **注释③**：将*注释②*结果应用到当前文档，并保存到历史操作数组中
- 最后，将 `receiveOperation (revision, operation)` 这个函数返回的 `operation` 发送给其他协同编辑的客户端

### UNDO/REDO
客户端从服务端接收到一个 `operation` 后，需要使用这个 `operation` 对 UNDO/REDO 栈中的所有操作进行 `transform` 变换
```js
transform (operation) {
  this.undoStack = this._transformStack(this.undoStack, operation)
  this.redoStack = this._transformStack(this.redoStack, operation)
}
_transformStack (stack, operation) {
  var newStack = []
  for (var i = stack.length - 1; i >= 0; i--) {
    var pair = WrappedOperation.transform(stack[i], operation)
    if (typeof pair[0].isNoop !== 'function' || !pair[0].isNoop()) {
      newStack.push(pair[0])
    }
    operation = pair[1]
  }
  return newStack.reverse()
}
```

## OT 富文本支持
关于扩展 ot.js 支持富文本属性，可以参考文件 [TextAction.js]() 和 [TextOperation.js]()，此处不展开介绍（后面再写文章对此介绍）

前面提到服务端保存了两套数据：
- 文档内容，这是一段纯文本字符串
- 文档编辑历史记录，这是一个保存 operation 的数组

注意到文档内容是纯文本，并不包含富文本信息。客户端打开文档时，需要将文档历史记录中所有的 operation 合并成一个 operation（合并逻辑可放在服务端处理，将合并后的 operation 发给客户端），然后对文档内容执行这个 operation，这样才会让文档显示出富文本样式。
```js
initClientContent () {
  // init the editor content with historyOps
  if (this.historyOps && this.historyOps.length) {
    var _ops = this.historyOps.map(wrappedOp => wrappedOp.wrapped)
    var initialTextOp = new TextOperation()

    _ops.forEach((op) => {
      var _textOp = TextOperation.fromJSON(op)
      initialTextOp = initialTextOp.compose(_textOp)
    })

    this.applyOperation(initialTextOp)
  }
}
```

## 参考链接
[Operational-Transformation/ot.js](https://github.com/Operational-Transformation/ot.js)
[Operational Transformation](http://operational-transformation.github.io/index.html)
[Understanding and Applying Operational Transformation](http://www.codecommit.com/blog/java/understanding-and-applying-operational-transformation)
[Operational Transformation算法图解](http://blog.csdn.net/pheecian10/article/details/78496854)
