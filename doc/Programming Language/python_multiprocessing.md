# multiprocessing 모듈
- CPU 바운드 VS I/O 바운드
  - <img width="562" alt="image" src="https://user-images.githubusercontent.com/47103479/222959897-1142d4a6-236a-4508-9461-0914ecfce8ca.png">
  
    - https://leimao.github.io/blog/Python-Concurrency-High-Level/
  - CPU 바운드 : CPU 바운드는 작업을 완료하는 시간이 주로 중앙 프로세서의 속도에 의해 결정되는 조건을 나타냅니다. 클럭 속도가 더 빠른 CPU일수록 프로그램의 성능이 더 높아질 것입니다.
  - 대부분의 단일 컴퓨터 프로그램은 CPU 바인딩입니다.
  - <img width="559" alt="image" src="https://user-images.githubusercontent.com/47103479/222959904-553327c3-bd65-43e9-83a6-0a5d57a03cf7.png">

    - https://leimao.github.io/blog/Python-Concurrency-High-Level/
  - I/O 바운드는 계산을 완료하는 데 걸리는 시간이 주로 입력/출력 작업이 완료되기를 기다리는 데 소요되는 시간에 의해 결정되는 조건을 나타냅니다. 이것은 CPU 바인딩되는 작업의 반대입니다. 
  - CPU 클럭 속도를 높이면 프로그램 성능이 크게 향상되지 않습니다. 반대로 더 빠른 메모리 I/O, 하드 드라이브 I/O, 네트워크 I/O 등과 같이 더 빠른 I/O가 있으면 프로그램의 성능이 향상됩니다.
  - 대부분의 웹 서비스 관련 프로그램은 I/O 바운드입니다
- 프로세스가 생성되면 CPU는 프로세스가 해야 할 작업을 수행합니다. CPU가 처리하는 작업의 단위가 바로 스레드(스레드 : 프로세스 내에서 실행되는 여러 작업의 단위)로  스레드 한 개로 동작하면 싱글 스레드, 여러 개의 스레드가 동작하면 멀티 스레딩입니다.
- 멀티 스레딩에서 스레드는 다수의 스레드끼리 메모리 공유와 통신이 가능하며 자원의 낭비를 막고 효율성을 향상합니다. 한 스레드에 문제가 생기면 전체 프로세스에 영향을 미칩니다.
- 멀티 스레드
  - <img width="561" alt="image" src="https://user-images.githubusercontent.com/47103479/222959999-99fa07fe-c546-4252-be3c-9d56f3572804.png">

    - https://leimao.github.io/blog/Python-Concurrency-High-Level/
  - 한 개의 단일 어플리케이션(응용프로그램) -> 여러 스레드로 구성 후 작업 처리
  - 시스템 자원 소모 감소(효율성), 처리량 증가(Cost 감소) 
  - 통신 부담 감소, 디버깅 어려움, 단일 프로세스에는 효과 미약, 자원 공유 문제(교착 상태, 데드락) , 프로세스 영향을 줍니다. 
- 멀티 프로세스
  - <img width="562" alt="image" src="https://user-images.githubusercontent.com/47103479/222959972-6eacc1eb-7266-4244-b0a3-87d660b11ef8.png">
  
    - https://leimao.github.io/blog/Python-Concurrency-High-Level/
  - 한 개의 단일 어플리케이션(응용프로그램) -> 여러 프로세스로 구성 후 작업 처리
  - 한 개의 프로세스 문제 발생은 확산 없습니다.(프로세스 Kill)
  - 캐시 체인지, Cost 비용 매우 높으며(오버헤드) 복잡한 통신 방식을 사용합니다.
