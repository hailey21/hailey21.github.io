---
title:  "추상 클래스보다는 인터페이스를 우선하라"
date: 2022-11-17 22:55:21 +0900
categories: Study 
---

# 아이템 20. 추상 클래스보다는 인터페이스를 우선하라

자바의 다중 구현 매커니즘은 인터페이스/추상클래스 두가지이며 자바 8부터 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다. 

# 추상 클래스

- 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 함
- 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어렵다
    
    → 같은 추상클래스를 확장하길 원하는 경우 그 추상 클래스는 계층구조상 두 클래스의 공통 조상이어야 하며, 이 방식은 큰 혼란을 일으킴 : 모든 자손이 강제로 이를 상속하게 되기 때문
    

# 인터페이스

- 다른 어떤 클래스를 상속했든 같은 타입으로 취급

## 인터페이스의 장점

1. 기존 클래스에 손쉽게 구현 가능 
    
    기존 클래스에 새로운 인터페이스를 구현하려면, 간단하게 해당 인터페이스를 클래스 선언문에 implements 구문으로 추가하고 인터페이스가 제공하는 메소드들을 구현하기만 하면 끝이다.
    
2. 믹스인 정의에 안성맞춤 
    
    믹스인이란 어떤 클래스의 주 기능에 추가적인 기능을 혼합한다 하여서 믹스인이라고 한다. 그러므로 믹스인 인터페이스는 어떤 클래스의 주 기능이외에 믹스인 인터페이스의 기능을 추가적으로 제공하게 해주는 효과를 준다. (예 : Comparable, Cloneable, Serializable)
    
    ```java
    public class Mixin implements Comparable {
    	@Override
    	public int compareTo(Object o) {
        	return 0;
        }
    }
    ```
    
3. 계층구조가 없는 타입 프레임워크를 만들 수 있다.
    
    ```java
    public interface Singer {
        AudioClip sing(Song s);
    }
    
    public interface Songwriter {
        Song compose(int chartPosition);
    }
    
    /* 인터페이스로 정의하면 가수 클래스가 Singer와 Songwriter를 모두 구현해도 
     * 전혀 문제되지 않는다. 
     * 심지어 Singer와 Songwriter 모두를 확장하고 
     * 새로운 메서드까지 추가한 제 3의 인터페이스를 정의할 수 있다. */
    public interface SingerSongwriter extends Singer, Songwriter {
        AudioClip strum();
        void actSensitive();
    }
    ```
    
    ```java
    // 추상 클래스로 구현할 경우 
    public abstract class Singer {
        abstract void sing(String s);
    }
    
    public abstract class SongWriter {
        abstract void compose(int chartPosition);
    }
    
    public abstract class SingerSongWriter {
        abstract void strum();
        abstract void actSensitive();
        abstract void Compose(int chartPosition);
        abstract void sing(String s);
    }
    ```
    
    - 추상 클래스로 만들었기 때문에 Singer 클래스와 SongWriter 클래스를 둘다 상속할 수 없어SIngerSongWriter라는 또 다른 추상 클래스를 만들어서 클래스 계층을 표현할 수 밖에 없다. 만약 이런 Singer 와 SongWriter와 같은 속성들이 많이 있다면, 그러한 클래스 계층구조를 만들기 위해 많은 조합이 필요하고 결국엔 고도비만 계층구조가 만들어질 것이다. 이러한 현상을 조합 폭발이라고 한다.
4. 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상 시키는 안전하고 강력한 수단이 된다.
    
    타입을 추상 클래스로 정의해두면 해당 타입에 기능을 추가하는 방법은 상속 뿐이다. 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 쉽게 깨진다.
    
    - 래퍼 클래스 : 기존에 인터페이스를 구현한 클래스를 주입받아 기존 구현체에 부가기능을 손쉽게 더할 수 있는 클래스

## 인터페이스의 단점

- 디폴트 메서드로 제공해서는 안되는 equals와 hashcod같은 Object 메서드도 존재한다.
- 인스턴스 필드를 가질 수 없고 public이 아닌 정적 멤버도 가질 수 없다.(private 정적 메서드 제외)
- 직접 만든 인터페이스 외에는 디폴트 메서드를 추가할 수 없다.

# 추상 골격 구현(skeletal implementation) 클래스

- 인터페이스와 추상 클래스의 장점을 모두 취할 수 있음
- 인터페이스로는 타입과 디폴트 메서드 정의, 골격 구현 클래스는 나머지 메서드들까지 구현한다.(템플릿 메서드 패턴)

```java
// 코드 20-1 골격 구현을 사용해 완성한 구체 클래스 (133쪽)
// int 배열을 받아 Integer 인스턴스의 리스트 형태로 보여주는 어댑터(Adapter) 이기도 하다.
static List<Integer> intArrayAsList(int[] a) { 
    Objects.requireNonNull(a);

    // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
    // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
    return new AbstractList<>() {
        @Override public Integer get(int i) {
            return a[i];  // 오토박싱(아이템 6)
        }

        @Override public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;     // 오토언박싱
            return oldVal;  // 오토박싱
        }

        @Override public int size() {
            return a.length;
        }
    };
}
// 박싱과 언박싱 때문에 성능은 그리 좋지 않다.
// 익명 클래스 형태를 사용했다.
// 사용자는 몇 가지 오버라이딩 메서드만을 통해 원하는 바를 구현했다.
```

- 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유로움
- 인터페이스를 구현한 클래스에서 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하여 활용할 수도 있다. 이는 래퍼 클래스에서의 활용법과 비슷하다. 이 방식을 시뮬레이트한 다중 상속이라 하고, 다중 상속의 많은 장점을 제공하며 동시에 단점은 피하게 해준다.

# 단순 구현(simple implementation)

골격 구현의 작은 변종으로 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상클래스가 아니란 점이 다르다. 쉽게 말해 가장 단순한 구현이다. 단순 구현은 추상 클래스와 다르게 그대로 써도 되거나 필요에 맞게 확장해도 된다. 단순 구현의 좋은 예로는 AbstractMap.SimpleEntry 가 있다.

# 핵심정리

일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 골격 구현은 ‘가능한 한’ 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다. ‘가능한 한’이라고 한 이유는, 인터페이스에 걸려 있는 구현상의 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.
