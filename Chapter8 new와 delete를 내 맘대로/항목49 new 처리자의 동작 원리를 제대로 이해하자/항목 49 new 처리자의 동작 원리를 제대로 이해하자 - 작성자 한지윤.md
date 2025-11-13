# 항목 49: new 처리자의 동작 원리를 제대로 이해하자 - 작성자: 한지윤

## 🎯 발표 목표

> “C++에서 메모리 할당 실패 시 어떻게 대응할 수 있을까?”
> 
> 
> `new-handler`를 활용하면 프로그램의 안정성을 유지하면서
> 
> **클래스별로 다른 예외 처리**를 구현할 수 있다!
> 

---

## 🚨 1. 문제 상황

보통 이렇게 메모리를 할당합니다.

```cpp
int* p = new int[1000000000L];
```

하지만 **할당 실패** 시

👉 `operator new`는 **std::bad_alloc 예외**를 던집니다.

이때, 예외가 발생하기 *직전*에

개발자가 지정한 함수를 호출하게 되는데,

그것이 바로 **new 처리자(new-handler)** 입니다.

---

## 💡 2. new 처리자란?

> operator new가 예외를 던지기 전,
> 
> 
> 사용자가 지정한 **에러 처리 함수를 자동으로 호출하는 장치**
> 

```cpp
namespace std {
    typedef void (*new_handler)();
    new_handler set_new_handler(new_handler p) throw();
}
```

### 예시

```cpp
void outOfMem() {
    std::cerr << "메모리 부족!" << std::endl;
    std::abort();
}

int main() {
    std::set_new_handler(outOfMem);
    int* p = new int[1000000000L];  // 실패 시 outOfMem 호출
}
```

---

## 🧭 3. new 처리자가 할 수 있는 5가지 행동

| 행동 | 설명 |
| --- | --- |
| ✅ 메모리 확보 | 사용 가능한 블록을 해제하거나, 캐시를 정리해 여유 확보 |
| 🔄 다른 new 처리자 설치 | `set_new_handler()`로 새 핸들러 교체 |
| 🗑️ 처리자 제거 | `set_new_handler(nullptr)`로 제거 |
| 🚨 예외 던지기 | `throw std::bad_alloc()` |
| ⛔ 복귀하지 않기 | `abort()` 또는 `exit()` 호출 |

➡ 반드시 이 중 하나의 조치를 해야 한다!

---

## 🧱 4. 클래스별로 다른 처리자 만들기

> 클래스마다 서로 다른 방식으로 “메모리 부족”을 처리하고 싶다면?
> 

```cpp
class X {
public:
    static void outOfMemory();
};
class Y {
public:
    static void outOfMemory();
};

X* p1 = new X;  // 실패 시 X::outOfMemory
Y* p2 = new Y;  // 실패 시 Y::outOfMemory
```

하지만 C++은 클래스별 new 처리자를 직접 지원하지 않기 때문에,

**직접 `set_new_handler`와 `operator new`를 구현해야 합니다.**

---

## ⚙️ 5. Widget 예제 (클래스별 new 처리자)

```cpp
class Widget {
public:
    static new_handler set_new_handler(new_handler p) throw();
    static void* operator new(size_t size) throw(bad_alloc);
private:
    static new_handler currentHandler;
};

new_handler Widget::currentHandler = 0;

new_handler Widget::set_new_handler(new_handler p) throw() {
    new_handler oldHandler = currentHandler;
    currentHandler = p;
    return oldHandler;
}
```

---

### 🔁 RAII로 안전하게 복구하기

```cpp
class NewHandlerHolder {
public:
    explicit NewHandlerHolder(new_handler nh) : handler(nh) {}
    ~NewHandlerHolder() { set_new_handler(handler); }
private:
    new_handler handler;
};
```

```cpp
void* Widget::operator new(size_t size) throw(bad_alloc) {
    NewHandlerHolder h(set_new_handler(currentHandler)); // Widget 전용 핸들러 설치
    return ::operator new(size);  // 실패 시 Widget 핸들러 호출
} // 이전 전역 핸들러 자동 복원
```

✅ `RAII` 덕분에 이전의 전역 new 처리자가 자동 복구됨

---

## 🧩 6. 여러 클래스에 적용하기 — Mixin 패턴

> “이걸 다른 클래스에서도 재사용하고 싶다!”
> 

```cpp
template<typename T>
class NewHandlerSupport {
public:
    static new_handler set_new_handler(new_handler p) throw();
    static void* operator new(size_t size) throw(bad_alloc);
private:
    static new_handler currentHandler;
};
```

```cpp
class Widget : public NewHandlerSupport<Widget> { ... };
```

🧠 이렇게 하면 클래스별로 독립된 new 처리자를 가질 수 있다.

---

## 🚫 7. 예외 불가 new (`std::nothrow`)

```cpp
Widget* pw1 = new Widget;                // 실패 시 예외 발생
Widget* pw2 = new (std::nothrow) Widget; // 실패 시 nullptr 반환
```

- `nothrow`는 **할당 실패에만 적용**,
    
    생성자 내부의 예외는 여전히 발생할 수 있다.
    
- 예외 대신 널 반환이 필요한 환경(임베디드 등)에 한정적으로 사용.

---

## 🧾 8. 정리

| 항목 | 요약 |
| --- | --- |
| 🎯 new-handler란 | 메모리 부족 시 호출되는 사용자 정의 함수 |
| 🧩 set_new_handler | 전역 또는 클래스별로 에러 처리자 등록 가능 |
| 🧱 클래스별 처리 | `operator new`와 RAII로 구현 가능 |
| 🔁 재사용성 | `NewHandlerSupport<T>` mixin 템플릿 활용 |
| 🚫 예외불가 new | 예외 방지는 제한적, 생성자 예외는 그대로 발생 |

---

<aside>

# 🔎 이것만은 잊지 말자!

- `set_new_handler` 함수를 쓰면 메모리 할당 요청이 만족되지 못했을 때 호출되는 함수를 지정할 수 있습니다.
- 예외불가(`northrow`) `new` 는 영향력이 제한되어 있습니다. 메모리 할당 자체에만 적용되기 때문입니다. 이후에 호출되는 생성자에서는 얼마든지 예외를 던질 수 있습니다.
</aside>