---
title: 스프링 배치
tags: spring java web
layout: post
---

## BATCH

스케쥴링되서 실행이된든 api를 사용해 호출을 하던 cron을 사용해 호출을 하던 사용자 인터렉션과 상관없이 혼자 실행되는 것을 스프링 배치라고함

주로 후속 처리나 일괄 처리, 대용량 처리 등을 이유로 사용한다.

>  QUARTZ는 스프링 배치를 실행시키는 Spring Batch의 보안제 역할이지 대체제가 아니다.

### 배치가 필요한 상황

- 일정 주기로 실행되어야 할 때
- 실시간 처리가 어려운 대량의 데이터를 처리해야할 때
- 즉 대용량 데이터 처리가 절대적인 요구 사항

### 배치 특징

- 배치는 기본적으로 대용량 처리를하기 위해서 chunk size , paging size등을 프레임워크 단에서 다 지원함
- 스프링 배치에서는 모든 데이터를 메모리에 쌓지 않는 조회방식이 기본 방식


- paging 혹은 cursor로 pageSize만큼만 읽어 오고 chunkSize만큼만 commit한다.

### 배치 구성 요소

- job, step, tasklet, reader, writer, processor가 존재
- 스프링 배치는 외부에서 파라미터를 주입받아 Batch 컴포넌트에서 사용할 수 있는데 이를 `JobParameter`라고 한다.
  - @Value("#{JobParameters[파라미터이름]}") String 이름
- @JobScope
  - 없으면 JobParameter가 작동이 안된다.
  - job이 실행되는 시점에 bean이 생성이 된다.
  - Late Binding (늦은 할당)
- @StepScope
  - 없으면 jobParameters가 작동이 안된다
  - Step이 실행될때 Bean이 생성이 된다.
- JobParameter
  - jobParameter 값에 따라 Reader와 Writer를 교체할 수 있다.
  - Long/String/Double/Date를 지원
  - Enum/LocalDate/LocalDateTime은 지원을 안한다.
  - 그래서 LocalDateTime을 사용해야하는 경우 매번 형변환을 진행함

#### JobParameter 컨버트

@Value를 사용해 한번만 컨버트할 수 있음

```java
@Getter
public class CreateDateJobParameter {
  private LocalDate createDate;

  @Value("#{jobParameters[createDate]}")
  public void setCreateDate(String createDate) {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    this.createDate = LocalDate.parse(createDate, formatter);
  }
}
```

```java
public class JobParameterBatchConfiguration {
  private static final String BATCH_NAME = "jobParameterBatch"
  private final CreateDateJobParameter jobParameter;
  
  @Bean(BATCH_NAME + "jobParameter");
  @JobScope
  public CreateDateJobParameter jobParameter() {
    return new CreateDateJobParameter();
  }
}
```

```java
public JpaPagingItemReader<Product> reader() {
  Map<String, Object> params = new HashMap<>();
  params.put("createDate", jobParameter.getCreateDate());
  
  return new JpaPagingItemReaderBuilder<Product>()
    .name(BATCH_NAME + "_reader")
    .entityManagerFactory(entityManagerFactory)
    .pageSize(chunkSize)
    .queryString("SELECT * FROM product")
    .parameterValues(params)
    .build();
}
```

#### 스프링 배치 메타 스키마

