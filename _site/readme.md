
# Game Maker to Godot dictionary
This document is for game maker devs like me that are moving their games or engine from GM:S to Godot. You can you your browser's search functionality to find GML functions and their equivalent in GDScript.

---

# Index

1. [Events](#events)
2. [Globals](#globals)
3. [Drawing functions](#drawing-functions)
4. [Instance functions](#instance-functions)
5. [Strings](#strings)
6. [Game functions](#game-functions)

---

# Events
On Game Maker when you need to code logic inside an object you use Events. There are many kinds of events but the most used ones are: `create`, `step`, `draw`.

## Create Event
When you need to declare variables for an object in GMS you do it inside the **Create Event**. The equivalent in Godot is a function called `_ready()`. The main difference here is that on GMS, the **Create Event** declares variables that are accesible from everywhere. In Godot if you want your variables to be exposed to other functions or Nodes (objects) you need to declare them outside of the `_ready()` function at the top of the document.

Imagine that we want to set the variable `player_speed` to `10` but if there are monsters present, you want it to be `5`.
In Game Maker you can code all this inside the **Create Event**:

GML **Create Event**
```gml
player_speed = 10;
monsters = instance_number(obj_monster);

if (monsters) {
    player_speed = 5;
}
```

In Godot you will have to code it like this:

GDScript **_ready()**
```gdscript
extends Node

func _ready():
    var player_speed = 10
    var monsters = get_tree().get_nodes_in_group("MONSTERS").size();
    if monsters:
        player_speed = 5
```

## Step Event

Simple **Step Event** function for moving an object.

GML **Step Event**
```gml
x += player_speed;
```

GDScript **_process()**
```gdscript
func _process(delta):
    position.x += player_speed
```

## Draw Event

Simple **Step Event** function for moving an object.

GML **Draw Event**
```gml
draw_rectangle(100, 120, 132, 152, false);
```

GDScript **_draw()**
```gdscript
func _draw():
    draw_rect(Rect2, Color, bool filled=true)
```

---

# Globals

In Game Maker you can declare globals very easy by just adding `global.` at the start of a variable. In Godot you can create a similar kind of variables via the [Singletons (AutoLoad)](http://docs.godotengine.org/en/3.0/getting_started/step_by_step/singletons_autoload.html) feature.

I recommend you to read the entry of the Godot documentation but to get a quick equivalent you can do the following:

1. First of all, create a `global.gd` script.

2. Then, Select Project > Project Settings from the menu, switch to the AutoLoad tab

3. Add a new entry with name “global” that points to this file:

![Autoload menu screenshot](http://docs.godotengine.org/en/3.0/_images/addglobal.png)

Now, whenever you run any of your scenes, the script is always loaded. The variables declared inside `global.gd` can be accesed or modified the same way you would do in GML: `global.variable_name`.

The cool thing about godot is that you can also declare functions inside the `global.gd` file that you can use from any other instance inside your game.

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
You can also create a color from standardised color names with `ColorN`. See the full list [here](http://docs.godotengine.org/en/stable/classes/class_@gdscript.html#class-gdscript-colorn).


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
> draw_set_color(c_red);
> draw_rectangle(100, 120, 132, 152, false);
> ```
>
> GDScript
> ```gdscript
> draw_rect(Rect2(Vector2(100, 120), Vector2(32, 32)), ColorN("red"), true)
> ```


## Visibility
GML
```gml
visible = true;
visible = false; 
```

GDScript
```gdscript
self.show()
self.hide()
```

---

# Instance functions

## Instance destroy

GML
```gml
instance_destroy()
```

GDScript
```gdscript
self.queue_free()
```


---

# Strings

String functions are a little bit different because in game maker everything is a function but in Godot they are methods. You can also treat Strings in godot like arrays.

## String Length

GML
```gml
string_length(string);
```

GDScript
```gdscript
string.length
```

## String Char At

GML
```gml
string_char_at(string, index);
```

GDScript
```gdscript
string[index]
```

## String Upper/Lower

GML
```gml
string_upper(string);
string_lower(string);
```

GDScript
```gdscript
string.to_upper()
string.to_lower()
```

## String Delete

GML
```gml
string_delete(string, index, count);
```

GDScript
```gdscript
string.erase(index, count)
```




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


## < Written by [Emilio Coppola](https://github.com/coppolaemilio) > 

Special thanks to a lot of poor souls that helped me answering questions on Discord, Reddit and Twitter.