---
title: 동시성처리
categories:
  - Dev
tags:
  - thread
  - event
  - asynchronous
  - concurrency
  - interview
last_modified_at: 2023-05-02T00:28:00-00:00
---

동시성 처리는 애플리케이션 내에서 동시에 실행되는 여러 스레드 또는 프로세스를 관리하는 프로세스를 말합니다. 이는 애플리케이션이 여러 프로세서 또는 코어를 활용하고 여러 요청 또는 작업을 동시에 처리할 수 있게 해주기 때문에 소프트웨어 개발에서 중요한 측면입니다.

---

동시성 처리에는 여러 가지 접근 방식이 있으며, 각 접근 방식에는 고유한 장단점이 있습니다. 가장 일반적인 접근 방식은 다음과 같습니다:

- 스레드 기반 동시성: 이 접근 방식에서는 애플리케이션이 여러 개의 스레드를 생성하고 관리하며, 각 스레드는 별도의 작업을 동시에 실행할 수 있습니다. 이를 통해 애플리케이션은 여러 프로세서 또는 코어를 활용할 수 있으며 성능과 확장성을 향상시킬 수 있습니다. 그러나 충돌과 오류를 피하기 위해 스레드를 신중하게 동기화하고 관리해야 하므로 애플리케이션에 복잡성을 더할 수도 있습니다.
- 이벤트 기반 동시성: 이 접근 방식에서는 애플리케이션이 이벤트 루프를 사용하여 작업 실행을 관리하고 예약합니다. 이벤트 루프는 사용자 입력이나 네트워크 요청과 같은 들어오는 이벤트를 수신 대기하고 적절한 작업을 실행하도록 디스패치합니다. 이 접근 방식은 성능과 확장성을 향상시킬 수 있지만 애플리케이션을 디버깅하고 유지 관리하기가 더 어려워질 수 있습니다.
- 비동기 프로그래밍: 이 접근 방식에서는 애플리케이션이 비동기 함수 및 콜백을 사용하여 작업 실행을 관리합니다. 비동기 함수를 사용하면 스레드가 작업을 시작한 다음 초기 작업이 실행되는 동안 다른 작업을 계속 실행할 수 있습니다. 이는 성능과 확장성을 향상시킬 수 있지만 코드를 읽고 이해하기 어렵게 만들 수도 있습니다.

전반적으로 동시성 처리는 애플리케이션이 여러 프로세서 또는 코어를 활용하고 여러 작업을 동시에 처리할 수 있도록 하는 소프트웨어 개발의 중요한 측면입니다. 선택하는 접근 방식은 특정 요구 사항과 애플리케이션의 요구 사항에 따라 달라집니다.

| 동시성 | 병렬성 |
| --- | --- |
| 동시에 실행되는 것 같이 보이는 것 | 실제로 동시에 여러 작업이 처리되는 것 |
| 싱글 코어에서 멀티 쓰레드를 동작 시키는 것 | 멀티 코어에서 멀티 쓰레드를 동작 시키는 방식 |
| 한번에 많은 것을 처리 | 한번에 많은 일을 처리 |
| 논리적인 개념 | 물리적인 개념 |

![concurrent_parallel](/assets/images/concurrent_parallel.png "Concurrent Parallel")


## What Is Concurrency?

python의 동시성은 스레드, 작업 및 프로세스와 같은 동시 요소의 조정을 나타낸다.

이러한 요소는 순서대로 실행되는 일련의 명령을 나타내며 서로 다른 사고 방식으로 생각할 수 있다

python의 동시성은 다중 처리, 스레딩 또는 비동기 프로그래밍을 통해 달성할 수 있다. 

멀티프로세싱은 여러 프로세스를 동시에 실행하는 반면 스레딩 및 비동기 프로그래밍은 단일 프로세서에서 실행되며 번갈아 실행되고

프로세스 속도를 높이는 방법을 찾습니다.

스레딩은 운영 체제가 언제든지 스레드를 중단하고 스레드 간에 전환할 수 있는 선점형 멀티태스킹을 사용한다.

이것은 스레드의 코드가 스위치를 처리할 필요가 없기 때문에 편리하지만 전환이 언제든지 발생할 수 있으므로 어려울 수 있다.

단일 python 문 중에도 마찬가지이다.

