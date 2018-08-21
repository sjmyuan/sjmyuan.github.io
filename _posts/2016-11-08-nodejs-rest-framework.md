---
title: NodeJS REST Framework
---

# Metrics

|Metric|Express|Hapi|Restify|Koa|Loopback|
|:-----|:------|:---|:------|:--|:-------|
|Github Stars|28224|6901|5875|12486|7665|
|Contributors|191|147|142|91|77|
|StackOverflow Questions|178464|263|29|77|29|

# Demo

## Express

~~~~~javascript
var express = require('express');
var Item = require('models').Item;
var app = express();
var itemRoute = express.Router();
 
itemRoute.param('itemId', function(req, res, next, id) {
  Item.findById(req.params.itemId, function(err, item) {
    req.item = item;
    next();
  });
});
 
// Create new Items
itemRoute.post('/', function(req, res, next) {
  var item = new Item(req.body);
  item.save(function(err, item) {
    res.json(item);
  });
});
 
itemRoute.route('/:itemId')
  // Get Item by Id
  .get(function(req, res, next) {
    res.json(req.item);
  })
  // Update an Item with a given Id
  .put(function(req, res, next) {
    req.item.set(req.body);
    req.item.save(function(err, item) {
      res.json(item);
    });
  })
  // Delete and Item by Id
  .delete(function(req, res, next) {
    req.item.remove(function(err) {
      res.json({});
    });
  });
 
app.use('/api/items', itemRoute);
app.listen(8080);
~~~~~

## Hapi
~~~~javascript
var Hapi = require('hapi');
var Item = require('models').Item;
var server = Hapi.createServer('0.0.0.0', 8080);
 
server.ext('onPreHandler', function(req, next) {
  if (req.params.itemId) {
    Item.findById(req.params.itemId, function(err, item) {
      req.item = item;
      next();
    });
  }
  else {
    next();
  }
});
 
server.route([
  {
    path: '/api/items/{itemId}',
    method: 'GET',
    config: {
      handler: function(req, reply) {
        reply(req.item);
      }
    }
  },
  {
    path: '/api/items/{itemId}',
    method: 'PUT',
    config: {
      handler: function(req, reply) {
        req.item.set(req.body);
        req.item.save(function(err, item) {
          reply(item).code(204);
        });
      }
    }
  },
  {
    path: '/api/items',
    method: 'POST',
    config: {
      handler: function(req, reply) {
        var item = new Item(req.body);
        item.save(function(err, item) {
          reply(item).code(201);
        });
      }
    }
  },
  {
    path: '/api/items/{itemId}',
    method: 'DELETE',
    config: {
      handler: function(req, reply) {
        req.item.remove(function(err) {
          reply({}).code(204);
        });
      }
    }
  }
]);
 
server.start();
~~~~

## Restify
~~~~javascript
var restify = require('restify');
var Item = require('models').Item;
var app = restify.createServer()
 
app.use(function(req, res, next) {
  if (req.params.itemId) {
    Item.findById(req.params.itemId, function(err, item) {
      req.item = item;
      next();
    });
  }
  else {
    next();
  }
});
 
app.get('/api/items/:itemId', function(req, res, next) {
  res.send(200, req.item);
});
 
app.put('/api/items/:itemId', function(req, res, next) {
  req.item.set(req.body);
  req.item.save(function(err, item) {
    res.send(204, item);
  });
});
 
app.post('/api/items', function(req, res, next) {
  var item = new Item(req.body);
  item.save(function(err, item) {
    res.send(201, item);
  });
});
 
app.delete('/api/items/:itemId', function(req, res, next) {
  req.item.remove(function(err) {
    res.send(204, {});
  });
});
 
app.listen(8080);
~~~~

## Koa
~~~~javascript
var Koa = require('Koa');
var route = require('Koa-route');
var app = Koa();

// REST API
app.use(route.get('/api/items', function*() {
    this.body = 'Get';
}));
app.use(route.get('/api/items/:id', function*(id) {
    this.body = 'Get id: ' + id;
}));
app.use(route.post('/api/items', function*() {
    this.body = 'Post';
}));
app.use(route.put('/api/items/:id', function*(id) {
    this.body = 'Put id: ' + id;
}));
app.use(route.delete('/api/items/:id', function*(id) {
    this.body = 'Delete id: ' + id;
}));

// all other routes
app.use(function *() {
  this.body = 'Hello world';
});

var server = app.listen(3000, function() {
  console.log('Koa is listening to http://localhost:3000');
});
~~~~

## Loopback
~~~~javascript
var loopback = require('loopback');
var app = module.exports = loopback();
 
var Item = loopback.createModel(
  'Item',
  {
    description: 'string',
    completed: 'boolean'
  }
);
 
app.model(Item);
app.use('/api', loopback.rest());
app.listen(8080);
~~~~

# Pros & Cons

## Express

+ Pros
  + Express has the biggest community out of all the web application frameworks for Node.js
  + It is matured framework, with almost 5 years of development behind it and now has StrongLoop taking control of the repository
  + Little learning curve, Express is nearly a standard for Node.js web application
  + Fully customizable
+ Cons
  + All end points need to be created manually, you end up doing a lot of the same code (or worse, start rolling your own libraries after a while)
  + There is no built in error handling
  + Every end point needs to be tested (or at the very least I recommend that you hit the end points with HTTP consumer to make sure they are actually there and don’t throw 500s)
  + Express has a larger foot print
  + Refactoring becomes painful because everything needs to be updated everywhere
  + Doesn’t come with anything “standard”, have to figure out your own approach

## Hapi

+ Pros
  + Very granular control over request handling
  + Detailed API reference with support for documentation generation
+ Cons
  + As with Express and Restify, hapi gives you great construction blocks, but you are left to your own devices figuring out how to use them.
  + There is a lot less examples or open source applications that use hapi
  + Hapi definitely seems to be more tailored towards bigger or more complex applications

## Restify

+ Pros
  + Automatic DTrace support for all your handlers (if you’re running on a platform that supports DTrace)
  + Doesn’t have unnecessary functionality like templating and rendering
  + Built in throttling
  + Built in SPDY support
+ Cons
  + The cons are all the same with Restify as they are with Express, lots of manual labor.

## Koa
+ Pros
  + Koa has the smallest code. It was started by TJ Holowaychuk, the creator of Express.js. Express was sold to Strongloop (the creators of Loopback.js) who did nothing with it (it was maintained solely by Doug Wilson). It's not stable yet
  + Koa has a small footprint, it is more expressive and it makes writing middleware a lot easier
  + Embracing ES6
+ Cons
  + Koa is still unstable and heavily in development.

## Loopback

+ Pros
  + Very quick RESTful API development
  + Convention over configuration
  + Built in models ready to use
  + RPC support
  + Fully configurable when needed
  + Extensive documentation
  + Fulltime team working on the project
  + Available commercial support
+ Cons
  + Learning curve can be pretty steep because there are so many moving parts
  + Can't find many people with opinions on Loopback. It seems fast to code, but tightly coupled to the database model magically. Every route needs a database model. And people don't seem to like its opinionated convention over configuration style
  + They make their initial documents look great, but they haven't implemented everything, and so they put shell documents out there with a small danger caveat that says it's not implemented

# Conclusion 
I choose to use express, because it is stable and has the biggest community. other frameworks have similar features to express,but has smaller community or is unstable.
