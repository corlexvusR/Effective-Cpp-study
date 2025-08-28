# 항목 2: #define을 쓰려거든 const, enum, inline을 떠올리자 - 작성자: 서영은

<aside>

# 💡이것만은 잊지말자!

- 단순한 상수를 쓸 때는, #define보다 const 객체 혹은 enum을 우선 생각하자.
- 함수처럼 쓰이는 매크로를 만들려면, #define 매크로보다 인라인 함수를 우선 생각하자.
</aside>

---

# 📌 선행 처리자 사용시 발생할 수 있는 문제 1. 컴파일, 디버깅 문제

```cpp
#define ASPECT_RATIO 1.653
```

우리에겐 ASPECT_RATIO가 `기호식 이름(symbolic name)`으로 보이지만 컴파일러에겐 전혀 보이지 않음.

왜? ⇒ 소스 코드가 컴파일러에게 넘어가기 전에 선행 처리자가 상수로 바꾸어 버리기 때문.

그 결과 숫자 상수로 대체된 코드에서 컴파일 에러가 발생하면 에러 메시지엔 ASPECT_RATIO가 뜨는게 아니라 1.653이 나오게 되어서, 디버깅 하기 힘들어짐.

`기호식 디버거(symoblic debugger)` 에서도 문제의 소지가 있음. 

왜? ⇒ 기호 테이블에 이름이 들어가지 않아서. 디버깅할 때 ASPECT_RATIO를 찾아서 할 수 없다는 뜻.

## C++의 컴파일 과정

전처리(Preprocessing) 단계 → 컴파일(Compile) 단계 → 어셈블(Assemble) 단계 → 링킹(Linking) 단계

![image.png](image.png)

1. **전처리 단계:** #include, #define 와 같은 전처리기 매크로들을 처리
2. **컴파일 단계:** 각각의 소스 파일들을 어셈블리 명령어로 변환
3. **어셈블 단계:** 어셈블리 코드들을 실제 기계어로 이루어진 목적 코드(Object file)로 변환
4. **링킹 단계:** 각각의 목적 코드들을 모아서 하나의 실행 파일로 만들어주는 단계

**참고** https://modoocode.com/319

**`선행 처리자란?`** #define, #include 등 **`#`**이 붙은 명령어들.

**`기호 테이블(symbol Table)이란?`**

컴파일러와 링커에서 사용하는 데이터 구조로, 프로그램에서 사용되는 식별자(변수 및 함수 이름 등) 목록과 유형, 범위(가시성), 메모리 위치 등 각 식별자에 대한 정보를 담고 있다.

컴파일러와 링커가 프로그램 전체에서 이러한 식별자에 대한 참조를 해결하는 데 도움이 된다.

- **참고**
    
    https://danielcs.tistory.com/239
    
    https://learn.microsoft.com/en-us/visualstudio/debugger/specify-symbol-dot-pdb-and-source-files-in-the-visual-studio-debugger?view=vs-2022#how-symbol-files-work
    
    https://learn.microsoft.com/en-us/visualstudio/debugger/how-to-use-the-call-stack-window?view=vs-2022
    
    https://csmylov.blogspot.com/2018/08/c-symbol-table.html
    

# ✅ 해결 방법! 매크로 대신 상수를 쓰자.

```cpp
const double AspectRatio = 1.653;  // 대문자로만 표기하는 이름은 대개
																	// 매크로에 쓰는 것이라, 이름 표기도 바꾼다.
```

언어 차원에서 지원하는 상수 타입의 데이터이기 때문에 컴파일러에도 보이며 기호 테이블에도 들어감.

상수가 부동소수점 실수 타입일 경우에는 컴파일을 거친 최종 코드의 크기가 #define을 썼을 때보다 작게 나올 수 있음.

**`왜?`** 매크로를 쓰면 코드에 ASPECT_RATIO가 전부 1.653으로 바뀌면서 목적 코드 안에 1.653의 사본이 등장 횟수만큼 들어가게됨. 그러나, 상수 타입의 AspectRatio는 아무리 여러 번 쓰더라도 사본은 딱 한 개만 생김.

## ⚠️ #define을 상수로 교체할 때 주의할 점 2가지

- 상수 포인터(constant pointer)를 정의하는 경우
    - 포인터는 물론이고, 포인터가 가리키는 대상까지 const로 선언해야 한다.
    
    ```cpp
    const char* const autorName = "Scott Meyers";
    const std::string autorName("Scott Meyers");
    ```
    
