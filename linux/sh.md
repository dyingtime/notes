## simple sh command
### awk
```sh
# get file name without ext name 
echo filepath|awk -F "/" '{print $NF}'|awk -F"." '{print $1}'
```