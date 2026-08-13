Screenshots for the site.

Hamnix (native) — referenced by ../native.html:
  desktop.png    the desktop at rest — panel, clock, icons
  menu.png       the application menu open
  browser.png    hambrowse rendering its built-in demo page
  terminal.png   hamsh in the terminal
  files.png      the file manager
  sysmon.png     the system monitor
  installer.png  the graphical installer

Hamnix Linux — referenced by ../linux.html:
  linux-menu.png           the Applications menu with a fly-out submenu
  linux-desktop.png        an installed UEFI + ext4 machine, hamsh listing /
  linux-update-notice.png  the "restart before opening new apps" notice,
                           captured on an installed disk
  linux-firefox.png        Firefox as a native Wayland client in a Hamnix window
  linux-steam.png          Steam's store Browse menu, through the X bridge

Every file here is captured from the real framebuffer scanout of a running
system (QEMU `screendump`), not from a host-side render. Host renders
misrepresent proportional text, so a screenshot taken that way is not the
same picture a user sees. Some are cropped to the window that appeared;
none are re-rendered, upscaled, composited or retouched.

Caveat on the Hamnix Linux captures: they come from test harnesses, and the
panel's CPU widget in those frames reads the machine that ran the capture,
not the guest in the picture. linux.html says so next to the images.
