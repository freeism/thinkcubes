### Base64 encode 와 encodeUrlSafe 차이

https://ko.wikipedia.org/wiki/베이스64

> 컴퓨터 분야에서 쓰이는 Base 64 (베이스 육십사)란 8비트 이진 데이터(예를 들어 실행 파일이나, ZIP 파일 등)를 문자 코드에 영향을 받지 않는 공통 ASCII 영역의 문자들로만 이루어진 일련의 문자열로 바꾸는 인코딩 방식을 가리키는 개념이다.
>  
> 원래 Base 64를 글자 그대로 번역하여 보면 64진법이란 뜻이다. 특별히 64진법이 컴퓨터에서 흥미로운 것은, 64가 2의 제곱수(64 = 26)이며, 2의 제곱수들에 기반한 진법들 중에서 화면에 표시되는 ASCII 문자들을 써서 표현할 수 있는 가장 큰 진법이기 때문이다. 즉, 다음 제곱수인 128진법에는 128개의 기호가 필요한데 화면에 표시되는 ASCII 문자들은 128개가 되지 않는다.
>  
> 그런 까닭에 이 인코딩은 전자 메일을 통한 이진 데이터 전송 등에 많이 쓰이고 있다. Base 64에는 어떤 문자와 기호를 쓰느냐에 따라 여러 변종이 있지만, 잘 알려진 것은 모두 처음 62개는 알파벳 A-Z, a-z와 0-9를 사용하고 있으며 마지막 두 개를 어떤 기호를 쓰느냐의 차이만 있다.

그런데 기존의 base64 mapping table을 보면, '+'와 '/' 문자가 존재한다. 이런 문자들은 http 통신을 할 때에 각각 공백과 path로 오해할 수 있는 부분이라서 문제가 발생한다. 그래서 urlsafe방식이 나오게 되었는데, '+'문자와 '/'문자를 각각 '-'문자와 '_'문자로 치환하여 사용한다.

```java
package java.util;

public class Base64 {
    public static class Encoder {

        /**
         * This array is a lookup table that translates 6-bit positive integer
         * index values into their "Base64 Alphabet" equivalents as specified
         * in "Table 1: The Base64 Alphabet" of RFC 2045 (and RFC 4648).
         */
        private static final char[] toBase64 = {
            'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M',
            'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z',
            'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm',
            'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z',
            '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '+', '/'
        };

        /**
         * It's the lookup table for "URL and Filename safe Base64" as specified
         * in Table 2 of the RFC 4648, with the '+' and '/' changed to '-' and
         * '_'. This table is used when BASE64_URL is specified.
         */
        private static final char[] toBase64URL = {
            'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M',
            'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z',
            'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm',
            'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z',
            '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '-', '_'
        };
    }
}
```

mapping table이 다르기때문에, 같은 base64라 하더라도 urlsafe와 standard는 혼용하여 사용할 수 없다.

```java
@Test
public void foo() {
    String sample = "한미르";
  
    String encoded1 = Base64Utils.encodeToString(sample.getBytes());
    System.out.println("encoded1 : " + encoded1);   // encoded1 : 66+8
  
    String encoded2 = Base64Utils.encodeToUrlSafeString(sample.getBytes());
    System.out.println("encoded2 : " + encoded2);   // encoded2 : 66-8
  
    String decoded1 = new String(Base64Utils.decodeFromString(encoded1));
    System.out.println("decoded1 : " + decoded1);   // decoded1 : 민
  
    String decoded2 = new String(Base64Utils.decodeFromUrlSafeString(encoded2));
    System.out.println("decoded1 : " + decoded2);   // decoded1 : 민
  
    try {
        String error1 = new String(Base64Utils.decodeFromUrlSafeString(encoded1));
        System.out.println("error1 : " + error1);
    } catch (Exception e) {
        e.printStackTrace();    // java.lang.IllegalArgumentException: Illegal base64 character 2b
    }
  
    try {
       String error2 = new String(Base64Utils.decodeFromString(encoded2));
       System.out.println("error2 : " + error2);
    } catch (Exception e) {
        e.printStackTrace();    // java.lang.IllegalArgumentException: Illegal base64 character 2b
    }
}
```