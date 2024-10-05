# 스택 버퍼 오버플로우

- 스택의 버퍼에서 발생하는 오버플로우

## 버퍼

- 버퍼: 데이터가 목적지로 이동되기 전에 보관되는 임시 저장소
- 스택에 있는 지역 변수는 스택 버퍼, 힙에 할당된 메모리 영역은 힙 버퍼


## 버퍼 오버플로우

- 말그대로 버퍼가 넘치는 것.
- 버퍼는 메모리상에 연속해서 할당되어 있으므로 어떤 버퍼에서 오버플로우가 발생하면 뒤에 있는 버퍼들의 값이 조작될 위험이 있다.
```c
// Name: sbof_auth.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_auth(char *password) {
  int auth = 0;
  char temp[16];
  strncpy(temp, password, strlen(password));
  //여기서 strncpy함수를 통해 temp버퍼를 복사할 때 temp의 크기인 16바이트가 아닌 인자로 전달된 password크기만큼 복사한다.
  if (!strcmp(temp, "SECRET PASSWORD"))
    auth = 1;
  return auth;
}

int main(int argc, char *argv[]) {
  if (argc != 2) {
    printf("Usage: ./sbof_auth ADMIN_PASSWORD\n");
    exit(-1);
  }
  if (check_auth(argv[1]))
    printf("Hello Admin!\n");
  else
    printf("Access Denied!\n");
}
```

- 위 코드에서는 스택 버퍼 오버플로우가 발생될 여지가 있다.
- check_auth함수에서 strncpy함수를 통해 temp 버퍼를 복사할 때 temp의 크기인 16바이트가 아니라 password크기만큼 복사하기 때문인데, argv\[1]에 16바이트가 넘는 문자열을 전달하면 이들이 모두 복사되어 스택 버퍼 오버플로우가 발생한다.
- temp버퍼에 오버플로우를 발생시켜 auth의 값이 0이 아닌 임의의 값으로 바꿀 수 있다. 이렇게 하면 인증 여부와 무관하게 if(check_quth(argv\[1]))이 참이 된다.

## 데이터 유출

- C언어에서 정상적인 문자열은 널 바이트로 종결, 표준 문자열 출력 함수들은 널 바이트를 문자열의 끝으로 인식함
- 버퍼 오버플로우를 발생시켜서 다른 버퍼와의 사이에 있는 널 바이트를 모두 제거하면 해당 버퍼를 출력시켜 다른 버퍼의 데이터도 같이 출력시키는 것이 가능함.
```c
#include <string.h>
#include <unistd.h>

int main() {
  char secret[16] = "secret message";
  char barrier[4] = {};
  char name[8] = {};
  memset(barrier, 0, 4);
  printf("Your name: ");
  read(0, name, 12);
  printf("Your name is %s. ", name);
}
```

- 위 코드에서는 8바이트 크기의 name버퍼에 12바이트 입력을 받음.
- secret(목표 데이터) 사이에 barrier라는 4바이트의 널 바이트로 채워진 배열이 존재함.
- 버퍼 오버플로우를 이용하여 널 바이트를 모두 널 바이트가 아닌 다른 값으로 변경하면 secret을 읽을 수 있다.

## 실행 흐름 조작

- 함수 호출 규약에서, 함수를 호출할 때 반환 주소를 스택에 쌓고 함수에서 반환될 때 이를 꺼내어 원래 실행 흐름으로 돌아간다.
- 스택 버퍼 오버플로우를 이용하여 반환 주소를 조작하면 실행 흐름을 바꿀 수 있다.

```c
#include <stdio.h>
#include <unistd.h>

void win() {
	printf("You won!\n");
}

int main(void) {
	char buf[8];
	printf("Overwrite return address %p:\n", &win);
	read(0. buf, 32);
	return 0;
}
```

- 위 코드에서는 win()의 주소를 출력하고 8바이트 버퍼 buf에 32바이트 입력을 받는다.
- buf이후에는 saved RBP값 8바이트와 반환 주소 8바이트가 있으므로 b'A' \* 16이후에 win()주소를 이어 붙여 보내면 win()함수를 호출하는 것이 가능하다


