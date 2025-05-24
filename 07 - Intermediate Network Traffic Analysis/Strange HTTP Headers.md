Fuzzing isn't the only indicative that something bad is happening. We might also look for strange behavior among HTTP requests, like:
- Weird Hosts
- Unusual HTTP Verbs
- Changed User Agents

## Finding Strange Host Headers

In Wireshark, let's filter for HTTP and let's exclude the packets where the host is our server's IP address. We might notice some packets where the host field is "127.0.0.1" or "admin". This is done in order to attempt to gain levels of access the attackers would not normally achieve through the legitimate host.
To prevent successful exploitation, do the following:
1. Ensure that our virtualhost or access configurations are setup correctly to prevent this form of access.
2. Ensure that our web server is up to date.

## Analyzing Code 400s and Request Smuggling

We might also see bad requests from the client. They are obviously a good place to start when detecting malicious actions.
Look at this crazy type of injection.
![[5-http-headers.webp]]
```txt
GET%20%2flogin.php%3fid%3d1%20HTTP%2f1.1%0d%0aHost%3a%20192.168.10.5%0d%0a%0d%0aGET%20%2fuploads%2fcmd2.php%20HTTP%2f1.1%0d%0aHost%3a%20127.0.0.1%3a8080%0d%0a%0d%0a%20HTTP%2f1.1 Host: 192.168.10.5
```

This will be decoded in our server like this.
```url-decoded
GET /login.php?id=1 HTTP/1.1
Host: 192.168.10.5

GET /uploads/cmd2.php HTTP/1.1
Host: 127.0.0.1:8080

 HTTP/1.1
Host: 192.168.10.5
```

This is referred to as HTTP request smuggling or CRLF (Carriage Return Line Feed).
If we find a code 200 in response to one of these, we cooked.
