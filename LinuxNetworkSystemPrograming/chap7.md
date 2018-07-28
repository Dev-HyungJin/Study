# 7장
## 1.I/O 멀티플렉싱
넌블록킹 I/O에 데이터가 수신된 경우에만 recv를 하게해 성능을 향상시키는 기법

> - _select_ 는 가장 큰 번호의 소켓 파일 기술자가 크면 오버헤드가 발생한다.
> - _poll_ 은 더욱 정교한 핸들링이 가능하며 중금 규모 (약 500~1000개 이하)에서 주로 쓰인다.
> - _epoll, /dev/poll, kqueue는 select, poll_ 보다 더 좋은 성능을 가지고 있으나 표준안에 명시된 기술이 아니므로 이식성이 떨어진다.
> - _select와 poll_ 은 레벨 트리거만 지원하며 _epoll_ 은 레벨/엣지 트리거를 지원한다.

### 1.1고성능 I/O 멀티플렉서의 탄생
90년대 중반 이후 대용량 네트워크 서비스가 필요해져 _select, poll_ 보다 좋은 성능의 함수들을 개발해야 했다. 그 결과 _O(1)_ 에 가까운 응답속도를 보여주는 고성능 I/O멀티 플렉서인  _epoll, kqueue, /dev/poll, IOCP_ 등이 도입되었다.
> - 리눅스 : _epoll_
> - BSD : _kqueue_
> - 솔라리스 : _/dev/poll_
> - 윈도우즈 : _IOCP_

## 2. _SELECT, PSELECT_
_select_ 는 I/O 멀티플렉싱을 사용하는 구조와 개념을 배우기에는 괜찮지만 함수 원형 자체가 직관적이지 못하고 파일 기술자 번호를 프로그래머가 따로 관리해야하는 등 불편한 점들이 많아 점점 사라지는 추세이다.

> **레벨트리거**
> 버퍼에 지정된 바이트 이상의 데이터가 쌓였는지 감시하는 기능으로 보통 1바이트를 기준으로한다.
> _select, pselect, epoll_ 모두 레벨 트리거를 사용한다.

### 2.1 차이점
**타임아웃**
> - _select_ : `struct timecal` 구조체를 사용해 마이크로초 단위까지 지정할 수 있다.
> - _pselect_ : `struct timespec` 구조체를 사용해 나노초 단위까지 지정 할 수 있다.

**시그널**
> - _select_ : 블록킹 중 시그널이 발생하면 에러로 리턴 후 빠져나가기 때문에 전역적인 시그널 블록 매스크를 프로그래밍 해야함. `sigprocmask` 나 `Pthread_sigmask` 함수로 추가적인 코딩 필요
> - _pselect_ : 함수 내부에서 시그널 블록 마스크를 대체해 좀 더 신뢰성 있게 동작하도록함

### 2.2 함수원형

```c++
int select (
	int nfds, 
	fd_set *readfds, 
	fd_set *writefds, 
	fd_set *exceptfds, 
	struct timeval *timeout);

int pselect(
	int nfds,
	fd_set *readfds,
	fd_set *writefds,
	fd_set *exceptfds,
	const struct timespec *timeout,
	const sigset_t *sigmask);

void FD_CLR(int fd, fd_set *fdset);
int FD_ISSET(int fd, fd_set *fdset);
void FD_SET(int fd, fd_set *fdset);
void DS_ZERO(fd_set *fdset);
```
**리턴값**
> 성공시 **이벤트가 발생한 파일 기술자 수**, 실패시 **-1**, 타임아웃 시간이 지나면 **0** 이 리턴된다.

**인자**

> **첫 번째 인자** 는 nfds는 입력받는 fd_set에 등록된 파일 기술자 중 가장 큰수에 +1 한 값을 넣어주면 된다.

> **두 번째 부터 네 번째 인자** 는 각각 읽기, 쓰기, 예외 상황 이벤트를 감시하는데 사용되며 감시할 수 있는 이벤트는 아래와 같다.

