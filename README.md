# Space Game
This is a Wii U homebrew game. You can run the [latest release](https://gbatemp.net/threads/release-space-game.414342/) via Homebrew Launcher or from the Wii U Menu. Watch a video of the gameplay in action [here](https://www.youtube.com/watch?v=KMuicPmOIHw)!

Through different iterations of Space Game, several branches on this repository have emerged. The three primarily interesting ones are:
- **[master](https://github.com/vgmoose/space/tree/master)** - The original userland exploit-based branch of Space Game, which uses libwiiu
- **[hbl_elf](https://github.com/vgmoose/space/tree/hbl_elf)** - A branch designed for HBL to be executed as an elf that uses dimok's dynamic libraries
- **[wut_working](https://github.com/vgmoose/space/tree/wut_working)** (default) - The future and current branch, which uses WUT to generate an RPX (see below)
- **[hbl_elf_v2_beta](https://github.com/vgmoose/space/tree/hbl_elf_v2_beta)** - The development branch that is currently based off hbl_elf, soon to be integrated with wut_working

![Logo](http://vgmoose.com/posts/24261201%20-%20[release]%20Space%20Game!%20(for%20Wii%20U).post/title.png)

Space game is a simplistic graphical shooter game that runs natively on the Wii U!

## How to Compile and Run
See [building the Homebrew Launcher](https://github.com/dimok789/homebrew_launcher#building-the-homebrew-launcher) 	

On this branch, [to parse ID3 tags](https://github.com/vgmoose/space/commit/43783378b02e10af0de9e439e442f86e8292eacc), you will also need the libid3tag portlib, which is not included alongside dimok's portlibs, but is available precompiled [here](https://github.com/vgmoose/space/files/871652/portlibs.zip).

## Porting to RPX
This is the RPX version of the game, the final format revision of Space Game. This format can be launched from HBL v1.4 and up, from the Wii U Menu if you can compile the program correctly, and theoretically from within Decaf.

At this point in time, it is based on v1.6 of the game which has the screen rendering modifications by [xhp-creations](https://github.com/xhp-creations) and [CreeperMario](https://github.com/CreeperMario). None of the improvements from the current version of the Space Game v2.0 development branch are present in this build, though they will come soon.

At this point in time, the code for an official RPX release is done. If you are running it from HBL, it functions just as before, though if you run it from its own title, you have the added power of the HOME Menu. You can open the web browser or the eShop mid-game and return to the game with no trouble (trust me, I've tried). However, while the code is complete, I am having trouble packaging it into a format that can be installed on the console via [WUPInstaller](https://github.com/Yardape8000/wupinstaller).

The current build still relies of some of the [dynamic_libs](https://github.com/Maschell/dynamic_libs), mostly because of mismatches in WUT and how HBL software use certain functions. There are also things missing in WUT, like padscore, so dynamic_libs are necessary for this.

Though, unlike previous attempted RPX builds of Space Game, this one actually has working sound. And just like the HBL ELF version, the soundtrack is embedded into the RPX file, so that the user cannot screw up the game by deleting the soundtrack accidentally.

Other than that, enjoy! -CreeperMario

## Credits and License
This program is licensed under [the MIT license](https://opensource.org/licenses/MIT), which grants anyone permission to do pretty much whatever they want as long as the copyright notice stays intact.*
 - Programmed by [VGMoose](http://vgmoose.com)
 - Additional programming by [CreeperMario](https://github.com/CreeperMario)
 - Based on Pong by [Relys](https://github.com/Relys)
 - Music by [(T-T)b](https://t-tb.bandcamp.com/)
 - Space ship sprite by [Gungriffon Geona](http://shmups.system11.org/viewtopic.php?p=421436&sid=c7c9dc0b51eb40aa10bd77f724f45bb1#p421436)
 - Logo font by [Iconian Fonts](http://www.dafont.com/ozda.font) 	
 - libwiiu/bugfixes: [MarioNumber1](https://github.com/MarioNumber1), [CreeperMario](https://github.com/CreeperMario),  [brienj](https://github.com/xhp-creations), [NWPlayer123](https://github.com/NWPlayer123), [dimok](https://github.com/dimok789)

*While the game is free software, the song [~\*cruise\*~ is copyright ©2015 by the band (T-T)b](https://t-tb.bandcamp.com/track/cruise). This license does not permit redistribution or reuse of this song, unless it is embedded within an authorized Space Game binary.

## Issues
If you have any issues with the code here when you try to use it in your own app, feel free to contact me!

If you have any filesize issues when trying to run this app, I suggest you try commenting out the giant logo byte array in images.c.

![In game](http://vgmoose.com/posts/24261201%20-%20[release]%20Space%20Game!%20(for%20Wii%20U).post/gameplay.png)

## Documentation
Since it was originally designed to be executed via the browser exploit in firmware versions 5.5.0 and 5.5.1, there were several challenges that were imposed on its development in terms of efficiency and storage.

### Binary Size Tricks
A main issue with this webkit exploit is that binaries that can be executed in the browser are capped at a certain file size. I hit issues when my binary (code550.bin) exceeded 21,400 bytes. It may not seem like it, but that's not much! I employed a couple of tricks to keep the binary size small:

#### Compiling with -O1 as a CFLAG
If you look at my Makefile and compare it to other ones for the libwiiu examples, you'll notice a few differences near the top of the file. The most important of these differences is the addition of the -O1 parameter to the CFLAGS variable. This sets the compiler's [optimization level](http://www.rapidtables.com/code/linux/gcc/gcc-o.htm) and can usually shave off up to 6,000 bytes!

There was one big issue though: I found that, whenever I compiled with -O1, very strangely, calling OSScreenFlipBuffersEx(0) in draw.c caused a crash. This didn't happen with OSScreenFlipBuffersEx(1)! I'm still not sure why this happens. I think, when compiled with -O1, the first screen buffer does not get properly initialized. Since this happens in loader.c, I moved the Makefile around to compile every file **except loader.c** with -O1. This way, it's all still linked together into one mostly compact binary and won't crash.

**Note**: I also compiled the [https://github.com/wiiudev/libwiiu](https://github.com/wiiudev/libwiiu) repo with the -O1 flag.

#### Compressing Bitmaps
A fair amount of the file size, even after compression, is due to the bitmap image storing that is employed. Images are stored directly in images.c, with the assistance of [this script](https://gist.github.com/vgmoose/1a6810aacc46c28344ab) that converts a bitmap to a compressed C char array. The bitmap is then drawn via modifications to the draw.c library. It does not use GX2 at this time.

My compression algorithm is a little complicated, but it is detailed at the top of images.c. It tries to store information about streaks of pixels as efficiently as possible. The way I did it only allows for 125 total colors in a palette. The algorithm can also be used to compress arrays, the trick is storing sequences of similar numbers as instructions, such as: {1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1} becomes {8, 1, 3, 0, 10, 1} (eight ones, three zeroes, ten ones).

### Speedup Tricks
Drawing was a pain here, but I found I was able to greatly increase my drawing speed by moving all of the coreinit.rpl function pointers into a struct and passing that around to the drawing library. This is possible because the pointers do not change, so looking them up every time a pixel is drawn is, as far as I can tell, just extra time that could be spent drawing!

I also had to wrap my head around the concept of using flipBuffers. From what I gather, calling "flipBuffers" acts as a sort of "commit" for everything you've previously drawn to appear on screen. Then the next calls you make to draw will be put into the next upcoming buffer that you then flip to again. This is done so that a seamless gameplay can be presented without flickering, despite the entire screen being redrawn every frame.

### Misc Math

#### Trig functions
A big issue I had using this program was with not using the standard math library. I was not able to get it working without the file size shooting way up. As such, I implemented estimations for sin, cos, and arctan in math.h. These are primarily used to calculate angles and spin the spaceship. I found myself drawing many right triangles on paper and reciting SOHCAHTOA over and over again before I finally managed to get it right!

#### (Pseudo-)Random numbers
This was another area that I struggled with. Fortunately, there's a method to get the current time, so that can be used as a seed. The implementation I ended up going with is based on an older version of [glibc's rand()](http://stackoverflow.com/questions/1026327/what-common-algorithms-are-used-for-cs-rand).

#### Matrix multiplication
In order to rotate the bitmap to the angle calculated by the trig functions, I had to use a [rotation matrix](https://en.wikipedia.org/wiki/Rotation_matrix). This is actually a lot easier than it looks once you have the algorithm to multiply two matrices. What did stump me for a while is that the matrix has to first be translated up and left by half its height and width before rotating, since the rotation happens around the top left point. By translating it, you put the center in the top left, which allows for the rotation to take place around the center. This is then rotated back.

I also make use of a scaling matrix for explosions, which similarly gets multiplied by the bitmap when it's time to blow up

### Creating bitmaps
I created my bitmaps in Photoshop using Mode -> Index color. I extracted the palettes manually by using a hex editor. I've provided the three bitmaps that I used in this repo, although the actual image files are not used by the game. There are other ways to create bitmaps like this (I believe older versions of Microsoft Paint support it)
