---
layout: post
title:  "Create request pipelne in JS"
date:   2016-05-06 23:50:00
categories: main
tags :
     - programming
     - javascript
---

Recently, I am working on some Node/JS based projects(AutoRest, Azure-SDK-for-Node). 
The core functionality is HTTP requests.
Assume you are sending a HTTP request and before this request I would like to setup some config for this set. for example the request credential, logging stragty, retry policy and so on. So our request lib should be configable to let user config their requests.
To demonstrate a request pipeline:
```
        ||||||||||||                            ||||||||||||||||||||||||||||||
------->|  request | ---| filter | filter | --> | send request, get response | --> | filter | filter | -->
        ||||||||||||                            ||||||||||||||||||||||||||||||
```

For example, I will need a logging filter for my request which will log out info in two cases. 1. when I init a request, log out the request body 2. Once there is a response, log out the response data. 

### How to create request pipeline in Python(or other sync lang)
Create a request pipeline base class which provide a before request hook and after request hook.
```python
class Pipeline():
     def __init__(self):
          self.before = []
          self.after = []
     
     def __before(self, req):
          for filter in self.before:
               request = filter(req)
          return req
     def __after(self):
          pass
     
     def request(self, req):
          pass
     
     def run(self):
          request = self.__before(request)
          res = self._request(req)

```

### Create the same thing in JS?
This is no long working in JS since most of its operations are async. What's the issue? you will not get the result/response util the callback. You cannot running every filter one by one you will need to start the second one after receive the first one's finish callback. 
```javascript
var one = function(resource, done) {
     // do something
     done(resource);
}

var two = function(resource, done) (
}

one(resouce, two);
```
     
