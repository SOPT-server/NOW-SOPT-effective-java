# 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.
그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킨다.


- ﻿﻿equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.  
    단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
- ﻿﻿equals(object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- ﻿﻿equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다 른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블 의 성능이 좋아진다

hashCode 재정의를 잘못한 경우 문제 포인트 -> 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다. 


좋은 해시 함수는 서로 다른 인스턴스에 다른 해시코드를 반환한다.


안 좋은 이슈
```Java
@Override public int hashCode() { return 42; }
```

Objects.hash() 이용한 예시 -> 성능이슈
```Java
@Override public int hashCode){
	return Objects.hash(lineNum, prefix, areaCode);
}
```

해시코드를 지연 초기화하는 hashCode method;
```Java
private int hashCode; // 자동으로 0으로 초기화된다.

@Override public int hashCode() {
	int result = hashCode;

	if (result == 0) {
		result = Short. hashcode(areacode) ;
		result = 31 * result + Short. hashCode(prefix);
		result = 31 * result + Short. hashCode (lineNum);
		hashCode = result;
	}
	return result;
```

성능을 높이기 위해서, 해시코드를 계산할 때 핵심 필드를 생략하면 안 된다.
