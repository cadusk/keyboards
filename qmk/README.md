# Kinesis Keyboard Mod with QMK and Kint project

Having a Kinesis Advantage keyboard is awesome but having QMK running in is even
nicer. This repo is a curated list of instructions for parts, soldering,
assembling, modifying keymaps, building firmwares and flashing them to the
keyboard.

In a nutshell, QMK is a framework that allows you to configure microcontrolled
devices to serve as input sources such as MIDI devices, mouses and keyboards.

There's huge C code base that's used for building the firmware and an
accompaning CLI tool to make building easier.

QMK's project page on Github
https://github.com/qmk/

The list is supported keyboards is just mind blowing.
For Kinesis keyboards though, there a few different ways to do it. The way I
chose was to use Michael Stapelberg's mod, using Teensy microcontrollers due to
the quality of documentation provided.

Links to repos, tweets, blog posts and youtube streams that explain in detail
his reasons and shows step-by-step instructions on how to set up your keyboard.

> TODO: add info about the project


## Setting up the development environment

To build and flash images to your keyboard a few tools are necessary.
At the end of this configuration steps, you should have a QMK workspace that
looks like:

```
./
├── qmk_firmware
├── teensy_loader_cli
└── venv
```

### QMK cli

QMK cli is a python tool that helps setting up your development environment and
compiling images among other things. Since it's a python application, I prefer
to use a virtual environment to avoid messing with system wide dependencies.

This is how you create a new python virtual environment for working with QMK.
```
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip wheel
pip install qmk
```

> NOTE: If you already have the virtual environment setup, next time all you
> have to do is to make sure `venv/bin/activate` is sourced so qmk cli is found
> by your termminal.

### Clone a copy of qmk_firwmare

If you're starting from scratch, it's easy to create a copy of QMK's firmware
project using `qmk` cli command:

```
qmk clone --branch kint36 cadusk/qmk_firmware ./qmk_firmware
qmk setup user.qmk_home=$(pwd)/qmk_firmware
```

> NOTE: Most of the changes and improvements this project will have, like
> support to Teensy 4.1, better debouncing and etc., will likely come from
> qmk/qmk_firmware or kinx-project/qmk_firmware. And for that reason it's
> necessary to add a few custom remotes to the local repository.
>
> `git remote add qmk https://github.com/qmk/qmk_firmware`
> `git remote add kinx https://github.com/kinx-project/qmk_firmware`
> `git fetch --all`

As a final step cd into the newly created qmk_firmware folder and run qmk's
system check up to make sure all dependencies are met.

```
cd qmk_firmware
git checkout kint36
make git-submodule
qmk setup
```

> NOTE: This process may require administrator's password in order to create OS
> level configuraion, like udev rules for detecting microcontrollers connected
> to the computer's USB.

### Clone a copy of Teensy Loader command line

Teensy Loader is required to flash images generated into the Teensy
microcontroller that sits inside your keyboard. This tool is distributed as
source code needs to be built locally to be used.

```
git clone https://github.com/PaulStoffregen/teensy_loader_cli
cd teensy_loader_cli
make
```

> NOTE: Teensy Loader will also need to update udev rules to make sure Teensy
> microcontrollers are detected.
> More info at their website: https://www.pjrc.com/teensy/loader_cli.html
>
> Make sure to go through the documentation above in case any dependency for
> your system in particular is required.

## Building images

Currently there are two relevant branches I am intereted in: `kint-36` and
`kint-41`, since those are the two types of microcontrollers that power my
configuration.

### custom kint36

```
cd qmk_firmware
git checkout kint36
make git-submodules

qmk compile -kb kinesis/kint36 -km cadusk
```

### custom kint41

```
cd qmk_firmware
git checkout kint41
make git-submodules

qmk compile -kb kinesis/kint41 -km cadusk
```

Images generated will be stored in a build folder inside qmk_firmware/.
Files are named in the format: `keyboard_controller_keymap.(hex|bin)`

```
$ ls ./qmk_firmware/.build/*.hex
kinesis_kint36_cadusk.hex
kinesis_kint41_cadusk.hex
```

### Flashing the images

To see the list of supported MCUs, use the command:
```
./teensy_loader_cli/teensy-loader-cli --list-mcus
```

In my case, I will use only MCUs `TEENSY36` and `TEENSY41`.

To flash images into your Teensy microcontroller, use the command below.
```
./teensy_loader_cli/teensy-loader-cli -w -v --mcu=TEENSY36 ./qmk_firmware/.build/kinesis_kint36_cadusk.hex
```

The microcontroller must be in flashing mode for the command above to work. If
this is a new microcontroller that is not currently running this image, the only
way to put it in *flashing mode* is by pushing the little reset button on the
top side of the microcontroller.

If this is already running this QMK image then putting it in *flashing mode*
should be as simple as pushing the *PROGRAM* button on the far right top of the
keyboard. It is programmed to put the microcontroller into *flashing mode*. See
it [here](https://www.pjrc.com/teensy/loader_cli.html).

When flashing is done, the microcontroller will restart automatically. It can be
noted watching the for LED's go on and off.

