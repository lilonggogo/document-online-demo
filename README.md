# document-online-demo
it's demo
## 实现思路：

#### 1. 服务器端：
   - 创建一个服务器应用，使用Node.js和Express框架。
   - 在服务器上存储文档内容，可以选择使用数据库或文件系统。
   - 使用实时通信库 Socket.io 来处理客户端与服务器之间的实时通信。
   - 监听客户端的文档更新事件，并将更新的文档内容广播给所有连接的客户端。

#### 2. 客户端：
   - 创建一个React应用，用于呈现文档编辑器和实时同步功能。
   - 在客户端使用富文本编辑器组件 ReactQuill 来提供富文本编辑功能。
   - 连接到服务器上的实时通信服务 Socket.io 以接收文档更新事件。
   - 监听编辑器内容的变化，并在内容更改时触发文档更新事件发送给服务器。
   - 接收来自服务器的文档更新事件，并更新本地的文档内容。

#### 3. 实时同步：
   - 当一个客户端在编辑器中进行更改时，将更改的内容发送到服务器。
   - 服务器接收到更新后的内容后，广播给所有连接的客户端。
   - 客户端接收到文档更新事件后，更新本地的文档内容，以便与服务器上的文档保持同步。
   - 这样，无论哪个客户端更改了文档内容，其他连接的客户端都会实时看到更新的内容。

上述是一个简单的实现思路，实际的在线文档应用可能需要更多的功能和复杂的处理逻辑。例如，可以添加用户认证、权限管理、版本控制等功能，以满足实际应用的需求。

将来还需要考虑文档的持久化和恢复、处理冲突以及性能优化等方面的问题

#### 以下是`socket.io-client`的基本用法：

1.  安装 `socket.io-client`：

    Copy

    ````
    npm install socket.io-client
    ```
    ````

1.  引入 `socket.io-client`：

    javascript

    ````
    import io from 'socket.io-client';
    ```
    ````

1.  连接到Socket.io服务器：

    javascript

    ````
    const socket = io('http://your-socket-io-server-url');
    ```

    替换 `'http://your-socket-io-server-url'` 为实际的Socket.io服务器URL。
    ````

1.  监听服务器发送的事件：

    javascript

    ````
    socket.on('event_name', (data) => {
      // 处理收到的数据
    });
    ```

    使用`socket.on`方法来注册对特定事件的监听器，当服务器发送相应事件时，回调函数将被执行。
    ````

1.  发送事件到服务器：

    javascript

    ````
    socket.emit('event_name', data);
    ```

    使用`socket.emit`方法发送事件到服务器，其中 `'event_name'` 是事件的名称，`data` 是要发送的数据。
    ````

1.  断开连接：

    javascript

    ````
    socket.disconnect();
    ```

    使用`socket.disconnect`方法手动断开与服务器的连接。
    ````
    
#### 在 Node.js 应用中使用 Socket.io服务端建立连接并监听客户端事件。

用`http`模块创建了一个 `httpServer` 实例，并使用 `socket.io` 将其升级为 WebSocket 服务器。创建一个 `users` 集合用于存储所有已连接的用户的 `socket.id`。当有用户连接时，将其 `socket.id` 添加到 `users` 集合中，并向此用户发送一个 `connectUser` 事件，并向所有已连接的用户广播一个 `users` 事件，以更新在线用户列表。

当有用户断开连接时，它会从 `users` 集合中删除该用户的 `socket.id`，并向所有已连接的用户广播一个 `users` 事件，以更新在线用户列表。

```
js
复制代码
const httpServer = require('http').createServer();
const io = require('socket.io')(httpServer, {
  cors: {
    origin: '*',
  },
});

const users = new Set();
let existingText = ''
let editingUser = '';

io.on('connection', (socket) => {
  console.log('a user connected');

  users.add(socket.id);
  socket.emit('connectUser', socket.id);
  socket.emit('text', existingText);
  io.emit('users', Array.from(users));

  socket.on('text', (newText) => {
    existingText = newText;
    io.emit('text', newText);
  });

  socket.on('editing', (userId) => {
    editingUser = userId
    io.emit('editing', userId)
  })

  socket.on('disconnect', () => {
    console.log('user disconnected');
    users.delete(socket.id);
    if (editingUser === socket.id) {
      editingUser = '';
      io.emit('editing', '');
    }
    io.emit('users', Array.from(users));
  });
});

httpServer.listen(3333, () => {
  console.log('listening on *:3333');
});
```

`socket.emit` 是用于发送事件到指定的 `socket` 对象，只有该 `socket` 对象会收到该事件，其他任何 `socket` 对象都不会收到该事件。

`io.emit` 是用于向所有连接到服务器的 `socket` 对象广播事件，所有连接到服务器的 `socket` 对象都会收到该事件。

在实现在线文档协作编辑的场景中，使用 `io.emit` 可以确保所有连接到服务器的用户都能够收到其他用户所做的更改，从而保证协作效率。

### 在文档的操作过程如何处理冲突

介绍两种流行的同步算法：操作转换（OT，Operational Transformation）和冲突无关数据类型（CRDT，Conflict-free Replicated Data Type）。

## 操作转换（OT）

