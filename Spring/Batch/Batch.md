# Spring Batch

# 기본 구조
### application.yml

```yaml
spring:
  profiles:
    active: local
spring.batch.job.names: ${job.name:NONE}
---
spring:
  datasource:
    driver-class-name: org.postgresql.Driver
    url: 
    username: 
    password: 
---
# JPA 설정
jpa:
  hibernate:
    ddl-auto: update
  properties:
    hibernate:
      dialect: org.hibernate.dialect.PostgreSQLDialect
      #        show_sql: true
      format_sql: true
devtools:
  livereload:
    enabled: true
```

### Job Configuration
```java
@RequiredArgsConstructor // 셍상지 DI를 위한 Lombok 어노테이션
@Configuration
public class SimpleJobConfiguration {
    private static final Logger Logger = LoggerFactory.getLogger(SimpleJobConfiguration.class);
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    @Bean
    public Job simpleJob() {
        reutrn jobBuilderFactory.get("simpleJob")
                .start(simpleStep1())
                .builde();
    }
    @Bean
    public Step simpleStep1() {
        return stepBuilderFactory.get("simpleStep1")
        .tasklet((contribution, chunkContext) -> {
            logger.info(">>>>> This is step1");
            return RepeatStatus.FINISHED;
        })
        .build();
    }
}
```

`@Configuration`

​	Spring Batch의 모든 Job은 `@Configuration`으로 등록해서 사용

`jobBuilderFactory.get("simpleJob")`

​	simpleJob이란 이름의 Batch Job 생성

​	job의 이름은 별도로 지정하지 않고 이렇게 Builder를 통해 지정

`stepBuilderFactory.get("simpleStep1")`

​	simpleStep1.이란 이름의 Batch Step 생성

​	jobBuilderFactory.get("simpleJob")과 마찬가지로 Builder를 통해 이름을 지정

`.tasklet((contribution, chunkContext))`

​	Step 안에서 수행될 기능들을 명시

​	Tasklet은 Step안에서 단일로  수행될 커스텀한 기능들을 선언할때 사용



Spring Batch에서 **Job은 하나의 배치 작업 단위**

Job 안에는 여러 Step이 존재하고 Step 안에 Tasklet 혹은 Reader&Processor&Writer 묶음이 존재

​	Tasklet과 Reader&Processor&Writer 한 묶음이 같은 레벨

------



#메타테이블

### BATCH_JOB_INSTANCE

Job Parameter(Spring Batch가 실행될때 외부에서 받을 수 있는 파라미터)에 따라 생성되는 테이블

같은 Batch Job이라도 Job Parameter가 다르면 BATCH_JOB_INSTANCE에 기록되고, Job Parameter가 같으면 기록되지 않음

### BATHCH_JOB_EXECUTION

BATCH_JOB_INSTANCE와 부모-자식관계

BATCH_JOB_INSTANCE가 성공/실패 했던 내역을 모두 가지고 있음

Spring Batch는 동일한 Job Parameter로 '성공'한 기록이 있을때만 재수행이 안됨

​	먼저 throw new IllegalArgumentException으로 실행에 실패했다면 BATCH_JOB_INSTANCE에 실행데이터가 만들어졌다. BATCH_JOB_EXECUTION에는 '실	패'했다는 데이터가 만들어진다.

​	같은 파라미터로 exception 발생안하도록 다시 실행시키면 같은 Job Parameter로 실행한 INSTANCE가 이미 있지만 실행에 성공하고 BATCH_JOB_EXECUTION에는 '성공'했다는 데이터가 만들어진다.

### BATHCH_JOB, BATHCH_JOB_INSTANCE, BATHCH_JOB_EXECUTION

Job

Job Instance

​	특정 Job Parameter로 실행한  Job

Job Execution

