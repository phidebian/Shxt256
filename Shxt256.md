
## Shxt256

Source-highlight for color capable terminal emulators..

This tiny project allows you to have a generalised access to colors used by
**source-highlight** for color capable terminal emulator like **xterm,
gnome-terminal, mate-terminal, terminator**, etc..

This is useful for **GDB** user's who can then easily customize their
source color in the CLI/TUI

## TL;DR
To use it directly in **GDB**, Once installed (cloned) simply do

 
```
$ cd InstallDir
$ export SOURCE_HIGHLIGHT_DATADIR=$PWD
$ gdb -tui YourProg

OR

$ cd InstallDir
$ SOURCE_HIGHLIGHT_DATADIR=$PWD gdb -tui YourProg

OR

Hide the above in a wrapper.

```

**GDB** will source-highlight your source code with the predefined colors.
You can then hack the TH-default.th file to change colors that suite your
needs. (see `Shxt256 gen-theme`).

## INSTALL

To install, simply git clone this project.

```sh
$ git clone https://github.com/phidebian/shxt256 InstallDir
```

This come with default setup (ColorMap, Theme, Languages) suitable for
a direct **GDB** use.

## WHY SHXT256

Initially to fix **GDB** CLI/TUI source rendering. **GDB** use
**source-highlight** to colorize the source code, which in turn
produce escape sequence suitable for color terminals.

In this whole chain, there are glitches that we try to workaround
here.

```text

+------------------+
|      SHELL       |  export SOURCE_HIGHLIGHT_DATADIR=InstallDir
+------------------+
         |
+------------------+
|       GDB        |  CLI/TUI use hard coded esc.style esc.outlang
+------------------+  provided by source-highlight.
         |
+------------------+
| source-highlight |  source-highlight use SOURCE_HIGHLIGHT_DATADIR to
+------------------+  access .lang .outlang .style.
         |            It use esc.style and esc.outlang to generate
         |            escape sequences for the X terminal.
         |            esc.outlang define a colormap that map color
         |            name on terminal color 'number'.
         |            The possible color names are Hard coded in
         |            source-highlight and the list is very limited (see
         |            SOURCE-HIGHLIGHT below)
         |            
+------------------+  
|    Xterminal     |  Display GDB source code with color and attributes
+------------------+  escape sequences rendering.
                      When accessing color by index [0..255] then the
                      color used for each index as a default color, but the
                      user can redefines them, so color #0 is
                      traditionally "black" but can setup to anything
                      else, and this is true for all indexes. Because
                      the default color for indexes [0..15] are
                      generally ugly many people redefines them to
                      smoother colors. Some redefine colors [0..32],
                      and theorically could define the whole range.
```

## HELP
In the following we will use `Shxt256` from the `InstallDir`
You can request the help like this
```sh
$ ./Shxt256 -h
$ ./Shxt256 help
```

