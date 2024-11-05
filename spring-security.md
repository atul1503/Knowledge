# Tips, best practices and important info to work with spring security.

1. You can disable csrf if using jwt.
2. Cors need to be enabled. Mobile app wil ignore cors but browser will use the cors security. Just make sure that cors allows all hosts and methods.
3. Session based auth is where the server keeps track of the session and the  browser automatically sends the session id in a cookie in all requests to the same server.
4. In Token based auth, the token itself is sufficient for the server to identify or authenticate the user. Tokens are sent to client for storage and the server doesnot store them so that is why token based auth is stateless. Tokens are also not automatically sent by the browser in each request  so the hacker has to find out the token and send it with any harmful request to the same server which is impossible unless they got the token in the transit. To prevent this stealing of token in transit we can use https.
5. Token have to be sent as a authorization header to the server explicitly with each request.
6. In jwt, browser or any http client can never automatically put authorization bearer tokens because browsers dont remember or store that like a cookie. Cookies are also stored only if the backend server sends a "set cookie" header on the response of a login or signup request. So you can ignore csrf by using jwt in authorization bearer header of your evwry request. Cors attack is also very much impossible since even if attacker sends a request from some other origin, he wont have the jwt token to authenticate. Hacker can get hold of the token if stored insecurely in the browser but that is the problem of the client or client's browser not the backend.
7. To enables https you have to just make changes in the  `application.properties` file and create a keystore file which includes a private key and certificate using either keygen or using certificate authority. Keygen is only used for self signed certificates.
Keystore file is a file which stores private keys and its associated certificate. It has `.jceks` extension.
8. In spring security, an authentication filter tries to populate the security context holder object with an authentication object if the user is authenticated. The filters that come after that filter may use the security context holder to know whether request is authenticated or not by simply checking the authentication object present in the security context holder.
Even this authentication object is used in authorization process to check if the authneticated user has access a certain resouce or not using the granted authority on it.
9. When using jwt, session creation policy should be stateless. This means that that the security context which contains the authentication object is not stored in an session and most importantly no session is created ever.
10. When using session creation policy as stateless or never, no `set cookie` header is sent in the response.
11. In spring security, the filter that is authorizing your paths or request uri path is executed after authentication is done. So the permitAll() or authenticated() are evaluated after authentication is done. Spring always authenticates all requests. If not authenticated using the upstream filters like username and password filter or its replacement then spring automatically authenticates those request with anonymous authentication using anonymous authentication filter.
12. If the configuration that you have defined says permitAll() for any path then all requests whether properly authenticated or with anonymous authentication is allowed to  access the path. But if is configured with authenticated() then only requests authenticated properly instead of anonymous authentication are allowed to access. There is also a anonymous() method which allows only anonymous authenticated requests to be allowed to access corresponding path.
13. For jwt, always subclass the once per request filter and in the doFilterInternal method, provide your authentication logic where at the end you set an auhtnetication object in security context and put this context into the security context holder. And put this custom filter at the place of username and password authentication filter. This filter will authenticate requests. This will also allow this custom filter to exist before the anonymous authentication filter.


# JWT setup

JWT is the best way to implement security.

## pom.xml dependencies

Make sure to have these dependencies.

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-api</artifactId>
			<version>0.11.5</version>
		</dependency>
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-impl</artifactId>
			<version>0.11.5</version>
		</dependency>
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-jackson</artifactId>
			<version>0.11.5</version>
		</dependency>
```

