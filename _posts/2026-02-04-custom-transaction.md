---
title: Transactional 한계 극복해보기
author: dnya0
date: 2026-02-04 22:16:00 +0900
categories: [Study, Spring]
tag: [Post, Study, Kotlin, Spring, Transactional, AOP]
math: true
mermaid: true
image:
  path: https://user-images.githubusercontent.com/84761609/220085479-19ee260a-d709-4a47-b788-416275d8a2d8.png
  alt: Spring Image
---

## 🥑 들어가며

이전 프로젝트를 진행하며 기술 블로그를 참고하여 트랜잭션 로직을 직접 구현한 경험이 있었다. 당시에는 AOP에 대한 이해가 부족해, 왜 그렇게 동작하는지 정확히 알지 못한 채 코드를 작성했었다. 하지만 오늘 [카카오페이 테크 블로그](https://tech.kakaopay.com/post/overcome-spring-aop-with-kotlin/)의 글과 그때 작성했던 코드를 다시 살펴보며, 그 이유를 명확히 이해할 수 있게 되었다. 이를 잊지 않기 위해 정리하게 되었다.

<br>

## 🚨 `@Transactional` 한계

[이전에 작성했던 글](https://dnya0.github.io/posts/Spring-AOP-Transactional/)을 보면 `@Transactional`의 한계를 비교적 명확하게 확인할 수 있다. 간단히 말해, AOP는 프록시 패턴을 기반으로 동작하며 `@Transactional`은 AOP의 대표적인 활용 사례라는 점이다. 이로 인해 트랜잭션은 객체 외부에서 최초로 진입하는 메서드를 기준으로 적용되며, 동일 객체 내부의 메서드 호출(self-invocation)에는 트랜잭션이 적용되지 않는다.

그렇다면 이러한 불편함은 어떻게 해결할 수 있을까?

해답은 아이러니하게도, 당시 원리를 제대로 이해하지 못한 채 작성했던 이전 프로젝트 코드에 있었다. 그동안은 트랜잭션 적용을 위해 `TransactionalService`와 같은 별도의 클래스를 만들어 사용해왔는데, 이 방식을 활용하면 트랜잭션을 위해 굳이 클래스를 분리하지 않아도 동일한 문제를 해결할 수 있다.

<br>

## 💡 `@Transactional` 극복해보기

### 구현

먼저, 나는 코드의 가독성을 위해 따로 `@ReadOnlyTransactional`을 만들었다. 나는 의도적으로 REQUIRED를 사용했다. 즉, 읽기에도 **항상 트랜잭션을 시작**하게 한다.

```kotlin
@Transactional(readOnly = true, propagation = Propagation.REQUIRED)
annotation class ReadOnlyTransactional()
```

Transaction을 걸어주기 위한 클래스 하나를 생성한다. `@Transactional`을 붙여 해당 로직에 AOP를 적용시킨다. 이 클래스가 트랜잭션 경계를 보장할 진입점 클래스가 되는 것이다.

```kotlin
@Component
class TransactionRunner {

    @Transactional
    fun <T> run(block: () -> T): T = block()

    @ReadOnlyTransactional
    fun <T> runReadOnly(block: () -> T): T = block()
}
```

하지만, 이 코드만 있다면 트랜잭션이 필요한 메서드마다 bean을 주입해야하는 불편함이 있다. 그래서 어디서나 간단히 쓸 수 있게 전역 퍼사드 `Tx`를 둔다. 초기화 가드를 넣어(1회 초기화 보장) 실수로 인한 오류를 줄인다.

```kotlin
object Tx {

    @Suppress("ktlint:standard:backing-property-naming")
    private lateinit var _txRunner: TransactionRunner

    fun initialize(txRunner: TransactionRunner) {
        _txRunner = txRunner
    }

    fun <T> writable(function: () -> T): T = _txRunner.run(function)

    fun <T> readable(function: () -> T): T = _txRunner.runReadOnly(function)

}
```

또한 부팅 시 1회 초기화시키도록 Configuration을 작성한다.

```kotlin
@Configuration
class TransactionConfig {

    @Bean("txInitBean")
    fun txInitialize(txRunner: TransactionRunner): InitializingBean =
        InitializingBean { Tx.initialize(txRunner) }
}
```

이제 내부 메서드를 호출하더라도, `Tx -> TransactionRunner(프록시) -> 실제 메서드 순`으로 호출되어 AOP가 적용된다.

그렇다면 언제 이 패턴이 유리할까? self-invocation 이슈가 잦은 팀/코드베이스나 코드 블록 단위로 경계를 명시하고 싶은 팀

<br>

### 전파와 합류 규칙 요약(REQUIRED 기준)

- `Tx.readable`은 `REQUIRED`이므로 바깥 트랜잭션이 없어도 **읽기 전용 트랜잭션**을 시작한다.
- `writable` 내부에서 `readable`을 호출하면, `readable`은 기존 트랜잭션에 합류하고 `readOnly=false(=writable)`의 성질을 유지한다. 즉 **읽기 호출**이더라도 바깥이 쓰기라면 쓰기 트랜잭션으로 동작한다.
  읽기를 **있을 때만 합류, 없으면 비트랜잭션**으로 두고 싶다면 `Propagation.SUPPORTS`로 바꾼다. 이 글의 테스트는 REQUIRED 기준이다.

<br>

### 테스트

트랜잭션 활성/읽기전용 여부, 합류, 커밋/롤백을 검증한다.

```kotlin

@ExtendWith(SpringExtension::class)
@ContextConfiguration(classes = [TxIntegrationTest.TxTestConfig::class])
class TxIntegrationTest {

    @Test
    fun writable_starts_transaction_and_not_readOnly() {
        var active = false
        var readOnly = true

        val result = Tx.writable {
            active = TransactionSynchronizationManager.isActualTransactionActive()
            readOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly()
            42
        }

        assertEquals(42, result)
        assertTrue(active, "writable should start a transaction")
        assertFalse(readOnly, "writable transaction should not be read-only")
    }

    @Test
    fun readable_with_REQUIRED_starts_transaction_and_is_readOnly() {
        var active = false
        var readOnly = false
        Tx.readable {
            active = TransactionSynchronizationManager.isActualTransactionActive()
            readOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly()
        }
        assertTrue(active, "readable with REQUIRED should start a transaction when none exists")
        assertTrue(readOnly, "readable transaction should be read-only")
    }

    @Test
    fun readable_inside_writable_joins_existing_tx_and_is_not_readOnly() {
        var innerActive = false
        var innerReadOnly = true

        Tx.writable {
            Tx.readable {
                innerActive = TransactionSynchronizationManager.isActualTransactionActive()
                innerReadOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly()
            }
        }

        assertTrue(innerActive, "readable should join existing transaction (SUPPORTS)")
        assertFalse(innerReadOnly, "joined transaction should keep writable semantics (not read-only)")
    }

    @Test
    fun writable_commits_on_success_and_rolls_back_on_runtime_exception() {
        var committedStatus: Int? = null
        var rolledBackStatus: Int? = null

        // commit path
        Tx.writable {
            TransactionSynchronizationManager.registerSynchronization(object : TransactionSynchronization {
                override fun afterCompletion(status: Int) {
                    committedStatus = status
                }
            })
        }
        assertEquals(TransactionSynchronization.STATUS_COMMITTED, committedStatus)

        // rollback path
        assertThrows(RuntimeException::class.java) {
            Tx.writable {
                TransactionSynchronizationManager.registerSynchronization(object : TransactionSynchronization {
                    override fun afterCompletion(status: Int) {
                        rolledBackStatus = status
                    }
                })
                throw RuntimeException("boom")
            }
        }
        assertEquals(TransactionSynchronization.STATUS_ROLLED_BACK, rolledBackStatus)
    }

    @TestConfiguration
    @EnableTransactionManagement
    class TxTestConfig {
        @Bean
        fun dataSource(): DataSource = DriverManagerDataSource().apply {
            setDriverClassName("org.h2.Driver")
            url = "jdbc:h2:mem:widdle-tx;DB_CLOSE_DELAY=-1;MODE=PostgreSQL"
            username = "sa"
            password = ""
        }

        @Bean
        fun transactionManager(ds: DataSource): PlatformTransactionManager = DataSourceTransactionManager(ds)

        @Bean
        fun transactionRunner(): TransactionRunner = TransactionRunner()

        // Initialize Tx with the TransactionRunner bean
        @Bean
        fun txInitializer(runner: TransactionRunner): InitializingBean = InitializingBean {
            Tx.initialize(runner)
        }
    }
}
```
