 
# 항목 42: typename의 두 가지 의미를 제대로 파악하자 - 작성자: 서무성

<aside>
🔎

# 이것만은 잊지말자!

- 템플릿 매개변수를 선언할 때, class 및 typename은 서로 바꾸어 써도 무방합니다.
- 중첩 의존 타입 이름을 식별하는 용도에는 반드시 typename을 사용합니다. 단, 중첩 의존 이름이 기본 클래스 리스트에 있거나 멤버 초기화 리스트 내의 기본 클래스 식별자로 있는 경우에는 예외입니다.
</aside>

---

### ❗ 파트 요약: 템플릿 코드에서  `typename`을 언제, 왜 사용해야 하는가?

`typename`은 템플릿 매개변수를 선언하는 용도 외에, 템플릿 정의 안에서 **이름이 타입을 가리킨다는 것을 컴파일러에게 명시적으로 알려주는** 중요한 역할을 합니다. 컴파일러가 타입인지 변수인지 헷갈릴 수 있는 모호한 상황을 해결하기 위해 사용됩니다.

---

### 템플릿 선언문에 쓰인 class와 typename의 차이

```cpp
template<class T> class Widget;      // "class"를 사용
template<typename T> class Widget;  // "typename"을 사용 (위와 동일)
```

- 질문의 답
    - 차이가 없다. 템플릿의 타입 매개변수를 선언할 때 class와 typename의 뜻이 완전히 똑같다.
    - 그냥 선호하는 방식으로 쓰면 된다.

### Class와 Typename이 다른 경우

- 템플릿 안에서 참조할 수 있는 이름의 종류가 두 가지라는 것을 먼저 알아야 한다.
- 함수 템플릿이 하나 있다고 가정.
    - STL과 호환되는 컨테이너를 받아드리도록 만들어짐
    - 이 컨테이너에 담기는 객체는 int에 대입할 수 있다.
    - 이 템플릿이 하는 일은 컨테이너에 담긴 원소들 중 두번째 것의 값을 출력하는 것이다.
    
    ```cpp
    template<typename C>                //컨테이너에 들어 있는
    void print2nd(const C& container)   // 두 번째 원소를 출력합니다.
    {                                   // 도저히 제 정신으로 짠 코드가 아닙니다!
    	if (container.size() >= 2) {
    		C::const_iterator iter(container.begin())； // 첫째 원소에 대한 반복자를 얻습니다.
    		+ + iter;                                    // iter를 두 번째 원소로 옮깁니다.
    		int value = *iter;                           // 이 원소를 다른 int로 복사합니다.
    		std: :cout << value;                         // 이 int를 줄력합니다.
    }
    ```
    
    - iter와 value가 강조되는 변수
    - C::const_iterator iter 와 같이 템플릿 매개변수에 종속된 것을 가리켜 의존 이름(dependent name)이라고 한다. 클래스 안에 중첩된 경우 중첩 의존 이름(nested dependent name)이라고 부른다. 여기서 C::const_iterator iter가 중첩 의존 이름이다.
    - 또 다른 지역변수, value는 int 타입. 템플릿 매개변수와 상관없는 타입이름으로 비의존 이름(non-dependent)라고 한다.

### 코드 안에 중첩 의존 이름이 있으면

```cpp
template<typename C>
void print2nd(const C& container)
{
  C::const_iterator * x;
  …
}
```

- 언뜻 보면, C::const_iterator 에 대한 포인터 변수 x를 선언하고 있는 것처럼 보인다.
- 하지만, 만약 우연히 const_iterator라는 이름의 정적 데이터 멤버가 C 안에 들어 있었다고 가정하면 x(전역 변수로 가정)와 곱하는 연산이 되어버린다.
- C의 정체가 무엇인지 알려주지 않으면 천재 컴파일러도 타입인지 아닌지 구분을 못함.
- C++는 이 모호함을 해결하기 위해 **템플릿 안에서 중첩 의존 이름을 만나면 우리가 타입이라고 알려주지 않는 한 타입이 아니라고 가정하는 규칙**을 적용한다.
    - 즉, 중첩 의존 이름은 기본적으로 타입이 아닌 것으로 해석한다.
    - 예외 있음
- 이 경우에 typename을 사용한다.
    
    ```cpp
    template<typename C>
    void print2nd(const C& container)
    {
      if (container.size() >= 2) {
    	  // typename이 없으면 타입이 아니라고 가정, 즉 이 경우 type으로 컴파일러가 확인함
    	  typename C::const_iterator iter(container.begin());
    	  …
      }
    }
    ```
    
    - 템플릿 안에서 중첩 의존 이름을 참조할 경우에는, 그 이름 앞에 typename 키워드를 붙이는 것을 잊지 마세요. (예외 있음)
- typename 키워드는 중첩 의존 이름만 식별하는데 써야 한다!
    
    ```cpp
    template<typename C>         // typename 쓸 수 있음("class"와 같은 의미) 
    void f(const **C**& container,   // typename 쓰면 안 됨 
    	**typename C::iterator** iter);  // typename 꼭 써야 함
    ```
    
    - C는 중첩 의존 타입 이름이 아님
    - 맨 위 C는 그냥 Class와 같은 의미
    - 맨 밑은 중첩 의존 이름이기 때문에

### typename의 예외

- “typename은 중첩 의존 타입 이름 앞에 붙여 주어야 한다”
- 중첩 의존 타입 이름이 기본 클래스의 리스트에 있거나
- 멤버 초기화 리스트 내의 기본 클래스 식별자로 있는 경우
    
    ```cpp
    template<typename T>
    class Derived: public Base<T>::Nested {   // 상속되는 기본클래스 리스트
    public :																	// typename 사용 불가
      explicit Derived(int x)                 // 멤버 초기화 리스트에 있는 
      : Base<T>::Nested(x) {                  // 기본 클래스 식별자 typename 사용 불가
        typename Base<T>::Nested temp;  // 중첩의존타입의 이름, typename 필요하다
        …
      }
      …
    };
    ```
    

### typename에 관한 예제

```cpp
temp1ate<typename IterT>
void workWithlterator(IterT iter)
{
	 typename std::iterator_traits<IterT>::value_type temp(*iter);
	 ...
}
```

- C++ 표준의 특성정보(tmts) 클래스(항목 47 참3)를 사용한 것
- IterT 타입의 객체로 가리키는 대상의 타입이란 뜻
- IterT 객체가 가리키는 것과 똑같은 타입의 지역 변수를 선언한 후 iter가 가리키는 객체로 temp를 초기화하는 문장이다. 만약 IterT가 vector<int>::iterator라면 temp의 타입은 int가 된다.
- 어쨌든, std::iterator_traits<IterT>::value_type 은 중첩 의존 타입 이름(value_type이 std::iterator_traits<IterT> 안에 중첩되어 있다.)이므로 이 이름 앞에는 typename을 꼭 써줘야 한다.
- 근데 님들 이거 그대로 쓸 수 있음?
- typedef로 쓰셈, 근데 typedef와 typename 겹쳐져 있어서 이상해 보임 논리적인 하자는 없음.
```C++
tempiate<typename IterT>
void workWithlterator(IterT iter)
{
	 typedef typename std::iterator_traits<IterT>::value_type value_type;
	 value_type temp(*iter);
	 ...
}
```