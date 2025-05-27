1. Gather Requirements
2. API Design / DB Design
3. High Level design Components
4. Deep Dive - talking about scaling and edge cases

## Requirements

| Functional                                        | Non Functional                 |
| ------------------------------------------------- | ------------------------------ |
| pass in a along url and return a unique short url | Minimize redirect latency      |
| redirect a short url to a long url                | 100 Million daily active users |
| Analytics for each link, to allow caching         | 1B read request per day        |
| Generate unique short urls                        | 1 - 5 B urls over all          |

1 B reads per day would be 11575 Requests per second, a normal DB would easily be able to handle 50k  to 200k read requests per second
How long is the URL going to be, since we need to store 1-5 B URLs
url that is 6 character long with a-z, A-Z and 0-9 characters will be 26+26+10 characters = 62 character
6 characters long so 62^6 == 56 Billion, more than enough

## API Design

POST /api/urls/shorten
request :{	url: "" }
response :{	url: "" }
the short url can be of format `bit.ly/{hashcode}`

GET /{hashcode}
response: redirect , handled with http 301 status code, but the browser caches all 301
and the caching screws up with the analytics because the request wont even hit our server, 
so we wont be able to count how many times a url was hit

instead use status code 302 instead, redirects but doesnt cache in the browser
```java
@GetMapping("/{code}")
public ResponseEntity<Void> redirectToOriginalUrl(@PathVariable String code) {
    String originalUrl = urlService.find(code);
    //return "redirect:" + originalUrl;
    HttpHeaders headers = new HttpHeaders();
    headers.setLocation(URI.create(originalUrl));
    return new ResponseEntity<>(headers, HttpStatus.MOVED_TEMPORARILY);
}
```

there are 12 k requests per second for redirection

a spring boot api can handle low thousands like 3-4k requests per second, so we will need load balancing over the server that is handling the get requests
## DB Design

```json
{
	shortUrl: string, //PK
    longUrl: string,
    uid: uuid,
    createdAt: date,
    usedCount: integer
}
```
if the long url is very very very long and depending on what the uid is

let size per row be 1kb worst case

5 B records 1kb each ==> 5TB

a single postgres instance can easily handle 32 TB

12k reads per second will require us to think about throughput latency and storage
storage is fine only 5TB data
latency/throughput => minimized with Redis
## High Level Design

![[Pasted image 20250526104103.png]]
with this the read service can be scaled independently of the write service


