## slice(), sliceArray()
```kotlin
fun <T> Array<out T>.slice(indices: IntRange): List<T>
fun <T> List<T>.slice(indices: IntRange): List<T>
fun <T> Array<T>.sliceArray(indices: IntRange): Array<T>
```
* Array나 List를 원하는 구간만 잘라서 새로운 List를 만들고 싶을 때 사용
* **fromIndex ~ toIndex 구간**으로 배열을 자름
* 원복 객체는 변경되지 않는다.
예시
```kotlin
val array = intArrayOf(0, 1, 2, 3, 4, 5)
val slicedList = array.slice(2..5)
val slicedArray = array.sliceArray(IntRange(2, 5))
. 
println(array.contentToString()) // [0, 1, 2, 3, 4]
println(slicedList) // [2, 3, 4, 5]
println(slicedArray.contentToString()) // [2, 3, 4, 5]
```

## copyOfRange()
```kotlin
fun <T> Array<T>.copyOfRange(
	fromIndex: Int,
	toIndex: Int
): Array<T>
```
* Array를 원하는 구간만 잘라서 새로운 Array를 만들고 싶을 때 사용.
* Array.slice(), sliceArray() 내부에서 호출되는 메소드임
* **fromIndex ~ toIndex - 1** 구간으로 배열을 자름
* 원본배열은 변경되지 않는다.
예시
```kotlin
val array = intArrayOf(0, 1, 2, 3, 4, 5)
val copiedArray = array.copyOfRange(2, 5)
println(copiedArray.contentToString()) // 2, 3, 4
```
## subList()
```kotlin
abstract fun subList(fromIndex: Int, toIndex: Int): List<E>
```
* List를 원하는 구간만 잘라서 보여주고 싶을 때 사용
* 원본 리스트와 같은 메모리 공간을 사용 // 얕은 복사
* **fromIndex ~ toIndex - 1 구간**으로 리스트를 자름
* 이 리스트가 변경되면 원본 리스트 역시 변경 된다.
예시
```kotlin
val list = mutableListOf(0, 1, 2, 3, 4, 5)
val subList = list.subList(2, 5)
println(list) // [0, 1, 2, 3, 4, 5]
println(subList) // [2, 3, 4]
```
## slice()와 subList()의 차이점
slice()는 원본 List를 잘라 새로운 List로 만들어 리턴하지만, subList()는 원본 List를 원하는 구간 만큼만 잘라서 보여준다
예시
```kotlin
val list = mutableListOf(0, 1, 2, 3, 4, 5)

val slicedList = list.slice(2..5)
val subList = list.subList(2, 6)
println(slicedList) // [2, 3, 4, 5]
println(subList) // [2, 3, 4, 5]

list[2] = 1
println(slicedList) // [2, 3, 4, 5]
println(subList) // [1, 3, 4, 5]
```