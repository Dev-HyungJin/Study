#8장
## 스레드 프로그래밍

### 1. _Pthread_ (POSIX 쓰레드), _OpenMP_
POSIX 쓰레드는 약어로 _pthread_ 라고 불리며 _POSIX_계열의 운영체제 대부분이 지원한다.
> 윈도우 계열에도 포팅 되어있으나 윈도우는 비표준 윈도우 네이티브 쓰레드의 성능이 뛰어나기 때문에 범용성이 중요한 경우를 제외하고는 _pthread_ 를 잘 사용하지 않는다.

OpenMP는 쓰레드 프로그래밍의 난해한 기법과 이기종 포팅이 어렵다는 것을 해결하기 위해서 몇 가지 쓰레드 기법을 단순화했다.

> OpenMP는 리눅시에서 사용되는 gcc뿐 아니라 마이크로소프트의 비주얼 스튜디오에도 적용된다. 하지만 각 회사의 컴파일러나 구현하는 방식에 따라 추가 라이브러리나 옵션이 필요하므로 OpenMP를 사용한다면 컴파일러와 해당 운영체제의 지원 여부를 확인해야한다.

OpenMP는 아래 처럼 함수나 메서드의 원형을 직접 호출하지 않고 지시어(directive) 선언 방식을 사용한다는 점이 특징이다. 아래 코드의 `#pragma omp`로 시작하는 지시어 부분이 OpenMp 병렬 처리 코드이다.
```c++
int main()
{
    #pragma omp parallel
    {
        printf("Hello world.\n);
    }
    return 0;
}
```

### 2. 멀티 쓰레딩과 성능 향상

병렬 처리(parallel processing)란 동시에 복수의 작업을 처리하는 것이고, 동시 처리(concurrent processing)란 복수의 작업을 짧은 간격으로 왔다갔다 하면서 처리하는 방식을 의미한다.

> CPU코어가 1개면 병렬처리는 할 수 없지만 멀티 쓰레드를 사용하면 레이턴시 하이딩으로 성능 향상 효과를 볼 수 있는 경우가 있다.

#### 2.1 레이턴시 하이딩
**레이턴시**

>I/O를 요청하고 응답이 오기까지 대기 하는 시간 CPU와 입출력하는 I/O장치의 응답 시간 차이가 클 수록 레이턴시가 커질 수 있다.

이처럼 PCU수가 쓰레드 수 보다 작은 경우에도 입출력 작업이 있는 경우 입출력 응답대기 시간동안 CPU가 다른 쓰레드의 작업을 할수 있기 때문에 비동기적 작업을 수행할 수 있게되고 성능 향상을 가져온다.

이렇게 레이턴시가 다른 작업에 의해 숨겨지는 것처럼 보이기 때문에 레이턴시 하이딩이라고 부르며 네트워크 입출력히나 디스크 입출력 빈도가 높은 경우에는 멀티 쓰레딩의 효과가 꽤 높아진다.
하지만 CPU가 이미 포화 상태일 경우 레이턴시 동안 처리할 여분의 CPU파워가 없으므로 오히려 느려질 수 있다.

### 3. 멀티 쓰레드 VS 멀티 프로세스

과거 쓰레드가 쓰이지 않던 시절에는 fork를 이용한 프로세스 단위의 멀티 태스킹을 구현해왔다. 하지만 이 방법은 시스템 전체의 성능을 향상 시킬 수 있지만 프로세스 한개의 성능향상과는 연관이 없기 때문에 멀시 쓰레딩이 필요하게 되었다.

**멀티 프로세스**의 경우, 각각이 독립된 프로세스로 작동하기 때문에 메모리 영역이 보호되는 장점도 있고 동기적 프로그래밍 모델을 사용하므로 디버깅이 편리한 장점이 있다.

**멀티 쓰레드**의 경우, I/O 처리에 강점을 가진다.

