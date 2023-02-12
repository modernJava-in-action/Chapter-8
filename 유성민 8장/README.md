## 컬렉션 API 개선
### 컬렉션 팩토리
  자바 9에서는 작은 컬렉션 객체를 쉽게 만들 수 있는 몇 가지 방법을 제공한다. 적은 요소를 포함하는 리스트를 만드는 자바 코드는 아래와 같다:
```java
  List<String> friends = new Arraylist<>();
  friends.add("Raphael");
  friends.add("Olivia");
```
 이 코드를 Arrays.asList() 팩토리 메서드를 이용하면 줄일 수 있다:
```java
  List<String> friends = Arrays.asList("Raphael", "Olivia");
```
하지만 위 코드로 생성한 리스트는 고정 크기이기 때문에 요소를 갱신할 순 있지만 새 요소를 추가하거나 요소를 삭제할 순 없다. 위 코드에서 요소를 추가하려 하면 UnsupportedOperationException이 발생한다.

#### 리스트 팩토리
  List.of 팩토리 메소드를 이용하면 간단하게 리스트를 만들 수 있다.
```java
  List<String> friends = List.of("Raphael", "Olivia", "Thibaut);
```
  하지만 이렇게 생성된 코드 역시 새로운 요소를 추가하려 하면 UnsupportedOperationException이 발생하게 된다.

#### 집합 팩토리
  Set.of를 사용하면 바꿀 수 없는 집합을 만들 수 있다.
```java
  Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```
만일 아래와 같이 중복된 요소를 제공해 집합을 만들려고 하면 IllegalArgumentException이 발생한다. 
```java
  Set<String> friends = Set.of("Raphael", "Olivia", "Olivia");
```
집함은 오직 고유의 요소만 포함할 수 있다는 것을 기억하자.

#### 맵 팩토리
  맵을 만들기 위해서는 키와 값이 있어야 한다. 자바 9에서는 두 가지 방법으로 바꿀수 없는 맵을 초기화할 수 있다. 첫번째 방법으로는 Map.of 팩토리 메서드에 키와 값을 번갈아 제공하는 방법이 있다:
```java
  Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibat", 26);
```
  다른 방법으로는 Map.Entry<K, V> 객체를 인수로 받으며 가변 인수로 구현된 Map.ofEntries 팩토리 메서드를 이용하는 것이 좋다.
```java
  import static java.util.Map.entry;

  Map<String, Integer> ageOfFriends = Map.ofEntries(entry("Raphael", 30),
                                                    entry("Olivia", 25),
                                                    entry("Thibaut", 26));
```

### 리스트와 집합 처리
  자바 8에서는 List, Set 인터페이스에 다음과 같은 메서드를 추가했다.
  - removeIf : 프레디케이트를 만족하는 요소를 제거한다. List나 Set을 구현하거나 그 구현을 상속받은 모든 클래스에서 이용할 수 있다.
  - replaceAll : 리스트에서 이용할 수 있는 기능으로 UnaryOperator 함수를 이용해 요소를 바꾼다.
  - sort : List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다.

#### removeIf 메서드
  다음은 숫자로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드이다:
```java
  for (Transaction transaction : transactions) {
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
      transactions.remove(transaction);
    }
  }
```
  하지만 위 코드는 ConcurrentModificationException을 일으킨다. 반복자의 상태가 컬렉션의 상태와 서로 동기화되지 않기 때문이다. 이 코드를 자바 8의 removeIf 메서드로 바꾸게 되면 코드가 단순해지고 버그도 예방할 수 있다. removeIf 메서드는 삭제할 요소를 가리키는 프레디케이트를 인수로 받는다.
```java
  transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

#### replaceAll 메서드
  기존 컬렉션의 요소를 새로운 요소로 바꾸려면 자바 8의 새로운 기능인 replaceAll을 사용하면 된다:
```java
  referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

### 맵 처리
  자바 8에서는 Map 인터페이스에 몇 가지 디폴트 메서드를 추가했다.

