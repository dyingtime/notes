## jq

### install
```shell
brew install jq
```

### foreach array
```shell
curl -X GET https://httpbin.org/json|jq -r '.slideshow.slides[]|"\(.title) \(.type)"'|while read title type;do echo $title $type; done

## Wake up to WonderWidgets! all
## Overview all
```

### read command
#### Loop through a text file 'demo.txt' and print each line:

```shell
while IFS= read -r line; do
  echo "Line: $line"
done <demo.txt
```