操作转换（OT）是一种实时协同编辑技术，它允许多个用户在网络环境中同时编辑共享文档。、这样当多个操作同时发生时，可以在不引起冲突的情况下正确合并。

### OT的基本原理

假设有两个用户A和B同时编辑一个文档。用户A在文档中插入字符 'a'，而用户B在同一位置插入字符 'b'。为了使这两个操作正确合并，我们可以使用转换函数将这些操作转换为相对于对方操作的形式。这样，最终的文档将包含两个字符 'ab' 或 'ba'，而不是出现冲突。

我们需要对用户A的操作进行转换，以适应用户B的操作。在这个例子中，我们可以将用户A的操作转换为在 "hb" 之间插入字符 "a"。最终的文档内容将变为 "habello"。

```
javascript
// 转换函数
function transform(clientOp, serverOp) {
  if (clientOp.position < serverOp.position) {
    return clientOp;
  } else if (clientOp.position > serverOp.position) {
    return {
      ...clientOp,
      position: clientOp.position + 1,
    };
  } else {
    // 当操作位置相同时，根据用户ID或操作的时间戳等信息对操作进行排序。
    // 在这个简化的示例中，我们仅比较操作中的字符。
    if (clientOp.character < serverOp.character) {
      return clientOp;
    } else {
      return {
        ...clientOp,
        position: clientOp.position + 1,
      };
    }
  }
}

// 初始文档
let document = "hello";

// 用户A和B的操作
const userAOperation = { type: "insert", position: 1, character: 1703138499941 };
const userBOperation = { type: "insert", position: 1, character: 1703138499966 };

// 使用转换函数处理用户A的操作
const transformedUserAOperation = transform(userAOperation, userBOperation);

// 应用操作到文档
document = document.slice(0, userBOperation.position) + userBOperation.character + document.slice(userBOperation.position);
document = document.slice(0, transformedUserAOperation.position) + transformedUserAOperation.character + document.slice(transformedUserAOperation.position);

console.log(document); // 输出 "habello"
```

1.  首先定义了一个`transform`函数，它接受两个操作（`clientOp`和`serverOp`）作为输入。根据操作的位置决定如何转换客户端操作，以适应服务器操作。
1.  然后，我们初始化文档为字符串`hello`。
1.  接着，我们定义了用户A和B的插入操作。每个操作都有一个类型（`insert`）、一个插入位置和一个插入字符。
1.  使用`transform`函数处理用户A的操作。在此示例中，我们将用户A的操作转换为在"hb"之间插入字符"a"。
1.  我们将转换后的操作应用于文档，首先应用用户B的操作，然后应用经过转换的用户A的操作。
1.  最后，我们将结果输出到控制台。在这种情况下，输出结果为`habello`。

### CRDT

当用户A和B执行插入操作时，他们将生成带有新标识符的新字符。例如，用户A插入 "a"，标识符为1；用户B插入 "b"，标识符为2。最后，文档将包含以下字符序列：h(0), a(1), b(2), e(3), l(4), l(5), o(6)。这样，无论操作的顺序如何，最终的文档内容都将是 "habello"。

```
javascript
复制代码
class Char {
  constructor(value, position, author) {
    this.value = value;
    this.position = position;
    this.author = author;
  }
}

// 应用操作函数
function applyOperation(document, operation) {
  const index = document.findIndex((char) => char.position >= operation.char.position);
  document.splice(index, 0, operation.char);
}

// 初始文档
let document = "hello".split("").map((char, index) => new Char(char, index, "initial"));

// 用户A和B的操作
const userAOperation = { type: "insert", char: new Char("a", 0.1, "userA") };
const userBOperation = { type: "insert", char: new Char("b", 0.2, "userB") };

// 应用操作到文档
applyOperation(document, userBOperation);
applyOperation(document, userAOperation);

// 转换文档回字符串并输出
const result = document.map((char) => char.value).join("");
console.log(result); // 输出 "habello"
```

1.  首先定义了一个`Char`类，用于表示文档中的字符。每个`Char`对象包含字符值、位置和作者信息。
1.  接下来定义了一个`applyOperation`函数，该函数接受文档和操作作为输入。函数根据操作中字符的位置信息将字符插入到正确的位置。
1.  然后初始化文档。在此示例中，我们将字符串`hello`转换为一个`Char`对象数组，每个对象包含字符值、位置和作者信息。
1.  接着定义了用户A和B的插入操作。每个操作都有一个类型（`insert`）和一个`Char`对象。
1.  将操作应用于文档，首先应用用户B的操作，然后应用用户A的操作。
1.  最后将文档转换回字符串并输出结果。输出结果为`habello`。

## 对比后的结论

OT和CRDT是解决协同编辑中实时同步问题的两种主要技术。它们各自具有优缺点，适用于不同的场景和需求。在选择合适的技术时，您需要权衡实现复杂性、性能、一致性等多个方面的因素。

OT算法在实现实时同步和强一致性方面具有优势，但实现较为复杂，且当用户数量增加时可能面临性能挑战。相反，CRDT提供了一种简单且可扩展的同步解决方案，但可能需要更多的存储空间和通信带宽，且只能保证最终一致性。
