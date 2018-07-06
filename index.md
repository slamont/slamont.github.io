# Personal Notes

## Today I Learned

### 2018-07-06

Today I learned about the command `run-parts` while looking for information on [hooks with certbot](https://stackoverflow.com/questions/42300579/letsencrypt-certbot-multiple-renew-hooks).

Its simple description is:
> run-parts - run scripts or programs in a directory

Very useful when you want to add the *dir.d* concept in one of your scripts.

It was not obvious to me right away when reading the manpage, but the scripts that you place in the directory that you will point run-parts to, should **not** have extensions.

i.e:
```
# /scripts
#  a.sh
#  b.sh
#  c.sh
run-parts /scripts
# Will not execute anything
#
# But with /scripts
#  a
#  b
#  c
run-parts /scripts
# Will execute the 3 scripts a,b and c
```

[run-parts manpage](http://manpages.ubuntu.com/manpages/bionic/man8/run-parts.8.html)

