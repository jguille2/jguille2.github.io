# Snap! colors
Reimplementing color system based in hsla, fixing some current problems and trying to make a more consistent behavior.

## Morphic.js - Color object

### Color object
Color object has 4 main properties, based in HSLA.

  - **hue** Related to H (hsla). Values into **[0,100)** interval. It maps original 0-360 degrees polar coordinates and wrapping like a roller.
    Wrapping 100 -> 0, 101 -> 1, -1 -> 99...

  - **shade** Related to L - Lightness(hsla). Values into **[0,400)** interval.It maps %L into a 0-200 scale, with a first wrapping (200 to 400) to show continuity. Beyond this (values > 400 and values < 0) wrapping like a roller.  
    Then 0 -> black, 100 -> full color, 200 -> white, 300 -> full color, 400 -> black.  
    And wrapping: 400 -> 0, 500 -> 100..., -1 -> 399, -100 -> 300, -400 -> 0 ...
    
  - **saturation** Related to S (hsla). Values are a **% [0,100]** interval. No wrapping: (values > 100) -> 100 and (values < 0) -> 0.

  - **opacity** Related to A - Alpha (hsla). Values are a **% [0,100]** interval. No wrapping: (values > 100) -> 100 and (values < 0) -> 0.

### Constructor
Constructor has three modes with a generic  
**Color (mode, param1, param2, param2, param3, param4)**

  - **Snap!**: _mode = 's'_  
    **Color('s', hue, saturation, shade, opacity)** Object construction from Snap properties (described above).  
    Building Colors with their own properties.  
    Default values are _hue_ = 0, _saturation_ = _shade_ = _opacity_ = 100

  - **RGBA**: _mode = number_  
    **Color(r, g, b, a)** Initial method. Construct object from rgba params. Fully used in Snap! code  
    Red - _r_, Green - _g_ and Blue - _b_ are 0-255 values and Alpha - _a_ is 0-1.  
    Default values are _r_ = _g_ = _b_ = 0 and _a_ = 1

  - **HSLA**: _mode = 'l'_  
    **Color('l', h, s, l, a)** New method. Construct object from hsla params.  
    Hue - _h_ is 0-360 (degrees), Saturation - _s_ and Lighness - _l_ are % values and Alpha - _a_ is 0-1.  
    Default values are _h_ = 0, _s_ = 100, _l_ = 50 and _a_ = 1

### Setting values
To ensure wrappings and ranges, we use these functions to assign properties values

  - **setHue**, **setSaturation**, **setShade** and **setOpacity**
  - **this.param** reports directly these 'natural' params... but often, we need to transform _shade_ (and its wrapping) into _lightness_ -> **reportLightness**
  - Fixed **precision pararams** to **0.01**: _hue_ [0.00, 99.99], _saturation_ [0.00, 100.00], _shade_ [0.00, 399.99] and _opacity_ [0.00,100.00]. With this precision, RGB calculations are coherents (reversible without changes) as integers [0, 255]

### Color functions
  - **copy** and **eq** (comparison) are the same, now in hsl
  - **toString** reports 'hsla(h, s%, l%, a)' Used in all Snap to paint with colors. It is a valid CSS color value (as past _rgba()_)
  - **mixed** is now in hsl (and it calculates the ponderant mixture with cylindrical coord)
  - **lighter** and *darker* are not using _mixed_ because it don't run fine with white and black (they are circles, not points). Now _lightness_ is used.
  - **dansDarker** is kept for backward comp (but not used in Snap code)

### New pen color blocks
  - **Two new action blocks**: _set_/_change_ pen color params: _hue_, _shade_, _saturation_ and _opacity_ (deliberately different order than HSLA, more appropriated for snappers/scratchers).
  - And **one new reporter** "pen color param↓" for these 4 color params.
  - Four old blocks deleted. Added to _blockMigrations_ for backward compatibility
  - Old functions kept for compatibility for JS coders

### RGB and HSV functions. Backward compatibility with color library and with JS coders
  - **set_hsv** and **hsv** migrated to new system
  - New **rgb** function to get RGB code from current based on HSL
  - _Getters_ **r**, **g** and **b** to call _rgb_ function as color properties (for bc compatibility)
  - Also RGB _setters_. This is not used neither into Snap code nor into libraries, but it was in the JS core... and it can be used for JS coders (and maybe useful for new libs).

### Color Palette
  - Added a gray scale and a palette of 20 basic colors to the _picker input_ slot
  - Added the same 20 basic colors to the palette into the _Paint Editor_
  - These colors are:
```javascript
    colors = ['rgb(0, 0,0)',  //black
        'rgb(128, 128, 128)', //gray
        'rgb(192, 192, 192)', //silver
        'rgb(255, 255, 255)', //white
        'rgb(139, 69, 19)',   //saddlebrown
        'rgb(128, 0, 0)',     //maroon
        'rgb(255, 0, 0)',     //red
        'rgb(255, 192, 203)', //pink
        'rgb(255, 165, 0)',   //orange
        'rgb(210, 105, 30)',  //chocolate
        'rgb(255, 255, 0)',   //yellow
        'rgb(128, 128, 0)',   //olive
        'rgb(0, 255, 0)',     //lime
        'rgb(0, 128, 0)',     //green
        'rgb(0, 255, 255)',   //aqua
        'rgb(0, 128, 128)',   //teal
        'rgb(0, 0, 255)',     //blue
        'rgb(0, 0, 128)',     //navy
        'rgb(128, 0, 128)',   //purple
        'rgb(255, 0, 255)'    //magenta
    ];
```
    
### RGB, HSV library
  - Added two blocks for HSLA codification (setter and getter)
  - Renamed library: "RGB, HSV and HSLA pen colors"
  
### Other minor changes: bug fixes, details...
  - Bug in _morphic.js_, _Morph.prototype.getPixelColor_ must scale _opacity_ input. Here _getImageData_ function returns _opacity_ in a 0-255 scale, and _RGB_ (old method) is waiting a 0-1 scale.
  - Changed default ColorSlotMorph color (from Color(145, 26, 68) to _red_)
  
### Not implemented - Notes for continue...
  - Block "switch to pen color %n" to set a color from the palette with its number (order) and name (by the drop-down)
  - A new "Color" Tab.
    - A "Current Color" object (no erasable) to store current (any) color. It's number (order) is 0
    - A set of colors: objects with name, order and a defined value (their color)
    - We can delete, redefine (change the color), rename and reorder (move) these.
    - We have a default palette (a set of colors) in every sprite
    - We can import/export palettes: trafic-lights, rainbow, 16 basic colors, grays...
    - Tree basic colors to manage this: "switch pen color to", "next pen color" and "pen color #" reporter
  - SVG support - Vector Editor. Possibility to paint with "pen color" (dinamically changed)
  - Check default (80, 80, 80) color for the initial Sprite
