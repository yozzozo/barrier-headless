# barrier-headless
This is a write up on how to run `barrier` (the synergy-core fork) on a headless raspberry pi. It's based on the [work](https://github.com/hishamk/headlesssynergysetup) of [hishamk](https://github.com/hishamk/), describing how to do the same thing for the original synergy.

## Why is this useful?
If you're running multiple machines, none of them being the primary one, and you don't want to have the server machine running all the time just to share mouse and keyboard, this guide might be for you.

## Limitations
The keyboard still sends keys to the host system. I have to look into how to disable that.

## Security considerations
I've read on some forums, that running barrier as service with root privileges is dangerous. So use at own risk.

## More Info
I'm no linux expert. I just customized what I found over at [hishamk](https://github.com/hishamk/)'s repo. For more technical details, please visit his repository.

# Setup steps
1. `apt update`
2. `apt upgrade`
3. Compile barrier. See their [wiki](https://github.com/debauchee/barrier/wiki/Building-on-Linux) for instructions.
   * It's important that you compile using the Release build configuration. I couldn't find another way but to edit the clean_build.sh.
4. `apt install xorg xserver-xorg-video-dummy`
5. Change hostname to beautify barrier config definition. Edit /etc/hostname accordingly
6. Create barrier config and copy it to `/etc/barrier.conf`
   * You can create the config on your normal desktop pc. Make shure you convert the line endings to LF if you used a Windows machine to create the file. For some reason barrier didn't like CRLF.
7. Run xorg and Barrier server: `X -config <path_to_your_dummy_config> && barriers -c <your_barrier_config> -f`

# Start headless Barrier on boot
1. Copy `xorg_dummy.config` to `/etc/X11/xorg.conf.d/` or whatever your distro uses
2. Copy `x_dummy.service` and `barrier.service` service files to `/etc/systemd/system/` or wherever your systemd config resides - chown both to root:root
3. Reload systemd: `systemctl daemon-reload`
4. Start X dummy: `systemctl start x_dummy`
5. Check its status: `systemctl status x_dummy`
6. Start barrier server: `systemctl start barrier`
7. Check its status: `systemctl status barrier`
8. Enable both services to start persistently: `systemctl enable x_dummy.service`; `systemctl enable barrier.service`
9. Reboot