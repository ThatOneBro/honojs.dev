---
title: Routing
weight: 200
---

# Routing

Routing of Hono is flexible and intuitive.
Let's take a look.

## Basic

```ts
// HTTP Methods
app.get('/', (c) => c.text('GET /'))
app.post('/', (c) => c.text('POST /'))
app.put('/', (c) => c.text('PUT /'))
app.delete('/', (c) => c.text('DELETE /'))

// Wildcard
app.get('/wild/*/card', (c) => {
  return c.text('GET /wild/*/card')
})

// Any HTTP methods
app.all('/hello', (c) => c.text('Any Method /hello'))
```

## Named Parameter

```ts
app.get('/user/:name', (c) => {
  const name = c.req.param('name')
  ...
})
```

or all parameters at once:

```ts
app.get('/posts/:id/comment/:comment_id', (c) => {
  const { id, comment_id } = c.req.param()
  ...
})
```

## Optional Parameter

```ts
// Will match `/api/animal` and `/api/animal/:type`
app.get('/api/animal/:type?', (c) => c.text('Animal!'))
```

## Regexp

```ts
app.get('/post/:date{[0-9]+}/:title{[a-z]+}', (c) => {
  const { date, title } = c.req.param()
  ...
})
```

## Chained route

```ts
app
  .get('/endpoint', (c) => {
    return c.text('GET /endpoint')
  })
  .post((c) => {
    return c.text('POST /endpoint')
  })
  .delete((c) => {
    return c.text('DELETE /endpoint')
  })
```

## Grouping

Group the routes with the Hono instance and add them to the main app with the route method.

```ts
const book = new Hono()

book.get('/', (c) => c.text('List Books')) // GET /book
book.get('/:id', (c) => {
  // GET /book/:id
  const id = c.req.param('id')
  return c.text('Get Book: ' + id)
})
book.post('/', (c) => c.text('Create Book')) // POST /book

const app = new Hono()
app.route('/book', book)
```

## Routing priority

Handlers or middleware will be executed in registration order.

```ts
app.get('/book/a', (c) => c.text('a')) // a
app.get('/book/:slug', (c) => c.text('common')) // common
```

```
GET /book/a ---> `a`
GET /book/b ---> `common`
```

When a handler is executed, the process will be stopped.

```ts
app.get('*', (c) => c.text('common')) // common
app.get('/foo', (c) => c.text('foo')) // foo
```

```
GET /foo ---> `common` // foo will not be dispatched
```

If you have the middleware that you want to execute, write the code above the handler.

```ts
app.use('*', logger())
app.get('/foo', (c) => c.text('foo'))
```

If you want a "_fallback_" handler, write the code below the other handler.

```ts
app.get('/foo', (c) => c.text('foo')) // foo
app.get('*', (c) => c.text('fallback')) // fallback
```

```
GET /bar ---> `fallback`
```
