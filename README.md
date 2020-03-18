### Fix screen resolution on startup using systemd
Tested on Ubuntu 18.04.

A `systemd` unit and associated `bash` script for updating the screen resolution on system startup.

Using tiling window managers like [i3](https://i3wm.org/) can cause the system's screen resolution to be reset to a non-ideal value after every system restart.

#### fixresolution.service
This unit ensures that the specified screen resolution is always set on startup.

It is a `user` service - meaning it belongs to a user - rather than a `system` service.

System services are controlled by `systemctl [action] serviceName.service`. User services are controlled by `systemctl --user [action] serviceName.service.`

User services can be saved in `$HOME/.config/systemd/user/`.

`systemctl` will reset the unit up to 100 times in the first 5 seconds of startup until the script is run successfully (when it returns 0). This is necessary because the script tries to run an X11 command `xrandr` - which requires that `DISPLAY` be set - before X11 has finished initializing the user's X-session.

#### Setup
Place `fixresolution.service` into `$HOME/.config/systemd/user/`, creating directories as necessary.

Place `fixresolution.sh` wherever you'd like in your `$HOME` directory. Make sure it is executable: `chmod u+x fixresolution.sh`.

In `fixresolution.service`:

- update the path `ExecStart=/home/patrick/recurse/bash-scripts/systemd-fix-resolution/fixresolution.sh` to match the path to your copy of `fixresolution.sh`.
- update the `--output eDP1` with your screen's interface (run `xrandr` and look for the label to the left of `connected` - values might be one of `eDP1, HDMI1, HDMI2, DP1, VIRTUAL1`)
- update the `--mode 1920x1080` with your required screen resolution.

Run `systemd --user daemon-reload` to have `systemctl` load the service that was saved in `.config/systemd/user/`.

Check that `systemd` can recognize the service using `systemctl --user status fixresolution`.

Run the service using `systemctl --user start fixresolution` and check the user `journalctl` logs to confirm that it ran without an error: `journalctl --user -u fixresolution -x` where `-x` provides explanatory text for logged events.

Enable the service to run on startup:

```
systemctl --user enable fixresolution
```

The screen resolution will now be set automatically when the machine starts and the user logs in.

#### fixresolutiontimer.timer
The timer is not used here but is a useful way to call a script some time after boot or startup.
