---
Title: AppleScript to Toggle the Mac Desktop
categories: [Code] # comma separated array
tags: [applescript] # comma separated array. TAG names should always be lowercase
---

Here's an AppleScript I use to quickly toggle desktop visibility for taking screenshots and recording screencasts that I thought might be useful for others.

Copy and paste the code into AppleScript Editor, or your editor of choice, compile and save. As always, scripts like this work best using <a href="http://www.red-sweater.com/fastscripts/">FastScripts</a> from Red Sweater Software.


````applescript
on run
    tell application "System Events"
        set frontMostApp to name of the first process whose frontmost is true
    end tell
    try
        set theDefault to ((do shell script "defaults read com.apple.finder CreateDesktop") as integer) as boolean
    on error -- if the default value doesn't already exist, create it...
        do shell script "defaults write com.apple.finder CreateDesktop 1"
        set theDefault to ((do shell script "defaults read com.apple.finder CreateDesktop") as integer) as boolean
    end try
    do shell script "defaults write com.apple.finder CreateDesktop " &amp; (((not theDefault) as integer) as string)
    tell application "Finder" to quit
    delay 1
    tell application "Finder" to launch
    tell application frontMostApp to activate
end run
````


*Note: This single script turns off the desktop if it's on, and turns it on if it's off - just to clear up the question should it need to be asked.*