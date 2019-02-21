---
title: Best Practices for Max: Users' Favorite Techniques Today
subtitle: Jeff Morris and Peter McCulloch
date: 2019-02-21
tags: ["MaxMSP", "Techniques", "Articles"]
---
Every once in awhile, in the life of the Max[^1] graphic programming environment for music, sound, and video, someone posts a call for users’ best practices of the day. This fall, Estevan Carlos Benson posted the call on the Max/MSP public Facebook group and received a healthy response from the community. This is a compiled edition of those responses. The ideas may not all be original;  some are simply gems from the Max documentation, but this collection shows what practices are foremost in the minds of Max coders today. The original post can be read at the [MaxMSP Facebook Page](https://www.facebook.com/groups/maxmspjitter/permalink/10156695451854392/); in each section we give credit to the people who originally commented on the thread.

In this document you will find advice for experienced Max users to build and tune patches that run more reliably and consistently. This is not the same as making interesting work. These  are not commandments, but suggestions for  new approaches to common technical problems.. We are making art, not building planes, so there are many ways to achieve similar results. Consider these to be currently accepted practices,  unless you have a good reason to do something differently.

Also keep in mind that artistic time (time spent on creative expression) is an important resource. Sometimes the best practices for intermediate or advanced users is not the best for beginners. Beginners often spend too much time optimizing or modularizing without enough experience to make good judgments about it.  Encountering frustration with poor practices can be a good experience to help understand the importance of better ones. For new users of Max, Cycling74’s own documentation is the best starting point.

--From: Todd Ingalls, Peter McCulloch, and Jeff Morris

[^1]: Max, historically limited to control-rate functions like MIDI, is also referred to by its audio library MSP, its video library Jitter, its Gen family of lower-level coding tools, or a combination of those names.



# Data
This is a general programming mantra, but a good one: “there are only three quantities in programming: zero, one, and many.” If you build something to handle three values, it should be able to handle N values[^2}.

Data structures are often challenging for new programmers. . Learning to use Max’s data structures with lists, coll, jit.matrix, and dict, among others, is a good way to become an advanced user. You’ll be able to manipulate data more powerfully and cleverly, and will use fewer patch cords.

Lists are the simplest data structure to work with and are an excellent tool for simplifying complex patches. If you find yourself needing multiple values to arrive in sync, consider sending them as one list. If you’re sending different values to multiple voices, for example volume, you could send one list of all volumes to all the voices and have each voice use zl.nth to pick off the volume that is specific to it.

There are several objects for building lists, including join, **pack**, **pak**, **prepend**, and **append**, each with different suggested use cases; **join** is the most general and can handle most tasks. The **zl.group** object can be used to build up lists of arbitrary sizes via its argument. Alternatively, give it no arguments and send a bang to output the current items as a list. 

![Image1](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/zlgroup.png)

Learn to use the **zl** objects with **vexpr**. It is totally worth your time and will enable you  to do much more. The **vexpr** object isn’t just for obvious math operations. You can also do things such as  finding the number of values above a given threshold: **vexpr $f1>20** into **zl.sum**. Counting the number of values inside a range is similarly easy: Connect **vexpr $f1 > 20 && $f2 < 40** to **zl.sum**. (Alternatively, you could multiply instead of using &&, and there are interesting uses for this). 

The **dict** object is the most flexible data structure in Max. It allows you to store key/value pairs in a named location. The value in each pair can be a single value, a 1-D array (a list), or another dictionary, so a single dict can contain many sub-dicts. Several objects in Max store their state as a dictionary and will output the name of this dictionary when you send the appropriate message; for example, you can use “linkdump” for live.step, and “dump_to_dict” for live.grid. Dictionaries make it easy to share large amounts of data between different parts of your patch without having to send (i.e., make a copy of) the actual data. Dict uses the JSON (JavaScript Object Notation) format making it  easy to access from within JavaScript; some users rarely use the built-in dict objects, preferring instead to work with dict inside of JavaScript.

When mapping a value to control other processes, it is often helpful to have a floor and a ceiling for the incoming value, especially if you are using **scale** or **scale~** for the remapping. Use **clip** or **clip~** in front of **scale** or **scale~** to enforce a minimum and maximum for the input. If your input value has a minimum of 0 and is at control rate, the function object can be used for interpolation; it will return the y value for the envelope at the input x value.

Don’t forget that once you input your audio, imagery, controller data, etc. in Max, they become numbers, and can be treated as any kind of data. Consider using a matrix for more than just video. Surprising results can come from creatively “misusing” data.

--From: Joshua Mark Goldberg, Peter McColluch, and Jeff Morris

[^2]: This is known as the Zero-One-Infinity rule. [Zero-One-Infinity Wiki](http://wiki.c2.com/?ZeroOneInfinityRule)



# Readability
Remember that artistic time is more expensive than computer time. Making your patches easy to read and understand will help reduce time spent troubleshooting so you can focus on creating.

Establish  a solid habit of adding comments to your code. This can help you identify parts whose functionality you want to copy into other patches later. Leave comments crediting sources where you have borrowed code and/or learned techniques from others. If you discovered the code in a discussion forum, a comment may help you track down the  original thread if you need to revisit later. Take time to label inlets and outlets in your subpatchers and abstractions. You can also tag your patches with metadata (in the Patcher Inspector) to make them easier to find in the File Browser. If others have downloaded your patches, this will help them discover when your patches are relevant to their needs at the time. 

Remember, however, that Max is already quite readable, so focus on describing overall intent and function and any sneaky bits whose functions aren’t obvious. From there, you might rely on good patch organization and descriptively naming your subpatchers and abstractions instead of comments. The fewer the objects in a patch or subpatch, the easier it is to understand.

Come Up with  your own naming conventions. It is common to insert your initials in an abstraction name so you recognize where they came from (e.g., jd.myabstraction). This also reduces the chance of accidentally using the name of a built-in object. Decide on a consistent way to name your **send** and **receive** objects. One example is to use CamelCase (so you can see that it’s a word you created) and extensions that indicate what format or unit will be expected on the other end, e.g., .b for bangs only, .i for integers, .f for floats, .ms for milliseconds, .dB for decibels, .amp for raw amplitude, .% for percentages, .mx for multipliers without units, and .com to receive multiple types of commands. 

The way you name your **send** and **receive** objects and the protocols you establish by putting **line** or **route** after the **receive** objects can basically establish your own scripting language, which you would use in **qlist** or semicolon message boxes, so practice this intentionally and consistently. For example, if you have a **receive** named Voice1 connected to a **route** accepting parameters like Hz and amp, and with a **line~** on each of its outlets, you can create your own readable code on top of Max, and the transition time for **line~** is even optional. To fade Voice1 to zero you could just bang a message box containing:

![Image2](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/voice1.png)

You can also send multiple messages at a time using this syntax. Use semicolons to separate messages going to different receives and use commas to separate messages sent to the same receive. You might even connect a **print Huh?** object to the last outlet of **route** so that if you mistype any commands, they appear in the Max Console like any other error message, preceded by the argument “Huh?”— just a shortened version of “I don’t understand the command: _______.”

Many programmers like to have a straight line down the left side of the patch, if possible. Objects should be ordered top-to-bottom in the order they operate. The only patch cords that go up are for feedback; many times these are also the only segmented patch cords.

![Image3](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/remotemessage.png)

--From: Estevan Carlos Benson, Seth Boney, Peter McColluch, Jeff Morris, Robin Parmar, John Sharp, Michael Sperone, and Michael Zbyszyński.



## Patch Layout
The Fix Width and Align features are top on many users’ lists for keeping your patches easy to understand. Keep inlets at the top edge of a subpatcher or abstraction and outlets along the bottom to make it easy to see where data paths lead outside it. Many insist on tidying as you go; some prefer to code in chunks first and then pause to neaten up. 

Avoid overlapping patch cords that have different start and end points. If they share a  start or end point, you might want them to overlap to  resemble an electrical schematic (and save some space). Many insist on using segmented patch cords but limit them  to two corners. Besides readability, this helps any feedback loops stand out because they will likely require more corners. The  occasional curved patch cord works well to connect two large sections of a patch. On the other hand, if you’re having trouble deciding the best way to route your patch cords you might  put some code into a subpatch. You can also use route to reduce the number of control rate patch cords connecting large portions of your patch across the screen. 

--From: Tom Hall, Peter McCulloch, Jeff Morris, Sam Pearce-Davies, and Sem Shimla.



## Colors
Some users make a habit of coloring **send** and **receive** objects (matching colors for each pair), or making any feedback patch cord red. Some prefer to color the objects that you can open by double-clicking, such as subpatchers, **coll**, Gen objects, or objects containing line-oriented code like JavaScript or OpenGL shaders. One suggested color scheme: **loadbang** objects are red, params in Gen are orange, **buffer~** objects and those that access them are blue, and inlets and outlets are green.

--From: Nathan Greywater, Peter McCulloch, Robin Parmar, and Andrej Prijic.



# Working
When building something complex, think of what you want to achieve before considering how to build it. Write out the types of messages your abstraction should understand. There are many equivalent ways to do the same thing, but the difference (and opportunity) is often in the details. If you make time for a dedicated design phase instead of designing on-the-fly as you build, you can avoid having only the available and familiar objects steer your creativity.

If you run into problems when working on large and complex patches, isolate the problem. Copy and paste sections onto blank patchers and check their operations individually. This can help you build healthy habits of compartmentalizing your code in subpatchers with as few inlets and outlets as possible. It can also give you a minimum working example you can share on a discussion forum if you need to ask others for help.

Remember that the easier things are to do, the more likely you are to do them. Write wrappers around GUI objects to provide convenient actions to customize your most-used toolset. For example, it might be handy to trigger an envelope in **function** by sending it a list dictating the peak amplitude and duration, or have a way to extend the length of a single segment in the envelope and having the rest of the segments slide over.

--From: Daniel Belquer, Peter McCulloch, and Jeff Morris.



## Time-Saving Tricks
The Paste Replace feature (pasting one cut or copied object in place of another one) is a popular time saver. This allows you to insert an object, subpatcher, or abstraction into existing code while keeping all the existing patch cords connected. Highlighting a patch cord and pressing shift-N will insert an object inline between the two connected objects. Conversely, to quickly remove the patch cords attached to an  object or group of objects, you can click or drag to highlight it, then cut and paste it back into place.

--From: Daniel Abu Kamelia, Matt Kav, and Jeff Morris.



## As You Code
Make a habit of providing default values for objects as arguments, when you create them. Consider how those defaults relate to the arguments of connected objects. For example, if **line~ 1** is connected to **~ 0** and you later delete the **line~**, your patch will be silenced. It may be better to have both arguments match.

Max does not always require you to use decimals to treat numbers as floats (instead of integers), but you might consider doing that  all the time anyway. Besides not having to wonder whether  it matters in each particular case, decimals can help you read long messages that dictate envelopes to **line~** if your amplitudes are all floats and ramp times are ints.

When creating a **coll** object, immediately set it to save with the patcher (via the Inspector or the @embed 1 attribute) or direct it to read a file upon loading. You can decide not to do this on occasions when you need something different, but without this habit you risk losing the data you put into **coll** without noticing.

--From: Peter McCulloch and Jeff Morris.



## Favorite Tools
Some tools in Max that users are quick to recommend: Use the floating Clue Window to see a running display of help related to the objects you’re working on. Use Projects feature to collect and organize all the files required for your patch. Unlock help patchers and copy any useful code out of them, for example, the **umenu** object and menu and poll messages that are always useful with a **hi** object.

![Image4](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/umenuhi.png)

Gen is learnable and well worth your while; it is useful for many things big and small. When you start doing conditional operations in **gen~**, **codebox** quickly becomes useful. You will be able to handle complex flow control better. You can always copy from the Code window (the C button on right sidebar).

On the other side of things, you may want to avoid using the Audio On/Off and Gain controls built into every patcher window because you could end up with mismatched gain levels across patches and hard-code the wrong volume settings to balance them. This can be especially problematic for newcomers. If you want to ensure your **dac~** toggles are always updated, no matter where audio is turned on or off, connect a **dspstate~** object to the toggle.

![Image5](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/dspstate.png)

--From: Jjuulien Bbaayle, Jeffrey Clark, Joshua Mark Goldberg, Kevin Kripper, Peter McCulloch, Jeff Morris, and Robin Parmar.



# Saving State
Max has multiple systems for storing presets, including, in order of complexity, the preset object, the snapshot system, and the pattr system. 

Preset systems can work as a way of organizing real-time compositions, but remember: you are working in a programming language rather than a DAW for a reason. If everything you cared about was easily described by discrete tracks of single values, then why would  you use  Max?

That being said, it is always a good idea to centrally initialize your patch. One approach is to use a single message box with all the pertinent **receive** object names and their initial values (this may be redundant if you give objects default arguments, but it’s an investment in safety and reliability):

![Image6](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/savestate.png)

You can also do this with **patcherargs** and **route**.[^3] This also allows you to use **dict**, which can store an abstraction’s internal state. The **dict.iter** object will output a series of key and value pairs for one level of a dict; if you have several instances of an abstraction which share the same initial values, consider using a shared dict to initialize them. This can be especially helpful for abstractions that accept many attributes. For GUI objects with the **parameter_enable** feature activated you can set initial values in each object’s Inspector.

Use **loadbang** or **loadmess** to trigger the initialization on load. Then you can bang them or send a “loadbang” message to a **thispatcher** object to reset your patch at any time.

![Image7](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/loadbangmess.png)

--From: Brian Del Toro, Peter McCulloch, and Robin Parmar.

[^3]: For a video tutorial on how to use patcherargs, visit: [Cycling74 Tutorials](https://cycling74.com/tutorials/patcher-arguments-video-tutorial-meet-the-patcherargs-object)



# Structure
Many users swear by using **trigger** whenever possible to control the order of data flow. If you do have two patch cords from one outlet, you might be inviting problems that will be difficult to track down (signals excepted, sometimes).

Never use timing objects like **delay** and **pipe** to fix ordering problems. Using them to address ordering problems creates subtle bugs that are difficult to find because their behaviors are affected by CPU load.

--From: Peter McCulloch, Robin Parmar, and Michael Zbyszyński.



## Centralization
You may find exceptions, and you’ll know when you need to make an exception, but in general Max users recommend only using one central **loadbang**, **metro**, and one master **dict** to store named collections as sub-dictionaries, as well as relying on **jit.world** for all your pre- and post-render bangs. If you need to synchronize sample-rate objects, you may need a master **phasor~** to drive everything, manipulating its output into the shape needed for each object while ensuring they’re all in sync.

Without a single central **dac~** object, you may find it difficult to figure out where a sound is coming from, or to fix clipping. It also makes it simpler to remap channels when your move to a different speaker setup. This can even be automated with the “set” message, for example, to make presets for surround sound and a stereo mixdown.

--From: Chris SW Anderson, Jacopo Carreras, Joshua Mark Goldberg, Sam Tarakajian, and Michael Zbyszyński.



## Sends and Receives
Some users prefer to avoid using **send** and **receive** objects excessively. Others focus on keeping the number of **receive** objects low, since each **send** object will send every **receive** object into action, even if it’s just to determine that no further action is needed in that part of the patch. Sending many unnecessary messages can adversely affect the performance of the scheduler, so some of your timings may suffer as a result. This is, however, largely dependent on what the rest of your patch is doing.

Send and receive objects are global by default, unless their names are prefixed with #0, e.g. **send #0_count**. These “#0 sends” can be helpful in reducing the complexity of abstractions; sometimes it is easier to use sends and receives instead of long patchcords. One potential pitfall of using **send** and **receive** is that the order of receipt is not always  easy to predict, especially when subpatchers are involved. One effective rule can be to use #0 receives on cold inlets (inlets that don’t trigger output upon receiving an input, usually all but the left inlet). This way, no output is triggered by the receiving object.

--From: Gary Lee Nelson, Peter McCulloch, Robin Parmar, Raphael Radna, and Michael Zbyszyński.



## Subpatchers and Abstractions
Keep  your number of inletslow, and parse inputs with **route** inside your subpatchers and abstractions. If you need multiple instances of a subpatcher, remember to reimplement it as an abstraction or in **bpatcher**, or **poly~**, so that if you need to make changes, editing the single file will update all its instances.

Of course, this means you should have a clear concept of what this abstraction should do before you begin building, and debug the patch thoroughly before using several instances of it. In the planning stage, try writing out examples of messages that the patch should understand, along with the output or actions caused by the inputs. This can save considerable time when you go to implement the abstraction because your design will take into account the functionality that you want.

--From: Peter McCulloch, Gary Lee Nelson, Robin Parmar, Raphael Radna, Kurt Ralske, and Michael Zbyszyński.



## Externals
Users are in two camps when it comes to using externals, and it appears to depend on their working situations: how portable projects have to be, whether projects have to be used by others, how much time is available, how long-lived projects need to be, or whether you’re using Max as a stage in prototyping.

Using external libraries certainly saves you from reinventing the wheel. However, when you have time, you can use them as opportunities to learn more and come closer to rolling your own. You may be amazed what you can learn by creating even the most basic objects yourself. One user loves coding externals just to use Max (as a wrapper) to test code with that will eventually be ported to other platforms like mobile applications.

Users might avoid externals in order to help ensure their projects will work when moving to other computers (e.g., studios, classrooms, laboratories, students’ or collaborators’ or customers’ computers, or computers dedicated to performance venues or installations), because they may behave differently on other operating systems (or not support them at all). Some of us have patches that have served us for a decade or more, relying on external developers to keep up with updates that may put your projects at risk in the long run. Even for making your own, consider developing with Gen instead. 

In a similar spirit, besides external objects, you might also avoid customizing workspaces as much as possible, so you don’t get tripped up (by relying on non-default behaviors) when working on a patch on another computer for an installation, in a lab, or on a performer’s computer, when stress is often high.

--From: Peter McCulloch Jeff Morris, Robin Parmar, and Timo Rozendal.



## poly~
Beyond letting you load a patcher as an object box in another patch, **poly~** gives you powerful dynamic control to load multiple instances of a patch, change patches, and control processing and communications, all on the fly. If your project needs to scale to different numbers of voices, **poly~** makes them easy to control, saves space, and can save CPU power. Even if each instance needs to do slightly different things, you can pipe in the individual variables with the “target” message. The **thispoly~** object outputs a unique voice number for each instance when it receives a bang. Use it with **zl.nth** to pick out the corresponding member of a list of values, or use it with **sprintf** to set a **receive** object to a unique value.

![Image8](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/poly.png)

Signal rate **poly~** is not to be confused with the control rate **poly** object. For polyphony, **poly** makes its data routing decisions based on when a note is released rather than when its amplitude envelope has finished sounding; it relies on the receiving synthesizer to know what to do in pertinent cases. This is especially problematic if notes have different release times.

In the spirit of “numbers are numbers,” **poly~** is useful outside of handling polyphonic instrument sounds. Remember that you can use it when you need multiple copies of any kind of patch, even with MIDI objects and Jitter objects. Use it to load different patches on the fly for major scene changes. With clever naming and routing schemes, you can use **poly~** for processing in series rather than in parallel, where each instance passes its output to the next instance, to perform sequential transformations.

--From: Julien Bloit, Vincent Goudard, Philippe Hughes, Todd Ingalls, Matt Kav, Peter McCulloch, and Gary Lee Nelson.



# Optimization
“Programmers waste enormous amounts of time thinking about, or worrying about, the speed of noncritical parts of their programs, and these attempts at efficiency actually have a strong negative impact when debugging and maintenance are considered. We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. Yet we should not pass up our opportunities in that critical 3%.” 
    --Donald Knuth[^4]
    
Optimization typically refers to minimizing the amount of processor power or memory resources a program uses. In a real-time context, some tradeoffs are unavoidable in order to ensure adequate timing. At extremes, this may make your program less convenient to use, more difficult to read, or less numerically accurate.

[^4]: Knuth, Donald. "Structured Programming with Goto Statements". Computing Surveys 6:4 (December 1974), pp. 261–301, §1



## Signals and Downsampling
If you try to save CPU power by using control-rate objects and avoiding signal-rate objects, you may overload the scheduler and disrupt the timing of critical processes. Some control-rate actions can get the scheduler clotted quite easily, but it's much harder to overload your CPU. When you do overload your CPU, it will be much easier to find a solution than when you've overloaded the scheduler. Once you get a hang of it, you can perform a lot of logic operations and controls in Max using only signal-rate objects. Signals have a higher upfront CPU cost than at the control rate, but the cost is relatively constant—it doesn’t generally cause spikes in CPU usage. If you optimize for worst-case run times, you end up with more stable real-time systems.

So, try to keep everything downstream from your first signal-rate object signal-based whenever possible. This makes sure that everything downstream will always execute the same, regardless of which machine it's running on or which other programs are running on that same machine at the same time. Instead of using **snapshot~** to convert a signal to the control rate, you could use **sah~** for similar purposes. If you’re converting from signal-rate to control-rate and back, look to see if there is a signal-rate alternative available. For further optimization, consider using signal-rate objects inside **poly~** and downsampling to reduce CPU usage. (Just be sure to turn off the resampling filters using the attribute @resampling 0.)

--From: Samuel Béland and Peter McCulloch.



## Reduce Unneeded Updates
General wisdom says to avoid using GUI objects as much as possible, for example **dac~** instead of **ezdac~**. These objects like **ezadc~**, **gain~**, and **snapshot~** can be great for quick demonstrations or tests, but they can drag down a patch’s speed. The **gate~** object often leads to clicks if you use it instead of some crossfading method. When using them, connect them to the side—don’t connect through these objects, so they can easily be deleted when they’re no longer needed for development. Also remember that it’s handy to double-click **dac~** to open the Audio Status window.

If you need to keep a **jit.pwindow** handy, use a gate to deactivate it when it’s not needed. If it is necessary to keep a GUI object active in your patch, see if you can adjust its refresh rate and make it as slow as you can. Similarly, for video, use **bline** instead of **line** so it does not update any faster than the video frame rate.

--From: Carlo Cattano, Jeffrey Clark, Sabina Covarrubias, Joshua Mark Goldberg, Kevin Kripper, Peter McCulloch, Jeff Morris, and Robin Parmar.



## Avoid Performance Hits
When using an interpreted environment such as **expr**, **if**, JavaScript, OpenGL shaders, etc., accomplish as much as as you can while inside that environment to minimize the performance hit when switching in and out of it. You might skip the **if** object altogether.

Use OpenGL objects, which do processing on the GPU instead of the CPU as much as possible, and have objects output textures instead of matrices where possible. It's amazing how fast you hit the CPU ceiling if you're doing everything using matrices rather than textures. If you don’t need the alpha channel in your video, always use the “@colormode uyvy” attribute when transitioning to the GPU; it reduces the data that needs to be handled. Finally, if you’re going to play any video larger than a 720p resolution, use Hap to load files directly into an OpenGL texture. For details, see [jit-gl-hap](https://cycling74.com/tools/jit-gl-hap/).

--From: Joshua Mark Goldberg, Peter McCulloch, Jeff Morris, and Sam Tarakajian.



# Production
Test recordings are helpful for  building, rehearsing, and evaluating pieces. If you have the overhead for it, record the input and output of the patch. When something goes wrong in performance, its useful  to hear what happened, and to have the same input signal to run through the patch. You might record each sound-making voice (or sub-groups of voices) to separate channels of a multichannel AIF or WAV file using **sfrecord~** so you can transport them as one file, keep all tracks in sync, and mix and edit them later in more detail. Of course, this will multiply the file size of your recordings. Max can separate the channels later, but some DAWs such as  Audacity and Reaper can too. The QuickRecord patch in the Extras menu can record up to 8 channels of output.

Build controls that you can easily access at any  time to reset your patch, mute audio, or perform other kinds of “panic button” features. If you have feedback delays in your piece, you can use the “clear” message to reset them. Try  to enable jumping to different points in your piece for the sake of rehearsal and sound checking. This can  be tricky, because time-based media doesnt  always handle  the same as stage lighting cues. When building a GUI for performer, standalone installation, or app, don't forget to create a control to open the Audio Status window (bang an “open” message into **dac~**). Even if you leave the default menus in place, the Audio Status window isn’t likely to be where a nervous troubleshooting performer might  look. 

![Image9](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/opendac.png)

Keep the performance interface simple. If cues are advanced sequentially, consider using the spacebar to trigger the advance. Clicking on cue numbers with a mouse or trackpad requires extra attentional bandwidth.

Consider t some things that might change with your piece from one performance to another. For example, in pieces that rely on reverb, provide a trim control for the input or output of the reverb so that you can freely adjust them to fit the acoustics of each venue. This can also be helpful with delays. 

For productions, make sure your I/O Vector Size, Signal Vector Size, and Sampling Rate are set the way  you need them at load time. Historically, bad crashes have been known to change these settings. If your patch is  used on other people’s computers, you might use **closebang** to set them back where they were. You can use **adstatus** or **dspstate~** to read and set these values from within your patch.

Remember that Max runs on two platforms. If your patch doesn’t work on one platform or hasn’t been tested on it yet, make that clear to anyone who downloads  it.

And finally, the “dirty” trick, attributed to Cort Lippe: Set the dirty bit in a patcher, either by making any change to the patch or by sending the message “dirty” to **thispatcher**. This way, if you accidentally close the patch or quit Max in the heat of a performance, Max will present a “Save changes…?” dialog box instead of closing.

![Image10](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/dirtytrick.png)

![Image11](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/dirtytricksave.png)

--From: Peter McCulloch, Cort Lippe, Barry Moon, Jeff Morris, and Robin Parmar.



# More Specific Tips
When using **function** for amplitude envelopes: your  amplitude envelope probably starts at 0., which would cause a click if you triggered the amplitude envelope again before the first envelope had finished. There are a number of ways to skip that first 0. Here is  the leading approach from the discussion: Set the outputmode attribute of **function** to “list” and connect **zl.slice 2** between **function** and **line~**, using the right outlet of **zl.slice**. If you’re using **curve~** instead of **line~**, use **zl.slice 3**.

![Image12](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/functionclick.png)

When you need to directly control the location of playback for sampling, consider implementing playback in terms of samples. This  logic is (often) easier to reason with, even though the numbers will get big. For example, connecting **sig~ 1** to **+=~** to **pong~ @mode wrap** to **index~** will give you looping playback positions, and you can set the start and end samples in the second and third inlets of **pong~**, respectively. The value of **sig~** is the playback speed. (Next, consider using **gen~** in lieu of **index~** for playback because **index~** doesn’t interpolate between samples.) You can also use **clip~** instead of **pong~** to play once without looping; bang **+=~** to restart playback manually. For recording, use **poke~** instead of **index~**.

![Image13](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/sampleplayback.png)

Instead of switching between multiple signals, see if you can use a weighted sum, basically crossfading between them instead. For example, rathern than  choosing between three LFO shapes via **selector~**, combine them using **matrix~ 3 1 0**. This works in many  places! Try using **matrix~** with **matrixctrl**, with the dialmode attribute enabled for flexible signal routing.

Here are two interesting feedback suppression techniques: Use signal-rate objects to run your signal through this arithmetic: y = x / (1 + 0.28*x^2). It asymptotically approaches zero above a certain input gain instead of hard clipping. 

![Image14](https://drive.google.com/drive/folders/1MC-iLjXCmDaj2R_8ai955MvhBn__d8i4/feedbacksupression.png)

A more transparent technique is to use the smoothed amplitude envelope of the audio output(s) to control the amount of frequency shift applied to each output via a **freqshift~** object. You can use the **clip~** and **scale~** technique to keep it from kicking in below a certain level. By shifting the spectrum of the signal, the feedback is dispersed. (This is a John Chowning trick, if I recall correctly.)

A little JavaScript for things like control-rate list processing and arithmetic can be convenient and make for much tidier patches.

Finally, **jit.bfg** has a wealth of potential. Check it out in Jitter Tutorial 50, and then search to see what others have done with it for inspiration.

--From: Joshua Mark Goldberg, Todd Ingalls, and Peter McCulloch.

# Tips for Growth
Treat your best practices list as a trigger list (after David Allen): you’ll never remember it all, so set a recurring reminder so that you reread  it occasionallyfor anything  that could improve your recent practices or help avoid recent problems.

Learn at least one other programming language. Max connects to a variety of programming languages, and sometimes a different language can help solve  problems, or parts of problems that are less idiomatic in Max. As with learning new musical instruments or natural languages, each one can help you  learn the next and understand them all better.

Finally, follow any of the active Max user forums you find, including Cycling74’s own discussion forum. Instead of using the Cycling74 forum search engine, you might Google "your search keywords site:https://cycling74.com/forums/ " or maybe just add “cycling74” to your search terms.

--From: Carlo Cattano, Peter McCulloch, and Jeff Morris.