Asyncio는 협력 멀티태스킹을 사용하여 작업이 협업하고 전환할 준비가 되면 알림을 보낸다.

이렇게 하려면 작업 코드를 수정하기 위한 추가 작업이 필요하지만 전환이 발생하는 시기를 알고

설계를 단순화하고 python 문 중에 중단을 방지하는 이점을 제공한다.

## What Is Parallelism?

병렬 처리는 여러 CPU 코어를 사용하여 작업을 동시에 실행하는 것을 말하며 python의 다중 처리를 통해 달성할 수 있습니다. 다중 처리에서 각 프로세스는 자체 python 인터프리터에서 실행되며 다른 코어에서 실행될수 있으므로 실제 동시 실행이 가능하다

동시성과 병렬성은 작업이 실행되고 전환되는 방식이 다르다.

| Concurrency Type | Switching Decision | Number of Processors |
| --- | --- | --- |
| 선제적 멀티태스킹(스레딩) | 운영 체제는 python 과 독립적으로 작업을 전환할 시기를 결정 | 1 |
| 협력적 멀티태스킹 (비동기식) | 태스크는 제어를 포기할 시기를 결정 | 1 |
| 다중 처리 (멀티프로세싱) | 프로세스는 서로 다른 프로세서에서 동시에 실행 | N |

### Synchronous Version

```python
import requests
import time

def download_site(url, session):
    with session.get(url) as response:
        print(f"Read {len(response.content)} from {url}")

def download_all_sites(sites):
    with requests.Session() as session:
        for url in sites:
            download_site(url, session)

if __name__ == "__main__":
    sites = [
        "https://www.jython.org",
        "http://olympus.realpython.org/dice",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")

# Downloaded 160 in 14.289619207382202 seconds
```

- 동기식 버전이 좋은 이유

이 버전의 코드의 가장 큰 장점은 바로 쉽다는 것. 생각의 흐름이 하나뿐이므로 다음 단계가 무엇이고 어떻게 작동할지 예측할 수 있음

- 동기식 버전의 문제점

동기식 버전의 가장 큰 문제점은 앞으로 제공할 다른 솔루션에 비해 상대적으로 느리다는 점. 

속도가 느리다고 항상 큰 문제가 되는 것은 아니지만 프로그램이 자주 실행된다면 어떨까? 실행하는 데 몇 시간이 걸린다면 어떨까?

스레딩을 사용하여 이 프로그램을 다시 작성하여 동시성에 대해 살펴본다.

### threading Version

```python
import concurrent.futures
import requests
import threading
import time

thread_local = threading.local()

def get_session():
    if not hasattr(thread_local, "session"):
        thread_local.session = requests.Session()
    return thread_local.session

def download_site(url):
    session = get_session()
    with session.get(url) as response:
        print(f"Read {len(response.content)} from {url}")

def download_all_sites(sites):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        executor.map(download_site, sites)

if __name__ == "__main__":
    sites = [
        "https://www.jython.org",
        "http://olympus.realpython.org/dice",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")

# Downloaded 160 in 3.7238826751708984 seconds
```

- Why the threading Version Rocks

- The Problems with the threading Version

14초에 비하면 훨씬 빠르긴 하지만 이를 구현하려면 코드가 조금 더 필요하며 스레드 간에 어떤 데이터가 공유되는지 잘 생각해야 합니다

스레드는 미묘하고 감지하기 어려운 방식으로 상호 작용할 수 있다. 이러한 상호 작용으로 인해 경쟁 조건이 발생할 수 있으며,

이로 인해 찾기가 매우 어려운 무작위적이고 간헐적인 버그가 자주 발생할 수 있다.

### asyncio Version

### asyncio 기본

asyncio의 일반적인 개념은 이벤트 루프라고 하는 단일 파이썬 객체가 각 작업이 실행되는 방법과 시기를 제어한다는 것

이벤트 루프는 각 작업을 인식하고 어떤 상태인지 알고 있습니다. 실제로 태스크의 상태는 여러 가지가 있을 수 있지만,

여기서는 두 가지 상태만 있는 간단한 이벤트 루프를 상상

준비 상태는 작업이 수행해야할 작업이 있고 실행할 준비가 되었음을 나타내며, 
대기 상태는 작업이 네트워크 작업과 같은 외부 작업이 완료되기를 기다리는 중임을 의미

