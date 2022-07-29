# Spring Batch

## 배치란

- Read — Extract
- Process — Transform
- Write — Load

## 배치 시나리오

- 배치 프로세스를 주기적으로 커밋한다.(커밋 전략 제공)
- 동시 다발적으로 Job의 배치처리를 진행한다.(대용량 병렬처리 제공)
- 실패시 수동or스케줄링에 의해 재시작 가능
- 의존관계가있는 Step여러개 순차처리
- Flow 구성을 통해서 유연한 배치 설계 구성
- 반복, 재시도, skip 처리 가능

## 배치 구성

![Untitled](Spring%20Batch%20e08eb26a13f342e6a5d5b2f9eb5aa7c0/Untitled.png)

- 스프링 배치 프레임워크를 통해 개발자가 만든 모든 Job과 Custom 코드를 포함
- 업무 로직에 집중하고 공통적인 기반기술은 프레임웤이 담당하게 한다.

- Job을 실행,모니터링,관리하는 API
- JobLauncher,Job,Step,Flow등이 속해있음

- Application, Core 모두 공통infrastructure 위에서 빌드
- Job 실행의 흐름과 처리를 위한 틀을 제공
- Reader,Processor,Writer,Skip,Retry등

## 스프링 배치 활성화

@EnableBatchProcessing

- 4개의 설정 클래스를 실행시키며 스프링 배치의모든 초기화 및 실행 구성이 이루어진다.
    - BatchAutoConfiguration
        - 스프링 배치가 초기화 될 때 자동으로 실행되는 설정 클래스
        - Job을 수행하는 JobLauncherApplicaitonRunner 빈을 생성
    - SimpleBatchConfiguration
        - JobBuilderFactory & StepBuilderFactory 생성
        - 스프링 배치의 주요 구성 요소를 프록시 객체로 생성
    - BatchConfigurerConfiguration
        - BasicBatchConfigurer
            - SimpleBatchConfiguration 에서 생성한 프록시 객체의 실제 대상 객체를 생성하는 설정클래스
            - 빈으로 의존성 주입받아서 주요 객체들을 참조해서 사용할 수 있다.
        - JpaBatchConfigurer
            - JPA 관련 객체를 생성하는 설정 클래스
        - Custom BatchConfigurer 인터페이스를 구현하여 사용할 수 있음
- 스프링 부트 배치 autoconfiguration클래스가 실행되면서 빈으로 등록된 모든 Job을 검색해서 초기화와 동시에 Job을 수행하도록 구성된다.

@EnableBatchProcessing 초기화 순서

1. SimpleBatchConfiguration
2. BatchConfigurerConfiguration
    - BasicBatchConfigurer
3. BatchAutoConfiguration

![Untitled](Spring%20Batch%20e08eb26a13f342e6a5d5b2f9eb5aa7c0/Untitled%201.png)

### Job — 일감

### Step — 일감 내부 단계

### Tasklet — 일감 내부 단계에서의 작업내용

### e.g.

```java
@Configuration
@RequiredArgsConstructor
public class HelloJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job helloJob() {
        return jobBuilderFactory.get("helloJob")
            .start(helloStep1())
            .next(helloStep2())
            .build();
    }

    @Bean
    public Step helloStep1() {
        return stepBuilderFactory.get("helloStep1")
            .tasklet(new Tasklet() {
                @Override
                public RepeatStatus execute(StepContribution contribution,
                    ChunkContext chunkContext) throws Exception {

                    System.out.println("-----------------------");
                    System.out.println("hello Batch1");
                    System.out.println("-----------------------");
                    return RepeatStatus.FINISHED;
                }
            })
            .build();
    }

    @Bean
    public Step helloStep2() {

        return stepBuilderFactory.get("helloStep2")
            .tasklet(new Tasklet() {
                @Override
                public RepeatStatus execute(StepContribution contribution,
                    ChunkContext chunkContext) throws Exception {

                    System.out.println("-----------------------");
                    System.out.println("hello Batch2");
                    System.out.println("-----------------------");
                    return RepeatStatus.FINISHED;
                }
            })
            .build();
    }
}

```

## 스프링 배치 메타데이터

- DB와 연동할 경우 메타테이블 생성해야함
- 스프링 배치에서 스키마 제공
- 스프링 배치의 실행 및 관리를 위한 스키마
- 과거, 현재의 실행에 대한 세세한 정보, 실행에 대한 성공 실패 여부 등 관리.
- org.springframework.batch.core.schema-*.sql

