## Getting Started
This workshop requires a copy of [Pico-8](https://www.lexaloffle.com/pico-8.php), which unfortunately costs $15. The good news is that Pico-8 is DRM-free, which means that you can share one license with your whole club.

It is also strongly recommended that you download this wonderful cheatsheet:

![](assets/cheatsheet.png)

One common practice is to fullscreen it behind Pico-8, like so:

![](assets/pico_with_cheatsheet.png)

## Let's go!
Tired of complex workflows and high-definition graphics? Well, the Pico-8 fantasy console is here to help. Through the course of this workshop you will make [this](demos/final.html) game. Along the way you'll learn how to use the Pico-8 console along with the basics of Lua scripting.

<iframe src="demos/demo.html" width="100%" height="700px">
  <image src="assets/demo.gif">
</iframe>

Before we begin, let's learn how to use Pico. It has three views:

 * The console mode

   ![](assets/console.gif)

   This is what you see on startup, and allows you to run a couple of basic commands. Don't worry too much about it for now.

 * The editor mode

   ![](assets/editor.png)

   This is where you make your game. It has the following tabs:
   * code editor
   * sprite editor
   * tilemap editor
   * sound effects editor
   * music editor

   **You can get to it by pressing `Esc` from the console mode.**

 * The game mode

   ![](assets/game.png)

   This is where you can test your game. **You can get to it by pressing `Ctrl-R`. To get back to the editor, press `Esc` twice.**

## Explore

![](assets/splore.gif)

Let's start out by playing other people's games. Type `splore` in the console, refresh the game list, and mess around with a couple - these were all made with Pico. After you find one you like, press `esc` to bring up the menu, `exit to splore`, `esc` to bring up the console, and `esc` one more time to bring up the editor.

![](assets/splore_exit.gif)

Try modifying the sprites or tiles and then rerunning the game with `Ctrl-R` to get familiar with the editor.

## Start making!
We can start a new game from scratch with the `reboot` command.

Let's make our game's player. To do this, first go to the editor mode. In the top right of the screen, you'll see five icons: The second icon from the left (the little face) will take you to the **sprite editor**.

We can make drawings in the (currently empty) blank square to the left. The square to the right filled with colors is our pallet; click on a color to select it for drawing. Try making a player and then some terrain tiles. When you finish a sprite you can click on the next blank spot in the bottom to start draw a new one.

![](assets/sprite_editor.gif)

You can use your terrain sprites to draw an island where the game will take place. Do this in the map editor (the next editor tab). At the bottom of the map view, select some sprites and add them to the map.

![](assets/map_editor.gif)

Now that you've made the player and setting, you'll need to use some code to display them to the screen. Feel free to pause here and customize your game: build a town or farm instead of an island, an actual player better instead of the atrocious smiley face, anything - the possibilites are endless!

## Code, oh my
Before we start coding, some theory:

![](assets/gameloop.png)

Every game has something called a gameloop, which is code that is called every frame. This gameloop is split into three parts: first, the game collects user input (is the right arrow pressed?). Then, the game updates some internal variables (move the player right). Finally, the game redraws the screen to reflect the new state.

In Pico-8, you can define that drawing phase by placing code in the following way:
```lua
function _draw()

 -- place code here!

end
```
Don't worry about what exactly is going on here, just understand that anything that goes in-between the `function` and `end` will be drawn onto the screen.

Sidenote: in Pico-8, any line of code that begins with `--` is ignored, so it's useful for writing various comments about the code.

With that in mind, let's draw our sprite.

```lua
function _draw()

 spr(1, 10, 10)

end
```
`spr` is what is called a function: some code that can be run at any point. We can also pass information into functions, which is what we're doing in this example. Concretely, `spr` accepts 3 pieces of information (in this order):
 * which sprite your drawing
 * x coordinate of the top left corner
 * y coordinate of the top left corner
Note how I give Pico-8 a number to indicate the sprite I'm drawing. Each sprite has its own number, and it can be found in the sprite editor, to the right of the sprite. In the image below, the sprite number is in number in the yellow circle.

![](assets/sprite_number.png)

So, `spr(1, 10, 10)` will draw sprite number 1 at the point `(10, 10).`

STOP: Before reading further, press `Ctrl-R` to execute your code. Please please do this before I tell you what happens!

Have you seen it?

...

When you actually execute this (`Ctrl-R`), you may notice you still have artifacts from the console. To fix this, we will use the `cls()` function, which clears the screen. That way, anything you typed in the console earlier will be erased and won't block our view of the game.

```lua
function _draw()

 cls() -- this clears the screen
 spr(1, 10, 10)

end
```
Now every frame will start from a blank screen.

It's very important we put `cls()` before putting `spr(1, 10, 10)`. Try thinking about why, and then test your guess--swap the order of the lines, and then run the program!

You shouldn't see anything on your screen. That's because Pico runs lines in order: First, it runs `cls()` and then it runs `spr(1, 10, 10)`. So, when you give Pico the code

```lua
function _draw()

 spr(1, 10, 10)
 cls()
 
end
```

you're telling Pico "draw a sprite, and then erase everything on the screen." Since "everything on the screen" includes our sprite, that gets erased as well. Uh oh! 

## Coordinates

One thing to note: Coordiantes work a little different in Pico-8. The point `(0, 0)` is in the top left corner, and making the y-coordinate bigger actually makes the point go down, not up.

![](assets/coordinate_grid.png)

So, the point `(10, 10)` can be thought of as going 10 pixels to the right of the top left pixel, and then going 10 pixels.

## Variables
Right now our "game" is a little boring, as the square isn't moving at all. The reason behind this is that we're always giving it the same coordinates. The solution? Variables.
Variables are essentially little boxes that allow you to store a value, like a number.
```lua
x = 3
```

Whenever Pico-8 comes across a variable it replaces that variable with the value that is currently stored in it, so

```lua
x = 3
y = x -- y is now 3
```
turns into
```lua
x = 3
y = 3 -- y is now 3
```
Following the same principal, the following code will increment x by 1:
```lua
x = 3
x = x + 1 -- x is now 4
```
(turns into)
```lua
x = 3
x = 3 + 1 -- x is now 4
```
We can use this in our code by making an `x` variable and then changing it periodically.
```lua
player_x = 3

function _draw()
 cls()
 spr(1, player_x, 10)
end
```
Pico-8 provides another function for the update phase of the game-loop called `_update`. This function is run every loop before the `_draw` function, and so allows us to modify any variables we want. Let's modify the `player_x` variable so that our sprite moves to the right:
```lua
player_x = 0

function _update()
 player_x = player_x + 1 -- increment x by 1
end

function _draw()
 cls()
 spr(1, player_x, 10)
end
```

![](assets/moving_face.gif)

Let's take a step back for a moment, and think about how Pico reads this code. What Pico will do is as follows: First, it creates the `player_x` variable, and then stores `0` inside of it. Next, it calls `_update()` and runs the code inside of it, which changes `player_x`. Then, it calls `_draw()` and runs the code inside of that function. Then, it calls `_update()` again. Then, it calls `_draw()` again. Then, it calls `_update()` again... Pico will keep doing this until the end of time (or until you quit the program--whichever comes first). 

Here's a visualization of what Pico does:

![](assets/pf_update_and_draw.apng)

## Values
A value is anything that you can place into a variable. The most basic type is a number. As you saw earlier, there are also several operations that result in numbers:
```lua
3 + 3 -- 6, addition
3 - 3 -- 0, subtraction
3 * 3 -- 9, multiplication
3 / 3 -- 1, division
3 ^ 3 -- 27, to the power of
```
Whenever Pico-8 comes across one of these operations, it replaces the operation with its result, so `3 + 3` becomes `6`.

The other important type is a boolean, which is just `true` or `false`. There are also operations that result in booleans:
```lua
4 == 4 -- true, equal to
4 ~= 4 -- false, not equal to

4 < 4 -- false, less than
4 <= 4 -- true, less than or equal to
4 > 4 -- false, greater than
4 >= 4 -- true, greater than or equal to

true and false -- false
true or false -- true
```
We can combine these to form complex expressions, like

## User input
Before coding a solution to any problem, it's always way more useful to approach it from a high level. So here's a problem: how do we give the user the ability to control the square?
 * Well, we need to make it so that the square only moves when the user presses certain buttons.
 * We need to increment x when the user presses 'right' and decrement x when the user presses 'left'.
 * We need to modify the `_update` function so that it check if 'right' or 'left' were pressed and modifies x accordingly.
But how do we run some code only sometimes? Welcome the `if` statement.
```lua
if true then
 print(1)
end
```
If the boolean between the `if` and `then` is `true` then the up to `end` is run. Otherwise, Pico-8 just skips the whole expression. This becomes very powerful when you combine it with the `btn` function, which returns a boolean depending on whether a button is pressed. We can consult the cheatsheet to find out more:

![](assets/keyboard.png)

As shown on the cheatsheet, `btn` accepts a number (0-6) that denotes a keyboard button. Knowing that, try to write an implementation. Once you're done you can check your solution against ours:

```lua
function _update()
 if btn(1) then
  player_x = player_x + 1
 end
end
```
Now, every frame, Pico checks if the right button is pressed, and if so increases the x. Here is the complete code with the other direction added:

```lua
player_x = 0
player_y = 0

function _update()
 if btn(1) then -- right
  player_x = player_x + 1
 elseif btn(0) then -- left
  player_x = player_x - 1
 end
end

function _draw()
 cls()
 spr(1, player_x, player_y)
end
```

Here's a visualization of what happens:

![](assets/pf_if1.apng)

The code under `elseif` will only run if the first `if` statement is false - this makes it useful for grouping multiple conditions that you know can't all be true (e.g. you know that the player can't be moving right and left at the same time). If we had just made another if for moving left, then pressing down both right and left would have caused the player to stand still.

