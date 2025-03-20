# OyasumiUSB
Dead Man's Switch to put your Mac to sleep

You're sitting in a coffee shop using your Mac, maybe zoning out while watching YouTube, and suddenly it's snatched from you and the thief runs off with it.  You're logged in, and all your personal data is available to the thief.  That would suck.  Or perhaps you're a journalist with sensitive data in the same situation.  It sounds a bit clichéd, but such grab-and-runs do take place.

I live in a safe city and my threat model doesn't realistically need to account for this, but some people may need to, so I was thinking of a way to ensure my computer would sleep or power off automatically if taken away from me without my permission. 
I came up with a simple way of creating a USB thumb drive that, when removed, puts your Mac to sleep.

This of course is only useful if you use a password to log in and have set your Mac up to require a password when logging in after sleep.

Here are the steps to create one for yourself:  

**Step 1: Create the Sleep Script in Zsh**
1.	Open Terminal on your Mac.
2.	Create a new script file using the following command:
		
nano ~/sleep_on_usb_removal.zsh

3.	In the nano editor, paste the following script:
		
#!/bin/zsh
pmset sleepnow

4.	Save the file by pressing CTRL + O, then press Enter. Exit nano by pressing CTRL + X.
5.	Make the script executable with the following command:
		
chmod +x ~/sleep_on_usb_removal.zsh

__________

**Step 2: Set Up the USB Thumb Drive**
1.	Plug in your USB thumb drive.
2.	Open Disk Utility (found in Applications > Utilities).
3.	Select your USB drive and choose "Erase." Format it as "Mac OS Extended (Journaled)" or "ExFAT" (if you want cross-platform compatibility). Name it something like SleepUSB.
4.	After formatting, open Terminal again and create a new directory on the USB drive:
		
mkdir /Volumes/SleepUSB

(at this point, I got a message that the directory already existed.  I ignored it.)

5.	Copy the script to the USB drive:
		
cp ~/sleep_on_usb_removal.zsh /Volumes/SleepUSB/

__________

**Step 3: Setting Up the LaunchAgent in the User Library**
1.	Open Terminal and create a plist file in the User Library:
		
nano ~/Library/LaunchAgents/com.user.usb_sleep.plist

2.	Paste the following XML configuration exactly as below.  The spacing is important, so if any issues, please use the plist file on this github:

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.usb_sleep</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/osascript</string>
        <string>-e</string>
        <string>tell application "Finder" to sleep</string>
    </array>
    <key>WatchPaths</key>
    <array>
        <string>/Volumes/SleepUSB</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>

3.	Set Permissions: Ensure the plist file has the correct permissions:
		
chmod 644 ~/Library/LaunchAgents/com.user.usb_sleep.plist

4.	Load the LaunchAgent: Use the following command to load the LaunchAgent:
		
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.user.usb_sleep.plist

__________

**Steps to Set Up the Automator Quick Action**
1.	Launch Automator from your Applications folder.
2.	Choose "New Document" and select "Quick Action."
3.	Set "Workflow receives" to "no input" in "any application."
4.	In the left sidebar, search for "Run Shell Script" and drag it into the workflow area.
5.	By default, the shell script action may contain "cat". You should delete that line and replace it with the following command:
~/sleep_on_usb_removal.zsh
This command will execute the script you created earlier to put your Mac to sleep.
6.	Go to File > Save, and name your Quick Action something like "Run Sleep Script."
Final Check
7.  Verify the Script Path: Make sure that the script sleep_on_usb_removal.zsh is located in your home directory. You can check by running:
ls ~ | grep sleep_on_usb_removal.zsh
If it's not there, ensure you place it in your home directory or adjust the path in the Automator action accordingly.
8.  Testing the Quick Action - This setup should allow you to manually trigger the sleep script using the Automator Quick Action. 
	•	Open the Services menu (found in the application menu or by right-clicking) and look for "Run Sleep Script."
	•	Select it to see if it successfully puts your Mac to sleep.

__________

**Sanity checks if something goes wrong.**
1. Ensure that the plist file is correctly formatted. You can use the plutil command to check for syntax errors:
plutil -lint ~/Library/LaunchAgents/com.user.usb_sleep.plist
2. Ensure the plist file has the correct permissions. Run:
chmod 644 ~/Library/LaunchAgents/com.user.usb_sleep.plist
3. Ownership: The ownership should be your user account. You can check it with:
ls -l ~/Library/LaunchAgents/com.user.usb_sleep.plist

If you made changes to the plist file or want to restart the LaunchAgent, you can unload and then load it again:
1.	Unload the LaunchAgent:
launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/com.user.usb_sleep.plist
2.	Load it Again:
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.user.usb_sleep.plist
