---
---
### 2018-09-13

Today I learned a very simple trick to get a mostly accurate date and time on a machine which doesn't have any ntp packages installed. With only `cat` and some socket tricks in Linux, it is possible to get the time from the legacy TIME protocol still available from NIST.

```
cat </dev/tcp/time.nist.gov/13
```
Will return something like the following:
```
58374 18-09-13 17:35:57 50 0 0  97.7 UTC(NIST) *  
```

It is also possible to do it using netcat:
```
nc time.nist.gov 13
```

---

Funny stuff, Star Wars in ascii !
```
cat </dev/tcp/towel.blinkenlights.nl/23
```
More telnet fun:
```
telnet telehack.com
```

* [Taken from a comment on superuser.com](https://superuser.com/a/635039)