- Multiprocessing VS Threading VS AsyncIO in Python
  - <img width="562" alt="image" src="https://user-images.githubusercontent.com/47103479/222960034-01ed93bb-5d3d-48c9-a35a-fb4080402d66.png">

    - https://leimao.github.io/blog/Python-Concurrency-High-Level/
  - 모든 프로세스가 서로 독립적이고 메모리를 공유하지 않기 때문에 Multiprocessing Python 프로그램은 많은 기본 스레드에서 여러 Python 인터프리터를 생성하여 사용 가능한 모든 CPU 코어와 기본 스레드를 완전히 활용할 수 있습니다. 
  - Python의 CPU 바인딩 작업의 경우 multiprocessing 성능을 최대화하는 데 사용하기에 완벽한 라이브러리입니다
  - Python을 사용하면 threading I/O를 기다릴 때 유휴 상태인 CPU를 더 잘 활용할 수 있습니다. 
  - 요청 대기 시간을 중첩하여 성능을 향상 할 수 있습니다. 모든 스레드가 동일한 메모리를 공유하기 때문에 파이썬에서 threading을 사용하여 협업 작업을 수행하려면 주의를 기울여야 하며 필요할 때 잠금을 사용해야 합니다.
  - Python의 I/O 바인딩 작업의 경우 threading 성능을 최대화하는 데 사용할 수 있는 좋은 라이브러리 후보입니다.
  - Python을 사용하면 asyncio I/O를 기다릴 때 유휴 상태인 CPU를 더 잘 활용할 수 있습니다. threading과 차이점은 단일 프로세스 및 단일 스레드라는 것입니다.
  - |동시성 유형|특징|사용 기준|비유|
    |:--:|:--:|:--:|:--:|
    |Multiprocessing|다중 프로세스, 높은 CPU 사용률|CPU-bound|10개의 주방, 10명의 셰프, 10가지 요리
    |Threading|단일 프로세스, 다중 스레드, 선점형 멀티태스킹, OS가 작업 전환을 결정합니다.	|Fast I/O-bound|하나의 주방, 10명의 셰프, 10가지 요리, 10명의 셰프가 모이면 주방은 붐빔
    |AsyncIO|단일 프로세스, 단일 스레드, 협력 멀티태스킹, 작업이 협력적으로 스위칭을 결정합니다.|Slow I/O-bound	|주방 하나, 셰프 한 명, 요리할 요리 열 가지
- multiprocessing 모듈 
  - 프로세스와 스레드 기반의 병렬 처리를 사용해서 작업을 대기열에 분산시키고 프로세스 간에 데이터를 공유할 수 있도록 합니다. 단일 컴퓨터의 멀티 코어 병렬 처리에 초점이 맞춰져 있습니다.
  - 가장 일반적인 사용법은 CPU 위주의 작업을 여러 프로세스로 병렬화하는 것으로 I/O 위주의 문제를 병렬화하는 데 OpenMP 모듈을 사용할 수도 있습니다.
  - multiprocessing 모듈로 처리할 수 있는 전형적인 작업의 예
    - CPU 위주의 작업을 Process나 Pool 객체를 사용해 병렬화
    - dummy 모듈을 사용해서 I/O 위주의 작업을 스레드를 사용하는 Pool로 병렬화
    - Queue를 통해 피클링(pickling)한 결과를 공유
    - 병렬화한 작업자 사이에서 바이트,원시 데이터 타입, 사전, 리스트 등의 상태를 공유
  - 파이썬 스레드가 OS의 네이트브 스레드이며(즉 파이썬 스레드는 실제 운영체제가 실행하는 스레드로, 에뮬레이션 된 것이 아닙니다), GIL에 의해 제한되며, 한 번에 오직 한 스레드만 파이썬 객체들과 상호작용할 수 있음을 알아야만 합니다.
  - 프로세스를 사용하여 여러 파이썬 인터프리터를 병렬로 실행할 수 있고, 각각의 인터프리터는 독립된 메모리 공간과 GIL을 가지며, 각각 순차적으로 실행됩니다.(따라서 각각의 GIL을 두고 경쟁하지 않음), 파이썬에서 CPU 위주의 작업의 속도를 높이는 가장 쉬운 방법입니다.
  - multiprocessing 모듈 소개
    - 프로세스와 스레드 기반의 병렬화를 위한 저수준 인터페이스를 제공합니다.
    - 프로세스(process) : 현재 프로세스를 포크(fork)한 복사본으로 새로운 프로세스 식별자가 부여되며 운영체제상에서 별도의 자식 프로세스로 작업을 실행합니다. Process를 시작하고 상태를 쿼리 할 수 있으며, 실행할 target 메서드를 지정할 수 있습니다.
    - 풀(Pool) : Process나 threading. Thread API를 감싸서 사용하기 편한 작업자 풀(worker pool)로 만들고 작업을 공유하고 합쳐진 결과를 반환합니다.
    - 큐(Queue) : 여러 생산자(producer)와 소지바(consumer)가 사용할 수 있는 FIFO(선입선출) 대기열
    - 파이프(Pipe) : 두 프로세스 사이의 단방향 또는 양방향 통신 채널 
    - 관리자(Manager) : 프로세스 간에 파이썬 객체를 공유하는 고수준의 관리된 인터페이스
    - ctypes :  프로세스를 포크한 다음 여러 프로세스가 원시 데이터 타입(예: 정수, 실수, 바이트)을 공유하게 해 줍니다. 

# Reference
- https://www.oreilly.com/library/view/high-performance-python/9781492055013/
- https://leimao.github.io/blog/Python-Concurrency-High-Level/
