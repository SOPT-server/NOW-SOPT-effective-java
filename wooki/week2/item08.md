<aside>
💡 finalize()와 clean()은 사용하지 말자.
자원을 닫으려면 trt-with-resources로 GC가 알아서 닫게 하자.

</aside>

- finalize(): Java9부터 deprecated
- clean(): finalize()보다는 낫지만 불필요하다.

Java에서 접근할 수 없는 객체를 회수하는 역할은 프로그래머가 아닌 GC이다.

finalize()와 clean()은 즉시 수행된다는 보장도 없을 뿐더러 보안 문제를 일으킬 수도 있다.