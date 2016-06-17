Replace all filenames in a folder with string "apple" to "orange"

    ls | sed 'p;s/apple/orange/g' | sed 's/.*/\"&\"/' | xargs -n2 mv
    
Merged from these answers:
[source 1](https://stackoverflow.com/a/12067078)
[source 2](https://stackoverflow.com/a/11709995)

--

Check if a filename in one folder exists in another folder

    ls | sed 's/.*/\"&\"/' | xargs -I{} bash -c 'if [ ! -f "/other/folder/{}" ]; then echo "{}"; fi;'
    
--

Extract the values within the specified XML tags

    cat file.xml | sed 's/<TAG\(.*\)TAG>/\1/g'