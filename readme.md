
# Game Maker to Godot dictionary
This document is for game maker devs like me that are moving their games or engine from GM:S to Godot. You can you your browser's search functionality to find GML functions and their equivalent in GDScript.

---

# Drawing functions
On game maker you can only use the drawing functions inside the draw event of an instance. On Godot you have to call the drawing functions inside a `func _draw()` of a [**CanvasItem**](http://docs.godotengine.org/en/3.0/classes/class_canvasitem.html).

## Making colors
GML
```gml
// These values are taken as being between 0 and 255
make_colour_rgb(red, green, blue);
```
And you also have the colors like `c_white`, `c_blue`, etc..

GDScript
```gdscript
# Constructs a color from an RGB profile using values between 0 and 1 (float)
Color(0.2, 1.0, .7)
# You can also set the color alpha by adding an aditional value
Color(0.2, 1.0, .7, 0.5)
```
You can also create a color from standardised color names with `ColorN`. See the full list [here](http://docs.godotengine.org/en/stable/classes/class_@gdscript.html#class-gdscript-colorn)


## Drawing a rectangle
GML
```gml
draw_rectangle(x1, y1, x2, y2, outline);
```

GDScript
```gdscript
draw_rect(Rect2, Color, bool filled=true)
```

> ## Example:
> To draw the same rectangle on both engines:
>
> GML
> ```gml
> draw_set_color(c_red)
> draw_rectangle(100, 120, 132, 152, false);
> ```
>
> GDScript
> ```gdscript
> draw_rect(Rect2(Vector2(100, 120), Vector2(32, 32)), ColorN("red"), true)
> ```

---

# Game functions
With this function you can end the game

GML
```gml
game_end();
```

GDScript
```gdscript
get_tree().quit()
```
---
## Sources
- [Game Maker docs](https://docs.yoyogames.com/)
- [Godot documentation](http://docs.godotengine.org/en/latest/index.html)

## Made possible by
A lot of poor souls that helped me answering questions on Discord, Reddit and Twitter.


[Emilio Coppola](https://github.com/coppolaemilio)
