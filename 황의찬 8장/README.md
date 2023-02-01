# CHapter 8 - 컬렉션 API 개선
8장에서는 Java 8, Java 9에서 추가되어 우리의 삶을 편리하게 만들어 줄 새로운 컬렉션 API의 기능을 배웁니다.  
먼저 작은 리스트, 집합, 맵을 쉽게 만들 수 있도록 Java9 에 새로 추가된 컬렉션 팩토리를 살펴본다.  
  
다음으로 Java 8의 개선 사항으로 리스트와 집합에서 요소를 삭제하거나 바꾸는 관용 패턴을 적용하는 방법을 배웁니다.  
마지막으로 맵 작업과 관련해 추가된 새로운 편리 기능을 살펴봅니다.  
  
## 8.1 컬렉션 팩토리
기존의 적은 요소를 포함하는 리스트를 만드는 코드
```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```
다음처럼 Arrays.asList() 팩토리 메서드를 이용하면 코드를 간단하게 줄일 수 있다.  
```java
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
```
고정 크기의 리스트를 만들었으므로 요소를 갱신할 순 있지만 새 요소를 추가하거나 요소를 삭제할 순 없다.  
예를 들어 요소를 갱신하는 작업은 괜찮지만 요소를 추가하려 하면 UnsupportedOperationException이 발생한다.  
```java
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
		friends.set(0, "Richard");
		try {
			friends.add("Thibaut");
		} catch (UnsupportedOperationException e) {
			e.printStackTrace();
		}
```
  
내부적으로 고정된 크기의 변환할 수 있는 배열로 구현되었기 때문에 이와 같은 일이 일어납니다.  
  
집합의 경우  
```java
Set<String> friends = new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut"));

Set<String> friends = Stream.of("Raphael", "Olivia", "Thibaut")
			.collect(Collectors.toSet());
```
하지만 두 방법 모두 매끄럽지 못하며 내부적으로 불필요한 객체 할당을 필요로 합니다.  
그리고 결과는 변환할 수 있는 집합이라는 사실에 주목해봅니다.  
  
Java 9에서 작은 리스트, 집합, 맵을 쉽게 만들 수 있도록 팩토리 메서드를 제공합니다.  
  
### 8.1.1 리스트 팩토리
List.of 팩토리 메서드를 이용해서 간단하게 리스트를 만들 수 있습니다.  
```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
```
friends 리스트에 요소를 추가하면, UnsupportedOperationException이 발생합니다. 변경할 수 없는 리스트가 만들어졌기 때문입니다.  
컬렉션이 의도치 않게 변하는 것을 막을 수 있기 때문입니다. 리스트를 바꿔야 하는 상황이라면 직접 리스트를 만들면 됩니다.  
마지막으로 null 요소는 금지하므로 의도치 않은 버그를 방지하고 조금 더 간결한 내부 구현을 달성했습니다.  
  
Collectors.toList() 컬렉터로 스트림을 리스트로 변환할 수 있다.  
데이터 처리 형식을 설정하거나 데이터를 변환할 필요가 없다면 사용하기 간편한 팩토리 메서드를 이용할 것을 권장한다.  
  
### 오버로딩 vs 가변 인수
List.of의 다양한 오버로드 버전이 있다는 사실을 알 수 있습니다.  
```java
static <E> List<E> of(E e1, E e2, E e3, E e4)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5)
```
아마 여러분은 왜 다음처럼 다중 요소를 받을 수 있도록 자바 API를 만들지 않은 것인지 궁금할 것입니다.  
```java
static <E> List<E> of(E...elements)
```
  
내부적으로 가변 인수 버전은 **추가 배열을 할당해서 리스트로 감쌉니다.** 따라서 배열을 할당하고 초기화하며 나중에 가비지 컬렉션을 하는 비용을  
지불해야 합니다. 고정된 숫자의 요소(최대 열개까지)를 API로 정의하므로 이런 비용을 제거할 수 있습니다.  
List.of로 열 개 이상의 요소를 가진 리스트를 만들 수도 있지만 이 때는 가변 인수를 이용하는 메서드가 사용됩니다.  
  
