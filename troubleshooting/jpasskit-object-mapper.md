### JPassKit 적용중 오류 발생

서비스에서 ios wallet을 제공하려고 하니, 예전과는 다르게 서버단 통신을 통해 인증받는 절차가 추가로 생겼단다.
다만, 애플에서 제공하는 서버쪽 [데모](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/PassKit_PG/index.html#//apple_ref/doc/uid/TP40012195)를 보면 ruby로 만들어져있다.
왜 하필 루비인가? swift도 아니고... 여튼 그걸 java로 porting하려니 이미 만들어 놓은 것이 있을 것 같아서 구글링했더니,
[jpasskit](https://www.google.co.kr/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiU65GN3q3RAhWBQZQKHbGAAZYQFgggMAA&url=https%3A%2F%2Fgithub.com%2Fdrallgood%2Fjpasskit&usg=AFQjCNH6lvlwdYkacoqqVValEtwGNUYG3w)이
그나마 제일 fork도 많이 되고, 사용도 하는 것 같아서 lib dependency를 추가했다.

```xml
<!-- PassKit -->
<dependency>
    <groupId>de.brendamour</groupId>
    <artifactId>jpasskit</artifactId>
    <version>0.0.8</version>
</dependency>
```

개발을 완료했는데, Test Case에서 오류가 나타나기 시작했다.

```bash
com.fasterxml.jackson.databind.JsonMappingException: Can not resolve PropertyFilter with id 'validateFilter'; no FilterProvider configured
```

난 jackson filter를 바꾼 적이 없는데 왜 에러가 나는 것인가?
처음에는 [jpasskit issue](https://github.com/drallgood/jpasskit/issues/38)를 보고 jackson lib의 version 호환성 문제가 있는 것 같아서
아래처럼 dependency처리를 했다.

```xml
<!-- PassKit -->
<dependency>
    <groupId>de.brendamour</groupId>
    <artifactId>jpasskit</artifactId>
    <version>0.0.8</version>
    <exclusions>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

위의 오류가 해결된 것처럼 보여서 SNAPSHOT version을 만들었는데, 됐다안됐다한다.
예를 들어서 local profile에서 하면 되고, develop profile에서 하면 오류나고...
혹은 전체 junit을 모두 돌리면 에러가 발생하는데, 에러나는 class만 테스트 돌리면 성공하고 ㅠ.ㅠ

그래서 해당 소스를 파보다가 문제점을 발견하였다.

우리의 프로젝트에서는 pojo type인 jackson object mapper를 bean으로 등록해서 사용하고 있다.
bean으로 등록하면 몇 가지 장점이 있는데, 자세한 설명은 이 글의 범위를 벗어나기 때문에 생략한다.

```java
@Primary
@Bean
public ObjectMapper objectMapper() {
    ObjectMapper objectMapper = new CustomObjectMapper();
    initializeObjectMapper(objectMapper);
  
    return objectMapper;
}
```

그래서 Object Mapper는 singleton으로 재사용하고 있는데, jpasskit은 Object Mapper를 변조시키고 있다.

```java
public final class PKFileBasedSigningUtil extends PKAbstractSIgningUtil {
    private static final String FILE_SEPARATOR_UNIX = "/";
    private static final String MANIFEST_JSON_FILE_NAME = "manifest.json";
    private static final String PASS_JSON_FILE_NAME = "pass.json";
    private ObjectWriter objectWriter;
  
    @Inject
    public PKFileBasedSigningUtil(ObjectMapper objectMapper) {
        this.addBCProvider();
        this.objectWriter = this.configureObjectMapper(objectMapper);
    }
...
```

```java
protected ObjectWriter configureObjectMapper(ObjectMapper jsonObjectMapper) {
    jsonObjectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
    jsonObjectMapper.setDateFormat(new ISO8601DateFormat());
    SimpleFilterProvider filters = new SimpleFilterProvider();
    filters.addFilter("validateFilter", SimpleBeanPropertyFilter.serializeAllExcept(new String[]{"valid", "validationErrors"}));
    filters.addFilter("pkPassFilter", SimpleBeanPropertyFilter.serializeAllExcept(new String[]{"valid", "validationErrors", "foregroundColorAsObject", "backgroundColorAsObject", "labelColorAsObject", "passThatWasSet"}));
    filters.addFilter("barcodeFilter", SimpleBeanPropertyFilter.serializeAllExcept(new String[]{"valid", "validationErrors", "messageEncodingAsString"}));
    filters.addFilter("charsetFilter", SimpleBeanPropertyFilter.filterOutAllExcept(new String[]{"name"}));
    jsonObjectMapper.setSerializationInclusion(Include.NON_NULL);
    jsonObjectMapper.addMixIn(Object.class, PKAbstractSIgningUtil.ValidateFilterMixIn.class);
    jsonObjectMapper.addMixIn(PKPass.class, PKAbstractSIgningUtil.PkPassFilterMixIn.class);
    jsonObjectMapper.addMixIn(PKBarcode.class, PKAbstractSIgningUtil.BarcodeFilterMixIn.class);
    jsonObjectMapper.addMixIn(Charset.class, PKAbstractSIgningUtil.CharsetFilterMixIn.class);
    return jsonObjectMapper.writer(filters);
}
```

확실해졌다. 위에서 상황마다 오류가 간헐적으로 발생하는 이유는 이와 같은 것이었다.
jpasskit이 실행되기 전까지는 정상적으로 동작한다.
그러다가 jpasskit을 한 번 거치면 이미 등록되어 있는 object mapper bean의 설정이 바뀌게 된다.
즉, 우리가 설정한 custom configuration들이 무시되어버려서, 전혀 엉뚱한 곳에서 에러를 일으킨다.

jpasskit에서 사용하는 object mapper는 특별한 설정이 필요한 것은 아니라, bean을 사용하지 않고 기본 object mapper를 생성해서 넘기는 식으로 수정하였다.
```java
private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

...

private byte[] createPKPassBinaries(PKPass pass, PKSigningInformation pkSigningInformation, InputStream thumbnail, InputStream thumbnail2x) throws Exception {
    return new PKFileBasedSigningUtil(OBJECT_MAPPER).createSignedAndZippedPkPassArchive(pass, createPKPassTemplate(thumbnail, thumbnail2x), pkSigningInformation);
}
```

All Clear.

해당 내용은 jpasskit에 [issue reporting](https://github.com/drallgood/jpasskit/issues/76)하여 신규 release(0.0.9)가 예정중이다 ^0^