> I/O 성능의 기준은 응답속도와 대역폭인데 CPU는 램이나 디스크보다 응답속도와 대역폭이 모두 높기 때문에 병목 현상이 발생할 수 밖에 없다. 

**멀티 프로세스** 방식에서는 프로세스간 통신을 하기 위해서는 IPC를 사용해야 하기 때문에 작업 단위를 쪼갤 수록 IPC에 의한 대역폭 사용량이 증가하게되고 결국 성능 한계에 부딪히는 시점이 오게 된다.

**멀티 쓰레드** 방식에서는 각 쓰레드가 프로세스 내부의 메모리 주소 공간을 공유하기 때문에 메모리 주소만 넘겨서 통신을 할 수 있게 되고 쓰레드간 통신 비용이 획기적으로 줄어든다.

### 4. 최적화와 멀티 쓰레딩의 문제점

멀티 쓰레딩은 **개발, 테스트, 디버깅이 매우 복잡해지기 때문에** 멀티 쓰레딩을 도입하지 않아도 요구 성능을 충족한다면 굳이 쓰지 않는 것이 좋다.

그럼에도 불구하고 멀티 쓰레드를 사용애햐 하는 경우라면 먼저 몇 가지를 체크해봐야 한다.
>- I/O 측면에서 병목이 제거된 상태인가?
>- 멀티 쓰레드 도입전의 코드는 다른 최적화 기법을 적용해 보았는가?
>- 멀티 쓰레드 도입전의 시스템에 가용할 수 있는 CPU나 관련 유휴 자원은 충분한가?
>- 효율을 극대화하려면 쓰레드의 숫자를 어떻게 결정해야 하는가?

### 5. 병렬 처리 패턴

병렬처리 프로그래밍을 하려면 무엇에 중점을 두고 병렬 처리할 것인지를 결정해야 한다.
- 작업의 단위나 순서의 흐름에 중점을 두면 **태스크 분해(task decomposition)** 형식으로 설계
- 데이터의 형태나 크기에 중점을 주면 **데이터 분해(data decomposition)** 형식으로 설계

#### 5.1 태스크 분해 방식
처리해야 하는 데이터나 행위가 매번 달라지거나, 연속적인 흐름을 띄는 경우에 적합하다.
- 콜백
- 트리구조
- 파면(wavefront) 분기
- 파이프라인(pipeline)

#### 5.2 데이터 분해 방식
동일한(혹은 비슷한) 작업을 하는 복수의 태스크를 생성하여 분업으로 효율을 높이는 구조를 말한다.
처리할 데이터는 다수의 독립적인 분절(chunk)로 쪼갤수 있어야한다. 적절한 로드 밸런싱이 되지 않으면 효율이 낮기 때문에 밸런스에 맞춰 청크를 할당하는 스케줄링의 해결이 중요하다.
- 압축
- 이미지 프로세싱
- 대규모 시뮬레이션
- 행렬 계산

### 6. 쓰레드 안전(Thread Safe)
쓰레드 안전이란 쓰레드에서 사용해도 안전한 코드를 총칭한다.
안전하다는 의미는 일차적으로는 프로세스의 치명적인 중단을 일으키지 않는 것을 의미하고 그 다음으로는 함수가 가진 의미, 즉 기능을 제대로 수행한다는 뜻이다.

#### 6.1 쓰레드 안전하지 않은 코드
스레드 안전하지 않은 코드는 예측할 수 없는 결과를 내놓을 가능성을 내포한 경우를 말한다.
아래 심각한 중단을 야기 하지는 않지만 기능적 문제가 있는 코드부터 살펴보자.

``` c++
// buf_sum이 정적(static)으로 선언되었기 때문에 복수의 쓰레드가 접근하면 결과를 알 수 없게 된다.
char* sum_strnum(const char* s1, const char* s2)
{
    static char buf_sum[16];
    sprintf(buf_sum, sizeof(buf_sum), "%d", atoi(s1)+atoi(s2));
    return buf_sum;
}
```

