# axios-rest-resource [![Build Status](https://travis-ci.org/keenondrums/axios-rest-resource.svg?branch=master)](https://travis-ci.org/keenondrums/axios-rest-resource)

Schema-based HTTP client powered by axios. Built with Typescript. Heavily inspired by AngularJS' `$resource`.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Installation](#installation)
- [Quick start](#quick-start)
- [URL token substituion](#url-token-substituion)
- [Custom resource schema](#custom-resource-schema)
- [In depth](#in-depth)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Installation

```
npm i axios-rest-resource axios
```

## Quick start

- Create resource module in your utils folder

  ```ts
  // utils/resource.ts
  import { ResourceBuilder } from 'axios-rest-resource'

  export const resourceBuilder = new ResourceBuilder({
    baseURL: 'http://localhost:3000',
  })
  ```

- Using a newly created resource builder create an actual resource

  ```ts
  // api/entity1.js
  import { resourceBuilder } from 'utils/resource'

  export const entity1Resource = resourceBuilder.build('/entity1')
  // exports an object
  // {
  //   create: (requestConfig) => axiosPromise // sends POST http://localhost:3000/entity1,
  //   read: (requestConfig) => axiosPromise // sends GET http://localhost:3000/entity1,
  //   readOne: (requestConfig) => axiosPromise // sends GET http://localhost:3000/entity1/{id},
  //   remove: (requestConfig) => axiosPromise // sends DELETE http://localhost:3000/entity1/{id},
  //   update: (requestConfig) => axiosPromise // sends PUT http://localhost:3000/entity1/{id}
  // }
  ```

- Use your resource whenever you want to make an AJAX call

  ```ts
  import { entity1Resource } from 'api/entity1'

  const resRead = entity1Resource.read()
  // sends GET http://localhost:3000/entity1
  // resRead is a Promise of data received from the server

  const resReadOne = entity1Resource.readOne({ params: { id } })
  // for id = '123'
  // sends GET http://localhost:3000/entity1/123
  // resReadOne is a Promise of data received from the server

  const resCreate = entity1Resource.create({ data })
  // for data = { field1: 'test' }
  // sends POST http://localhost:3000/entity1 with body { field1: 'test' }
  // resCreate is a Promise of data received from the server

  const resUpdate = entity1Resource.update({ data, params: { id } })
  // for data = { field1: 'test' } and id = '123'
  // sends PUT http://localhost:3000/entity1/123 with body { field1: 'test' }
  // resUpdate is a Promise of data received from the server

  const resRemove = entity1Resource.remove({ params: { id } })
  // for id = '123'
  // sends DELETE http://localhost:3000/entity1/123
  // resRemove is a Promise of data received from the server
  ```

## URL token substituion

axios-rest-resource applies [interceptorUrlFormatter](src/url-formatter.ts) interceptor by default. It handles {token} substitution in URLs.

## Custom resource schema

- Create resource module in your utils folder

  ```ts
  // utils/resource.ts
  import { ResourceBuilder } from 'axios-rest-resource'

  export const resourceBuilder = new ResourceBuilder({
    baseURL: 'http://localhost:3000',
  })
  ```

- Using a newly created resource builder create an actual resource

  ```ts
  // api/entity2.js
  import { resourceSchemaDefault } from 'axios-rest-resource'
  import { resourceBuilder } from 'utils/resource'

  export const entity2Resource = resourceBuilder.build('/entity2', {
    ...resourceSchemaDefault,
    doSomething: {
      method: 'post',
      url: '/do-something',
    },
  })
  // exports an object
  // {
  //   create: (requestConfig) => axiosPromise // sends POST http://localhost:3000/entity2,
  //   read: (requestConfig) => axiosPromise // sends GET http://localhost:3000/entity2,
  //   readOne: (requestConfig) => axiosPromise // sends GET http://localhost:3000/entity2/{id},
  //   remove: (requestConfig) => axiosPromise // sends DELETE http://localhost:3000/entity2/{id},
  //   update: (requestConfig) => axiosPromise // sends PUT http://localhost:3000/entity2/{id},
  //   doSomething: (requestConfig) => axiosPromise // sends POST http://localhost:3000/entity2/do-something
  // }
  ```

- Use your resource whenever you want to make an AJAX call

  ```ts
  import { entity2Resource } from 'api/entity2'

  const resRead = entity2Resource.read()
  // sends GET http://localhost:3000/entity2
  // resRead is a Promise of data received from the server

  const resReadOne = entity2Resource.readOne({ params: { id } })
  // for id = '123'
  // sends GET http://localhost:3000/entity2/123
  // resReadOne is a Promise of data received from the server

  const resCreate = entity2Resource.create({ data })
  // for data = { field1: 'test' }
  // sends POST http://localhost:3000/entity2 with body { field1: 'test' }
  // resCreate is a Promise of data received from the server

  const resUpdate = entity2Resource.update({ data, params: { id } })
  // for data = { field1: 'test' } and id = '123'
  // sends PUT http://localhost:3000/entity2/123 with body { field1: 'test' }
  // resUpdate is a Promise of data received from the server

  const resRemove = entity2Resource.remove({ params: { id } })
  // for id = '123'
  // sends DELETE http://localhost:3000/entity2/123
  // resRemove is a Promise of data received from the server

  const resDoSomething = entity2Resource.doSomething()
  // sends POST http://localhost:3000/entity2/do-something
  // resDoSomething is a Promise of data received from the server
  ```

You custom schema does not need to extend default schema if you do not want that

```ts
// api/entity.js
import { resourceBuilder } from 'utils/resource'

export const entityResource = resourceBuilder.build('/entity', {
  doSomething: {
    method: 'post',
    url: '/do-something',
  },
})
// exports an object
// {
//   doSomething: (requestConfig) => axiosPromise // sends POST http://localhost:3000/entity/do-something
// }
```

Alternatively you can use a partial of a default schema

```ts
// api/entity.js
import { resourceSchemaDefault } from 'axios-rest-resource'
import { resourceBuilder } from 'utils/resource'

const { read, readOne } = resourceSchemaDefault

export const entityResource = resourceBuilder.build('/entity', {
  read,
  readOne,
})
// exports an object
// {
//   read: (requestConfig) => axiosPromise // sends GET http://localhost:3000/entity,
//   readOne: (requestConfig) => axiosPromise // sends GET http://localhost:3000/entity/{id},
// }
```

## In depth

What does `ResourceBuilder` do exactly upon creation?

When you call `new ResourceBuilder(axiosConfig)`

1.  If your `axiosConfig` doesn't have `headers.Accept` property it sets it to 'application/json'.
1.  It creates a new instance of axios passing `axiosConfig` to `axios.create`.
1.  It adds `interceptorUrlFormatter` to request interceptors of the newly created instance of axios.
1.  It exposes the newly created instance of axios for further modifications at `axiosInstance`.

Each instance of ResourceBuilder has its own `axiosInstance`. It's useful if you want to do something more with your axios instance like adding an interceptor.

```ts
import { ResourceBuilder } from 'axios-rest-resource'
import axios, { AxiosInstance } from 'axios'

const resourceBuilder = new ResourceBuilder({
  baseURL: 'http://localhost:3000',
})
resourceBuilder.axiosInstance.interceptors.response.use(myCustomResponeInterceptor)

export { resourceBuilder }
```
