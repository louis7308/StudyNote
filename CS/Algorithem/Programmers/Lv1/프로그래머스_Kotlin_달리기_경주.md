## 출제자의 의도
해설진이 추월한 선수의 이름을 부른다면 추월한 선수의 등수가 그 앞의 선수로 바뀌고
경주가 끝났을 때 선수들의 이름을 1등부터 등수 순서대로 반환하는 로직을 만들어달라

## 문제 풀이
1. players에 있는 리스트를 전부 HashMap에<선수이름, 등수>로 매핑을 시켜놓습니다.
2. callings에서 이름이 불린 선수의 등수를 HashMap에서 가져옵니다.
3. 그리고 나서 등수의 - 1 값을 가지고 있는 선수를 찾습니다. 찾은 후 부터는 둘의 등 수를 바꿔줍니다.
4. 그리고 마지막 리스트에는 HashMap에 있는 값들을 등 수에 오름차순 순서대로 가져와서 정렬 후 반환해줍니다.

## 코드
```kotlin
class Solution {
    fun solution(players: Array<String>, callings: Array<String>): Array<String> {
        val playerPos = players.indices.associateBy { players[it] }.toMutableMap()
        
        
        
        for(calling in callings) {
            val pos = playerPos[calling] ?: continue // 3
            
            players[pos] = players[pos-1] // 3 poe
            players[pos-1] = calling // 2 // kai
            playerPos[players[pos]] = pos // playerPos[poe] = 3
            playerPos[calling] = pos - 1 // playerPos[kai] = 2
        }
        
        return players
    }
}
```