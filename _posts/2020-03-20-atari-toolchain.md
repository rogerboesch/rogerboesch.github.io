## 🕹️ Atari 2600 Programming - Setup a toolchain on macOS

The best to show and explain how programming for the Atari 2600 works, is to try out by yourself. And luckily it's now much more easier as it was at the time when the Atari was famous. It's possible today to cross-compile (means programming on your Mac or PC) and also use an emulator for testing. A huge improvement since then.

For this 2nd part we will use just command line tools to have a solid understanding how all works below the hood.


### Assembler
The only programming language which was available in the seventies for the Atari was Assembler (No worries, it's not as horrible as it sounds...). This is also the reason why I want to use it in this tutorial at first, because we can work closely on the hardware and besides that, its fun to learn a "new" language :)

The most flexible and solid Assembler for the Atari 2600 platform (and others) is DASM. DASM is an open source 8-bit Macro-Assembler and available for all major platforms (also macOS) and what we use at first. It has no dependencies to any other libraries and is therefore easy to build from source.


### Emulator
For the emulator we have mainly two choices:

- Stella which is by far the best and most comprehensive Emulator of the Atari 2600. It has a lot of features, especially useful for debug and disassemble code.
- Z26, a more lightweight alternative and the one I use here in my toolchain
A disadvantage of both emulators is the dependency from SDL (Simple Direct Media Layer). SDL is a cross-platform development library designed to provide low level access to audio, keyboard, mouse, joystick, and graphics hardware via OpenGL and Direct3D.


### Toolchain

#### Prerequisites

The toolchain has two dependencies on macOS

- XCode command line tools: I assume (if you work on macOS) you have already installed XCode and the XCode command line tools. You can verify if you have installed them by enter `xcode-select -p` in the terminal window.
- SDL (Simple Direct Media Layer): The most easiest way to install SDL2 is to use Homebrew. A package manager that simplifies installing tools and libraries on macOS. If you have installed Homebrew already, then just type in `brew install sdl2`. If you dont have Homebrew then please install it  now at first :)


#### Installation
The rest is simple now because I have written a script that downloads all, makes the right settings and setup a toolchain that works perfectly with any editor and the command line.

{% highlight console %}
$ git clone https://github.com/rogerboesch/ataridev.git
$ cd ataridev
$ ./install-tools.sh
{% endhighlight %}

Thats all! You have now a working toolchain to write Assembler code that even will run on a real Atari 2600, but in our case at first on the emulator!

Follow the instructions in the terminal window how to assemble and start the example:

Open a new terminal window and type the following commands: 

{% highlight console %}
$ atari
$ cd examples
$ ataribuild.sh sample1.asm
{% endhighlight %}

That's it for now, but stay tuned for the  next article which shows you how to start creating a game!