단순화된 이벤트 루프는 이러한 각 상태에 대해 하나씩 두 개의 작업목록을 유지

준비된 작업 중 하나를 선택하고 다시 실행을 시작합니다. 해당 작업은 이벤트 루프에 협조적으로 제어권을 다시 넘길 때까지 완전한 제어권을 갖는다.

실행중인 작업이 이벤트 루프에 제어권을 다시 넘기면 이벤트 루프는 해당 작업을 준비 또는 대기 목록에 배치한 다음 대기 목록에 있는 각 작업을 살펴보고 I/O 작업이 완료되어 준비가 되었는지 확인. 준비 목록에 있는 작업은 아직 실행되지 않았음을 알기 때문에 아직 준비된 상태임을 알수 있음.

모든 작업이 다시 올바른 목록으로 정렬되면 이벤트 루프는 실행할 다음 작업을 선택하고 프로세스가 반복됩니다. 단순화된 이벤트 루프는 가장 오래 대기 중인 작업을 선택하여 실행합니다. 이 프로세스는 이벤트 루프가 완료될 때까지 반복됩니다.

비동기화의 중요한 점은 태스크가 의도적으로 제어권을 포기하지 않는다는 것입니다. 작업 도중에 중단되지 않습니다. 따라서 스레딩보다 비동기에서 리소스를 좀 더 쉽게 공유할 수 있습니다. 코드를 스레드에 안전하게 만드는 것에 대해 걱정할 필요가 없습니다.

### async and await

### Back to Code

```python
import asyncio
import time
import aiohttp

async def download_site(session, url):
    async with session.get(url) as response:
        print("Read {0} from {1}".format(response.content_length, url))

async def download_all_sites(sites):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for url in sites:
            task = asyncio.ensure_future(download_site(session, url))
            tasks.append(task)
        await asyncio.gather(*tasks, return_exceptions=True)

if __name__ == "__main__":
    sites = [
        "https://www.jython.org",
        "http://olympus.realpython.org/dice",
    ] * 80
    start_time = time.time()
    asyncio.get_event_loop().run_until_complete(download_all_sites(sites))
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} sites in {duration} seconds")

# Downloaded 160 in 2.5727896690368652 seconds
```

스레딩 예제에서는 최적의 스레드 수가 무엇인지 명확하지 않았지만

비동기화의 멋진 장점 중 하나는 스레딩보다 훨씬 더 잘 확장된다는 것. 각 작업은 스레드보다 훨씬 적은 리소스와 시간이 소요되므로 더 많은 작업을 생성하고 실행하는 것이 효과적.

또 다른 미묘한 문제는 작업 중 하나가 협력하지 않으면 협력적 멀티태스킹의 모든 이점이 사라진다는 것.

코드의 사소한 실수로 인해 한 작업이 실행되지않고 프로세서를 오랫동안 잡아두어 실행이 필요한 다른 작업을 고갈시킬 수 있습니다.

태스크가 제어권을 다시 넘겨주지 않으면 이벤트 루프가 중단될 방법이 없습니다.

### multiprocessing Version

```python
import requests
import multiprocessing
import time

session = None

def set_global_session():
    global session
    if not session:
        session = requests.Session()

def download_site(url):
    with session.get(url) as response:
        name = multiprocessing.current_process().name
        print(f"{name}:Read {len(response.content)} from {url}")

def download_all_sites(sites):
    with multiprocessing.Pool(initializer=set_global_session) as pool:
        pool.map(download_site, sites)

if __name__ == "__main__":
    sites = [
        "https://www.jython.org",
        "http://olympus.realpython.org/dice",
    ] * 80
    start_time = time.time()
    download_all_sites(sites)
    duration = time.time() - start_time
    print(f"Downloaded {len(sites)} in {duration} seconds")

# Downloaded 160 in 5.718175172805786 seconds
```

다중 처리 버전의 문제점은 비동기 및 스레딩 버전보다 확실히 느립니다:

하지만 cpu-bound program에서는 multiprocessing 충분히 효과적이다

