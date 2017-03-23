### Lombok @ExtensionMethod 오류

[Lombok](https://projectlombok.org/features/index.html)에서 getter/setter/constructor/log 등을 만들어주는 기능을 아주 잘 사용하고 있다. 이제는 lombok이 없으면 개발을 하기 힘들 지경. 새로운 회사에 와서 가장 먼저 pom.xml에 추가하자고 주장했던 것 중에 하나도 lombok이었다.

최근에는 [Lombok Experimental](https://projectlombok.org/features/experimental/index.html)도 관심을 많이 가지고 [@UtilityClass](https://projectlombok.org/features/experimental/UtilityClass.html)와 같은 걸 잘 사용하고 있다. 그러다가 [@ExtensionMethod](https://projectlombok.org/features/experimental/ExtensionMethod.html)를 발견했는데, 처음에는 응??하다가 와!!했다.

```java
@ExtensionMethod({java.util.Arrays.class})
public class LombokExample {
    public String test() {
        int[] intArray = {5, 3, 8, 2};
        intArray.sort();
   }
 }

public class VanillaJava {
    public String test() {
        int[] intArray = {5, 3, 8, 2};
        java.util.Arrays.sort(intArray);
    }
}
```

그런데 정작 적용해보려니까 안되는 것이다 :( 한참 삽질 후..

(시간적인 공백...)

아직 [intellij lombok plugin에서 지원하지 않는 것](https://github.com/mplushnikov/lombok-intellij-plugin/issues/21)이었다. 처음 논의가 시작된 게 2013년인데, 아직도 적용 안된 것은 당황스러웠다.

>  it would be very very very hacky to do (by [Techable](https://github.com/Techcable))

~~이래서 직접 만들어쓰는 오픈소스 커미터들이 생기나보다~~ 