위와 같은 경우에는 뮤텍스 락을 적용해도 buf_sum에 신뢰성을 줄 수 없어 쓰레드 안전을 확보하기는 힘들기 때문에 재진입성 함수로 바꿔야만 한다.

#### 6.2 재진입성
재진입성(reentrancy)이란 병렬 실행을 보장하도록 작성된 형태를 취하여 쓰레드 안전을 획득한 경우를 말한다. 병렬 실행을 보장한다는 의미는 해당 코드를 실행하다가 다른 쓰레드가 동일한 함수를 호출하여도 각각의 쓰레드에서 호출한 작업은 서로 독립적이라는 뜻이다.

이런 구현을 가능하게 하려면 함수 내부에서는 어떤 의존 관계도 가지면 안되므로 정적(static)공간이나 공유 객체를 사용하지 않는 구조로 작성되어야만 한다.

#### 6.3 뮤텍스를 통한 쓰레드 안전
재진입 구조는 정적 객체를 사용하는 경우나 극히 제한적인 코드에서만 적용할 수 있다. 만일 전역 변수로 선언된 공간을 사용해야만 하는 경우라면 재진입 함수로는 만들 수 없고 뮤텍스를 통해 쓰레드 안전을 획득 해야만 한다.

#### 6.4 비동기 시그널 안전, 비동기 취소 안전
시그널은 언제 발생할지 모르는 대표적인 비동기 요소이다. 그래서 유닉스 계열에서는 함수가 시그널에 의해 인터럽트 되면 EINTR 에러로 리턴되도록 설계된 경우가 많다.

과거에는 EINTR을 제대로 처리하지 않아도 프로세스가 중단되거나 하는 경우는 흔치 않았기 때문에 이에 대한 처리가 미흡한 경우가 많았다. 하지만 쓰레드 환경에서는 크리티컬 섹션에 진입하는 과정에서 시그널 개입으로 실패해버리면 데드락에 빠질 가능성이 있기 때문에 주의를 요한다.

예를 들어 sem_wait 함수를 생각해보자. IEEE std 1003.b의 규격에 속한 함수들은 모두 쓰레드 안전을 만족하기 때문에 sem_wait도 쓰레드 안전한 함수이다. 하지만 sem_wait의 맨 페이지를 보면 EINTR 에러 코드가 존재하고 있다.
즉 시그널에 의해서 인터럽트 될 가능성이 있는 함수이다. 예를 들어 sem_wait로 블록하고 있다가 시그널 인터럽트가 발생하면 sem_wait는 실패하면서 EINTR로 빠져나온다는 것이다.

이와 반대로 sem_post는 **비동기 시그널 안전(async-signal_safe)** 로 시그널 처리기에 의해 인터럽트 되지 않는다. 비동기 시그널 안전 함수들은 시그널 처리기에 의해 인터럽트 되지 않지만 일부 함수들에 EINTR이 존재하는 경우가 있다. 이들은 본격적으로 처리가 시작되기 전에는 실패하지만 일단 EINTR이 시작되면 안전하게 끝마치도록 설계되어 있다.

비동기 시그널 안전과 비슷하게 **비동기 취소 안전(async-cancel-safe)** 함수라는 것도 존재한다. 이는 쓰레드를 취소시켰을 때도 안전하게 실행되는 경우를 의미한다.

예를 들어 pthread_cancel 함수는 특정 쓰레드를 취소시킬 수 있는데 비동기 취소 요청을 받으면 쓰레드가 갑작스럽게 중단될 수 있다. 그런데 이때 어떤 함수를 수행중이었다면 함수가 제대로 실행되지 않을 수 있다. 비동기 취소 안전 함수는 위와 같은 상황에서 작업을 안전하게 완료할 수 있다.