Notice that `elseif` doesn't require a separate `end`, and is sort-of grouped with the if. There is one last type of if and that is else - else is only evaluated if the previous `if`s and `elseif`s were all false. Here is an example to sum this up:

```lua
if false then
 -- this is not run
elseif false then
 -- this is not run
elseif false then
 -- this is not run
else
 -- this is run
end
```

Each `if` block has to have exactly one `if` and then can also optionally have one `else` and any number of `elseifs`.

## Part 2: the map

If you remember, earlier you drew a map where your game would take place. But currently, the background of our game is a blank screen. This is very boring--let's get our map in the game!

To do this, we're going to start drawing our map to the game. We can do this in Pico using the `map` function. Since `map` draws the map onto the screen, we'll add it to the `_draw` function.

```lua
function _draw()
 cls()
 spr(1, player_x, player_y)

 map()
end
```

![](assets/map_no_player.png)

Now, our map appears--but our player disappears! This is because Lua runs code in order. So, it will start by clearing the screen. Then, it will draw the sprite. Then, it will draw the map. But once we draw the map, it covers the player! Since we want the player to cover the map, we'll draw the map first, and then the player.

```lua
function _draw()
 cls()
 map()

 spr(1, player_x, player_y)
end
```

![](assets/map_fixed.png)