## SOURCE-HIGHLIGHT
Source-highlight *.lang* files define the Language Element (LE's) of a
given language (C for instance).

Source-highlight *.style* file define what color and attribute to use
per LE's.

The color names usable in a *.style* file is very limited, it is hard
coded, i.e "green", "red", "darkred", "blue", "brown", "pink",
"yellow", "cyan", "purple", "orange", "brightorange", "darkgreen",
"brightgreen", "black", "teal", "gray", "darkblue", "white".

Source-highlight *.outlang* define how to render the output, for
terminal this is *esc.outlang* or *esc256.outlang*.

The *.outlang* file defines a color map, that map the few legal color
name found in *.style* to colors and attribute, but here again some
*.outlang*, like *esc.outlang*, only define a subset of the limited
subset of colors. 

To work around all those limitations, we define **xt256.outlang** wich
define how we emit attributes to the terminal in a generalised manner.

We then define our own color map mechanism with no limit on color
names, and color values (See COLORMAP).

On top of color map we define our own theme mechanism that define a
color name for each LE's found in all languages. This way if many
language define the *variable* LE, we can have a homoneous color mapped
to the *variable* for all languages. (See THEME)

## XTERMINALS
You have to decide if you want to use the Xterminal color indexes or
true color #RRGGBB. The later use long escape sequences, so may be not
too good on slow lines, yet this tends not to be a problem
nowaday. With color indexes, escape sequences are shorter, but then you
got to decide if you want to use the inherited color index color map,
or if you prefere to reset the terminal so the color for each index
are always the same color.

In its default usage **shxt256** use the inherited color indexes
colors, this is because it is very common for users to defines their
own colors for at least indexes [0.15], so we inherit that.

In order to check what colors are displayed for color indexes you can
use

```sh
$ ./Shxt256 show-indexes
$ ./Shxt256 si

```
The color index can then be used in your own color map. (See below).

## COLORMAP
The color maps are named **CM-ColorMap.cm** The **CM-** namespace is
used to differenciate shxt256 files (start with uppercase) with
source-highlight distribution files (lower cases).

The color map will map your own color name to Xterminal color. We have
predefined color maps.

```sh
$ ls *.cm
CM-html.cm  CM-minecraft.cm  CM-roblox.cm  CM-x11.cm
```

It is very easy to create one, you can look at `Shxt256 import` for
examples that import colormap from the web.

The CM-ColorMap.cm sytax is a `bash` sourced file like this
```sh
#List of X11 RGB Colors (/usr/share/X11/rgb/txt)
name+=(NavyBlue              ) ; color+=(\#000080)
name+=(DodgerBlue4           ) ; color+=(\#104e8b)
name+=(DodgerBlue3           ) ; color+=(\#1874cd)
name+=(MidnightBlue          ) ; color+=(\#191970)
```

In this example the colors are given by #RRGGBB true color code.

It is possible to give the colors by index like

```sh
name+=(Red)    ; color+=(1)    # Small index [0..7]
name+=(Yellow) ; color+=(@154) # Big   index [0..255]
```
Note that small index 1 is the same as big index @1 except that escape
sequence sent for @1 is longer. It is perfectly ledgit to mix'n'match
all the color specifications.

When a CM-ColorMap.cm is defined it is possible to see its rendering
with

```sh
$ ./Shxt256 show-colormap CM-x11.cm
$ ./Shxt256 show-colormap CM-x11
$ ./Shxt256 show-colormap x11
$ ./Shxt256 sc html
```

## THEME
A theme TH-theme.th file is a mapping between language elements (LE's)
defined in Language.lang files and a color name found in a
CM-ColorMap.cm file.

The structure is of a TH-theme.th file is.
```
// Set a ColorMap.cm to your liking on the next line.
// ColorMap=CM-x11.cm
ColorMap=CM-html.cm
and_but              Normal ;
argument             Normal ;
assignment           Normal ;
```

We have a predefined theme `TH-style1.th`

To create your own theme, yo do
```sh
$ ./Shxt256 new-theme foo
$ ./Shxt256 nt foo
```

This creates TH-foo.th from a pristine template, all the LE's are set
to `Normal;` meaning not rendered.

Then you edit TH-foo.th, uncomment a color map of your choice, or add
one you have created.

Then you assign color names from the color name set defined in yout
CM-ColorMap.color to LE's given in your TH-foo.th.

Now you can create a TH-Theme.style file (See STYLE)

## STYLE

A `.style` is consumed (needed) by source-highlight, you generate one
from a TH-Theme.th file

```sh
$ ./Shxt256 gen-style TH-foo.th
$ ./Shxt256 g TH-foo.th
$ ./Shxt256 g TH-foo
$ ./Shxt256 g foo

$ ./Shxt256 g style1
TH-style1.style is generated, you can use it with source-highlight.
```

The generated *.style* file is named after the theme name, so TH-foo.th
produce TH-foo.style, the later is the one we will provide to
source-highlight(1).

**NOTE:** It is important to regen your `.style` file from your `.th`
file after each`.th` modification. This check can be enforced in a wrappers
for source-highlight(1), and the same for *GDB(1)*.

When your `.style` is generated you can check its rendering with

```sh
$ ./Shxt256 show-style TH-style1.th
$ ./Shxt256 show-style TH-style1
$ ./Shxt256 show-style style1
```
**NOTE:** LE's set as "Normal;" are not displayed, so doing a
`show-style` on a pristine theme display nothing.

## CHECKING
You can check your latest TH-style1.style rendering on a source file
like this
```sh
$ export SOURCE_HIGHLIGHT_DATADIR=$PWD
$ source-highlight -f xt256 --style-file=TH-style1.style -i ~/hw/hw.c 
```
**NOTE:** `xt256` i.e `xt256.outlang` is the file we use for rendering
escape sequences for Xterminals.

## GDB
Because **GDB** use harcoded `esc.outlang` and `esc.style` to access
source-highlight we got to symlink them to our setup, this is done by
doing.

```sh
$ ./Shxt256  gdb link style1
$ ll esc.*
lrwxrwxrwx 1 phi phi  13 Mar  9 06:49 esc.outlang -> xt256.outlang
-rw-r--r-- 1 phi phi 949 Mar  6 10:39 esc.outlang.ori
lrwxrwxrwx 1 phi phi  15 Mar  9 06:49 esc.style -> TH-style1.style
-rw-r--r-- 1 phi phi 660 Mar  6 10:39 esc.style.ori
```

At this point *GDB(1) will use your own *.style* file with the
*xt256.outlang* terminal rendering escape sequences.


From there you can run GDB with to proper environment setup
```sh
$ export SOURCE_HIGHLIGHT_DATADIR=$PWD
$ gdb -tui ~/hw/hw

$ SOURCE_HIGHLIGHT_DATADIR=$PWD gdb -tui ~/hw/hw

$ Gdb -tui ~/hw/hw # Wrapper (see below)
```

**NOTE:** The export can be setup in a *Gdb* wrapper, along with
a check that TH-Theme.th and TH-Theme.style are in sync.

## FILES
In the **InstallDir** we find the current source-highlight files, most of them are lowercase.

Our files are uppercase except when notified.

- `CM-ColorMap.cm` are color map files
- `TH-Theme.th` are theme files
- `TH-Theme.style` are generated style files
- `xt256.outlang` is out output driver for source-highlight.
- `c.lang.ori` is the original source-highlight file while `c.lang` is
  our hacked version, it mostly add a rule so the `label` following a
  `goto` produce a `label` LE, and then will match the color of a
  `label:` statement.
- Help.lang define a tiny language used by shell script for self
  documenting.   
