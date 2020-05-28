2020-05-27
==
command for unzip
```
#!/bin/bash
dir=./*
for file in $dir
do
        echo "$file"
        if [[ "$file" == *".zip" ]]; then
                echo "${file:2}"
                unzip "${file:2}" -d /mnt/c/Users/USER/Desktop/codelab/oilbank/app/src/main/res/drawable
                rm "${file:2}"
        fi

done


``` 