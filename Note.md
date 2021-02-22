# StudyRxSwift
Study RxSwift with KxCoding

### Hello, RxSwift
#### 1/98 Hello, RxSwift
- RxSwift를 공부하는 순서
<br>1. Swift Language
<br>2. Functional Programming, Protocol Oriented Programming
<br>3. RxSwift

- RxSwift의 장점
<br> 단순하고 직관적인 코드를 작성할 수 있다

- RxSwift 공부하는 법
<br> 처음부터 RxSwift가 무엇인지 명확히 이해하는 것은 불가능하다.
<br> 처음부터 이해하려고 하면 지친다. 코드를 통해 작은 것부터 하나씩 배워 나가라.
<br> 처음부터 이해되는 것은 말이 안 되니 이해 안 되는 부분은 그냥 건너 뛰어라.


#### 2/98 Hello Reactive Programming
- RxSwift로 코드를 작성하면 값이나 상태의 변화에 따라서 새로운 결과를 도출하는 코드를 비교적 쉽게 작성할 수 있다
  - 반응형 프로그래밍(Reactive Programming)
  
### Key Concepts
#### 3/98 Observable and Obervers #1
- 가장 중요한 Observable
<br> Observable은 Observable Sequence 또는 Sequence라고 부른다
<br> Observable은 Event를 전달한다
<br> Observer는 Observable을 감시하고 있다가(Subscribe) 전달되는 Event를 처리한다
<br> Observer를 구독자(Subscriber)라고도 한다
- Observable은 세 가지 Event를 전달한다(Next, Error, Completed)
  - Observable에서 발생한 새 Event는 Next Event를 통해서 구독자로 전달된다
  - Event에 값이 포함되어 있다면 Next와 함께 전달된다
  - RxSwift에서는 이를 Emission(방출, 배출)이라고 한다
  - Observable에서 error가 발생하면 Error event가 작동되고, 정상적으로 작동되면 Completed가 작동된다
  - Error event나 Completed event는 Emission이라고 하지 않고 Notification이라고 부른다
  - Observable의 Life cycle은 Completed에서 종료된다
  
- Observable을 생성하는 두 가지 방법
  - (1) Create 연산자를 통해서 Observable의 동작을 직접 구현하는 방법
    - Create 연산자는 Observable Protocol type에 선언되어 있는 Type 메소드이다
    - RxSwift에서는 이런 메소드들을 연산자라고 부른다
    - Create 연산자는 하나의 클로저를 파라미터로 받는다 (Observer를 받아서 Disposable을 리턴한다)
    - Disposable은 메모리 정리에 필요한 객체이다.
<br>
<pre>
<code>
Observable<Int>.create { (observer) -> Disposable in
  observer.on(.next(0))
  observer.onNext(1)
  
  observer.onCompleted()
  
  return Disposables.create()
}
</code>
</pre>

  - (2) Create가 아닌 다른 연산자를 사용하는 방법
    - 미리 정의된 규칙에 따라 이벤트를 전달하는 from 연산자
    - from 연산자를 활용해서 create 연산자로 만든 Observable과 동일한 이벤트를 전달하는 Observable을 생성한다
    - from 연산자는 파라미터로 전달한 배열에 있는 요소를 순서대로 방출하고 Completed 이벤트를 전달하는 Observable을 생성한다
    - 단순히 순서대로 방출되는 Observable을 생성할 때는 create 연산자로 직접 구현하는 것보다 from을 활용하는 것이 좋다
    - Observable이 생성된 상태일 뿐 방출되거나 event가 전달되지 않는다
    - Observable은 event가 어떤 순서로 전달되어야 하는지 정의할 뿐이다
    - 구현했던 클로저가 실행되거나 from 연산자로 만든 Observable에서 event가 전달되는 시점은 Observer가 Observable을 구독하는 시점이다
    - 이 시점에 Next event를 통해서 두 개의 정수가 순서대로 방출되고 이어서 completed event가 전달된다
<br>
<pre>
<code>
Observable.from([0, 1])
</code>
</pre>

#### 4/98 Observable and Obervers #2
- 실제로 event가 전달되는 시점은 observer가 구독을 시작하는 시점
  - Observer는 Observable에서 전달되는 event를 처리한다
  - 이것을 구독한다고 표현한다
  - 그래서 observer를 구독자라고 표현하기도 한다
- observer가 구독을 시작하는 방법
  - Observable에서 subscribe 메소드를 호출하는 것
  - subscribe 메소드는 Observable과 observer를 연결한다
  - 두 요소를 연결해야 이벤트가 전달되므로 Rx에서 가장 기초적이고 필수적이다
  
<br>
<pre>
<code>
let o1 = Observable<Int>.create { (observer) -> Disposable in
   observer.on(.next(0))
   observer.onNext(1)
   
   observer.onCompleted()
   
   return Disposables.create()
}

// 첫번째 방법
o1.subscribe { 
  print($0)
  if let elem = $0.element {
      print(elem)
  }
}
--> 출력결과
next(0)
0
next(1)
1
completed

// 두번째 방법
o1.subscribe(onNext: { elem in
   print(elem)
})
--> 출력결과
0
1

</code>
</pre>


- observer는 동시에 2개 이상의 event를 처리하지 않는다
  - Observable은 observer가 하나의 event를 처리한 후에 이어지는 event를 전달한다
  - 여러 event를 동시에 전달하지 않는다


#### 5/98 Disposables

<pre>
<code>
Observable.from([1, 2, 3])
  .subsribe(onNext: { elem in
    print("Next", elem)
  }, onError: { error in
    print("Error", error)
  }, onCompleted: {
    print("Completed")
  }, onDisposed: {
    print("Disposed")
  })
출력결과
  --> 
  Next 1
  Next 2
  Next 3
  Completed
  Disposed
  
</code>
</pre>

- Dispose는 Observable이 전달하는 event는 아니다
- 파라미터로 클로저를 전달하면 Observable과 관련된 모든 리소스가 제거된 후에 호출된다
- Observable이 Completed나 Error event로 해제되었다면 리소스도 해제된다
- 그럼에도 되도록 해제할 때는 Disposable을 사용하는 것이 좋다
- dispose()를 직접 호출하기보다는 disposeBag을 사용하는 것이 좋다
- dispose() 메소드를 직접 호출하면 completed event가 전달되지 않으므로, 직접 사용하지 않는 것이 좋다
- 만약 특정 시점에 해지하고 싶다면 take until 메소드를 사용하는 것이 좋다


#### 6/98 Operators

- Observable과 관련된 여러 메소드들을 연산자라고 부른다
- 연산자의 특징
  - 연산자들은 Observable 상에서 동작하고, 새로운 Observable을 리턴한다
  - Observable을 리턴하기 때문에 두 개 이상의 연산자를 연달아 호출할 수 있다
  - 연산자는 보통 subscribe 메소드 앞에 추가한다
  - 그래야 구독자로 전달된 최종 데이터가 우리가 원하는 데이터가 된다
  - 연산자를 호출할 때는 호출 순서에 유념해야 한다 -> 호출 순서에 따라 다른 결과가 나올 수 있다

<pre>
<code>
let bag = DisposeBag()
Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9])
    .take(5) // take 연산자는 소스 옵저버블이 방출하는 요소 중에서 파라미터로 지정한 수만큼 방출하는 새로운 옵저버블을 생성한다 -> 처음 5개의 요소만 전달한다
    .filter { $0.isMultiple(of: 2) } // 짝수만 구독자로 전달 -> 1~5 사이의 짝수인 2와 4만 전달됨
    .subscribe { print($0) }
    .disposed(by: bag)
--> 출력결과
next(2)
next(4)
Completed
</code>
</pre>

