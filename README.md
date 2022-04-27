# useful-scripts
### Ready for clipboard → .zshrc / .bashrc (or grab the shell scripts)

#### Contents

*\*nix*

* [YouTube](#YouTube-without-ads "YouTube (without ads)")

*macOS*

* [Now Playing](#Now-Playing-Apple-Music "Now Playing (Apple Music)")

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

#### Now Playing (Apple Music)

<img src="images/nowplaying.png" width="400"/>

Enjoy a Now Playing "widget" from your terminal.  Uses standard Unix tooling/piping, Applescript for interfacing with Apple Music, Spark for the progress bar, and Viu for displaying the album art images.  Pairs well with the additional Apple Music scripts I've included here.

Dependencies: [Spark](https://github.com/holman/spark), [Viu](https://github.com/atanunq/viu), [Doug's album-art.applescript](https://dougscripts.com/itunes/2014/10/save-current-tracks-artwork/)

Configuration: 

* Adjust the dimensions of the album art (the two calls to viu) to ensure a square appearance with your terminal emulator's line spacing
* Configure a valid path to album-art.applescript, e.g. ~/Library/Scripts/album-art.applescript

```ZSH
np() {
  init=1
  while :
  do
  	vol=$(osascript -e 'get volume settings')
  	duration=$(osascript -e 'tell application "Music" to get {player position} & {duration} of current track')
  	arr=(`echo ${duration}`)
 	 curr=$(cut -d . -f 1 <<< ${arr[-2]})
 	 diff=$(( end - curr ))
 	 bar=$(spark 0 $diff ${arr[-1]})
 	 currMin=$(echo $(( curr / 60 )))
 	 currSec=$(echo $(( curr % 60 )))
  if [ ${#currMin} = 1 ]
  then
  	currMin="0$currMin"
  fi
  if [ ${#currSec} = 1 ]
  then
  	currSec="0$currSec"
  fi
  if (( curr < 2 || init == 1 )); then
  	init=0
 	name=$(osascript -e 'tell application "Music" to get name of current track')
  	name=${name:0:50}
  	artist=$(osascript -e 'tell application "Music" to get artist of current track')
  	artist=${artist:0:50}
  	record=$(osascript -e 'tell application "Music" to get album of current track')
  	record=${record:0:50}
  	end=$(cut -d . -f 1 <<< ${arr[-1]})
  	endMin=$(echo $(( end / 60 )))
  	endSec=$(echo $(( end % 60 )))
  	if [ ${#endMin} = 1 ]
  	then
    		endMin="0$endMin"
  	fi
  	if [ ${#endSec} = 1 ]
  	then
    		endSec="0$endSec"
  	fi
  	rm ~/Library/Scripts/tmp*
  	osascript ~/Library/Scripts/album-art.applescript
  	if [ -f ~/Library/Scripts/tmp.png ]; then
    		art=$(clear; viu ~/Library/Scripts/tmp.png -w 39 -h 13)
  	else 
    		art=$(clear; viu ~/Library/Scripts/tmp.jpg -w 39 -h 13)
  	fi
  fi
  vol=$(echo $(( $(awk -F ':|,' '{print $2}' <<< $vol) / 12)))
  if [ $vol = 0 ]; then
  	volIcon=🔇
  else
  	volIcon=🔊
  fi
  volBars='▁▂▃▄▅▆▇█'
  volBG=${volBars:$vol:-1}
  vol=${volBars:0:$vol}
  paste <(printf %s "$art") <(printf %s "") <(printf %s "") <(printf %s "") <(printf %s "") <(printf '%s\n' "$name" "$artist - $record" "$(echo $currMin:$currSec ⎮'\e[00;36m'${bar:1:1}''${bar:1:1}'\033[0m'⎮ $endMin:$endSec)" "$volIcon $(echo "\e[0;32m$vol\033[0m$volBG")")
  sleep 1
  done
}
```
