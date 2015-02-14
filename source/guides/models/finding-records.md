The Ember Data store provides a simple interface for finding records of a single
type through the `store` object's `find` method.

The first argument to `store.find()` is always the record type. The optional second
argument determines if a request is made for all records, a single record, or a query.

## Finding All Records of a Type
To find all records for a type, call find with only the record type parameter:

```javascript
App.PostRoute = Ember.Route.extend({
  model: function() {
    return this.store.find('post');
  }
});
```

`find` returns a `DS.PromiseArray` that fulfills to a `DS.RecordArray`
containing all the records returned by the adapter.

It's important to note that `DS.RecordArray` is not a JavaScript array.
It is an object that implements [`Ember.Enumerable`][1]. This is important
because, if you want to retrieve records by index, the `[]` notation
will not work--you'll have to use `objectAt(index)` instead.

[1]: /api/classes/Ember.Enumerable.html

#### Server Response to a FindAll request

When using the default `RESTAdapter` and `RESTSerializer` Ember Data
will perform the following steps to attempt to request the `post` data
from your server.

First the `RESTAdapter` will attempt to build the URL it will use to
request the `post` data. It does this by pluralizing and camelCasing
the record type name. For the above example, `post` would become the
url `/posts`. The `RESTAdapter` will then make a `GET` XHR (or ajax)
request to the `/posts` url.

Once the `GET /posts` requests is fufilled the adapter will return the
data from the response to Ember Data's store. The store will use the
`RESTSerializer` to extract the data and normalize it into a form the
store can understand.

Given the following model definitions:

```js
App.Post = DS.Model.extend({
  title: DS.attr('string'),
  comments: DS.hasMany('comment')
});

App.Comment = DS.Model.extend({
  body: DS.attr('string'),
  post: DS.belongsTo('post')
});
```

The `RESTSerializer` would expect the response from your server to
look like the following json:

```json
{
  "post": [{
    "id": 1,
    "title": "Rails is omakase",
    "comments": [1, 2]
  }],
  "comments": [{
    "id": 1,
    "body": "FIRST",
    "post": 1,
  }, {
    "id": 2,
    "body": "Rails is unagi",
    "post": 1
  }]
}
```

One thing you may notice is this response contains the data for three
records instead of only the `post` record that was originally
requested. This is a pattern called "sideloading". Sideloading allows
your application to reduce the number of HTTP requestes necessary to
load data by also including related records in the response. In the
above example we can see the post response is nested under the `post`
key and related comment records are nested under the `comments` key.

On the individual records in your payload, the `RESTSerializer` expects
every record to have a unique (per type) identifyer under the `id`
property. It also expects all of the attributes and relationship to
use the same property names in the payload as the property names on
the model.



#### Customizing the Adapter and Server FindAll request
asdf

### Finding a Single Record

If you provide a number or string as the second argument to `store.find()`,
Ember Data will assume that you are passing in an ID and attempt to retrieve a record of the type passed in as the first argument with that ID. This will
return a promise that fulfills with the requested record:

```javascript
var aSinglePost = this.store.find('post', 1); // => GET /posts/1
```

### Querying For Records

If you provide a plain object as the second argument to `find`, Ember Data will
make a `GET` request with the object serialized as query params. This method returns
`DS.PromiseArray` in the same way as `find` with no second argument.

For example, we could search for all `person` models who have the name of
`Peter`:

```javascript
var peters = this.store.find('person', { name: "Peter" }); // => GET to /persons?name=Peter
```