### Subjects
#### 7/98 Subject Overview
- Subject를 이해하기 위해서는 Observer와 Observable에 대해 이해해야 한다
- Observable은 이벤트를 전달한다
- Observer는 Observable을 구독하고 전달되는 이벤트를 처리한다
- Observable은 Observer와 달리 다른 Observable을 구독하지 못한다
- 마찬가지로 Observer는 다른 Observer로 이벤트를 전달하지 못한다
- 반면 Subject는 다른 Observable로부터 이벤트를 받아서 구독자로 전달할 수 있다
- 다시 말해 Subject는 Observable인 동시에 Observer이다
- RxSwift는 네 가지 Subject를 제공한다
  - PublishSubject: Subject로 전달되는 새로운 이벤트를 구독자로 전달한다
  - BehaviorSubject: 생성시점의 시작 이벤트를 지정한다 그리고 Subject로 전달되는 이벤트 중에서 가장 마지막에 전달된 최신 이벤트를 저장해 두었다가 새로운 구독자에게 최신 이벤트를 전달한다
  - ReplaySubject: 하나 이상의 최신 이벤트를 버퍼에 저장한다. Observer가 구독을 시작하면 버퍼에 있는 모든 이벤트를 전달한다
  - AsyncSubject: Subject로 Completed 이벤트가 전달되는 시점에 마지막으로 전달된 Next 이벤트를 구독자로 전달한다
- RxSwift는 Subject를 래핑하고 있는 두 가지 Relay를 제공한다 -> 이전 버전에서 제공되던 Variable이 Relay로 대체 됐다
  - PublishRelay: PublishSubject를 래핑한 것
  - BehaviorRelay: BehaviorSubject를 래핑한 것
  - Relay는 일반적인 Subject와 달리 Next 이벤트만 받고 나머지 Completed 이벤트와 Error 이벤트는 받지 않는다
  - 주로 종료 없이 계속 전달되는 이벤트 시퀀스를 처리할 때 활용한다
 
 
#### 8/98 Publish Subject
- PublishSubject는 Subject로 전달되는 이벤트를 Observer에게 전달하는 가장 기본적인 형태의 Subject이다
- Subject는 Observable인 동시에 Observer이다
  - 다른 Source로부터 이벤트를 전달받을 수 있고, 다른 Observer로 이벤트를 전달할 수 있다
  - Observable에서 Observer로 Next 이벤트를 전달할 때 Observer로 onNext 메소드를 호출하고, 파라미터로 요소를 전달한다
  - Subject 역시 Observer이기 때문에 onNext를 호출할 수 있다
- PublishSubject는 구독 이후에 전달되는 새로운 이벤트만 구독자로 전달한다
- PublishSubject는 이벤트가 전달되면 즉시 구독자에게 전달한다. 그래서 Subject가 최초로 생성되는 시점과 첫 번째 구독이 시작되는 시점 사이에 전달되는 이벤트는 그냥 사라진다.
  - 이벤트가 사라지는 것이 문제가 된다면 Replay Subject를 사용하거나 Hold Observable을 사용한다


#### 9/98 Behavior Subject
- Behavior Subject는 Publish Subject와 유사한 방식으로 동작한다
  - Subject로 전달된 이벤트를 구독자로 전달하는 것은 동일하다
  - 하지만 Subject를 생성하는 방식에 차이가 있다
    - Behavior Subject를 생성할 때는 Publish Subject와 다르게 하나의 값을 전달한다
    - 숫자를 전달하는 이유는 타입 파라미터가 Int로 선언되었기 때문
    - 또 다른 차이는 Subject를 구독할 때 나타난다
    - Publish Subject는 내부에 이벤트가 저장되지 않은 상태로 생성된다
    - 그래서 Subject로 이벤트가 전달되기 전까지 구독자로 이벤트가 전달되지 않는다
    - Behavior Subject를 생성하면 내부에 Next 이벤트가 생성되고, 생성자로 전달한 값이 저장된다
    - 새로운 구독자가 추가되면 저장되어 있던 Next 이벤트가 바로 전달된다
    - 다시 Behavior Subject로 Next 이벤트를 전달하면 Observer로 Next 이벤트가 전달된다
    - 이 시점에 새로운 Observer가 추가되면 가장 최신 Next 이벤트를 Observer로 전달한다
    
    
#### 10/98 Replay Subject
- Behavior Subject는 가장 최근 Next 이벤트 하나를 저장했다가 새로운 구독자로 전달한다
  - 최신 이벤트를 제외한 나머지 모든 이벤트는 사라진다
- 두 개 이상의 이벤트를 저장해두고 새로운 구독자로 전달하고 싶다면 Replay Subject를 사용한다
- Replay Subject는 create 메소드로 생성한다
  - Subject를 생성할 때 buffer의 크기를 지정하는데(bufferSize), bufferSize를 3으로 하면 세 개의 이벤트를 저장하는 buffer가 생성된다
  - 이 때 전달되는 것은 가장 마지막에서부터 세 개의 이벤트를 전달한다
- Replay Subject는 지정된 buffer 크기만큼 최신 이벤트를 저장하고 새로운 구독자에게 전달한다
- buffer는 메모리에 저장되기 때문에 항상 메모리 사용량에 신경 써야 한다
- 필요 이상으로 큰 buffer를 쓰는 것은 피해야 한다
- Replay Subject는 종료 여부에 관계 없이 항상 buffer에 저장돼 있는 이벤트를 새로운 구독자에게 전달한다


#### 11/98 Async Subject
- Async Subject는 이전의 Subject들과 이벤트를 전달하는 시점에 차이가 있다
- Publish Subject, Behavior Subject, Replay Subject는 Subject로 이벤트가 전달되면 즉시 구독자에게 전달한다
- 반면 Async Subject는 Subject로 Completed 이벤트가 전달되기 전까지 어떤 이벤트도 구독자로 전달하지 않는다
- Completed 이벤트가 전달되면 그 시점에 가장 최근에 전달된 넥스트 이벤트 하나를 구독자에게 전달한다
- Async Subject는 Completed 이벤트가 전달된 시점을 기준으로 가장 최근에 전달된 하나의 Next 이벤트를 구독자에게 전달한다
- 만약 Async Subject로 전달된 Next 이벤트가 없다면 그냥 Completed 이벤트만 전달하고 종료한다
- Error 이벤트가 전달된 경우에는 Next 이벤트가 구독자에게 전달되지 않고, Error 이벤트만 전달된다


#### 12/98 Relays (0216 여기까지)
- RxSwift는 Publish Relay와 Behavior Relay를 제공한다
- Relay는 Subject와 유사한 특징을 가지고 있고, 내부에 Subject를 래핑하고 있다
- Publish Relay는 Publish Subject를 래핑하고 있고, Behavior Relay는 Behavior Subject를 래핑하고 있다
- Relay는 Subject와 마찬가지로 다른 Source로부터 이벤트를 받아서 구독자에게 전달한다
- 가장 큰 차이는 Next 이벤트만 전달한다는 것이다
- Completed 이벤트와 Error 이벤트는 전달 받지도 않고, 전달 하지도 않는다
- 그래서 Subject와 달리 종료되지 않는다
- 구독자가 Dispose 되기 전까지 계속 이벤트를 처리한다
- 그래서 주로 UI 이벤트 처리에 활용된다
- Relay는 RxSwift 프레임워크가 아닌 RxCocoa 프레임워크를 통해 제공된다
- Subject에서는 onNext를 사용하지만, Relay에서 Next이벤트를 전달할 때는 accept 메소드를 사용한다
- accept 메소드를 호출하고 값을 전달하면 구독자에게 Next 이벤트가 전달된다
- Behavior Relay는 Behavior Subject와 마찬가지로 하나의 값을 생성자로 전달한다
- Behavior Relay는 value라는 속성을 제공한다
  - Behavior Relay가 저장하고 있는 Next 이벤트에 접근해서 여기에 저장되어 있는 값을 리턴한다 
  - 이 속성은 읽기 전용이고, 이 안에 있는 값을 바꿀 수는 없다
  - 값을 바꾸고 싶다면, accept 메소드를 통해 새로운 넥스트 이벤트를 전달해야 한다



### Create Operators
#### 13/98 just, of, from (0217 여기부터)
- just는 하나의 항목을 방출하는 Observable을 생성한다
<pre>
<code>
let disposeBag = DisposeBag()
let element = "smile"

Observable.just(element)
  .subscribe { event in print(event) }
  .disposed(by: disposeBag)
==> 출력결과
next(smile)
completed

Observable.just([1, 2, 3])
  .subscribe { event in print(event) }
  .disposed(by: disposeBag)
