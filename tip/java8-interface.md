### Java8에서 interface 변경

자바8에서 interface에 [default](http://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)라는 지시자가 추가되었다. 원래 자바의 interface에는 구현체가 들어갈 수가 없었다. 느슨한 결합에 대한 최소한의 제약 사항과 상수정도를 지정해줄 수 밖에 없었다.
```java
public interface Vehicle {
    int countOfWheels = 4;
    
    void startEngine();
    void stopEngine();
}
```

만약 여러 가지 객체에서 사용되는 공통 메소드를 지정하고, 필요한 경우에만 다르게 지정하고 싶으면 abstract class를 사용해야만 했다.
```java
public abstract class Car implements Vehicle {
    @Override
    public void startEngine() {
      System.out.println("부르릉");
    }
    
    @Override
    public void stopEngine() {
      System.out.println("털털털");
    }
}

public class Sedan extends Car {
    ...
}

public class Suv extends Car {
    ...
}

public class SportsCar extends Car {
    @Override
    public void startEngine() {
      System.out.println("부아아앙");
    }
   
    ...
}
```

그런데 자바8에서 추가된 default라는 지시자가 생기면서 바로 interface에 기본 구현체를 지정할 수 있게 되었다.
```java
public interface Car extends Vehicle {
    default void startEngine() {
      System.out.println("부르릉");
    }
    
    default void stopEngine() {
      System.out.println("털털털");
    }
}

public class Sedan implements Car {
    ...
}

public class Suv implements Car {
    ...
}

public class SportsCar implements Car {
    @Override
    public void startEngine() {
      System.out.println("부아아앙");
    }
   
    ...
}
```

사실 interface 자체가 약간 결합도가 높아진(?) 경향이 생겨서, abstract class와의 차이점을 크게 느끼지 못하겠다. 다만, effective java 혹은 clean code에서 원체 extends를 지양하고, implements를 사용하라는 말이 많았었는데 혹시 그런 영향이 아닐까 조심스레 생각해본다. 보통은 현상에 의해서 룰이 생기게 되는데, 간혹 룰에 의해서(룰을 맞추기 위해서) 역으로 현상이 맞춰 가는 경우가 생기기도 하기 때문이다.
> Item18: Prefer interfaces to abstract classes (by effective java)

어찌됐든 간에, 결국 자바도 다중상속의 위험에 노출(~~원래도 안되는 것은 아니었으나~~)되게 되었다. 
```java
public interface CellPhone {
    Bell getBell();

    default void sound() {
        System.out.println("ring, ring, ring~");
    }
}

public interface MusicPlayer {
    Bell getBell();

    default void sound() {
        System.out.println("music, music, music~");
    }
}

public class IPhone implements CellPhone, MusicPlayer {
    @Override
    public Bell getBell() {
      return new Bell();
    }
}
```

다행히도 다중상속의 경우 Runtime이 아니라 Compile 단계에 오류를 생성해준다.
```bash
Error:(7, 8) java: class kr.co.freeism.foo.IPhone inherits unrelated defaults for sound() from types kr.co.freeism.foo.CellPhone and kr.co.freeism.foo.MusicPlayer
```

당연한 얘기겠지만, default에 의한 메소드를 선택할 수 없기 때문에 직접 구현체를 만들어 주면 된다.

```java
public class IPhone implements CellPhone, MusicPlayer {
    @Override
    public Bell getBell() {
      return new Bell();
    }
    
    @Override
    public void sound() {
      System.out.println("ring, music, ring, music~");
    }
}
```

혹은, default에 의한 메소드를 참조해줄 수도 있다.
```java
public class IPhone implements CellPhone, MusicPlayer {
    @Override
    public Bell getBell() {
      return new Bell();
    }
    
    @Override
    public void sound() {
      CellPhone.super.sound();
      MusicPlayer.super.sound();
    }
}
```

실제로 실무에서 한 번 적용해봤는데, 굳이 abstract class를 만들고 싶지 않았던 케이스에서 간단하게 사용할 수 있어서 좋았다.
```java
public interface BookingRequest {
    ServiceType getServiceType();
    String getCouponCode();
  
    default boolean isAvailableServiceType() {
      return getServiceType().isAvailable();
    }
  
    default boolean isExistCouponCode() {
      return StringUtils.isNotEmpty(getCouponCode());
    }
    
    ...
}
```