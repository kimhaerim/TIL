# SingleTone

> 디자인 패턴에 대해 공부하고자 하고 처음으로 나온 싱클턴 패턴

간단하게 말하면 싱글턴 패턴은 클래스 인스턴스를 하나만 만들고, 그 인스턴스로의 전역 접근을 제공하는 것

- 자원을 많이 잡아먹는 인스턴스가 있다면 유용한 기법

```java
public class Singleton {
    private static Singleton uniqueInstance;

    // 기타 인스턴스 변수

    private Singleton() {}

    public static Singleton getInstance() {
        if(uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return Singleton;
    }
}
```

- 생성자를 private이기 때문에 getInstance() 메소드를 통해서 생성 가능
  - null인 경우에만 생성하고 아니라면 미리 생성된 인스턴스를 반환

### 장점

new 연산자를 통해 고정된 메모리 영역을 사용하므로 메모리 낭비 방지
다른 클래스 간의 데이터 공유가 쉬움

### 단점

멀티쓰레딩 문제 발생 :동기화 처리가 되지 않았을 때, 2개의 인스턴스가 생길 수 있음
