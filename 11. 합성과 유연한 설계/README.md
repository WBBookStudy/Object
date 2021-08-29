# 합성과 유연한 설계

코드를 재사용하기 위해서는 객체 합성이 클래스 상속보다 더 좋은 방법이다.  
상속 관계는 클래스 사이의 정적인 관계인 데 비해 합성 관계는 객체 사이의 동직인 관계이다. -> 코드 작성 시점에서 결정한 상속 관게는 변경이 불가능하지만 합성 관계는 실행시점에서 동적으로 변경 할 수 있다.  

## 상속을 합성으로 변경하기
#### 코드 재사용을 위해 상속을 남용했을 때 직면할 수 있는 문제
 - 불필요한 인터페이스 상속 문제
 - 메서드 오버라이딩의 오작용 문제
 - 부모 클래스와 자식 클래스의 동시 수정 문제

합성을 사용하면 상속이 초래하는 세 가지 문제점을 해결 할 수 있다.

### 불필요한 인터페이스 상속문제: java.util.Properties와 java.util.Stack

```Java
public class Properties {
    private Hashtable<String, String> properties = new Hashtable <>();

    public String setProperty(String key, String value) {
        return properties.put(key, value);
    }

    public String getProperty(String key) {
        return properties.get(key);
    }
}
```
> Properties클래스에서 상속 관계를 제거하고 Hashtable을 Peroperties의 인스턴스 변수로 포함시키면 합성 관계로 변경할 수 있다.
-> 이제 더 이상 불필요한 Hashtable의 오퍼레이션들이 Properties 클래스의 퍼블릭 인터페이스를 오염시키지 않는다.

```Java
public class Stack<E> {
    private Vector<E> elements = new Vector<>();

    public E push(E item) {
        elements.addElement(item);
        return item;
    }

    public E pop() {
        if (elements.isEmpty()) {
            throw new EmptyStackException();
        }
        return elements.remove(elements.size() - 1);
    }
}
```
> Stack 클래스의 변경

### 메서드 오버라이딩의 오작용 문제: InstrumentedHashSet

```Java
public class InstrumentedHashSet<E> {
    private int addCount = 0;
    private Set<E> set;

    public InstrumentedHashSet(Set<E> set) {
        this.set = set;
    }

    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }

    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```
> HashSet 인스턴스를 내부에 포함한 후 HashSet의 퍼블릭 인터페이스에서 제공하는 오퍼에이션들을 이용해 필요한 기능을 구현하면 된다.

#### 위의 경우에는 이전과 같이 불필요한 오퍼레이션들이 퍼블릭 인터페이스에 스며드는것을 방지하기 위함이었지만 이번 경우에는 HashSet이 제공하는 퍼블릭 인터페이스를 그대로 제공해야 한다.

```Java
public class InstrumentedHashSet<E> implements Set<E> {
    private int addCount = 0;
    private Set<E> set;

    public InstrumentedHashSet(Set<E> set) {
        this.set = set;
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    @Override public boolean remove(Object o) {
        return set.remove(o);
    }

    @Override public void clear() {
        set.clear();
    }

    @Override public boolean equals(Object o) {
        return set.equals(o);
    }

    @Override public int hashCode() {
        return set.hashCode();
    }

    @Override public Spliterator<E> spliterator() {
        return set.spliterator();
    }

    @Override public int size() {
        return set.size();
    }

    @Override public boolean isEmpty() {
        return set.isEmpty();
    }

    @Override public boolean contains(Object o) {
        return set.contains(o);
    }

    @Override public Iterator<E> iterator() {
        return set.iterator();
    }

    @Override public Object[] toArray() {
        return set.toArray();
    }

    @Override public <T> T[] toArray(T[] a) {
        return set.toArray(a);
    }

    @Override public boolean containsAll(Collection<?> c) {
        return set.containsAll(c);
    }

    @Override public boolean retainAll(Collection<?> c) {
        return set.retainAll(c);
    }

    @Override public boolean removeAll(Collection<?> c) {
        return set.removeAll(c);
    }
}
```
> 이것이 포워딩!

### 부모 클래스와 자식 클래스의 동시 수정 문제: PersonalPlaylist

```Java
public class PersonalPlaylist {
    private Playlist playlist = new Playlist();

    public void append(Song song) {
        playlist.append(song);
    }

    public void remove(Song song) {
        playlist.getTracks().remove(song);
        playlist.getSingers().remove(song.getSinger());
    }
}
```
> 안타깝게도 playlist의 경우에는 합성으로 변경하더라도 가수별 노래 목록을 유지하기 위해 Playlist와 PersonalPlaylist를 함께 수정해야 하는 문제가 해결되지는 않는다.
-> 그렇다 하더라도 여전히 상속보다 합성을 사용하는것이 좋다. 향후에 Playlist의 내부 구현을 변경하더라도 파급효과를 최대한 personalPlaylist 내부로 캡슐화할 수 있기 때문!


































