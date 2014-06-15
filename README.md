# tween.lua

[![Build Status](https://travis-ci.org/kikito/tween.lua.svg?branch=master)](https://travis-ci.org/kikito/tween.lua)

Tweening in Lua. Based on "jQuery's animate function"(http://api.jquery.com/animate/

# Examples

```lua
local tween = require 'tween'

-- increase the volume of music from 0 to 5 in 10 seconds
local music = { volume = 0, path = "path/to/file.mp3" }
local musicTween = tween.new(10, music, {volume = 5})
...
musicTween:update(dt)

-- make some text fall from the top of the screen, bouncing on y=300, in 4 seconds
local label = { x=200, y=0, text = "hello" }
local labelTween = tween.new(4, label, {y=300}, 'outBounce')
...
labelTween:update(dt)

-- fade background from white to black and foregrond from black to red in 2 seconds
-- Notice that you can use subtables with tween
local properties = {bgcolor = {255,255,255}, fgcolor = {0,0,0}}
local fadeTween = tween.new(2, properties, {bgcolor = {0,0,0}, fgcolor={255,0,0}}, 'linear')
...
fadeTween:update(dt)
```

# Interface

## Tween creation

``` lua
local t = tween.new(time, subject, target, [easing])
```

Creates a new tween.

* `time` means how much the change will take until it's finished. It must be a positive number.
* `subject` must be a table with at least one key-value. Its values will be gradually changed by the tween until they look like `target`. All the values must be numbers, or tables with numbers.
* `target` must be a table with at least the same keys as `subject`. Other keys will be ignored.
* `easing` can be either a function or a function name (see the easing section below). It's default value is `'linear'`
* `t` is the object that must be used to perform the changes - see the "Tween methods" section below.

This function only creates and returns the tween. It must be captured in a variable and updated via `t:update(dt)` in order for the changes to take place.

## Tween methods

``` lua
local complete = t:update(dt)
```

Gradually changes the contents of `subject` to that it looks more like `target` as time passes.

* `t` is a tween returned by `tween.new`
* `dt` must be a positive number. It will be added to the internal time counter of the tween. Then `subject`'s values will be updated so that they approach `target`'s using the selected easing function.
* `complete` is `true` if the tween has reached its limit (its *internal clock* is `>= time`). It is false otherwise.

When the tween is complete, the values in `subject` will be equal to `target`'s. The way they change over time will depend on the chosen easing function.

This method is roughtly equivalent to `t:set(self.clock + dt)`.

``` lua
local complete = t:set(clock)
```

Moves a tween's internal clock to a particular time.

* `t` is a tween returned by `tween.new`
* `clock` is a positive number or 0. It's the new value of the tween's internal clock.
* `complete` works like in `t:update`; it's `true` if the tween has reached its end, and `false` otherwise.

If clock is greater than `t.time`, then the values in `t.subject` will be equal to `t.target`, and `t.clock` will
be equal to `t.time`.


``` lua
t:reset()
```

Resets the internal clock of the tween back to 0, resetting `subject`.

* `t` is a tween returned by `tween.new`

This method is equivalent to `t:set(0)`.

# Easing functions

Easing functions are functions that express how slow/fast the interpolation happens in tween.

tween.lua comes with a lot of easing functions already built-in (adapted from "Emmanuel Oga's easing library"(https://github.com/EmmanuelOga/easing

They can be divided into several big families:

* `linear` is the default interpolation. It's the simplest easing function.
* `quad`, `cubic`, `quart`, `quint`, `expo`, `sine` and `circle` are all "smooth" curves that will make transitions look natural.
* The `back` family starts by moving the interpolation slightly "backwards" before moving it forward.
* The `bounce` family simulates the motion of an object bouncing.
* The `elastic` family simulates inertia in the easing, like an elastic gum.

Each family (except `linear`) has 4 variants:
* `in` starts slow, and accelerates at the end
* `out` starts fast, and decelerates at the end
* `inOut` starts and ends slow, but it's fast in the middle
* `outIn` starts and ends fast, but it's slow in the middle

| family | in    | out     | inOut    | outIn    |
| [linear](https://github.com/kikito/tween.lua/raw/master/graphs/linear.png)   |linear   |linear     |linear      |linear      |
| [quad](https://github.com/kikito/tween.lua/raw/master/graphs/quad.png)       |inQuad   |outQuad    |inOutQuad   |outInQuad   |
| [cubic](https://github.com/kikito/tween.lua/raw/master/graphs/cubic.png)     |inCubic  |outCubic   |inOutCubic  |outInCubic  |
| [quart](https://github.com/kikito/tween.lua/raw/master/graphs/quart.png)     |inQuart  |outQuart   |inOutQuart  |outInQuart  |
| [quint](https://github.com/kikito/tween.lua/raw/master/graphs/quint.png)     |inQuint  |outQuint   |inOutQuint  |outInQuint  |
| [expo](https://github.com/kikito/tween.lua/raw/master/graphs/expo.png)       |inExpo   |outExpo    |inOutExpo   |outInExpo   |
| [sine](https://github.com/kikito/tween.lua/raw/master/graphs/sine.png)       |inSine   |outSine    |inOutSine   |outInSine   |
| [circle](https://github.com/kikito/tween.lua/raw/master/graphs/circle.png)   |inCirc   |outCirc    |inOutCirc   |outInCirc   |
| [back](https://github.com/kikito/tween.lua/raw/master/graphs/back.png)       |inBack   |outBack    |inOutBack   |outInBack   |
| [bounce](https://github.com/kikito/tween.lua/raw/master/graphs/bounce.png)   |inBounce |outBounce  |inOutBounce |outInBounce |
| [elastic](https://github.com/kikito/tween.lua/raw/master/graphs/elastic.png) |inElastic|outElastic |inOutElastic|outInElastic|

You may want to give a look to the graphs folder on this repository to get a better idea of what I'm talking about.

When you specify an easing fucntion, you can either give the function name as a string. The following two are equivalent:

```lua
local t1 = tween.new(10, subject, {x=10}, tween.easing.linear)
local t2 = tween.new(10, subject, {x=10}, 'linear')
```

But since `'linear'` is the default, you can also do this:

```lua
local t3 = tween.new(10, subject, {x=10})
```

# Gotchas / Warnings

* `tween` does not have any defined time units (seconds, milliseconds, etc). You define the units it uses by passing it a `dt` on `tween.update`. If `dt` is in seconds, then `tween` will work in seconds. If `dt` is in milliseconds, then `tween` will work in milliseconds.
* `tween` can work on deeply-nested subtables (the "leaf" values have to be numbers in both the subject and the target)

# Installation

Just copy the tween.lua file somewhere in your projects (maybe inside a /lib/ folder) and require it accordingly.

Remember to store the value returned by require somewhere! (I suggest a local variable named tween)

``` lua
local tween = require 'tween'
```
You can of course specify your own easing function. Just make sure you respect the parameter format.

# Specs

This project uses [busted](http://olivinelabs.com/busted/) for its specs. In order to run them, install busted, and then execute it on the top folder:

```
busted
```

# Credits

The easing functions have been copied from EmmanuelOga's project in

https://github.com/emmanueloga/easing

See the LICENSE.txt file for details.

# Changelog

v2.0.0:

* the library no longer has "an internal list of tweens". Instead, `tween.new` returns an individual tween, which
  must be updated individually with `t:update(dt)`
* tweens can go forwards and backwards (trying to set the internal clock to a negative number raises an error)