> | fd_set인수 | 감지 기능 이벤트 |
> |----------------------------|-----------------------------------------------|
> | 읽기 가능 (readfds) | 소켓 수신 버퍼에 데이터가 도착한 경우. <br>소켓 수신 버퍼에 접속 연결 해제 요청(FIN)이 발생한 경우. <br>리스너 소켓의 경우에 새로운 접속(SYN)이 있는 경우. |
> | 쓰기 가능 (writefds) | 송신 소켓 버퍼에 빈 공간이 생긴 경우. <br>반대편에서 연결을 끊은 경우. <br>TCP 스트림에 데이터를 전송 가능한 경우. (e.g 넌블럭킹 connect) |
> | 예외 상황 (errorfds) | TCP의 OOB데이터 (URG 플래그)가 수신된 경우. |

> **다섯 번째 인자** 는 타임아웃 시간으로 `null` 을 입력하면 타임아웃이 존재하지 않으며 감시하는 이벤트가 발생할 때까지 무한 대기한다.
이벤트는 소켓의 모드(블럭킹/넌블럭킹)와 무관하게 동작하지만 감지 후 recv나 send를 할 때 블럭되지 않도록 넌블럭킹 모드를 많이 사용한다.

### 2.3 `FD_SET` 매크로
`fd_set` 구조체에 이벤트를 감시할 파일 기술자를 저장 하고자 할 때 `FD_SET` 매크로를 사용한다.
아래 `FD_SET` 매크로들은 `fd_set`을 하나의 긴 비트 단위 공간으로 인식해 0, 1을 세팅하는 구조로 되어있다.

> | | |
> |-------------------------------|--------------------------------------------------|
> | FD_ZERO(rd_set *set) | set을 초기화한다. |
> | FD_SET(int fd, fd_set *set) | set에 파일 기술자 fd를 등록한다. |
> | FD_CLR(int fd, fd_set *set) | set에서 파일 기술자 fd를 해제한다. |
> | FD_ISSET(int fd, fd_set *set) | set에 파일 기술자 fd가 등록되어 있는지 확인한다. |

`FD_SET(0, &fds)` 라고 호출 하면 fds의 첫 번재 비트가 1이 된다.
여기서 중요한 점은 `fd_set` 은 입력 값 이기도 하지만 출력 값 이기도 하다는 점이다. select가 성공적으로 등록되었다면 `DF_ISSET` 으로 확인하는 절차를 거쳐야한다.

## 3. _POLL_
_poll_ 은 _select_ 의 복잡한 인수 리스트를 정리하고 외부에 저장해야 하는 파일 기술자 번호와 개수를 _poll_ 함수에서 사용하는 인수를 그대로 사용할 수 있도록 했다. 이 덕분에 코드는 깔끔해 졌지만 **실제 성능상** 으로 **큰 향상은 없다** .

> _poll_ 도 _pselect_ 처럼 시그널 블록 마스크 기능이 있는 _ppoll_ 이 존재하지만 SUS 표준은 아니고 GNU 리눅스 확장이다.

_poll_ 은 루프 횟수를 줄일 수 있는 해결책으로 감시할 파일 기술자 번호를 `struct pollfd` 구조체에 넣어서 관리하도록 하였다. 따라서 4번 850번 파일 기술자를 감시하더라도 두 번의 루프로 해결된다. 하지만 파일 기술자의 개수가 늘어나면 늘어날수록 _select_ 와 큰 차이를 보이지 않으므로 **대용량 네트워크 프로그램** 에 도입할 때는 둘다 좋은 해결책은 아니다.

```c++
int poll(struct poolfd *ufds, 
	unsigned int nfds, 
	int timeout);

struct pollfd
{
	int fd;         // 파일 기술자
	short events;   // 요구된 이벤트
	short revents;  // 반환된 이벤트
};
```

