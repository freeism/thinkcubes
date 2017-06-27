### Swagger에서 httpMethod multiple-value를 지원하지 않음

아무런 문제가 없어보이는 코드에서 오류가 발생했다.
```java
2017-06-27 10:43:11.540 ERROR 4008 --- [on(2)-127.0.0.1] s.d.s.r.o.OperationHttpMethodReader      : Invalid http method: GET,POSTValid ones are [[Lorg.springframework.web.bind.annotation.RequestMethod;@5f82b5c8]

java.lang.IllegalArgumentException: No enum constant org.springframework.web.bind.annotation.RequestMethod.GET,POST
	at java.lang.Enum.valueOf(Enum.java:238)
	at org.springframework.web.bind.annotation.RequestMethod.valueOf(RequestMethod.java:35)
	at springfox.documentation.swagger.readers.operation.OperationHttpMethodReader.apply(OperationHttpMethodReader.java:49)
	at springfox.documentation.spring.web.plugins.DocumentationPluginsManager.operation(DocumentationPluginsManager.java:123)
	at springfox.documentation.spring.web.readers.operation.ApiOperationReader.read(ApiOperationReader.java:73)
	at springfox.documentation.spring.web.scanners.CachingOperationReader$1.load(CachingOperationReader.java:50)
	at springfox.documentation.spring.web.scanners.CachingOperationReader$1.load(CachingOperationReader.java:48)

```

분명 HttpMethod는 multi value를 지원하는 값이다.
```java
/**
 * The HTTP request methods to map to, narrowing the primary mapping:
 * GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE.
 * <p><b>Supported at the type level as well as at the method level!</b>
 * When used at the type level, all method-level mappings inherit
 * this HTTP method restriction (i.e. the type-level restriction
 * gets checked before the handler method is even resolved).
 * <p>Supported for Servlet environments as well as Portlet 2.0 environments.
 */
RequestMethod[] method() default {};
```

나중에야 사용하고 있는 swagger에서 multi value를 지원하지 않는 것을 알게 되어 기록을 남겨둔다.
```java
@ApiOperation(
    httpMethod = "GET,POST",
    value = "모바일결제 완료 응답, 웹뷰응답"
)
@RequestMapping(value = "/mobile/next", method = { RequestMethod.GET, RequestMethod.POST })
```
```java
/**
 * Corresponds to the `method` field as the HTTP method used.
 * <p>
 * If not stated, in JAX-RS applications, the following JAX-RS annotations would be scanned
 * and used: {@code @GET}, {@code @HEAD}, {@code @POST}, {@code @PUT}, {@code @DELETE} and {@code @OPTIONS}.
 * Note that even though not part of the JAX-RS specification, if you create and use the {@code @PATCH} annotation,
 * it will also be parsed and used. If the httpMethod property is set, it will override the JAX-RS annotation.
 * <p>
 * For Servlets, you must specify the HTTP method manually.
 * <p>
 * Acceptable values are "GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS" and "PATCH".
 */
String httpMethod() default "";
```

> you can currently only have one HTTP verb per op.
https://github.com/swagger-api/swagger-ui/issues/183