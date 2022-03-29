# useful-scripts
### Ready for clipboard → .zshrc .bashrc

#### Contents
- [1. YouTube](#YouTube-without-ads "YouTube (without ads)")

#### YouTube (without ads)
Uses DDG w/ params to force YouTube video results from your query, and plays the results you select w/ [MPV](https://github.com/mpv-player/mpv) or continuously queries depending on your subsequent input.  Could easily be reconfigured for use with [Lynx](http://lynx.browser.org), [w3m](http://w3m.sourceforge.net), etc. instead of [ddgr](https://github.com/jarun/ddgr).

e.g. `yt portal 2 longplay`

Dependencies: [ddgr](https://github.com/jarun/ddgr), [MPV](https://github.com/mpv-player/mpv)
```ZSH
yt() {
    #!/bin/sh
    if [ $1 = "q" ]; then
        return
    fi
    query="'$*'"  
    isFound=0
    url=""
    output="$(ddgr -x --np --unsafe site:youtube.com/watch $query)\n"
    clear
    echo "$output"
    echo -ne "Input # or search term (q to quit): "
    read term
    echo "$output" | while read line || [[ -n $line ]];
    do
        if [ $isFound -eq 1 ]; then
            url=$line
            break
        fi
        if [[ $line == *$term". "* ]]; then
            isFound=1
        fi
    done
    if [ $isFound = 0 ]; then
	    yt $term
    else
        mpv $url
	    yt $query
    fi 
}
```
