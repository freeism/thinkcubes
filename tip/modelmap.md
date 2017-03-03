### ModelAndView의 addObject() 분석

최근에는 API서버만 개발하다보니 RestContoller만 다루다가, 간만에 ModelAndView를 다룰 일이 생겼다. 오랜만에 ModelAndView를 사용하니 Object를 어떻게 추가해야하는지 기억이 가물가물해서 찾아본 김에 기록을 남긴다.

ModelAndView를 다룰 때는 단순히 `return new ModelAndView("/foo");`와 같이 쓰는 경우는 드물고, 대부분 아래처럼 object parameter를 mapping해서 사용하게 된다.
```java
@Controller
public class FooController {
    @RequestMapping("/foo1")
    public ModelAndView foo1() {
      return new ModelAndView("/foo").addAllObjects(createMap());
    }
  
    @RequestMapping("/foo2")
    public ModelAndView foo2() {
      return new ModelAndView("/foo").addObject("foo", new FooModel());
    }
    
    @RequestMapping("/foo3")
    public ModelAndView foo3() {
      return new ModelAndView("/foo").addObject(new FooModel());
    }
    
    private Map<String, Object> createMap() {
      Map<String, Object> map = Maps.newHashMap();
  
      map.put("name", "이름");
      map.put("age", 20);
  
      return map;
    }
  
    @Data
    static class FooModel {
      private String name = "이름";
      private int age = 20;
    }
}
```

위의 addObject 세가지 케이스가 어떻게 다른지 확인을 해봤다.

우선, addObject를 할 때 들어가는 곳은 `ModelAndView.model`이다. 이 모델은 `ModelMap`이라는 객체타입인데, 결국 아래와 같이 `LinkedHashMap`으로 만들어져있다.
```java
public class ModelMap extends LinkedHashMap<String, Object> {
    ...
}
```

그래서 `addAllAttributes(Map)`라는 간단한 메소드를 제공할 수 있다. 입력되는 object의 type이 Map이라면 위에 정의된 ModelMap에 값을 모두 복사하면 되기 때문이다.
```java
/**
 * Copy all attributes in the supplied {@code Map} into this {@code Map}.
 * @see #addAttribute(String, Object)
 */
public ModelMap addAllAttributes(Map<String, ?> attributes) {
    if (attributes != null) {
      putAll(attributes);
    }
    return this;
}
```

그리고 아래처럼 key, value의 형태로 object mapping하는 게 가장 흔한 방법이다. Map이 아닌 일반적인 Object의 경우에는 key, value형태로 mapping하기 때문에 Map에는 key와 value형태로 들어가게 된다.
```java
/**
 * Add the supplied attribute under the supplied name.
 * @param attributeName the name of the model attribute (never {@code null})
 * @param attributeValue the model attribute value (can be {@code null})
 */
public ModelMap addAttribute(String attributeName, Object attributeValue) {
    Assert.notNull(attributeName, "Model attribute name must not be null");
    put(attributeName, attributeValue);
    
    return this;
}
```

마지막으로 아래처럼 그냥 object만으로 mapping하는 케이스가 있다. 여러 가지 복잡한(?) 구현이 되어 있지만, 결국 하는 일은 object의 class shortName을 key로 넣고, object를 value로 넣어서 mapping하고 있다.
```java
/**
 * Add the supplied attribute to this {@code Map} using a
 * {@link org.springframework.core.Conventions#getVariableName generated name}.
 * <p><emphasis>Note: Empty {@link Collection Collections} are not added to
 * the model when using this method because we cannot correctly determine
 * the true convention name. View code should check for {@code null} rather
 * than for empty collections as is already done by JSTL tags.</emphasis>
 * @param attributeValue the model attribute value (never {@code null})
 */
public ModelMap addAttribute(Object attributeValue) {
    Assert.notNull(attributeValue, "Model object must not be null");
    
    if (attributeValue instanceof Collection && ((Collection<?>) attributeValue).isEmpty()) {
      return this;
    }
    
    return addAttribute(Conventions.getVariableName(attributeValue), attributeValue);
}
```

```java
public static String getVariableName(Object value) {
    Assert.notNull(value, "Value must not be null");
    boolean pluralize = false;
    Class valueClass;
    
    if(value.getClass().isArray()) {
      valueClass = value.getClass().getComponentType();
      pluralize = true;
    } else if(value instanceof Collection) {
      Collection name = (Collection)value;
      
      if(name.isEmpty()) {
        throw new IllegalArgumentException("Cannot generate variable name for an empty Collection");
      }

      Object valueToCheck = peekAhead(name);
      valueClass = getClassForValue(valueToCheck);
      pluralize = true;
    } else {
      valueClass = getClassForValue(value);
    }

    String name1 = ClassUtils.getShortNameAsProperty(valueClass);
    
    return pluralize ? pluralize(name1) : name1;
}

private static String pluralize(String name) {
    return name + "List";
}
```

소스를 보다가 추가로 알게 된 사실이 있다. Array나 Collection 타입의 경우에는 class shortName에 추가로 List가 붙어서 attributes가 된다. 해당 로직이 오래 되었나보다. 요즘엔 List보다는 plural type으로 그냥 s를 붙이는 경우가 많기 때문이다.