![Untitled](Spring%20Batch%20e08eb26a13f342e6a5d5b2f9eb5aa7c0/Untitled%202.png)

다음 쿼리를 실행해서 쿼리생성 가능

- 혹은 자동생성 가능
- spring.batch.jdbc.initial-schema : ALWAYS,(EMBEDED),NEVER
    - 운영할때는 NEVER 사용 추천

스키마 생성 메타데이터 테이블

Job

job-instance

job-execution

job-execution-context

job-execution-params

step

step-execution

step-execution-context

# 배치 도메인

## Job

![Untitled](Spring%20Batch%20e08eb26a13f342e6a5d5b2f9eb5aa7c0/Untitled%203.png)

### 개념

- 배치 계층 구조에서 가장 상위에 있는 개념, 하나의 배치작업 자체를 의미한다.
- JobConfiguration을 통해서 생성되는 객체 단위로, 배치작업을 어떻게 구성하고 실행할 것인지 명세해놓은 객체
- 배치Job을 구성하기 위한 취상위 인터페이스, 스프링 배치가 기본 구현체를 제공한다.
- 여러 Step을 포함하고있는 컨테이너, 반드시 한개 이상의  Step으로 구성해야함.

### 기본 구현체

- SimpleJob
    - 순차적으로 Step을 실행시키는 Job
    - 모든Job에서 유용하게 사용할 수 있는 표준 기능을 가지고있다.
- FlowJob
    - 특정한 조건과 흐름에 따라 Step을 구성하여 실행하는 Job
    - Flow객체를 실행시켜서 작업을 진행한다.

## JobInstance

![Untitled](Spring%20Batch%20e08eb26a13f342e6a5d5b2f9eb5aa7c0/Untitled%204.png)

![Untitled](Spring%20Batch%20e08eb26a13f342e6a5d5b2f9eb5aa7c0/Untitled%205.png)

### 개념

- Job이 실행될 때 생성되는 논리적 실행단위.
- Job의 설정과 구성은 동일하지만 실행되는 시전에 처리하는 내용은 다르기때문에 실행을 구분한다.

## JobParameter

### 개념

- 하나의 Job에 존재할 수 있는 여러개의  Job instance를 구분하기 위한 용도

## JobExecution

![Untitled](Spring%20Batch%20e08eb26a13f342e6a5d5b2f9eb5aa7c0/Untitled%206.png)

### 개념

- JobInstance에서 한번의 시도를 의미하는 객체
- Job실행중에 발생한 정보들을 저장하고 있다.
- FAILED/COMPLETED/STARTING/STARTED/STOPPED/ABANDONED/UNKNOWN
- BatchStatus를 가지고있다.

## STEP

### 개념

- BatchJob을 구성하는 독립적인 하나의 단계, 실제 배치 처리를 정의하고 컨트롤하는데 필요한 모든 정보를 가지고있다.
- Job의 세부작업을 Task기반으로 설정하고 명세해놓은 객체

### 기본 구현체

- TaskletStep
    - 기본
- PartitionStep
    - 멀티스레드 방식으로 Step을 여러개로 분리해서 사용
- JobStep
    - Step내에서 Job을 실행
- FlowStep
    - Step내에서 Flow 실행
    

## StepExecution

![Untitled](Spring%20Batch%20e08eb26a13f342e6a5d5b2f9eb5aa7c0/Untitled%207.png)

### 개념

- Step에 대한 한번의 시도, Step실행중에 발생한 정보들을 저장하고있는 객체
- Step이 매번 시도될 때마다 생성된다.
- Job이 재시작해도 성공적으로 완료된  Step은 재실행하지 않는다.

## ExecutionContext

### 개념

- 프레임워크에서 관리하는 key-value 컬렉션
- StepExecution / JobExecution 객체의 상태를 저장하는 공유객체
- Job 재시작시 이미 처리한 Row데이터는 건너 뛰고 수행하도록 할 때 상태 정보를 활용한다.

## JobRepository

### 개념

