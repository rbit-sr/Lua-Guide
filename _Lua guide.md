# Lua guide

This document provides a complete overview of every feature the Velo Lua integration provides. We will start at the basics and work our way up towards more complex features. While this guide does not expect you to know the language Lua itself, you are required to at least know basic programming concepts like variables, functions, control flow statements (`if else`, `while`, `for`), arrays and tables/dictionaries.

## What is the Velo Lua integration?

With update 2.4.0, Velo now supports integration of Lua scripts. *Lua* is a powerful scripting language that is easily embeddable into different systems (like Velo). With its simple procedural syntax, it allows for quick and easy script writing with little code.

These scripts allow you to automate any process that you could otherwise do manually via Velo commands. This includes querying and modifying values of the game's internal state (position, velocity, ...), changing Velo settings, saving and loading savestates, spawning and despawning actors, placing tiles, automating inputs, drawing shapes and text to screen and more. 

Note that these scripts cannot be used for cheating and trying to call any command that needs to modify the game's current state will throw an error when in online multiplayer.

If you want to know more about Velo and how to install, please read the contained `_README.txt` file. For a deeper guide of the Lua language itself, refer to the [official guide](https://www.lua.org/pil/1.html).

## Velo's command system

Update 2.3.0 brought a new command console with lots of useful commands to run. These commands are integral to our Lua scripts as they provide an interface for our scripts to communicate with Velo and the game. Any command you can run in the Velo console can also be called from within our Lua scripts.

You can open up the console by pressing CTRL+Z. In order to change this hotkey, press F1 and change it under "Console" -> "enabled". Having opened up the console, you can now start typing commands by pressing ENTER and execute them by pressing ENTER again. Type `help` to get a list of commands and `helpAll` to get a complete list of all commands, which includes a lot of more niche commands only useful for Lua scripts. Command names are not case-sensitive, so `helpall` would work, too.

As a simple example, open up any map and enter `get Player.actor.position.x`. This will return the player's current x-position. You can then change this value and teleport the player around by typing `set Player.actor.position.x 4000` for example.

Command parameters are provided as a space-separated list. In case a string value containing spaces needs to be passed, it must be enclosed by double quotes `"`. (This `Player.actor.position.x` parameter is a string parameter actually, but it contains no spaces so it doesn't require any quotes.)

What Lua scripts now allow you to do is to add new custom commands that can be run like any other command. Running such a custom command will then execute the script's code.

## Getting started and Hello World

Let's write a simple Hello World script. Open up the folder "Velo\\scripts" where all scripts need to be located. As you can see, it already contains a couple of demo scripts. Create a new text-file and name it "helloWorld.lua". (It **must** use the ".lua"-extention, it cannot be ".txt". Enable file extentions in your file explorer to see or just copy another script and rename it.)

Open up your newly created file with any text-editor, type `echo("Hello World!")` and save. Then, open up your game, open up the console, type `helloWorld` and it should print to your console. Note that you do not need to restart your game when making changes. Just save the script and run it again.

The file name of your script is essential as it is what determines the command to type, i.e. a script named "x.lua" will be run by typing `x` (without the extension).

Next, note that `echo` is just another console command and we could just as well have entered `echo "Hello World!"` into the console. Every console command is available as a pre-defined Lua function in fact. The only difference now is the syntax; as for the console, parameters are passed as a space-separated list, while in Lua, commands need to be called like functions, enclosing the parameters as a comma-separated list into `()`. Note further that commands in Lua are now case-sensitive and you need to always enclose strings with double quotes `"` or single quotes `'` (both work in Lua). Here is another example to illustrate the difference:

- Console: `set Player.actor.position.x 4000`
- Lua: `set("Player.actor.position.x", 4000)`

You can see the Lua function signature of a command by typing `help [command]`. If you type `help echo` for example, you should see it being `void(string)`.

Lastly, please note that these scripts are blocking the game's execution. If a script takes 1 second to execute, then it will also freeze the game for 1 second. This is not a problem however and we will soon see how to utilize callbacks to prevent any problems.

## Passing parameters

As you might have noticed, most commands take some number of parameters while our `helloWorld` script took none as it didn't require any. Let's expand our script a little by adding an extra paramater to determine how many times `Hello World!` should be printed.

If you just type `helloWorld 3`, then it actually does pass an extra `3` parameter to our script. It just doesn't read or make use of it. All parameters are stored in an `arg` array, which you can access with the usual array-access syntax `arg[1]`, `arg[2]`, ... . Note that Lua arrays use 1-based indexing. In order to pass parameters to a script, just put them after the script name as a space-separated list. Note that even though we are in the console, the parameters we pass use Lua syntax, meaning that in order to pass a string, you have to enclose it with quotes.

We can make Lua execute a certain chunk of code multiple times by enclosing it in a `for i = first, last do ... end` construct. In our example, it looks the following:
```lua
-- helloWorldRepeat.lua    <- this is a comment
for i = 1, arg[1] do
    echo(tostring(i) .. ": Hello World!")
end
```
This creates a new index `i` and makes it count up from `1` to `arg[1]` while executing the inner code on each iteration. `tostring` allows you to convert numbers and other types to their string representation. You can concatenate strings via `..`. Note that in Lua, we don't use parentheses to enclose blocks but instead use the `end` keyword to signal when a block ends.

## Basic data types

Lua supports the following basic data types:
- `number` (integers and reals)
- `boolean` (`true` and `false`)
- `string` (enclosed by `"` or `'`)
- `nil` (containing no value)
- `function`

When declaring a new variable, you don't specify the data type and instead just write it like so:

```lua
x = 20
y = x + 43.5
y = "Hello!"
```

Note that variables can also mutate their type and you can assign a string to a variable that was a number before just fine.

## `get` and `set`

Now let's go over two of the most important commands to know about. `get` and `set` allow you to query different values from the game's memory and modify them. Their basic usage is the following:

- `get [field]`
- `set [field] [value]`

The field determines the value to query or modify. 

### Specifying the target

In order to specify a field, you need to first specify the target object of which you wish to query or modify some value. We already saw `Player` as a possible target, but let's list a couple more possible targets:

- `Player`
- `Grapple`
- `Rocket`
- `Obstacle`
- `DroppedObstacle`
- `Fireball`
- `SwitchBlock` (gate)
- `CCamera` (not an actor, but can still be targetted)
- ...

You can get a complete list by typing `listTargets`. There may exist multiple actors of a specific type at once and so you can further specify its index by adding a `#` and the index (0-based). For example:

- `Player#0` (player 1)
- `Player#1` (player 2 or ghost)
- `Obstacle#19`

If you don't pass any index, it will use `0` by default. You can see how many of a specific type exist by using the command `count [type]`. To get a complete list of all currently existing actors sorted by how close they are to the player, type `locate [type]`.

### Field accessor path

Next, you need to add a dot-separated path of field accessors. This is similar to pretty much any programming language allowing for some form of (nested) structures or objects. In fact, SpeedRunners was written in an object-oriented programming language and with these field accessors, you are actually accessing the fields of each object as they are represented internally. 

Let's list a couple of example `Player` fields:

- `boost` (float, from `0.0` to `2.0`)
- `boostacoke` (float, from `0.0` to `10.0`)
- `canGrapple` (bool)
- `isClimbing` (bool)
- `isGrappling` (bool)
- `isInAir` (bool)
- `isOnGround` (bool)
- `isSliding` (bool)
- `isSwinging` (bool)
- `itemId` (byte, type `listItems` for a list of all IDs)
- `jumpState` (int, `0`: can ground jump, `1`: can air jump, `2`: cannot jump)
- `moveDirection` (`1` or `-1`, useful to figure out swing direction, flip value to initiate a moonwalk)
- `swingAngle` (float)
- `swingRadius` (float)

Each actor type has a unique list of fields. You can get a complete list by typing `listFields [type]`. Note that apart from primitive types like `float` or `bool`, fields can also be of any object type. The `Player`'s `sprite` field for example is of type `CAnimatedSpriteDrawComponent`, and the type `CAnimatedSpriteDrawComponent` contains a field `rotation`. Therefore, using `Player.sprite.rotation` you can access the sprite's `rotation` field.

Most importantly, every actor contains a field `actor` of type `CActor` which contains values common to every actor type. This is where you can access `position` and `velocity`, which are of type `Vector2`. A `Vector2` just contains two floats `x` and `y`. Examples:

- `get Player.actor.position.x`
- `set Rocket#3.actor.position.y 2000`
- `set DroppedObstacle#2.actor.velocity.x 800`

Many actors further contain references to other actors. `Player` for example contains references to every item object they own:

- `get Player.grapple.isConnected`
- `get Player.freezeRay.actor.isCollisionActive`
- `get Fireball#1.owner.actor.position.x`

### Array and `List` fields

Some fields may be of array or `List` type (functionally, there is no difference between these two). A `FreezeRay` for example is made up of several smaller sprites placed next to each other to form a long ray. It contains a field `sprites` or type `CAnimatedSpriteDrawComponent[]`. You can access individual elements with the usual array access syntax (0-based) by enclosing the index into square brackets `[]`:

- `get Player.freezeRay.sprites[3].position.x`
- `get TriggerSaw#1.hitbox.vertices[2].x` (hitbox is an octagonal shape of type `CConvexPolygon`)
- `get Player.droppedObstacles[4].isBroken`

You can get the number of elements of an array or `List` using the `count` field:

- `get Player.rockets.count`

### Special fields

Velo provides a couple of special fields that you can query but are not actually stored by the game. These are always prefixed by an underscore `_`:

- `Vector2`: `_length` or `_a` (float, gets the length of the vector, both do the same)
- `Player`: `_grappleCooldown`, `_slideCooldown` and `_surfCooldown` (float, note that these three cooldowns are handled a bit differently by the game's code and therefore are not stored directly)

Furthermore, there is a proxy target of type `Velo` which provides a couple more fields. Type `listFields Velo` to see them all. A couple examples:

- `get Velo.isIngame`
- `get Velo.isOnline`
- `get Velo.delta`
- `get Velo.screenWidth`

### 0-based vs 1-based indexing

As already mentioned, arrays in Lua use 1-based indexing (like our `arg` array). The game SpeedRunners and Velo mod however are written in C#, which uses 0-based indexing. This is why `get` and `set` also make use of 0-based indexing. If a script calls `set("Player#0.boost", 2)` or `get("Player.fireballs[0].isOnGround")`, then this is actually passed to Velo and run as a regular Velo command internally. Lua will have nothing to do with the actual execution.

### Example scripts

With all of this out of the way, let's write a couple of example scripts. Here's a simple implementation of a `move [x] [y]` command, which offsets the player by the specified x- and y-coordinates:

```lua
-- move.lua
x = get("Player.actor.position.x")
y = get("Player.actor.position.y")
set("Player.actor.position.x", x + arg[1])
set("Player.actor.position.y", y + arg[2])
```

Here, `arg[1]` and `arg[2]` are the x- and y-offsets passed into the script when calling `move [x] [y]`.

What happens however if you only pass `move 30`? The console will print an error message `"Index was outside the bounds of the array."` because `arg` has a length of 1 but we are trying to access the second element via `arg[2]`. To make the error message more specific, you can add some error checking to the beginning:

```lua
if #arg ~= 2 then
    echoErr("Error, you need to provide exactly 2 parameters 'x' and 'y'!")
    return
end
```

Note that `#arr` returns the length of an array. Furthermore, we can check for inequality in Lua using the `~=` operator. The `echoErr` command works like `echo`, just that it prints the message with the error color (red by default).

Here is a script that removes all obstacles from the map by making them invisible and uncollidable:

```lua
-- removeObstacles.lua
n = count("Obstacle")

for i = 0, n - 1 do
    target = "Obstacle#" .. tostring(i)
    set(target .. ".isBroken", true)
    set(target .. ".sprite.isVisible", false)
    set(target .. ".collisionFilter.categoryBits", 0)
    set(target .. ".breakTimer", -1 / 0)
end
```

We first get the count of all obstacles and then iterate through every possible index. We then set each obstacle's `isBroken` state to true, make their sprites invisible and make them uncollidable by setting the collision filter's category bits to `0`. Lastly, in order to prevent them from respawning again, we set the `breakTimer` to negative infinity, where infinity can be written as `1 / 0`. The obstacles would otherwise respawn once `breakTimer` counts up to `20.0`.

Note that the same could have been achieved way easier using the `despawn` command we haven't seen yet. This method however has the advantage that we could write another script that makes each obstacle reappear again.

## Callbacks

We have already seen how commands like `get` and `set` provide a communication interface between Velo and our scripts. These command calls are always initiated by our scripts. Callbacks provide another communication interface, this time however they are called by Velo on specific events. They allow you to define a function that then gets called by Velo on specific events. Let's list them all:

- `onPreUpdate()`: Gets called right before the game's update method gets called.
- `onPostUpdate()`: Gets called right after the game's update method was called.
- `onPreDraw()`: Gets called right before the game's draw methods were called.
- `onPostDraw()`: Gets called right after the game's draw methods were called.
- `onPreDrawLayer(layer)`: Gets called right before all objects in layer `layer` get drawn.
- `onPostDrawLayer(layer)`: Gets called right after all objects in layer `layer` were drawn.
- `onPostPresent()`: Gets called right after the GPU finished drawing the next frame.
- `onSetInputs(playerIndex)`: Gets called right after the player's inputs have been polled.
- `onPlayerReset(playerIndex)`: Gets called when a player's state gets reset.
- `onLapFinish(time)`: Gets called when the player finishes a lap.
- `onEcho(text)`: Gets called when some text gets printed to the console.
- `onReceiveRuns(requestId, runs)`: Gets called when receiving the response to a runs request.
- `onDownloadFinish(id, name)`: Gets called when the download of a recording has finished.
- `onStop()`: Gets called when the script is being stopped or the game exits.

We will see the significance of these in the remainder of this document. 

### Example scripts

Let's look at two examples first:

```lua
-- echoLapTime.lua
onLapFinish = function(time)
    echo(tostring(time))
end
```

This is a simple script that just prints out the final time to the console whenever the player finishes a lap. In order to achieve that, we define a callback function that gets called by Velo whenever the player finishes a lap.

We can define functions in Lua using the `function(args...) ... end` syntax. We assign a function to a variable called `onLapFinish`, which is a special name that Velo is able to recognize. This function then gets called every time the player has finished a lap, passing the final time to it.

```lua
-- rightRG.lua
wasSwinging = false

onPostUpdate = function()
	local swinging = get("Player.isSwinging")
	local moveDir = get("Player.moveDirection")

	if not wasSwinging and swinging and moveDir == 1 then
		local radius = get("Player.swingRadius")
		set("Player.swingRadius", radius + 11.3)
	end

	wasSwinging = swinging
end
```

This is a script that, when started, allows the player to do reverse grapples to the left (by default, they only work to the right). While not going too deep into the technical details of the game, the main idea here is that we want to increase the `Player.swingRadius` value a little whenever the player starts swinging, triggering the bug the same way it works for reverse grapples to the right. In order to achieve that, we need to continuously check on each frame whether the player is swinging or not, and if they do, change the value.

This is where the `onPostUpdate` callback comes in. This callback function gets called every time right after the game has finished its own update function, which handles all the physics and more. 

Note that `local` is a keyword that limits the scope of the declared variable to its current block (otherwise it would be global).

### Lifetime of scripts

Whenever you call a script that does not define any callbacks, it will be executed once (blocking the game's execution as stated earlier) and then be forgotten about (clearing all global variables).

If your script registers a callback function however, its state will be kept alive after finishing its initial execution, remembering all the global variables. The game continues its execution and then, whenever the callback function's event occurs, it will go back to that script and run the callback (with the stored state). Note that callback function calls are blocking, too.

You can stop a script using the `stop [name]` command and restart it using `restart [name]`, which will pass the same initial parameters to the script again. Furthermore, you can call `exit()` inside a script to stop it (do not use `stop`!). Note that `exit()` will not immediately exit the script but only on the next update. You may imagine it being more of a "request stop" function. You can get a list of all currently running scripts via `listRunning`.

## Inputs

Sometimes you might want your script react to certain hotkeys or have it write inputs to the player. Let's discuss how this can be achieved.

### Key inputs

In order to detect key inputs, Velo provides the following 4 commands. These commands take in a Windows virtual key code as a parameter and return a boolean that tells the key's current state.

- `isDown [keycode]` (true if the key is currently held)
- `isUp [keycode]` (true if the key is currently not held)
- `isPressed [keycode]` (true if the key is currently held and was not held on the previous frame)
- `isReleased [keycode]` (true if the key is currently not held and was held on the previous frame)

You can see a list of all the key codes [here](https://learn.microsoft.com/en-us/windows/win32/inputdev/virtual-key-codes). Alternatively, type `detectKey` into the console and press the key you wish to know the key code of.

Here is a simple example script that refills the player's boost bar on pressing F:

```lua
-- giveBoost.lua
onPreUpdate = function()
    if isPressed(0x46) then -- F key code
        set("Player.boost", 2)
    end
end
```

It just checks whether the F key is pressed on every frame and if so, sets the player's boost to `2`.

Apart from single keys, these commands can also handle combinations with SHIFT and CTRL. Perform a bitwise `or` with `MOD_SHIFT` (`0x100`) for a SHIFT+ combination and `MOD_CTRL` (`0x200`) for a CTRL+ combination. Examples:

- `isPressed(MOD_SHIFT | 0x31)` (SHIFT+1 is pressed)
- `isHeld(MOD_CTRL | 0x20)` (CTRL+SPACE is held)

Note that when passing single keys, these commands will not react to such key combinations; `isPressed(0x46)` will only detect F being pressed alone, but it does not react to SHIFT+F, CTRL+F or SHIFT+CTRL+F. In order to obtain modifier independent keys, Velo provides the `MOD_ANY` (`0x800`) flag; `isPressed(MOD_ANY | 0x46)` will detect any of F, SHIFT+F, CTRL+F or SHIFT+CTRL+F.

### Mouse inputs

For mouse buttons, you can use the same 4 commands as above with the following codes:

- `LEFT`, `RIGHT`, `MIDDLE`, `X1`, `X2`

You can further query the cursor position by using `get` with the following `Velo` fields:

`Velo.mouse.x`, `Velo.mouse.y`

### Controller inputs

For controller buttons, you can use the same 4 commands as above with the following codes:

- `A`, `B`, `X`, `Y`, `LEFT_SHOULDER`, `RIGHT_SHOULDER`, `LEFT_STICK`, `RIGHT_STICK`, 
`START`, `BACK`, `BIG_BUTTON`, `TRIGGER_LEFT`, `TRIGGER_RIGHT`, `DPAD_LEFT`, `DPAD_RIGHT`, `DPAD_UP`, `DPAD_DOWN`

You can further query the controller sticks and triggers by using `get` with the following `Velo` fields:
- `Velo.leftStick.x`, `Velo.leftStick.y`, `Velo.rightStick.x`, `Velo.rightStick.y`, `Velo.leftTrigger`, `Velo.rightTrigger`

### Querying and sending player inputs

Sometimes you might want to know which inputs a player is currently pressing, independent of which layout they are using. For this, `Player` has a couple of bool fields storing each input state that you can query using `get`:

`Player.leftHeld`, `Player.rightHeld`, `Player.jumpHeld`, `Player.grappleHeld`, `Player.slideHeld`, `Player.boostHeld`, `Player.itemHeld`

The following example gives the player a massive speed boost whenever they hold boost, depending on which direction they are currently holding:

```lua
-- giveBoost.lua
onPreUpdate = function()
    left = get("Player.leftHeld")
    right = get("Player.rightHeld")
    boost = get("Player.boostHeld")

    if not boost or left == right then
        return
    end

    if left then
        set("Player.actor.velocity.x", -1500)
    else
        set("Player.actor.velocity.x", 1500)
    end
end
```

In order to send inputs, you can use the same fields but with `set` instead. We need to be a bit more careful however; if we set inputs on `onPreUpdate()`, they will immediately be overwritten by the game's input polling on the next update, and if we set inputs on `onPostUpdate()`, they will also be ignored because all the updates have already been done. In order to solve this problem, Velo provides the `onSetInputs(playerIndex)` callback, which gets called right after the player's inputs have been polled but before all the player's physics updates are being done.

Here is an example script that automates spam grappling whenever V is held:

```lua
-- autoSpam.lua
onSetInputs = function(playerIndex)
    if isUp(0x56) then -- V key code
        return
    end

    local canGrapple = get("Player.canGrapple")
    local swinging = get("Player.isSwinging")
    local swingAngle = get("Player.swingAngle")
    local moveDir = get("Player.moveDirection")

    local holdGrapple = canGrapple -- grapple whenever possible
    local holdJump = not swinging

    if 
        swinging and 
        ((moveDir == 1 and swingAngle <= math.rad(90)) or 
        (moveDir == -1 and swingAngle >= math.rad(90))) 
    then
        holdGrapple = false -- release on reaching 90 degrees
    end

    set("Player.grappleHeld", holdGrapple)
    set("Player.jumpHeld", holdJump)
end
```

This script simply presses grapple whenever possible, releases grapple on reaching 90 degrees and holds jump whenever the player is not swinging. We ignore the player index, which is a value between `0` and `3`. Note that `Player.swingAngle` is given in radians.

## Lua tables

Tables are Lua's data structuring mechanism. You can imagine tables being similar to dictionaries / hash tables like in other languages, just more powerful. Tables associate keys and values of any type with each other. Example:

```lua
a = {} -- create an empty table
a[3] = "Hello" -- associate value "Hello" with key 3
a["x"] = false -- associate value false with key "x"
a["x"] = 7 -- change "x"'s value to 7
```

Adding a bit of syntactic sugar, Lua also allows accessing values of string keys using the dot syntax:

```lua
a["x"] = 7
a.x = 7 -- does the same as above

a[x] -- this is different!
```

You can initialize tables like so:

```lua
a = { ["x"] = 5, [4] = "Hello" }
b = { x = 5 } -- just writing `x` is another syntactic sugar for `["x"]`
c = { y = { z = 6, w = 1 } } -- nesting is possible, too
```

Note that there is no fixed relationship between a variable that holds a table and the table itself. In this regard, tables behave similar to objects like in many other programming languages:

```lua
a = { x = 7 }
b = a
b.x = 5
-- now a.x is also 5!
```

### Arrays

Arrays in Lua are just tables with numbers as keys. These numbers range from `1` to the array's length (remember, arrays use 1-based indexing). You can initialize an array like so:

```lua
arr = { "a", "b", "c", "d" }
arr[2] -- equals "b"
#arr -- length of arr, equals 4
```

### `Vector2`

As we have already seen, fields like `Player.actor.position` and `Player.actor.velocity` are of type `Vector2`. They represent a two-dimensional vector (or point) with an x- and y-coordinate. So far, whenever we wanted to get the Player's position or velocity, we needed to call `get` twice in order to retrieve `x` and `y` individually. We can do better however; if you call `get("Player.actor.position")`, it actually returns a table with keys `"x"` and `"y"`. These tables further provide `+`, `-`, `*` and `/` operators that we can use. You can create a new such table using `Vector2:new(x, y)`.

This allows us to simplify the `move [x] [y]` script from earlier like so:

```lua
-- move.lua
position = get("Player.actor.position")
offset = Vector2:new(arg[1], arg[2])
set("Player.actor.position", position + offset)
```

Note that outside Lua, `Vector2` uses the syntax `x,y`:

- `set Player.actor.velocity 1200,-300`

### `Color`

Similarly, some commands use colors, for which some utilities are provided. You can create a new color table using `Color:new(r, g, b, a)` which then contains fields `"r"`, `"g"`, `"b"` and `"a"`. Note that these values must range between `0` and `255`. If you don't specify `a`, it defaults to `255`. Operators `+`, `-`, `*` and `/` are provided (multiplying two colors behaves like first converting them to the [0, 1] range, multiplying them component-wise and then converting the result back to the [0, 255] range).

Note that outside Lua, `Color` uses the syntax `r,g,b,a` or `r,g,b`

## Time

The `Velo` proxy target provides several fields that allow you to query different types of time. Note that a lot of these times use ticks as their unit, where 1 tick equals 100 nanoseconds. You can convert these to seconds using `TICKS_PER_SECOND` (`10000000`).

- `Velo.delta`: The game's measured elapsed time between the previous and current update in ticks (note that it's `0` when the game is frozen)
- `Velo.deltaSec`: Same as `Velo.delta`, but as a float measuring the seconds
- `Velo.totalTime`: The game's measured total time since the start of the game in ticks (note that it doesn't count up when the game is frozen)
- `Velo.realTime`: The current system time in ticks (ticks since epoch, which is 01.01.1601)
- `Velo.realDelta`: The elapsed time between the previous and current update in ticks
- `Velo.realDeltaSec`: Same as `Velo.realDelta`, but as a float measuring the seconds

The difference between the `real` and non-`real` versions is that the `real`-versions use the system clock while the non-`real` versions are what the game uses internally. If the game is frozen, like when TASing or pausing a replay, the non-`real` versions are affected by this and `delta` is `0` and `totalTime` does not count up.

Here is a simple example script that periodically measures the current framerate and prints it to the console:

```lua
-- fps.lua
totalElapsed = 0
frames = 0

onPreUpdate = function()
    totalElapsed = totalElapsed + get("Velo.realDeltaSec")
    frames = frames + 1
    if totalElapsed >= 1 then
        echo(tostring(frames))
        totalElapsed = 0
        frames = 0
    end
end
```

On each frame, we increment the `frames` counter and add the elapsed time to `totalElapsed`. Once 1 second has passed, we print the `frames` counter and reset our variables.

### Recorders and replays

`Velo` further provides a couple of fields to query the current recorder's or replay's seek position. We will refer to a recorder or replay as a *seekable*:

- `Velo.frame`: The seekable's current frame position
- `Velo.frameStart`: The seekable's first frame
- `Velo.frameEnd`: One past the seekable's last frame (subtract 1 to get the last frame)
- `Velo.seek`: The seekable's current position in seconds
- `Velo.seekStart`: The seekable's first possible position in seconds
- `Velo.seekEnd`: The seekable's last possible position in seconds

Note that `frameStart` and `seekStart` are not necessarily equal to `0`. If there is a lap reset or lap finish in the recording, the `0`-position is always the very first frame where a lap reset or lap finish occurred (whichever comes first). Any frame or seek position before that is negative. This is to ensure that the actual run always starts at `0` and that ghosts always sync perfectly (be it new lap, 1 lap or TAS recordings). You can take the difference between the end and start to get the recording's total length/count.

You can then seek through the seekable using the following commands:

- `jump [frames]`: Jump frames forward
- `jumpBack [frames]`: Jump frames back
- `jumpTo [frame]`: Jump to frame
- `jumpSec [seconds]`: Jump seconds forward
- `jumpBackSec [seconds]`: Jump seconds back
- `jumpToSec [second]`: Jump to second
- `jumpToEnd`: Jump to the last frame

Note that these only work on replays and TAS-projects. Some of these may freeze the game. Use `freeze` and `unfreeze`.

## Drawing

Velo provides a few commands that allow you to draw simple shapes and text to the screen. Note that these commands need to be called on `onPostDraw()` or any of `onPreDrawLayer(layer)` and `onPostDrawLayer(layer)`. Furthermore, they only draw the specified shape or text for a single frame, meaning that these draw commands need to be called continuously on every frame.

### Screen coordinates

Every draw command takes in position values as a `Vector2` with the coordinate system's origin being on the screen's top-left corner and a single unit representing a single pixel.

When drawing something, we may either want to draw it to a fixed position on the screen or draw it inside the level at some position. The first case is easy and we can just pass the position value unaltered. The second case requires us to transform the coordinates from world coordinates to screen coordinates. This is because the camera can have different offsets and zoom levels, making both coordinate systems different. 

For these transformations, Velo provides `worldToScreen [position]` and `screenToWorld [position]` commands. Use the former for the situation described above. Here is an example where the latter might become useful:

```lua
-- snapToMouse.lua
enableCursor(true)

onPostUpdate = function()
    if isPressed(LEFT) then
        mouse = get("Velo.mouse")
        position = screenToWorld(mouse)
        set("Player.actor.position", position)
    end
end

onStop = function()
    enableCursor(false)
end
```

This script just snaps the player to the current cursor position whenever you left click. The field `Velo.mouse` uses screen coordinates while the player uses world coordinates. We can show or hide the cursor using the `enableCursor [enabled]` command. `onStop()` is a callback function that gets called whenever the script get stopped.

### Rectangles

You can draw filled rectangles using `drawRect [position] [size] [color]`. Note that `position` and `size` are of `Vector2` type and `color` of `Color` type. To only draw the rectangle's outline, use `drawRectOutline [position] [size] [thickness] [color]`.

Here is a simple example that always just draws a small red rectangle on the cursors current position:

```lua
-- mouseRect.lua
onPostDraw = function()
    position = get("Velo.mouse")
    size = Vector2:new(8, 8)
    drawRect(position - size / 2, size, Color:new(255, 0, 0))
end
```

### Lines

You can draw lines similarly using `drawLine [start] [end] [thickness] [color]`. 

Here is an example that draws a line going out of the player, representing the velocity vector:

```lua
-- velocityVector.lua
onPostDraw = function()
    position = get("Player.actor.position")
    velocity = get("Player.actor.velocity")
    hitboxSize = Vector2:new(25, 45) -- the player's hitbox
    center = position + hitboxSize / 2
    startPos = worldToScreen(center)
    endPos = worldToScreen(center + velocity)
    drawLine(startPos, endPos, 3, Color:new(255, 0, 0))
end
```

This time, we transform world coordinates (the player) to screen coordinates.

### Triangles

You can draw triangles using `drawTriangles [triangles] [color]` where `triangles` is a `Vector2` array of vertex triplets denoting each triangle's vertices.

Here is an example drawing an hourglass shape made up of two triangles:

```lua
-- hourglass.lua
onPostDraw = function()
    width = get("Velo.screenWidth")
    height = get("Velo.screenHeight")
    drawTriangles({
            Vector2:new(0, 0),
            Vector2:new(width, 0),
            Vector2:new(width / 2, height / 2),

            Vector2:new(width / 2, height / 2),
            Vector2:new(width, height),
            Vector2:new(0, height)
        },
        Color:new(255, 0, 0)
    )
end
```

Using the `drawColoredTriangles [triangles] [colors]` command, we can even set colors to each vertex individually, which are then mixed on each triangle when drawing. Here, `colors` is a `Color` array.

Here is the same hourglass example but with some mixed colors:

```lua
-- hourglass.lua
onPostDraw = function()
    width = get("Velo.screenWidth")
    height = get("Velo.screenHeight")
    drawColoredTriangles({
            Vector2:new(0, 0),
            Vector2:new(width, 0),
            Vector2:new(width / 2, height / 2),

            Vector2:new(width / 2, height / 2),
            Vector2:new(width, height),
            Vector2:new(0, height)
        }, {
            Color:new(255, 0, 0),
            Color:new(0, 255, 0),
            Color:new(255, 255, 255),

            Color:new(0, 0, 0),
            Color:new(0, 0, 255),
            Color:new(255, 255, 0)
        }
    )
end
```

### Text

In order to draw text, use the `drawText` command. This command has the following signature:

- `drawText [text] [position] [size] [align] [scale] [rotation] [font] [fontSize] [shadow] [color]`

You can imagine the text to be drawn to be enclosed inside a bounding box of position `position` and size `size`. The `align` vector determines the alignment, where a value of `0` puts the text as far left or up as possible without leaving the bounding box and `1` puts the text as far right or down as possible. Values inbetween are interpolated, meaning that `0.5` would center the text for example.

`font` is a path to the font file which uses the `Content` folder as the root directory.

`shadow` is a bool determining whether the text should drop a shadow or not.

Here is an example drawing "Hello World!" to the top center of the screen:

```lua
-- helloWorld2.lua
onPostDraw = function()
    width = get("Velo.screenWidth")
    height = get("Velo.screenHeight")
    drawText(
        "Hello World!",
        Vector2:new(0, 0), -- position
        Vector2:new(width, height), -- size
        Vector2:new(0.5, 0), -- alignment
        1, -- scale
        0, -- rotation
        "UI\\Font\\Souses.ttf", 24,
        true, -- drop a shadow
        Color:new(255, 0, 0)
    )
end
```

### Layering

When using `onPostDraw()`, everything we draw will always be on top of everything (including Velo's own elements). What if we want something to be drawn behind certain objects, like the tile layer? The game makes use of a layering system in order to ensure a certain draw order; the tile layer belongs to the `"Collision"` layer while the player belongs to the `"LocalPlayersLayer"`, where the latter is drawn after the former, making the player always appear in front every tile.

In order to make use of these layers ourself, Velo provides the `onPreDrawLayer(layer)` and `onPostDrawLayer(layer)` callbacks, which are called before and after each layer is drawn. `layer` is a string determining the layer's ID.

Here is a list of all the layer IDs in order:
`"DefaultUILayer"`, `"VersionNumberLayer"`, `"PopupLayer"`, `"TopUILayer"`, `"UserUILayer"`, `"CursorUILayer"`, `"BackgroundLayer0"`, `"BackgroundLayer1"`, `"BackgroundLayer2"`, `"BackgroundLayer3"`, `"BackgroundLayer4"`, `"BackgroundLayer5"`, `"NonParallaxingBackLayer"`, `"ParallaxLayer: 0.800"`, `"ParallaxLayer: 0.825"`, `"ParallaxLayer: 0.850"`, `"ParallaxLayer: 0.875"`, `"ParallaxLayer: 0.900"`, `"ParallaxLayer: 0.925"`, `"ParallaxLayer: 0.950"`, `"ParallaxLayer: 0.975"`, `"BackObjectLayer"`, `"Background 0"`, `"Background 1"`, `"MiddleObjectLayer"`, `"Shading"`, `"Overlay"`, `"GameplayObjectsLayer"`, `"Collision"`, `"ObjectLayer"`, `"TrailBehindRemotePlayersLayer"`, `"RemotePlayersLayer"`, `"TrailInFrontOfRemotePlayersLayer"`, `"TrailBehindLocalPlayersLayer"`, `"LocalPlayersLayer"`, `"TrailInFrontOfLocalPlayersLayer"`, `"tilePreview"`, `"temp"`

If you want to draw to a specific layer, you can just use an `if`-check to ensure the current layer `layer` is the correct one.

## Demo: Implementing a simple speedometer

We now have all the tools necessary to implement our own little speedometer as a Lua script. Let's go step-by-step through the whole implementation. The full demo is contained in your "scripts" folder.

### Setting up variables

Let's first set up a couple of global variables that we need:

```lua
REFRESH_INTERVAL = 0.1
refreshTimer = 0
speed = 0
enabled = true
```

We want the speedometer to refresh every 0.1 seconds (as determined by `REFRESH_INTERVAL`). For this, we create a timer variable `refreshTimer` and initialize it with `0`. On each refresh, we further store the measured speed into `speed`. Lastly, we want to be able to toggle our speedometer using F3. We create a variable `enabled` for the current toggle state.

### Handling the toggle

```lua
onPostDraw = function()
    if isPressed(0x72) then -- Windows virtual key code for F3
        enabled = not enabled
    end

    if not enabled then
        return
    end
```

Next, we create an `onPostDraw()` callback where we want to implement all the logic. We use `isPressed` to check whether F3 was pressed or not. If it was, we toggle the state of `enabled`. If `enabled` is false, we return (skipping the upcoming draw call).

### Checking if ingame

```lua
if not get("Velo.isIngame") then
    return;
end
```

We check whether we are currently ingame and return if not. This is to prevent errors from the upcoming `get` calls.

### Ticking the refresh timer

```lua
refreshTimer = refreshTimer - get("Velo.realDeltaSec")
if refreshTimer <= 0 then
    speed = get("Player.actor.velocity._length")
    refreshTimer = REFRESH_INTERVAL
end
```

We tick down the refresh timer by subtracting `Velo.realDeltaSec` which gives us the elapsed time since the last update call. Once our refresh timer goes below `0`, we update our `speed` variable and set the timer back to `REFRESH_INTERVAL`. In order to calculate the player's speed from their velocity, we use the special `_length` field. Alternatively, you can use `Vector2:length(v)` as well.

### Determining the screen coordinates

We determine the screen coordinates the following way:

```lua
local position = get("Player.actor.position")
local offset = Vector2:new(12.5, -120)
local screenCoords = worldToScreen(position + offset)
```

We first get the player's position and then transform it from world coordinates to screen coordinates using `worldToScreen`. Additionally, we apply a little offset to make the speedometer appear more centered and above the player. `12.5` is half of the player's hitbox width.

### Drawing the text

Finally, we draw the speedometer's text to the screen:

```lua
    local scale = 1 / get("CCamera.zoom")
    local r = math.min(speed / 1500, 1)

    drawText(
        tostring(math.floor((speed + 2.5) / 5) * 5), -- the text to draw
        screenCoords,
        Vector2:new(), -- bounding box size (let's just keep it at zero)
        Vector2:new(0.5, 0), -- alignment (let's center it horizontally)
        scale,
        -0.05, -- rotation
        "UI\\Font\\Souses.ttf", 24,
        true, -- drop a shadow
        Color:mix(Color:new(0, 200, 200), Color:new(180, 0, 0), r)
    )
end -- ending the callback function
```

In order for the text to scale properly, we first need to query the camera's current zoom level and take the reciprocal. We further calculate a color mix ratio `r` which should be `0` when the speed is `0` and `1` when the speed is `1500` or beyond. Lastly, we call the `drawText` function, passing all the required parameters. By doing `math.floor((speed + 2.5) / 5) * 5`, we can round the speed value to the nearest multiple of 5. Note that we pass a bounding box size of zero, allowing us to precisely place the text in relation to the point `screenCoords`. We pass a horizontal alignment of `0.5` to center it horizontally. Using `Color:mix(c1, c2, r)`, you can linearly interpolate between two colors using the ratio `r`. Here, we interpolate between cyan and red.

Feel free to expand on this script as you wish.

## Registering subcommands

In some situations, you might want multiple commands to belong to a single script; imagine wanting to write a `lock` script that takes in a field and a value and then continuously writes the value to the field on every frame, essentially "locking" its value ("lock" the player's boost to `2`, or "lock" the player's position to some point). The problem now is that you might want to add several commands like `add`, `remove`, `list`, `clear` and so on in order to manage all your locks. We cannot just write a separate script for each of these because different scripts don't share their variables.

To solve this problem, Velo allows you to register subcommands by adding a new function to the `cmds` table. This function can take any number of parameters, which you can then pass when calling the command. If script `x` defines a subcommand `y`, it can then be called via `x.y [args]...`. As with callbacks, registering a subcommand also extends the script's lifetime.

Let's take a look at the following example:

```lua
-- maneki.lua
counter = 1
cmds["say"] = function(text)
    echo(tostring(counter) .. ": Maneki-neko says '" .. text .. "'!")
    counter = counter + 1
end
```

After running `maneki`, you can then type `maneki.say "Hello"` in order to execute our registered `say` subcommand, which takes a single parameter `text`. As demonstrated by the `counter` variable, we can now retain the script's state between several invocations.

Let's see how one might implement the `lock` script:

```lua
-- lock.lua
locks = {}

cmds["add"] = function(field, value)
    locks[field] = value
end

cmds["remove"] = function(field)
    locks[field] = nil
end

cmds["list"] = function()
    for f, v in pairs(locks) do
        echo(f .. ": " .. tostring(v))
    end
end

cmds["clear"] = function()
    locks = {}
end

onPostUpdate = function()
    if not get("Velo.isIngame") then
        return
    end
    for f, v in pairs(locks) do
        set(f, v)
    end
end
```

After calling `lock`, we can now use our 4 registered subcommands like so:

- `lock.add [field] [value]`: Lock a field to a value
- `lock.remove [field]`: Remove a field's lock
- `lock.list`: List all locks
- `lock.clear`: Clear all locks

Their basic usage is `lock.add "Player.boost" 2` or `lock.add "Player.actor.position" Vector2:new(3000, 2000)`. Note that once again, the parameters use Lua syntax as opposed to the regular Velo console syntax, meaning that strings have to be enclosed by quotes and you need to use `Vector2:new(x, y)` to pass vectors.

We store each lock as a key-value-pair into a `locks` table with the field to lock being the key and the value to set being the value.

## Executing scripts on startup

Sometimes you might want a script to always be running as soon as you open up the game without having to manually type its command each time. This is where the "onStart.lua" script comes into play. This script is special as Velo will always run it once the game starts.

You can use this script to run other scripts using the `run(name, args...)` function. Here, `name` is the name of the script and `args...` are the arguments to pass.

Examples:

```lua
-- onStart.lua
echo("Started!")
run("speedometer")
run("lock")
run("lock.add", "Player.actor.velocity", Vector2:new(1200, 0))
```

Note that Lua also provides a similar `require(name)` function which takes another script and executes it. Doing this however makes the other script run as part of the current script; if we used `require` in the above example instead, it would make all scripts run as part of the "onStart" script, making them all share the same global variables. This especially applies to all the callbacks and subcommands, allowing some scripts to overwrite them over others'. We would further have to call `onStart.add` instead of `lock.add` and they would all appear as `onStart` when calling `listRunning`.

## Error handling

Some Velo commands can throw an error if run on bad conditions; a lot of commands can only be run when ingame or not in an online match. It is therefore a good idea to guard your scripts via `get("Velo.isIngame")` and `get("Velo.isOnline)` checks.

Once an error was thrown, your script will terminate and you need to start it again. In order to catch an error, Lua provides a `pcall` function, which takes in a function, calls it and then returns a status bool telling you whether the function call was successful or any errors were thrown.

Here is an example:

```lua
-- errorTest.lua
function someFunc()
    x = get("Player.actor.position.x")
    echo(tostring(x))
end

status, err = pcall(someFunc)
if not status then
    echoErr(err.Message)
end
```

The function `someFunc` just prints out the player's current x-position. This requires you to be ingame however, so it will throw an error if we are not. We wrap the function call into the `pcall` function, which then returns the status `status` and, in case of an error, the error `err`. If `status` is false, we then print the error message contained in `err`.

Note that `err.Message` in this case will return `"A .NET exception occurred in user-code."`, which is not very helpful. This is because the actual error occurred internally on Velo's side of things, not in our script. In order to retrieve the actual error, we can just do `err.InnerException.Message` and get `"You must be ingame!"`.

With more compact syntax, our example becomes:

```lua
-- errorTest.lua
status, err = pcall(function()
    x = get("Player.actor.position.x")
    echo(tostring(x))
end)
if not status then
    echoErr(err.InnerException.Message)
end
```

If you don't care about the error message, you can simply do `if pcall(...) then ... end` or `if not pcall(...) then ... end`.

## Querying and modifying Velo settings

Similar to `get` and `set`, there are also `getSt [setting]` and `setSt [setting] [value]` which allow you to query and modify Velo settings. Instead of having to specify a target and field path, you instead need to specify a module and setting path. The module can be `Appearance` or `Offline Game Mods` for example. The setting path is then determined by the nesting structure, where we have to separate each path element using a dot like with `get` and `set`. Note that you may use underscores `_` instead of spaces.

Here are a couple of examples, some for the Velo console and some in Lua:

- `getSt Appearance.popup.opacity`
- `setSt Offline_Game_Mods.physics.max_speed 2000`
- `setSt "Offline Game Mods.physics.max speed" 2000` (needs quotes)

```lua
-- underscores not strictly required
getSt("Rope_Indicators.general.max_count")

setSt("Console.stats_window.style.offset", Vector2:new(200, 100))

setSt("Speedometer.enabled", Toggle:new(0x72, true)) -- key code and toggle state

setSt(
    "Input_Display.boxes.left_box", 
    -- position, size and label
    InputBox:new(Vector2:new(0, 64), Vector2:new(64, 64), "<")
)

setSt("Appearance.grapple.rope_color", Color:new(0, 255, 255))

setSt(
    "Appearance.grapple.rope_color", 
    -- period, offset, discrete and colors
    ColorTransition:new(1000, 0, false, {
        Color:new(255, 0, 0),
        Color:new(255, 255, 0),
        Color:new(0, 255, 0),
        Color:new(0, 255, 255),
        Color:new(0, 0, 255),
        Color:new(255, 0, 255),
    })
)
```

Using `defaultSt [setting]` we can restore the default value of a setting and using `defaultAllSt [module]` we can restore the default value for every setting of some module. Pass `nil` as the module to completely restore every default for every module.

## Managing savestates

You can manage savestates using the following commands:

- `saveSs [name]`
- `loadSs [name]`
- `renameSs [oldName] [newName]`
- `copySs [sourceName] [targetName]`
- `deleteSs [name]`
- `listSs`
- `clearSs`
- `undoSs`

Every savestate is given some name. Using `saveSs [name]` you can create a new savestate and using `loadSs [name]` you can load it. Use `undoSs` to undo the loading of the previously loaded savestate. Every savestate is stored to file under "Velo\\savestates", ensuring they are always retained even after restarting the game. The 10 savestate slots accessible via hotkeys use the names `"ss1"`, ..., `"ss10"`.

## Spawning and despawning actors

Using `spawn [type] [position]` you can spawn a new actor to the level and using `despawn [target]` you can despawn one.

Examples:

- `spawn("Pickup", Vector2:new(4000, 3000))`
- `despawn("DroppedObstacle#6")`

Note that `spawn` does not behave very well yet, especially when spawning player items. They tend to be invisible and have no collision, which you can fix with subsequent `set` calls in some cases.

## Getting and setting tiles

Using `getTile [x] [y]` we can get the tile ID at some position and using `setTile [id] [x] [y]` we can place a tile. Please refer to the following list of IDs:

 - `0`: empty
 - `1`: full
 - `2`: wall, dots on the left
 - `3`: grapple ceiling
 - `4`: wall, dots on the right
 - `5`: checkered
 - `6`: floor slope, upper tip on the right
 - `7`: floor slope, upper tip on the left
 - `8`: stairs, upper tip on the right
 - `9`: stairs, upper tip on the left
 - `10`: checkered floor slope, upper tip on the right
 - `11`: checkered floor slope, upper tip on the left
 - `12`: ceiling slope, lower tip on the right
 - `13`: ceiling slope, lower tip on the left
 - `14`: checkered ceiling slope, lower tip on the right
 - `15`: checkered ceiling slope, lower tip on the left

To get the width and height of the current tile map, query `Velo.tileWidth` and `Velo.tileHeight`. The top-leftmost tile is placed at `0`, `0`.
 
Please note that the tile map's coordinate system uses different units than actors, as each tile is made up of 16 "pixels". If we refer to what the tile map uses as *tile coordinates* and what the actors use as *world coordinates*, we can transform tile to world coordinates by multiplying `x` and `y` by `16` and world to tile coordinates by dividing by `16` and truncating the fractional part. You can use the helper functions `tileToWorld(position)` and `worldToTile(position)` for this.

The following example script places floor tiles below the player whenever `B` is held down:

```lua
-- floorPlacer.lua
onPreUpdate = function()
    if isUp(0x42) then -- B key code
        return
    end

    position = get("Player.actor.position")
    hitboxSize = Vector2:new(25, 45)
    tilePosition = worldToTile(position + Vector2:new(hitboxSize.x / 2, hitboxSize.y))
    setTile(1, tilePosition.x, tilePosition.y + 1)
end
```

## Console escape sequences

Velo's console allows for certain escape sequences when using `echo` and `echoErr` in order to change the text's style and color. These escape sequences are directly inlined into the text and apply from the position they were placed at until being overwritten by another escape sequence. Each escape sequence has the form `$[x:...]` where `x` determines the option you want to change. Let's list them all:

- `$[i:true]`, `$[i:false]`: Make the style italics or non-italics
- `$[b:true]`, `$[b:false]`: Make the style bold or non-bold
- `$[c:r,g,b]`: Change the color
- `$[in:n]`: Change the indentation

Here is an example for changing styles:

- *italics* normal **bold** ***both***
- `$[i:true]italics $[i:false]normal $[b:true]bold $[i:true]both`

Here is an example for changing indentation:

```
Test
  Test
    Test
    Test
  Test
Test
```
`Test\n$[in:2]Test\n$[in:4]Test\nTest\n$[in:2]Test\n$[in:0]Test`

The current indentation value is preserved through any form of line-breaks. The `help` command for example makes use of this escape sequence.

## Querying the leaderboard

Velo provides a couple of commands that allow you to query runs from the leaderboard. These commands send a new request to the leaderboard server and return a unique request ID. Once the response comes in, Velo will call the `onReceiveRuns(requestId, runs)` callback, where `requestId` is the request ID returned from the command earlier and `runs` is an array of `RunInfo` tables. If the request failed, the `runs` array will be `nil`.

### `RunInfo` table

The `RunInfo` tables contain the following fields:

- `id`: ID of the run
- `playerId`: Steam ID of the player
- `createTime`: Unix time of when the run was created or submitted
- `runTime`: Final time of the run in milliseconds
- `mapId`: ID of the map the run was played on
- `categoryId`: The run's category
- `place`: The run's place (0-based, `0` corresponds to 1st place)
- `wasWr`: Whether the run was a WR at any point
- `hasComments`: Whether the run has comments
- `outdated`: Whether the run was played on an old map version
- `deletedRecording`: Whether the run's recording was deleted
- `noSavestate`: Whether recording should not load any savestates
- `tas`: Whether the run is a TAS
- `newGCD`: Whether the run was played using the new grapple cooldown of 0.20s (as opposed to the old 0.25s)
- `fixBounceGlitch`: Whether the run was played using the bounce glitch fix
- `dist`: Distance
- `groundDist`: Ground distance
- `swingDist`: Swing distance
- `climbDist`: Climb distance
- `avgSpeed`: Average speed
- `grapples`: Grapple count
- `jumps`: Jump count
- `boostUsed`: Boost used (in percent)

### Map IDs

You can type `listMaps` to get a complete list of all map IDs. You can further use `get Velo.mapId` to get the current map ID. In general however, these map IDs have the following structure:

- Officials: `0` - `16` in chronological order (`0` is "Metro", `16` is "Laboratory")
- Origins: `55` - `84` in chronological order (`55` is "Prologue", `83` is "Final Challenge", `84` is "Unused")
- RWs and old RWs except: May change from update to update
- non-curated Workshop: Steam ID (you can see in the map page's URL)

### Category IDs

- `NEW_LAP` (`0`): New lap
- `ONE_LAP` (`1`): 1 lap
- `NEW_LAP_SKIP` (`2`): New lap (Skip)
- `ONE_LAP_SKIP` (`3`): 1 lap (Skip)
- `ANY_PERC` (`4`): Any%
- `HUNDRED_PERC` (`5`): 100%
- `EVENT` (`6`): Event

### Requests

Velo provides the following request commands:

- `requestWRRuns [type] [place]`
    - `type`: `0`: Velo curated, `1`: Other
    - `place`: 0-based place (`0` is 1st place)
- `requestPlayerPBRuns [player] [type]`
    - `player`: Steam ID of the player
    - `type`: `0`: Velo curated, `1`: Other
- `requestMapCategoryRuns [mapId] [categoryId] [filter] [start] [count]`
    - `mapId`: Map ID
    - `categoryId`: Category ID
    - `filter`: `0`: PBs only, `1`: WR history, `2`: All
    - `start`: Index of first run to request
    - `count`: Total number of runs to request (max `100`)
- `requestRecentRuns [filter] [start] [count]`
    - `filter`: `0`: All, `1`: WRs only
    - `start`: Index of first run to request
    - `count`: Total number of runs to request (max `100`)
- `requestTASRuns`

### Downloading recordings

You can use the `download [id] [name]` command to download and save a recording from the leaderboard. Once a download has finished, you will be notified via the `onDownloadFinished(id, name)` callback.

### Example

The following example requests the top 5 runs of the current map, downloads their recordings and then sets their ghosts:

```lua
-- setGhosts.lua
COUNT = 5
mapId = get("Velo.mapId")
requestId = requestMapCategoryRuns(mapId, NEW_LAP, 0, 0, COUNT)

for i = 0, COUNT - 1 do
    prepareGhost(i)
end

onReceiveRuns = function(requestId_, runs)
    if requestId_ == requestId then
        for i, run in ipairs(runs) do
            download(run.id, "_temp" .. tostring(i))
        end
    end
end

nextGhostIndex = 0

onDownloadFinished = function(id, name)
    setGhost(name, nextGhostIndex)
    deleteRec(name)
    nextGhostIndex = nextGhostIndex + 1

    if nextGhostIndex == COUNT then
        exit()
    end
end
```

On `onReceiveRuns`, we loop through all received runs and use the `download` command to request the recordings for each of them. Note that the construct `for i, elem in ipairs(arr) do ... end` will iterate through an `arr`, where `elem` is each element and `i` the corresponding index. 

On `onDownloadFinished`, we use `setGhost [name] [ghostIndex]` to set the ghosts to each recording we downloaded. We further use `prepareGhost` to spawn each ghost in advance. This is not necessary strictly speaking but cuts down on a lot of waiting time; whenever the `setGhost` command is used, Velo needs to ensure the ghost has existed for at least 1 second to prevent crashes. If it didn't yet, Velo will wait and freeze the game.

Once every ghost has been set, we call `exit()` to stop the script.