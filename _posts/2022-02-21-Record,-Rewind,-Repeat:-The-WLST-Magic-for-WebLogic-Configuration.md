To simplify the process of setting up a domain, you can record your configuration steps in the Administration Console as a sequence of WLST commands, which can then be played back using WLST.

WLST is a command-line scripting environment designed for managing, creating, and monitoring WebLogic Server domains. It is automatically installed on your system along with WebLogic Server.


You can record the WLST commands that are automatically executed in the background for every action performed in WebLogic by using the WebLogic consoles.

To record the WLST commands from the console, please follow the steps outlined below.


* Go to Preferences→ WLST Script Recording (Preferences is located at the top center bar next to Home and Logout options)
* Fill in the general properties such as base script directory, script file name, append mode etc.
* Press Lock and Edit
* go to Preferences→WLST Script Recording → Control → Start Recording
* Now you can do the actions using the console and after doing the changes come back to the WLST Script Recording and hit "Stop recording"
* The WLST commands recorded will be available in the file specified in "General tab", it will also be shown in the window.


The Administration Console does not capture WLST commands related to the following:

- Alterations to security information managed by a security provider, such as the addition or removal of users, roles, and policies.
- Modifications to deployment plans.
- Deployment actions, including installation, uninstallation (deletion), redeployment (updating), starting, and stopping.
- Operations performed at runtime on Control or Monitoring pages, such as the initiation and termination of applications or servers.