#### forEach 메서드
  자바 8에서부터 Map 인터페이스는 BiConsumer를 인수로 받는 forEach 메서드를 지원한다. 아래는 이를 활용한 반복 코드이다:
```java
  ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```

#### 정렬 메서드
  자바 8에서는 맵의 항목을 쉽게 비교할 수 있는 몇가지 방법을 제공한다.
  - Entry.comparingByValue
  - Entry.comparingByKey
  아래 코드를 통해 사용법을 알아보자:
```java
  favouriteMovies.entrySet().stream()                   //Map<String, String> favouriteMovies
                 .sorted(Entry.comparingByKey())
                 .forEachOrdered(System.out::println);
```
  위 코드는 Key 값을 기준으로 알파벳 순으로 정렬한다.

#### getOrDefault 메서드
  기존에는 찾으려는 키가 존재하지 않으면 널이 반환되므로 NullPointerException을 방지하려면 결과가 널인지 확인해야 했다. 하지만 getOrDefault 메서드를 이용하면 기본값을 반환하는 방식으로 쉽게 이 문제를 해결할 수 있다. 이 메서드는 첫번째 인수로 키를 두번째 인수로 기본값을 받으며 맵에 키가 존재하지 않으면 두번째 인수로 받은 기본값을 반환한다.
```java
  favouriteMovies.getOrDefault("Olivia", "Matrix");   //Olivia 라는 키가 존재하지 않으면 Matrix 를 값으로 반환한다.
```

#### 계산 패턴
  맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황이 있다면 아래 메서드들을 사용하면된다.
  - computeIfAbsent : 제공된 키에 해당하는 값이 없으면(값이 없거나 널), 키를 이용해 새 값을 계산하고 맵에 추가한다.
  - computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
  - compute : 제공된 키로 새 값을 계산하고 맵에 저장한다.

  정보를 캐시할 때 computeIfAbsent를 활용할 수 있다. 파일 집합의 각 행을 파싱해 SHA-256을 계산한다고 가정하자. 기존에 이미 데이터를 처리했다면 이 값을 다시 계산할 필요가 없다. 여러 값을 저장하는 맵을 처리할 때도 이 패턴을 유용하게 활용할 수 있다. `Map<K, List<V>>`에 요소를 추가하려면 항목이 초기화되어 있는지 확인해야한다. 여기에 computeIfAbsent 메서드를 활용하면 키가 존재하지 않으면 값을 계산해 맵에 추가하고 키가 존재하면 기존 값을 반환하게 할수 있다.
```java
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class CacheExample {

  private MessageDigest messageDigest;

  public static void main(String[] args) {
    new CacheExample().main();
  }

  public CacheExample() {
    try {
      messageDigest = MessageDigest.getInstance("SHA-256");
    }
    catch (NoSuchAlgorithmException e) {
      e.printStackTrace();
    }
  }

  private void main() {
    List<String> lines = Arrays.asList(
        " Nel   mezzo del cammin  di nostra  vita ",
        "mi  ritrovai in una  selva oscura",
        " che la  dritta via era   smarrita "
    );
    Map<String, byte[]> dataToHash = new HashMap<>();

    lines.forEach(line ->
        dataToHash.computeIfAbsent(line, this::calculateDigest));
    dataToHash.forEach((line, hash) ->
        System.out.printf("%s -> %s%n", line,
            new String(hash).chars().map(i -> i & 0xff).mapToObj(String::valueOf).collect(Collectors.joining(", ", "[", "]"))));
  }

  private byte[] calculateDigest(String key) {
    return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
  }

}
```
  computeIfPresent 메서드는 현재 키와 관련된 값이 맵에 존재하며 널이 아닐때만 새 값을 계산한다.

#### 삭제 패턴
  제공된 키에 해당하는 맵 항목을 제거하는 reomve 메서드는 이미 알고있을 것이다. 자바 8에서는 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전 메서드를 제공한다.
```java
  favouriteMovies.remove(key, value);
```
  사용법은 매우 간단하다, key와 value 둘다 넘겨주면 된다.

