### Integer.getInteger()의 잘못된 사용
###### 부제 : 누가 이따구로 네이밍한거야

애시당초에 올바른 데이터타입을 사용했다면 문제가 없었겠지만, 레거시 코드에 붙이다보면 타입캐스팅이 필요한 경우가 있다.
```java
@RequestParam(value = "idx", required = true) String idx,
@RequestParam(value = "value", required = true) String value
```

평상시라면, required=true인 값이기 때문에 apache commons를 이용해서 파싱했을 것이다
```java
int idxAsInt = NumberUtils.toInt(idx);
int valueAsInt = NumberUtils.toInt(value);
```

그런데 이후에 로직이 변경될 경우, default value = 0으로 적용되면 side-effect이 크게 발생할 것 같았다. 
그래서 명시적인 parsing error를 만들기 위해 NumberUtils를 사용하지 않고 Integer 클래스를 사용하기로 했다.
```java
int idxAsInt = Integer.getInteger(idx);
int valueAsInt = Integer.getInteger(value);
```

원래라면 ```Integer.valueOf(idx)```처럼 사용했을텐데, 왜 굳이 저 시점에는 ```Integer.getInteger(idx)```를 썼는지 모르겠다.
valueOf라는 네이밍이 (한국정서상) 와닿지 않는 네이밍이어서 그랬을 수도 있고, 혹은 autocomplete되어 나왔는데 생각없이 썼을 수도 있다.

문제는 NPE가 발생하는 것이다.
```
java.lang.NullPointerException
	at kr.co.freeism.FooServiceTest.test(FooServiceTest.java:45)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
  ...
```

> getInteger(String nm)
  Determines the integer value of the system property with the specified name.
>
> valueOf(String s)
  Returns an Integer object holding the value of the specified String.
>
> parseInt(String s)
  Parses the string argument as a signed decimal integer.
>
> ref. http://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html
  
우리가 일반적으로 원하는 기능은 string인 값을 integer로 변환하는 것인데, 그렇다면 ```Integer.valueOf()```나 ```Integer.parseInt()```를 사용해야 한다.
```Integer.getInteger()```는 system property를 읽어오는 메소드라서 대부분의 경우(?) null을 return할 것이다. 

그래서 NPE가 발생하게 된다.
```java
Integer idxAsInteger = Integer.getInteger(idx);   // -> null
int idxAsInt = Integer.getInteger(idx);   // -> NPE : 결과가 null이기 때문에, auto-unboxing하면서 NPE가 발생함.
```

네이밍은 이처럼 중요하다. 이름만 보고 생각하는 기능대로 동작하지 않는 메소드는 위험하다.
> You know you are working on clean code when each routine you read turns out to be pretty much what you expected. 
  You can call it beautiful code when the code also makes it look like the language was made for the problem.
>
> ref. clean code

이미 아주 오래된 메소드인데, 왜 deprecate가 되지 않는지 궁금하다.
> ref. http://konigsberg.blogspot.kr/2008/04/integergetinteger-are-you-kidding-me.html
