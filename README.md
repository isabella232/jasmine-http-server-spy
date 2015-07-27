# jasmine-http-server-spy [![Build Status](https://drone.io/bitbucket.org/atlassian/jasmine-http-server-spy/status.png)](https://drone.io/bitbucket.org/atlassian/jasmine-http-server-spy/latest)

> Creates jasmine spy objects backed by a http server. Designed to help you write integration tests where your code 
makes real http requests to a server, where the server is controlled via the familiar jasmine spy api.
  
 
## Install

```
$ npm install --save jasmine-http-server-spy
```

## Usage

```coffee
# Integration jasmine spec
jasmineHttpServerSpy = require 'jasmine-http-server-spy'

describe 'Test', ->
    beforeAll (done) ->
        @httpSpy = jasmineHttpServerSpy.createSpyObj('mockServer', [
            {
                method: 'get'
                url: '/some-url-to-mock'
                handlerName: 'getSomeUrlToMock'
            }
        ])
        @httpSpy.server.start 8082, done
    
    afterAll (done) ->
        @httpSpy.server.stop done
       
    afterEach (done) ->
        @httpSpy.getSomeUrlToMock.calls.reset()
        
    it 'all the things', (done) ->
        # 1. Define what mock server would return
        @httpSpy.getSomeUrlToMock.and.returnValue 
            code: 200
            body: 
                data: 10
        # 2. ... calls to main service that uses 'http://localhost:8082/some-url-to-mock'
        # 3. Assert mock server has been called as expected
        expect(@httpSpy.getSomeUrlToMock).toHaveBeenCalled()
        # or
        expect(@httpSpy.getSomeUrlToMock).toHaveBeenCalledWith jasmine.objectContaining(
            body: 
                data: "something"
        )
```

## API

### Handler's expected output

Handler function result will end up in the http response mock server gives back. 
You can define ```code``` and ```body``` at the moment:
 
```coffee
httpSpy.getSomeUrlToMock.and.returnValue {code: 200, body: {data: []}}
# or
httpSpy.getSomeUrlToMock.and.returnValue {code: 401, body: {message: 'Please login first'}}
```

### Handler's input

While handlers are jasmine spy objects, you can define a callback function to make response dynamic. For example:

```coffee
httpSpy.getAnswerForANumber.and.callFake (req) ->
    code: 200
    body:
        if req.body.number is 42
            {answer: 'The answer to the ultimate question of life, the universe and everything'}
        else
            {answer: "I don't know"}
```

You can expect following properties in the first argument of this callback:
 
#### body

JS Object representing JSON body of a request. This object defaults to ```{}```.
 
#### query

Object containing all query parameters used. This object defaults to ```{}```.

#### originalUrl

Requested original URL. For example request to ```http://localhost:8082/mockService/users?something``` end up as 
```/mockService/users?something``` in ```originalUrl```

#### headers

Object containing all headers provided with request.

#### params

An object containing properties mapped to the named route "parameters". 
For example, if you have the route ```/user/:name```, then the "name" property is available as ```req.params.name```. 
This object defaults to ```{}```.

## Contribute

Feel free to fork it here https://bitbucket.org/atlassian/jasmine-http-server-spy/fork and make a pull request. 
 Issues and suggestions can be added here https://bitbucket.org/atlassian/jasmine-http-server-spy/issues
