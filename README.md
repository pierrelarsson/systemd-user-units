# systemd-user-units
systemd **user** units and scripts which try to adhere to the somewhat limited design/guidelines regarding user units.

## installation
```
git clone https://github.com/pierrelarsson/systemd-user-units.git /usr/local/lib/systemd
```

### systemd-xinit
systemd-xinit implements what *startx|xinit* does, but through/for systemd.
it will create a transient *xinit.service* unit, which will wait for the Xserver to report the acquired display-number (Xserver -displayfd argument).
thereafter it will call *exec()* to replace itself with the Xserver-binary.
when the transient *xinit.service* unit receives the display-number it will try to establish a connection to the Xserver to validate connectivity as well to keep Xserver from terminating or resetting for *timeout* (default 10) seconds.
the *[DISPLAY]* variable will then be set/imported into the systemd user instance, followed by ExecStartPost(s) to serially start the *[XDG_SESSION_DESKTOP]*-session-pre.target and *[XDG_SESSION_DESKTOP]*-session.target

#### setup
make sure to set your desired graphical session via the *XDG_SESSION_DESKTOP* environment variable. eg.
```
echo XDG_SESSION_DESKTOP=x11 > ~/.config/environment.d/99-graphical-session.conf
```

#### xserver
the script defaults to execute /usr/libexec/Xorg, which should be **non** SETUID (on at least Fedora systems).

#### trivia
i'd prefered to implement this through socket activation, but due to the descissions taken in https://lists.x.org/archives/xorg-devel/2014-February/040476.html combined with systemd transitioning to cgroup v2 where users does not own their login session (scope), this is just not possible.
however we're still required to have (at least) one process running in the session to be considered active (which will be the Xserver itself).

### x11-session-pre.target and x11-session.target
generic targets, which will bind to graphical-session-pre and graphical-session respectively.
it expects a service to beeing aliased as *window-manager.service*. as well as wanting *x11-notifications.service* and *x11-screensaver.service*, eg.
```
systemctl --user enable i3.service xscreensaver.service xfce4-notifyd.service
```