**리턴값**
> 성공시 **이벤트가 발생한 파일 기술자 수**, 실패시 **-1**, 타임아웃 시간이 지나면 **0** 이 리턴된다.

**인자**
> **첫 번째 인자** `pollfd` 는 감시할 파일 기술자와 이벤트의 정보를 담고 있는 배열 주소이다. 
> 각 이벤트는 아래와 같이 정의되어 있다.

> |  |  |
> |----------|---------------------------------------------------------------------------------------|
> | POLLIN | 읽기 버퍼에 데이터가 있다. (cf. TCP의 연결 요청도 일기 데이터에 포함됨) |
> | POLLPRI | 우선순위 데이터를 사용한다. (e.g. TCP의 OOB 데이터가 감지됨) |
> | POLLOUT | 쓰기 버퍼가 사용 가능한 상태 (e.g.  버퍼가 비워졌거나 넌블럭킹 connect가 완료된 상태) |
> | POLLERR | 연결에 에러가 발생함 |
> | POLLHUP | 닫힌 연결에 쓰기 시도 감지 |
> | POLLNVAL | 무효한 파일 기술자를 지정한 에러(연결되지 않은 파일 기술자를 지정함) |

## 4. _EPOLL_

### 4.1 고성능 네트워킹 모델

1990년대 말 인터넷이 폭발적으로 증대 되면서 C10K문제가 생기게 됐고 이 문제를 해결하기 위해 
_fork_ 를 통한 프로세스 다중화나 멀티 스레드를 사용하는 방법들이 고안되었다.

그러나 실제 문제는 I/O 속도보다는 네트워크상의 응답속도가 느린 것 이기었기 때문에 
기존의 poller호출을 더 효율적으로 만들어야 했다. 그래서 아예 새로운 형태의 poller인
_epoll, kqueue ,/dev/poll_ 등이 도입되었다.

### 4.2 _SELECT, POLL_ , _EPOLL_ 의 차이

1984년 IBM에서 발표한 PC/AT는 6MHz의 CPU와 512KB의 램, 20MB의 하드디스크를 사용했다.
이처럼 1MiB도 되지 않는 작은 메모리에서 API를 구동해야 했기 때문에 대부분의 API함수들은 
커널 내부에서 메모리를 소모하지 않도록 설계해야만 했다. 즉, 시스템 콜 함수들은 호출될 때 필요한 모든 정보를 커널에 복사하고
커널은 모든 작업을 완료하면 다시 유저 영역으로 모든 정보를 복사 후 정보를 폐기하는 과정을 거쳤다.

따라서 시스템 콜 횟수가 증가할 수록 복사하는 메모리 용량이 비례해서 늘어나므로 대역폭을 많이 소모하게 된다. 
이렇게 커널 내부에 어떠한 상태도 기록하지 않는 API를 _Stateless API_ 라고 부른다. _select, poll_ 함수도 _Stateless API_ 이므로
select는 항상 호출 할 때 마다 정보를 복사한다.

이런 비효율적인 구조의 근본적 원인은 감시 대상인 파일 기술자 정보를 커널 내부에 보관하지 않기 때문에 매번 함수 호출이 있을 때 마다
파일 기술자 정보를 복사하기 때문이라고 생각했다. 그리고 1990년대 후반에서 2000년대로 넘어오면서 메모칩 가격이 대폭 하향되어 메모리를 아껴야하는 필요성이 적어지게되어
메모리를 낭비하더라고 성능을 높일 수 있는 방향으로 프로그램 페러다임이 변하게 되었다. 따라서 새롭게 제안되는 함수들은
감시할 파일 기술장 정보에 변화가 있을 때만 파일 기술자 정보를 복사하도록 설계 하였다.

