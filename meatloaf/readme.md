# Meatloaf

## Introduction

Take an ESP32S3, I took the "ESP32S3 DEVKITC 1". 
Connect 6 wires to an IEC DIN connector (I used a female one).
See [directory](https://github.com/idolpx/meatloaf/tree/main/boards) for which pins.
This is scary, because IEC is 5V and ESP32 is 3V3. However, it seems that the ESP pins 
are 5V [compliant](https://github.com/bdring/6-Pack_CNC_Controller/issues/31) for low currents.
Finally flash firmware, this can be done with PlatformIO (Arduino via VSCode) or 
the [web](https://flash.meatloaf.cc). I had some trouble find the correct image (sorry, forgot).


## Links

- [GitHub](https://github.com/idolpx/meatloaf)
- [Server](https://meatloaf.cc)
- [Short codes](https://meatloaf.cc/sc/s/shortcodes.php)
- [Documentation](https://github.com/idolpx/meatloaf/wiki)
- [Flash your EPS via the web](https://flash.meatloaf.cc)


## Usage

This is my list, see also [documentation](https://github.com/idolpx/meatloaf/wiki/Using-Your-Meatloaf).

- What is good to know is that Meatloaf `LOAD`s directories.
  If `LOAD "$",8` and then `LIST` shows there is a directory `GAMES`, 
  a next command could be `LOAD "GAMES",8` and `LIST`.
  If there is a program `HERO`, a next command could be `LOAD "HERO",8`.
  Then you would finally do a `RUN`.

  The `LOAD` is an implicit `CD`.

  To go up, see [traverse documentation](https://github.com/idolpx/meatloaf/wiki/Using-Your-Meatloaf#traverse-your-file-system), didn't try this yet.

  It seems that Meatloaf treats archives (like zip files) as directories.

- Some files are stored in the flash area of the ESP (by lack of an attached SD card).
  `LOAD "$",8` shows the files, `LOAD "WIFI",8` loads the program to setup the 
  WiFi credentials in the ESP (in the Meatloaf), `LOAD "FB",8` loads the file browser.

  It seems that the Meatloaf device recognizes speedloaders on the C64 side and 
  addopts to it.

- The first thing I encountered about Meatloaf, was in a presentation:
  `LOAD"ML:TELETEXT64",8`. This loads a teletext browser from the Meatloaf server.
  This server has many more applications, you find them via `LOAD "ML:$",8`.

  What is also important to know, the teketext browser uses the Meatloaf device to 
  retrieve teletext pages from various internet sites.

- Trying a d64 on some webserer. There is a d64 file at
  [https://github.com/maarten-pennings/C64howto/raw/refs/heads/main/recursion/recursion.d64](https://github.com/maarten-pennings/C64howto/raw/refs/heads/main/recursion/recursion.d64).
  That string is too long for a BASIC line, but we can cut it in two

  ```basic
  A$="HTTPS://GITHUB.COM/MAARTEN-PENNINGS/C64HOWTO/RAW/REFS/HEADS/MAIN/"
  LOAD A$+"RECURSION/RECURSION.D64",8
  ```

- Because URLs are a bit long, the Meatloaf server has [short codes](https://meatloaf.cc/sc/s/shortcodes.php).
  I made a short code `C64HOWTO` that maps to `https://github.com/maarten-pennings/C64howto/raw/refs/heads/main`.
  This allows `LOAD "ML:C64HOWTO/RECURSION/RECURSION.D64",8`.

- There is a browser extension, e.g. for Chrome 
  [send-to-meatloaf](https://chromewebstore.google.com/detail/send-to-meatloaf/dofemlliemmbfmdbbjfpdaooaklfmdki).
  I'm not sure yet how this works, but somehow this sends the URL to the Meatloaf server.
  A `LOAD "ML:*",8`, then loads that file.

- The Meatloaf device has an _internal website_, the Meatloaf Manipulator.
  On a PC browser open `meatloaf.local`.
  In the top-left, we see a folder icon. Clicking that shows the filesystem in the flash filesystem.
  On the PC, drop a file in the browser, and the file will be uploaded to the flash filesystem of the meatloaf device.
  This file can then be `LOAD`ed.

- On the website you can also select another device (e.g. 9 instead of 8), and map that to a URL.
  The IDs 4-30 are supported. It is also a printer and a clock.

- The Meatloaf device _shares its flash filesystem_ via the SMB protocol.
  On a PC enter `net use M: 192.168.178.4`, assuming your Meatloaf has ip number 192.168.178.4.
  Then copy files to the M: drive; these files can then be `LOAD`ed.

- The Meatloaf device has a _telnet server_.
  From a PC log in.
  One of the commands you can give is `wget ...` from somewhere in local network, or world wide internet.
  This files can then be `LOAD`ed.

- The telnet server is also a log device.
  See [documentation](https://github.com/idolpx/meatloaf/wiki/Advanced-Meatloaf-Operations) for debugging via the ESP com port.

- What I did noy yet manage to do, but which should be possible is 
  `LOAD "SMB://*,8` to access shared files from a PC or a NAS.

- Didn't try it yet, but on the Meatloaf server there is a BBS browser.
  Get it via `LOAD"ML:PTERM",8,1`, then run it and use the listed BBSes, or add your own link.

- It seems it is possible to compose a HTTP request from BASIC, and retrieve the response.
  See [documentation](https://github.com/idolpx/meatloaf/wiki/Advanced-Meatloaf-Operations).


(end)
