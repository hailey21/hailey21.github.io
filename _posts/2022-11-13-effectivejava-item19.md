---
title:  "상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라"
date: 2022-11-12 21:17:10 +0900
categories: Study 
---

아이템 18에서 상속을 염두에 두지 않고 설계했고 상속할 때의 주의점도 문서화해놓지 않은 ‘외부’ 클래스를 상속할 때 위험을 경고했다. 
‘외부’란 프로그래머의 통제권 밖에 있어서 언제 어떻게 변경될지 모른다는 뜻이다.

# 상속을 고려한 설계

- 메서드를 재정의 하면 어떤 일이 일어나는지를 정리하여 문서로 남겨야 함
    - 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.
    - 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지를 포함해야 한다.
    - 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다.
    - 재정의 가능 메서드 : public과 protected 메서드 중 final이 아닌 모든 메서드
    

```java
// packate java.util.AbstractCollection

/**
* {@inheritDoc}
*
* @implSpec
* This implementation iterates over the collection looking for the
* specified element.  If it finds the element, it removes the element
* from the collection using the iterator's remove method.
*
* <p>Note that this implementation throws an
* {@code UnsupportedOperationException} if the iterator returned by this
* collection's iterator method does not implement the {@code remove}
* method and this collection contains the specified object.
*
* @throws UnsupportedOperationException {@inheritDoc}
* @throws ClassCastException            {@inheritDoc}
* @throws NullPointerException          {@inheritDoc}
*/
```

이 설명에 따르면 iterator 메서드를 재정의 하면 remove 메서드의 동작에 영향을 줌을 확실히 알 수 있다. iterator메서드로 얻은 반복자의 동작이 remove 메서드의 동작에 주는 영향도 정확히 설명했다.

# hook, protected 메서드

효율적인 하위 클래스를 큰 어려움 없이 만들 수 있게 하려면 **클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.**

```java
// package java.util.AbsctractList

/**
* Removes from this list all of the elements whose index is between
* {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
* Shifts any succeeding elements to the left (reduces their index).
* This call shortens the list by {@code (toIndex - fromIndex)} elements.
* (If {@code toIndex==fromIndex}, this operation has no effect.)
*
* <p>This method is called by the {@code clear} operation on this list
* and its subLists.  Overriding this method to take advantage of
* the internals of the list implementation can <i>substantially</i>
* improve the performance of the {@code clear} operation on this list
* and its subLists.
*
* @implSpec
* This implementation gets a list iterator positioned before
* {@code fromIndex}, and repeatedly calls {@code ListIterator.next}
* followed by {@code ListIterator.remove} until the entire range has
* been removed.  <b>Note: if {@code ListIterator.remove} requires linear
* time, this implementation requires quadratic time.</b>
*
* @param fromIndex index of first element to be removed
* @param toIndex index after last element to be removed
*/
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```

이 메서드를 제공한 이유는 단지 하위 클래스에서 부분리스트의 clear 메서드를 고성능으로 만들기 쉽게 하기 위해서다.

상속용 클래스를 설계할 때 어떤 메서드를 protected로 노출해야 할지는 어떻게 결정할까? 방법은 따로 없고 심사숙고해서 잘 예측해본 다음 실제 하위 클래스를 만들어 시험해 보는것이 최선이다. protected 메서드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 한 적어야 한다. 

상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 ‘유일’하다.

# 상속을 허용하는 클래스가 지켜야 할 제약

- 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
    
    ```java
    // Super.java
    public class Super {
    
        public Super() {
            // 잘못된 예시!!
            overrideMe();
        }
    
        public void overrideMe() {
        }
    }
    
    // Sub.java
    public class Sub extends Super {
    
        private final Instant instant;
    
        public Sub() {
            super();
            instant = Instant.now();
        }
    
        @Override
        public void overrideMe() {
            System.out.println(instant);
        }
    
        public static void main(String[] args) {
            Sub sub = new Sub();
            sub.overrideMe();
        }
    }
    ```
    
    상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다. 이 예시가 instant를 두 번 출력하리라 기대했겠지만, 첫 번째는 null을 호출한다. 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overridMe를 호출하기 때문이다. → final 필드의 상태가 이 프로그램에서는 두가지이다. (정상이라면 단 하나 뿐이어야 한다.)
    
- `clone`과 `readObject` 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.
    - readObject는 하위 클래스의 상태가 미처 다 역직렬화되기 전에 재정의한 메서드를 호출함
    - clone은 하위 클래스의 clone 메서드가 복제본의 상태를 (올바른 상태로) 수정하기 전에 재정의한 메서드를 호출한다.
- `Serializable`을 구현한 상속용 클래스가 `readResolve`나 `writeReplace` 메서드를 갖는다면 이 메서드들은 private이 아닌 `protected`로 선언해야 한다.
    - private으로 선언한다면 하위 클래스에서 무시되기 때문이다.

# 일반적인 구체 클래스의 상속 금지

일반적인 구체 클래스의 경우, 전통적으로 final도 아니고 상속용으로 설계되거나 문서화되지도 않았다. 이로 인해, 클래스에 변화가 생길 때마다 하위 클래스를 오동작하게 만들 수 있다. 이 문제를 해결하는 가장 좋은 방법은 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이다.

상속을 금지하는 방법은 2가지다.

- 클래스를 `final`로 선언한다. (더 쉬운 방법)
- 모든 생성자를 `private`이나 `package-private`으로 선언하고, `public` 정적 팩터리를 만들어준다.

# 정리

- 상속용 클래스는 내부에서 스스로를 어떻게 사용하는지(자기사용 패턴) 모두 문서로 남겨야 하며, 문서화한 것은 그 클래스가 쓰이는 한 반드시 지켜야 한다.
- 다른 이가 효율 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 `protected`로 제공해야 할 수도 있다.
- 클래스를 확장해야 할 명확한 이유가 없다면, 상속을 금지하는 것이 낫다.
- 상속을 금지하려면, 클래스를 `final`로 선언하거나 생성자 모두를 외부에서 접근할 수 없도록 만들면 된다.
