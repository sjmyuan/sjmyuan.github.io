---
title: NodeJs Promise
---
+ async work was trigged when new promise
+ operation in Constructor function is async
+ done is used to register callback
+ then will new a promise, the promise's constructor will invoke done
+ every child promise will be triggered when parent is resolved or rejected
