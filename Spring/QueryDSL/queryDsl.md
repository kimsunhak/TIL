## Spring QueryDSL 정리

#### 결과를 DTO로 반환

##### 1. Propertie 접근

```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {
  	
  	private final JPAQueryFactory queryFactory;
  	
  	@Override
	  public List<MemberListDto> findAllMemberName() { 
        return queryFactory.select(Projections.bean(
                MemberListDto.class,
                member.memberNo,
                member.memberName))
                .from(member)
                .fetch();
    }
}
```

##### 2. Filed 직접 접근

```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {
  	
  	private final JPAQueryFactory queryFactory;
  	
  	@Override
	  public List<MemberListDto> findAllMemberName() { 
        return queryFactory.select(Projections.fields(
                MemberListDto.class,
                member.memberNo,
                member.memberName))
                .from(member)
                .fetch();
    }
}
```



