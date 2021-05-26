# Code Modification Report
Berikut adalah bagian-bagian kode yang diubah

## Makefile
Line 3 :
```C
CS333_PROJECT ?= 2
```
Line 16 :
```C
CS333_UPROGS += _date
```
## syscall.c
Line 105 - 115:
```C
#ifdef CS333_P1
extern int sys_date(void);
#endif
#ifdef CS333_P2
extern int sys_getuid(void);
extern int sys_getgid(void);
extern int sys_getppid(void);
extern int sys_setuid(void);
extern int sys_setgid(void);
extern int sys_getprocs(void);
#endif
```
Line 145 - 152:
```C
#ifdef CS333_P2
    [SYS_getuid] sys_getuid,
    [SYS_getgid] sys_getgid,
    [SYS_getppid] sys_getppid,
    [SYS_setuid] sys_setuid,
    [SYS_setgid] sys_setgid,
    [SYS_getprocs] sys_getprocs,
#endif
```
Line 184 - 191:
```C
#ifdef CS333_P2
    [SYS_getuid] "getuid",
    [SYS_getgid] "getgid",
    [SYS_getppid] "getppid",
    [SYS_setuid] "setuid",
    [SYS_setgid] "setgid",
    [SYS_getprocs] "getprocs",
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
Line 33 - 39:
```
SYSCALL(date)
SYSCALL(getuid)
SYSCALL(getgid)
SYSCALL(getppid)
SYSCALL(setuid)
SYSCALL(setgid)
SYSCALL(getprocs)
```

## syscall.h
Line 25 - 31:
```C
#define SYS_date SYS_halt + 1
#define SYS_getuid SYS_date + 1
#define SYS_getgid SYS_getuid + 1
#define SYS_getppid SYS_getgid + 1
#define SYS_setuid SYS_getppid + 1
#define SYS_setgid SYS_setuid + 1
#define SYS_getprocs SYS_setgid + 1
```

## sysproc.c
Line 94 - 160:
```C
#ifdef CS333_P1
int sys_date(void)
{
  struct rtcdate *d;

  if (argptr(0, (void *)&d, sizeof(struct rtcdate)) < 0)
    return -1;
  cmostime(d);
  return 0;
}
#endif

#ifdef CS333_P2
uint sys_getuid(void)
{
  return myproc()->uid;
}

uint sys_getgid(void)
{
  return myproc()->gid;
}

uint sys_getppid(void)
{
  if (!myproc()->parent)
  { // check if parent is null
    return myproc()->pid;
  }
  return myproc()->parent->pid;
}

int sys_setuid(void)
{
  int uid;

  if (argint(0, &uid) < 0)
    return -1;
  if (uid < 0 || uid > 32767)
    return -1;
  myproc()->uid = uid;
  return 0;
}

int sys_setgid(void)
{
  int gid;

  if (argint(0, &gid) < 0)
    return -1;
  if (gid < 0 || gid > 32767)
    return -1;
  myproc()->gid = gid;
  return 0;
}

int sys_getprocs(void)
{
  uint max;
  struct uproc *table;
  if (argint(0, (void *)&max) < 0)
    return -1;
  if (argptr(1, (void *)&table, sizeof(&table) * max) < 0)
    return -1;
  return getprocs(max, table);
}
#endif
```

## proc.h
Line 66 - 72:
```C
#ifdef CS333_P2
  uint uid;
  uint gid;

  uint cpu_ticks_total;
  uint cpu_ticks_in;
#endif
```

## proc.c
Line 9 - 11:
```C
#ifdef CS333_P2
#include "uproc.h"
#endif //CS333_P2
```
Line 160 - 167:
```C
#ifdef CS333_P1
  p->start_ticks = ticks;
#endif

#ifdef CS333_P1
  p->cpu_ticks_total = 0;
  p->cpu_ticks_in = 0;
#endif
```
Line 205 - 208:
```C
#ifdef CS333_P2
  p->uid = DEFAULT_UID;
  p->gid = DEFAULT_GID;
#endif
```
Line 278 - 281:
```C
#ifdef CS333_P2
  np->uid = curproc->uid;
  np->gid = curproc->gid;
#endif
```
Line 427 - 429:
```C
#ifdef CS333_P2
      p->cpu_ticks_in = ticks;
#endif
```
Line 470 - 472:
```C
#ifdef CS333_P2
  p->cpu_ticks_total += (ticks - p->cpu_ticks_in);