### 8.1.2 집합 팩토리
List.of와 비슷한 방법으로 바꿀 수 없는 집합을 만들 수 있습니다.  
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends);
```
중복된 요소를 제공해 집합을 만들려고 하면 Olivia라는 요소가 중복되어 있다는 설명과 함께 IllegalArgumentException이 발생합니다.  
집합은 오직 고유의 요소만 포함할 수 있다는 원칙을 상기시킵니다.  

### 8.1.3 맵 팩토리
맵을 만드는 것은 조금 복잡한데 키와 값이 있어야 하기 때문입니다.  
Java 9에서는 두 가지 방법으로 바꿀 수 없는 맵을 초기화할 수 있습니다.  
Map.of 팩토리 메서드에 키와 값을 번갈아 제공하는 방법으로 맵을 만들 수 있습니다.  
```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
System.out.println(ageOfFriends);
```
열 개 이하의 키와 값 쌍을 가진 작은 맵을 만들 때는 이 메서드가 유용하다.  
그 이상의 맵에서는 `Map.Entry<K, V>` 객체를 인수로 받으며 가변 인수로 구현된 Map.ofEntries 팩토리 메서드를 이용하는 것이 좋다.  
이 메서드는 키와 값을 감쌀 추가 객체 할당을 필요로 한다.  
```java
Map<String, Integer> ageOfFriends = Map.ofEntries(entry("Rapheal", 30),
					entry("Olivia", 25),
					entry("Thibaut", 26));
System.out.println(ageOfFriends);
```
Map.entry는 Map.Entry 객체를 만드는 새로운 팩터리 메서드다.  
  
## 8.2 리스트와 집합 처리
Java 8에서는 **List, Set 인터페이스**에 다음과 같은 메서드를 추가했습니다.  
- removeIf : 프레디케이트를 만족하는 요소를 제거한다. List나 Set을 구현하거나 그 구현을 상속받은 모든 클래스에서 이용할 수 있다.  
- replaceAll : 리스트에서 이용할 수 있는 기능으로 UnaryOperator (T -> T)를 이용해 요소를 바꾼다.  
- sort : List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다.  
  
이들 메서드는 호출한 컬렉션 결과를 바꾼다.  
새로운 결과를 만드는 스트림 동작과 달리 이들 메서드는 기존 컬렉션을 바꾼다. 컬렉션을 바꾸는 동작은 에러를 유발하며 복잡함을 더한다.  
Java 8에 removeIf와 replaceAll를 추가한 이유가 바로 이 때문이다.  
  
### 8.2.1 removeIf 메서드
다음은 숫자로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드입니다.  
```java
for (Transaction transaction : transactions) {
	if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
		transactions.remove(transaction);
	}
}
```
위 코드는 ConcurrentModificationException을 일으킵니다. 내부적으로 for-each 루프는 Iterator 객체를 사용하므로 위 코드는 다음과 같이 해석됩니다.  
```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
	Transaction transaction = iterator.next();
	if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
	transactions.remove(transaction);
	}
}
```
두 개의 개별 객체가 컬렉션을 관리한다는 사실을 주목해봅니다.  
- Iterator 객체, next(), hasNext()를 이용해 소스를 질의합니다.  
- Collection 객체 자체, remove()를 호출해 요소를 삭제합니다.  
  
결과적으로 반복자의 상태는 컬렉션의 상태와 서로 동기화되지 않습니다. Iterator 객체를 명시적으로 사용하고 그 객체의 remove()  
메서드를 호출함으로 이 문제를 해결할 수 있다.  
```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
	Transaction transaction = iterator.next();
	if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
	iterator.remove();
	}
}
```
이 코드 패턴은 Java 8의 removeIf 메서드로 바꿀 수 있습니다.  
removeIf는 삭제할 요소를 가리키는 프레디케이트를 인수로 받습니다.  
```java
transactions.removeIf(transaction ->
			Character.isDigit(transaction.getReferenceCode().charAt(0)));
```
  
### replaceAll 메서드
List 인터페이스의 replaceAll 메서드를 이용해 리스트의 각 요소를 새로운 요소로 바꿀 수 있다.  
스트림 API를 사용하면 다음처럼 문제를 해결할 수 있었다.  
```java
List<String> referenceCodes = Arrays.asList("a12", "C14", "b13");
referenceCodes.stream()
	.map(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1))
	.collect(Collectors.toList())
	.forEach(System.out::println);
```
하지만 이 코드는 새 문자열 컬렉션을 만듭니다. 우리가 원하는 것은 기존 컬렉션을 바꾸는 것입니다.  
```java
for (ListIterator<String> iterator = referenceCodes.listIterator();
		iterator.hasNext();) {
		String code = iterator.next();
		iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
	}
System.out.println(referenceCodes);
```
코드가 조금 복잡해졌습니다. Java 8의 기능을 이용하면 다음처럼 간단하게 구현할 수 있습니다.  
```java
referenceCodes.replaceAll(code ->
			Character.toUpperCase(code.charAt(0)) + code.substring(1));
```