==> 출력결과
next([1, 2, 3])
completed
</code>
</pre>
- just는 ObservableType 프로토콜에 Type 메소드로 선언되어 있다
- 파라미터로 하나의 요소를 받아서 Observable을 리턴한다
- from 연산자와 자주 혼동하게 되는데, just로 생성한 Observable은 파라미터로 전달한 요소를 그대로 방출한다는 사실을 기억하라
- 만약 두 개 이상의 요소를 방출할 Observable을 만들어야 한다면 just로는 불가능하고, of 연산자를 사용해야 한다
- of의 경우 가변 파라미터로 설정되어 있어서 여러 개의 값을 동시에 전달할 수 있다
<pre>
<code>
let disposeBag = DisposeBag()
let apple = "Apple"
let orange = "Orange"
let kiwi = "Kiwi"

Observable.of(apple, orange, kiwi)
  .subscribe { element in print(element) }
  .disposed(by: disposeBag)
==> 출력결과
next(Apple)
next(Orange)
next(Kiwi)
completed

Observable.of([1, 2], [3, 4], [5, 6])
  .subscribe { element in print(element) }
  .disposed(by: disposeBag)
==> 출력결과
next([1, 2])
next([3, 4])
next([5, 6])
completed
</code>
</pre>
- of 역시 ObservableType 프로토콜의 Type 메소드로 선언되어 있다
- 방출할 요소를 원하는 수만큼 전달할 수 있다
- 배열에 저장된 요소를 하나씩 방출하고 싶다면 from 연산자를 사용하면 된다
- from 역시 ObserableType 프로토콜의 Type 메소드로 선언되어 있다
- 첫 번째 파라미터로 배열을 받고, 리턴형은 배열이 아니라 배열에 포함된 요소이다
- 배열에 포함된 요소를 하나씩 순서대로 방출한다
- sequence 형식을 전달할 수도 있다
<pre>
<code>
let disposeBag = DisposeBag()
let fruits = ["Apple", "Kiwi", "Mango"]
Observable.from(fruits)
  .subscribe { element in print(element) }
  .disposed(by: disposeBag)
==> 출력결과
next("Apple")
next("Kiwi")
next("Mango")
completed
</code>
</pre>
- 하나의 요소를 방출하는 Observable을 생성할 때는 just 연산자를 사용한다
- 두 개 이상의 요소를 방출하는 Observable을 생성할 때는 of 연산자를 사용한다
- just와 of 연산자는 항목을 그대로 방출하기 때문에, 배열을 전달하면 배열이 방출된다
- 배열에 저장된 요소를 하나씩 방출하는 Observable이 필요하다면 from 연산자를 사용한다


#### 14/98 range, generate
- 정수를 지정된 수만큼 방출하는 Observable을 생성하기 위해서 range 연산자와 generate 연산자를 사용한다
- 첫 번째 파라미터에는 시작할 정수를 입력한다(실수를 입력하면 컴파일 에러가 발생한다)
- 두 번째 파라미터에는 방출할 정수의 수를 전달한다
<pre>
<code>
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 5)
  .subsribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next(1)
next(2)
next(3)
next(4)
next(5)
completed
</code>
</pre>
- range 연산자는 시작 값에서 1씩 증가하는 squence를 생성한다
- 증가되는 크기를 바꾸거나 감소하는 sequence를 생성하는 것은 불가능하다
- 이걸 가능하게 하는 것이 generate 연산자이다
- generate 연산자는 총 네 개의 파라미터를 받는다(initialState, condition, scheduler, iterate)
- 첫 번째 파라미터인 initialState는 시작값을 전달한다. 즉, 가장 먼저 방출되는 값이 들어간다
- 두 번째 파라미터인 condition에 true를 리턴하는 경우에만 요소가 방출된다, false를 리턴하면 completed 이벤트를 전달하고 바로 종료한다
- 세 번째는 일단 무시
- 네 번째 파라미터인 iterate에는 값을 바꾸는 코드를 전달한다. 보통 값을 증가시키거나 감소시키는 코드를 전달한다
<pre>
<code>
let disposeBag = DisposeBag()

// iterate로 전달하는 식($0+2)은 값을 2씩 증가시키는 것을 의미
Observable.generate(initialState: 0, condition: { $0 <= 10 } , iterate: { $0 + 2 })
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next(0)
next(2)
next(4)
next(6)
next(8)
next(10)
completed

let red = "RED"
let blue = "BLUE"

// 두 번째 파라미터 안의 { $0.count < 5 }는 문자열의 길이가 5보다 작을 때 true를 리턴함을 의미한다
// 세 번째 파라미터 안의 { $0.count.isMultiple(of:2) ? $0 + red : $0 + blue }는 현재 문자열 뒤에 다른 색의 문자열을 추가하도록 구현함을 의미
Observable.generate(initialState: red, condition: { $0.count < 5 }, iterate: { $0.count.isMultiple(of:2) ? $0 + red : $0 + blue })
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next(RED)
next(RED,BLUE)
next(RED,BLUE,RED)
next(RED,BLUE,RED,BLUE)
completed
</code>
</pre>
- generate 연산자의 경우 range 연산자와 다르게 파라미터 형식이 정수로 제한되지 않는다


#### 15/98 repeatElement
- 동일한 요소를 반복적으로 방출하는 Observable을 생성할 수 있도록 해주는 연산자가 repeatElement이다
- repeatElement는 ObservableType 프로토콜의 Type 메소드로 선언되어 있다
- 첫 번째 파라미터로 요소를 전달하면 이 요소를 반복적으로 방출하는 Observable을 리턴한다
- 반복적의 의미는 설명에 나와 있는 대로(infinitely) 무한적으로 방출하는 것을 의미한다
<pre>
<code>
let disposeBag = DisposeBag()
let element = "Heart"
Observable.repeatElement(element)
  .subscribe { print($0) }
==> 출력결과
next(Heart)
next(Heart)
next(Heart)
... 무한반복

Observable.repeatElement(element)
  .take(4)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next("Heart")
next("Heart")
next("Heart")
next("Heart")
completed
</code>
</pre>
- repeatElement 연산자를 사용할 때는 방출되는 횟수를 제한해주는 것이 아주 중요하다
- take 연산자를 사용해서 방출 횟수를 지정할 수 있다

#### 16/98 deferred
- deferred의 뜻은 '연기'임
- deferred 연산자를 활용하면 특정 조건에 맞춰 Observable을 생성할 수 있다
- deferred 연산자는 Observable을 리턴하는 클로저를 파라미터로 받는다
<pre>
<code>
let disposeBag = DisposeBag()
let animals = ["Dog", "Cat", "Fox"]
let fruits = ["Apple", "Mango", "Kiwi"]
var flag = true

let factory: Observable<String> = Observable.deferred {
  flag.toggle() // flag의 상태를 뒤집는 역할, 기존에 flag가 true였기 때문에 toggle 이후에는 false가 된다
  // flag에 true가 저장되어 있다면 animals 배열에 있는 문자열을 방출하는 Observable을 리턴
  if flag {
    return Observable.from(animals) // from 연산자를 사용하면 배열 내 요소들을 순차적으로 방출할 수 있다
  } else { // flag에 false가 저장되어 있다면 fruits 배열에 있는 문자열을 방출하는 Observable을 리턴
    return Observable.from(fruits) 
  }
}

factory
  .subscribe { print($0) }
  .dispose(by: disposeBag)
// 한 번 더 호출해서 flag 값을 다시 true로 변경
factory
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next("Apple")
next("Mango")
next("Kiwi")
completed
next("Dog")
next("Cat")
next("Fox")
completed
</code>
</pre>

#### 17/98 create
- Observable이 동작하는 방식을 직접 구현하고 싶다면 create 연산자를 사용해야 한다
<pre>
<code>
let disposeBag = DisposeBag()
enum MyError: Error {
  case error
}

