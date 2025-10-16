# 항목 38: “has-a(…는 …를 가짐)” 혹은 “is-implemented-in-terms-of(…는 …를 써서 구현됨)”를 모형화할 때는 객체 합성을 사용하자 - 작성자: 권우현

<aside>

# 💡이것만은 잊지말자!

- 객체 합성(composition)의 의미는 public 상속이 가진 의미와 완전히 다릅니다.
- 응용 영역에서 객체 합성의 의미는 has-a(...는...를 가짐)입니다. 구현 영역에서는 is-implemented-in-terms-of(...는 ...를 써서 구현됨)의 의미를 갖습니다
</aside>

---

# 1. 개요

- 합성(composition) 이란?
    - 어떤 타입의 객체들이 그와 다른 객체들이 그와 다른 타입의 객체들을 포함하고 있을 경우에 성립하는 그 타입들 사이의 관계
    - 포함된 객체들을 모아서 이들을 포함한 다른 객체를 합성한다는 뜻
    
    ```cpp
    class Address{...}; // 누군가의 거주지
    
    class PhoneNumber{...};
    
    class Person{
    public:
    ...
    private:
    	std::string name; // 이 클래스를 이루는 객체 중 하나
    	Address address; // 마찬가지
    	PhoneNumber voiceNumber; // 역시 마찬가지
    	PhoneNumber faxNumber; // 이것도 마찬가지
    };
    ```
    
    ⇒ Person 객체는 string, Address, PhoneNumber  객체로 이루어져 있음
    
    ⇒ 합성 대신에 쓰이는 다른 용어들
    
    - 레이어링(layering)
    - 포함(containment)
    - 통합(aggregation)
    - 내장(embedding)

# 2. 합성의 두 가지 뜻

- 합성이라는 것은 다음 두 가지 뜻을 가짐
    - 소프트웨어의 개발에서 대하는 영역(domain)이 두 가지 이기 때문
        - 응용 영역(application domain)
            - 객체 중에는 우리 일상생활에서 볼 수 있는 사물을 본 뜬 것들 : 사람，이동 수단，비디오 프레임 등
            - “has-a(…는 …를 가짐)” 관계
        - 구현 영역(implementation domain)
            - 버퍼，뮤텍스,  탐색 트리 등 순수하게 시스템 구현만을 위한 인공물
            - “is-implemented-in-terms-of(…는 …를 써서 구현됨)” 관계

# 3. is - a 와 has - a 구분

```cpp
class Address{...}; // 누군가의 거주지

class PhoneNumber{...};

class Person{
public:
...
private:
	std::string name; // 이 클래스를 이루는 객체 중 하나
	Address address; // 마찬가지
	PhoneNumber voiceNumber; // 역시 마찬가지
	PhoneNumber faxNumber; // 이것도 마찬가지
};
```

Person 클래스가 나타내는 관계는 has-a 관계.

하나의 Person 객체는 이름, 주소, 음성전화 및 팩스 전화 번호를 가지고 있음

사람이 이름의 일종 (Person is a name), 주소의 일종 (Person is an address) 라고 말할 수 없움.

사람이 이름을 가지고 사람이 주소를 가진다고 말하는 것이 자연스러움. 

그래서 is - a 관계랑 has - a 관계를 헷갈리는 경우는 별로 없음

# 4. is - a 와 is - implemented - in - terms - of 구분

---

어떤 객체 구성을 만들어야 한다고 가정해보자.

1. 중복 원소가 없는 집합체
2. 저장공간도 적게 차지하는 클래스

---

⇒ 이러한 템플릿이 필요, 표준 라이브러리의 set 템플릿을 활용해보자~!

## 1) 문제점

1. set 템플릿이 원소 한 개당 포인터 세 개의 오버헤드가 걸리도록 구현되어있음
    1. 대개 균형 탐생 트리 (balanced search tree)로 구현되어 있기 때문
    2. 탐색, 삽입, 삭제에 걸리는 시간 복잡도를 로그 시간으로 보장하기 위해 

⇒ 흠, 적당하지 않은 템플릿이네, 그냥 하나 만들까?/// 음 그래도 개발잔데 코드 재사용 해야지

