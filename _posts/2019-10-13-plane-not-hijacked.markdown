---
layout: post
title:  "Plane not hijacked"
---

I am not very fond of long flights. Sitting in a crowded plane for many hours can be tiring. Fortunately the entertainment system comes to the rescue. For me, the most interesting feature of it is the moving map displaying current location of the plane - it beats even the best newest movies.

A few months ago I have been flying from Europe to the US. We have started from Germany and I have been indulging in my best pleasure during the flight – watching the moving map. This particular system presented the current location of the plane on the map – as most these systems do, but it also had one unique feature – it also displayed a panel with more “technical” information: current speed of the plane, azimuth, altitude and most importantly geographic coordinates of the current position.

I am a map geek, I work for a company very much involved in GIS systems, I am a geocacher – so looking at the geographic coordinates is something I do quite frequently. I know by heart that Warsaw – my home city is roughly at 52 degrees north and 21 degrees east.

So I stared at the coordinates on the screen in front of me and contemplated their slow change. The plane started heading north-northwest, so the longitude decreased slowly and the latitude increased at a slightly higher rate. So far so good.

The crew served the meal, I had to devote my whole attention to eating and not spilling anything, so I have temporarily stopped watching the changing coordinates. In the meantime we have crossed the prime meridian.

After the meal I have returned to my pleasure. At that time we have been above north-western part of the UK – something around 55 degrees north and 7 degrees west. The direction of the plane has changed to be closer to west-northwest – so the latitude increased much more slowly than when we started and the longitude…

At this moment I have noticed something very weird. I have expected the longitude to increase steadily. We have been already on the west hemisphere, we have been heading westward, so the longitude should increase with time. But it did not! I could clearly see that the longitude decreased: 8°45’W …  8°44’W …  8°43’W …  8°42’W

This was totally puzzling. I have been checking a few times what my geographic knowledge says about this situation and the answer was always the same. This meant that instead of going northwest in the direction of the US, we have been heading northeast in the direction of… Svalbard???

OK – stay calm, has someone hijacked the plane? I have looked around, the crew was taking back the dishes after the meal without any sign of anxiety. I have not remembered any sudden turn of the plane, the sun was still more or less on the correct side of the plane. I have looked at the coordinates once again: 8°18’W …  8°17’W …  8°16’W …  8°15’W. Clearly something was wrong – but what? I had no idea. Was the GPS system of the plane – the most likely source of this information – so seriously broken?

I kept looking at the coordinates:  8°02’W …  8°01’W …  8°00’W …  9°59’W

Once I saw that after 8°00’W instead of 7°59’W came 9°59’W, I had a moment of enlightenment. I immediately knew what was happening – this was just a small error in the code that displayed the coordinates. I could even imagine how this code might look like. The plane was not hijacked – what a relief! 

Obviously I have never seen the actual code displaying the coordinates in the entertainment system of a plane. But here is my attempt of “reverse engineering” such code.

Most likely the geographic coordinates are retrieved from a GPS system of the plane as a pair of two floating numbers – one for latitude and one for longitude. Since I have been crossing the prime meridian, but not the equator we will discuss only the longitude. A common convention is that positive values of the longitude mean coordinates east of the prime meridian and negative values mean coordinates west of the prime meridian. The float value represents degrees with some fraction that is usually converted into arc minutes (each arc minute is 1/60 of degree) and arc seconds (1/60 of arc minute).

How could one implement a function to display longitude in a human readable form? For simplicity lets leave out arc seconds and display only degrees and arc minutes.

A very simple implementation in Python might look like this.

```python
import math

def display_coord(coord):
    degrees = math.floor(coord)
    minutes = math.floor((coord - degrees) * 60)
    if degrees < 0:
        degrees = -degrees
        suffix = "W"
    else:
        suffix = "E"
    print("%d\u00b0 %02d' %s" % (degrees, minutes, suffix))
```

For coordinates on the eastern hemisphere this implementation is fine. This is how it would work at the beginning of my flight.

    display_coord(3.04)       3° 02' E
    display_coord(3.02)       3° 01' E
    display_coord(3.00)       3° 00' E
    display_coord(2.98)       2° 58' E
    display_coord(2.96)       2° 57' E

However when the coordinate values become negative – for the western hemisphere – this code starts to exhibit exactly the same behaviour as I have observed during my flight.

    display_coord(-7.94)      8° 03' W
    display_coord(-7.96)      8° 02' W
    display_coord(-7.98)      8° 01' W
    display_coord(-8.00)      8° 00' W
    display_coord(-8.02)      9° 58' W

Fixing this issue is trivial – simply check if the coordinate is negative at the beginning of the function.

```python
def display_coord(coord):
    if coord < 0:
        coord = -coord
        suffix = "W"
    else:
        suffix = "E"
    degrees = math.floor(coord)
    minutes = math.floor((coord - degrees) * 60)
    print("%d\u00b0 %02d' %s" % (degrees, minutes, suffix))
```

In this form the function behaves as expected also for negative coordinates.

    display_coord(-7.94)      7° 56' W
    display_coord(-7.96)      7° 57' W
    display_coord(-7.98)      7° 58' W
    display_coord(-8.00)      8° 00' W
    display_coord(-8.02)      8° 01' W

Is there a moral in this story? Probably not. The error is quite insignificant – for a fast moving plane a rough approximation of its coordinates rounded to whole degrees is sufficient to assess current position. So it is actually quite likely that nobody else ever noticed this error – at first glance everything looks OK. It required a real coordinate freak to spot it.

And now what a good story to tell it is!