하지만 위 상황에서는 리턴받을 쓰레드가 이미 취소되어 제대로 종료되었는지 알기 힘들기 때문에 비동기 취소는 잘 쓰이지 않고 대부분 지연된 취소를 사용하는 것이 좋다.

#### 6.5 원자적 실행
컴퓨팅에서 원자적 실행(atomic operation)이란 더는 쪼갤 수 없는 단위로 실행되는 연산을 의미하며 시스템 프로그래밍에서는 멀티 쓰레드가 실행되는 환경에서 다른 쓰레드가 끼어들 수 없는 실행 단위를 의미한다.

#### 6.6 파일 입출력
멀티 쓰레딩을 도입하면 가장 골치 아픈 것이 I/O처리 부분이다. 만일 복수의 쓰레드가 하나의 파일 기술자에 읽거나 쓰기를 시도하면 어떻게 될까? read, write 함수를 쓰레드 안전 함수이므로 프로그램은 죽지 않겠지만 실행 순서에 따라 원하지 않는 결과가 나올 수도 있다.

따라서 파일의 블록 구간을 나눠 1구간은 첫번째 쓰레드, 2구간은 두번째 쓰레드, 3구간은 세번째 쓰레드가 기록하도록 나눠준다면 문제가 발생하지 않을 것이다.
이때 pthread를 사용해야하고 AIO를 사용하면 쓰레드 안전은 물론이고 더 뛰어난 성능으로 설계할 수 있다.


### 7. POSIX 쓰레드 (pthread)

#### 7.1 pthread의 특징

일반적인 시스템 호출 함수들은 정수형을 리턴하는 구조일 때는 -1이 실패, 포인터 형을 리턴하는 구조일 때는 NULL이 실패로 정의되어 있다. 그러나 pthread는 성공하면 0을 리턴하고 실패하면 errno에 해당하는 값을 직접 리턴하도록 되어있다. 이는 pthread 함수가 에러를 발생했는데 다른 함수들을 처리하다가 에러 번호를 덮어쓰지 않도록 하기 위해 만들어진 것이다.

그런데 간혹 pthread 함수 중에 포인터형을 리턴하는 경우도 있다. 이 경우에는 실패 자체가 없는 함수이므로 실패했을 때 에러 코드를 어떻게 읽어야 할지 신경 쓸 필요가 없다.

또, pthread 함수들은 시그널에 의해 인터럽트되지도 않기 때문에 시그널에 의한 실패를 고민하지 않아도 된다. 하지만 시그널 함수에서는 사용하면 안되며 결론적으로 pthread 함수들은 시그널 인터럽트에 대해 자유롭지만 시그널 핸들러에서 사용할 수 있는 비동기 시그널 함수는 아니다.

#### 7.2 pthread의 생성 및 종료, 취소

|||
|-|-|
|pthread_create|쓰레드를 생성한다.|
|pthread_exit|쓰레드를 종료한다.|
|pthread_join|쓰레드를 프로세스에 병합한다.|
|pthread_detach|쓰레드를 프로세스에서 분리한다.|
|pthread_cancel|쓰레드를 취소한다.|

```c++
int pthread_create(
    pthread_t *restrict thread, 
    const pthread_attr *restrict attr, 
    void *(*start_routine)(void*), 
    void *restrict arg);

void pthread_exit(void *value_ptr);

```

`pthread_create`가 실행되려면 최소한 2개의 인수를 필요로한다. 바로 첫 번째 인수인 thread와 3번째 인수인 `start_routine`이 해당된다.

**첫 번째 인수인 `thread`** 에는 생성된 쓰레드의 ID를 리턴받을 공간으로서 쓰레드 고유의 ID를 의미한다. 이를 TID라고 부르며 프로세스내에서만 고유성을 유지한다.

**세 번째 인수인 `start_routine`** 인수는 쓰레드가 구동할 함수로서 리턴 타입은 `void *`, 인수는 `void *` 형 한개를 받는 형태로 선언되어 있어야한다.

