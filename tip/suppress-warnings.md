### @SuppressWarnings 활용

#### Overview
나는 `@SuppressWarnings`를 별로 좋아하지 않는다. 코드를 잘 만들면 되는데, 굳이 정적분석을 무시할 필요가 없다는 생각때문이다. 그런데 코딩을 하다보니 반드시 써야할 경우가 생겼다. 단순히 IDE에서 노란 밑줄을 보고 무시해도 되지만, 불필요한 경고가 너무 많아지면 정말 필요한 경고를 무시하게 되는 경향이 생긴다. [깨진 유리창 이론](https://en.wikipedia.org/wiki/Broken_windows_theory)은 굳이 언급하지 않아도 될 정도로 유명하다.
그래서 이번 기회에 해당 내용을 정리해두려고 한다.

#### Definition
[`@SuppressWarnings`](https://docs.oracle.com/javase/7/docs/api/java/lang/SuppressWarnings.html)는 컴파일러가 정적분석을 진행할 때 오류가 아니라고 마킹해주는 역할을 하고 있다. JDK 1.5 버전부터 지원하다보니, value값이 String으로 선언되어 있다. Enum이 아니다보니 생각보다 쓰기가 쉽지 않다. (심지어 오타가 나거나 틀려도 오류없이, 그냥 SuppressWarnings만 되지 않는다)

* all : 모든 경고를 억제
* boxing : boxing/unboxing 오퍼레이션과 관련된 경고를 억제
* cast : 캐스트 오퍼레이션과 관련된 경고를 억제
* dep-ann : 권장되지 않는 어노테이션과 관련된 경고를 억제
* deprecation : 권장되지 않는 기능과 관련된 경고를 억제
* fallthrough : switch 문에서 누락된 break 문과 관련된 경고를 억제
* finally : 리턴되지 않는 마지막 블록과 관련된 경고를 억제
* hiding : 변수를 숨기는 로컬과 관련된 경고를 억제
* incomplete-switch : switch 문에서 누락된 항목과 관련된 경고를 억제(enum case)
* javadoc : javadoc 경고와 관련된 경고를 억제
* nls : 비nls 문자열 리터럴과 관련된 경고를 억제
* null : 널(null) 분석과 관련된 경고를 억제
* rawtypes : 원시 유형 사용법과 관련된 경고를 억제
* resource : 닫기 가능 유형의 자원 사용에 관련된 경고 억제
* restriction : 올바르지 않거나 금지된 참조 사용법과 관련된 경고를 억제
* serial : 직렬화 가능 클래스에 대한 누락된 serialVersionUID 필드와 관련된 경고를 억제
* static-access : 잘못된 정적 액세스와 관련된 경고를 억제
* static-method : static으로 선언될 수 있는 메소드와 관련된 경고를 억제
* super : 수퍼 호출을 사용하지 않는 메소드 겹쳐쓰기와 관련된 경고를 억제
* synthetic-access : 내부 클래스로부터의 최적화되지 않은 액세스와 관련된 경고를 억제
* sync-override : 동기화된 메소드를 오버라이드하는 경우 누락된 동기화로 인한 경고 억제
* unchecked : 미확인 오퍼레이션과 관련된 경고를 억제
* unqualified-field-access : 규정되지 않은 필드 액세스와 관련된 경고를 억제
* unused : 사용하지 않은 코드 및 불필요한 코드와 관련된 경고를 억제

#### Example

실제 적용 코드는 아래와 같다. 
아래처럼 JSP에 사용되고 있는 모델이 있는데, 날짜와 시간을 분리해서 사용해야 하는 코드였다.

```java
@Data
public class PaymentRequest {
    private String expiredDate;
    private String expiredTime;
}
```

물론 해당 코드는 정상적으로 동작하고 있고, 문제가 없었다. 하지만 **변수는 직관적인 데이터형을 사용하고, 실제 사용하는 곳에서 casting해서 사용해야 된다**는 생각이 있었다. 그래서 아래처럼 `LocalDateTime`형으로 변경하였다.

```java
@Data
public class PaymentRequest {
    private LocalDateTime expiredAt;
}
```

위 모델을 JSP에서 날짜와 시간으로 분리해서 사용해도 문제가 없으나, **JSP는 최대한 간결하게 만들자**는 개인적인 원칙이 적용되었다.

```java
@Data
public class PaymentRequest {
    private LocalDateTime expiredAt;

    public String getDateExpiredAt() {  // use by jsp
        return expiredAt.format(DateTimeFormatter.ofPattern("yyyyMMdd"));
    }

    public String getTimeExpiredAt() {  // use by jsp
        return expiredAt.format(DateTimeFormatter.ofPattern("HHmm"));
    }
}
```

문제는 JSP에서 사용하는 변수라서 참조가 걸리지 않아서 계속 `never used`라며 경고 메시지가 노출된다. 문제는 해당 코드가 참조되지 않기 때문에 지워지기 쉬운 코드가 되어버렸다는 것이다. 나 또한, 다른 사람이 만들어 놓은, 사용되지 않는 코드에 대해 삭제하고 싶은 욕구가 강하게 들곤 한다. 그래서 `// used by jsp`라고 주석을 달았지만 썩 만족스럽지 않다.

```java
@Data
public class PaymentRequest {
    private LocalDateTime expiredAt;

    @SuppressWarnings("unused")
    public String getDateExpiredAt() {  // use by jsp
        return expiredAt.format(DateTimeFormatter.ofPattern("yyyyMMdd"));
    }

    @SuppressWarnings("unused")
    public String getTimeExpiredAt() {  // use by jsp
        return expiredAt.format(DateTimeFormatter.ofPattern("HHmm"));
    }
}
```

위처럼 `@SuppressWarnings("unused")`를 사용해서 정적오류를 제거했다.

#### Conclusion

모든 코드는 제작자의 의도를 남겨야 하고, 해당 코드는 사용하고 있는 코드임을 확실하게 표시해줄 필요가 있다고 생각한다. 물론 비정상적인 코드를 만들고 `@SuppressWarnings`를 남발하는 것은 비추천한다.

> [참고사이트](http://www.ibm.com/support/knowledgecenter/ko/SS5JSH_9.1.1/org.eclipse.jdt.doc.user/tasks/task-suppress_warnings.htm)
