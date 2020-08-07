# Turbo Vision

An independent, backwards-compatible port of Turbo Vision 2.0 for modern Linux systems.

![tvedit on Konsole](https://user-images.githubusercontent.com/20713561/81506401-4fffdd80-92f6-11ea-8826-ee42612eb82a.png)

I started this as a personal project at the very end of 2018. It is now very close to feature parity with the original, and I am not spending much time on it anymore, so I'm making it open in case it may be useful to someone.

Even if the linux port cannot compete against other widespread solutions (such as SET's port, which has tons of additional features), I still consider the improvements made to the Windows implementation to be useful. For example, the TVEDIT application included here can be a good replacement for the missing `EDIT.COM` on 64-bit Windows.

The main goals of this project are:

* Making Turbo Vision work on Linux in the least intrusive way.
* Keeping it functional on DOS/Windows.
* Being as compatible as possible at the source code level with old Turbo Vision applications. This has led me to implement some of the Borland C++ RTL functions, as explained below.

Initially, I planned to add features like full-fledged Unicode support into Turbo Vision. However, I realized that doing so would require either extending the API or breaking backwards compatibility, and that a major rewrite would be most likely necessary.

**Update:** The API has been extended to support Unicode input and output. See <a href='#unicode'> Unicode Support</a> for more information.

As a GUI toolkit, Turbo Vision is pretty much outdated. Modern technologies tend to separate appearance specification from behaviour specification, and Turbo Vision is especially bad at that. Considering all its other limitations, it seems that Turbo Vision cannot fulfill the necessities of modern users and programmers. If some TUI library is to revolutionize our terminal applications in the future, that library won't be Turbo Vision, although it may be inspired by it.

## Build environment

### Linux

Turbo Vision can be built as an static library with CMake and GCC/Clang.

```sh
cmake .
make
```

The above produces the following files:

* `libtvision.a`, which is the Turbo Vision library.
* The demo applications `hello`, `tvdemo`, `tvedit` and `tvdir`, which were bundled with the original Turbo Vision (although some of them have a few improvements).

The build requirements are:

* A compiler supporting C++ 17.
* `libncursesw` (note the 'w').
* `libgpm` (for mouse support in the linux console) (optional).

The minimal command line required to build a Turbo Vision application (e.g. `hello.cpp` with GCC) from this project's root is (notice the shell expansion for the include directories):

```sh
g++ -o hello hello.cpp libtvision.a -Iinclude/{,tvision,override} -lncursesw -lgpm
```

`-Iinclude/tvision` is only necessary if your application uses Turbo Vision 1.x includes (e.g. `#include <tv.h>` instead of `#include <tvision/tv.h>`). `-lgpm` is only necessary if Turbo Vision was built with `libgpm` support.

The `include/override` directory contains headers emulating the Borland C++ RTL (some come straight from Borland). Turbo Vision won't compile without them. This also means that including `tvision/tv.h` will bring several std names to the global namespace.

### Windows/DOS

Turbo Vision can still be built either as a DOS or Windows library with Borland C++.

I can confirm the build process works with:

* Borland C++ 4.52 with the Borland DOS PowerPack.
* Turbo Assembler 4.0.

You may face different problems depending on your build environment. For instance, Turbo Assembler needs a patch to work under Windows 95. On Windows XP everything seems to work fine. On Windows 10, MAKE may emit the error 'Fatal: Command arguments too long', which can be fixed by upgrading MAKE to the one bundled with Borland C++ 5.x.

Yes, this works in 64-bit Windows 10. What won't work is the Borland C++ installer, which is a 16-bit applications. You will have to run it on another environment or try your luck with winevdm.

A Borland Makefile can be found in the `project` directory. Build can be done by doing:

```sh
cd project
make.exe <options>
```

Where `<options>` can be:

* `-DDOS32` for 32-bit DPMI applications (which still works in 64-bit Windows).
* `-DWIN32` for 32-bit native Win32 applications (not possible for TVDEMO, which relies on `farcoreleft()` and other antiquities).
* `-DDEBUG` to build debug versions of the application and the library.
* `-DTVDEBUG` to link the applications with the debug version of the library.
* `-DOVERLAY`, `-DALIGNMENT={2,4}`, `-DEXCEPTION`, `-DNO_STREAMABLE`, `-DNOTASM` for things I have nave never used but appeared in the original makefiles.

This will compile the library into a `LIB` directory next to `project`, and will compile executables for the demo applications in their respective `examples/*` directories.

I'm sorry, the root makefile assumes it is executed from the `project` directory. You can still run the original makefiles directly (in `source/tvision` and `examples/*`) if you want to use different settings.

## Features

### Linux

* Ncurses-based terminal support.
* Mouse and key modifiers support on the linux console.
* Overall better display performance than SET's or Sergio Sigala's ports.
* Reads UTF-8 input from the terminal and displays UTF-8 text, but still works with 8-bit ASCII characters internally.
* Implementation of some Borland C++ RTL functions: `findfirst`, `findnext`, `fnsplit`, `_dos_findfirst`, `_dos_findnext`, `getdisk`, `setdisk`, `getcurdir`, `filelenght`.
* Accepts both Unix and Windows-style file paths in 'Open File' dialogs.
* Simple segmentation fault handler that gives you the chance to 'continue running' the application if something goes wrong.
* Compatibility with 32-bit help files.

There are a few environment variables that affect the behaviour of all Turbo Vision applications:

* `TVISION_DISPLAY`: strategy for drawing to screen. Valid values are `ncurses` and `ansi`. Default is `ansi`, which is a custom strategy that avoids redundant buffering and UTF-8 to wide char conversions. If you have issues, you may try `ncurses` instead.
* `TVISION_MAX_FPS`: limit of times screen changes are drawn to the terminal, default `60`. This helps keep the drawing performance reasonable. Special values for this option are `0`, to disable refresh rate limiting, and `-1`, to actually draw to the terminal in every call to THardwareInfo::screenWrite (useful for debugging).
* `TVISION_ESCDELAY`: the delay of time, in milliseconds, that should be waited after receiving an ESC key press. If another key is pressed during this delay, it will be interpreted as an Alt+Key combination.
* `TVISION_CODEPAGE`: the character set used internally by Turbo Vision. Only `437` and `850` are supported at the moment, although adding more costs as little as adding an array of translations in `source/linux/tables.cpp`.

### Windows

* Only compatible with the Win32 Console API.
* Applications fit the console window size instead of the buffer size.
* The console buffer is restored when exiting or suspending Turbo Vision.
* `kbCtrlC`, Shift+Arrow, `kbShiftTab` and AltGr key combinations work properly.
* No busy polling for events.

### All platforms

* Middle mouse button support (`mbMiddleButton`), even on DOS.
* Horizontal and vertical mouse wheel support (`evMouseWheel`), even on DOS.
* Arbitrary screen size support (with up to 65535 rows or columns).
* Graceful handling of screen resize events.
* Windows can be resized also from their bottom left corner.
* Support for `kbCtrlUp` and `kbCtrlDown` key codes (which don't work on 16-bit DOS, don't ask me why). They can be used to move windows faster with the keyboard (as `kbCtrlLeft` and `kbCtrlRight` already did).
* Improved usability of menus: they can be closed by clicking twice on them, even submenus.
* Improved usability of scrollbars: dragging them also scrolls the page. Clicking on an empty area of the scrollbar moves the thumb right under the cursor. They are responsive by default to mouse wheel events.
* Views don't lose their sizes when extremely resized.
* Support for LF line endings in `tvdemo` and `tvedit`. `tvedit` preserves the line ending on file save but all newly created files use CRLF by default.
* `tvedit`: drag scroll with middle mouse button.
* `tvedit`: delete whole word with `kbCtrlBack` and `kbCtrlDel` (note to linux users: they might not work on terminal emulators, but they do work on the console).
* `tvedit`: smart Home key (toggles between beginning of line and beginning of indented text).
* `tvedit`: support for files bigger than 64 KiB on 32-bit or 64-bit builds.
* `tvdemo`: event viewer applet useful for event debugging.
* `tvdemo`: option to change the background pattern.

## Not implemented

### Linux

* `evMouseAuto`.

## API reference

See the [Turbo Vision 2.0 Programming Guide](https://archive.org/details/bitsavers_borlandTurrogrammingGuide1992_25707423). There also exists a [Turbo Vision For C++ User's Guide](https://archive.org/details/BorlandTurboVisionForCUserSGuide/mode/1up), but in my opinion the former is more intuitive and easier to understand.

## API changes

* On Linux, screen writes are buffered and are usually sent to the terminal once for every iteration of the active event loop (see also `TVISION_MAX_FPS`). If you need to update the screen during a busy loop, you may use `TScreen::flushScreen()`.
* `TDrawBuffer` is no longer an static array. The equivalent of `sizeof(TDrawBuffer)/sizeof(ushort)` is the `.lenght()` method.
* Several constructors and methods now receive or return `const char*` instead of `char*` or have been the `const` qualifier.
* `TTextDevice` is now buffered, so if you were using `otstream` you may have to send `std::flush` or `std::endl` through it for `do_sputn` to be invoked.
* The `buttons` field in `evMouseUp` events is no longer empty. It now indicates which button was released.
* `TRect` methods `move`, `grow`, `intersect` and `Union` now return `TRect&` instead of being `void`, so that they can be chained.
* `TOutlineViewer` now allows the root node to have siblings.
* New class `TStringView`, which is a clone of `std::string_view`. You shouldn't need it unless you are programming in Borland C++, which has no `std::string_view`.
* Unicode support, see below.

## Screenshots

You will find some screenshots [here](https://github.com/magiblot/tvision/issues/7). Feel free to add your own!

## Contributing

If you know of any Turbo Vision applications whose source code has not been lost and that could benefit from this, let me know.

<div id="unicode"></div>

# Unicode Support

On Linux, the Turbo Vision API has been extended to allow receiving Unicode input and displaying Unicode text. The supported encoding is UTF-8, for a number of reasons:

* It is compatible with already present data types (`char *`), so it does not require intrusive modifications to existing code.
* It is the same encoding used for terminal I/O, so redundant conversions are avoided.
* Conformance to the [UTF-8 Everywhere Manifesto](http://utf8everywhere.org/), which exposes many other advantages.

## Reading Unicode input

The traditional way to get text from a key press event is as follows:

```c++
// 'ev' is a TEvent, and 'ev.what' equals 'evKeyDown'.
switch (ev.keyDown.keyCode) {
    // Key shortcuts are usually checked first.
    // ...
    default: {
        // The character is encoded in the current codepage
        // (usually CP437).
        char c = ev.keyDown.charScan.charCode;
        // ...
    }
}
```

Already existing Turbo Vision classes that deal with text input (most notably `TEditor` and `TInputLine`) depend on this methodology, which has not changed. Single-byte characters, when representable in the current codepage, continue to be available in `ev.keyDown.charScan.charCode`.

Unicode support consists in two new fields in `ev.keyDown` (which is a `struct KeyDownEvent`):

* `char text[4]`, which may contain whatever was read from the terminal: usually a UTF-8 sequence, but possibly any kind of raw data.
* `uchar textLength`, which is the number of bytes of data available in `text`, from 0 to 4.

So a Unicode character can be retrieved from `TEvent` in the following way:

```c++
switch (ev.keyDown.keyCode) {
    // ...
    default: {
        std::string_view s {ev.keyDown.text, ev.keyDown.textLength};
        processText(s);
    }
}
```

Let's see it from another perspective. If the user types `ñ`, a `TEvent` is generated with the following `keyDown` struct:

```c++
KeyDownEvent {
    union {
        .keyCode = 0xA4,
        .charScan = CharScanType {
            .charCode = 164 ('ñ'), // In CP437
            .scanCode = 0
        }
    },
    .controlKeyState = 0x200 (kbInsState),
    .text = {'\xC3', '\xB1', '\x00', '\x00'}, // In UTF-8
    .textLength = 2
}
```
However, if they type `€` the following will happen:
```c++
KeyDownEvent {
    union {
        .keyCode = 0x0 (kbNoKey), // '€' not part of CP437
        .charScan = CharScanType {
            .charCode = 0,
            .scanCode = 0
        }
    },
    .controlKeyState = 0x200 (kbInsState),
    .text = {'\xE2', '\x82', '\xAC', '\x00'}, // In UTF-8
    .textLength = 3
}
```
If a key shortcut is pressed instead, `text` is empty:
```c++
KeyDownEvent {
    union {
        .keyCode = 0xB (kbCtrlK),
        .charScan = CharScanType {
            .charCode = 11 ('♂'),
            .scanCode = 0
        }
    },
    .controlKeyState = 0x20C (kbCtrlShift | kbInsState),
    .text = {'\x00', '\x00', '\x00', '\x00'},
    .textLength = 0
}
```
So, in short: views designed without Unicode input in mind will continue to work exactly as they did before, and views which want to be Unicode-aware will have no issues in being so.

## Displaying Unicode text

The original design of Turbo Vision uses 16 bits to represent a *screen cell*—8 bit for a character and 8 bit for [BIOS color attributes](https://en.wikipedia.org/wiki/BIOS_color_attributes).

On Linux, a new `TScreenCell` type is defined in `<tvision/scrncell.h>` which is capable of holding a UTF-8 sequence in addition to extended attributes (bold, underline, italic...). However, you should not write text into a `TScreenCell` directly but make use of unicode-aware API functions instead.

### Text display rules

A character provided as argument to any of the Turbo Vision API functions that deal with displaying text is interpreted as follows:

* Non-printable characters in the range `0x00` to `0xFF` are interpreted as characters in the active codepage. For instance, `0x7F` is displayed as `⌂` and `0xF0` as `≡` if using CP437. As an exception, `0x00` is always displayed as a regular space. These characters are all one column wide.
* Character sequences which are not valid UTF-8 are interpreted as sequences of characters in the current codepage, as in the case above.
* Valid UTF-8 sequences with a display width other than one are taken care of in a special way, see below.

For example, the string `"╔[\xFE]╗"` may be displayed as `╔[■]╗`. This means that box-drawing characters can be mixed with UTF-8 in general, which is useful for backwards compatibility. If you rely on this behaviour, though, you may get unexpected results: for instance, `"\xC4\xBF"` is a valid UTF-8 sequence and is displayed as `Ŀ` instead of `─┐`.

One of the issues of Unicode support is the existence of [ｗｉｄｅ](https://convertcase.net/vaporwave-wide-text-generator/) characters and [combining](https://en.wikipedia.org/wiki/Combining_Diacritical_Marks) characters. This conflicts with Turbo Vision's original assumption that the screen is a grid of cells occupied by a single character each. Nevertheless, these cases are handled in the following way:

* *Wide* characters can be drawn anywhere on the screen and nothing bad happens if they overlap partially with other characters.
* Zero-width characters are always drawn as `�`, so they become one column wide.
* No notable graphical glitches will occur as long as your terminal emulator respects character widths as measured by `wcwidth`.

Here is an example of such characters:
![Wide character display](https://user-images.githubusercontent.com/20713561/89314567-18e16400-d67a-11ea-950f-d4c35d40e5f7.png)

### Unicode-aware API functions

The usual way of writing to the screen is by using `TDrawBuffer`. A few methods have been added and others have changed their meaning:

```c++
void TDrawBuffer::moveChar(ushort indent, char c, ushort attr, ushort count);
void TDrawBuffer::putChar(ushort indent, ushort c);
```
`c` is always interpreted as a character in the active codepage.

```c++
void TDrawBuffer::moveStr(ushort indent, const char *str, ushort attr);
void TDrawBuffer::moveStr(ushort indent, TStringView str, ushort attr); // New
void TDrawBuffer::moveCStr(ushort indent, const char *str, ushort attrs);
void TDrawBuffer::moveCStr(ushort indent, TStringView str, ushort attrs); // New
```
`str` is interpreted according to the rules exposed previously.

```c++
void TDrawBuffer::moveStr(ushort indent, const char *str, ushort attr, ushort width, ushort begin=0); // New
void TDrawBuffer::moveStr(ushort indent, TStringView str, ushort attr, ushort width, ushort begin=0); // New
```
`str` is interpreted according to the rules exposed previously. However:
* `width` specifies the maximum number of display columns that should be read from `str`.
* `begin` specifies the number of display columns that should be skipped at the beginning of `str`. This is useful for horizontal scrolling. If `begin` is in the middle of a *wide* character, the remaining positions in that character are filled with spaces.

```c++
void TDrawBuffer::moveBuf(ushort indent, const void *source, ushort attr, ushort count);
```
This function's name is misleading. Even in its original implementation, `source` is treated as a string. So, on Linux, this is equivalent to `moveStr(indent, TStringView((const char*) source, count), attr)`.

There are other useful unicode-aware functions:

```c++
int cstrlen(const char *s);
int cstrlen(TStringView s); // New
```
Returns the displayed length of `s` according to the aforementioned rules, discarding `~` characters.

```c++
int strwidth(const char *s); // New
int strwidth(TStringView s); // New
```
Returns the displayed length of `s`.

On Borland C++, these methods assume a single-byte encoding and all characters being one column wide. This makes it possible to write encoding-agnostic `draw()` and `handleEvent()` methods that work on both platforms without a single `#ifdef`.

The functions above depend on the following lower-level functions. You will need them if you want to fill `TScreenCell` objects with text manually. You may find complete descriptions in `<tvision/ttext.h>`.

```c++
size_t TText::next(TStringView text);
size_t TText::prev(TStringView text, size_t index);
size_t TText::wseek(TStringView text, int count);
#ifndef __BORLANDC__
void TText::eat(TScreenCell *cell, size_t n, size_t &width, TStringView text, size_t &bytes);
void TText::next(TStringView text, size_t &bytes, size_t &width);
void TText::wseek(TStringView text, size_t &index, size_t &remainder, int count);
#endif
```

For drawing `TScreenCell` buffers directly, the following methods are available:

```c++
void TView::writeBuf(short x, short y, short w, short h, const TScreenCell *b); // New
void TView::writeLine(short x, short y, short w, short h, const TScreenCell *b); // New
```

### Example: Unicode text in menus and status bars

It's as simple as it can be. Let's modify `hello.cpp` as follows:

```c++
TMenuBar *THelloApp::initMenuBar( TRect r )
{
    r.b.y = r.a.y+1;
    return new TMenuBar( r,
      *new TSubMenu( "~Ñ~ello", kbAltH ) +
        *new TMenuItem( "階~毎~料入報最...", GreetThemCmd, kbAltG ) +
        *new TMenuItem( "五劫~の~擦り切れ", cmYes, kbNoKey, hcNoContext ) +
        *new TMenuItem( "העברית ~א~ינטרנט", cmNo, kbNoKey, hcNoContext ) +
         newLine() +
        *new TMenuItem( "E~x~it", cmQuit, cmQuit, hcNoContext, "Alt-X" )
        );
}

TStatusLine *THelloApp::initStatusLine( TRect r )
{
    r.a.y = r.b.y-1;
    return new TStatusLine( r,
        *new TStatusDef( 0, 0xFFFF ) +
            *new TStatusItem( "~Alt-Ç~ Exit", kbAltX, cmQuit ) +
            *new TStatusItem( 0, kbF10, cmMenu )
            );
}
```
Here is what it looks like:

![Unicode Hello](https://user-images.githubusercontent.com/20713561/89353910-bd35cb80-d6b7-11ea-9ad6-0f608e5ae01c.png)

## Unicode support across standard views

Support for creating Unicode-aware views is in place, but most of the views that are part of the original Turbo Vision library have not been adapted to handle Unicode.

* At least `TFrame`, `THistoryViewer`, `TListViewer` and `TMenuBox` are able to display Unicode text properly.
* Automatic shortcuts in `TMenuBox` won't work with Unicode text, as shortcuts are still compared against `event.keyDown.charScan.charCode`.
* `TInputLine` can display and process Unicode text.
* `TEditor` assumes a single-byte encoding both when handling input events and when displaying text. So it won't display UTF-8 but at least it has a consistent behaviour.