> 쓰레드 ID(`pthread_t`)는 정수 타입일까?
> gcc에서 `pthread_t`는 정수형 이지만 운영체제에 따라 구조체를 사용하는 경우도 있다. 따라서 쓰레드 ID를 비교할 때는 `pthread_equal`함수를 사용해 비교하도록 하자.

`pthread_exit`는 쓰레드를 리턴하는 함수로서 현재 쓰레드를 종료 시킨다. 여기서 `value_ptr`은 쓰레드 함수가 리턴할 값으로서 이 값은 다른 쓰레드에서 `pthread_join` 함수를 호출하면 읽을 수 있는 값이 된다. 쓰레드 스택내에 존재하는 주소는 쓰레드와 함께 파괴되기 때문에 `value_ptr`로 넘기면 안된다.

```c++
int pthread_join(pthread_t thread, void **value_ptr);
int pthread_detach(pthread_t thread);
int pthread_self(void);
```

`pthrad_join`은 분리되지 않은 쓰레드가 종료하기를 기다리는 함수로서 쓰레드를 병합한다고 표현한다. 첫 번째 인수에는 기다릴 쓰레드ID가 지정되고 두 번째 인수에는 쓰레드가 종료하면서 리턴할 값을 받을 수 있다.

`pthread_detach`를 실행한 쓰레드는 분리되는데 이는 병합될 필요가 없는 쓰레드를 의미한다. 분리된 쓰레드는 병합을 기다리지 않고 리턴과 동시에 해제되지만 분리되었다고 해서 프로세서를 벗어나는 것은 아니며 리턴 값을 전해줄 수 있는 방법만 사라진 것으로 보면 된다.

`pthread_cancel`은 지정한 TID의 쓰레드를 취소, 즉 중지 시킨다는 뜻이다. 하지만 요청 후 바로 중지하는 것이 아니라 pthread_testcancel을 호출한 곳이나 입출력 관련 시스템 호출 지점에서 중지되며 이를 지연된(deferred) 취소라고 한다.

#### 7.4 pthread 뮤텍스
뮤텍스(mutex)는 Mutual Exlusion의 약자로서 락(lock) 매커니즘의 하나이다.

|||
|-|-|
|pthread_mutex_init|뮤텍스 변수를 초기화한다.|
|pthread_mutex_lock|뮤텍스 잠금을 시도한다. 잠금할 수 있을 때까지 블록된다.|
|pthread_mutex_trylock|뮤텍스 잠금을 시도한다. 잠금할 수 없다면 에러로 리턴한다.(넌블록킹)|
|pthread_mutex_timedlock|뮤텍스 잠금을 시도한다. 지정된 타임아웃 시간만 블록된다.|
|pthread_mutex_unlock|뮤텍스 잠금을 해제한다|
|pthread_mutex_destroy|뮤텍스를 파괴한다.|
|PTHREAD_MUTEX_INITIALIZER|pthread_mutex_t 뮤텍스 변수 선언 초기화 매크로|

##### 7.4.1 pthread 뮤텍스의 초기화
`PTHREAD_MUTEX_INITIALIZER` 매크로와 `pthread_mutex_init` 함수를 이용해 초기화할 수 있다. 둘 중에 아무 방법이나 사용해도 되지만 `PTHREAD_MUTEX_INITIALIZER` 매크로는 변수를 선언하면서 직접 초기화 할 때만 사용되는 극히 제한적인 방법이다.

``` c++
int pthread_mutex_init(
    pthread_mutex_t *restrict mutex,
    const pthread_mutexattr_t *restrict attr);
int pthread_mutex_destroy(pthread_mutex_t* mutex);
```
`pthread_mutex_initd`은 두개의 인수를 받는다.
- mutex : 초기화 할 뮤텍스
- attr : 초기화할 뮤텍스의 속성