- 클래스 멤버로 상수를 정의하는 경우
    - 어떤 상수의 유효범위를 클래스로 한정하고자 하고, 상수의 사본 개수가 한 개를 넘지 못하게 하고 싶다면 정적(static) 멤버로 만들어야함
    
    ```cpp
    class GamePlayer
    {
    	private:
    		static const int NumTurns = 5;  // 상수 선언
    		int scores[NumTurns];           // 상수 사용하는 부분
    };
    ```
    
    여기서 NumTurns는 선언(declaration) 된 것. 정의가 아님.
    
    C++에서는 정의가 마련되어 있어야 하는 게 보통이지만, 정적 멤버로 만들어지는 정수류 타입(각종 정수 타입, char, bool 등)의 클래스 내부 상수는 예외임.
    
    주소를 취하지 않는 한, 정의 없이 선언만 해도 아무 문제가 없게 되어 있음.
    
    단, 클래스 상수의 주소를 구한다든지, 주소를 구하지 않는데도 컴파일러가 정의하라고 하는 경우에는 아래처럼 별도의 정의를 제공해야 함.
    
    ```cpp
    const int GamePlayer::NumTurns; // NumTurns의 정의.
    ```
    
    이때 클래스 상수의 정의는 헤더 파일이 아니라, 구현 파일에 둔다.
    
    정의에는 상수의 초기값이 있으면 안 됨. 왜? ⇒ 클래스 상수의 초기값은 해당 상수가 선언된 시점에서 바로 주어지기 때문. (즉, NumTurns는 선언될 당시에 바로 초기화된다는 것)
    
    #define은 클래스 상수를 정의하는 데 쓸 수도 없을 뿐 아니라 어떤 형태의 캡슐화 혜택도 받을 수 없음.
    
    물론, 상수 데이터 멤버는 캡슐화가 됨. NumTurns가 그러함.
    
    좀 오래된 컴파일러는 위의 문법을 허용 안하는 경우가 있음. 그럴때는 초기값을 상수 정의 시점에 주자.
    
    ```cpp
    class CostEstimate
    {
    	private:
    		static const double FudgeFactor;  // 정적 클래스 상수의 선언 (헤더 파일)
    };
    
    const double CostEstimate::FudgeFactor = 1.35; // 정적 클래스 상수의 정의 (구현 파일)
    ```
    
    **한 가지 예외** ⇒ 클래스를 컴파일하는 도중에 클래스 상수의 값이 필요할 때.
    
    예를 들면 배열 멤버를 선언할 때. 컴파일러가 컴파일 과정에서 배열의 크기를 알아야하기 때문.
    
    이런 경우 **`나열자 둔갑술(enum hack)`** 사용.
    
    **나열자(enumerator) 타입의 값은 int가 놓일 곳에도 쓸 수 있다**는 C++의 특성을 활용한 것.
    
    ```cpp
    class GamePlayer
    {
    	private:
    		enum { NumTurns = 5 };  // NumTurns를 5에 대한 기호식 이름으로 만든다.
    		int scores[NumTurns];
    };
    ```
    
    **나열자 둔갑술을 알아두면 좋은 점**
    
    - 내가 선언한 정수 상수를 다른 사람이 주소를 얻거나 참조자를 쓰는 것이 싫으면 enum을 써서 못하게 할 수 있음.
    - 지극히 실용적인 이유로, 상당히 많은 코드에서 사용되고 있는 기법이기 때문.
    템플릿 메타프로그래밍의 핵심 기법이기도 함.

# 📌 #define 지시자의 또다른 오용 사례 2. 매크로 함수

함수처럼 보이지만 함수호출 오버헤드를 일으키지 않는 매크로를 구현하는 경우.

```cpp
// a와 b 중에 큰 것을 f에 넘겨 호출
#define CALL_WITH_MAX(a, b) ((a) > (b) ? (a) : (b))
```

**문제가 되는 경우**

```cpp
int a = 5, b = 0;

CALL_WITH_MAX(++a, b);     // a가 두 번 증가
CALL_WITH_MAX(++a, b+10);  // a가 한 번 증가
```

a가 증가하는 횟수가 달라짐. `왜?` ⇒ 비교를 통해 처리한 결과가 어떤 것이냐에 따라 달라져서.

#define은 단순히 텍스트 치환임. 따라서 컴파일러가 보기 전에 아래와 같이 바뀌게 됨.

```cpp
((++a) > (b) ? (++a) : (b));
((++a) > (b+10) ? (++a) : (b+10));
```

## ✅ 해결 방법! 인라인 함수에 대한 템플릿을 사용하자

```cpp
template<typename T>
inline void callWithMax(const T& a, const T& b)
{
	(a > b ? a: b);
}
```

이 함수는 템플릿이기 때문에 `동일 계열 함수군(family of functions)`을 만들어냄.

동일한 타입의 객체 두 개를 인자로 받고 둘 중 큰 것을 f에 넘겨서 호출하는 구조.

함수 본문에 괄호를 여러번 쓸 필요도 없고, 인자를 여러 번 평가할지도 모른다는 걱정도 없어짐.

뿐만 아니라, 진짜 함수이기 때문에 유효범위 및 접근 규칙을 그대로 따라감.

**`동일 계열 함수군이란?`** 하나의 템플릿으로 만들어 질 수 있는 모든 함수들을 통칭. (int, double, char 등)