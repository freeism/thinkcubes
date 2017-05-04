### Guava Joiner/splitter 활용

나는 variable type을 정확하게 하는 편을 좋아하는데, 외부API를 연동하게 되면 어쩔 수 없이 텍스트를 다루어야 할 때 생긴다. 그럴 때마다 Guava 유틸 클래스들을 자주 이용하는 편인데, 특히 Joiner는 오래 전부터 사용하고 있었다. 그러다가 최근에 아래 로직을 refactoring하려다보니 Joiner의 반대 기능이 필요했다. 

```java
private Map<String, String> parseHttpResponseToMap(HttpEntity httpEntity) throws IOException {
    // Get response body from entity.
    byte[] entityBytes = EntityUtils.toByteArray(httpEntity);
    String responseBody = (new String(entityBytes, "EUC-KR")).trim();
    
    // Parse key and value to map.
    String[] keyValues = responseBody.split("&");
    Map<String, String> responseMap = new HashMap<>();
    for (String keyValue : keyValues) {
        String[] kv = keyValue.split("=");
        if (kv.length == 1) {
            responseMap.put(kv[0], null);
        } else {
            responseMap.put(kv[0], kv[1]);
        }
    }
    
    return responseMap;
}
```

그래서 찾아봤더니 Guava에 Splitter가 존재했다. 누가 그랬던가 `개발자가 예상한 것처럼 기능이 존재하고 동일하게 동작하는 코드는 아름답다` 라고.

```java
public class JoinerAndSplitterTest {
    @Test
    public void exJoiner() {
        Map<String, Object> map = Maps.newHashMap();
        map.put("name", "freeism");
        map.put("job", "engineer");
        map.put("locale", "ko");

        String result = Joiner.on("&").withKeyValueSeparator("=").join(map);

        System.out.println(result);   // name=freeism&job=engineer&locale=ko
    }

    @Test
    public void exSplitter() {
        String str = "name=freeism&job=engineer&locale=ko";

        Map<String, String> result = Splitter.on("&").withKeyValueSeparator("=").split(str);

        System.out.println(result);   // {name=freeism, job=engineer, locale=ko}
    }

    @Test
    public void exSplitterUsingTrim() {
        String str = "name=freeism &job= engineer&locale=ko ";

        Map<String, String> result1 = Splitter.on("&").withKeyValueSeparator("=").split(str);
        System.out.println(result1);  // {name=freeism , job= engineer, locale=ko }

        Map<String, String> result2 = Splitter.on("&").withKeyValueSeparator(Splitter.on("=").trimResults()).split(str);
        System.out.println(result2);   // {name=freeism, job=engineer, locale=ko}
    }
}
```