> 뮤텍스 타입
>- **NORMAL** : 일반적으로 사용되는 뮤텍스로서 가볍다는 장점이 있지만 데드락이나 다른 모든 에러 상황에 대해 감지하지 않으므로 불완전하거나 실뢰할 수 없는 논리적 모순이 있을 때는 데드락에 빠질 수 있다.
>- **ERRORCHECK** : 뮤텍스는 에러를 체크하는 기능이 탑제된 뮤텍스로서 데드락이나 다른 에러 상황에 대해 블록되지 않고 에러로 리턴해준다. 에러를 감지하기 위한 오버헤드가 존재한다.
>- **RECURSIVE** : 재귀적 잠금이 가능한 뮤텍스이다. 소유권을 가진 뮤텍스가 다시 잠금을 해야하는 상황일 때 사용한다. (잠금 횟수만큼 해제가 필요하다)

##### 7.4.2 베리어(Barrier)
뮤텍스를 이용해 쓰레드를 지정된 묶음 단위로 처리하는 방법

##### 7.4.3 스핀락(SpinLock)
아주 짧은 시간동안의 락으로 처리가 완료될 수 있는 경우 컨텍스트 스위칭과 스케줄링에 의해 불필요하게 많은 오버헤드가 발생 할 수 있다.
이를 피하기 위해 락이 해제되기 전까지 CPU에 리소스를 반납하지 않고 유지하도록 하는 기능을 의미한다.
하지만 획득한 쓰레드를 오랜시간 동안 반납하지 않으면 매우 비효율적일 수 있으므로 파일, 네트워크 I/O작업에서는 사용하지 않는 것이 좋으며
CPU가 1개일 경우에는 다른 쓰레드에서 락이 해제될 수 없기 때문에 스핀락을 사용할 수 없다.

##### 7.4.4 판독자-기록자 락(Reader-Writer Locks, rwlocks)
읽기 작업만 하는 경우에는 데이터 오염이 생기지 않는 다는 점에 착안해 만들어진 잠금 형태로 공유 자원의 읽기 작업이 빈번한 경우 좋은 성능을 낼 수 있다.

##### 7.4.5 쓰레드 로컬 저장소 (TLS, Thread Local Storage)
전역 변수를 쓰레드별로 다른 장소로 대체하는 기능으로 과거에 전역 변수나 정적 공간을 사용하던 함수들을 쓰레드 안전한 코드로 변경할 때 도움을 준다.
TLS의 구현 방식은 플랫폼이나 언어별로 다르기 때문에 상황에 따라 다른 TLS를 선택해 사용할 수 있으나
프로그램의 호환성이 중요한 경우에는 pthread의 TLS구현을 따르는 것이 좋다.

##### 7.4.6 Robust 뮤텍스
쓰레드가 뮤텍스를 잠근 채 종료했을 때 데드락 상황을 방지하기 위한 기능이다.
이느 ㄴ논리적인 프로그래밍 오류로 충분히 예방할 수 있지만 좀 더 직관적이고 가벼운 뮤텍스 기능의 필요성에 따라 도입되었다.

#### 7.5 이벤트 핸들러
쓰레드가 갑자기 종료되거나 Fork를 사용하는 등 예외적인 이벤트 처리를 위해 쓰레드 종료시 실행되는 클린업 기능과 Fork시 실행되어야 할 함수를 지정하는 기능이 제공된다.
##### 7.5.1 쓰레드 클린업 핸들러
쓰레드가 종료될 때 실행해야하는 핸들러 함수를 등록하는 기능이다. 프로세스 종료시 호출되는 atexit 함수의 쓰레드 버전이라고 생각하면 편리할 것이다.
클린업 기능은 신뢰성 있는 쓰레드 프로그래밍에 꼭 필요한 기능이므로 숙지해 두는 것이 좋다.
```c++
void pthread_cleanup_push(void(*routine)(void*), void *arg);
void pthread_cleanup_pop(int excute);
```
`pthread_cleanup_push`는 핸들러를 등록하는 것으로서 rountine은 클린업 함수, arg는 routine함수가 사용할 인수 주소가 된다.
`pthread_cleanup_pop`은 등록했던 핸들러를 제거하는 함수로서 스택 구조처럼 가장 최근에 등록된 순으로 제거된다. 제거될 때 execute 인수가 0이 아닌 값을 넣어주면 등록된 함수를 실행한 뒤에 제거한다.
클린업 핸들러는 항상 push와 pop의 짝이 맞아야 하고 핸들러를 등록, 해제하는 오버헤드가 있기 때문에 스핀 락 처럼 성능을 중시하는 경우에는 되도록 쓰지 않는 방향으로 설계해야한다.