​	특정 Job P{arameter로 실행한 Job의 1번째 시도 혹은 다음번 시도

### BATHCH_JOB_EXECUTION_PARAM

BATHCH_JOB_EXECUTION 테이블이 생성될 당시에 입력 받은 Job Parameter

------



#Spring Batch Job Flow

**Step**

실제 Batch 작업을 수행하는 역할 Batch로 실제 처리하고자 하는 기능과 설정을 모두 포함

Step들간의 순서 혹은 처리 흐름을 제어할 필요가 있음

### Next

Step을 순차적으로 연결시킬때 사용

```java
@Slf4j // log 사용을 위한 Lombok 어노테이션
@RequiredArgsConstructor // 생성자 DI를 위한 Lombok 어노테이션
@Configuration
public class StepNextJobConfiguration {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;
    @Autowired
    private StepBuilderFactory stepBuilderFactory;
    private static final Logger logger = LoggerFactory.getLogger(SimpleJobConfiguration.class);
    @Bean
    public Job stepNextJob() {
        return jobBuilderFactory.get("stepNextJob")
                .start(step1())
                .next(step2())
                .next(step3())
                .build();
    }
    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>> This is Step1");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>> This is Step2");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
    @Bean
    public Step step3() {
        return stepBuilderFactory.get("step3")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>> This is Step3");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```



------

### 지정한 Batch Job만 실행하기

application.yml

​	`spring.batch.job.names: ${job.name:NONE}` 추가



Program arguments로 `job.name` 값이 넘어오면 해당 값과 일치하는 Job만 실행

`job.name`이 있으면 `job.name`값을 할당, 없으면 `NONE` 할당

`spring.batch.jab.names`에 `NONE`이 할당되면 어떤 배치도 실행하지 않음



Program arguments 입력 예시 : `--job.name=stepNextJob` `{Job Parameter}`

실제 운영 환경에서는 `java -jar batch-application.jar --job.name=simpleJob`과 같이 실행

------



### 조건별 흐름 제어 FLOW

next로 Step이 순차적으로 실행 될 때, 앞의 Step에서 오류가 나면 나머지 Step은 실행 못하는데 오류가 나도 다른 Step을 실행하도록 조건을 추가할 수 있다



조건

- on()

  캐치할 ExitStatus 지정

  on("*")일 경우 모든 ExitStatus가 지정

- to()

  다음으로 이동할 Step 지정

- from()

  start(step()).on() 이후에 추가로 이벤트를 캐치하려면 .from.on()을 써야함

- end()

    1. on(*) 뒤에 있는 end는 FlowBuilder를 반환

       이러한 end를 계속 사용하면 from을 이어갈 수 있음

    2. build() 앞에 있는 end는 FlowBuilder를 종료



on은 ExitStatus에 해당하므로 분기처리를 위해 상태값 조정이 필요하다면 ExitStatus를 조정해야함

`contribution.setExitStatus(ExitStatus.FAILED);`

```
@Bean
    public Step conditionalJobStep1() {
        return stepBuilderFactory.get("step1")
                .tasklet((contribution, chunkContext) -> {
                    logger.info(">>>>> This is stepNextConditionalJob Step1");
                    /**
                     ExitStatus를 FAILED로 지정한다.
                     해당 status를 보고 flow가 진행된다.
                     **/
                    contribution.setExitStatus(ExitStatus.FAILED);
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
```



------

### BatchStatus vs ExitStatus

**BatchStatus**

Job 또는 Step의 실행 결과를 Spring에서 기록할 때 사용하는 Enum

사용되는 값 : COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN 등



**ExitStatus**

Enum이 아님

`public static final ExitStatus UNKNOWN = new ExitStatus("UNKNOWN");`



기본적으로 ExitStatus의 exitCode는 Step의 BatchStatus와 같도록 설정되어 있지만, 본인이 **커스텀한 exitCode를 사용**하려면

```java
public class SkipCheckingListener extends StepExecutionListenerSupport {
    public ExitStatus afterStep(StepExecution stepExecution) {
        String exitCode = stepExecution.getExitStatus().getExitCode();
        if (!exitCode.equals(ExitStatus.FAILED.getExitCode()) && 
              stepExecution.getSkipCount() > 0) {
            return new ExitStatus("COMPLETED WITH SKIPS");
        }
        else {
            return null;
        }
    }
```
`StepExecutionListener`에서 먼저 Step이 성공적으로 수행되었는지 확인
```
!exitCode.equals(ExitStatus.FAILED.getExitCode()
```
StepExecution의 skip 횟수가 0보다 큰 경우
```
stepExecution.getSkipCount() > 0
```
커스텀한 exitCode를 갖는 ExitEtatus를 반환
```
return new ExitStatus("COMPLETED WITH SKIPS");
```
------
### Decide
따로 Step들 간의 분기를 담당하고 다양한 분기처리를 위해 `JobExecutionDecider` 사용
```java
@Slf4j
@Configuration
@RequiredArgsConstructor
public class DeciderJobConfiguration {
    private static final Logger logger = LoggerFactory.getLogger(SimpleJobConfiguration.class);
    
    private final JobBuilderFactory jobBuilderFactory;
    
    private final StepBuilderFactory stepBuilderFactory;
    
    @Bean
    public Job deciderJob() {
        return jobBuilderFactory.get("deciderJob")
                .start(startStep())
                .next(decider()) // 홀수 & 짝수 구분
                .from(decider()) // decider의 상태가
                    .on("ODD") // ODD라면
                    .to(oddStep()) // oddStep로 간다.
                .from(decider()) // decider의 상태가
                    .on("EVEN") // ODD라면
                    .to(evenStep()) // evenStep로 간다.
                .end() // builder 종료
                .build();
    }
    
    @Bean
    public Step startStep() {
        return stepBuilderFactory.get("startStep")
                .tasklet((contribution, chunkContext) -> {
                    logger.info(">>>>> Start!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
    
    @Bean
    public Step evenStep() {
        return stepBuilderFactory.get("evenStep")
                .tasklet((contribution, chunkContext) -> {
                    logger.info(">>>>> 짝수입니다.");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
    
    @Bean
    public Step oddStep() {
        return stepBuilderFactory.get("oddStep")
                .tasklet((contribution, chunkContext) -> {
                    logger.info(">>>>> 홀수입니다.");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
    
    @Bean
    public JobExecutionDecider decider() {
        return new OddDecider();
    }
    
    public static class OddDecider implements JobExecutionDecider {
        @Override
        public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
            Random rand = new Random();
            int randomNumber = rand.nextInt(50) + 1;
            logger.info("랜덤숫자: {}", randomNumber);
            if(randomNumber % 2 == 0) {
                return new FlowExecutionStatus("EVEN");
            } else {
                return new FlowExecutionStatus("ODD");
            }
        }
    }
}
```
JobExecutionDecider 인터페이스를 구현한 OddDecider가 분기 로직을 전담
```
public static class OddDecider implements JobExecutionDecider
```
Step으로 처리하는 것이 아니기 때문에 ExitStatus가 아닌 FlowExecutionStatus로 상태관리
```
return new FlowExecutionStatus("EVEN");
```


# Spring Batch Scope & Job Parameter
## JobParameter 와 Scope
JobParameter : Spring Batch의 여러 Batch 컴포넌트에서 사용하도록 외/내부에서 받아오는  파라미터
```
@Value("#{jobParameters[파라미터명]}")
```
​	`jobParameters` 외에도 `jobExecutionContext`, `stepExecutionContext` 등도 SpEL로 사용할 수 있다.
​	@JobScope에서 stepExecutionContext는 사용할 수 없다.
JobParameter를 사용하기 위해 Spring Batch 전용 Scope를 선언해야함
1. JobScope
2. StepScope

##### **참고**

##### < 기억보단기록을 (Jojoldu)님 :  https://jojoldu.tistory.com/325 >
