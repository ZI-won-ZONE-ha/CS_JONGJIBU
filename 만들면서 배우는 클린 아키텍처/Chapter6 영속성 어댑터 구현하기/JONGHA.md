# 의존성 역전

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/5a592545-e6db-4cb2-823d-0bf785c560fc)

애플리케이션의 서비스에서는 영속성 기능을 사용하기 위해 포트 인터페이스를 호출한다. 

육각형 아키텍처에서 영속성 어댑터는 주도되는 혹은 아웃고잉 어댑터이다. 즉, 애플리케이션에 의해 호출된다. 

포트는 애플리케이션 서비스와 영속성 코드 사이의 간접적인 계층이다. 이를 통해 의존성을 역전 시켜, 영속성 코드를 리팩터링 해도 코어의 코드는 변경 되지 않는다. 

# 영속성 어댁터의 책임

1. 입력을 받는다.
    1. 포트 인터 페이스를 통해 입력을 받는다. 
2. 입력을 데이터 베이스 포맷으로 매핑한다.
    1. 입력 모델을 데이터 베이스 테이블 구조를 반영한 JPA 엔티티 객체로 매핑한다.
    2. 매핑하지 않는 전략은 8장에서 다룬다.
    3. **영속성 어댑터 입력 모델이 영속성 어댑터 내부에 있는 것이 아니라 애플리케이션 코어에 있다. → 이를 통해 영속성 어댑터 내부를 변경하는 것이 바뀌지 않는 것이다.** 
3. 입력을 데이터베이스로 보낸다. 
    1. 쿼리를 날리고 결과를 받아 온다
4. 데이터베이스 응답을 포트에 정의된 출력 모델로 매핑해서 반환한다. 
    1. **출력 모델이 영속성 어댑터가 아니라 애플리케이션 코어에 위치한다.**

# 포트 인터페이스 나누기

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/f6b81b4c-e801-4b4a-90d8-7e41056a05ca)

하나의 아웃고잉 포트 인터페이스에 모든 데이터 베이스 연산을 모아두면 불필요한 의존성이 생긴다. 

이를 인터페이스 분리 원칙 (Interface Segregation Principle, ISP) 를 통해 해결 할 수 있다. 

즉, 클라이언트가 오로지 자신이 필요로하는 메서드만 알면되도록 넓은 인터페이스를 특화된 인터페이스로 분리해야 한다. 는 것이다. 다음과 같이 고칠 수 있다. 

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c88f15f3-734c-4906-a166-f7c0c4071745)

이를 통해 불필요한 의존성을 제거하고 필요한 메서드에만 의존한다. 포트의 이름이 명확하다는 장점이 있다. 

# 영속성 어댑터 나누기

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/40cdded6-15e6-4f06-a5aa-8af9d742853f)

영속성 연산이 필요한 도메인클래스(애그리거트) 당 하나의 영속성 어댑터를 구현하는 방식을 선택할 수 있다. 또한 기술 별로 나눌 수 있다. 

- JPA, ORM 의 영속성 포트를 구현하며 성능을 개선하기 위해 평범한 SQL 을 이용하는 다른 종류의 포트를 구현하는 경우.
- 애그리거트 당 하나의 영속성 어댑터 방식을 이용한다면,명확한 바운디드 컨텍스트를 나누는데 도움이 된다.
    - 바운디드 컨텍스트 : 경계를 의미하며, 어떤 맥락이 다른 맥락에 있는 무엇인가를 필요로 하면, 전용 인커밍 포트를 통해 접근한다는 뜻이다.

