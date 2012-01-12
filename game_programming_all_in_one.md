Game Programming All in One
===========================

By Jonathan S. Harbour, this book covers 2D game programming using the cross
platform Allegro framework with the C language.  The sample code uses Allegro 4,
which is outdated.  Allegro 5 has a new, backwards-incompatible API.  Since the
example code uses an old version of Allegro, I'll skim through it to learn
general ideas instead of taking precise notes.

Demystifying Game Development
=============================

This chapter is an overview of game development - it doesn't include anything
technical.

Harbour says the game industry employs millions of workers, most are specialized
in their own fields.  All of this is for entertainment.

There's little difference between programming for a particular console or a PC,
it's all based on C or C++.  The console SDKs include libraries that you link
into your program.  Many companies now produce the same games for multiple
platforms.

2D games are not dead, what sets them apart are fantastic gameplay.  It's best
to find a niche and use it to develop a great game.

This book focuses on the Allegro game library, originally developed by Shawn
Hargreaves for the Atari ST.  Allegro abstracts the operating system so your
code can compile on any supported platforms.

Getting Started with the Allegro Game Library
=============================================

Download Allegro from <http://alleg.sourceforge.net/>.  Install instructions
for your OS should be on there also.  Learn how to compile Allegro programs
here: <http://wiki.allegro.cc/index.php?title=Compiling_Allegro_Programs>.

Harbour gives us a step-by-step guide on setting up several C++ IDEs with
Allegro.  He then gives us an example program to compile to make sure everything
works.

The first function used is:

    int allegro_init();

This initializes the library and sets the `allegro_id` information which
contains information like what version of Allegro you're using:

    extern char allegro_id[];