Observable<String>.create { (observer) -> Disposable in
  guard let url = URL(string: "https://www.apple.com" 
    else { 
    // 여기서의 옵저버는 클로저로 전달된 파라미터를 의미함
    observer.onError(MyError.error)
    return Disposables.create() // Disposable이 아니라 Disposables로 써야 한다
  }
  
  guard let html = try? String(contentsOf: url, encoding: .utf8) else {
    observer.onError(MyError.error)
    return Disposables.create()
  }
  // 문자열이 정상적으로 저장되었다면 Observer로 전달한다 -> 문자열을 방출한다
  observer.onNext(html) // 요소를 방출할 때는 onNext를 사용한다
  observer.onCompleted() // observer로 completed 이벤트가 전달된다
  
  return Disposables.create() // 마지막으로 Disposable을 생성해서 리턴해주면 모든 리소스가 정리되고 Observable이 정상적으로 종료된다
}
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
(html 내용이 출력됨)
</code>
</pre>
- Observable을 종료하기 위해서는 onError 또는 onCompleted 메소드를 반드시 호출해야 한다
- 둘 중 하나라도 호출하면 Observable이 종료되기 때문에, 그 이후에 onNext를 호출하면 요소가 방출되지 않는다
- onNext를 호출하려면 onCompleted() 메소드 또는 onError() 전에 호출해야 한다



#### 18/98 empty, error(2020/02/18 여기까지)
- 두 연산자가 생성한 Observable은 next 이벤트를 전달하지 않는다는 공통점이 있다
- 둘 다 어떠한 요소도 방출하지 않는다
- empty 연산자는 completed 이벤트를 전달하는 Observable을 생성한다 
- 요소를 방출하지 않기 때문에 요소의 형식은 중요하지 않다
- empty 연산자는 파라미터가 없다
- observer가 아무런 동작 없이 종료돼야 할 때 사용한다
<pre>
<code>
let disposeBag = DisposeBag()

Observable<Void>.empty()
    .subscribe { print($0) }
    .disposed(by: disposeBag)
==> 출력결과
completed
</code>
</pre>

- error 연산자는 error 이벤트를 전달하고 종료한다
- error 연산자는 파라미터로 error를 받는다
<pre>
<code>
let disposeBag = DisposeBag()
enum MyError: Error {
  case error
}

Observable<Void>.error(MyError.error)
    .subscribe { print($0) }
    .disposed(by: disposeBag)
==> 출력결과
error(error)
</code>
</pre>

### Filtering Operators
#### 19/98 ignoreElementsOperator (2020/02/19 여기부터)
- ignoreElements는 Observable이 방출하는 next 이벤트를 필터링하고 completed 이벤트와 error 이벤트만 구독자로 전달한다
- ignoreElements는 파라미터를 받지 않는다
- 리턴형은 Completable인데, Completable은 트레이츠라고 부르는 특별한 Observable이다
- Completable은 completed 또는 error 이벤트만 전달하고 next 이벤트는 무시한다
- 주로 작업의 성공과 실패에만 관심이 있을 때 사용한다
<pre>
<code>
let disposeBag = DisposeBag()
let fruits = ["Apple", "Pear", "Grape"]

Observable.from(fruits)
  .ignoreElements()
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
completed
</code>
</pre>

#### 20/98 elementAt Operator
- elementAt은 특정 인덱스에 위치한 요소를 제한적으로 방출한다
- elementAt은 정수 인덱스를 파라미터로 받아서 Observable을 리턴한다
- 해당 인덱스의 요소를 방출하고 completed 이벤트를 전달받는다
<pre>
<code>
let disposeBag = DisposeBag()
let fruits = ["Apple", "Pear", "Grape"]

Observable.from(fruits)
  .elementAt(1)
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(Pear)
completed
</code>
</pre>


#### 21/98 filter Operator
- filter 연산자는 클로저를 파라미터로 받는다
- 클로저는 predicate로 사용된다 (predicate는 '서술하다', '단언하다', '주장하다', '속성'의 뜻을 가짐)
- true를 리턴하는 요소가 연산자가 리턴하는 Observable에 포함된다
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .filter { $0.isMultiple(of: 2) }
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(2)
next(4)
next(6)
next(8)
next(10)
completed
</code>
</pre>


#### 22/98 skip, skipWhile, skipUntil Operator
- skip 연산자를 통해 특정 요소를 무시할 수 있다
- skip 연산자는 정수를 파라미터로 받는다
- Observable이 방출하는 요소 중에서 지정된 수만큼 무시한 다음에 이후에 방출되는 요소만 구독자로 전달한다
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .skip(3) // 인덱스가 아니라 count로 사용되는 것이다 -> 인덱스였으면 5부터 출력됐을 것이다
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(4)
next(5)
next(6)
next(7)
next(8)
next(9)
next(10)
completed
</code>
</pre>

- skipWhile은 클로저를 파라미터로 받는다 
- 이 클로저는 filter 연산자와 마찬가지로 predicate로 사용되고, 클로저에서 true를 리턴하는 동안 방출되는 요소를 무시한다
- 클로저에서 false를 리턴하면 그때부터 요소를 방출하고, 이후에는 조건에 관계 없이 모든 요소를 방출한다
- 연산자는 방출되는 요소를 포함한 Observable을 리턴한다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .skipWhile { !$0.isMultiple(of: 2) } // skipWhile 연산자에 2가 전달되면 false가 리턴되고 이때부터 이어지는 모든 요소가 방출된다(구독자로 전달된다)
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(2)
next(3)
next(4)
next(5)
next(6)
next(7)
next(8)
next(9)
next(10)
completed
</code>
</pre>

- skipUntil 연산자는 Observable 타입을 파라미터로 받는다
- 다른 Observable을 파라미터로 받고, 이 Observable이 next 이벤트를 전달하기 전까지, 원본 Observable이 전달하는 이벤트를 무시한다
- 이런 특징 때문에 파라미터로 전달되는 Observable을 trigger라고 부르기도 한다

<pre>
<code>
let disposeBag = DisposeBag()

let subject = PublishSubject<Int>()
let trigger = PublishSubject<Int>()

subject.skipUntil(trigger)
  .subscribe { print($0) }
  .dispose(by: disposeBag)
  
subject.onNext(1)
// 아직 trigger가 요소를 방출한적이 없기 때문에 subject가 방출한 요소는 구독자로 전달되지 않는다

trigger.onNext(0)
// 이번에는 trigger에서 요소를 방출하고 있다. 그런데 subject가 이전에 방출했던 요소는 여전히 구독자로 전달되지 않는다
// skipUntil은 trigger가 요소를 방출한 이후부터 원본 Observable에서 방출되는 요소들을 구독자로 전달한다

subject.onNext(2)

==> 출력결과
next(2)
completed
</code>
</pre>



#### 23/98 take, takeWhile, takeUntil, takeLast Operator
- take 연산자를 통해 요소의 방출 조건을 다양하게 구성할 수 있다
- take 연산자는 네 가지 형태가 있다(take, takeWhile, takeUntil, takeLast)
- take 연산자는 정수를 파라미터로 받아서 해당 숫자만큼만 요소를 방출한다
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .take(5)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
==> 출력결과
next(1)
next(2)
next(3)
next(4)
next(5)
completed
</code>
</pre>

- takeWhile 연산자는 클로저를 파라미터로 받아서 predicate로 사용한다
- true를 리턴하면 구독자에게 전달된다 -> 요소를 방출한다
- 연산자가 리턴하는 Observable에는 최종적으로 조건을 만족하는 요소들만 포함된다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .takeWhile { !$0.isMultiple(of: 2) }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
==> 출력결과
next(1) // 이후에도 홀수가 방출되지만 구독자로는 전달되지 않는다 (takeWhile 연산자는 클로저가 false를 리턴하면 더 이상 요소를 방출하지 않는다)
completed
</code>
</pre>

- takeUntil 연산자
- takeUntil 연산자는 ObservableType을 파라미터로 받는다
- Observable을 파라미터로 받는다는 의미이다
- 파라미터로 전달한 Observable에서 next 이벤트를 전달하기 전까지 원본 Observable에서 next 이벤트를 전달한다
- trigger 상수에 저장된 Observable을 파라미터로 전달하는 예시
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let subject = PublishSubject<Int>()
let trigger = PublishSubject<Int>()

subject.takeUntil(trigger)
  .subscribe { print($0) }
  .dispose(by: disposeBag)

subject.onNext(1)
subject.onNext(2)

trigger.onNext(0) // 이때부터 요소가 방출되지 않는다

subject.onNext(3)
==> 출력결과
next(1)
next(2)
completed
</code>
</pre>

- takeLast 연산자는 정수를 파라미터로 받아서 Observable을 리턴한다
- 리턴되는 Observable에는 원본 Observable이 방출한 요소 중에서 마지막으로 방출한 n개의 요소가 포함된다
- takeLast 연산자의 가장 중요한 점은 구독자로 전달되는 시점이 딜레이 된다는 점이다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let subject = PublishSubject<Int>()
subject.takeLast(2)
    .subscribe { print($0) }
    .dispose(by: disposeBag)

numbers.forEach { subject.onNext($0) }

subject.onNext(11) // 끝에 두 요소가 10, 11로 바뀐다
subject.onCompleted()

==> 출력결과
next(10)
next(11)
completed
</code>
</pre>


#### 24/98 single Operator
- single 연산자는 원본 Observable에서 첫 번째 요소만 방출하거나, 조건과 일치하는 첫 번째 요소만 방출한다
- 두 개 이상의 요소가 방출되는 경우 error가 발생한다
- single 연산자는 단 하나의 요소만 방출해야 정상적으로 종료된다
- 원본 Observable이 아무 요소도 방출하지 않거나, 두 개 이상의 요소를 방출하는 경우 error가 발생한다
- single 연산자는 아무 파라미터가 없는 연산자와, predicate를 파라미터로 받은 연산자 두 경우가 있다
- single 연산자는 새로운 요소를 방출하면 구독자에게 바로 전달된다
- 다른 요소가 방출될 수도 있으므로 single 연산자가 리턴하는 Observable은 원본 Observable에서 completed 이벤트가 전달할 때까지 대기한다
- completed 이벤트가 전달되는 시점에 하나의 요소만 방출된 시점이라면 구독자에게 completed 이벤트가 전달되고, 그 사이에 다른 요소가 방출되었다면 구독자에게는 error 이벤트가 전달된다. 이와 같은 방식을 통해 하나의 요소만 방출되는 것을 보장받을 수 있다.
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.just(1)
  .single()
  .subscribe { print($0) }
  .dispose(by: disposeBag)
 
Observable.form(numbers)
  .single()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

Observable.from(numbers)
  .single { $0 == 3 }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
let subject = PublishSubject<Int>()
  
subject.single()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

subject.onNext(100)

==> 출력결과
next(1)
completed
next(1)
error(Sequence contains more than one element.)
next(3)
completed
completed
next(100)
</code>
</pre>


#### 25/98 distinctUntilChange Operator
- distinctUntilChange 연산자는 동일한 항목이 연속적으로 방출되지 않도록 필터링 해준다
- 이 연산자는 파라미터가 없다.
- 원본 Observable에서 전달되는 두 개의 요소를 순서대로 비교한 다음에 이전 요소와 동일하면 방출하지 않는다
- 두 개의 요소를 비교할 때는 비교 연산자를 활용해 비교한다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 1, 3, 2, 2, 3, 1, 5, 5, 7, 7, 7]