![image](https://github.com/ZI-won-ZONE-ha/CS_JONGJIBU/assets/87687210/c76b7108-8d06-4186-9766-7b761ede96f8)

# JPA 예제

- 코드
    
    account 
    
    ```java
    package io.reflectoring.buckpal.account.domain;
    @AllArgsConstructor(access = AccessLevel.PRIVATE)
    public class Account {
    
    	@Getter private final AccountId id;
    	@Getter private final Money baselineBalance
    	@Getter private final ActivityWindow activityWindow;
    
    	public static Account withoutId(
    					Money baselineBalance,
    					ActivityWindow activityWindow) {
    		return new Account(null, baselineBalance, activityWindow);
    	}
    
    	public static Account withId(
    					AccountId accountId,
    					Money baselineBalance,
    					ActivityWindow activityWindow) {
    		return new Account(accountId, baselineBalance, activityWindow);
    	}
    
    	public Optional<AccountId> getId(){
    		return Optional.ofNullable(this.id);
    	}
    
    	public Money calculateBalance() {
    		/** 생략 **/
    	}
    
    	public boolean withdraw(Money money, AccountId targetAccountId) {
    		/** 생략 **/
    	}
    
    	private boolean mayWithdraw(Money money) {
    		return Money.add(
    				this.calculateBalance(),
    				money.negate())
    				.isPositiveOrZero();
    	}
    
    	public boolean deposit(Money money, AccountId sourceAccountId) {
    		/** 생략 **/
    	}
    
    	@Value
    	public static class AccountId {
    		private Long value;
    	}
    }
    ```
    
    AccountJpaEntity
    
    ```java
    package io.reflectoring.buckpal.account.adapter.out.persistence;
    
    @Entity
    @Table(name = "account")
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    class AccountJpaEntity {
    
    	@Id
    	@GeneratedValue
    	private Long id;
    
    }
    ```
    
    - ActivityJpaEntity
    
    ```java
    package io.reflectoring.buckpal.account.adapter.out.persistence;
    
    @Entity
    @Table(name = "activity")
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    class ActivityJpaEntity {
    
    	@Id
    	@GeneratedValue
    	private Long id;
    
    	@Column
    	private LocalDateTime timestamp;
    
    	@Column
    	private Long ownerAccountId;
    
    	@Column
    	private Long sourceAccountId;
    
    	@Column
    	private Long targetAccountId;
    
    	@Column
    	private Long amount;
    
    }
    
    ```
    
    - ActivityRepository
    
    ```java
    interface ActivityRepository extends JpaRepository<ActivityJpaEntity, Long> {
    
    	@Query("select a from ActivityJpaEntity a " +
    			"where a.ownerAccountId = :ownerAccountId " +
    			"and a.timestamp >= :since")
    	List<ActivityJpaEntity> findByOwnerSince(
    			@Param("ownerAccountId") Long ownerAccountId,
    			@Param("since") LocalDateTime since);
    
    	@Query("select sum(a.amount) from ActivityJpaEntity a " +
    			"where a.targetAccountId = :accountId " +
    			"and a.ownerAccountId = :accountId " +
    			"and a.timestamp < :until")
    	Long getDepositBalanceUntil(
    			@Param("accountId") Long accountId,
    			@Param("until") LocalDateTime until);
    
    	@Query("select sum(a.amount) from ActivityJpaEntity a " +
    			"where a.sourceAccountId = :accountId " +
    			"and a.ownerAccountId = :accountId " +
    			"and a.timestamp < :until")
    	Long getWithdrawalBalanceUntil(
    			@Param("accountId") Long accountId,
    			@Param("until") LocalDateTime until);
    
    }
    ```
    
    - AccountPersistenceAdapter
    
    ```java
    @RequiredArgsConstructor
    @Component
    class AccountPersistenceAdapter implements
    		LoadAccountPort,
    		UpdateAccountStatePort {
    
    	private final SpringDataAccountRepository accountRepository;
    	private final ActivityRepository activityRepository;
    	private final AccountMapper accountMapper;
    
    	@Override
    	public Account loadAccount(
    					AccountId accountId,
    					LocalDateTime baselineDate) {
    
    		AccountJpaEntity account =
    				accountRepository.findById(accountId.getValue())
    						.orElseThrow(EntityNotFoundException::new);
    
    		List<ActivityJpaEntity> activities =
    				activityRepository.findByOwnerSince(
    						accountId.getValue(),
    						baselineDate);
    
    		Long withdrawalBalance = orZero(activityRepository
    				.getWithdrawalBalanceUntil(
    						accountId.getValue(),
    						baselineDate));
    
    		Long depositBalance = orZero(activityRepository
    				.getDepositBalanceUntil(
    						accountId.getValue(),
    						baselineDate));
    
    		return accountMapper.mapToDomainEntity(
    				account,
    				activities,
    				withdrawalBalance,
    				depositBalance);
    
    	}
    
    	private Long orZero(Long value){
    		return value == null ? 0L : value;
    	}
    
    	@Override
    	public void updateActivities(Account account) {
    		for (Activity activity : account.getActivityWindow().getActivities()) {
    			if (activity.getId() == null) {
    				activityRepository.save(accountMapper.mapToJpaEntity(activity));
    			}
    		}
    	}
    
    }
    ```
    

다음과 같이 Account와 AccountJpaEntity, Activity와 ActivityJpaEntity 에는 양방향 매핑이 존재한다. 

- 매핑하지 않는 전략은 8장에서 다룬다.
    - 이는 JPA 에 종속적일 수 밖에 없다. JPA는 기본 생성자를 필요로 한다.
    - 영속성 계층에서는 성능 측면에서 @ManyToOne 관계가 적절할 수 있지만, 데이터의 일부만 가져오기를 바라면 도메인 모델에서는 이 관계가 반대가 될 수 있다.
- 영속성 타협 없이 풍부한 도메인 모델을 원한다면, 도메인 모델과 영속성 모델을 매핑하는 것이 좋다.

# 데이터베이스 트랜잭션

영속성 어댑터는 어떤 데이터베이스 연산이 같은 유스케이스에 포함되는지 알지 못하기 떄무넹 언제 트랜잭션을 열고 닫을지 결정할 수 없다. 이 책임은 영속성 어댑터 호출을 관장하는 서비스에 위임해야 한다.

# 유지보수 가능한 소프트웨어를 만드는 데 어떻게 도움이 될까?

영속성 어댑터를 만들면 도메인 코드가 영속성과 분리된 풍부한 도메인 모델을 만들 수 있다. 

좁은 포트 인터페이스로, 포트마다 다른 방식으로 구현할 수 있는 유연함 또한 생긴다. 그리고, 다른 영속성 기술을 교체할 때도 도메인 코드를 신경쓰지 않아도 된다.
