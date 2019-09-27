## 가비지 컬렉션

### 1. 가비지 컬렉션 과정
- Stop the world : GC을 실행하기 위해 JVM이 어플리케이션 실행을 멈추는 것 / GC 튜닝이란 이 시간을 줄이는 것임.
<br/><br/>

- Young 영역 : 새롭게 생성한 객체 대부분이 존재. 이 영역에서 객체가 사라질 때 Minor GC가 발생함.
- Old 영역 : Young 영역에서 살아남은 객체가 여기로 복사. Young 영역보다 크게 할당하며, 크기가 큰 만큼 GC는 적게 발생. Major GC(Full GC)가 발생.
<br/>
![Alt text](images/gc-data-flow.png)

<br/>
- Permanent 영역은 Method Area라고도 함. Old 영역에서 살아남은 객체가 영원히 남아있지는 않음. Major GC에 포함됨.
<br/>

![Alt text](images/card-table.png)

- Minor GC 발생 시, Old 영역에서 참조하는 객체들을 Card Table에서 참조하여 처리.

<br/>

### 2. Young 영역의 구성
- Eden / Survivor 2개 로 구성
- 객체 생성 -> Eden에 위치 -> GC 발생 -> 살아남은 객체는 Survivor 중 하나로 이동
- 하나의 survivor이 가득차면, 다른 survivor로 보냄 (즉, 하나는 항상 비어있어야만 함)
- GC 발생마다 Eden에서 Survivor로 이동 -> 게속해서 살아남으면 Old로 이동.

![Alt text](images/young-gc.png)

<br/>
- (참고) bump-the-pointer : Eden 영역에 할당된 마지막 객체 추적 -> 새로 넣을수 있을 지 판단
- (참고) TLABs(Thread-Local Allocation Buffers) : thread-safe 하게 위를 실행 / 각 thread별 할당된 Eden 영역만 검사

<br/>

### 3. Old 영역 GC
GC종류
 - Serial GC
 - Parallel GC
 - Parallel Old GC
 - Concurrent Mark & Sweep GC(CMS)
 - G1(Garbage First) GC
<br><br>

 - Serial GC : Mark(Old 영역에 살아있는 객체 식별) -> Sweep(힙 앞 부분부터 확인하여 살아있는 것만 남긴다) -> Compaction(각 객체들이 연속되게 쌓이도록 힙의 가장 앞 부분부터 채운다)
  적은 메모리와 코어의 개수가 적을 때.

 - Parallel GC(-XX:+UseParallelGC) : Serial GC보다 메모리가 충분하고 코어의 개수가 많을 때 유리

 - Parallel Old GC(-XX:+UseParallelOldGC) : Mark-Summary-Compaction 단계 / Summary 단계는 앞서 GC를 수행한 영역에 대해서 별도로 살아있는 객체를 식별.

 - CMS GC(-XX:+UseConcMarkSweepGC) :<br>
![Alt text](images/cms-gc.png)
 initial Mark : 클래스 로더에서 가장 가까운 객체 중 살아있는 객체만 찾음.
 Concurrent Mark : 살아있다고 확인한 객체에서 참고하고 있는 객체들을 따라가면서 확인. 다른 스레드가 실행 중인 상태에서 동시에 진행.
 Remark : Concurrent Mark에서 새로 추가되거나 참조가 끊긴 객체를 확인
 Concurrent Sweep : 쓰레기를 정리

 응답속도가 매우 중요할 때 CMS GC를 이용하지만, 다른 GC보다 메모리를 많이 사용하며, Compaction 단계가 제공되지 않기 때문에 -> 조각난 메모리 관리를 해야함.

 - G1 GC :<br>
![Alt text](images/g1-gc.png)<br>
 바둑판의 각 영역에 객체를 할당하고 GC를 실행.
해당 영역이 꽉 차면 다른 영역에서 객체를 할당하고 GC를 실행.
Young의 세가지 영역에서 데이터가  Old 영역으로 이동하는 단계가 사라진 GC방식.

### 4. Minor GC 추가
[MINOR GC 1차 발생시]<br>
EDEN에서 ALIVE하면 SURVIVE1으로 이동한다.<br>
EDEN CLEAR<br>
[MINOR GC 2차 발생시]<br>
EDEN에서 [alive] – > Survivor 2<br>
Survivor  1  [alive] – > Survivor 2<br>
Eden    Clear<br>
Survivor 2 Clear<br>
[MINOR GC 3차 발생시]<br>
객체 생성시간이 오래지나면 Eden과 survivor영역에 있는 오래된 객체들을 Old 영역으로 이동<br>
이런방식의 GC알고리즘을 Copy & Scavenge라고 한다. 이방법은 속도가 빠르며 작은 크기의 메모를 Collection하는데 매우 효과적이다. Minor Gc는 자주 일어나기 때문에 GC에 소요되는 시간이 짧은 알고리즘에 적합.<br>


### 5. Parallel Old GC(-XX:+UseParallelOldGC) 에 대한 조사

 - Java7 Update 4 이상이면 UseParallelGC와 UseParallelOldGC 중 어느 하나만 사용해도 결국 UseParallelOldGC처럼 동작한다.
 (링크 : http://sarc.io/index.php/java/478-gc-useparallelgc-useparalleloldgc)

 - 복수의 Thread로 GC작업을 수행한다<br>
 - "-XX:+UseParallelGC" 옵션을 사용하여 Minor GC 에서 parallel collector를 활성화 할 수 있다.<br>
 - "-XX:+UseParallelOldGC" 옵션을 사용하여 Major GC에서 parallel collector를 활성화 할 수 있다.<br>
 - 복수의 Thread를 사용하여 GC를 수행하기 때문에 Serial Collector에 비해서 GC가 빨리 수행된다.<br>
 - 최대 성능을 내기 위한 GC라고 생각된다.<br>
(링크 : http://dimdim.tistory.com/entry/Java-GC-%ED%83%80%EC%9E%85-%EB%B0%8F-%EC%84%A4%EC%A0%95-%EC%A0%95%EB%B3%B4-%EC%A0%95%EB%A6%AC)
