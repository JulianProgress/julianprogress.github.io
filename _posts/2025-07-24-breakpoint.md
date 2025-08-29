---
layout: post
title: "[Python] Breakpoint 사용해보기"
date: 2025-07-24
tags: [Python, Debugging]
categories: Python
excerpt_image: https://media.geeksforgeeks.org/wp-content/uploads/20190902105053/Debugging-Tips-To-Get-Better-At-It.png
---
![banner](https://media.geeksforgeeks.org/wp-content/uploads/20190902105053/Debugging-Tips-To-Get-Better-At-It.png)

내가 파이썬을 사용할 때마다 느꼈던 불편함 중 하나는 디버깅이었을 것이다. 초기에 파이썬을 사용할 때는 디버깅을 위해 일부러 함수 호출 앞뒤로 값을 찍어보기 위해 귀찮게 일일히 `print()` 를 붙여가며 디버깅을 했던 기억이 있다.

하지만 역시 파이썬이라고 했던가. 파이썬에서는 이러한 상황에서 아주 유용하게 사용할 수 있는 `breakpoint()` 라는 기본 내장함수를 제공한다.
breakpoint 함수는 파이썬의 기본 디버거인 PDB (Python Debugger) 를 호출하는 함수로 이를 활용하면 효율적인 파이썬 디버깅이 가능해진다.

## 기본 사용법
아래 예시를 보자:

```[python]
def double(x):
   print(x)
   x_ = x * 2
   print(x_)
   return x_
val = 3
print(f"{val} * 2 is {double(val)}")
```

위 예제에서와 같은 상황은 파이썬을 개발하다보면 자주 마주치는 상황이다. 함수 내에서 특정 알고리즘에 의해 변수값이 어떻게 변하는지 tracking 이 필요할 때가 많은데 그럴때마다 보통은 위의 예제처럼 앞뒤에 `print()` 함수를 붙여서 변수 값 변화를 체크하게 된다. 

이와 같은 상황에서 디버깅 모드로 진입하고 싶은 지접에 `breakpoint()` 를 사용하면 더 효율적으로 디버깅이 가능해진다:

```[python]
def double(x):
   breakpoint()
   return x * 2
val = 3
print(f"{val} * 2 is {double(val)}")
```

그래서 위와 같이 코드를 작성하면 아래 예시처럼 pdb로 빠지게 된다:

```[bash]
$ python debug_test.py
-> return x * 2
(Pdb) p x
3
(Pdb) continue
3 * 2 is 6
```
여기서 `(Pdb) p x` 부분의 p는 값을 출력하라는 명령어로, `p x` 를 입력하면 x 변수의 값을 출력하라는 의미이다. 

## pdb 명령어 정리
`p` 외에도 pdb 에서 사용할 수 있는 명령어들 중 자주 사용할 법한 명령어를 정리해보자.

- `p expression`: expression 을 평가하고 값을 출력하는 명령어
- `n`: 현재 함수의 다음 줄로 넘어가거나 반환할 때까지 계속 실행
- `s`: 현재 줄을 실행하고, 멈출 수 있는 가장 첫번째 줄에서 멈춤

**`n`과 `s` 차이점**  
`s` 는 호출된 함수 안에서 멈추는데 반면, `n` 은 호출된 함수를 실행하고 현재 함수 바로 다음 줄에서 멈춘다. 아래 예시를 보자:

```[python]
# PDB의 s(step)와 n(next) 차이점 예시
def add_numbers(a, b):
    """두 숫자를 더하는 함수"""
    print(f"add_numbers 함수 내부: a={a}, b={b}")
    result = a + b
    print(f"결과: {result}")
    return result

def multiply_numbers(x, y):
    """두 숫자를 곱하는 함수"""
    print(f"multiply_numbers 함수 내부: x={x}, y={y}")
    result = x * y
    print(f"결과: {result}")
    return result

def main():
    print("프로그램 시작")
    
    # 여기서 브레이크포인트 설정
    import pdb; pdb.set_trace()
    
    # 1번 줄: add_numbers 함수 호출
    sum_result = add_numbers(5, 3)
    
    # 2번 줄: multiply_numbers 함수 호출  
    mult_result = multiply_numbers(4, 2)
    
    # 3번 줄: 결과 출력
    print(f"최종 결과: sum={sum_result}, mult={mult_result}")
    
    print("프로그램 종료")

if __name__ == "__main__":
    main()
```

- `c` 또는 `continue`: 다음 브레이크포인트까지 계속 실행
- `l` 또는 `list`: 현재 위치 주변의 소스 코드를 보여줌
- `w` 또는 `where`: 현재 스택 트레이스를 보여줌
- `h` 또는 `help`: 도움말을 보여줌
- `q` 또는 `quit`: 디버거를 종료하고 프로그램을 중단

## 실제 활용 팁

### 1. 조건부 브레이크포인트
특정 조건에서만 디버깅을 하고 싶다면 다음과 같이 사용할 수 있다:

```python
def process_data(items):
    for i, item in enumerate(items):
        if i == 5:  # 5번째 아이템에서만 디버깅
            breakpoint()
        # 처리 로직
        result = item * 2
    return result
```

### 2. 환경변수로 breakpoint 비활성화
배포 환경에서는 breakpoint() 가 실행되지 않도록 환경변수를 설정할 수 있다:  
```bash
PYTHONBREAKPOINT=0 python your_script.py
```

### 3. 다른 디버거 사용
기본 pdb 대신 다른 디버거를 사용하고 싶다면:
```bash
PYTHONBREAKPOINT=ipdb.set_trace python your_script.py
```

## 마무리
`breakpoint()` 함수는 파이썬 3.7부터 추가된 기능으로, 그 이전 버전에서는 `import pdb; pdb.set_trace()`를 직접 사용해야 했다. 하지만 이제는 간단히 `breakpoint()`만 입력하면 되니 훨씬 편리해졌다.  
print문을 이용한 디버깅도 나름의 장점이 있지만, 복잡한 로직이나 반복문이 많은 코드에서는 `breakpoint()`를 활용한 대화형 디버깅이 훨씬 효율적이다. 한 번 익숙해지면 파이썬 개발 생산성이 크게 향상될 것이다.  
