# VPinball on RPi4 on Batocera

So I got vpinball working on Batocera *for a rpi4*!  I will say the performance is lacking but if you feel like trying it out, here it is.

You will need at least a 4GB model, 1GB does not work, it runs out of memory (go figure).

There is a bunch of typing and editing. Not much point and click. So be ready. I'm also writing this from the perspective that I'm doing everything from a Linux system. Windows systems some of the commands and procedures will change, like scp.

You will need at least one table .vpx file.  You will need the roms for the table in zip format (do not unzip this file). The rom file will be something like "something_something.zip".  For example hs_l4.zip.

You will need an installed and working Batocera 41.  I didn't test any other versions.

You will need to be able to ssh and scp to the Batocera system. Let's give it a hostname of batocera for this blurb.  Using ssh and scp is beyond the scope of this document but there is plenty of tutorials from the internet, and it's quite simple.

You will need to know how to use vi, vim, or nano to edit text files on a Linux system (batocera). Again, I'll leave you to find how on the internet.

You will need a github account to download "vpinballx standalone".  Create your account and get this.  You will want the 64bit version. The current filename is VPinballX_GL-10.8.1-2883-a1b5d19-Release-linux-aarch64-rpi.zip, to give you an idea.  https://github.com/vpinball/vpinball/actions is a good starting point.  Click on one of the builds and find one for aarch64-rpi.  No other ones will work.

Uncompress this zipfile so you have VPinballX_GL-10.8.1-2883-a1b5d19-Release-linux-aarch64-rpi.tar.

So that is the prerequisites.  We need to put everything in place and go from there.

The nice thing is that batocera seems to be built off of a base and then compiled for the different platforms supported like mac or x86_64, or in our case, aarch64-rpi.  So what you find in x86_64, you'll find in aarch64-rpi.  One thing we want to find is the setup required for vpinball. And guess what? It's there, just a bit hidden. So let's work through that.

First thing is to get the binaries in place. A bit involved but it can be done in a few steps.
So scp the binary to batocera.  It'll be something along the lines of:

```scp VPinballX_GL-10.8.1-2883-a1b5d19-Release-linux-aarch64-rpi.tar root@batocera:/userdata/```

In the above command you can also fill in the ip instead of batocera if you aren't using any type of DNS. If you don't know, use the ip.

Now ssh to your batocera and login as root (default password is linux)
It will drop you in a directory "/userdata/system".  Use the command "pwd" to verify that.

Now you need to put the binaries in the proper place.  Batocera says for the x86 version, drop the binaries into /usr/bin/vpinball.  However, this is an overlay and will get overwritten every power cycle, and it's also of limited size.  Where is there space?  /userdata

So let's use /userdata as our binary holder and point /usr/bin/vpinball to it.

```
cd /userdata
mkdir vpinball
cd vpinball
tar xvf ../VPinballX_GL-10.8.1-2883-a1b5d19-Release-linux-aarch64-rpi.tar
[a bunch of output]
```

You have now put it in /userdata/vpinball.  So now make a soft link to here in /usr/bin/vpinball

```
ln -s /userdata/vpinball /usr/bin/vpinball
```

That will make it temporary. To make it permanent run:

```
batocera-save-overlay
```

Now when batocera goes looking for the vpinball binary, it'll find it there.  We just need to tell batocera that it's ready. And that happens through something called Emulation Station.

Let's go there and make a backup, just in case.

```
cd /etc/emulationstation
cp es_systems.cfg es_systems.cfg.orig
```

Edit es_systems.cfg.  You can use vi, vim, nano, or whatever else you may find.  But what you want to is cut & paste the following, one line above the last line (which reads </systemList>).  Do not remove any other lines in the file. Copy EXACTLY as seen, the spaces are important:
```
  <system>
        <fullname>Visual Pinball X</fullname>
        <name>vpinball</name>
        <manufacturer>Randy Davis</manufacturer>
        <release>2000</release>
        <hardware>pinball</hardware>
        <path>/userdata/roms/vpinball</path>
        <extension>.vpx</extension>
        <command>emulatorlauncher %CONTROLLERSCONFIG% -system %SYSTEM% -rom %ROM% -gameinfoxml %GAMEINFOXML% -systemname %SYSTEMNAME%</command>
        <platform>vpinball</platform>
        <theme>vpinball</theme>
        <emulators>
            <emulator name="vpinball">
                <cores>
                    <core default="true">vpinball</core>
                </cores>
            </emulator>
        </emulators>
  </system>
```