At the end of the main function, use `allegro_exit()` to exit allegro.  After
the main function, the macro `END_OF_MAIN()` is required for cleanup.  You
can print out system information with:

    extern char allegro_id[];
    printf(Allegro version = %s\n", allegro_id);

2D Vector Graphics Programming
==============================

A **graphics primitive** is a function that draws a geometric shape.  Video
cards were designed to render geometric primitives with special effects.  It
renders triangles made up of vertices.  

**Blit** means bit-block transfer, a method of transferring a chunk of memory -
usually from system memory to the video card.  The **pixel** is the smallest
element on a screen.

`allegro_init` creates a global screen pointer called `screen`.  It uses
double buffering (render one screen on the monitor, another off-screen, and
switch between them).

The following program initializes full-screen video mode with Allegro 4:

    #include <stdlib.h>
    #include "allegro.h"

    int main(void) {
      allegro_init();
      install_keyboard();
      int ret = set_gfx_mode(GFX_AUTODETECT_WINDOWED, 640, 480, 0, 0);
      if (ret != 0) { 
        allegro_message(allegro_error); 
        return 1;
      }
      //display screen resolution
      textprintf(screen, font, 0, 0, makecol(255, 255, 255), 
                 "%dx%d", SCREEN_W, SCREEN_H);
      while (!key[KEY_ESC]);
      allegro_exit();
      return 0;
    }
    END_OF_MAIN()

`set_gfx_mode` detects the current computer's graphics settings and sets the
program's window.  `allegro_message` can be used for an alert box.
`allegro_error` is a global variable containing the most recent error message.
`textprintf` displays text in any video mode.

The next program draws text to the screen:

    include "allegro.h"
    int main(void) {
      char *filename = "allegro.pcx";
      BITMAP *image;
      int ret;
      allegro_init();
      install_keyboard();
      set_color_depth(16);

      ret = set_gfx_mode(GFX_AUTODETECT_WINDOWED, 640, 480, 0, 0); 
      if (ret != 0) {
        allegro_message(allegro_error);
        return 1; 
      }

      image = load_bitmap(filename, NULL); 
      if (!image) {
        allegro_message("Error loading %s", filename);
        return 1;
      }

      blit(image, screen, 0, 0, 0, 0, SCREEN_W, SCREEN_H);
      destroy_bitmap(image);
      textprintf_ex(screen,font,0,0,1,-1,"%dx%d",SCREEN_W,SCREEN_H);
      while (!keypressed());
      allegro_exit();
      return 0;
     }
     END_OF_MAIN() 

The simplest pixel drawing function is:

    void putpixel(BITMAP *bmp, int x, int y, int color)

To draw random pixels onto the screen:

    while(!key[KEY_ESC]) {
      //set a random location
      x = 10 + rand() % (SCREEN_W-20);
      y = 10 + rand() % (SCREEN_H-20);

      //set a random color
      red = rand() % 255;
      green = rand() % 255;
      blue = rand() % 255;
      color = makecol(red,green,blue);

      //draw the pixel
      putpixel(screen, x, y, color); 
    }

To draw lines use `hline` or `vline`:

    void hline(BITMAP *bmp, int x1, int y, int x2, int color)
    void vline(BITMAP *bmp, int x, int y1, int y2, int color)

Draw rectangles with:

    void rect(BITMAP *bmp, int x1, int y1, int x2, int y2, int color)
    void rectfill(BITMAP *bmp, int x1, int y1, int x2, int y2, int color)

Allegro also provides a callback feature, just use `do_line`:

    void do_line(BITMAP *bmp, int x1, y1, x2, y2, int d, void (*proc))

Your callback should have the format:

    void doline_callback(BITMAP *bmp, int x, int y, int d)

Draw circles and ellipses with:

    void circle(BITMAP *bmp, int x, int y, int radius, int color)
    void circlefill(BITMAP *bmp, int x, int y, int radius, int color)
    void ellipse(BITMAP *bmp, int x, int y, int rx, int ry, int color)
    void ellipsefill(BITMAP *bmp, int x, int y, int rx, int ry, int color)

These each have their own callback facility, just prefix each function with
`do_`.

The `spline` function draws curves based on four (x,y) input points:

    void spline(BITMAP *bmp, const int points[8], int color)

Triangles can be drawn with three points:

    void triangle(BITMAP *bmp, int x1, y1, x2, y2, x3, y3, int color)

And polygons can be drawn with:

    void polygon(BITMAP *bmp, int vertices, const int *points, int color)

You can use `floodfill` to fill in random regions within your program:

    void floodfill(BITMAP *bmp, int x, int y, int color)

There are three functions for primary text output:

    void textout_ex(BITMAP *bmp, const FONT *f, const char *s,
                    int x, int y, int color, int bg)
    void textout_centre_ex(BITMAP *bmp, const FONT *f, const char *s,
                           int x, int y, int color, int bg)
    void textout_right_ex(BITMAP *bmp, const FONT *f, const char *s,
                          int x, int y, int color, int bg)

Each of these have a `printf`-like equivalent.  These take a format string
and variable arguments:

    void textprintf_ex(BITMAP *bmp, const FONT *f, int x, int y, 
                       int color, int bg, const char *fmt, ...);
    void textprintf_centre_ex(BITMAP *bmp, const FONT *f, int x, int y, 
                              int color, int bg, const char *fmt, ...);
    void textprintf_right_ex(BITMAP *bmp, const FONT *f, int x, int y, 
                             int color, int bg, const char *fmt, ...);

Writing Your First Allegro Game
===============================

The first example game in this book is "Tank War", a two-player game on a single
screen.  The tanks are a series of rectangles:

    void drawtank(int num) {
      int x = tanks[num].x;
      int y = tanks[num].y;
      int dir = tanks[num].dir;

      //draw tank body and turret
      rectfill(screen, x-11, y-11, x+11, y+11, tanks[num].color);
      rectfill(screen, x-6, y-6, x+6, y+6, 7);

      //draw the treads based on orientation
      if (dir == 0 || dir == 2) {
        rectfill(screen, x-16, y-16, x-11, y+16, 8);
        rectfill(screen, x+11, y-16, x+16, y+16, 8); 
      } else if (dir == 1 || dir == 3) {
        rectfill(screen, x-16, y-16, x+16, y-11, 8);
        rectfill(screen, x-16, y+16, x+16, y+11, 8); 
      }

      //draw the turret based on direction
      switch (dir) {
        case 0:
          rectfill(screen, x-1, y, x+1, y-16, 8);
          break;
        case 1:
          rectfill(screen, x, y-1, x+16, y+1, 8);
          break;
        case 2:
          rectfill(screen, x-1, y, x+1, y+16, 8);
          break;
        case 3:
          rectfill(screen, x, y-1, x-16, y+1, 8);
          break;
      } 
    }

    void erasetank(int num) {
      //calculate box to encompass the tank int left = tanks[num].x - 17;
      int top = tanks[num].y - 17;
      int right = tanks[num].x + 17;
      int bottom = tanks[num].y + 17;

      //erase the tank
      rectfill(screen, left, top, right, bottom, 0);
    }

The projectiles from tanks are also small rectangles.  `fireweapon` is used
to draw the bullet in the correct direction.  It looks at the pixels in front
to determine if it's a hit or to keep moving it.

    void fireweapon(int num) {
      int x = tanks[num].x;
      int y = tanks[num].y;

      //ready to fire again?
      if (!bullets[num].alive) {
        bullets[num].alive = 1;
        //fire bullet in direction tank is facing
        switch (tanks[num].dir) {
          //north
          case 0:
            bullets[num].x = x;
            bullets[num].y = y-22;
            bullets[num].xspd = 0;
            bullets[num].yspd = -BULLETSPEED;
            break;
          //east
          case 1:
            bullets[num].x = x+22;
            bullets[num].y = y;
            bullets[num].xspd = BULLETSPEED;
            bullets[num].yspd = 0;
            break;
          //south
          case 2:
            bullets[num].x = x;
            bullets[num].y = y+22;
            bullets[num].xspd = 0;
            bullets[num].yspd = BULLETSPEED;
            break;
          //west
          case 3:
            bullets[num].x = x-22;
            bullets[num].y = y;
            bullets[num].xspd = -BULLETSPEED;
            bullets[num].yspd = 0;
        }
      }
    }

The `updatebullet` actually updates the bullet's rendering on the screen.

    void updatebullet(int num) {
      int x = bullets[num].x;
      int y = bullets[num].y;

      if (bullets[num].alive) {
        //erase bullet
        rect(screen, x-1, y-1, x+1, y+1, 0);

        //move bullet
        bullets[num].x += bullets[num].xspd;
        bullets[num].y += bullets[num].yspd;
        x = bullets[num].x;
        y = bullets[num].y;

        //stay within the screen
        if (x < 5 || x > SCREEN_W-5 || y < 20 || y > SCREEN_H-5) {
          bullets[num].alive = 0;
          return;
        }

        //draw bullet
        x = bullets[num].x;
        y = bullets[num].y;
        rect(screen, x-1, y-1, x+1, y+1, 14);

        //look for a hit
        if (getpixel(screen, bullets[num].x, bullets[num].y)) {
          bullets[num].alive = 0;
          explode(num, x, y);
        }

        //print the bullet position
        textprintf_ex(screen, font, SCREEN_W/2-50, 1, 2, 0,
          "B1 %-3dx%-3d B2 %-3dx%-3d", bullets[0].x, bullets[0].y,
          bullets[1].x, bullets[1].y);
      }
    }

There's an array called `key` that stores values of keys pressed.  We'll use
this to move the tanks.

    void getinput() {
      //hit ESC to quit if (key[KEY_ESC])
      gameover = 1;
      //WASD / SPACE keys control tank 1
      if (key[KEY_W]) forward(0);
      if (key[KEY_D]) turnright(0);
      if (key[KEY_A]) turnleft(0);
      if (key[KEY_S]) backward(0);
      if (key[KEY_SPACE]) fireweapon(0);

      //arrow / ENTER keys control tank 2
      if (key[KEY_UP]) forward(1);
      if (key[KEY_RIGHT]) turnright(1);
      if (key[KEY_DOWN]) backward(1);
      if (key[KEY_LEFT]) turnleft(1);
      if (key[KEY_ENTER]) fireweapon(1);

      //short delay after keypress
      rest(10);
    }

The `checkpath` function checks to see if the tank's pathway is clear.

    int checkpath(int x1,int y1,int x2,int y2,int x3,int y3) {
      if (getpixel(screen, x1, y1) ||
          getpixel(screen, x2, y2) ||
          getpixel(screen, x3, y3))
        return 1;
      else
        return 0;
    }

The structs that define the tanks and bullets are located in the header.

    struct {
      int x, y;
      int dir, speed;
      int color;
      int score;
    } tanks[2];

    struct {
      int x, y;
      int alive;
      int xspd, yspd;
    } bullets[2];

Getting Input from the Player
=============================

To start detecting keyboard input, you'll need to initialize Allegro:

    int install_keyboard();

The `allegro_exit` function will handle uninitializing it, but if you need to
do so before the exit is called use:

    void remove_keyboard();

Allegro keeps track of keys pressed in a global variable:

    extern volatile char key[KEY_MAX];

Check Allegro's `keyboard.h` for constants of each key on the keyboard.  A
few examples are:

* `KEY_UP`
* `KEY_DOWN`
* `KEY_LEFT`
* `KEY_RIGHT`
* `KEY_SPACE`
* `KEY_LSHIFT`

The typical game loop is:

    while (!key[KEY_ESC]) {
      // do some stuff
    }

Besides using the `key` variable, you can also use buffered keyboard input which
reads input via ASCII code.

    int readkey();                // returns next ASCII code or waits
    int ureadkey(int *scancode);  // returns next Unicode key or waits

The ASCII code is actually a two-byte integer value, the low byte is the ASCII
code that changes depending on modifier keys and the high byte is the scan
code regardless of modifier keys.

For demos, you can simulate key presses using:

    void simulate_keypress(int key);
    void simulate_ukeypress(int key, int scancode);
    void clear_keybuf();

To detect mouse input, you'll also need to initialize it:

    int install_mouse();
    void remove_mouse(); // optional, only if you want to disable it

The current mouse position is available via:

    extern volatile int mouse_x;
    extern volatile int mouse_y;
    extern volatile int mouse_z;  // wheel
    extern volatile int mouse_b;  // mouse button

`mouse_b` is just packed bits.  Use bit masks to detect which button was
pressed:

    mouse_b & 1; // left button
    mouse_b & 2; // right button
    mouse_b & 4; // center button

You can personalize the mouse cursor with:

    void set_mouse_sprite(BITMAP *sprite);
    void set_mouse_sprite_focus(int x, int y); // default is top-left
    void show_mouse(BITMAP *bmp);  // tell mouse where to be drawn
    void scare_mouse();
    void unscare_mouse();
    void position_mouse(int x, int y);
    void set_mouse_range(int x1, int y1, int x2, int y2);
    void set_mouse_speed(int xspeed, int yspeed);

Mastering the Audible Realm
===========================

The first thing to do is detect digital sound drivers.  This function returns
zero if none are available or the maximum number of voices a driver can
provide:

    int detect_digi_driver(int driver_id)

You then need to reserve digital and Midi sound drivers.  If you reserve too
many, the quality will drop.  Pass `-1` to restore voice setting to default:

    void reserve_voices(int digi_voices, int midi_voices)

You can control the volume per voice.  Pass `-1` to reset, use `0` for maximum
volume without distortion.  Use `1` when panning, each time you increase the
parameter by one the sound of each voice is halved.

    void set_volume_per_voice(int scale)

After configuring the sound system, Allegro can initialize it with:

    int install_sound(int digi, int midi, const char *cfg_path)

For `digi` the default is `DIGI_AUTODETECT`, for `midi` the default is
`MIDI_AUTODETECT`.

If you ever need to disable sound use:

    void remove_sound()

This gets called by default when Allegro exits.  The `set_volume` function
lets you set volume from 0 to 255.  Use `-1` for one parameter to leave it
unchanged:

    void set_volume(int digi_volume, int midi_volume)

To load a sample file in `WAV` or `VOC` format and play it:

    SAMPLE *load_sample(const char *filename)
    int play_sample(const SAMPLE *spl, int vol, int pan, int freq, int loop)
    void stop_sample(const SAMPLE *spl)
    void adjust_sample(const SAMPLE *spl, int vol, int pan, int freq, int loop)
    SAMPLE *create_sample(int bits, int stereo, int freq, int len)
    void destroy_sample(SAMPLE *spl)

The same thing for voices:

    int allocate_voice(const SAMPLE *spl)
    void deallocate_voice(int voice)
    void reallocate_voice(int voice, const SAMPLE *spl)
    void release_voice(int voice)
    void voice_start(int voice)
    void voice_stop(int voice)

To adjust the loop status:

    void voice_set_playmod(int voice, int playmode)

The available playmodes are:

* `PLAYMODE_PLAY`
* `PLAYMODE_LOOP`
* `PLAYMODE_FORWARD`
* `PLAYMODE_BACKWARD`
* `PLAYMODE_BIDIR` reverses direction on each finish

You can also control the voice volume, pitch, and panning:

    int voice_get_volume(int voice)
    void voice_set_volume(int voice, int volume)
    void voice_ramp_volume(int voice, int time, int endvol)
    void voice_stop_volumeramp(int voice)
    void voice_set_frequency(int voice, int frequency)
    int voice_get_frequency(int voice)
    void voice_sweep_frequency(int voice, int time, int endfreq)
    void voice_stop_frequency_sweep(int voice)
    int voice_get_pan(int voice)
    void voice_set_pan(int voice, int pan)
    void voice_sweep_pan(int voice, int time, int endpan)
    void voice_stop_pan_sweep(int voice)

MIDI music uses different functions.  It uses the struct `midi`:

    int get_midi_length(MIDI *midi)
    int play_midi(MIDI *MIDI, int loop)
    void midi_pause()
    void midi_resume()

Basic Bitmap Handling and Blitting
==================================

Introduction to Sprite Programming
==================================

Sprite Animation
================

Advanced Sprite Programming
===========================

Programming the Perfect Game Loop
=================================

Programming Tile-Based Scrolling Backgrounds
============================================

Creating a Game World: Editing Tiles and Levels
===============================================

Loading Native Mappy Files
==========================

Vertical Scrolling Arcade Games
===============================

Horizontal Scrolling Platform Games
===================================

The Importance of Game Design
=============================

Using Datafiles to Store Game Resources
=======================================

Playing Movies and Cut Scenes
=============================

Introduction to Artificial Intelligence
=======================================

Multi-Threading
===============

Publishing Your Game
====================