##### 7.5.2 쓰레드 fork 핸들러
쓰레드가 fork를 호출할 때 실행할 핸들러 함수를 등록하는 기능이다. 하지만 멀티 쓰레드 환경에서는 fork를 지양하는 것이 신뢰성이 높기 때문에 이 기능은 되도록 사용하지 않는 편이 좋다.

### 6. OpenMP 프로그래밍
>OpenMP는 간단한 지시어를 사용하여 멀티 쓰레딩을 적용할 수 있는 기법으로서 이식성이 높고 빠르게 습득할 수 있는 장점이 있다.

물론 쓰레드 사이의 긴밀한 협조와 제어가 필요한 네트워크 프로그래밍이나 각종 데몬 프로세스에 적용하려면 OpenMP는 제약이 많아서 적용하기 어려울 수 있지만 수치 계산이나 알고리즘 테스트 같은 분야에서 각광받고 있다.

```c++
//#pragma omp parallel [clause[[,]clause]...] new-line
//    structured-block

int main(){
#pragma omp parallel
    {
        // 이 블럭은 병렬처리된다.
        printf("Hello OpenMP\n");
    }
}
```

각 컴파일러에 맞는 명령어 인자를 이용해 위 코드를 컴파일 하면 시스템의 CPU수에 맞게 쓰레드가 생성되어 병렬처리 블럭이 실행됨을 볼 수 있다.
>CPU가 4개인 시스템에서는 *Hello OpenMP*가 4번 출력된다.

실행될 쓰레드 수를 유저가 지정할 수도 있는데 아래 세가지 방법을 통해 지정할 수 있다.

```c++
// 1. omp_set_num_threads()
#include<omp.h>
int main(){
omp_set_num_threads(2);
...
}

// 2. num_threads()
#pragma omp parallel num_therads(2)

// 3. Eviroment Option
// in Bash Shell
# export OMP_NUM_THREADS=2

```

### 7. 성능을 고려한 프로그래밍
실제 프로젝트에 쓰레드를 도입하면 여러가지 문제가 발생 할 수 있다. 그중 대표적인 문제가 `가짜공유(false sharing)`으로 캐시라인의 크기를 고려하지 않고 프로그래밍했을 때 발생한다.

예를 들어 아래 프로그램의 경우 각 스레드별로 다른 배열을 참조하는 것 처럼 보이고 성능상에도 큰 문제가 없어보인다.
```c++
#define NUM_THREADS 4
int cnt_sheep[NUM_THREADS];

int main(){
#pragma omp parallel for
    for(int i=0; i<NUM_THREADS; i++){
        countSheep(i);
    }
}

void countSheep(int idx){
    // something with cnt_sheep[idx]
}
```
하지만 실제로 CPU가 읽어보는 캐시라인 크키가 32바이트가 넘기 때문에 cnt_sheep 배열 전체가 캐시라인에 올라오게 되고
각 쓰레드에서 캐시에 올라온 값을 변경 할 때마다 캐시 미스가 발생해 성능에 악영향을 끼치게된다.

컴파일러의 최적화 옵션을 사용하거나 소스코드에서 캐시라인 단위로 정렬해 코드를 작성하는 방법을 사용하면 가짜 공유로 인한 문제를 해결할 수 있다.

