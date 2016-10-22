# C++ Remote Development in CLion

## What I want

Develop C++ programs on local Mac OS X, test on remote Linux, **without leaving CLion**,
so I can enjoy the IDE features.

## What I have on local

- Mac OS X
- CLion Educational Edition
- A C++ project

## What I have on remote

- Linux
- SSH
- SFTP

## Solution

Summary: **Sync source files via Remote Host Access (Deployment). Run programs via Terminal+SSH**.

### Set up SFTP and auto sync

1. Preferences -> Deployment -> Add -> Fill in SFTP crediential (host, username, password, home directory)
2. Switch to "Mapping" tab -> Filling local and remote path (Your files will be uploaded to the remote path)
3. "OK"
4. Tools -> Deployment -> Automatic Upload

Effect: Everytime you modified a source file, it will be auto synced to the remote Linux via SFTP.

### Compile and execute programs on remote

1. Open the Terminal plugin in CLion
2. Access remote host via SSH
3. Compile and execute on remote host using command line

Effect: Nothing fancy. It's SSH.

## Q&A

1. Q: Where to git? A: *Local. It's the primary copy and you have IDE support.*

## To-Do

1. Remote Run and Debug with IDE support ( https://blog.jetbrains.com/clion/2016/07/clion-2016-2-eap-remote-gdb-debug/ )

2. System symbols


## References:

+ Basically the same thing: http://zhihu.com/question/23551546/answer/112556010  (Chinese)

+ Similar solution on PyCharm (however the "remote interpreter" part is not applicable to C++): 
https://blog.jetbrains.com/pycharm/2013/03/how-pycharm-helps-you-with-remote-development/