Observable.from(numbers)
  .distinctUntilChanged()
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(1)
next(3)
next(2)
next(3)
next(1)
next(5)
next(7)
completed
</code>
</pre>

2020/02/19 여기까지

#### 26/98 debounce, throttle Operator (2020/02/20 여기부터)
- 두 연산자는 짧은 시간동안 반복적으로 방출되는 이벤트를 제어한다는 공통점이 있다
- 연산자로 전달하는 파라미터도 동일하다
- 하지만 연산의 결과는 완전히 다르다
- debounce 연산자는 두 개의 파라미터를 받는다
  - 첫 번째 파라미터(dueTime: RxTimeInterval)에는 시간을 전달한다
  - 이 시간은 연산자가 next 이벤트를 방출할지 결정하는 조건으로 사용된다
  - Observer가 next 이벤트를 방출한 다음 지정된 시간 동안 다른 next 이벤트를 방출하지 않는다면 해당 시점에 가장 마지막으로 방출한 next 이벤트를 구독자에게 전달한다
  - 반대로 지정된 시간 이내에 또 다른 next 이벤트를 방출했다면 타이머를 초기화한다 (이해하는 것이 중요한 부분)
  - 타이머를 초기화한 다음에 다시 지정된 시간 동안 대기한다
  - 이 시간 이내에 다른 이벤트가 방출되지 않는다면 마지막 이벤트를 방출하고 이벤트가 방출된다면 타이머를 다시 초기화한다
  - 두 번째 파라미터(scheduler: SchedulerType)에는 타이머를 실행할 스케줄러를 전달한다

<pre>
<code>
let disposeBag = DisposeBag()
let buttonTap = Observable<String>.create { observer in
  DispatchQueue.global().async {
    for i in 1...10 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.3) // 0.3초 주기로 10번 방출
    }
    Thread.sleep(forTimeInterval: 1) // 1초 동안 쓰레드 중지
    
    for i in 11...20 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.5) // 0.5초 주기로 10번 방출
    }
    
    observer.onCompleted()
  }
  
  return Disposables.create {
  
  }
}

buttonTap
  .debounce(.milliseconds(1000), scheduler: MainScheduler.instance) // 1초의 debounce
  .subscribe { print($0) }
  .disposed(by: disposeBag)  
==> 출력결과
next(Tap 10)
next(Tap 20)
completed
// debounce 연산자는 지정된 시간 동안 새로운 이벤트가 방출되지 않으면 가장 마지막에 방출된 이벤트를 구독자에게 전달한다 -> 타이머가 만료되기 전에 새로운 이벤트가 방출되었기 때문에 타이머를 초기화 함
</code>
</pre>


- throttle 연산자 (스로틀이라는 용어는 엔진의 힘이나 속도가 규제되는 구조라는 의미)
- throttle 연산자는 세 개의 파라미터(dueTime: RxTimeInterval, latest: Bool, scheduler: SchedulerType)를 받는다
- 기본값을 가진 두 번째 파라미터는 생략하는 경우가 많기 때문에 debounce와 파라미터가 동일하다고 생각해도 무방하다
- 첫 번째 파라미터에는 반복 주기를 전달하고, 세 번째 파라미터에는 스케줄러를 전달한다
- throttle은 지정된 주기 동안 하나의 이벤트만 구독자에게 전달한다
- 보통 두 번째 파라미터는 기본값을 사용하는데, 주기를 엄격하게 지킨다
- 두 번째 파라미터에 false를 부여하면, 반복 주기가 경과한 다음 가장 먼저 방출되는 이벤트를 구독자에게 전달한다

<pre>
<code>
let disposeBag = DisposeBag()
let buttonTap = Observable<String>.create { observer in
  DispatchQueue.global().async {
    for i in 1...10 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.3)
    }
    Thread.sleep(forTimeInterval: 1)
    for i in 11...20 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.5)
    }
    observer.onCompleted()
  }
  return Disposables.create()
}

buttonTap
  .throttle(.milliseconds(1000), scheduler: MainScheduler.instance)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력 결과
next(Tap 1)
next(Tap 4)
next(Tap 7)
next(Tap 10)
next(Tap 11)
next(Tap 12)
next(Tap 14)
next(Tap 16)
next(Tap 18)
next(Tap 20)
completed
</code>
</pre>

- throttle 연산자는 next 이벤트를 지정된 주기마다 하나씩 구독자에게 전달한다
- 반면 debounce 연산자는 next 이벤트가 전달된 다음 지정된 시간이 경과하기까지 다른 이벤트가 전달되지 않는다면 마지막으로 방출된 이벤트를 구독자에게 전달한다
- 짧은 시간 동안 반복되는 tap 이벤트나 delegate 이벤트를 처리할 때는 throttle을 사용하고, debounce는 주로 검색 기능을 구현할 때 사용한다
- debounce를 활용해 사용자가 짧은 시간동안 연속적으로 타이핑을 할 때는 검색작업을 실행하지 않다가, 타이핑을 멈추면 검색을 실행한다
- toArray 연산자는 별도의 파라미터를 받지는 않는다
- 하나의 요소를 방출하거나 error 이벤트를 방출하는 ObservableType의 메소드이다
- 하나의 요소를 방출하고 바로 종료한다

