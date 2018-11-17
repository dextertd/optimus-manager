optimus-manager
==================

This Linux program provides a solution for GPU switching on Optimus laptops (a.k.a laptops with dual Nvidia/Intel GPUs).

Only Xorg is supported (no Wayland), and only Archlinux (plus derivatives like Manjaro) is supported for now.


The Why
----------

On Windows, the Optimus technology works by dynamically offloading rendering to the Nvidia GPU when running 3D-intensive applications, while the desktop session itself runs on the Intel GPU.

However, on Linux, the Nvidia driver provides no such offloading capabilities ([yet](https://devtalk.nvidia.com/default/topic/957981/linux/prime-render-offloading-on-nvidia-optimus/post/5276481/#5276481)), which makes it difficult to use the full potential of your machine while kepping a reasonable battery life.

Currently, if you have Linux installed on an Optimus laptop, there are three methods to use your Nvidia GPU :

- **Run your whole X session on the Intel GPU and use [Bumblebee](https://github.com/Bumblebee-Project/Bumblebee) to offload rendering to the Nvidia GPU.** While this mimic the behavior of Optimus on Windows, this is an unofficial, hacky solution with three major drawbacks : 1. a small performance hit (because Bumblebee has to use your CPU to copy frames over to the display) 2. no support for Vulkan (thus, it is incompatible with DXVK and any native game using Vulkan, like Shadow of the Tomb Raider for instance) 3. you will be unable to use any video output (like HDMI ports) connected to the Nvidia GPU, unless you have the open-source `nouveau` driver loaded to this GPU at the same time.

- **Use [nvidia-xrun](https://github.com/Witko/nvidia-xrun) to have the Nvidia GPU run on its own X server in another virtual terminal**. Not optimal for performance since you have two X servers running at the same time. Also, you do not have acess to your normal desktop environment while in the virtual terminal of the Nvidia GPU, and in my own experience, the nvidia driver likes to crash when switching between virtual terminals.

- **Run your whole X session on the Nvidia GPU and disable rendering on the Intel GPU.** This allows you to run your applications at full performance, with Vulkan support, and with access to all video outputs. However, since your power-hungry Nvidia GPU is turned on at all times, it has a massive impact on your battery life.
This method is often called Nvidia PRIME, but technically PRIME is just the technology that allows your Nvidia GPU to send its frame to the built-in display of your laptop *via* the intel GPU.

An acceptable middle ground could be to use the third method *on demand* : switching the X session to the Nvidia GPU when you need extra rendering power, and then switching it back to Intel when you are done, to save battery life.

Unfortunately the X server configuration is set-up in a permanent manner with configuration files, which makes switching a hassle because you have to rewrite those files every time you want to switch GPUs. You also have to restart the X server for those changes to be taken into account.

The present tool does that for you : it dynamically writes the X configuration at boot time, rewrites it every time you need to switch GPUs, and also loads the appropriate kernel modules to make sure your GPUs are properly turned on/off.

Note that this is nothing new : Ubuntu has been using that method for years with their `prime-select` script.

In practice, here is what happens when switching to the Intel GPU (for example) :
1. Your login manager is automatically stopped, which also stops the X server (warning : this closes all opened applications)
2. The Nvidia modules are unloaded and `nouveau` is loaded instead to switch off the card (this can also be done with `bbswitch` if `nouveau` does not work)
3. The configuration for X and your login manager is updated (note that the configuration is saved to dedicated files, this will *not* overwrite your own configuration files)
4. The login manager is restarted.


I am well-aware this is still a *hacky* solution. I will happily deprecate this tool the day Nvidia implements proper GPU offloading in their Linux driver.