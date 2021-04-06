# systemd-user-units
systemd **user** units and scripts which try to adhere to the somewhat limited (work-in-progress) design/guidelines concerning user units.

## Installation
```
git clone https://github.com/pierrelarsson/systemd-user-units.git /usr/local/lib/systemd
```

### systemd-xinit
systemd-xinit implements what *startx|xinit* does, but for/through systemd.  

A transient *xinit.service* unit will be created, which will wait for the Xserver to report the acquired display-number (man:Xserver(1) -displayfd).  
Finally it will call *exec()* to replace itself with the Xserver-binary.  

When the transient *xinit.service* unit receives the display-number, it will try to establish a connection to the Xserver to validate connectivity.  
This connection also serves the purposes of preventing the Xserver from terminating on provisional connections (eg. from xrdb, xset, setxkbmap),  
but also to **cause** the Xserver to terminate if no permanent connection has been made during the *timeout* (default 10) seconds.  

The *[DISPLAY]* environment variable will be set/imported into the systemd user instance,  
followed by ExecStartPost(s) to **sequentially** start the *[XDG_SESSION_DESKTOP]*-session-pre.target and *[XDG_SESSION_DESKTOP]*-session.target

#### Setup
Make sure to set your desired graphical session via the *XDG_SESSION_DESKTOP* environment variable. eg.
```
echo XDG_SESSION_DESKTOP=x11 > ~/.config/environment.d/99-graphical-session.conf
```

#### Xserver
The script (systemd-xinit) defaults to execute /usr/libexec/Xorg, which should be **non** SETUID (at least on Fedora systems).

#### Trivia
I'd prefered to implement this through socket activation,
but due to the descissions taken in https://lists.x.org/archives/xorg-devel/2014-February/040476.html,
combined with systemd transitioning to cgroup v2 where users does not own their login session (scope), this is simply just not possible.
However we'd still be required to have (at least) one process running in the logind session to be considered active (which in this case will be the Xserver itself).

### x11-session-pre.target & x11-session.target
Generic targets, which will bind to graphical-session-pre and graphical-session respectively.
Requires xhost.service, xrdb.service & xkbmap.service, as well as expecting a service unit to be aliased as *window-manager.service*.
Also defines **Wants** dependencies on *x11-notifications.service* and *x11-screensaver.service*, eg.
```
systemctl --user enable i3.service xscreensaver.service xfce4-notifyd.service
```

### x11.socket & xorg-x11.service
Units that **could** have been used for X socket activation if *GetSessionByPID* had not been used whitin X.org server (as described above).
Alternatively if the user had full access to their own logind session scope (cgroup), the unit could have been started/migrated into that scope.

### xsettings.service
Oneshot service to execute $HOME/.Xsettings and $HOME/.Xsettings-$HOSTNAME, which may contain user preference options (eg. xset, xsetroot, xsetpointer, xrandr, xinput etc.)

### blueman-applet.service
A tray applet for managing bluetooth.

### i3.service
An improved dynamic, tiling window manager.

### insync.service
An alternative Google Drive and Microsoft OneDrive client for Linux (among others).

### krb5-renew.service & krb5-renew.timer
Renew Kerberos ticket-granting ticket, with accompanying timer to periocially trigger the service.

### xfce4-notifyd.service
Notification service for the Xfce Desktop Environment, but works well with other as well (eg. i3).

### nm-applet.service
NetworkManager applet.

### pasystray.service
PulseAudio controller for the system tray.

### nvidia-settings.service
Configure the NVIDIA graphics driver.

### rexterm.service
**Re**starting XTerm service which I use for my i3 window manager scratchpad.

### x11vnc.service
Allow VNC connections to the Xserver session.

### xhost.service
Limits X11 access to the local/current $USER.

### xkbmap.service
Set the keyboard using the X Keyboard Extension.  Conflicts with xmodmap.service.

### xmodmap.service
Utility for modifying keymaps and pointer button mappings in X.  Conflicts with xkbmap.service.

### xrdb.service
Oneshot service to update the X server resource database from /etc/X11/Xresources and $HOME/.Xresources.

### xscreensaver.service
Extensible screen saver and screen locking framework.

### gnome-keyring-pam.path & gnome-keyring-pam.service
Initialize a gnome-keyring-daemon started via PAM (pam_gnome_keyring.so).  
Also migrates that process into the service cgroup (user.slice/user-$UID.slice/user@$UID.service/gnome-keyring-pam.service).

### gnome-keyring-daemon.service
Manually start the gnome-keyring daemon. Be aware that you'll have to authenticate when using this.
