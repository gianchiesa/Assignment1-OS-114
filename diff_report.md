# Code Modification Report
Berikut adalah bagian-bagian kode yang diubah

## Makefile
Line 16 :
```C
CS333_UPROGS += _date
```
## syscall.c
Line 109 - 112:
```C
#ifdef CS333_P1
// internally, the function prototype must be ’int’ not ’uint’ for sys_date()
extern int sys_date(void);
#endif // CS333_P1
```
Line 139 - 141:
```C
#ifdef CS333_P1
[SYS_date]    sys_date,
#endif
```
Line 182 - 185:
```C

#ifdef PRINT_SYSCALLS
cprintf("%s -> %d\n",
        syscallnames[num], curproc->tf->eax);
#endif
```

## user.h
Line 28 - 30:
```C
#ifdef CS333_P1
int date(struct rtcdate*);
#endif // CS333_P1
```

## usys.s
Line 33:
```
SYSCALL(date)
```

## syscall.h
Line 24:
```C
#define SYS_date    SYS_halt+1
```

## sysproc.c
Line 100 - 110:
```C
#ifdef CS333_P1
int
sys_date(void)
{
  struct rtcdate *d;
  if (argptr(0, (void*)&d, sizeof(struct rtcdate)) < 0 )
    return -1;
  cmostime(d);
  return 0;
}
#endif
```

## proc.h
Line 52:
```C
uint start_ticks;
```

## proc.c
Line 151:
```C
p->start_ticks = ticks;
```
Line 566 - 578:
```C
#ifdef CS333_P1
  int elapsed = ticks - p->start_ticks;
  int elapsed_sec = elapsed / 1000;
  int get_ms_value = elapsed % 1000;
  char* afterpoint = "";
  if (get_ms_value >= 10 && get_ms_value < 100) {
    afterpoint = "0";
  }
  if (get_ms_value < 10) {
    afterpoint = "00";
  }
  cprintf("%d\t%s\t\t%d.%s%d\t%s\t%d\t", p->pid, p->name, elapsed_sec, afterpoint, get_ms_value, state_string, p->sz);
#endif
```
