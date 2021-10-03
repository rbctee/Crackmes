# Sh4ll1

Analyzing the binary `crackMe1.bin` with `radare2`, I discovered the `main` function:

```txt
[0x00000a8d]> fs symbols
[0x00000a8d]> f

# ...
# 0x00000a8d 21 main
# ..
```

Inside I found the following code:

```asm
0x00000a8d      55             push rbp
0x00000a8e      4889e5         mov rbp, rsp

0x00000a91      e83affffff     call sym systemv()          ; sym.systemv__
0x00000a96      e851ffffff     call sym systemo()          ; sym.systemo__
0x00000a9b      b800000000     mov eax, 0

0x00000aa0      5d             pop rbp
0x00000aa1      c3             ret
```

As you can see, there are the two procedures:

- `systemv`
- `systemo`

The first one runs the following instructions:

```asm
; var int64_t var_ch @ rbp-0xc
; var int64_t var_8h @ rbp-0x8
; var int64_t var_4h @ rbp-0x4

0x000009d0      55             push rbp                    ; systemv()
0x000009d1      4889e5         mov rbp, rsp

0x000009d4      c745fc050000.  mov dword [var_4h], 5
0x000009db      c745f8070000.  mov dword [var_8h], 7
0x000009e2      c745f4f50100.  mov dword [var_ch], 0x1f5

0x000009e9      90             nop
0x000009ea      5d             pop rbp
0x000009eb      c3             ret
```

It sets some variables:

- `rbp-0x4` to 0x5
- `rbp-0x8` to 0x7
- `rbp-0xc` to 0x1f5 (501)

The second function (systemo), does the following:

```asm
; var int64_t var_10h       @ rbp-0x10
; var int64_t var_ch        @ rbp-0xc
; var int64_t var_8h        @ rbp-0x8
; var int64_t var_4h        @ rbp-0x4

; [OMITTED]

0x560667a009f4      mov     eax, dword [rbp - 8]            ; copies 0x7 into RAX
0x560667a009f7      add     dword [rbp - 4], eax            ; adds 0x7 to 0x5 -> 0xc
0x560667a009fa      mov     eax, dword [rbp - 4]            ; copies 0xc into RAX
0x560667a009fd      imul    eax, eax, 0x2d                  ; signed multiply: eax = 0xc * 0x2d = 540

0x560667a00a00      mov     dword [rbp - 0xc], eax          ; store the int. 540 into the third local var.
0x560667a00a03      mov     dword [rbp - 0x10], 0           ; ... 0 into the fourth local var.

0x560667a00a0a      lea     rsi, str.Password:_
0x560667a00a11      lea     rdi, [std::cout]
0x560667a00a18      call    std::basic_ostream<char, ...    ; entire function name omitted
                                                            ; it simply prints RSI ("Password: ")

0x560667a00a1d      lea     rax, [rbp - 0x10]               ; copy the address of the input string into RAX
0x560667a00a21      mov     rsi, rax
0x560667a00a24      lea     rdi, [std::cin]
0x560667a00a2b      call    std::istream::operator>>(int&)  ; convert the input string to integer

0x560667a00a30      mov     eax, dword [rbp - 0x10]         ; Copy the new integer into EAX
0x560667a00a33      cmp     eax, dword [rbp - 0xc]          ; Compares EAX with 0x021c (540)
0x560667a00a36      jne     0x560667a00a62                  ; jump to negative message if they are not equal
0x560667a00a38      lea     rsi, str.Good_password
0x560667a00a3f      lea     rdi, [std::cout]
; prints "Good password"

0x560667a00a62      lea     rsi, str.Bad_password
0x560667a00a69      lea     rdi, [std::cout]
; prints "Bad password"
```

If I had to convert the assembly code into C++, it should be something like this:

```cpp
#include <iostream>

using namespace std;

int main() {
    int n;

    cout << "Password: ";
    cin >> dec >> n;
  
    if (n == 540) {
        cout << "Good password\n";
    } else {
        cout << "Bad password\n";
    }

    return 0;
}
```

And compile it with `gpp -w main.cpp`.

## References

1. [Local Variables](https://stackoverflow.com/questions/14535266/local-variables-offset-from-stack-base-pointer/14535289)
