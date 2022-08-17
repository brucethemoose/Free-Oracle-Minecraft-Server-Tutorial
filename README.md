# What? A *free* Minecraft server?

Yes, Oracle will give you 4 good cores, 24GB and a big SSD for free. This is better than Aternos or even some paid hosts, but you have to configure everything yourself.

Follow this Oracle guide **to the letter**: https://blogs.oracle.com/developers/post/how-to-set-up-and-run-a-really-powerful-free-minecraft-server-in-the-cloud

But there are gaps in the guide, particularly if you want to squeeze every drop of performance out of it for a modded and/or populated server.

# Selecting a Virtual Machine Instance

- If you intend to run a modded server (or a highly populated vanilla server), when you get to this section, ignore Oracle's advice and opt for 4 cores and 12-24GB RAM.
- Select the latest version of Oracle Linux as your image, which it doesn't always default to:
![Oracle9](https://user-images.githubusercontent.com/8422224/185014138-54e002e2-e101-4c58-a94a-778755d0a2e1.PNG)
- Scroll down to the "boot volume" tab. Select between 75GB and 200GB of storage (which is the limit for free instances), and move the "VPU" slider all the way to the right:
![Storage](https://user-images.githubusercontent.com/8422224/185015099-36c819f0-940c-4fe0-a336-a2f2cba52364.PNG)

# SSH

The "Connect to the Running VM in the Cloud" instance points you to an article about SSH clients. But for a Minecraft server, you probably want VSCode as your SSH terminal, as it makes manipulating files on the server easy.

- Download VSCode: https://code.visualstudio.com/
- Go to the "Plugins" tab on the left and install the "Remote SSH" plugin.
- Install an SSH client per the instructions here: https://code.visualstudio.com/docs/remote/troubleshooting#_installing-a-supported-ssh-client
- Press `F1`, start typing `SSH Open`, and open your ssh configuration file. If you have more than one, edit the one in your user folder:  
  ![SSH](https://user-images.githubusercontent.com/8422224/185022082-1406c5ae-5a9a-40c8-968f-efb886b26190.PNG)
- Enter the public IP of your server, and the path to your SSH key you downloaded from Oracle:  
 ![Config](https://user-images.githubusercontent.com/8422224/185022809-e2e88b12-c6f6-42bc-8624-5182c47ed376.PNG)
- To connect with your server, hit the green button in the bottom left of VScode:  
 ![Green](https://user-images.githubusercontent.com/8422224/185023259-17108ceb-73f8-4847-bbe8-7e72e6b034e5.PNG)
- Hit `Connect to host` in the window that pops up, then hit `Oracle`, and confirm any prompts that pop up.
- Now go to the "File" tab on the left and hit "Open Folder", then click "Continue".  
![OpenFolder](https://user-images.githubusercontent.com/8422224/185024808-cbb76aec-ae4c-4e59-8c9e-9a733271d676.PNG)
- Now you can create folders and files, move them from your desktop, and download them off the server from this panel. You can also open and edit files like `server.properties` or mod config files directly in vscode. 



# Setting up your server

- Consider taking a crash course in linux CLI, either from [text guides like this](https://scicomp.aalto.fi/scicomp/shell/) or from a Youtube video. The linux terminal is in the bottom of VSCode in the "terminal" tab, and you can open multiple terminals with the "+" button.
- Java 17 can be installed with the command `sudo yum install java-17-openjdk.aarch64`. You can find other Java versions with `sudo yum search JDK`

# Installing Minecraft

- Modded minecraft servers can be downloaded as complete zips from Modrinth, Curseforge and so on. You can either download them/extract them locally and (slowly) upload the whole folder with VSCode, or you can directly download the zip with `wget (url to zip)` and extract it with `unzip (name of zip file` afterin `cd`ing into the directory you want it in.
- .sh files in modpacks must be made executable with `sudo chmod +x (.sh file). 
- If you want the Minecraft server to auto restart after crashing, you can add a bash `while true...done` loop to the sh file.
- Running the server with an `& disown` at the end of the command, such as `./server-start.sh & disown`, will keep the server running after closing the ssh terminal.

# Performance Tips

- Start the server/script with the prefix `sudo nice -n -18` to ensure the server gets priority over other processes. 
- Set `sync-chunk-writes=false` in your server.properties file, and use a backup mod like FTB Backups, as you should do that anyway.  
- These are my current java arguments, though some (including zgc) are being benchmarked as I type this: `-server -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=100 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=16M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:MaxTenuringThreshold=1 -XX:ConcGCThreads=3 -XX:+ExplicitGCInvokesConcurrent -XX:G1RSetUpdatingPauseTimePercent=12 -XX:+PerfDisableSharedMem -XX:+UseStringDeduplication -XX:+UseFastUnorderedTimeStamps -XX:AllocatePrefetchStyle=1 -XX:+OmitStackTraceInFastThrow -XX:ThreadPriorityPolicy=1 -XX:+UseNUMA -XX:-DontCompileHugeMethods`
- TODO: Running GraalVM EE instead of OpenJDK can provide a ~15% speedup. But as of this post, it us [missing](https://blogs.oracle.com/developers/post/how-to-install-oracle-java-in-oracle-cloud-infrastructure) from OCL9's repos, so it has to be [manually installed](https://docs.oracle.com/en/graalvm/enterprise/22/docs/getting-started/index.html), and you have to use GraalVM 22.1.0 instead of the latest release.
- TODO: [Huge Pages] provides a nice speedup. but Oracle's default security configuration seems to prevent *either* implementation from working. `-XX:+UseTransparentHugePages` in particular seems to silently fail.  
- TODO: Kernel and IO tuning.
