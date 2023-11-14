오늘은 코틀린을 활용해본 사람들이라면 Collection 자료주조인 List, Set, Map 등을 공부해보겠습니다!

코틀린의 고차함수를 이용하기 때문에 더욱 강력한 동작을 수행 할 수 있습니다. ( 강력하다? 간편하게 가능하다는 것이죠. )

### sort()
Collection을 정렬해주는 역활을 수행합니다.
* **sorted()** 를 활용하면 정렬이 된 새로운 객체를 반환합니다.
* **sortByDescending()** 등을 사용하면 내림차순 정렬이 가능합니다.
> [!tip]
> 내림차순이란? 5 -> 4 -> 3 -> 2 -> 1 뒤로 갈 수 록 수가 작아지는
> 오름차순이란? 1 -> 2 -> 3 -> 4 -> 5 뒤로 갈 수 록 수가 커지는

```kotlin
fun main() {
	val a = mutableListOf(3, 2, 1)
	a.sort() // 오름차순 정렬
	println(a)

	val sorted = a.sortedByDescending { it } // 내림차순, 정렬된 새로운 Collection 반환
	println(sorted)

	// sortBy() : 각 객체가 갖고있는 프로퍼티를 기준으로 정렬
	val list = mutableListOf(1 to "A", 2 to "B", 100 to "C", 50 to "D") // Pair 객체 등록
	list.sortBy { it.second }
	println(list)
}
```
함수형 프로그래밍 언어 답게 sortBy() 라는 고차함수를 통해 Collection을 구성하는 각 객체들의 특정 프로퍼티를 기준으로 정렬을 할 수 있도록 해준다.
위 예제에서는 Pair 객체의 second 값을 기준으로 정렬하게 된다. 따라서 위 예제를 실행해보았을 때, 아래와 같은 결과가 나온다.
```kotlin
[1, 2, 3]
[3, 2, 1]
[(1, A), (2, B), (100, C), (50, D)] // 두번째 Pair인 <Int, String> String 으로 정렬 되어진다.
```


### map()
Collection을 구성하는 각 요소들에 대해 특정 표현식에 의거하여 변형을 거친 뒤 새로운 Collection을 반환해준다.
* **mapIndexed()** mapIndex는 Iterable의 index 값을 가진 transform function을 통해 index를 포함한 List로 변형 시킬 수 있다.
* **mapTo()** mapTo는 특정한 수신값(destination)에 transform시켜 넣어주는 연산이다.
```kotlin
fun main() {
	val a: List<Int> = listOf(1, 2, 3)
	val b = a.map { it * 10 }

	val c = ('a'..'z').mapIndexed { index, i -> Pair(index, i) }

	val mutableList: MutableList<Char> = mutableListOf()
	('a'..'z').toList().mapTo(mutableList) { it }
	

	println(b)
	println(c)
	println(mutableList)
}
```
결과는
```kotlin
[10, 20, 30]
[(0, a), (1, b), (3, d), (4, e), (5, f)...]
[a, b, c, d, e, f, g, ...]
```

### forEach()
Collection을 구성하는 요소들을 깔끔하게 하나씩 순회 할 수 있다.
* **forEachIndexed()** 라는 고차함수가 있는데 이는 Index를 가져와 사용할 수 있다.
```kotlin
fun main() {
	val a: List<Char> = listOf('A', 'B', 'C')

	a.forEach { println(it) }

	a.forEachIndexed { index, c -> println("$Index : $c") }
}
```
결과는
```
A
B
C
0 : A
1 : B
2 : C
```

### filter()
특정 조건에 부합하는 요소만 걸러내서 새로운 Collection을 반환해주는 녀석이다. Boolean 값을 반환하는 표현식을 주입해준다.
```kotlin
fun main() {
	val a: List<Int> = listOf(1, 2, 3, 4, 5, 6)
	val b = a.filter { it % 2 == 0 }
	println(b)
}
```
결과는
```kotlin
[2, 4, 6]
```

### find()
find는 최초로 조건에 부합하는 녀석을 반환해주는 녀석이다.
조건에 부합하는 녀석이 끝날때까지 나타나지 않는다면 null을 반환하는 특징이 있다.
```kotlin
fun main() {
	val a: List<Int> = listOf(1, 2, 3, 4, 5, 6)

	val b = a.find { it % 2 == 0 }

	val c = a.findLast { it % 2 == 0 }

	println(b)
	println(c)
}
```
반대로 findLast() 라는 친구는 조건에 부합하는 녀석이 가장 마지막 녀석을 반환해준다
결과는
```kotlin
2
6
```

