- 실제 write-up 작성 시 플래그를 구하는 과정을 적어주세요!  

## 문제 분석

소스 코드는 아래와 같은데

```c
// Name: r2s.c
// Compile: gcc -o r2s r2s.c -zexecstack

#include <stdio.h>
#include <unistd.h>

void init() {
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
}

int main() {
  char buf[0x50];

  init();

  printf("Address of the buf: %p\n", buf);
  printf("Distance between buf and $rbp: %ld\n",
         (char *)__builtin_frame_address(0) - buf);

  printf("[1] Leak the canary\n");
  printf("Input: ");
  fflush(stdout);

  read(0, buf, 0x100);
  // 0x50 < 0x100
  printf("Your input is '%s'\n", buf);

  puts("[2] Overwrite the return address");
  printf("Input: ");
  fflush(stdout);
  gets(buf);
  // unsafe function

  return 0;
}

```

- 버퍼의 주소와 rbp와 buf 사이의 주소 차이를 알려준다.

```c
printf("Address of the buf: %p\n", buf);
  printf("Distance between buf and $rbp: %ld\n",
         (char *)__builtin_frame_address(0) - buf);
```

- 사용자 입력이 두 번 존재한다.

```c
char buf[0x50]
read(0, buf, 0x100)
//버퍼보다 큰 입력
gets(buf)
//입력값 크기 고정하지 않음
```


---

## 익스플로잇 시나리오


### 1. 카나리 우회

- 두 번째 입력으로 반환 주소 덮기 가능. 그러나 카나리가 조작되면 프로그램 강제 종료
- 첫 번째 입력에서 카나리를 먼저 구하고 두 번째 입력에 사용
- buf에 오버플로우를 발생시켜서 카나리를 구하자


### 2. 셸 획득

- 카나리를 구했으니 두 번째 입력으로 반환 주소를 덮을 수 있다. 
- 반환 주소에 get_shell()의 주소를 덮을 수 없다. 그러한 함수가 없으므로
- 셸을 획득하는 코드를 직접 주입하고 해당 주소로 실행 흐름을 옮겨야 한다.

  ---
### 익스플로잇

  ...

  ---
