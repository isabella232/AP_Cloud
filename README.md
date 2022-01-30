# AP_Cloud
Fleet management solution for ArduPilot Drones

TLDR:  no it doesn't work yet.

Goals - a 'web gui' for:

 - a "drone management" solution
 - ability to easily get logs off the vehicle
 - ability to see drone version info in a web interface 
 - ability to update the firmware on each drone  (with appropriate hardware)
 - search through the logs and their timestamps 
 - download logs for offline review/analysis
 - and display a gui showing how recently each drone had uploaded.
 - run some sort of 'log analyser' on each log after its uploaded for stats/params/faults/etc.
 - perhaps integrate with plot.ardupilot.org , or a copy of it, for a 'plot this log' function

Technical expectations:

 - each drone needs a companion computer such as a 'raspberry pi' or similar running a ssh server
 - each drone needs the ability for AP_Cloud to ssh-into-it to run commands.
 - each drone may need a cronjob to run periodic task/checks , or this may be 'pushed' over ssh.?
 - each drone needs to be able to also ssh "into" the AP_Cloud server for log upload/s


What works?

 - We hava a basic Node.js based web 'server'.  it uses Express. 
 - It has a front-page that lists all the drones, and a per-drone page for each drone.  it doesn't do anything else yet.
 - on the per-drone page, it list all the logs it knows about for that drone and these details: date/time of log, log size, distance travelled, flight time. 
 - We have a configuration system based on config/default.json that lists the configured drones and connection/details for them.
 - The 'server' can run arbitrary SSH commands internally, to-each-drone and push arbitrary files from this 'server' to each drone's 'companion' computer, if they are connected onver wifi. 
 - We have a scheduling system that allows us to run "jobs" periodically, such as 'every minute' or 'every 5 minutes', and it runs some basic jobs.
 - We have 'demo' (harmless ) jobs that will try to ssh to all the configured drones via 'ssh' , unless the config file says "ssh_enable": "0" for that drone.
 - These 'ssh jobs' assume that thee configured private-key will allow the Node server to ssh to the drone without any password, etc.  no password/s are saved, just uses a key.
 - it scans through log files it finds on-disk, and run mavflighttime.py on each of them to determine 'time-in-air' and 'total-distance-travelled' for each log.
 - if, while running, it finds a new/updated file ondisk that wasn't there before, it scans it and add it to the system.
 - log files on-disk must be at least 1 minute old before it will try scanning them.( incase its a in-progress upload)
 - if will re-scan files if their mtime ondisk is newer than the last scan, ie its changed since it was scanned.


Prerequisites to use this:
 - AP_Cloud runs on Linux only right now, sorry.
 - AP_Cloud requires a 'node' version of at least v12  ( we use 'nvm' to switch between node versins, but there are other tools)
 - you know its a work-in-progress , right? 
 - you must have 'mavflighttime.py' in the path. [ its part of mavproxy ], we might bundle it later.


How to use it:
```bash
cd AP_Cloud
npm install
node index.js
```
...Server is running on localhost:8123 ..

Wait for a minute or 2 for it to scan the log files ondisk... you will see progress of this in the console.log output.

visit http://localhost:8123/ in your browser and/or look at hte console after 30 seconds or so to see scheduled tasks being run

Click one of the 'detail' pages for a drone, for eample http://localhost:8123/drone3 and if you have logs in ./logs/drone3/ you'll see info about them listed

How to configure it:
 - Edit the config/default.json file and make it represent your list of ssh-able drones.
 - Edit index.js or any of the other files to make it do more cools stuff, and send a PR to us. 
 - The webpage currently use a simple templating tools called 'pug'.  we're not super attached to pug, but look in ./views/*.pug for the two pages that we have right now.



![pic1](https://github.com/ArduPilot/AP_Cloud/blob/main/screenshot1.png?raw=true)
![pic2](https://github.com/ArduPilot/AP_Cloud/blob/main/screenshot2.png?raw=true)


Note: this software is intended to run "in the cloud" on VM's etc, and as such is licenced slightly differently to the normal GPL. The license is the "GNU Affero General Public License" which means that if you offer a commercial solution based on this software, to users who interact with it over-the-network, your server must also allow them to download the source code corresponding to the modified version running there.

https://en.wikipedia.org/wiki/GNU_Affero_General_Public_License