## Borders

Our game is fine now. We can move around as we please on this cool island. However, we have an issue: The player can run off the screen!

![](assets/run_off_screen.gif)

Remember how coordinates work in Pico? Here's a quick refresher:

![](assets/coordinate_grid.png)

The top left corner is (0, 0), and going up makes your y-coordinate go down. So, if we go up above the very top of the screen, our why coordinate will go down below 0. So, to prevent you from leaving the screen from the side, let's prevent your y-coordinate below 0.

```lua
function _update()
 -- get input and move the player like before

 if player_y < 0 then -- if the player went too far up
  player_y = 0 -- send the player back to the edge
 end
end
```

However, the player can still exit the map from the right, left, or bottom. Try to fix this yourself, remembering that the screen is 128 pixels wide by 128 pixels tall.

One solution might be this:

```lua
function _update()
 if player_x < 0 then
  player_x = 0
 elseif player_x > 128 then
  player_x = 128
 end

 if player_y < 0 then
  player_y = 0
 elseif player_y > 128 then
  player_y = 128
 end
end
```

However, this code has a very subtle flaw...

![](assets/half_off_one.gif)

The player can actually go off-screen. The reason that this happens has to due with how coordinates in Pico work. Let's look at what happens when we move the player off the right edge of the screen:

![](assets/square_edge.gif)

Because the coordinates refer to the left hand corner checking the bottom or right edge of the screen isn't actually going to work because the upper left hand corner can be greater than them while the sprite is still off-screen. Instead, what you need to do is to draw an imaginary line that equals the edge minus the square size and check that instead:

![](assets/square_edge_line.png)

Here is the updated code:

```lua
width = 10 -- notice the new variables
height = 10 -- these are important so that it's easier to change the square size later on

function _update()
 if player_x < 0 then
  player_x = 0
 elseif player_x > 128 - width then
  player_x = 128 - width
 end

 if player_y < 0 then
  player_y = 0
 elseif player_y > 128 - height then
  player_y = 128 - height
 end
end
```
