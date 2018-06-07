Ncurses Basics
================================================================================

> Note that many of these functions can be prefixed with the character `w` to call a variant that allows the user to pass in a specific window handle instead of defaulting to `stdscr`.  I'm not going to include those variants, however, each variant should be exactly the same except it accepts an extra window handle as the first argument, i.e. `move(y,x)` vs `wmove(win, y, x)`.

## Initialize/Destroy

`initscr()` is how you initialize ncurses for the current window (called `stdscr`)
  be sure to call `endwin()` before your program terminates otherwise you might leave the terminal in an odd state.

You can also temporarily leave curses mode, i.e.
```
def_prog_mode();   // save the tty modes
endwin();          // end ncurses (temporarily)
// ... do whatever
reset_prog_mode(); // return to previous tty mode stored by def_prog_mode
refresh();         // restore screen contents
// ... we are back in ncurses mode
endwin();          // end ncurses for real this time
```

## Window Refresh

`refresh()` will refresh the `stdscr` screen with any updates that have been made. Refresh will only update the portions of the window that have been changed. use `wrefresh(<window>)` to refresh other windows.

`clear()` clears the screen. Seems common to call before exiting the program, also `move(0,0)` also seems common.

## Cursor Functions

Each screen will maintain a cursor position that is updated with characters are output to the screen.

`getcury(win)` and `getcurx(win)`     // get the current cursor position
`move(y, x)`                          // move cursor for `stdscr` to (y, x)
`getmaxy(win)` and `getmaxx(win)`     // get the maximum y and x coordinates

It can also be moved manually by calling `move(row, col)`.

`curs_set()` can make the cursor `invisible`, `normal` or `very visible`.

## Output


There are 3 classes of functions that can output to the screen:

1. `addch()` print a single character with attributes
2. `printw()` print formatted output (similar to printf)
3. `addstr()` print raw strings

Note that these function will not have any effect on the screen until `refresh()` is called.

> Note that many of these functions can be prefixed with `mv` (i.e. `mvprintw`) to both "move" and output to the screen, however, I will be omitting them since they just bloat the API and really have no benefit to just calling `move` separately.

```C
 printw(fmt, ...);            // printf to `stdscr` at the current current position

// Note: I don't think the `printw` functions should have the `mv` variants.
// There's no reason they can't be 2 separate calls since both operations are orthogonal.

 addch(ch | A_<attr1> | A_<attr2> ...)      // print the character `ch` to `stdscr` with the given attributes

// i.e.
addch('j' | A_BOLD | A_UNDERLINE); // print the letter 'j' to `stdscr` in bold and underlined

 addstr(string)          // print `string` to `stdscr` at the current current position
 addnstr(string, length)      // same add addstr, limits characters to 'length'
```

Note that `ncurses` defines some special characters you can pass to print things like tables, lines, etc.  These are prefixed with `ACS_` in `ncurses.h`.

> Note: ncurses also has `vprintw` and `vwprintw` to support varargs.

## Keyboard Input

Like ouptut functions, input functions can also be divided into 3 categories:
1. `getch()` get a single character
2. `scanw()` get formatted input
3. `getstr()` get strings

Note that `getch()` functions will not read all input immediately because of "line-buffering" unless `raw()` or `cbreak()` have been configured.

#### "Key Management" functions

`raw()` and `cbreak()` tell ncurses to capture all keyboard events instead of line buffers. the difference is that `cbreak()` will not capture some control characters and instead process them as signals.

`echo()` and `noecho()`.  `noecho()` is the more common setting, give the app more control by allowing it to decide when and when not to echo input.

`keypad()` enables reading of function keys like F1, F2, arrow keys, etc.  Almost all interactive programs will enable this (i.e. arrow keys are very important)

## Attributes

`attrset()`, `attron()` and `attroff()` turn attributes on and off, i.e.
```
attron(A_BOLD); // enable bold font
attroff(A_BOLD); // disable bold font
attrset(A_BOLD); // disables all attributes except bold
```

`attr_get()` gets the current attributes and color pair of the window.

`attr_` functions.  These are similar to the above functions but take parameters of type `attr_t`.