- 배치작업 중의 정보를 저장하는 저장소 역할을 수행.
- Job이 언제 수행되었고, 언제 끝났고, 몇번 실행되었고 등 배치 작업과 관련된 모든 metadata 저장
- @EnableBatchProcessing 선언하면 자동으로 빈생성된다
- BatchConfigurer 인터페이스를 구현하거나 BasicBatchConfigurer 상속해서 createJobRepository() 오버라이딩을 통해 커스터마이징 가능
    
    ```java
    @Override
    protected JobRepository createJobRepository() throws Exception {
        JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
        factory.setDataSource(dataSource);
        factory.setTransactionManager(transactionManager);
        factory.setIsolationLevelForCreate("ISOLATION_SERIALIZABLE");
        factory.setTablePrefix(“SYSTEM_"); 
        factory.setMaxVarCharLength(1000); 
        return factory.getObject();
    }
    ```
    

## JobLancher

### 개념

- 배치 Job을 실행시키는 역할
- Job,JobParameter를 인자로 받아서 배치작업 수행 후 JobExecution 반환.
- JobLauncher.run(Job,JobParameter); — 실행
- 기본적으로 동기적으로 실행되며 taskExecutor를 SimpleAsyncTaskExecutor로 설정시 바로 JobExecution을 반환한 후 배치처리를 완료한다.

```java
@Configuration
@RequiredArgsConstructor
public class HelloJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job simpleJob() {
        return jobBuilderFactory.get("helloJob")// JobBuilder 를 생성하는 팩토리,  Job 의 이름을 매개변수로 받음
            .start(helloStep1())// 처음 실행 할 Step 설정,  최초 한번 설정, 이 메서드를 실행하면 SimpleJobBuilder 반환
            .next(jobStep())// 다음에 실행 할 Step 설정, 횟수는 제한이 없으며 모든 next() 의 Step 이 종료가 되면 Job 이 종료된다
            .next(helloStep3())
            .incrementer(parameters -> null)// JobParameter 의 값을 자동을 증가해 주는 JobParametersIncrementer 설정
            .validator(valid())// JobParameter 를 실행하기 전에 올바른 구성이 되었는지 검증하는 JobParametersValidator 설정
            .preventRestart()// Job 의 재 시작 가능 여부 설정, 기본값은 true
            .listener()// Job 라이프 사이클의 특정 시점에 콜백 제공받도록 JobExecutionListener 설정
            .build();// SimpleJob 생성
    }

    @Bean
    public Job flowJob() {
        return jobBuilderFactory.get("flowJob")// Flow 시작하는 Step 설정
            .start(helloStep1())// 처음 실행 할 Step 설정,  최초 한번 설정, 이 메서드를 실행하면 SimpleJobBuilder 반환
                .on("c*t")// Step의 실행 결과로 돌려받는 종료상태 (ExitStatus)를 캐치하여 매칭하는 패턴, TransitionBuilder 반환
                .to(helloStep3())
                .on("*")
                .stop()
            .from(helloStep1())
                .on("*")
                .to(helloStep3())
                .on("*")
                .end()
            .next(helloStep1())
            .end()
            .build();// SimpleJob 생성
    }

    @Bean
    public JobParametersValidator valid() {
        System.out.println("---------valid success---------");
        return parameters -> {

        };
    }

    @Bean
    public Step helloStep1() {
        return stepBuilderFactory.get("helloStep1")
            .tasklet((contribution, chunkContext) -> {
                System.out.println("-----------------------");
                System.out.println("hello Batch1");
                System.out.println("-----------------------");
                return RepeatStatus.FINISHED;
            })
            .startLimit(10)//step의 실행횟수 조정. 설정값을 초과하면 startLimitException이 발생한다.
            .allowStartIfComplete(true)//재시작 가능한 job에서 step의 성공여부와 상관없이 step을 실행
//            .listener()
            .build();
    }

    @Bean
    public Step jobStep() {
        return stepBuilderFactory.get("jobStep")
            .job(simpleJob())
            .launcher(new SimpleJobLauncher())
            .parametersExtractor(new DefaultJobParametersExtractor())
            .build();
    }

    @Bean
    public Step helloStep3() {

        return stepBuilderFactory.get("helloStep3")
            .tasklet((contribution, chunkContext) -> {
                System.out.println("-----------------------");
                System.out.println("hello Batch3");
                System.out.println("-----------------------");
                return RepeatStatus.FINISHED;
            })
            .build();
    }
}
```

# Chunk

## ItemReader

### 개념

- 다양한 입력으로부터 데이터를 읽어서 제공하는 인터페이스
- txt,csv,XML,Json,DB,JMS,Kafka,등등
- ChunkOrientedTasklet 설정 필요

## ItemProcessor

### 개념

- ItemReader로부터 읽어들인 데이터를 가공
-