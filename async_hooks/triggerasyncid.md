
`triggerAsyncId` is the `asyncId` of the resource that caused (or "triggered")
the new resource to initialize and that caused `init` to call. This is different
from `async_hooks.executionAsyncId()` that only shows *when* a resource was
created, while `triggerAsyncId` shows *why* a resource was created.

The following is a simple demonstration of `triggerAsyncId`:

```js
const { fd } = process.stdout;

async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = async_hooks.executionAsyncId();
    fs.writeSync(
      fd,
      `${type}(${asyncId}): trigger: ${triggerAsyncId} execution: ${eid}\n`);
  }
}).enable();

net.createServer((conn) => {}).listen(8080);
```

Output when hitting the server with `nc localhost 8080`:

```console
TCPSERVERWRAP(5): trigger: 1 execution: 1
TCPWRAP(7): trigger: 5 execution: 0
```

The `TCPSERVERWRAP` is the server which receives the connections.

The `TCPWRAP` is the new connection from the client. When a new
connection is made, the `TCPWrap` instance is immediately constructed. This
happens outside of any JavaScript stack. (An `executionAsyncId()` of `0` means
that it is being executed from C++ with no JavaScript stack above it.) With only
that information, it would be impossible to link resources together in
terms of what caused them to be created, so `triggerAsyncId` is given the task
of propagating what resource is responsible for the new resource's existence.

