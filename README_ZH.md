Magisk Detector
==============================

Anti Magisk Hide
-------------
The core of Magisk Hide is to mount the namespace. After Magiskd waits for the separation of the mount namespace of the zygote child process from the parent process, it unloads all Magisk mounted content from the child process. Due to the nature of the mounting namespace, the unload operation will affect the child process of this child process, but will not affect zygote. zygote is the parent process of all application processes. If zygote is processed by Magisk Hide, all applications will lose root. When zygote starts a new process, there is a `MountMode` parameter. When it is `Zygote.MOUNT_EXTERNAL_NONE`, the new process does not mount the storage space, so there is no mounting namespace separation step.

There are two cases of compliance, one is that the appops read storage space op is ignored, and the other is that the process is an isolated process. The former application itself cannot be operated, and the latter has been supported since Android 4.1. In addition, an interesting feature of the isolation process is the random UID, which is different every time it runs. This interesting feature causes Magisk Hide to skip this process and not process it before detecting the mounted namespace.

Detect Magisk module
-------------
Although the Magisk module can be hidden on the file system, the modified content has been loaded into the process memory, and it can be found by checking the process maps. The data displayed by maps contains the device where the file is loaded. The Magisk module will cause the path of some files to be in the system partition or vendor partition, but the device location displayed is the data partition.

Detect MagiskSU
------------
Under normal circumstances, applications cannot connect to sockets that were not created by themselves, but Magisk modified SELinux. All applications can connect to sockets in the magisk domain. Each Magisk su process will create a socket and try to connect all the sockets. The number of sockets that are not rejected by SELinux is the number of su processes. The reliability of this detection method depends entirely on the strictness of the SELinux rules. If the Android version is too low or too high, there will be problems.

Test SELinux policy
--------------
SU software must pay attention to the following principles when modifying the SELinux policy, otherwise it will leave security loopholes and ways to be detected.
- One of the source domain and the target domain is the custom domain of the su program, and it is safe to add rules;
- Both parties are non-application domains and need to have reasonable and necessary reasons;
- One side is the application domain, and adding rules is very dangerous. If it is the source domain, you should refuse to add it.

Detect init.rc modification
--------------
Randomness is only effective if it cannot be traversed. If it can be traversed, use statistical methods to find out exactly what is different every time.
