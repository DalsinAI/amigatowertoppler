Tower Toppler for AmigaOS 3.x
=============================

A port of Tower Toppler 1.3 (Andreas Röver's remake of Nebulus, SDL 2,
https://gitlab.com/roever/toppler, commit
c8bf02b0f8bf8875a454758227bf2bfc029487c3), for AmigaOS 3.2 on a 68040 or
68060 with an FPU and an RTG screen, on GFOpenGPU's SDL 2.

Status: runs in an AmiCradle lab (a copy of Instance-32, AmigaOS 3.2);
see "Testing" at the end.


What you need
-------------

- A 68040 or 68060 with an FPU (not an LC040 or EC040), AmigaOS 3.2, an RTG
  card (or an AmiCradle instance).
- GFOpenGPU: gfopengpu.library and LIBS:GFOpenGPU/SDL2.module and
  SDL2_mixer.module (the Ogg Vorbis title music and the WAV sounds),
  installed by OpenUp.
- About 2.5 MB of disk. toppler.dat and toppler.ogg (the game's data and
  music, GPL like the game, see Licences.txt) are in the drawer.


How to play
-----------

Run Install-Toppler from its icon (it asks where the Toppler drawer goes,
checks the CPU, the FPU and GFOpenGPU's modules, and keeps your settings
and high scores on an update), or copy the Toppler drawer anywhere.
Double-click toppler, or from a Shell:

    cd Toppler
    toppler [-f] [-s] [-dN]

-f opens a 640x480 screen of its own instead of a window on the
Workbench (the game remembers it in its settings; Options, Graphics,
Fullscreen turns it off again); -s plays with no sound; -d0 to -d9 print more about what the
game does.

Climb each tower to the door at its top, avoiding (or snowballing) what
lives on it, then catch fish from the submarine on the way to the next.

- Menus: cursor up and down choose, Space or Return takes, Esc goes back
  (and from the main menu quits). Left and right on "Start" pick the
  mission. Left idle, the title shows a demo; any key ends it.
- Game: cursor left and right walk round the tower, up enters a door or
  rides a lift, down rides a lift down, Space jumps (with a direction) or
  throws a snowball; P pauses; Esc opens the game menu (return to the game,
  options, quit game).
- Right Amiga+Q quits at once, from anywhere, as the window's close gadget
  does.
- Options (in the menus) redefine the keys, change the speed and the
  graphics detail; a 68040 may want the cheaper waves and less alpha.

The settings (.toppler.rc), high scores (toppler.hsc) and any missions you
make with the level editor are kept in the drawer's User drawer.


Building
--------

GFOpenAmigaGCC 16.2.0-open6 (the os32-gcc16-open6 stove, which carries the
NDK, GFOpenGPU's SDL 2 kit and zlib):

    make -f amiga/Makefile.amigaos -j2 STOVE=/path/to/os32-gcc16-open6

The default STOVE is ~/AmiCradle/stoves/os32-gcc16-open6. Output:
build/amigaos/Toppler/ (toppler, toppler.dat, toppler.ogg, ReadMe,
ReadMe-toppler, COPYING.txt, Licences.txt, User/, with GlowIcons),
build/amigaos/Toppler.info and Install-Toppler with its icon beside them.
The icons come from amiga/icons/icons.json and the Installer script from
amiga/installer/installer.json (amicradle's scripts/port_icons.py and
port_installer.py). toppler.dat is upstream's prebuilt file: rebuilding it
needs POV-Ray, GIMP and ImageMagick, and isn't done here.


What was changed
----------------

- amiga/Makefile.amigaos: upstream's sources (every src/*.cc) with the
  kit's SDL2 and SDL2_mixer and the stove's zlib; no gettext (English
  text), no global high score folder.
- src/decl.cc, src/decl.h: data_dir() and local_dir(): on the Amiga the
  data is read from PROGDIR: and the player's files go to PROGDIR:User/
  (made when first needed) in place of $HOME/.toppler; elsewhere they are
  upstream's TOP_DATADIR and ~/.toppler. alpha_scandir() puts no second
  "/" after a path that ends in "/" or ":" (AmigaDOS reads "//" as the
  parent drawer). <algorithm> for std::sort.
- src/level.cc: missions are looked for in the archive, PROGDIR:User/ and
  PROGDIR: (not "./", which is no path on AmigaDOS). <algorithm> for
  std::find_if.
- src/highscore.cc: the high score file is local_dir()'s; the lock file
  code is only built with a global high score folder (HISCOREDIR), as
  there is none here.
- src/screen.cc: -f is SDL_WINDOW_FULLSCREEN (a 640x480 screen) rather than
  _DESKTOP (the Workbench's size).
- src/keyb.cc: Right Amiga+Q quits.
- src/main.cc: a missing toppler.dat is said in an Intuition requester
  (amiga/amiga_message.c) and the Shell, not a crash; after SDL_Quit() the
  empty screens SDL2.module leaves are closed or sent behind the Workbench
  (amiga/amiga_screens.c, from the Seven Kingdoms and Abbaye ports).
  <ctime> for time().
- amiga/stack.c: 256 KB of stack (the icon asks for it too) and a $VER
  string.

Frame pacing is upstream's: dcl_wait() holds each frame to its time with
SDL_Delay (55 ms less 5 ms a speed step: 35 ms in the menus, 55 ms in a
game at the default speed).

Big-endian: toppler.dat's directory and its words are read byte by byte
(archi.cc builds each value from its bytes, little-endian on every
machine); the missions are bytes; the pictures are 32-bit surfaces with
explicit masks; the sounds are WAV and Ogg through SDL2_mixer. The high
score file (toppler.hsc) is written as the machine's own structure, so it
is not interchangeable with a PC's; nothing else is.


Known limits
------------

- All drawing is the 68k's: SDL's software surfaces, then one streaming
  texture a frame. With every effect on, a 68040 may not keep up; the
  graphics options turn the costly ones off.
- toppler.hsc from a PC version can't be used (see above).
- The level editor was not tried on the Amiga.
- No translations (gettext isn't in the stove).


Testing
-------

In an AmiCradle lab (t4-port-b, a copy of Instance-32), 5 November 2026,
from SYS:Games/Toppler:

- The NEBULUS title menu in a 640x480 window, in the right colours, with
  the towers and the reflection.
- Start: the submarine comes up to the "Tower of eyes" and the toppler
  gets out; cursor right walks him round the turning tower and off a
  ledge, he falls into the sea, a life goes (three to two) and the tower
  starts again; the time and score count; Esc opens the game menu and
  "Return to Game" goes back.
- Left idle, the title runs the demos (the "DEMO" towers); a key ends them.
- Esc from the main menu quits ("Thanks for playing!"), Right Amiga+Q
  quits during a game, and -f opens a 640x480 screen; each time the
  Workbench was back in front with nothing left over, and
  User/.toppler.rc was written.
- Not heard: the lab is muted, so the music and sounds were not checked
  by ear; the debug log shows the sound system starting with no error.
- Not tried: a whole tower to its door, the bonus game, a high score
  entry, the level editor, a joystick.
- Seen once, on the first start: Space and Return on the title menu did
  not start a game. The title runs a demo after 7 seconds without a key,
  and with the test's slow keypresses the keys landed in the demo instead
  (which a key ends, back to the menu); started again and driven more
  quickly, the menu answered every key.
