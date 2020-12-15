
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
14. [Data Structures](#data-structures)

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

Simple **Draw Event** function for drawing a rectangle.

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
> draw_rect(Rect2(Vector2(100, 120), Vector2(32, 32)), Color("red"), true)
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
show()
hide()
visible = true
visible = false
set_visibility(true)
set_visibility(false)
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
queue_free() # for Nodes, waits until next frame to delete all queued Nodes at once
free() # for all Objects, deletes immediately
```

Note that deleting a Node will also delete all of its attached children automatically.

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
string.length()
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

# Data Structures

## Stack, Queue, List, Serialization

Godot's Array doubles as the stack and queue data structure. Unfortunately, Godot has no Lists for the scripting API (although the engine itself has them).

GML **global Stack functions**
```gml
stack = ds_stack_create();

// ...do various insertions

if ds_stack_empty(stack) {
    // is empty
}
else if ds_stack_size(stack) == 3 {
    // there are 3 items on the stack
    stack2 = ds_stack_create();
    ds_stack_copy(stack2, stack);
    top = ds_stack_top(stack2); /top is the item at index 2
    ds_stack_pop(stack2); //top's value is no longer in stack2
    ds_stack_push(stack2, 5); //top's previous index now has value 5
    serialized_data = ds_stack_write(stack2); //converts stack2 into a data string
    stack3 = ds_stack_read(serialized_data); //deserializes the data string into another stack object.
}
ds_stack_clear(stack);
ds_stack_destroy(stack);
```

GDScript **Array type** as "stack"
```gdscript
var arr = []
if arr.empty(): # if not arr: also works
    pass
else if arr.size(): # else if arr: also works
    var arr2 = arr.duplicate()
    var top = arr2.back()
    arr2.pop_back()
    arr2.push_back(5)
    
    # a stringified version of the array, but not actually serialized data
    var arr_str = str(arr2)

    # all serialized data is stored on Resource objects which automatically serializes all properties of any kind.
    var res_script = GDScript.new()
    res_script.source_code = "extends Resource\n"
    res_script.source_code += "var array"
    res_script.reload()
    # ^ you could also just create an actual script and load it, e.g.
    res_script = load("res://res_script.gd")

    var res = res_script.new()
    res.array = arr2
    ResourceSaver.save("res://my_array.tres", res) # serializes all properties on res for you
    var res2 = load("res://my_array.tres")

arr.clear()
# because Arrays are allocated with a reference counter, they automatically delete themselves when no references to them exist anymore.
```

In addition to `push_back()` and `pop_back()` to simulate a stack, Godot's arrays also have `push_front()` and `pop_front()`. The combination of `push_front()` and `pop_back()` gives you a queue's features.

## Maps

Godot's Dictionary object is the equivalent of GML's DS Map. They can be inlined into GDScript code just like Arrays by using curly brackets.

Unlike DS Maps however, Arrays, Dictionaries, and other data (even custom Objects!) can be made into a key *or* value in a Dictionary (or a value in an Array). This is contrary to what the Yoyo Games docs have to say about DS Maps:

> NOTE: While these functions permit you to add lists and maps within a map, they are useless for anything other than JSON, and nested maps and lists will not be read correctly if written to disk or accessed in any other way.

GML **global Map functions**
```gml
map = ds_map_create();
ds_map_add(map, "level", 100);
ds_map_add(map, 5.2, "hello");
map[? "super"] = "awesome";
ds_map_add(map, "super", "goodbye"); // fails because key "super" already exists
if ds_map_exists("super") {
    // "super" is a key
    ds_map_replace(map, "super", "goodbye"); //succeeds because this is a "replace" operation
    ds_map_delete(map, "super"); // now key "super" is gone
}

if ds_map_empty(map) {
    // it is empty
}
else if ds_map_size(map) == 2 {
    // has 3 key-value pairs
}

// iteration
var size, key, i;
size = ds_map_size(inventory);
key = ds_map_find_first(inventory);
for (i = 0; i < size; i++;) {
    if key != "level" {
        key = ds_map_find_next(inventory, key)
    }
    else {
        break;
    }
}

map2 = ds_map_copy(map); // map2 is now a copy of map3

serialized_data = ds_map_write(map); //converts map into a data string
map3 = ds_map_read(serialized_data); //deserializes the data string into another map object.

ds_map_clear(map); // it is now empty
ds_map_destroy(map); // memory is freed
```

GDScript **Dictionary type**
```gdscript
var dict = {"hello": "world"} # initialization, just like an array, could be empty with '{}'
dict["level"] = 100
dict[self] = {
    3.4: "PI",
    [1, 2, 3]: GDScript.new(),
}

if dict.has(self):
    dict[self] = "testing" # just deleted all of those objects because their references disappeared by overwriting the value
    dict.erase(self) # the key-value pair is now gone.

if dict.empty():
    pass # it is empty
if not dict:
    pass # same
if dict.size(): # if dict: also works
    pass # the Dictionary is non-empty
if dict.size() == 2:
    pass # there are 2 key-value pairs that exist

# iteration
for a_key in dict:
    print(dict[a_key])

# Iterate over first 3 key-value pairs.
# Safely exit early if it isn't that large
var i = 0
for a_key in dict:
    var value = dict[a_key]
    i += 1
    if i == 3:
        break

# string keys can be directly accessed as if properties on an object
var value = dict.level

# copy the Dictionary
var dict2 = dict.duplicate()

# serialization is the same
var res_script = load("res://res_script.gd")
var res = res_script.new() # res_script.gd needs to define a property, in this case called 'foo'
res.foo = dict2
ResourceSaver.save("res://res.tres", res)
var res2 = load("res://res.tres")
# these print the same thing
print(dict2)
print(res2.foo)

dict.clear()
# going out of scope means that the local variables 'dict', 'dict2', 'res_script', 'res', etc. disappear and all references to them end.
# Because they either an Array, a Dictionary, or a Reference object, they are freed automatically when all references to them vanish.
```

For JSON manipulation, you can also freely translate a combination of Arrays, Dictionaries, and other data from GDScript into a JSON string and back by using the JSON global.

JSON **parse(string)**
```gdscript
var p = JSON.parse('{"hello": "world"}')
if p and p.error == OK:
    if typeof(p.result) == TYPE_DICTIONARY:
        print(p.result["hello"]) # prints "world"
    if typeof(p>result) == TYPE_ARRAY:
        pass # could be possible, with a different JSON string
else:
    print("parse error")
```

## Priority Queue

Godot does not provide its own Priority Queue object. Instead, to accomplish its functionality, you must either implement your own special Node class that handles its own sub-hierarchy of nodes, or you must use an Array in which you sort it after every mutable operation. Array comes with a built-in `sort` function for simple types and the option to specify your own sorting algorithm with a `sort_custom` method.

Array **sort_custom(script, static_function_name)**
```gdscript
class MyCustomSorter:
    static func sort(a, b):
        if a[0] < b[0]:
            return true
        return false

var my_items = [[5, "Potato"], [9, "Rice"], [4, "Tomato"]]
my_items.sort_custom(MyCustomSorter, "sort")
```
The "min" and "max" are then always located at `front()` and `back()`, respectively.

## Grid

Godot does not provide its own grid class (the TileMap and GridMap nodes are purely for visualization purposes). To create a 2D array, you need to create an array of arrays:

GDScript **Array of Arrays**
```gdscript
var arr = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
]
print(arr[1][2]) # prints 5
```

There are no special utilities methods to assist in 2D array operations. You will have to code those manually, perhaps with a script extending `Reference`.

```gdscript
# grid.gd
extends Reference
var data = []

func _init(width = 0, height = 0):
    resize(width, height)

func get_width():
    var max = 0
    for arr in data:
        if arr.size() > max:
            max = arr.size()
    return max

func get_height():
    return data.size()

func resize(width, height):
    data.resize(height)
    for arr in data:
        arr.resize(width)

# ...etc.
# node.gd
extends Node
const Grid = preload("res://grid.gd")
func _ready():
    var grid = Grid.new(3, 5) # initializes Grid with width=3, height=5
    # end of scope, local variable 'grid' exits, no references to the Grid object. It inherits Reference, therefore it is freed.
```

---

## Sources
- [Game Maker docs](https://docs.yoyogames.com/)
- [Godot documentation](http://docs.godotengine.org/en/latest/index.html)


### Written by [Emilio Coppola](https://github.com/coppolaemilio) and [Additional contributors](https://github.com/coppolaemilio/gamemaker-godot-dictionary/graphs/contributors)
Special thanks to a lot of poor souls that helped me answering questions on Discord, Reddit and Twitter.