```python
import multiprocessing
import time

def cpu_bound(number):
    return sum(i * i for i in range(number))

def find_sums(numbers):
    with multiprocessing.Pool() as pool:
        pool.map(cpu_bound, numbers)

if __name__ == "__main__":
    numbers = [5_000_000 + x for x in range(20)]

    start_time = time.time()
    find_sums(numbers)
    duration = time.time() - start_time
    print(f"Duration {duration} seconds")

# Duration 2.5175397396087646 seconds
# Duration 10.407078266143799 seconds (단순 실행시)
```

프로젝트에 어떤 동시성 모듈을 사용할지 결정하는 데 도움이 되는 몇 가지 결정 사항에 대해 논의해 보겠습니다.

이 프로세스의 첫 번째 단계는 동시성 모듈을 사용할지 여부를 결정하는 것입니다. 여기 예제에서는 각 라이브러리가 매우 단순해 보이지만, 동시성에는 항상 추가적인 복잡성이 수반되며 종종 찾기 어려운 버그가 발생할 수 있습니다.

성능 문제를 파악할 때까지 동시성 추가를 보류한 다음 어떤 유형의 동시성이 필요한지 결정하세요. 도널드 크누스는 "성급한 최적화는 프로그래밍에서 모든 악의 근원(또는 적어도 대부분의 악의 근원)입니다."라고 말했습니다.

`프로그램을 최적화해야 한다고 결정한 후에는 프로그램이 CPU 바인딩형인지 I/O 바인딩형인지 파악하는 것`이 다음 단계입니다. I/O 바인딩 프로그램은 대부분의 시간을 어떤 일이 일어나기를 기다리는 데 소비하는 반면, CPU 바인딩 프로그램은 데이터를 처리하거나 숫자를 최대한 빨리 처리하는 데 시간을 소비한다는 점을 기억하세요.

보시다시피 `CPU에 바인딩된 문제는 멀티프로세싱을 사용해야만 실제로 이득을 얻을 수 있습니다`. 스레딩과 비동기화는 이러한 유형의 문제에 전혀 도움이 되지 않습니다.

`I/O 바운드 문제의 경우`, 파이썬 커뮤니티에는 일반적인 경험 법칙이 있습니다: `"가능하면 비동기화를 사용하고, 꼭 필요한 경우 스레딩을 사용하세요."` 비동기화는 이러한 유형의 프로그램에 최고의 속도 향상을 제공할 수 있지만, 때로는 비동기화를 활용하기 위해 포팅되지 않은 중요한 라이브러리가 필요할 수 있습니다. 이벤트 루프에 대한 제어권을 포기하지 않는 모든 작업은 다른 모든 작업을 차단한다는 점을 기억하세요.


## 포인트 전환 로직 설계시 중복처리를 제한 그리고 동시성에 대해서.

이전 포인트 전환결과를 캐싱하면 동일한 변환을 여러 번 수행하는 것을 피할 수 있습니다.

다음은 사전을 캐시로 사용하는 간단한 Python 코드 예제입니다.

```python
conversion_cache = {}

def convert_points(company_a_points):
    if company_a_points in conversion_cache:
        return conversion_cache[company_a_points]

    # Perform the conversion logic here
    company_b_points = perform_conversion(company_a_points)

    # Cache the result
    conversion_cache[company_a_points] = company_b_points

    return company_b_points

def perform_conversion(points):
    # Implement the actual conversion logic
    return points * conversion_factor
```

이 예에서는 `conversion_cache`라는 사전을 사용하여 이전 포인트 전환결과를 저장합니다. 

`convert_points` 함수는 주어진 `company_a_points` 값에 대한 변환 결과가 이미 캐시에 있는지 확인합니다.

그렇다면 캐시된 결과가 반환됩니다.

그렇지 않으면 `perform_conversion` 함수를 사용하여 변환이 수행되고 반환되기 전에 결과가 캐시됩니다.

중복 방지 외에도 포인트 전환 기능을 개발할 때 고려해야 할 다른 조건은 다음과 같습니다.

1. 입력 유효성 검사: 입력 포인트가 유효한지 확인합니다(예: 음수가 아니거나 특정 범위 내 또는 올바른 데이터 유형). 잘못된 입력을 처리하고 오류를 반환하거나 예외를 발생시키기 위해 함수에 검사를 추가할 수 있습니다.

    ```python
    def convert_points(company_a_points):
        if company_a_points < 0:
            raise ValueError("Points must be non-negative")
        # ...
    ```