### Tranforming Operators
#### 27/98 toArray Operator
- toArray 연산자는 Observable이 방출하는 모든 요소를 배열에 담은 다음, 이 배열을 방출하는 Observable을 생성한다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let subject = PublishSubject<Int>()
  
subject
  .toArray()
  .subscribe { print($0) }
  .disposed(by: dispseBag)
  
subject.onNext(1)
subject.onNext(2)
subject.onCompleted()
==> 출력결과
success([1,2])
</code>
</pre>


#### 28/98 map Operator (2020/02/21 여기부터)
- map 연산자는 Observable 배출하는 항목을 대상으로 함수를 실행하고 결과를 새로운 Observable로 배출한다
- map 연산자를 사용하다보면 파라미터와 동일한 형식을 리턴해야 한다고 생각하는 경우가 많지만 그런 제약은 없다
- map 연산자는 Observable이 방출하는 요소들을 대상으로 클로저를 실행하고 그 결과를 구독자에게 전달한다
- 클로저로 전달되는 파라미터의 형식은 소스 Observable이 방출하는 요소와 동일하다
- 하지만 클로저가 리턴하는 값의 방식은 고정되어 있지 않으며, 원하는 형식으로 리턴할 수 있다.
 
<pre>
<code>
let disposeBag = DisposeBag()
let skills = ["Siwft", "SwiftUI", "RxSwift"]

Observable.from(skills)
//  .map { "Hello, \($0)"}
  .map { $0.count }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
==> 출력결과
// next(Hello, Swift)
// next(Hello, SwiftUI)
// next(Hello, RxSwift)
next(5)
next(7)
next(7)
completed
</code>
</pre>


#### 29/98 flatMap Operator
- flatMap 연산자는 모든 Observable이 방출하는 항목을 모아서 최종적으로 하나의 Observable을 리턴한다
- 개별 항목이 개별 Observable로 변환되었다가 다시 하나의 Observable로 합쳐지기 때문에 처음에는 이해하기 어렵다
- 실제 프로젝트에서 활용해보면 금방 이해 된다
- flatMap 연산자는 클로저를 파라미터로 받는데, BehaviorSubject를 원하는대로 변환한 다음 새로운 Observable을 리턴해야 한다
- flatMap이 내부적으로 여러 개의 Observable을 생성하지만, 최종적으로 모든 Observable이 하나의 Observable로 합쳐지고, 방출되는 항목들이 순서대로 구독자에게 전달된다
- flatMap 연산자는 원본 Observable이 방출하는 항목을 새로운 Observable로 변환한다. 새로운 Observable은 항목이 업데이트 될 때마다 새로운 항목을 방출한다
- 이렇게 생성된 모든 Observable은 최종적으로 하나의 Observable로 합쳐지고, 모든 항목들이 이 Observable을 통해서 구독자로 전달 된다
- 단순히 처음에 방출된 항목만 구독자로 전달되는 것이 아니라 업데이트된 최신 항목도 구독자로 전달된다
- 이 연산자는 네트워크 요청을 구현할 때 자주 활용한다
<pre>
<code>
let disposeBag = DisposeBag()

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()
  
subject
  .flatMap { $0.asObservable() } // subject를 observable로 변경시킨다
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
subject.onNext(a)
subject.onNext(b)

a.onNext(11)
b.onNext(22)

==> 출력결과
next(1)
next(2)
next(11)
next(22)

</code>
</pre>


#### 30/98 flatMapFirst, flatMapLatest Operator (2020/02/22 여기부터)
- flatMap 연산자에서 파생된 연산자들이다
- flatMap 연산자에 대한 이해가 우선돼야 한다
- flatMapFirst의 연산자, 리턴형은 flatMap과 동일하다
- 하지만 연산자가 리턴하는 Observable에는 처음에 변환된 Observable이 방출하는 항목만 포함된다

<pre>
<code>
let disposeBag = DisposeBag()

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()
  
subject
  .flatMapFirst { $0.asObservable() }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
subject.onNext(a)
subject.onNext(b)

a.onNext(11)
b.onNext(22)
b.onNext(222)
a.onNext(111)

==> 출력결과
next(1) 
next(11)
next(111) // a에 대한 내용만 출력된다

</code>
</pre>

- flatMapLatest는 원본 Observable이 방출하는 항목을 Observable로 변환하는 것은 동일하다
- 모든 Observable이 방출하는 항목을 하나로 병합하지 않는다. 대신 가장 최근의 항목을 방출한 Observable을 제외한 나머지는 모두 무시한다

<pre>
<code>
let disposeBag = DisposeBag()

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()
  
subject
  .flatMapLatest { $0.asObservable() }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
subject.onNext(a)
a.onNext(11)
subject.onNext(b)
b.onNext(22)
a.onNext(11)

==> 출력결과
next(1) 
next(11)
next(2)
next(22) 
// a.onNext(11)은 구독자로 전달되지 않아 출력되지 않는다

</code>
</pre>

- flatMapLatest는 원본 Observable이 방출하는 요소를 새로운 Observable로 변환하고 가장 최근에 변환된 Observable이 방출하는 요소만 구독자에게 전달한다

#### 31/98 scan Operator
- scan 연산자는 accumulator function을 활용한다 (accumulator는 누산기, 축압기 등을 의미한다)
- 이 연산자는 기본값으로 연산을 시작하고, 원본 Observable이 방출하는 항목을 대상으로 변환을 실행한 다음 결과를 방출하는 하나의 Observable을 리턴한다
- 원본이 방출하는 항목의 수와 구독자로 전달되는 항목의 수가 동일하다
- 첫 번째 파라미터로 기본값을 전달하고, 두 번째 파라미터에는 클로저(두 개의 파라미터)를 전달한다
- 이 연산자는 작업 결과를 누적 시키면서 중간 결과와 최종 결과가 모두 필요한 경우에 사용한다

<pre>
<code>
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 10)
  .scan(0, accumulator: +)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next(1)
next(3) // 1 + 2
next(6) // 3 + 3
next(10) // 6 + 4
next(15) // 10 + 5
next(21) // 15 + 6
next(28) // 21 + 7
next(36) // 28 + 8
next(45) // 36 + 9
next(55) // 45 + 10
completed
</code>
</pre>


#### 32/98 buffer Operator
- buffer 연산자는 특정 주기 동안 옵저버블이 방출하는 항목을 수집하고 하나의 배열로 리턴한다
- RxSwift에서는 이런 동작을 컨트롤드 버퍼링이라고 한다
- 세 개의 파라미터(timeSpan: 항목을 수집할 시간(DispatchTimeInterval), count: 수집할 항목의 최대 숫자(Int), scheduler: SchedulerType)
- 연산자의 리턴형은 Type 파라미터가 배열로 선언되어 있다
- 지정된 시간동안 수집한 항목들을 배열에 담아서 리턴한다

<pre>
<code>
let disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .buffer(timeSpan: .seconds(2), count: 3, scheduler: MainScheduler.instance)
  .take(5) // 5개만 방출
  .subscribe { print($0) }
  .disposed(by: disposerBag)
==> 출력 결과
next([0])
next([1, 2, 3])  
next([4, 5])
next([6, 7])
next([8, 9])
completed  
// Observable은 1초마다 항목을 방출하고 있고, buffer 연산자는 2초마다 세 개씩 수집하고 있다
// buffer 연산자는 첫 번째 파라미터로 전달한 timeSpan이 경과하면 수집된 항목들을 즉시 방출한다
// 두 번째 파라미터로 지정한 수만큼 수집되지 않았더라도 즉시 방출한다 (0만 출력되거나 1, 2, 3 세 개가 출력되는 경우)
</code>
</pre>



