# systemd-user-units

# systemd-xinit
Units, (as well as scripts) which try to align to the somewhat limited documentation regarding systemd __user__ units.
These will most likely need a revamp when fedora switches to systemd >= 247.

## Flow
I've spent alot of time trying to circumvent the limitations of systemd in combination with Xorg due to the descissions taken in/from https://lists.x.org/archives/xorg-devel/2014-February/040476.html, in combination with systemd transitioning to cgroup v2, where the user does not own their login session (scope). Hence this was the most clean/systemd-alike approach i could think of:

1. systemd-xinit is executed, preferably through aliasing *sysx* in your *.<sh>rc*-file to
   ''' alias sysx="exec systemd-xinit" '''
   prefixing the command with exec will prevent returning to console when terminating, which can be considered a __BIG__ security issue.

2. a. a transient systemd unit (xinit.service) will be created, which will be picked up in step (3).
   b. the process will then exec/replace itself with the xserver executable,
      using the environment variable *XDG_VTNR* (passed by systemd on local tty's) as vt#.
      this is required, since systemd/logind needs an active process in the current VT session to be respected as active,
      but as well for the current Xorg process to be able to tie to an active user session (https://lists.x.org/archives/xorg-devel/2014-February/040476.html).

3. the transient unit will (re)-execute *systemd-xinit* again, but this time with the environment variable *XSERVERPID* set.
   a. this causes the script to wait for an available/active display number on stdin (as defined by the -displayfd argument to Xserver).
   b. it will then connect as a (bare minimum) xclient to the xserver to verify connectivity, as well to to keep the session alive while the session is brought up.
   c. after a succefull connection to the xserver, we will signal systemd to carry on with the specified ExecStartPost ([XDG_SESSION_DESKTOP]-session-pre.target).
   d. after [timeout] seconds, the simple xclient will disconnect causing the server to terminate if no application(s) has connected.

4. everything should now be closed as expected when shutting down (eg. by issuing *systemctl --user stop graphical-session.target*)

# WIP