⇒ 연결리스트 (linked list) 이용해서 코드 재사용을 해보자.!

1. 어떤 식으로 재사용을 해볼까?
    1. Set 템플릿을 만들되, list 에서 파생된 형태부터 시작하도록 만든다. 
        1. Set을 구현하려면 내부에 **원소들을 저장할 자료구조**가 있어야 함
        2. `std::list`는 이걸 다 지원한다:
            1. `push_back` → 삽입 가능
            2. `std::find`(알고리즘) → 탐색 가능
            3. `erase` → 삭제 가능
            4. `size()` → 크기 알 수 있음
    2. Set<T>는 list<T>로부터 상속을 받음
    3. Set 객체는 list의 일종이 되는 것. 

⇒ 항목 32의 설명에서  D와 B사이에 is - a 관계가 성립하면 B에서 참인 것들이 전부 D에서도 참이어야 하는데,

⇒ list 객체는 중복 원소를 가질 수 있는 컨테이너

⇒ 3051이라는 값을 list<int>에 두 번 삽입되면, 3051의 사본 두개를 품게 됨.

⇒ 하지만 Set 객체는 원소가 중복되면 안됨

<aside>
💡

아! public 상속은 이 관계를 모형화 하는데 맞지 않는구나!

Set 객체는 list 객체를 써서 구현되는 (is implemented in terms of) 형태의 설계가 가능..!

</aside>

- `Set`은 내부적으로 데이터를 저장할 방법이 필요해 → 그 역할을 `std::list`에게 맡기자
- 즉, `Set`은 `list`를 **직접 상속받아 보여주는 게 아니라**, 자기 속에 감춰 두고 그것을 **도구처럼 활용해서** 구현하면 된다.
- 이게 바로 “**is implemented in terms of**”라는 말의 의미.

👉 “Set은 List를 이용해서 구현되었을 뿐, Set 자체가 List는 아니다.”

### 옳게 구현한 예

```cpp
template<class T> 
class Set { 
public:
    bool member(const T& item) const;
    void insert(const T& item);
    void remove(const T& item);
    std::size_t size() const;
private:
    std::list<T> rep;   // <-- 핵심: Set이 list를 "내부 구현부"로 사용
};

```

⇒ Set 의 멤버변수로 list를 가지고 있음.

```cpp
// 윗 부분
template<typename T>
bool Set<T>::member(const T& item) const {
    return std::find(rep.begin(), rep.end(), item) != rep.end();
}

// 아랫 부분
template<typename T>
void Set<T>::insert(const T& item) {
    if (!member(item)) rep.push_back(item);
}

```

1. 윗 부분
    - `std::find`는 **[first, last) 구간에서 값이 item과 같은 원소를 찾는 알고리즘**이야.
    - `rep.begin()` ~ `rep.end()`는 `list` 전체 범위.
    - 즉, **리스트 안에서 item을 찾는다**는 뜻.
    
    반환값 해석:
    
    - `std::find`가 끝까지 못 찾으면 `rep.end()`를 반환해.
    - 못 찾았다면 `rep.end()` → `!= rep.end()` → `false`.
    - 찾았다면 해당 위치(iterator)를 반환 → `!= rep.end()` → `true`.
    
    👉 결론: `member`는 **“집합에 item이 들어 있느냐?”**를 검사하는 함수야.
    

1. 아랫 부분
    1. 동작 순서:
        1. `member(item)`으로 **이미 있는지 확인**.
        2. 없으면(`!member(item)`), `rep.push_back(item)`으로 리스트 끝에 원소 추가.
        3. 있으면 아무것도 하지 않음.
    2. 포인트:
        1. `list`의 `push_back`은 그냥 무조건 뒤에 원소를 붙여 → 중복이 생길 수 있어.
        2. 하지만 `Set`은 중복을 허용하지 않는 자료구조 → `member`로 선행 검사해서 **중복을 방지**하는 거야.
        
        👉 즉, `insert`는 `list`를 직접 노출하는 대신, “집합스럽게” 동작하도록 래핑(wrapper) 한 거야.
        
    

⇒ 우왕! set 와 list 의 관계는 is - implemented - in - terms - of 관계 구나!