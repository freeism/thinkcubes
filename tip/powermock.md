### PowerMock 사용

예전에 [PowerMock사용법(final class의 테스트)](http://freeism.co.kr/tc/693?category=3)라는 제목으로 블로깅을 한 적 있다. 그게 2011년 7월이니까 근 6년 가까이 PowerMock을 사용할 일이 생기지 않았다. 단순히 [SpringRunner](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/context/junit4/SpringRunner.html)나 [Mockito](http://site.mockito.org/)만으로도 충분한 테스트를 만들 수 있었다.

그런데 최근에 작업중인 프로젝트에서 시간에 의존하는 테스트가 존재했다. 그래서 평소에는 Ignore시키고 있다가 필요시(?)에 테스트를 돌리는 식으로 되어 있었다.
```java
public class DateUtilTest {
    @Test
    @Ignore("실제 종료시간에는 테스트가 깨진다")
    public void isCurrentTimeIsBetweenMidnightEndOfTimeTest() {
        assertFalse("자정부터 서비스 종료시간 사이 시간입니다.", dateUtil.isCurrentTimeIsBetweenMidnightEndOfTime());
    }
}
```

그래서 아래처럼 PowerMock을 이용해서 LocalDateTime 객체를 mocking하게 되었다. 시간에 대한 정보가 로직에 있는 경우에는 아래처럼 테스트해야 되는 케이스가 많이 발생할 것 같다.
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({DateUtil.class, LocalDateTime.class})
public class DateUtilTest {
    private static final LocalDate NOW = LocalDate.now();
    private static final LocalDate YESTERDAY = NOW.minusDays(1);
    private static final LocalDateTime TODAY_ONE_OCLOCK = LocalDateTime.of(NOW, LocalTime.of(1, 0, 0));
    private static final LocalDateTime TODAY_SIX_OCLOCK = LocalDateTime.of(NOW, LocalTime.of(6, 0, 0));
  
    @Test
    public void isCurrentTimeIsBetweenMidnightEndOfTimeIfYesterdayAndBeforeEndTime() {
      PowerMockito.mockStatic(LocalDateTime.class);
      when(LocalDateTime.now()).thenReturn(TODAY_ONE_OCLOCK);
  
      boolean result = DateUtil.isCurrentTimeIsBetweenMidnightEndOfTime(YESTERDAY);
  
      assertThat(result, is(true));
    }
  
    @Test
    public void isCurrentTimeIsBetweenMidnightEndOfTimeIfYesterdayAndAfterEndTime() {
      PowerMockito.mockStatic(LocalDateTime.class);
      when(LocalDateTime.now()).thenReturn(TODAY_SIX_OCLOCK);
  
      boolean result = DateUtil.isCurrentTimeIsBetweenMidnightEndOfTime(YESTERDAY);
  
      assertThat(result, is(false));
    }
}
```

:exclamation: 주의할 점은 `PrepareForTest`에서 `LocalDateTime.class`뿐 아니라 `DateUtil.class`(테스트대상객체)도 함께 등록되어야 한다는 사실이었다. 이것때문에 삽질을 좀 했다. 만약 테스트대상객체가 함께 등록되지 않으면, Test Class에서 `LocalDateTime.now()`를 찍어보면 mocking된 정보로 나오지만, 테스트객체에서 찍어보면 실제 정보로 표시된다.
