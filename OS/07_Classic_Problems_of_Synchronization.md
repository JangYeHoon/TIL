# Classic Problems of Synchronization

- Bounded-Buffer Problem
- Readers-Writes Problem
- Dining-Philosophers Problem



## Bound-Buffer Problem

- N개의 버퍼가 있을 때 각 버퍼는 하나의 아이템을 포함할 수 있음

- Semaphores

  - Mutex
    - 버퍼 풀 액세스를 위한 상호 배제
    - 1로 초기화
  - Full
    - 전체 버퍼 수를 계산
    - 0으로 초기화
  - Empty
    - 빈 버퍼 수 계산
    - N으로 초기화

  ```c
  Producer:
  do {
  	// produce an item in nextp
  	wait(empty);
  	wait(mutex);
  	// add nextp to buffer
  	signal(mutex);
  	signal(full);
  } while(TRUE)
  ```

  ```c
  Consumer:
  do {
  	wait(full);
  	wait(mutex);
  	// remove an item from the
  	// buffer to nextc nextp
  	signal(mutex)
  	signal(empty)
  } while(TRUE)
  ```



## Readers-Writers Problem

- 다수의 동시 프로세스에서 데이터 집합 공유

  - Readers : 데이터 세트 읽기만 수행, 업데이트는 수행하지 않음
  - Writers : 읽기와 쓰기 모두 가능

- Problem

  - 여러 readers가 동시에 읽을 수 있음. 한명의 writer만 공유 데이터에 액세스할 수 있음

- Semaphores & shared data

  - read_count
    - 데이터 세트를 읽고 있는 프로세스 수 추적
  - mutex
    - read_count를 업데이트할 때 상호 배제를 보장
    - 1로 초기화
  - rw_mutex
    - Writer를 위한 상호 배제 세모포어 기능
    - 1로 초기화

  ```c
  Writer:
  do {
  	wait(rw_mutex);
  	// writing is performed
  	signal(rw_mutex);
  } while (TRUE)
  ```

  ```c
  Reader:
  do {
  	wait(mutex);
  	readcount++;
  	if (readcount == 1)
  		wait(rw_mutex);
  	signal(mutex):
  	// reading is performed
  	wait(mutex);
  	readcount--;
  	if (readcount == 0)
  		signal(rw_mutex);
  	signal(mutex);
  } while(TRUE);
  ```



## Dining-Philosophers Problems

- 데드락에 관한 유명한 비유

- 철학자는 생각과 식사를 번갈아가면서 살아감

- 철학자가 5명인 경우

  - 젓가락은 총 5개, 단 음식을 먹으려면 2개의 젓가락을 사용해야 함
  - 동시에 음식을 먹을 수 있는 사람은 두명 
  - 운이 좋으면 5명의 철학자들이 돌아가면서 생각과 식사를 이어감
  - 하지만 모두가 젓가락을 들고 식사를 하려고 시도하면 누구도 식사를 할 수 없는 상태. 즉, 데드락에 빠짐
  - 공유 데이터
    - 밥 그릇(데이터 세트)
    - Semaphore chopstick[5]가 1로 초기화

- Philosopher i의 구조

  ```c
  Philosopher i:
  do {
  	wait(chopstick[i]);
  	wait(chopstick[(i+1)%5]);
  	// eat
  	signal(chopstick[i]);
  	signal(chopstick[(i+1)%5]);
  } while(TRUE)
  ```

- Monitor Solution to Dining-Philosophers problem

  ```c
  monitor DiningPhilosophers
  {
  	enum {THIKING, HUNGRY, EATING} state [5];
  	condition self [5];
  	
  	void pickup (int i) {
  		state[i] = HUNGRY;
  		test(i);
  		if (state[i] != EATING) self[i].wait;
  	}
  	
  	void putdown (int i) {
  		state[i] = THINKING;
  		// test left and right neighbors
  		test((i + 4) % 5);
  		test((i + 1) % 5);
  	}
  	
  	void test (int i) {
  		if ((state[(i + 4) % 5] != EATING) && 
  			(state[i] == HUNGRY) && 
  			(state[(i + 1) % 5] != EATING)) {
  			state[i] = EATING;
  			self[i].signal();
  		}
  	}
  	
  	initialization_code() {
  		for (int i = 0; i < 5; i++)
  			state[i] = THINKING;
  	}
  }
  ```

- 각 철학자 i는 다음 순서로 `pickup()`과 `putdown()` 작업을 호출

  ```c
  DiningPhilosophers.pickup(i);
  	EAT
  DiningPhilosophers.putdown(i);
  ```

- 데드락은 발생하지 않지만 기아(starvation)은 발생할 수 있음



##### Reference

- https://parksb.github.io/article/10.html
- Provided by Operating Systems Concepts, 10th Edition

