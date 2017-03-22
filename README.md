# wacom-profile-daemon
A tool for automatically switching graphics tablet options and area mapping based on the active window. 

# Overview
This simple script runs in the background and applies custom tablet settings depending on the rules
specified in a configuration file. It supports configurations for multiple tablet devices at once.

# Installation
All you need is a copy of the `wacom-profile-daemon.py` file. Put it anywhere you like and make it executable.
The script requires python3 and [xsetwacom](http://linuxwacom.sourceforge.net/wiki/index.php/Tablet_Configuration) 
(libwacom) to be installed to your environment.

A sample configuration file (`sample-config.json`) is also provided.

# Usage
If you've saved the script on your `PATH`, it's as simple as:
```
wacom-profile-daemon.py --daemon path-to-config-file
```

# The Configuration File
The configuration file defines the rules and settings the daemon will follow. It is in 
[JSON](http://beginnersbook.com/2015/04/json-tutorial/) format.

I recommend taking a look at the 
[sample configuration file](https://github.com/cluracan/wacom-profile-daemon/blob/master/sample-config.json)
provided first.


The general hierarchy follows. Note that it is in an abbreviated format.
```
"device prefix"*:
  "rule name"*:
    "window-title"?: "regular expression",
    "window-class"?: "class name",
    "window-id"?: "window id",
    "mapping"?: "window"|"monitor index"|"monitor id",
    "pad"?: [ "option command"* ],
    "stylus"?: [ "option command"* ],
    "eraser"?: [ "option command"* ]
```

The configuration is first divided into rulesets whose keys are device prefixes. A ruleset with a key of 
`"Wacom Cintiq"` will match the devices `"Wacom Cintiq 13HD"`, `"Wacom Cintiq 22HD"`, and so on.
You can use `xsetwacom list devices` to find a suitable device prefix. Leave out suffixes like "Pen" or "Pad"
to allow the ruleset to apply to all components of that device.

The next layer is composed of rules. The names of the rules don't matter, but they should be descriptive.

A rule may contain a matcher (`"window-title"`, `"window-class"`, `"window-id"`). These will be used to match
the window with focus. Multiple matchers will be ANDed together. Other parts of a rule define mapping and
key bindings that will be applied upon a successful match.  
If a rule does not contain a matcher, it will always be applied, and will effectively be a "default" rule.  
Multiple rules may be applied at a time. Rules are applied from least specific to most specific.

* `"window-title"` matches a window whose title matches the 
[regular epxression](http://www.regular-expressions.info/tutorial.html) provided. 
* `"window-class"` matches a window whoses class matches the string provided.
* `"window-id"` matches a window whose id matches the string provided.

The `xprop WM_NAME` or `xprop WM_CLASS` commands are useful in determining the exact title 
or classs of a window.

The `"mapping"` option remaps the area of the tablet to either an entire display or a window. Aspect-ratio
will be maintained while utilizing as much of your tablet as possible. This option
is obviously not recommended for Cintiq and other pen-display devices.
 
* A value of `"window"` will remap the tablet to the active window. Your pen will only be able to
move within the window, potentially giving you more tablet resolution to work with.
* An identifier (such as `"HDMI-0"`, or `"DP-1"`) will remap the tablet to the
display that identifier belongs to. You can find the identifiers for all your displays with the `xrandr`
tool.  
* A number will be treated as an index to a connected display as outputed by `xrandr`. Hence, `0` will
  be the first connected display, `1` will be the second, and so on.

The `"pad"`, `"stylus"`. and `"eraser"` lists allow you to specify settings to apply to pad, stylus, 
and eraser devices, respectively. Each string is passed directly to `xsetwacom` like 
`xsetwacom set DEVICE-ID YOUR-STRING-HERE`. You can get a full list of settings available for a particular
device/component with `xsetwacom -s get DEVICE-ID-OR-NAME all`. Take a look at the 
[xsetwacom wiki](http://linuxwacom.sourceforge.net/wiki/index.php/Tablet_Configuration) for a detailed
rundown of settings and syntax.

As an example, here is a full configuration for a Wacom Intuos S 2 device that maps the tablet to the
first active display and binds button 2 on the pad
to undo (`ctrl+z`) by default, but binds button 2  to the key `e` when in a window whose title ends with 
"MyPaint" comes into focus.

```
{
  "Wacom Intuos S 2": {
    "default-rule": {
      "mapping": 0,
      "pad": [
        "Button 2 'key ctrl z'"
      ]
    },
    "mypaint-rule": {
      "window-title": "\bMyPaint$",
      "pad": [
        "Button 2 'key e'"
      ]
    }
  }
}
```