#### 33/98 window Operator
- window 연산자는 버퍼 연산자처럼 timeSpan과 maxCount를 지정해서 원본 Observable이 방출하는 항목들을 작은 단위의 Observable로 분해한다
- buffer와 달리 window 연산자는 수집된 항목을 방출하는 Observable을 리턴한다
- 리턴된 Observable이 무엇을 방출하고, 언제 방출하는지 이해하는 것이 중요하다
- 파라미터는 buffer와 똑같이 첫 번째로는 timeSpan, 두 번째로는 count, 세 번째로는 scheduler를 전달한다
- buffer 연산자와의 차이는 리턴형에 있다. buffer는 수집된 배열을 방출하는 Observable을 리턴, window 연산자는 Observable을 방출하는 Observable을 리턴
- Observable이 방출하는 Observable을 inner Observable이라고 한다
- inner Observable은 지정된 최대항목수만큼 방출하거나 지정된 시간이 경과하면 completed 이벤트를 전달하고 종료한다

<pre>
<code>
let disposeBag = DisposeBag()
Observable<Int>.interval(.seconds(1), scheduler: MainSchedulerType)
  .window(timeSpan: .seconds(5), count: 3, scheduler: MainScheduler.instance)
  .take(5)
  .subcribe { 
    print($0) 
    if let observable = $0.element {
      observable.subscribe { print(" inner: ", $0) }
    }
  }
  .disposed(by: disposeBag)
==> 출력결과
next(RxSwift.AddRef<Swift.Int>) // AddRef는 특별한 형태의 Observable이다. inner Observable이 바로 AddRef Observable이다. 구체적으로 공부할 필요는 없다. 구독할 수 있다는 걸 이해해라
  inner: next(0)
  inner: next(1)
  inner: next(2)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(0)
  inner: next(1)
  inner: next(2)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(3)
  inner: next(4)
  inner: next(5)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(6)
  inner: next(7)
  inner: next(8)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(9)
  inner: next(10)
  inner: next(11)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(12)
  inner: next(13)
  inner: next(14)
  inner: completed
completed
</code>
</pre>


#### 34/98 groupBy Operator
- groupBy 연산자는 Observable이 방출하는 요소를 원하는 기준으로 그루핑할 때 사용한다
- 파라미터로 클로저로 받고, 클로저는 요소를 파라미터로 받아서 키를 리턴한다
- 키의 형식은 hasable 프로토콜을 채용한 형식으로 한정되어 있다
- 연산자를 실행하면 클로저에서 동일한 값을 리턴하는 요소끼리 그룹으로 묶이고 그룹에 속한 요소들은 개별 Observable을 통해 방출된다
- 연산자가 리턴하는 Observable을 보면 TypeParameter가 grouped Observable로 선언되어 있다
- 방출하는 요소와 함께 키가 저장되어 있다


<pre>
<code>
let disposeBag = DisposerBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
  .groupBy { $0.count }
  .subscribe(onNext: { groupedObservable in
    print("== \(groupedObservable.key)")
    groupedObservable.subscribe { print(" \($0)") }
  })
  .disposed(by: disposeBag)
==> 출력결과
// next(GroupedObservable<Int, String>(key: 5, source: RxSwift.(unkown context at $115767838) .GroupedObservableImpl<Swift.String>))
// next(GroupedObservable<Int, String>(key: 6, source: RxSwift.(unkown context at $115767838) .GroupedObservableImpl<Swift.String>))
// next(GroupedObservable<Int, String>(key: 4, source: RxSwift.(unkown context at $115767838) .GroupedObservableImpl<Swift.String>))
// next(GroupedObservable<Int, String>(key: 3, source: RxSwift.(unkown context at $115767838) .GroupedObservableImpl<Swift.String>)) 
== 5
  next(Apple)
== 6
  next(Banana)
  next(Orange)
== 4
  next(Book)
  next(City)
== 3
  next(Axe)
  completed
  completed
  completed
  completed

</code>
</pre>

- groupBy 연산자를 활용할 때는 보통 flatMap 연산자와 toArray 연산자를 활용해서 그루핑된 최종 결과를 하나의 배열로 방출하도록 구현한다

