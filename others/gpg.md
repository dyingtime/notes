## 使用GPG签名

```shell
 # generate key
gpg --default-new-key-algo rsa4096 --gen-key
# gpg: key E59D16482998F0D4 marked as ultimately trusted

# list key
gpg --list-secret-keys --keyid-format LONG
# sec   rsa4096/E59D16482998F0D4 2022-05-21 [SC] [expires: 2024-05-20]
#     585639369A6B7EC1BA93ABF3E59D16482998F0D4
# uid                 [ultimate] dyingtime <dyingtime@foxmail.com>

# export key
gpg --armor --export E59D16482998F0D4

# add output to github settings -> SSH and GPG keys -> New GPG key

# add gpg key to git
git config --global user.signingkey E59D16482998F0D4

# sign commit
git config commit.gpgsign true

# delete gpg key
gpg --delete-secret-keys E59D16482998F0D4
```