#endif //P2
```
Line 601 - 613:
```C
// cprintf("TODO for Project 2, delete this line and implement procdumpP2P3P4() in proc.c to print a row\n");
int elapsed = ticks - p->start_ticks;
int total = p->cpu_ticks_total;
int ppid;
if (p->parent)
{
  ppid = p->parent->pid;
}
else
{
  ppid = p->pid;
}
cprintf("%d\t%s\t\t%d\t\t%d\t%d\t%d.%d\t%d.%d\t%s\t%d\t", p->pid, p->name, p->uid, p->gid, ppid, elapsed / 1000, elapsed % 1000, total / 1000, total % 1000, state_string, p->sz);
```
Line 1000 - 1032:
```C
#ifdef CS333_P2
int getprocs(uint max, struct uproc *table)
{
  int i = 0;
  struct proc *p;
  acquire(&ptable.lock);
  if (!table || max <= 0)
  {
    release(&ptable.lock);
    return -1;
  }
  for (p = ptable.proc; p < &ptable.proc[NPROC]; p++)
  {
    if (i >= max)
      break;
    if (p->state != EMBRYO && p->state != UNUSED)
    {
      table[i].pid = p->pid;
      table[i].uid = p->uid;
      table[i].gid = p->gid;
      table[i].ppid = (!p->parent) ? p->pid : p->parent->pid;
      table[i].elapsed_ticks = ticks - p->start_ticks;
      table[i].CPU_total_ticks = p->cpu_ticks_total;
      table[i].size = p->sz;
      safestrcpy(table[i].state, states[p->state], sizeof(table[i]).state);
      safestrcpy(table[i].name, p->name, sizeof(table[i]).name);
      i++;
    }
  }
  release(&ptable.lock);
  return i;
}
#endif // CS333_P2
```

## defs.h
Line 12 - 14:
```C
#ifdef CS333_P2
struct uproc;
#endif // CS333_P2
```
Line 12 - 14:
```C
#ifdef CS333_P2
int getprocs(uint max, struct uproc *table);
#endif // CS333_P2
```

## ps.c
Line 1 - 43:
```C
#ifdef CS333_P2
#include "types.h"
#include "user.h"
#include "uproc.h"

int main(void)
{
    struct uproc *table;
    int i;
    uint max = 72;
    int catch = 0;
    uint elapsed, decimal, seconds, seconds_decimal;
    table = malloc(sizeof(struct uproc) * max);
    catch = getprocs(max, table);
    if (catch == -1)
        printf(1, "\nError: Invalid max or NULL uproc table\n");
    else
    {
        printf(1, "\nPID\tName\tUID\tGID\tPPID\tElapsed\tCPU\tState\tSize");
        for (i = 0; i < catch; ++i)
        {
            decimal = table[i].elapsed_ticks % 1000;
            elapsed = table[i].elapsed_ticks / 1000;
            seconds_decimal = table[i].CPU_total_ticks % 1000;
            seconds = table[i].CPU_total_ticks / 1000;
            printf(1, "\n%d\t%s\t%d\t%d\t%d\t%d.", table[i].pid, table[i].name, table[i].uid, table[i].gid, table[i].ppid, elapsed);
            if (decimal < 10)
                printf(1, "00");
            else if (decimal < 100)
                printf(1, "0");
            printf(1, "%d\t%d.", decimal, seconds);
            if (seconds_decimal < 10)
                printf(1, "00");
            else if (seconds_decimal < 100)
                printf(1, "0");
            printf(1, "%d\t%s\t%d", seconds_decimal, table[i].state, table[i].size);
        }
        printf(1, "\n");
    }
    free(table);
    exit();
}
#endif // CS333_P2
```

## testsetuid.c
Line 1 - 10:
```C
#ifdef CS333_P2
#include "types.h"
#include "user.h"

int main(int argc, char *argv[])
{
    printf(1, "***** In %s: my uid is %d\n\n", argv[0], getuid());
    exit();
}
#endif
```

## time.c
Line 1 - 35:
```C
#ifdef CS333_P2
#include "types.h"
#include "user.h"

int main(int argc, char *argv[])
{
    int t1 = 0, t2 = 0, elapsed = 0, decimal = 0, pid = 0;
    ++argv;
    t1 = uptime();
    pid = fork();
    if (pid < 0)
    {
        printf(1, "Ran in 0.000 seconds\n");
        exit();
    }
    else if (pid == 0)
    {
        exec(argv[0], argv);
    }
    else
    {
        wait();
        t2 = uptime();
        decimal = (t2 - t1) % 1000;
        elapsed = (t2 - t1) / 1000;
        printf(1, "%s ran in %d.", argv[0], elapsed);
        if (decimal < 10)
            printf(1, "00");
        else if (decimal < 100)
            printf(1, "0");
        printf(1, "%d seconds\n", decimal);
    }
    exit();
}
#endif // CS333_P2
```