![Spring Batch Meta-Data ERD](https://docs.spring.io/spring-batch/docs/current/reference/html/images/meta-data-erd.png)



##### Batch Job Instance

job이 실행되면 새로운 row를 만든다. job이름과 job이 시작될때 넘겨받은 파라미터를 Serialize (직렬화)해서 저장한다.

| 필드명             | 설명                                       |
| --------------- | ---------------------------------------- |
| JOB_INSTANCE_ID | 인스턴스 Id, PK이다.<br/>getId 메소드를 이용해 value를 가져올 수 있다 |
| VERSION         |                                          |
| JOB_NAME        | 오브젝트로 부터 얻은 job 이름, 인스턴스를 식별하기 위해 필요하기 때문에 null이 될수 없다. |
| JOB_KEY         | 같은 job의 인스턴스로부터 분리해서 식별하기 위한 직렬화키다.      |

##### Batch Job Execution Params

job 파라미터가 저장된다. 

| 필드명              | 설명                                       |
| ---------------- | ---------------------------------------- |
| JOB_EXECUTION_ID | job execution과 연결짓기 위한 외래키다. 각 execution에 관해서는 여러개의 로우를 가질 수 있다. |
| TYPE_CD          | 저장된 값의 타입을 나타낸다. null이 될 수 없다.           |
| KEY_NAME         | 파라미터 키 이름                                |
| STRING_VAL       |                                          |
| DATE_VAL         |                                          |
| LONG_VAL         |                                          |
| DOUBLE_VAL       |                                          |
| IDENTIFYING      | 매개 변수가 instance의 job_key에 포함이 될지 말지 결정하는 플래그 |

##### Batch Job Execution

job의 실패+성공횟수만큼 row가 생성된다. job의 실행 내용을 담고 있다.

| 필드명              | 설명                                       |
| ---------------- | ---------------------------------------- |
| JOB_EXECUTION_ID | execution을 구분하기 위한 기본키                   |
| VERSION          |                                          |
| JOB_INSTANCE_ID  | 어느 인스턴스에 속해있는 가를 나타내는 외부키. 각 인스턴스는 한개 이상의 execution을 갖는다. |
| CREATE_TIME      | execution이 생성된 시간                        |
| START_TIME       | execution이 실행된 시간                        |
| END_TIME         | execution이 끝난 시간, 실패 혹은 성공은 고려하지 않는다. job이 running이 아니면서 이 컬럼이 비어있는 것은 프레임워크에서 에러가 난거라고 볼 수 있다. |
| STATUS           | `COMPLETED`, `STARTED` 두개의 상태를 가지는 execution 상태를 나타낸다. |
| EXIT_CODE        | execution의 종료 코드를 나타낸다. 커맨드 라인 job일때는 숫자로 전환된다 |
| EXIT_MESSAGE     | job이 어떻게 끝났는지에 대한 상세 메시지가 나온다. 만약 실패로 끝났다면 스택 트레이스가 있을 것이다. |
| LAST_UPDATED     | 마지막으로 지속된 시간을 나타내는 타임스탬프                 |

##### Batch Step Execution

step 실행 내용을 담고 있다. step의 실패+성공 횟수만큼 row가 생성된다.

| 필드명                | 설명                                       |
| ------------------ | ---------------------------------------- |
| STEP_EXECUTION_ID  | 기본키                                      |
| VERSION            |                                          |
| STEP_NAME          | 스텝 이름                                    |
| JOB_EXECUTION_ID ? | 외래키, jobExecution, StepExecution, Step 등의 execution 이름이 올 수 있ㄷ |
| START_TIME         | 실행 시간을 나타낸다                              |
| END_TIME           | 끝난 시간을 나타낸다. 실패했는지 성공했는지 여부는 고려하지 않는다. 실행중이지 않으면서 이 컬럼이 비어있다면 에러가 난거라고 볼 수 있다. |
| STATUS             | `COMPLETED`나 `STARTED` 둘중 하나의 값을 갖는다.    |
| COMMIT_COUNT       | 스탭이 실행시간 동안 얼마나 커밋되었나                    |
| READ_COUNT         | 실행동안 얼마나 아이템을 읽었나                        |
| FILTER_COUNT       | 실행동안 얼마나 아이템을 필터했나                       |
| WRITE_COUNT        | 실행되는 동안 얼마나 쓰이고 커밋되었는가                   |
| READ_SKIP_COUNT    | 아이템이 얼마나 읽기 스킵되었는가                       |
| WRITE_SKIP_COUNT   | 아이템이 얼마나 스킵 되었는가                         |
| PROCESS_SKIP_COUNT | 아이템이 얼마나 processing 스킵되었는가               |
| ROLLBACK_COUNT     | 롤백한 수, 이 카운트가 각 롤백이 일어났다면                |
| EXIT_CODE          |                                          |
| EXIT_MESSAGE       | 어떻게 끝났는지 내용 포함                           |
| LAST_UPDATED       |                                          |

##### Batch Job Execution Context

Job의 ExecutionContext와 관련된 모든 정보를 가진다. 정확히 하나하나이고 작업 실행에 필요한 모든 데이터가 포함되어 있다. 일반적으로 실패 후 검색해야 하는 상태를 나타내므로 중단된 위치에서 시작할 수 있다.

| 필드명                | 설명                                      |
| ------------------ | --------------------------------------- |
| JOB_EXECUTION_ID   | 외래키, JobExecution과 관련된 행이 두개 이상 있을 수 있음 |
| SHORT_CONTEXT      |                                         |
| SERIALIZED_CONTEXT | 모든 serialized된 컨텍스트                     |

##### Batch Step Execution Context

Step의 ExecutionContext와 관련된 모든 정보를 보유한다. 특정 단계 실행을 위해 유지해야하는 모든 데이터가 포함되어 있다. 이 데이터는 일반적으로 실패 후 검색해야 하는 상태를 나타내므로 중단된 위치에서 시작할 수 있다.

| 필드명                | 설명                         |
| ------------------ | -------------------------- |
| STEP_EXECUTION_ID  | 외래키                        |
| SHORT_CONTEXT      | SERIALIZED_CONTEXT의 스트링 버전 |
| SERIALIZED_CONTEXT | 모든 serialized된 컨텍스트        |

#### 스프링 배치 관리도구들

Cron

Spring MVC + Api Call

Spring Batch Admin (Deprecated)

Quartz + Admin

CI Tools (Jenkins/ Teamcity 등)



#### 일반적인 배치 Jar 실행 명령어

```shell
java -jar Application.jar \
--job.name=job이름 \
job파라미터이름1=job파라미터값1 \
job파라미터이름2=job파라미터값2
```

#### 젠킨스 공통 설정 관리

```shell
java -jar \
### 모든 Batch Job의 공통 코드
-XX: +UseG1GC \
-Dspring.profiles.active=dev \
Application.jar \
###
--job.name=job이름 \
job파라미터이름1=job파라미터값1 \
job파라미터이름2=job파라미터값2
```

젠킨스 설정에서 Global Properties로 설정할 수 있음

```shell
java -jar \
$BATCH_JAR \
--job.name=job이름 \
job파라미터이름1=job파라미터값1 \
job파라미터이름2=job파라미터값2
```

#### 무중단 배포

`readlink`를 사용, 링크파일을 실행하는 것이 아닌 원본 파일을 실행

`java -jar $(readlink / ../app.jar)`가 `java -jar / ../v1.jar`

- app.jar는 v1.jar를 링크중


- v1.jar를 실행중
- 배포하여 app.jar를 v2.jar로 링크 (v1.jar는 영향 없음)
- 젠킨스는 `java -jar $(readlink / ../app.jar)` => `java -jar / ../v2.jar`

`readlink`를 전부 선언하기 귀찮다면 이것 역시 공통 설정으로 환경설정하면 됨

`jaca -jar ${JAR_OPTS} \$(\${JAR_NAME})`



#### 같은 Job인데  스케줄만 다르게 파라미터만 다르게 하고싶을 때

원본 Job 변경을 한 곳에서 하기 위해 파이프라인을 사용한다.

여러 작업이 순차적으로 실행되어야 할때 step으로 나누기보다는 파이프라인을 고려



#### 멱등성

연산을 여러번 적용하더라도 결과가 달라지지 않는 성질

애플리케이션 개발에서 멱등성이 깨지는 경우 > 제어할 수 없는 코드를 생성할 때

```java
public JpaPagingItemReader<Product> reader(@Value("#{jobParameters[createDate]}") String createDate) {
  params.put("createDate")
  
  return new JpaPagingItemReaderBuilder<Product>()
    .name(BATCH_NAME + "_reader")
    .entityManagerFactory(entityManagerFactory)
    .pageSize(chunkSize)
    .queryString("select ..")
    .parameterValues(param)
    .build();
}
```



Date Parameter 플러그인

LocalDate.now(), LocalDate.now().plusDays(1), LocalDateTime.now() 등을 파라미터로 지원



#### 테스트코드

##### 테스트코드 1

```java
@Configuration
@ConditionalOnProperty(name = "job.name", havingValue = "JOB_NAME") // job.name에서 havingValue의 값일때만 configuration이 활성화된다. 그 외에는 스프링 빈에 올라가지 않는다.
public class CustomerCopyBatchConfiguration {
  public static final String JOB_NAME = "customerCopyJob";
  private static final String STEP_NAME = "customerCopyStep"
}
```

```kotlin
@TestPropertySource("job.name=customerCopyJob") // 젠킨스 실행하듯이 똑같이 실행할 수 있다.
@SpringBootTest
class CustomerCopyBatchRealTest extends Specification {
  @Autowired
  JobLauncherTestUtils jobLauncherTestUtils;
  @Autowired
  CustomerRepository customerRepository;
  
  def
     given:
  customerRepository.save(new Customer());
  
  JobParametersBuilder builder = new JobParametersBuilder();
  
  when:
  JobExecution jobExecution = jobLauncherTestUtils.launchJob(builder.toJobParameters())
}
```

##### 테스트코드 2

테스트케이스 100개가 넘기 시작하면서 기하 급수적으로 느려지는 전체 테스트 수행 속도

원인은 `@ConditionalOnProperty`

스프링은 전체 테스트 수행시 Environment가 변경될 때마다 Spring Context 재시작

Environment가 변경되는 조건

- 테스트코드에서 @MockBean / @SpyBean 사용할 때
- 테스트 코드에서 @TestPropertySource로 환경변수 변경할때
- @ConditionalOnProperty로 테스트마다 Config가 다를때

해결법: 모든 Config를 Loading 한 뒤에 원하는 Job Bean을 실행한다.



```java
public class JobTestUtils {
  @Autowired
  private ApplicationContext applicationContext;
  @Autowired
  private JobRepository jobRepository;
  @Autowired
  private JobLauncher jobLauncher;
  
  public JobLauncherTestUtils getJobTester(String jobName) {
    Job bean = applicationContext.getBean(jobName, Job.class);
    JobLauncherTestUtils jobLauncherTestUtils = new JobLauncherTestUtils();
    JobLauncherTestUtils.setJobLauncher(jobLauncher);
    JobLauncherTestUtils.setJobRepository(jobRepository);
    JobLauncherTestUtils.setJob(bean);
    return jobLauncherTestUtils;
  }
}
```

```java
// ConditionalOnProperty 모두 제거

@RunWith(SpringRunner.class)
@SpringBootTest
public class JobParameterBatchConfigurationTest {
  @Autowired private JobTestUtils jobTestUtils;
  @Autowired private ProductRepository productRepository;
  
  @Test
  public void jobParameter정상출력_확인() throws Exception {
    //given
    
    JobParameters jobParameters = JobParameterBuilder<>().addString("createDate", createDate)
      
      JobExecution jobexecution = jobTestUtils.getJobTester(JOB_NAME).launchJob(jobParameters);
    
    assertThat(jobExecution.getStatus()).isEqualTo(BatchStatus.COMPLETED);
  }
}
```



#### JPA & Spring Batch

JPA N + 1 문제

@OneToMany 관계에서 하위 엔티티들을 Lazy Loading으로 가져올때마다 조회 쿼리가 추가로 발생하는 이슈

해결방법

1. Join Fetch

   `SELECT adBond FROM AdBond adBond JOIN FETCH adBond.offsetBound`

   하위 엔티티 2개 종류 이상에서 Join Fetch 사용시 MultipleBagFetchException

2. default_batch_fetch_size

   ```yaml
   spring:
     jpa:
       properties:
         hibernate.default_batch_fetch_size: 1000
       show-sql: true
   ```

   하위 엔티티를 Loading 할때 지정된 숫자만큼 상위 엔티티 Id를 Where IN ()에 넣어서 조회한다.

   하지만 이 옵션은 jpaPagingItemReader에서는 작동하지 않는다.

3. Persist Writer

   모든 item에 대해 merge 수행 -> 처음 데이터가 save가 될때도 update 쿼리가 항상 실행 됨

   ​

## 출처

[[우아한테크세미나\] 190926 우아한스프링배치 by 우아한형제들 이동욱님 - YouTube](https://www.youtube.com/watch?v=_nkJkWVH-mo)

[Meta-Data Schema (spring.io)](https://docs.spring.io/spring-batch/docs/current/reference/html/schema-appendix.html#metaDataArchiving)