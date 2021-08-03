스프링 배치(Spring Batch)
===========

배치 프로세싱은 **일괄처리** 라는 뜻을 가지고 있다. 일괄처리의 의미는 **일련의 작업을 정해진 로직** 으로 수행하는 것이라 할 수 있다.  
일괄처리가 필요한 경우는 아래와 같다  
1. 대용량의 비즈니스 데이터를 복잡한 작업으로 처리해야 하는 경우
2. 특정한 시점에 스케쥴러를 통해 자동화된 작업이 필요한 경우 (ex: push 알림, 월 별 리포트)
3. 대용량 데이터의 포맷을 변경, 유효성 검사 등의 작업을 트랜잭션 안에서 처리 후 기록해야 하는 경우
위와 같은 경우에 배치 어플리케이션을 작성하여 처리하게 된다. 실제로 엔터프라이즈 환경에서는 정말 다양한 종류의 작업들을 **배치** 를 이용하여 처리하고 있다
위와 같은 요구사항을 처리해줄 수 있는 배치와 관련된 어플리케이션 제작의 편의를 위해서 **스프링 배치** 프레임워크를 만들어 표준화하게 되었다

### 배치의 일반적인 사용 시나리오

![spring-batch-senario](https://user-images.githubusercontent.com/47099798/127942091-8642cc9f-4450-4eed-8795-14f24d1b032b.jpeg)

* 데이터베이스, 파일 또는 큐에서 데이터 읽기
* 데이터를 정의한 방식으로 처리
* 처리된 데이터를 데이터 쓰기
스프링 배치는 위와 같은 방식으로 사용자와의 상호작용없이 반복적으로 데이터를 트랜잭션 단위로 처리할 수 있도록 구현되어 있고, 개발자는 **데이터 처리** 에 대한 비즈니스 로직에만 집중하여 **배치 프로세스** 를 작성할 수 있다

### 스프링 배치가 제공할 수 있는 비즈니스 시나리오
1. 주기적인 배치 프로세스
2. 동시적인 배치 프로세스: 작업의 병렬 처리
3. 단계별 엔터프라이즈 메시지 기반 처리
4. 대규모 작업에 대한 병렬 배치 프로세스
5. 실패 후 수동 또는 예약된 재시작
6. 단계별 순차 처리
7. 부분처리: 레코드 건너 뛰기 (ex: rollback 시)
8. 배치 작업 처리의 단위가 작은 경우, 기존 저장 프로시저/스크립트가 있는 경우 전체 배치에 대한 트랜잭션 처리

### 스프링 배치 계층 구조

![spring-batch-structure](https://user-images.githubusercontent.com/47099798/127942118-543c9109-2c7b-47d6-aa3c-a6365eb51118.jpeg)

스프링 배치 프레임워크는 확장성과 최종 사용자를 염두해두고 설계되었기 때문에 위와 같이 Application, Batch Core 그리고 Batch Infrastructure 로 설계되어 있다
* Application: 개발자가 작성한 모든 배치 작업과 사용자 정의 코드 포함
* Batch Core: 배치 작업을 시작하고 제어하는데 필요한 핵심 런타임 클래스 포함 (JobLauncher, Job, Step)
* Batch Infrastructure: 개발자와 어플리케이션에서 사용하는 일반적인 Reader 와 Writer 그리고 RetryTemplate 와 같은 서비스를 포함
스프링 배치는 계층 구조과 위와 같이 설계되어 있기 때문에 개발자는 **Application** 계층의 비즈니스 로직에 집중할 수 있고, 배치의 동작과 관련된 것은 **Batch Core** 에 있는 클래스들을 이용하여 제어할 수 있다

### 배치 원칙 및 가이드
* 일반적으로 같은 서비스 환경에서 동작하는 서비스와 배치는 서로에게 영향을 미칠 수 있기 때문에 **배치와 서비스에 영향을 최소화** 할 수 있도록 구조와 환경에 맞게 디자인 해야 한다
* 배치 어플리케이션 내에서 가능화한 복잡한 로직은 피하고 **단순** 하게 설계해야 한다
* 데이터 처리하는 곳과 데이터의 저장소는 물리적으로 가능한 가까운 곳에 위치하도록 한다
* 데이터베이스 I/O, 네트워크 I/O, 파일 I/O 등의 **시스템 리소스의 사용을 최소화** 하고 최대한 많은 데이터를 메모리 위해서 처리하도록 한다
* 데이터 무결성을 위해 적절한 **검사 및 기록** 하는 코드를 추가한다

### 스프링 배치 예제
예제 시작 전 알아야 하는 내용
* 스프링 배치 메타 테이블
* Job, Step, Tasklet

#### 스프링 배치 메타 테이블

![spring-batch-meta-tables](https://user-images.githubusercontent.com/47099798/127942132-b2319fb9-4bb8-4002-a42b-720cd82e3c31.jpeg)

개발자가 작성한 작업이 아주 잘 작성되었다 하더라고, 작성된 일괄 작업들이 어떤 주기에 따라 지속적으로 동작한다면 성공 확률이 100%가 절대로 될 수 없을 것이다  
따라서 스프링 배치에서는 작업을 수행하면서 **일련의 상태** 에 관한 메타 데이터들을 **메타 테이블** 에 저장해서 실패한 작업에 대한 기록을 남겨 실패에 대한 대비를 준비할 수 있게 도와주는 역할을 하게 된다

#### Job, Step, Tasklet
* Job: 배치 처리 과정을 하나의 단위로 만들어 표현한 객체이고 여러 Step 인스턴스를 포함하는 컨테이너 
* Step: Step 은 실질적인 배치 처리를 정의하고 제어하는데 필요한 모든 정보가 있는 도메인 객체
* Tasklet: Step 안에서 수행될 비즈니스 로직 전략의 인터페이스 

```
@EnableBatchProcessing //배치 기능 활성화
@SpringBootApplication
public class BatchApplication {
    public static void main(String[] args) {
        SpringApplication.run(BatchApplication.class, args);
    }
}
```

위 코드와 같이 스프링 배치의 기능을 활성화하기 위해서 설정과 관련된 어노테이션 **@EnableBatchProcessing** 을 사용한다  

##### Tasklet 설정

```
@Slf4j
public class TutorialTasklet implements Tasklet {

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        log.debug("executed tasklet !!");
        return RepeatStatus.FINISHED;
    }
}
```


##### Job 설정

```
@Configuration
@RequiredArgsConstructor
public class TutorialConfig {
    private final JobBuilderFactory jobBuilderFactory; //Job 빌더 생성용
    private final StepBuilderFactory stepBuilderFactory; //Step 빌더 생성용

    //JobBuilderFactory를 통해서 tutorialJob을 생성
    @Bean
    public Job tutorialJob() {
        return jobBuilderFactory.get("tutorialJob")
                .start(tutorialStep()) //Step 설정
                .build();
    }

    // StepBuilderFactory를 통해서 tutorialStep을 생성
    @Bean
    public Step tutorialStep() {
        return stepBuilderFactory.get("tutorialStep")
                .tasklet(new TutorialTasklet()) //Tasklet 설정
                .build();
    }
}
```
위 코드와 같이 상단에서 정의한 TutorialTasklet 으로 Step 을 만들고, 만들어진 Step 을 Job 에 등록해주었다

##### Quartz 스케쥴러 적용하기

```
@EnableScheduling //스케쥴러 기능 활성화
@EnableBatchProcessing //배치 기능 활성화
@SpringBootApplication
public class BatchApplication {
    public static void main(String[] args) {
        SpringApplication.run(BatchApplication.class, args);
    }
}
```

##### 스케쥴러 클래스 생성

```
@Component
@RequiredArgsConstructor
public class TutorialScheduler {
    private final Job job; // tutorialJob
    private final JobLauncher jobLauncher;

    //5초마다 실행
    @Scheduled(fixedDelay = 5 * 1000L)
    public void executeJob () {
        try {
            jobLauncher.run(
                    job,
                    new JobParametersBuilder()
                            .addString("datetime", LocalDateTime.now().toString())
                    .toJobParameters() //job parameter 설정
            );
        } catch (JobExecutionException ex) {
            System.out.println(ex.getMessage());
            ex.printStackTrace();
        }
    }
}
```

**@Scheduled** 어노테이션을 이용하여 일정한 주기마다 작성한 Job 이 실행되도록 설정




  
