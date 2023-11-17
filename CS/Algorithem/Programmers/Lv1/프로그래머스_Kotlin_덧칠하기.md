## 출제자의 의도
흠.. 예산을 아끼기 위해서 최소한의 롤러 칠을 해야한다.
결론 이미 롤러 범위에 해당하여 칠해진 곳을 또 칠하지 않는 방법을 잘 생각해봐야 할 거 같다.


## 문제 풀이
1. endpoint = 0 으로 잡는다
2. 칠해야 하는 section에 fold() 함수를 실행한다 count 초기 값은 0으로
3. area 값이 endpoint 보다 크다면 endpoint 값에 area 값과 롤러 길이(m)을 더하고 - 1을 진행한다.
4. count 값을 증가시켜준다.
5. area 값이 endpoint보다 작다면 이미 페인트가 칠해진 곳이므로 count 값을 반환해준다.


## 코드 
```kotlin
class Solution {
	fun solution(n: Int, m: Int, section: IntArray): Int {
        var endpoint = 0
        return section.fold(0) { count, area ->
	        if(area > endpoint) {
		        endpoint = area + m - 1
		        count++
	        } else {
		        count
	        }
        }
    }
}
```