`chgat()` functions.  Set attributes for a group of characters that are already on the screen.

#### List of Attributes
```
A_NORMAL        Normal display (no highlight)
A_STANDOUT      Best highlighting mode of the terminal.
A_UNDERLINE     Underlining
A_REVERSE       Reverse video
A_BLINK         Blinking
A_DIM           Half bright
A_BOLD          Extra bright or bold
A_PROTECT       Protected mode
A_INVIS         Invisible or blank mode
A_ALTCHARSET    Alternate character set
A_CHARTEXT      Bit-mask to extract a character
COLOR_PAIR(n)   Color-pair number n
```

## Mouse Input

`mousemask(mmask_t newmask, mmask_t *oldmask)` changes the mouse events you want to listen to.  Once mouse events have been enabled, `getch()` functions will return KEY_MOUSE everytime a mouse event occurs.  Then use `getmouse(MEVENT* event)` to get the event.

```C
typedef struct
{
    short id;       // id to distinguish multiple devices
    int x, y, z;    // event coordinates (characer-cell)
    mmask_t bstate; // button state
} MEVENT;
```

#### Mouse Events
```
BUTTON1_PRESSED          mouse button 1 down
BUTTON1_RELEASED         mouse button 1 up
BUTTON1_CLICKED          mouse button 1 clicked
BUTTON1_DOUBLE_CLICKED   mouse button 1 double clicked
BUTTON1_TRIPLE_CLICKED   mouse button 1 triple clicked
BUTTON2_PRESSED          mouse button 2 down
BUTTON2_RELEASED         mouse button 2 up
BUTTON2_CLICKED          mouse button 2 clicked
BUTTON2_DOUBLE_CLICKED   mouse button 2 double clicked
BUTTON2_TRIPLE_CLICKED   mouse button 2 triple clicked
BUTTON3_PRESSED          mouse button 3 down
BUTTON3_RELEASED         mouse button 3 up
BUTTON3_CLICKED          mouse button 3 clicked
BUTTON3_DOUBLE_CLICKED   mouse button 3 double clicked
BUTTON3_TRIPLE_CLICKED   mouse button 3 triple clicked
BUTTON4_PRESSED          mouse button 4 down
BUTTON4_RELEASED         mouse button 4 up
BUTTON4_CLICKED          mouse button 4 clicked
BUTTON4_DOUBLE_CLICKED   mouse button 4 double clicked
BUTTON4_TRIPLE_CLICKED   mouse button 4 triple clicked
BUTTON_SHIFT             shift was down during button state change
BUTTON_CTRL              control was down during button state change
BUTTON_ALT               alt was down during button state change
ALL_MOUSE_EVENTS         report all button state changes
REPORT_MOUSE_POSITION    report mouse movement
```

note: see `mouse_trafo()` to convert mouse co-ordinates to screen relative co-ordinates.

## Color

`has_colors()` returns TRUE if colors are supported
`start_color()` starts "color" functionality

`init_pair(pair_handle, forground_color_handle, background_color_handle)` create a "color pair handle"

you can start a color using
`attron(COLOR_PAIR(pair_handle))`

#### Predefined Colors
```
COLOR_BLACK   0
COLOR_RED     1
COLOR_GREEN   2
COLOR_YELLOW  3
COLOR_BLUE    4
COLOR_MAGENTA 5
COLOR_CYAN    6
COLOR_WHITE   7
```

`init_color(color_handle, red, green, blue)` change the color definition for `color_handle`

Note that the functions `color_content` and `pair_content` can get the current values for color handles and pair handles.

## Windows

`newwin(height, width, starty, startx)` allocates a new window, `delwin(win)` will delete it

#### Window Borders
`border(left, right, top, bottom, topLeft, topRight, bottomLeft, bottomRight)` draws a border around a window using the given characters for each part.  You can pass 0 to use the default characters.  You can also pass special characters like `ACS_VLINE`, `ACS_ULCORNER`, etc.

#### Screen/Window Dumping

`scr_dump()` will dump the screen contents to a file and can be restored using `scr_restore()`. Similarly `putwin()` and `getwin()` can be used to save/restore a window to/from a file.

`copywin()` can also be used to coopy a window into another window.