2. 전환 규칙: 전환 논리가 정확하고 A사 및 B사에서 정의한 규칙을 준수하는지 확인합니다. 여기에는 특별한 전환율 또는 추가 조건이 필요한 특정 포인트 값과 같은 극단적인 경우 처리가 포함될 수 있습니다.
3. 동시성: 응용 프로그램에 변환 기능에 동시에 액세스하는 여러 스레드 또는 프로세스가 있는 경우 잠금 또는 세마포어와 같은 동기화 메커니즘을 사용하여 캐시가 올바르게 업데이트되고 경합 상태가 방지되도록 하는 것이 좋습니다.
4. 캐시 관리: 애플리케이션의 요구 사항에 따라 보다 정교한 캐시 관리 전략이 필요할 수 있습니다. 여기에는 과도한 메모리 사용을 방지하기 위해 LRU(최소 사용) 캐시를 사용하거나 캐시 크기 제한을 설정하는 것이 포함될 수 있습니다.

동시성을 조금더 자세히 뜯어보면

다중 스레드 또는 다중 프로세스 환경에서 동시성을 처리하려면 잠금과 같은 동기화 메커니즘을 사용하여 캐시가 올바르게 업데이트되고

경합 상태가 발생하지 않도록 할수 있다.

다음은 예제 코드가 포함된 단계별 설명.

1. 필요한 모듈 가져오기
    - python 에서 잠금 및 스레드를 사용하려면 threading 모듈을 가져와야 합니다
    
    ```python
    import threading
    ```
    

2. 잠금 개체를 만든다
    
    캐시에 대한 액세스를 동기화하하는 데 사용할 잠금 개체를 만든다.
    
    ```python
    cache_lock = threading.Lock()
    ```
    

3. 전환 기능 업데이트
    
    캐시를 확인하고 업데이트하기 전에 잠금을 획득하도록 convert_points 함수를 수정한다. 
    
    잠금은 한 번에 하나의 스레드만 캐시에 액세스할 수 있도록 경합 상태를 방지한다.
    
    ```python
    def convert_points(company_a_points):
        with cache_lock:
            if company_a_points in conversion_cache:
                return conversion_cache[company_a_points]
    
            # Perform the conversion logic here
            company_b_points = perform_conversion(company_a_points)
    
            # Cache the result
            conversion_cache[company_a_points] = company_b_points
    
        return company_b_points
    ```
    
    위 예제에서는 with 문을 사용하여 잠금을 자동으로 획득하고 해제합니다
    
    스레드가 with 블록에 들어가면 잠금을 획득합니다. 첫번째 스레드가 잠금을 보유하고 있는 동안 다른 스레드가 잠금을 획득하려고 하면
    
    잠금이 해제 될때까지 대기합니다. 첫번째 스레드가 with 블록을 종료하면 잠금이 해제되어 
    
    대기 중인 스레드가 잠금을 획득하고 진행할 수 있습니다
    
    잠금을 사용하여 캐시에 대한 액세스를 동기화하면 캐시가 올바르게 업데이트되고 경합 상태가 방지됩니다.
    
    전체 예제 코드는 아래와 같습니다.
    
    ```python
    import threading
    
    conversion_cache = {}
    cache_lock = threading.Lock()
    
    def convert_points(company_a_points):
        with cache_lock:
            if company_a_points in conversion_cache:
                return conversion_cache[company_a_points]
    
            # Perform the conversion logic here
            company_b_points = perform_conversion(company_a_points)
    
            # Cache the result
            conversion_cache[company_a_points] = company_b_points
    
        return company_b_points
    
    def perform_conversion(points):
        # Implement the actual conversion logic
        return points * conversion_factor
    ```
    
    이 코드는 다중 스레드 python 애플리케이션에서 공유 리소스(캐시)에 액세스할 때 잠금을 사용하여 동시성을 처리하는 방법을 보여준다.
    
    이 접근 방식은 경합 상태를 방지하고 캐시의 정확성을 보장하는 데 도움이 될 수 있다.
    
## 참고자료

[https://seamless.tistory.com/42](https://seamless.tistory.com/42)

[https://realpython.com/python-concurrency/](https://realpython.com/python-concurrency/)