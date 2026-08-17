# **Introduction to PsychoPy** 
 
### [Matteo De Matola](https://github.com/matteo-d-m) 

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/coneco-lab/open-lab-toolkit/blob/main/tutorials/psychopy-tutorial.ipynb)

This notebook is a quick introduction to PsychoPy, a Python library to program psychological experiments. It aims to illustrate the 5 steps that must be taken to set up a standard cognitive neuroscience experiment, where subjects watch some kind of visual stimuli during neuroimaging. 

The material in this notebook was taught by me ([Matteo De Matola](https://github.com/matteo-d-m)) in Lecture 3.0 of [Python for (open) Neuroscience](https://github.com/vigji/python-cimec-2024) (2024 iteration), a course taught by Luigi Petrucco at the University of Trento's doctoral school in Cognitive \& Brain Sciences. 

## **[PsychoPy](https://psychopy.org/) is a Python library to program psychological experiments**  

![PsychoPy logo](https://psychopy.org/_static/Psychopy%20Logotext.png)

Like many Python libraries, PsychoPy is:
- Free
- Open source
- The result of a large-scale collaborative effort

If you use PsychoPy in published work, remember to cite the authors:

[Peirce, J., Gray, J. R., Simpson, S., MacAskill, M., Höchenberger, R., Sogo, H., ... & Lindeløv, J. K. (2019). PsychoPy2: Experiments in behavior made easy. Behavior Research Methods, 51, 195-203](https://link.springer.com/article/10.3758/s13428-018-01193-y)

PsychoPy contains functions and classes to:
- Generate visual stimuli
- Generate auditory stimuli
- Time events
- Interact with external hardware, including:
    - Keyboards
    - Response boxes
    - Cameras
    - Eye tracking devices
    - Neuroimaging systems
- & more

Research at [CoN&Co](https://www.cimec.unitn.it/en/CoNeCo) involves people watching things on a screen and pressing buttons during neuroimaging.

Therefore, this notebook will focus on:

- Generating visual stimuli 
- Representing the screen and interacting with it
- Presenting stimuli with the right timing
- Representing different experimental conditions
    - Their balance
    - Their order
- Representing buttons and encoding buttonpresses
- Representing the neuroimaging system and interacting with it

Unfortunately, PsychoPy does not work well when run from Jupyter notebooks like this one. Therefore, you should use this notebook as reading material and write your PsychoPy code in a separate Python (`.py`) file. **Don't try to run code cells from this notebook, or you will encounter errors that might get you stuck on frozen windows (and you might have to Ctrl+Alt+Canc your way out...)**

# **Step 1: Generating Visual Stimuli**

## **The `visual` module**

The `visual` module is a subsection of PsychoPy that contains classes and functions to:

- Draw a variety of geometric stimuli 
    - `ShapeStim`
        - `Rect`
        - `Circle`
        - `Polygon`
        - `Line`
        - `Pie`
- Display images and draw patterns
    - `SimpleImageStim`
    - `GratingStim`
    - etc.
- & more: go [here](https://psychopy.org/api/visual/index.html) for the full list of options and the associated documentation

Stimuli are drawn on a specific _window_, which you can think of as one of the usual windows on your computer (e.g., a browser window or a Microsoft Word window). 
This window can be as large as the screen, or smaller.  

Opening a window is as simple as:


```python
from psychopy import visual

window = visual.Window(color="black")
```

`Window` objects represent the canvas where you draw your stimuli.   

But that canvas cannot exist without some hardware to support it. That hardware is your monitor (or _screen_).

Different monitors have different physical properties. The most basic ones are:
- Height 
- Width
- Pixel resolution

Moreover, different monitors can be used in different ways: 
- Distance from screen
    - Vital if your stimuli are in [degrees of visual angle (°VA)](https://en.wikipedia.org/wiki/Visual_angle)

Specify the physical properties of your monitor and how you use it, or the `Window` class will be instantiated with default settings that might be suboptimal for you!


```python
from psychopy import monitors

MY_SCREEN_NAME = "a_coneco_screen"
MY_PIXEL_WIDTH = 1366
MY_PIXEL_HEIGHT = 768
MY_SCREEN_WIDTH = 47 # cm
SUBJECT_DISTANCE_FROM_SCREEN = 65 # cm

monitor = monitors.Monitor(name=MY_SCREEN_NAME)
monitor.setSizePix([MY_PIXEL_WIDTH, MY_PIXEL_HEIGHT])
monitor.setWidth(MY_SCREEN_WIDTH)
monitor.setDistance(SUBJECT_DISTANCE_FROM_SCREEN)                    
monitor.saveMon()                    
```

Once you have set monitor properties and initialised a window, you can draw a stimulus on it. 

For example, you can draw a white rectangle on a black background:


```python
window = visual.Window(color="black")
stimulus = visual.rect.Rect(win=window,
                            fillColor="white")
```

# **Step 2: Present Stimuli with the Right Timing**

Timing can be very important. You need to make sure that:
- Stimuli appear when you want them to
- Stimuli remain on for as long as you want them to
- Stimuli are synchronised to the physiological events of interest
    - For example: if you are doing EEG, your stimuli will be probably interleaved by milliseconds (instead of seconds). But if you are doing fMRI, they will be interleaved by seconds

Each stimulus must have a given display time (measured in seconds).

For example:


```python
display_time = 0.1 # one tenth of a second = 100 ms
```

This display time must be transformed into a number of frames.
- A frame is one of many still images that compose a complete moving picture ([Wikipedia](https://en.wikipedia.org/wiki/Film_frame))

Importantly, different screens can display different numbers of frames in one second
- The number of frames displayed per second is the frame (or _refresh_) rate (Hz) of your screen ([Wikipedia](https://en.wikipedia.org/wiki/Frame_rate))
    - Often used interchangeably, but _refresh rate_ is better when talking about screens (and _frame rate_ when talking about GPUs) 

To transform a display time into a number of frames, you must multiply it by your screen's refresh rate:


```python
REFRESH_RATE = 60 # Hz
frames_per_stimulus = display_time*REFRESH_RATE
```

The number of frames per item will serve as a range to iterate over.

This means that you basically tell your machine: "Display this item for as many frames as you can fit in a given time interval":


```python
for frame in range(frames_per_stimulus):
    # display item
```

But, how do you actually _display_ an item?

- The `flip()` method of `Window` objects
    - Move between frames 
- The `setAutoDraw()` method of all `ShapeStim` objects
    - Draw the stimulus on every frame, without explicit frame-by-frame instruction


```python
window = visual.Window(color="black")
stimulus = visual.rect.Rect(win=window,
                            fillColor="white")

stimulus.setAutoDraw(True)
for frame in range(frames_per_stimulus):
    window.flip()
```

**Interim exercise:**

- If you don't have a Python installation: 
    - Get one following the instructions at [this HTTPS URL](https://github.com/vigji/python-cimec-2024/blob/main/docs/python-installation.md)
- If you have a Python installation, but don't have PsychoPy:
    - Open an Anaconda prompt or terminal
    - Optional: create a dedicated `conda` environment
    - Type `pip install psychopy`
- In any code editor, create a `.py` file where you:
    - Initialise a `Monitor` with a given name, then set its pixel size
    - Initialise a `Window` object as follows:
        - `window = visual.Window(size=monitor.getSizePix(), fullscr=True, monitor=monitor.name, color="black")`
    - Create a stimulus of your choice 
    - Display the stimulus for 10 seconds at a 60 Hz refresh rate
    - Save the `.py` file and run it(from either the code editor or terminal)

Note: a code editor is something like [VSCode](https://code.visualstudio.com/download), [PyCharm](https://www.jetbrains.com/pycharm/), or [Spyder](https://www.spyder-ide.org/). 

If you don't have a code editor, you can use Notepad (on Windows), TextEdit (on Mac), or Text Editor (on Linux).

# **Step 3: Represent Different Experimental Conditions**

In neuroscientific experiments, we usually study how a subject responds to some stimulation (sensory, emotional, magnetic, electrical, etc.).

For statistical power, we want to measure the same response several times. 

Therefore, we want an experiment with multiple _trials_: stereotyped, repeated presentations of the same stimulus or stimulus sequence.

We also want to study how a subject responds to different _kinds_ of stimulation (e.g., seeing a square vs. seeing a circle).

Therefore, we want an experiment with multiple _conditions_: variations of the stimulus or stimulus sequence presented to the subject in any given trial.

In this context, we need code to:
- Define different conditions

- Administer each condition for multiple trials
    - The number of trials per condition will vary depending on our design: balance
    
- Administer the conditions in the order that we want 
    - That's often random

## **Define Different Conditions, Step 1: The `conditions.csv` File**

In PsychoPy, you need to define your conditions in an external `.csv` file that has:
- One row per condition
- One column per condition parameter:
    - Presence vs. absence of a given stimulus (1,0)
    - Colour of the stimulus ("yellow", "blue",...)
    - Whatever you can think of...

If you want to give higher weight to one condition (i.e., present it more times than others), you can put it in multiple rows 

### **Define Different Conditions, Step 2: The `data.importConditions` Function**

Each experimental trial corresponds to one condition. Therefore, each experimental trial is one row of the `conditions.csv` file.

PsychoPy implements trials as iterations over the rows of the `conditions.csv` file.

To do so, it reads the `conditions.csv` file and transforms it in an [iterable](https://realpython.com/python-iterators-iterables/#getting-to-know-python-iterables) (i.e., a Python object that you can iterate over).

This is done by the `importConditions` function from module `data`. 


```python
from psychopy import data

CONDITIONS_FILE_NAME = "conditions.csv"
my_conditions = data.importConditions(CONDITIONS_FILE_NAME)
```

### **Administer Each Condition for Multiple Trials, in the Order You Want: The `data.TrialHandler` Class**

The output of `data.importConditions` is meant to be the input to `data.TrialHandler`.

The class `TrialHandler` from module `data` implements methods to:
- Iterate over conditions (as read by `data.importConditions`)
- Do it as many times as you want (one trial per condition vs. multiple trials per condition)
- Do it in the order you want (sequential vs. random)


```python
my_trials = data.TrialHandler(trialList=my_conditions, 
                              nReps=1,
                              method="random")
for trial in my_trials:
    # do things
```

**Pay attention to the `method` argument:**

- You probably want to pass `random` to avoid order effects:

    shuffles the conditions, and _"all conditions occur once before the second repeat"_
- You can also pass `fullRandom`, but be aware that 

    _"you could potentially run all trials of one condition before any trial of another"_

- Finally, you can pass `sequential` if that's OK for your design. 

    _"presents the conditions in the order they appear in the list"_ (i.e., in the `conditions.csv` file)

# **Step 4: Represent Buttons and Encode Buttonpresses**

The `event` module contains classes and functions to _"handle input from keyboard, mouse and joystick"_

Of particular interest for standard experimental psychology, the `getKeys()` function continuously checks for keypresses and continuously returns a list of the presses that it finds.
- For example: if you press `left`, `getKeys()` will detect it and return a list of type: `["left"]`
- You can specify the keys that you are interested in with the `keyList` argument (`type: list`). 
    - If you do so, `getKeys()` will check exclusively for the keys contained in `keyList`


```python
from psychopy import event

pressed_key = event.getKeys(keyList=["left","right"]) 
```

Once you have stored the keypress in a variable, you can use it for response scoring:


```python
response = pressed_key[0]
if response == "left":
    correct = 1        # means "yes"
elif response == "right":
    correct = 0        # means "no"
```

## **Measuring Response Timing**

Often, response timing is important.
- Example: reaction time is a dependent variable

PsychoPy provides a wealth of tools to measure time. These tools are scattered across the `core` and `clock` modules. I believe this might change in the future, with all timing tools migrating to `clock`. In the meantime, you can visit [`core`](https://www.psychopy.org/api/core.html)'s and [`clock`](https://www.psychopy.org/api/clock.html)'s documentation to choose the tool that suits your needs.

### **The `Clock` class from the `core` module**

Pseudocode:


```python
from psychopy import core

response_clock = core.Clock()
for trial in my_trials:
    # present your stimuli 
    if response:
        reaction_time = response_clock.getTime()
        response_clock.reset
```

Resetting the clock after taking the time is **VERY** important. 

If you forget to do it, you will end up taking incremental times:

- First reaction time: `clock.getTime()` - $t_{init}$

    where $t_{init}$ is the time when you initialised your clock (i.e., when `response_clock = core.Clock()` was executed)
- Second reaction time: `clock_getTime()` + first reaction time
- Third reaction time: `clock_getTime()` + second reaction time
- etc...

What you really want is: 
- Any reaction time: `clock.getTime()` - $t_{start}$

    where $t_{start}$ is the time when you presented you stimulus (i.e., the beginning of a frame or trial)

Real code:


```python
from psychopy import visual, core, event

window = visual.Window(color="black")
stimulus = visual.rect.Rect(win=window,
                            fillColor="white")

response_clock = core.Clock()

for trial in my_trials:
    stimulus.setAutoDraw(True)
    window.callOnFlip(response_clock.reset) # analogous to setAutoDraw
    event.clearEvents()                     # clears old  keypresses 
    for frame in range(frames_per_stimulus):
        keys = event.getKeys(keyList=["left","right"])
        if len(keys) > 0:
            response = keys[0]   
            reaction_time = response_clock.getTime() 
            break
        window.flip()
```

**Interim exercise:**

Extend the script from the previous exercise. You want to:
- Accept responses 
- Take reaction times
- Print the reaction time

When you are done with your modifications, run the script to see if it works!

# **Step 5: Represent the Neuroimaging System and Interact With It**

When you do a behavioural experiment during neuroimaging, you want a link between neuroimaging and behavioural data.
For example, you might want to:
- Mark EEG time series at stimulus presentation times
- Present stimuli at the right moment of an MRI acquisition sequence

**Problem:** 

- The PsychoPy code for the behavioural task runs in one computer
- The neuroimaging data are acquired by another / the MRI scans are controlled by another
- You need a piece of hardware to connect two computers and carry signals between them

The connection between the two computers is often implemented by a parallel or serial port.

Simplifying:
- A parallel port is a hardware interface that can transmit multiple signals in parallel. It is composed of multiple _pins_ connected to a cable with multiple threads (one per pin) ([Wikipedia](https://en.wikipedia.org/wiki/Parallel_port))
<p align="center">
  <img src="./files/parallel-port.png" />
</p>

- A serial port is a hardware interface that can transmit one signal at a time ([Wikipedia](https://en.wikipedia.org/wiki/Serial_port))
  - USB ports are serial: USB = Universal Serial Bus

<p align="center">
  <img src="./files/serial-port.png" width="450" height="auto" />
</p>

PsychoPy has a `hardware` module that contains:

- The `parallel` submodule, to send signals through parallel ports
- The `serial` submodule, to send signals through serial ports

We'll make the example with `parallel`, showing how to mark an EEG trace through a parallel port. However, `serial` is similar in concept. In both cases, you need to:
- Import the appropriate submodule (`parallel` / `serial`)
- Initialise an object that represents your port. This object will have one key attribute: the port's _address_ (basically, its name)
- Write code to have signals travelling from / to the port

Example: marking an EEG trace through a parallel port


```python
from psychopy import parallel

port = parallel.ParallelPort(address="0x0378") # can vary
port.setPin(pinNumber=1,                       # send signal through pin number 1
            state=1)
# display your stimulus
port.setPin(pinNumber=1,                       # stop sending signal through pin number 1
            state=0)
```

You can go [here](https://github.com/coneco-lab/modified-ant/blob/main/task-and-eeg/docs/neurone-triggers.md) for a bit more detail on marking EEGs through parallel ports...

... And you can go [here](https://psychopy.org/hardware/fMRI.html) for a tutorial on synchronising stimulus presentation with the activity of MRI scanners (includes code for serial ports, which applies beyond the MRI use case). 

# **Bonus: The Importance of Timing**

We have covered PsychoPy's basic concepts. Knowing about them is enough to start building an experiment. 

But if you do build an experiment, you need to make sure it works as intended. To this end, time is of the essence: the behavioural task **must** be synchronised with a subject's brain dynamics.

Enemies:
- Software delays
- Hardware delays

## **Software Delays**

Any instruction (i.e., code line) takes time to execute. You must make sure that the execution time of instructions suits your scientific needs.

Example:
- You want the transition between two stimuli to take 5 ms

- You have a scientific reason to want that 
    - For example, you expect the two stimuli to be synchronised with neural events that occur 5 ms apart

- Your code takes 7 ms to execute 
    - Unlikely, but I'm just making an example
    
- You have a problem!

### **Time your code**

There are ways to time code execution, in and outside PsychoPy: 
- The `time` library 
- PsychoPy-specific tools

**The `time` library**
- **First tool: `time.time()`**


```python
import time

start_time = time.time()
# do things
end_time = time.time()
time_taken_by_things = end_time - start_time
```

Calls to `time.time()` return the number of seconds since the Epoch, i.e., January 1, 1970 at midnight (an arbitrary reference value). 

Calling `time.time()` two consecutive times will return two incremental numbers (i.e., $ t $ and $ t+\tau $).

The difference between such numbers is the time elapsed between the two calls to `time.time()`: $ \tau = (t + \tau) - t $  

Calls to `time.time()` are great for code with execution time in the order of the seconds. If you expect your code to run faster than that (e.g., because you are timing a very simple instruction), `time.time()` may not be precise enough. From `time.time()`'s [documentation](https://docs.python.org/3/library/time.html): 

_"Note that even though the time is always returned as a floating point number, not all systems provide time with a better precision than 1 second. While this function normally returns non-decreasing values, it can return a lower value than a previous call if the system clock has been set back between the two calls"_

**The `time` library**
- **Second tool: `time.perf_counter()`**

Luckily, the `time` library provides a more precise stopwatch than `.time()`. That is `time.perf_counter()`, whose documentation says: 

_"Return the value (in fractional seconds) of a performance counter, i.e. a clock with the highest available resolution to measure a short duration"_


```python
from time import perf_counter

start_time = perf_counter()
# do things
end_time = perf_counter()
precise_time_taken_by_things = end_time - start_time
precise_time_taken_by_things
```

There's even a `time.perf_counter()` variant that returns the time in nanoseconds: `time.perf_counter_ns()`. It works the same as `perf_counter()`, but it returns the time in nanoseconds (instead of fractions of a second). This means that instead of getting $ \tau = 0.000000001 \ s $, you will get $ \tau = 1 \ ns $. In practice, these two values are the same. But the value expressed in fractional seconds may be less precise because it is encoded as a `float` (read [here](https://realpython.com/python-data-types/#floating-point-numbers-representation) for more).  

**PsychoPy-specific timing tools:**
- **Clocks (see above)**
- **`core.getTime()`**
    - Gets the time since the `core` module of PsychoPy was loaded in the current working session

**Beware of estimation error**

In general, the execution speed of any instruction can vary between runs:
- Code quality
- Available CPU resources

Therefore, all time measurements carry some uncertainty. Ideally, you should time the same code multiple times and calculate statistics on the results you obtain.

## **Hardware Delays**

Even if your software is perfect, there might be delays caused by the hardware. Example: 
- You are running a visual experiment
- The monitor takes some milliseconds to render stimuli
    - This results in:
        - A delay between the time a stimulus is created in code and the time it is rendered on screen
        - A mismatch between the expected and real stimulus presentation time
        - If you are running an EEG experiment, you are marking the trace at the wrong time 

### **Measuring Stimulus Presentation Time: The Photodiode**

A photodiode ([Wikipedia](https://en.wikipedia.org/wiki/Photodiode)) is an electronic component that generates a current when it's hit by light. If you place a photodiode in front of a stimulus' screen location, you will get an instantaneous signal when the stimulus appears (provided sufficient contrast between the stimulus and the background). This will allow you to check for delays between expected presentation time and the real presentation time:
- Real presentation time measured by the photodiode
- Expected presentation time measured by the marker on the EEG trace 
    - Marking an EEG trace via parallel port is almost instantaneous, so no delays here

Using a photodiode can be tricky, especially because you should connect it to an oscilloscope. You might want to seek help from a lab technician for that.

# **Final Considerations**

## **PsychoPy is Great**

- You can run virtually any type of visual / auditory experiment
- You can interact with quite a lot of hardware
- All you need is a bit of Python expertise 
    - The material covered in [this](https://github.com/vigji/python-cimec-2024) course should be enough to get you on ramp!
    - If you want a real-world example, you can check out our own code at [this](https://github.com/coneco-lab/modified-ant) GitHub repository

## **PsychoPy has Flaws**

- If you follow the developers' advice, you will install the Standard PsychoPy Version (SPV). This will get you:
    - A code editor called Coder
    - A GUI called Builder, where you can build experiments by pointing-and-clicking
    - A kind of terminal / process manager called Runner

- In our own opinion, the SPV is bad because:
    - It comes with its own Python installation. If you already have another, things might get messy 
    - The Coder is a very bad editor. Alternatives like [VSCode](https://code.visualstudio.com/) have more features and are easier to use
    - The Builder encourages users to avoid coding (more on this below) 

- The Builder is meant to:
    - Empower people who can't code
    - Accelerate prototyping for the people who can code
    - Run [online experiments](https://www.psychopy.org/online/index.html), for which it is actually required
        - Running online experiments? Check out [this crib sheet](https://discourse.psychopy.org/t/psychopy-python-to-javascript-crib-sheet/14601)

- Personal opinion: unless you are running online experiments, you should never use the Builder 
    - Coding is more flexible (i.e., it lets you do more things)
    - Coding builds a better understanding of what you are doing
        - We should all be coding everything we use
        - We shall strive to eliminate black boxes. We can't know everything, but we can always try

- PsychoPy developers promote the Builder heavily, so most online tutorials are Builder-centred
    - Bad news for coders
    - The documentation for PsychoPy's codebase is:
        - Hard to find, as it's buried under a myriad articles about the GUI ([found it!](https://www.psychopy.org/api/index.html))
        - Poorly maintained (personal opinion)
