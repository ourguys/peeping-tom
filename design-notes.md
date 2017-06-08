# API DESIGN NOTES

### Considerations

Since peeping-tom doesn't use twitter's API to access the service, we have to be very cautious about rate limiting it. This creates the issue of who manages the processing of a batch of request. If it is the client, the server only needs to process and respond to one request at a time. If it is the server, the client can send a batch of requests and the server needs to provide a protocol for the client to get the results. 

If the client handles the logic for requesting a processing of batch users/tweets, the service has can rate limit and doesn't have to handle any progressive updates (updating the client as new portions of the batch gets completed instead of waiting for the whole batch to be completed to return a result) but the server would have to send a response for every process request, and the client has to do some sort of coordination (maybe as simple as serializing requests) to make sure that the server responds before a new request is made so that it doesn't affect the service rate limit against twitter. This latter issue is that like: the service has a rate limit it cannot succeed and it manages this rate limit (10 requests per 20 seconds), the client cannot make more requests than more requests the server can make against twitter. The client does not know the rate it should use. If there is just a single client, the server just needs to inform its own rate. If there are multiple clients, then the issue becomes of distributing that rate across multiple clients. This seems like a dead end of coordination hell. 

Another approach: the service accepts the request, passes to the client a token which the client can then use to check the status of the request. the service would distribute through its own logic its rate limit against twitter, populating a buffer which is consumed by the client through using that token. All the client needs to do is then poll or wait until the buffer delivers results. If heroku allows for because it isn't blocking ports, we don't have to use a restful API and we instead can stream the response through using tcp. This would make the task easier for the client to consume the out because the client can then just wait for a response. 

### Data design
for requests on user pages
```(json)
  {
    "username": "blah", 
    "likes": 23000, 
    "tweets": 2000, 
    "followers": 200,
    "following": 20,
    "account-age": <unix-epoc-date>
  } 
```
- as a transport format, we can simplify and reduce the size by JSONifyng an array. The fields will be implicit and based on order. 

### other ideas 
- for optimization, maybe peeping-tom should cache results using a rolling caching scheme where we cache a fixed amount and expire items out of the cache as we add new stuff. 
