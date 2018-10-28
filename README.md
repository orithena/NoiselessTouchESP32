# NoiselessTouchESP32

Noiseless Touch library for ESP32 (Arduino Core) to get reliable touch events.

## Description

The `touchRead(pin)` function (provided by the ESP32 Arduino Core) measures 
the capacitive input on a touch pin of the ESP32. It returns a value of 
~64 (plusminus 10, depending on conditions) when that pin is open, and a 
value of ~15 (plusminus 10) when that pin is touched by a human.
Variations of the measurements depend on external factors in the surrounding
of the pin and the cable attached to that pin. During usage, it should not 
fluctuate by more than plusminus 1 (if it does, you've got very high 
capacitive fluctuations in the environment).
Depending on your interrupts etc, errorneous measurements may come up.

This library attempts to smooth over errorneous measurements and provide 
non-fluctuating measurement values. To do this, it keeps a history of past 
measurements and calculating a mean value out of that history while ignoring
extreme outliers in the history. The resulting mean value is compared to the
last output: If the result is within the bounds of the old value plusminus
the hysteresis value, then the old value is returned. If the result is
outside the hysteresis range, the new value is returned.

Note that the functions in this library may need multiple runs before they 
react to a touch. So make sure that you call them once every 10-50 microseconds
or more often.

### Install

#### Manual Install

Windows: Download the .zip and extract it in Documents\Arduino\Libraries\\\{Place "NoiselessTouchESP32" folder here}

Linux: Download the .zip and extract it in ~/Sketchbook/Libraries/{Place "NoiselessTouchESP32" folder here}

MacOS: Download the .zip and extract it in ~/Documents/Arduino/libraries/{Place "NoiselessTouchESP32" folder here}

Via Git command line:
```
cd <Arduino IDE library folder>
git clone https://github.com/orithena/NoiselessTouchESP32.git
```

#### Manually through IDE

Download the .zip, go to Sketch > Include Library > Add .zip Library > Select the downloaded .zip file.

### Include

On pin `T0`/GPIO4, with default history length (6) and hysteresis (3):
```
#include <NoiselessTouchESP32.h>
NoiselessTouchESP32 touchbutton(T0);
```
or with custom history length (1..16) and hysteresis (1..63):
```
#include <NoiselessTouchESP32.h>
NoiselessTouchESP32 touchbutton(T0, 8, 20);
```

The history length determines the smoothness of the resulting reading,
but also the reaction speed. History lengths below 3 remove the ability
to ignore single extreme outliers. Generally, a reaction is expected after
history_length/2 measurement runs.

*  Extremely low history length (1..2) = No error correction = Immediate reaction.
*  Low history length (3..7) = Possibly erratic measurements = Fast reaction.
*  High history length (8..16) = Smoother measurements = Slow reaction.

Hysteresis determines the amount of change that has to happen before the
output changes. Since the noise on the pin is at plusminus 1 when I tested
it, a hysteresis of 2 would suffice. The touch sensor then might react at 
2 cm distance already (depending on the size of the electrode you use and
how much of your hand you hold over the electrode). If you want the library
functions to react only when the electrode really is touched, use a hysteresis
of 16 or more.

*  Low hysteresis (1..2) = Reaction at 2cm or even more distance.
*  Medium hysteresis (3..15) = Reaction at 0..2cm distance before touching the electrode.
*  High hysteresis (16..32) = Reaction on touch.
*  Extremely high hysteresis (33..63) = May not react.

### Read functions

All functions in this section read the current touch pin value and add it to
the history.

```
int distance = touchbutton.read_raw_mean();
```
Returns the raw value after ignoring outliers and calculating the mean value.
Generally returns something between 0 and 90 (with the normal open value in 
my setup being between 50 and 70). Basically, this is reading the touch button
with a hysteresis value of 0, but the hysteresis value still has meaning
here for detecting outliers.

```
int distance = touchbutton.read_with_hysteresis();
```
Returns the raw value after ignoring outliers, calculating the mean value and
clamping the value in the current hysteresis range. Generally returns something
between 0 and 90 (with the normal open value in my setup being between 50 and 70).
This is the function to use when you need steady measurement values.

```
int direction = touchbutton.changed();
```
Returns 1 if the users hand got nearer to the touch pin, 0 if distance stayed
the same as to last call, -1 if the hand moved farther away. This function
needs low hysteresis to distinguish distances. Depending on the users hand speed,
this function may return a different amount of -1 and +1!

```
bool touch_occured = touchbutton.touched();
```
Returns true if the pin has been touched, false if not. This is a one-shot 
event, i.e. this function returns true once and the user has to take the
hand away and touch again before this function returns true again.

```
bool is_touching = touchbutton.touching();
```
Returns true if the user is currently touching, false if not. This is a
repetitive event, i.e. this function returns true repeatedly if the user
continues to touch.

### Additional functions

These functions do not read a new value and do not add to the history.

```
int value = touchbutton.last_value();
```
Same as `read_with_hysteresis()`, but without reading a new value from the 
touch pin. This is useful if you use `changed()`, `touching()` or `touched()`
to branch in your program, but still need the touch value from that event 
afterwards.

```
int mean_touch_value_with_old_data = touchbutton.value_from_history();
```
Same as `read_raw_mean()`, but without reading a new value from the touch pin.
I assume that you will need it only in very special circumstances, which I
cannot foresee right now.
