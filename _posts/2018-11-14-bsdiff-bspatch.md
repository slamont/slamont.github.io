---
---
### 2018-11-14

Today I learned about the command `bsdiff` and its counterpart `bspatch`.
I already knew the concept of binary diffs and that it could be used to reduce the size of certain updates, but I never took the time try it.

I decided to try it this morning, so here it goes!

Lets create an example:

#### Initial state
You have two computer with a binary at version x.y.a and you get a newer version x.y.b on the first computer. But transferring the whole binary would take to much bandwidth. So you want to transfer only the differences between x.y.a and x.y.b to the second computer.

I simulated this with 2 directories, computer_a and computer_b. The binaries are simply 2 version of a simple application I used from time to time.

```
$ tree
.
├── computer_a
│   ├── freemind-bin-max-1.0.0.zip
│   └── freemind-bin-max-1.0.1.zip
└── computer_b
    └── freemind-bin-max-1.0.0.zip

2 directories, 3 files

```

#### Generating Binary diff on computer_a

The command `bsdiff` take 3 arguments, oldfile, newfile and the name of the resulting patch file.
```
bsdiff computer_a/freemind-bin-max-1.0.0.zip computer_a/freemind-bin-max-1.0.1.zip computer_a/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```
We then have the following tree.
```
.
├── computer_a
│   ├── freemind-bin-max-1.0.0.zip
│   ├── freemind-bin-max-1.0.1.zip
│   └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff
└── computer_b
    └── freemind-bin-max-1.0.0.zip

2 directories, 4 files

```
Lets look at the size of the files.
```
$ ls -lh computer_a/
total 75M
drwxrwxr-x 2 lams lams 4.0K Nov 14 14:41 .
drwxrwxr-x 4 lams lams 4.0K Nov 14 11:17 ..
-rw-rw-r-- 1 lams lams  36M Nov 14 14:18 freemind-bin-max-1.0.0.zip
-rw-rw-r-- 1 lams lams  36M Nov 14 14:17 freemind-bin-max-1.0.1.zip
-rw-rw-r-- 1 lams lams 3.3M Nov 14 14:41 freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```
See, both binaries are 36MB each but the resulting binary patch is only 3.3MB

#### Transfer the patch file to computer_b

We simulate the transfer of the 3.3 MB patch file to computer_b by simply copying it to the computer_b directory.
```
.
├── computer_a
│   ├── freemind-bin-max-1.0.0.zip
│   ├── freemind-bin-max-1.0.1.zip
│   └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff
└── computer_b
    ├── freemind-bin-max-1.0.0.zip
    └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff

2 directories, 5 files
```

#### Creating new version from the old file and the patch file on computer_b

On the computer_b, we use the command `bspatch` with the old file and the patch file to construct the new file.
The command use the same arguments order as `bsdiff`.
```
bspatch computer_b/freemind-bin-max-1.0.0.zip computer_b/freemind-bin-max-1.0.1.zip computer_b/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```  
We then have the following tree.
```
.
├── computer_a
│   ├── freemind-bin-max-1.0.0.zip
│   ├── freemind-bin-max-1.0.1.zip
│   └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff
└── computer_b
    ├── freemind-bin-max-1.0.0.zip
    ├── freemind-bin-max-1.0.1.zip
    └── freemind_patch_from_1.0.0_to_1.0.1.bsdiff

2 directories, 6 files
```
Same sizes
```
ls -lah computer_a/* computer_b/*
-rw-rw-r-- 1 lams lams  36M Nov 14 14:18 computer_a/freemind-bin-max-1.0.0.zip
-rw-rw-r-- 1 lams lams  36M Nov 14 14:17 computer_a/freemind-bin-max-1.0.1.zip
-rw-rw-r-- 1 lams lams 3.3M Nov 14 14:41 computer_a/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
-rw-rw-r-- 1 lams lams  36M Nov 14 14:21 computer_b/freemind-bin-max-1.0.0.zip
-rw-rw-r-- 1 lams lams  36M Nov 14 15:00 computer_b/freemind-bin-max-1.0.1.zip
-rw-rw-r-- 1 lams lams 3.3M Nov 14 14:54 computer_b/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```
Same sum
```
md5sum computer_a/* computer_b/*
3d15122b99d5c830eb9c35034b66d525  computer_a/freemind-bin-max-1.0.0.zip
bb217c2566e1476f11f1a68ff88a5669  computer_a/freemind-bin-max-1.0.1.zip
55d838e290e01c40daedc7443aec69e4  computer_a/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
3d15122b99d5c830eb9c35034b66d525  computer_b/freemind-bin-max-1.0.0.zip
bb217c2566e1476f11f1a68ff88a5669  computer_b/freemind-bin-max-1.0.1.zip
55d838e290e01c40daedc7443aec69e4  computer_b/freemind_patch_from_1.0.0_to_1.0.1.bsdiff
```
All this by transferring only 3.3MB to computer_b instead of 36MB.
Off course, those are simple files with sizes that are insignificant for today's internet and computers. But if you have in mind embedded computers that requires big updates on low bandwidth networks, then I think it should be something to consider.

* [Good resources for bsdiff/bspatch](http://www.daemonology.net/bsdiff/)
* [Man pages for bsdiff](http://manpages.ubuntu.com/manpages/bionic/man1/bsdiff.1.html)
* [Man pages for bspatch](http://manpages.ubuntu.com/manpages/bionic/man1/bspatch.1.html)