#### 교체 패턴
  맵의 항목을 바꾸는 데 사용할 수 있는 두 개의 메서드가 추가되었다:
  - replaceAll : BiFunction을 적용한 결과로 각 항목의 값을 교체한다. 이 메서드는 이전에 살펴본 List의 replaceAll과 비슷한 동작을 수행한다.
  - Replace : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다.
```java
  favourtieMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```
  위 코드를 실행하면 모든 value가 대문자로 변환된다.

#### 합침
  두 그룹의 연락처를 포함하는 두개의 맵을 합치려면 putAll 메서드를 사용할 수 있다.
```java
    Map<String, String> family = Map.ofEntries(
        entry("Teo", "Star Wars"),
        entry("Cristina", "James Bond"));
    Map<String, String> friends = Map.ofEntries(entry("Raphael", "Star Wars"));

    System.out.println("--> Merging the old way");
    Map<String, String> everyone = new HashMap<>(family);
    everyone.putAll(friends);           //friends의 모든 항목을 everyone으로 복사한다.
    System.out.println(everyone);
```
  중복된 키가 없다면 위 코드는 잘 동작한다, 하지만 값을 좀 더 유연하게 합쳐야 한다면 새로운 merge 메서드를 이용할 수 있다. 다음 코드는 두 영화의 문자열을 합치는 방법으로 문제를 해결한다.
```java
    Map<String, String> friends2 = Map.ofEntries(
        entry("Raphael", "Star Wars"),
        entry("Cristina", "Matrix"));

    System.out.println("--> Merging maps using merge()");
    Map<String, String> everyone2 = new HashMap<>(family);
    friends2.forEach((k, v) -> everyone2.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));   //중복된 키가 있으면 두 값을 연결
    System.out.println(everyone2);
```

### 개선된 ConcurrentHashMap
  ConcurrentHashMap 클래스는 동시성 친화적이며 최신 기술을 반영한 HashMap 버전이다. ConcurrentHashMap은 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 혀용한다. 따라서 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다.

#### 리듀스와 검색
  ConcurrentHashMap은 스트림에서 봤던 것과 비슷한 종류의 세가지 새로운 연산을 지원한다:
  - forEach : 각 (키, 값) 쌍에 주어진 액션을 실행
  - reduce : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
  - search : 널이 아닌 값을 반환할 때까지 각 (키, 값) 함수를 적용
  
  다음처럼 키에 함수 받기, 값, Map.Entry (키, 값) 인수를 이용한 네 가지 연산 형태를 지원한다.
  - 키 값으로 연산 (forEach, reduce, search)
  - 키로 연산 (forEachKey, reduceKeys, searchKeys)
  - 값으로 연산 (forEachValue, reduceValues, searchValues)
  - Map.Entry 객체로 연산 (forEachEntry, reduceEntries, searchEntries)

  이들 연산은 ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행하기 때문에 이들 연산에 제공한 함수는 계산이 진행되는 동안 바뀔수 있는 객체, 값, 순서등에 의존하지 않아야 한다. 또한 이들 연산에 병렬성 가준값을 지정해야 한다. 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다, 기준값을 1로 지정하면 공토 스레드 풀을 이용해 병렬성을 극대화한다. Long.Max_VALUE를 값으로 설정하면 한개의 스레드로 연산을 실행한다. 아래 예제에서는 reduceValues 메서드를 이용해 맵의 최댓값을 찾는다:
```java
  ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
  long parallelistThreshold = 1;
  Optional<Integer> maxValue = Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

#### 계수
  ConcurrentHashMap 클래스는 맵의 매핑 개수를 반환하는 mappingCount 메서드를 제공한다. 기존의 size 메서드 대신 이 메서드를 사용하는것이 매핑의 개수가 int의 범위를 넘어서는 이후의 상황을 대처할 수 있기 때문에 더 바람직하다.

#### 집합뷰
  ConcurrentHashMap 클래스는 집합 뷰로 반환하는 KeySet이라는 메서드를 제공한다. 맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다. newKeySet이라는 메서드를 이용하면 ConcurrentHashMap으로 유지되는 집합을 만들수도 있다.