### any(), all(), none()
Collection 각 구성 요소들을 하나씩 검사해보며 Boolean을 반환하는 함수들이다.
```kotlin
fun main() {
	val list: List<Int> = listOf(1, 2, 3, 4)

	if(list.any { it % 2 == 0 }) {
		println("짝수 데이터가 존재합니다. ")
	}

	if(list.all { it % 2 == 0 }) {
		println("모두 짝수 데이터 입니다.")
	} else {
		println("홀 수 데이터도 존재한다.")
	}

	if(list.none { it > 10 }) {
		println("10보다 큰 원소가 없습니다.")
	}
}
```
* **any()** 는 조건을 하나 만 만족하더라도 true를 반환합니다.
* **all()** 는 조건을 모두 만족하는 요소들 일 때만 true를 반환합니다.
* **none()** 은 조건을 모두 만족하지 않는 요소들만 존재할 때 true를 반환합니다.

### flatMap()
flatMap은 감싸져 있는 Collection을 하나로 합치기 위한 연산입니다.
Collection을 새로 만들고, 이들을 하나의 Collection으로 Flatten 하여 반환한다.
```kotlin
fun main() {
	val testList = listOf(listOf(1, 2), listOf(7, 8, 9), mutableListOf(4, 5, 6))
	
	val b = testList.flatMap { it }
	val c = testList.flatMap { it.take(1) }

	println(b)
	println(c)
}
```
결과는
```kotlin
[1, 2, 7, 8, 9, 4, 5, 6]
[1, 7, 4]
```

### partition()
어떤 원소에 대해 특정 조건을 걸어서 조건에 부합하는 녀석들과 부합하지 않는 녀석들 이렇게 두 Collection으로 분리해준다.
이때 Pair 형태로 분리되게  되는데 **조건에 부합하는 녀석들은 first**로 가고 **아닌 녀석들은 second로 간다**
```kotlin
fun main() {
	val a = listOf(1, 2, 3, 4, 5, 6)

	val partition = a.partition { it % 2 == 0 }

	println(partition.first) // 조건 부합
	println(partition.second) // 조건 부합 X
}
```
결과는
```kotlin
[2, 4, 6]
[1, 3, 5]
```

### getOrElse()
Collection에 인덱스로 값을 참조 했을 때, 만약 해당 인덱스에 값이 없을  경우 지정된 스코프 내에서 원하는 동작들을 수행하고 스코프 내 가장 마지막 줄 코드의 반환값을 뱉는다.
```kotlin
fun main() {
	val a = listOf(1, 2, 3, 4, 5, 6)

	println(a.getOrElse(2) { 10 })

	println(a.getOrElse(10) {
		println("10번째 원소가 없습니다.")
		"반환되는 값 필요"
	})
}
```
결과는
```
3
10번째 원소가 없습니다.
반환되는 값 필요
```

### reduce(), fold()
Collection을 구성하는 모든 원소들에 대해 누적합을 계산하는 함수들이다.
고차함수이기 때문에 누적합을 어떻게 쌓아올리는 지에 대해 표현식을 걸어줄 수 있다.
* **fold()** 의 경우 초기 값을 설정해줄 수 있다.
* **reduce()** 는 첫 번째 요소를 acc 로 사용하고, 두 번째 요소 부터 연산하게 된다.
```kotlin
fun main() {
	val a = listOf(1, 3, 5)

	println("Fold : ${a.fold(0) { acc, i -> 
		acc + i * 2
	}}")

	println("Reduce : ${a.reduce { acc, i -> 
		acc + i * 2
	}}")
}
```
결과는
```kotlin
Fold : 18
Reduce : 17
```
위 예제의 경우 **현재 누적합 + (현재 값 * 2)** 라는 표현식을 통해 누적합을 쌓이게 된다.

#### fold()의 결과가 18이 나온 과정
1. acc : 0, i : 1 -> 0 + (1 * 2) = 2
2. acc : 2, i : 3 -> 2 + (3 * 2) = 8
3. acc : 8, i : 5 -> 8 + (5 * 2) = 18

#### reduce()의 결과가 17이 나온 과정
1. acc : 1, i : 3 -> 1 + (3 * 2) = 7
2. acc : 7, i : 5 -> 7 + (5 * 2) = 17
