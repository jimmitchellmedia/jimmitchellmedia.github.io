---
Title: AppleScript to Add Files in the Mac Finder
categories: [Code] # comma separated array
tags: [applescript] # comma separated array. TAG names should always be lowercase
---

Today I had the need to add a bunch of named text files to a folder in the Finder on my Mac.

I found it a major pain to open BBEdit, make a new document, save it to where I wanted it, and then manually copy &amp; rename the file back in the Finder. A lot of effort just to get 7 or 8 empty files with different names.

So, I threw together this little AppleScript to add empty files instead. The gist is once launched in the Finder (using <a href="http://www.red-sweater.com/fastscripts/">FastScripts</a> by <a href="http://www.red-sweater.com/">Red Sweater</a> naturally. Keystroke: cmd+option+shift+N), a dialog pops up that lets you enter a file title. Then the Finder creates that file in the front-most window. If no window is open, the file is added to your Desktop.

I found this to be more than twice as fast as the "traditional" method of making several files. Hopefully, it can help you out too. Remember, use it with <a href="http://www.red-sweater.com/fastscripts/">FastScripts</a> for quick keystroke access. Save as a script in the "Finder" scripts folder ``("~/Library/Scripts/Applications/Finder/")``.

Here's the code. Just copy and paste into AppleScript Editor and save...


````applescript
property defaultFileName : "newFile.txt"
tell me to activate
set theFileName to text returned of (display dialog "Enter a file name:" default answer defaultFileName)
tell application "Finder"
    activate
    if the (count of windows) is not 0 then
        set theFolder to (folder of the front window) as text
        set theFolder to POSIX path of theFolder
    else
        set theFolder to POSIX path of (get path to desktop)
    end if
    set addedFile to (theFolder &amp; theFileName)
    do shell script "touch '" &amp; addedFile &amp; "'"
    if the (count of windows) is not 0 then
        set addedFile to (POSIX file addedFile) as alias
        select addedFile
    end if
end tell
````