Save this file. Recall above where I said we just need to enable the "hidden" Visual Pinball stuff? This does it.

Run

```
batocera-save-overlay
```

once more to save the work.

Now something interesting.  If you try to run VPinballX_GL with a table, you will get some errors about duplicate lines. We need to remove those lines from the default ini configuration.

```
cd /usr/bin/vpinball/assets
```

You will edit the following lines OUT of the file Default_VPinballX.ini (ie. delete them). Note that this may change with version however, in 3 sections the lines are always:

```
Elasticity =
ElasticityFalloff =
Friction =
Scatter =
```

They can be found in and around lines 872, 988, and 1095 in the file Default_VPinballX.ini in the above directory.
In vi you would type in "872G" to get to line 872. Then "dd" to delete each line. Again, get instructions from the internet if you are not sure.  Make sure you do not delete any other lines and only the instances found around those line numbers.  There are other places where these lines must remain.

Save the file.

Now you need your table (the .vpx file) and your roms (something_something.zip).
The tables go into /userdata/roms/vpinball/<table name>

```
mkdir /userdata/roms/vpinball
mkdir /userdata/roms/vpinball/<table name>
```

So you would scp the .vpx to /userdata for example, similar to how you did it for the VPin*.tar earlier.
Then

```
mv table.vpx /userdata/roms/vpinball/<table name>
```

An example with a table called fake.vpx

```
scp fake.vpx root@batocera:/userdata
Then login to batocera with root
cd /userdata
mkdir /userdata/roms/vpinball
mkdir /userdata/roms/vpinball/fake
mv fake.vpx /userdata/roms/vpinball/fake
```

Now "ls /userdata/roms/vpinball/fake" should show you the file fake.vpx.  If not, go back and do it again, you made an error.

It's time to reboot. Once you reboot you should see an option for Visual Pinball. And inside it should contain the "fake" table you set up.

Batocera says the roms should be in a directory /userdata/roms/vpinball/table_name/roms.  So log back in and create the directory and scp and copy the rom files into that directory.

Also, "mkdir /userdata/vpinball/plugins" as it will complain otherwise.

Once more, just in case

```
batocera-save-overlay
```

Again reboot your system.
Go into "Visual Pinball", select your table and open it.

It'll open and begin running. It might freeze the first time, this is normal. Quit the game and then go back and it will work. This is a known "problem" with Visual Pinball, from now on the table will start normally. I have to do this for all tables the first run.

So let's talk performance. There isn't any. It's horrible. You can't follow the ball at all. Very few frames per second.  I don't recommend it at all.  So, no, you didn't do anything wrong. RPi4 isn't really meant for this purpose.

You can try the following settings plus 720p resolution in batocera and perhaps it'll be good enough? Find and edit the following section to look like this.

```
MaxFramerate =
MaxPrerenderedFrames =
FXAA = 0
Sharpen =
ScaleFXDMD =
DisableAO = 1
DynamicAO =
SSRefl = 0
PFReflection = 0
MaxTexDimension =
AAFactor =
MSAASamples =
DisableDWM = 1
UseNVidiaAPI =
HDRDisableToneMapper =
HDRGlobalExposure =
ForceBloomOff = 1
ForceMotionBlurOff = 1
ForceAnisotropicFiltering = 0
CompressTextures =
SoftwareVertexProcessing =
```

It's almost playable. Sort of. There's a tiny bit of input lag and you can almost follow the ball.  Anyone got any more suggestions?

I wonder what a RPi5 would do but I don't have one to test with.

However, that's how to get Visual Pinball on your RPi4 if you feel so adventurous.

Have fun