<pre>
<code>
let disposeBag = DisposerBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
//  .groupBy { $0.count }
  .groupBy { $0.first ?? Character(" ) }
  .flatMap { $0.toArray() }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
// next(["Book", "City"])
// next(["Axe"])
// next(["Apple"])
// next(["Banana", "Orange"])
next(["Banana", "Book"])
next(["City"])
next(["Apple", "Axe"])
next(["Orange"])
completed
</code>
</pre>

// 2020/02/22 여기까지

// 2020/02/23 여기부터

### Combining Operators
#### 35/98 startWith Operator
- Observable squence 앞에 새로운 요소를 추가하는 startWith 연산자
- Observable이 요소를 방출하기 전에 다른 항목들을 앞 부분에 추가한다
- 주로 기본값이나 시작값을 지정할 때 활용한다
- 파라미터는 가변 파라미터이다 -> 파라미터로 전달하는 하나 이상의 값을 Observable sequence 앞 부분에 추가한다. 그 다음 새로운 Observable을 리턴한다
- last in, first out의 특성이 있다
<pre>
<code>
let bag = DisposeBag()
let numbers = [1, 2, 3, 4, 5]

Observable.from(numbers)
  .startWith(0)
  .startWith(-1, -2)
  .startWith(-3)
  .subscribe { print($0) }
  .disposed(by: bag)
  
--> 출력결과
next(-3)
next(-1)
next(-2)
next(0)
next(1)
next(2)
next(3)
next(4)
next(5)
completed
</code>
</pre>



#### 36/98 concat Operator
- 두 개의 Observable을 연결하는 concat 연산자
- concat 연산자는 Type 메소드와 Instance 메소드로 구성돼 있다
- Type 메소드로 구현된 concat 연산자는 파라미터로 전달된 콜렉션에 있는 모든 Observable을 순서대로 연결한 하나의 Observable을 리턴한다
- Instance 메소드로 구현된 concat 연산자는 대상 Observable이 completed 이벤트를 전달한 경우에 파라미터로 전달한 Observable을 연결한다
- 만약 error 이벤트가 전달된다면 Observable은 연결되지 않는다
- 대상 Observable이 방출하는 요소만 전달되고, error 이벤트가 전달된 다음에 종료된다 -> 이건 Type 메소드로 구현된 concat 연산자도 마찬가지
- concat 연산자는 두 Observable을 단순하게 연결한다 -> 연결된 모든 Observable이 방출하는 요소들이 방출 순서대로 정렬되지는 않는다
- 이전 Observable이 모든 요소들을 방출하고 completed 이벤트를 전달해야 이어진 Observable이 방출을 시작한다


<pre>
<code>
let bag = DisposeBag()
let fruits = Observable.from(["Apple", "Kiwi", "Peach"])
let animals = Observable.from(["Dog", "Mouse", "Monkey"])
Observable.concat([fruits, animals])
  .subscribe { print($0) }
  .disposed(by: bag)
  
animals.concat(animals)
  .subscribe { print($0) }
  .disposed(by: bag)
--> 출력결과
// next("Apple")
// next("Kiwi")
// next("Peach")
// next("Dog")
// next("Mouse")
// next("Monkey")
// completed // completed는 연결된 옵저버블이 모든 요소를 방출한 후에 전달된다
next("Dog")
next("Mouse")
next("Monkey")
next("Apple")
next("Kiwi")
next("Peach")
completed
</code>
</pre>


#### 37/98 merge Operator
- 여러 Observable이 방출하는 이벤트를 하나의 Observable에서 방출하도록 병합하는 merge 연산자
- concat 연산자와 혼동하기 쉽지만 동작 방식이 다르다
- concat은 하나의 Observable이 모든 요소를 방출하고 completed 이벤트를 전달하면 이어지는 Observable을 연결
- merge는 두 개 이상의 Observable을 병합하고 모든 Observable에서 방출하는 요소들을 순서대로 방출하는 Observable을 리턴한다
- merge는 두 개 이상의 Observable이 방출하는 요소들을 병합한 하나의 Observable을 리턴한다
- 단순히 뒤에 연결하는 것이 아니라 하나의 Observable로 합쳐준다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let oddNumbers = BehaviorSubject(value: 1)
let evenNumbers = BehaviorSubject(value: 2)
let negativeNumbers = BehaviorSubject(value: -1)

let source = Observable.of(oddNumbers, evenNumbers, negativeNumbers)

source
  .merge(maxConcureent: 2) // 이렇게 하면 병합 가능한 Observable의 수가 2로 제한된다
  .subscribe { print($0) }
  .disposed(by: bag)

oddNumbers.onNext(3)
evenNumbers.onNext(4)

evenNumbers.onNext(6)
oddNumbers.onNext(5)

// oddNumbers.onCompleted() // oddNumbers는 이벤트 못 받는다
// oddNumbers.onError(MyError.error)
// evenNumbers.onNext(8)
// evenNumbers.onCompleted() // evenNumbers도 이벤트 못 받는다

negativeNumbers.onNext(-2) // maxConcurrent가 2이기 때문에 negativeNumber는 병합되지 않는다
oddNumbers.onCompleted()
--> 출력결과
next(1)
next(2)
next(3)
next(4)
next(6)
next(5)
// error(error) -> 8 전에 error가 전달 되었으므로 종료됨
//next(8)
// completed
next(-2) // oddNumbers에서 completed 이벤트를 전달하면 병합대상에서 oddNumbers가 제외되고, queue에 저장되어 있던 negativeNumbers가 병합대상에 추가된다(maxConcurrent는 2니까)
</code>
</pre>



#### 38/98 combineLatest Operator
- 소스 Observable이 방출하는 최신 요소를 병합하는 combineLatest 연산자
- combine은 결합한다는 의미이다
- 소스 Observable을 결합한 다음 파라미터로 전달한 함수를 실행하고 결과를 방출하는 새로운 Observable을 리턴한다
- 핵심은 연산자가 리턴한 Observable이 언제 이벤트를 방출하는지 이해하는 것이다
- 두 개의 Observable과 클로저를 파라미터를 받는다
- Observable이 Next 이벤트를 통해 전달하는 요소들은 클로저 파라미터를 통해 클로저에 전달된다
- 이후 클로저는 실행 결과를 리턴하고 연산자는 최종적으로 이 결과를 방출하는 Observable을 리턴한다
- 다양한 오버로딩이 있는데, 클로저를 전달하지 않는 경우에는 리턴 타입이 달라진다
- 파라미터로 전달한 Observable이 방출하는 요소들을 하나의 tuple로 합친 다음, 이 tuple을 방출하는 Observable을 리턴한다
- Observable을 최대 여덟개까지 전달할 수 있는 연산자들이 선언되어 있다
- 파라미터의 수만 다르고 동작 방식은 동일하다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let greetings = PublishSubject<String>()
let languages = PublishSubject<String>()
  
Observable.combineLatest(greetings, languages) { lhs, rhs
  -> String in
  return "\(lhs), \(rhs)"
}
  .subscribe { print($0) }
  .disposed(by: bag)
  
greetings.onNext("Hi")
languages.onNext("World!")

greetings.onNext("Hello")
languages.onNext("RxSwift")

// greetings.onCompleted()
greetings.onError(MyError.error)
languages.onNext("SwiftUI")

languages.onCompleted()

--> 출력결과
next(Hi World!)
next(Hello World!)
next(Hello RxSwift)
error(error) // 소스 Observable 중 error가 하나라도 전달되면 그 즉시 구독자에게 error 이벤트를 전달하고 종료한다
// next(Hello SwiftUI)
// completed

</code>
</pre>


#### 39/98 zip Operator
- Indexed Sequencing을 구현하는 zip 연산자
- combineLatest와 비교해서 이해하면 쉽게 이해할 수 있다
- zip 연산자는 소스 Observable이 방출하는 요소를 결합한다
- Observable을 결합하고 클로저를 실행한 다음 이 결과를 방출하는 result Observable을 리턴한다
- 집 연산자는 클로저에게 중복되는 요소를 전달하지 않고, index가 일치하는 짝을 전달한다
- 첫 번째 요소는 첫 번째 요소와 결합하고, 두 번째 요소는 두 번째 요소와 결합한다
- 소스 Observable이 방출하는 요소들을 순서를 일치시켜 결합하는 것을 Indexed Sequencing이라고 한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let numbers = PublishSubject<Int>()
let strings = PublishSubjct<String>()
  
Observable.zip(numbers, strings) { "\($0) - \($1)"}
  .subscribe { print($0) }
  .disposed(by: bag)
  
numbers.onNext(1)
strings.onNext("one")

numbers.onNext(2)
strings.onNext("two")

// numbers.onCompleted()
numbers.onError(MyError.error)

strings.onNext("three") // 짝이 없으므로 출력되지 않는다
strings.onCompleted()
--> 출력결과  
next(1 - one)
next(2 - two)
error(error) // 소스 Observable 중에서 하나라도 error 이벤트를 전달하면 즉시 구독자에게 error 이벤트가 전달되고 종료된다
// completed
</code>
</pre>


#### 40/98 withLatestFrom Operator
- trigger Observable이 Next 이벤트를 방출하면 data Observable이 가장 최근에 방출한 Next 이벤트를 구독자에게 전달하는 withLastestFrom 연산자
- 연산자를 호출하는 Observable을 trigger Observable이라고 부르고
- 파라미터로 전달하는 Observable을 data Observable이라고 부른다
- trigger Observable이 Next 이벤트를 방출하면 data Observable이 가장 최근에 방출한 next 이벤트를 구독자에게 전달한다
- 회원가입 기능을 구현할 때 사용할 수 있다
- 이 연산자는 두 가지 형태로 사용한다
- 첫 번째는 data Observable과 클로저를 파라미터로 받는다
- 클로저에는 두 Observable이 방출하는 요소가 전달되고 여기에서 결과를 리턴한다
- 연산자가 최종적으로 리턴하는 Observable은 클로저가 리턴하는 결과를 방출한다
- 두 번째 형태는 trigger Observable에서 Next 이벤트를 전달하면 파라미터로 전달한 data Observable에서 가장 최근에 방출한 넥스트 이벤트를 가져온다
- 이벤트에 포함된 요소를 방출하는 Observable을 리턴한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()

trigger.withLatestFrom(data)
  .subscribe { print($0) }
  .disposed(by: bag)
  
data.onNext("Hello")
trigger.onNext(())
trigger.onNext(())

// data.onCompleted()
// data.onError(MyError.error)
// trigger.onNext(())
trigger.onCompleted()

--> 출력결과
next(Hello)
next(Hello)
// error(error) // completed와 달리 error는 바로 전달된다
// next(Hello) // completed가 아닌 next가 전달 된다
completed


</code>
</pre>



#### 41/98 sample Operator
- trigger Observable이 Next 이벤트를 전달할 때마다 data Observable이 Next 이벤트를 방출하지만, 동일한 Next 이벤트를 반복해서 방출하지 않는 sample 연산자
- dataObservable.withLatestFrom(triggerObservable) 과 같은 형태로 사용한다 -> withLatestFrom 연산자와 반대다
- data Observable에서 연산자를 호출하고 trigger Observable을 파라미터로 전달한다
- trigger Observable에서 Next 이벤트를 전달할 때마다 data Observable이 최신 이벤트를 방출한다
- 동일한 Next 이벤트를 반복해서 방출하지 않는다


<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()
  
data.sample(trigger)
  .subscribe { print($0) }
  .disposed(by: bag)

trigger.onNext(())
data.onNext("Hello")

trigger.onNext(())
trigger.onNext(()) // 한 번 더 해도 추가로 Next 이벤트가 방출되지 않는다

// data.onCompleted()
// trigger.onNext(())

data.onError(MyError.error)

--> 출력결과
next(Hello)
error(error) // error는 trigger Observable이 Next 이벤트를 방출하지 않더라도 즉시 구독자에게 전달된다
// completed // sample 연산자는 completed 이벤트를 그대로 전달한다


</code>
</pre>

#### 42/98 switchLatest Operator
- 가장 최근에 방출된 옵저버블을 구독하고, 이 옵저버블이 전달하는 이벤트를 구독자에게 전달하는 switchLatest 연산자
- 가장 최근 옵저버블이 방출하는 이벤트를 구독자에게 전달한다
- 어떤 옵저버블인지 가장 최근 옵저버블인지 이해하는 것이 핵심이다

#### 43/98 reduce Operator
- 시드 값과 옵저버블이 방출하는 요소를 대상으로 클로저를 실행하고 최종 결과를 옵저버블로 방출하는 reduce 연산자

