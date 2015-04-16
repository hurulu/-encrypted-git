# encrypted-git
how to set a encrypted git repo

Original link [https://gist.github.com/shadowhand/873637](https://gist.github.com/shadowhand/873637)

## First repo configration

Generate a salt and set a pass=123456, which will be used later
```
openssl rand -hex 8
78a2972dfebb6df5
```

```bash
mkdir ~/.gitencrypt
cat ~/.gitencrypt/clean_filter_openssl 
#!/bin/bash

SALT_FIXED=78a2972dfebb6df5
PASS_FIXED=123456

openssl enc -base64 -aes-256-ecb -S $SALT_FIXED -k $PASS_FIXED
```

```bash
cat ~/.gitencrypt/smudge_filter_openssl 
#!/bin/bash

# No salt is needed for decryption.
PASS_FIXED=123456

# If decryption fails, use `cat` instead. 
# Error messages are redirected to /dev/null.
openssl enc -d -base64 -aes-256-ecb -k $PASS_FIXED 2> /dev/null || cat
```

```bash
cat ~/.gitencrypt/diff_filter_openssl 
#!/bin/bash

# No salt is needed for decryption.
PASS_FIXED=123456

# Error messages are redirect to /dev/null.
openssl enc -d -base64 -aes-256-ecb -k $PASS_FIXED -in "$1" 2> /dev/null || cat "$1"
```

Place this file to the directories that you want to encrypt on github 
```
cat .gitattributes 
* filter=openssl diff=openssl
[merge]
    renormalize=true
```
And add the following to the repo/.git/config
```
[filter "openssl"]
    smudge = ~/.gitencrypt/smudge_filter_openssl
    clean = ~/.gitencrypt/clean_filter_openssl
[diff "openssl"]
    textconv = ~/.gitencrypt/diff_filter_openssl
```

#Set up another repo
clone the first repo and then change the configurations as in the first repo.
```
git reset --hard HEAD
```
