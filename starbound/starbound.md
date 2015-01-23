#Starbound

##A Game
```
 / Pos: 365 HP: 100                              \ 
|                                                 |
|                                                 |
|        ,~                                       |
|       /..\             o                        |
|       ....             T                        |
|       ....             ,~                       |
|      /....`_~         /..                       |
|    ,-........         ...\       ,__~           |
|   /..........\       /#...    ,_-....\          |
|   .....#......      /.....   /........    ,_~   |
|  /...........#    ,-......`_-#...#....`__-...\  |
| /.#...........    ........................#...  |
|-..............\  /............................`_|
|#...........#...`-..............#...........#....|
|..............#...............................#..|
|...............#.......@...........#.............|
|.........@.......................#...........#...|
 \_______________________________________________/

```

##The Menu

```
-+STARBOUND v1.0+-
  0. Exit
  1. Info
  2. Move
  3. View
  4. Tools
  5. Kill
  6. Settings
  7. Multiplayer
> 
```

#Analysis

##Overview
```c
int __cdecl main()
{
  __int32 v0; // eax@3
  int v2; // [sp+10h] [bp-104h]@2

  init();
  while ( 1 )
  {
    alarm(0x3Cu);
    (*(void (**)(void))&me[508])();
    if ( !readn(&v2, 256) )
      break;
    v0 = strtol((const char *)&v2, 0, 10);
    if ( !v0 )
      break;
    (*(void (**)(void))&me[4 * v0 + 468])();
  }
  do_bye();
  return 0;
}
```

##The flag
```c
int __cdecl get_server_key(int a1)
{
  int v1; // esi@1
  signed int i; // ebx@1
  char v4; // [sp+14h] [bp-268h]@4
  char v5[524]; // [sp+70h] [bp-20Ch]@1

  memset(v5, 0, 0x200u);
  v1 = open("/home/flags/starbound", 0);
  for ( i = 0; i <= 511; i += 32 )
  {
    read(v1, &v5[i], 0x20u);
    lseek(v1, 0, 0);
  }
  memfrob(v5, 512);
  MD5_Init(&v4);
  MD5_Update(&v4, v5, 512);
  return MD5_Final(a1, &v4);
}
```

##Remarks
* `me` is a big global buffer
* `readn(&v2, 100u)`: stack overflow?
* `get_server_flag`

#The Plan

##Steps
* [Hijack the control flow](#step1-hijack-the-control-flow-1)
* [Read the flag onto `me`](#step2-read-the-flag)
* [Dump the flag using `cmd_info`](#step3-dump-the-flag)

##Step1: Hijack the Control Flow (1)
```
-+STARBOUND v1.0+-
  0. Exit
  1. Info
  2. Move
  3. View
  4. Tools
  5. Kill
  6. Settings
  7. Multiplayer
> 6

-+STARBOUND v1.0: SETTINGS+-
  0. Exit
  1. Back
  2. Name
  3. IP
  4. Toggle View
> 2 AAAAA... <<<<<<< put rop code onto the stack
Enter your name:  <<<<<<< overflow to hijack control flow
```

##Step1: Hijack the Control Flow (2)

```
.bss
0x08057F80      me (char[512])
...
0x080580C8      host
...
0x080580D0      player name (me[336]) <<<<<< overflow (0x100)
...             (cmds)
0x08058178
0x0805817c      (me[508])  <<<<<< hijack (executes in loop)
0x08058180      end of me
```

##Step1: Hijack the Control Flow (3)
```c
int __cdecl main()
{
  __int32 v0; // eax@3
  int v2; // [sp+10h] [bp-104h]@2

  init();
  while ( 1 )
  {
    alarm(0x3Cu);
    (*(void (**)(void))&me[508])(); // <<<<<< Hijack
    if ( !readn(&v2, 256) ) // <<<<<<< Overflow
      break;
    v0 = strtol((const char *)&v2, 0, 10);
    if ( !v0 )
      break;
    (*(void (**)(void))&me[4 * v0 + 468])();
  }
  do_bye();
  return 0;
}
```

##Step2: Read the flag (1)
```
-+STARBOUND v1.0: SETTINGS+-
  0. Exit
  1. Back
  2. Name
  3. IP
  4. Toggle View
> 2 AAAAA... <<<<<<< *put rop code onto the stack*
Enter your name:  <<<<<<< overflow to hijack control flow
```

##Step2: Read the flag
```
ROP stack:
+------------------+
|                  |
+------------------+
```

##Step3: Dump the flag
``` c
// cmd_info
__printf_chk(
    1, 
    " +  Player: %s (from %s)\n",
    134578384, // 0x80580d0: me[336] <- player name
    *(_DWORD *)&me[328]);
```
