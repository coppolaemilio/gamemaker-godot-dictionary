
# Game Maker to Godot dictionary
This document is for game maker devs like me that are moving their games or engine from GM:S to Godot. The first section gives a brief overview of the framework differences. The rest gives an API comparison for specific GML functions and their GDScript equivalent. You can use your browser's search functionality to find particular GML functions.

---

# Index

1. [Framework](#framework)
    1. [Objects](#objects)
    2. [Scenes](#scenes)
    3. [Inheritance](#inheritance)
2. [Events](#events)
3. [Scripts](#scripts)
4. [Globals](#globals)
5. [Drawing functions](#drawing-functions)
6. [Instance functions](#instance-functions)
7. [Strings](#strings)
8. [Random functions](#random-functions)
9. [Math functions](#math-functions)
10. [Game functions](#game-functions)
11. [Room functions](#room-functions)
12. [Window functions](#window-functions)
13. [Other functions](#other-functions)

---
# Framework

Below is a list of basic framework differences between the two engines. Understanding these will help you more quickly adapt to Godot's workflow.

## Objects

In Game Maker, you create a `room` filled with a list of `objects` that each may hold a `sprite` and a series of `events` which in turn trigger a series of `actions`.
Your scripted behavior exists in the `actions` portion. To reproduce this behavior multiple times between objects, you create a `script` that can be executed as a single `action`.

In Godot, you create a `scene` filled with a hierarchy of special `Objects` called `Nodes`. Rather than having you specify logic attached to a generic object though, Godot provides
a variety of customized objects for you. When you create a `Script`, you are in fact *extending* an existing `Object` type with custom features.

Objects in Godot can have any combination of...

- properties (like defining a variable during the Create Event)
- constants (same, but they cannot be changed)
- methods (like an action or series of actions)
    - Note that some methods are "notifications", triggered by the engine automatically. These are often preceded with an underscore '_'.
- signals (like custom events that other objects can react to)

Many of the code comparisons you see will illustrate a method, a.k.a. function, in place of an event's logic. Rather than having several scripts that each supply the logic for a single event's action(s), Godot has you define a single script per object with multiple functions defined for each "event" (perhaps even one for multiple).

## Scenes

Game Maker's rooms provide a flat list of the different types of assets that exist in the room. In order to have multiple sprites or physical objects come together to form the same object, you have to manually position them all during a step event by globally accessing them from within the room.

Godot's scenes are more small scale and self-contained. Your scenes can effectively be an object unto themselves as you add all of the necessary Sprites, PhysicsBody2Ds, and other Nodes to your scene's hierarchy to build up your object. That scene can then be `instanced` within a larger scene, similar to creating an object in a room. What's more, the nodes lower in the hierarchy will automatically move relative to their parent, so there is no need to manually position them.

## Inheritance

In Game Maker, there is a concept of having a "parent" object. Child objects then inherit properties and functions from the parent. However, in Godot there are two distinct concepts: Inheritance and Ownership.

When you construct a scene, you are creating a hierarchy of nodes with parent-child relationships. The node at the top, i.e. the "root of the scene", provides a single point of contact between this scene and others. However, the child nodes do not inherit its functionality. The parent merely "owns" each child node and may pass on information to it that it can use. 

This is why a child Node2D will move relative to its parent. The parent tells the child where the parent exists, and the Node2D object will take that into account when positioning itself.

To implement inheritance, you create a script and place it on a "base" object. You may then define custom features for this new type. These scripts will know the properties, constants, methods, and signals of the base object they are extending.

---
# Events
In Game Maker, when you need to code logic inside an object, you use Events. There are many kinds of events but the most used ones are: `create`, `step`, `draw`.

## Create Event
When you need to declare variables for an object in GMS you do it inside the **Create Event**. The equivalent in Godot is a function called `_ready()`. The main difference here is that on GMS, the **Create Event** declares variables that are accessible from everywhere. In Godot if you want your variables to be exposed to other functions or Nodes (objects) you need to declare them outside of the `_ready()` function at the top of the document.

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

var player_speed = 10

func _ready():
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

## Destroy event

Godot does not provide an equivalent notification function for the **Destroy Event** but it can be accessed through the actual notification callback.

GDScript **_notification(what)**
```gdscript
func _notification(what):
    match p_what:
        NOTIFICATION_PREDELETE:
            # execute logic before deleting the object.
```

Another option is to easily code up your own in GDScript. Instead of destroying that node with the usual `queue_free()` you create a function called `destroy()` and execute some code before self deleting.

GDScript **destroy()**
```gdscript
func destroy():
    # Here you write whatever you want to 
    # run before removing the node
    self.queue_free()
```

---

# Scripts

In game maker you can create scripts that you can call from any object in your project. In Godot, the so called "scripts" are called functions and you can declare all the custom `functions` that you want inside any node.

To compare between the two you can see this simple "script" that will add two numbers:

GML: Create new script called add_numbers
```gml
return argument0 + argument1;
```

GDScript: Inside a node's code
```gdscript
func add_numbers(argument0, argument1):
    return argument0 + argument1
```

In Godot instead of using the names `argumentX` it is recommended that you use a more descriptive name. Every argument named when you declare the function will create a variable that you can use from inside of it.

```gdscript
func add_numbers(number_one, number_two):
    return number_one + number_two
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

> ### Example:
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

## Drawing text
Drawing text is a bit more tricky in Godot. Make sure you declare the font resource outside of the `_draw()` function.

GML
```gml
draw_text(x, y, string);
```

GDScript
```gdscript
draw_string(font,Vector2(x,y),string,color,separation)
```

> ### Example:
> To draw the same rectangle on both engines:
>
> GML
> ```gml
> draw_set_font(fn_bitter);
> draw_set_font(make_color_rgb(0,0,0));
> draw_text(140, 100, "Hello world");
> ```
> 
> GDScript
> ```gdscript
> var font = load('res://fonts/Bitter.tres')
> func _draw():
> 	draw_string(Bitter,Vector2(140,100),"Hello world", Color(0,0,0,1),-1)
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
instance_destroy();
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

# Random functions

## Choose

GML
```gml
var value = choose(1, 2, 3);
```

In order to achieve something similar in Godot you have to first create an array with all the options and then get a random value from that array.

GDScript
```gdscript
var options = [1, 2, 3]
var value = options[randi() % options.size()])
```

---


# Math Functions

## Arc Sine
These functions take in a single float and return the arc sin

GML
```gml
arcsin();
```

GDScript
```gdscript
asin()
```

## Arc Cosine
These functions take in a single float and return the arc cos

GML
```gml
arccos();
```

GDScript
```gdscript
acos()
```

## Arc Tangent
These functions take in a single float and return the arc tan

GML
```gml
arctan();
```

GDScript
```gdscript
atan()
```

---

# Game functions

## Game end
With this function you can quit the game.

GML
```gml
game_end();
```

GDScript
```gdscript
get_tree().quit()
```

---
# Room functions

## Change room
GML
```gml
room_goto(room_name)
```

GDScript
```gdscript
get_tree().change_scene("res://nameofthescene.tscn")
```

---

# Window functions

## Set caption
GML
```gml
window_set_caption(string);
```

GDScript
```gdscript
OS.set_window_title(string)
```

---

# Other functions

## Open website on browser
This will open the specified URL on the browser.

GML
```gml
url_open( 'http://yoyogames.com' );
```

GDScript
```gdscript
OS.shell_open('http://godotengine.org/')
```

## Distance to object
This is a similar way of calculating the distance from an instance to another. On gdscript all the variables have to be Vector2. Also, be mindful that not all nodes have the `position` variable.

```gml
distance_to_object(obj_Player)
```

```gdscript
position.distance_to(Player.position)
```


---

## Sources
- [Game Maker docs](https://docs.yoyogames.com/)
- [Godot documentation](http://docs.godotengine.org/en/latest/index.html)


## < Written by [Emilio Coppola](https://github.com/coppolaemilio) > 
### [Additional contributors](https://github.com/coppolaemilio/gamemaker-godot-dictionary/graphs/contributors)
Special thanks to a lot of poor souls that helped me answering questions on Discord, Reddit and Twitter.