처음 구현된 것은 FreeBSD 프로젝트의 Jonathan Lemon이 _Stateful API_ 로 설계한 _kqueue_ 로 커널과 메모리 사이에 메모리 복사가 적게 일어나고
매번 루프를 돌 필요가 없기 때문에 상당히 좋은 효과를 보여주었다. 이후 리눅스에서도 동일한 개념의 _epoll_ 을 도입한다.

### 4.3 _EPOLL_ (event poll)

- Stateful 함수
- Edge trriger 지원

#### 4.3.1 함수 원형

```C++
// 함수 원형 확보
int epoll_create(int size);
// 파일 기술자와 감시할 이벤트 등록/해제
// 감시할 조건에 변화가 있을 때만 호출함
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
// epoll에서 이벤트를 리턴 받음
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

#### 4.3.2 _EDGE TRIGGER_, _LEVEL TRIGGER_
- _edge trigger_ : 전위 차가 발생하는 edge부분에서 상태 변화를 감지하는 것
- _level trigger_ : 상태의 변화가 일정한 전위 수준(votage level)을 넘었는지 감지 하는 것

> poll이 1을 리턴했고 검사해보니 4번 소켓에 _POLLIN_ 이벤트가 있었다고 할때 4번 소켓에 _RECV_ 를 하지 않으면 
>
> _level trriger_ 에서 poll은 다시 _POLLIN_ 을 발생시킨다.
>
> _edge trriger_ 는 더이상 이벤트를 발생 시키지 않는다.

_edge trriger_ 는 어플리케이션 개발에서 많은 이점을 준다.

예를 들어 어플리케이션 헤더가 40 바이트의 고정된 길이를 가지고 있을 때 30바이트만 수신이 되었다면
_edge trigger_ 는 아무런 처리를 하지 않고 다음 데이터가 수신되었을 때 데이터를 처리하면 되지만
_level trriger_ 는 지속적으로 수신할 데이터가 있음을 감지하기 때문에 불필요한 오버헤드가 생기고 
코드 상에서도 별도의 처리가 필요하게 된다.

_edge trriger_ 는 비동기 소켓 프로그래밍을 할 때도 유용한 면이 많다.

_epoll_wait_ 를 호출해 이벤트를 감지하는 부분과 소켓으로 부터 데이터를 수신하는 디스패쳐(dispatcher)를
스레드로 분리했을 때 _level trigger_ 를 사용하면 중복된 호출이 발생할 수 있어 소켓 버퍼를 읽었는지 확인을 해야하기 때문이다.
과거에는 이 부분에 대한 작업이 매우 다로웠으나 _edge trriger_ 의 사용으로 쓰레드간 동기화 차이로 발생하는 문제가 상당부분 쉽게 해결되었다.

### 4.3.3 _EPOLL_ 의 생성
```C++
int epoll_create(int size);
// size 가 의미가 없어졌기 때문에 flag만 세팅할 수 있게 변경됨
int epoll_create1(int flags);
```
`epoll_create` 의 인수인 `size`는 _epoll_ 에 등록할 수 있는 파일 기술자들의 개수 제한으로 
커널이 파일 기술자와 이벤트 정보를 담아둘 메모리 크기를 결정한다. 하지만 **커널 2.6.8 부터는 이 값은 무시**되고
커널이 동적으로 메모리를 관리한다.
> `size`값이 완전히 무시되는 것은 아니고 양수를 지정하게 되어 있어 일반적으로 1을 넣어 생성하고
> 실제 최대 크기는 커널의 `fs.epoll.max_user_watches` 값의 제한을 받는다.

> `fs.epoll.max_user_watches`의 기본 값은 시스템 _Low Memory_ 의 약 4% 수준을 사용하는 것으로 결정되는데
epoll 1개의 파일 기술자가 약 90~160 바이트를 사용하므로 _free low memory * 0.04 / 90_ 정도가 된다. 64bit 커널은
_low momory, high memory_ 를 나누지 않으므로 _Total Free memory_ 로 계산된다.

### 4.3.4  _EPOLL_ 의 제어
```c++
int epoll_ctl(int epfd, int op, struct epoll_event *event);
struct epoll_event{
    __uint32_t events;   // Epoll events
    epoll_data_t data;   // User data
}
typedef union epoll_data{
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;
```
_epoll_ 에 파일 기술자를 등록하거나 삭제, 교체 하는 작업은 epoll_ctl로 하게된다.
4개의 인수를 사용하는데 순서대로 아래와 같은 의미를 갖는다.
- epoll_create_로 생성된 epoll 파일 기술자
- 조작할 작업(Operation)
- 적용 대상 파일 기술자
- 이벤트 구조체

**op 인수 값**

> |  |  |
> |---------------|----------------|
> | EPOLL_CTL_ADD |해당 파일 기술자와 이벤트를 epoll에 등록한다. |
> | EPOLL_CTL_DEL |해당 파일 기술자의 정보를 epoll에서 제거한다. |
> | EPOLL_CTL_MOD |해당 파일 기술자의 이벤트를 교체한다. |

**4번째 인수** 는 `epoll_event`구조체로서 등록하거나 교체할 때 사용하는 멤버는 2개이다.
- data.fd : 감시할 파일 기술자
- events : 감시할 이벤트 지정

**events 인수 값 (TCP 기준)**
> |  |  |
> |---------------|----------------|
> | EPOLLIN |읽기 버퍼에 데이터가 있다. |
> | EPOLLPRI |우선 순위 데이터를 사용한다. |
> | EPOLLOUT |쓰기 버퍼가 사용 가능한 상태<br>(e.g. 버퍼가 비워졌거나 넌블럭킹 connect가 완료된 상태) |
> | EPOLLERR |연결에 에러가 발생함 |
> | EPOLLHUP |닫힌 연결에 쓰기 시도 감지 |
> | EPOLLONESHOT |이벤트 감시를 일회용으로 사용한다.<br>한번 감지된 후에는 해당 파일 기술자의 이벤트 마스크를 비활성화 시키므로 epoll_ctl로 이벤트 마스크를 재설정 할 때 까지 (EPOLL_CTL_MOD 행동) 이벤트를 감지하지 않는다. |
> | EPOLLET |이벤트를 엣지 트리거로 작동시킨다. |

### 4.3.5  _EPOLL_ 의 이벤트 수신

```c++
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout, const sigset_t *sigmask);
```
**두 번째 인수**인 `events`는 감지된 이벤트를 반환할 메모리 공간으로 그 크기를 maxevents에 지정해 줘야한다.
공간이 부족할 경우에는 이벤트의 일부만 반환될 수도 있다.

**네 번째 인수** `timeout`은 밀리초 단위의 타임아웃으로 음수를 주면 무한으로 대기하게 된다. 
**0**을 주면 넌블럭킹으로 동작해 바로 리턴하게 되어 있다.

`epoll wait`함수는 성공하면 이벤트가 발생한 파일 기술자의 개수를 리턴한다는 점에서 _poll_ 이나 _select_ 와 같지만
`events`인수에는 이벤트가 발생한 파일 기술자만 반환된다는 점에서 중요한 차이를 만든다.

_poll_ 이나 _select_ 처럼 일일이 루프를 돌면서 검사할 필요가 없기 때문에 대규모 소켓을 감시할 때 효율이 높다.

> 실제 TCP 소켓을 사용하는 메신저나 각종 통신 서비스중에 리얼타임 poller가 감지하는 소켓이 5%미만이라는 통계가 있다.
> 즉 1000개의 소켓을 감시하고 있다면 실제로 이벤트가 발생한 소켓은 50개 미만이라는 것이다.
> 만약 소켓이 10만개라면 10만개의 소켓을 검사하는 것과 5000개의 소켓을 검사하는 것은 응답 속도에서 상대적으로 큰 차이를 보이게된다.


