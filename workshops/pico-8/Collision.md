## Big maps, small screens

Now, we still have an issue: The map is big. So big, the screen can't fit it all at once! 

The player can try moving off the screen, but it's no help: The player runs off the screen to a different part of the map, but Pico doesn't follow him, and we only see the starting area.

![](assets/running_off.gif)

So, we need to tell Pico to run along with the player once they go to another part of the map. We can do this using the `camera` function. 

Pico has a little camera which takes pictures of what it sees, and then reports it back to us. At the start of the game, the camera is at the top left corner of the map, so it only sees the tiny portion of the map that can fit on the screen. Since our code only tells the player to move, and never moves the camera, we don't follow the player around!

So, we need to move the camera around with the player. One way of doing it might be to tell the camera to move with the player, by adding this to the end of our `_update` function

```lua
function _update()
 camera(player_x, player_y) -- move the camera to (player_x, player_y), the same place the player is
end
```

But this doesn't end up looking very nice...

![](assets/bad_camera.gif)

The issue is the camera is in the same spot as the top left of the player. And since the camera makes the pixel it occupies become the top left corner of the picture it takes, the player will always be in the top left. 

We can fix this in multiple ways: We could try to offset the camera a little so that the player ends up in the center of the screen (how Mario does it), or make it so the camera stays mostly still, but 'jumps' everytime the player leaves the screen (like Zelda). We'll be implementing the second approach in this workshop, but feel free to try implementing the first one if you want to. 

## Fixing the camera

Let's divide the map into a number of screen sized 'rooms.' We want the camera to display the 'room' the player is currently in, like so:

![](assets/camera_goal.gif)

To achieve this, we'll need to do two things:
 * Determine the top left corner of the room the player is in
 * Move the camera to that top left corner
 
So, let's try and figure out the top left corner of the room the player is in.

To do this, let's look at a few examples. Remember that the screen is 128 by 128 pixels, and our rooms are also going to be 128 by 128 pixels.

The starting room the player is in has a top left corner of `(0, 0).` If the player moves 128 pixels to the right of that point, to get to `(128, 0),` they will run into the second room. So, the top left corner of this second room is `(128, 0).` However, if the player moves to `(134, 0)` they will still be in the second room. To deal with this, I will start by dividing the x-coordinate by 128 like so:

```lua
function _update()
 roomx = flr(player_x / 128)
end
```

The `flr` function stands for floor, and just means to round down to the nearest whole number: So `flr(14.32)` is 14, and `flr(4213.999)` is 4213. Why might this be useful? Because it tells us how many times 128 fits into the x-coordinate of the player. So, `flr(134 / 128)` and `flr(128 / 128)` both give back 1, indicating that 128 goes into both 134 and 128 only once. 

Then, we multiply this number by 128.
```lua
function _update()
 roomx = flr(player_x / 128)
 roomx = roomx * 128
end
```
Thus, if you're at an x-coordinate of 134, the first line will set `roomx` to 1, and the next line will set `roomx` to 128. More generally, the first line of code will find out how many rooms to the right you've gone, and then the next line of code will multiply that number by 128, since each room is 128 pixels wide. This will help us find the x coordinate of the top left corner of the room: First, we find which room we're talking about by doing `flr(player_x / 128),` and then we multiply by 128 since each room is 128 pixels wide.

We can do something similar to find the y-coordinate (try and figure it out yourself before checking our code) and then set the camera to this `(roomx, roomy)` coordinate.

```lua
function _update()
 roomx = flr(player_x / 128)
 roomx = roomx * 128
 
 roomy = flr(player_y / 128)
 roomy = roomy * 128
 
 camera(roomx, roomy)
end
```

Now, we will have the camera system we wanted!


## Editor tabs

move all your chicken functions to one tab so it's easier to read pls

TODO: this

## Borders

Our game is feeling lively now: There's chickens, a player, and great island. Except, the player currently can leave the island and enter an infinite void of darkness...

![](assets/depressing_void.gif)

Why is this happening? Well, our map is only so big. When I moved the player above the top of the map, the game had no idea what that area looks like, so it drew nothing. We'll need to prevent the player from leaving the map.

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

However, the player can still exit the map from the right, left, or bottom. Try to fix this yourself, remembering that the map is 1024 pixels wide by 512 pixels tall (if you didn't use all of the map, you might want to make these bounds smaller).

One solution might be this:

```lua
function _update()
 if player_x < 0 then
  player_x = 0
 elseif player_x > 1024 then
  player_x = 1024
 end

 if player_y < 0 then
  player_y = 0
 elseif player_y > 512 then
  player_y = 512
 end
end
```

However, this code has a very subtle flaw...

![](assets/half_off.gif)

Remember that `(player_x, player_y)` is just the top left corner of the player. So, this code will only check if the player's top left corner is good, not the entire player. This works fine for the top and left sides of the screen, but not the bottom or right sides since it's possible for the bottom pixels of the player to be past the line without this code picking up on it.

So, we shouldn't check if `player_y > 512.` We should check if `player_y + 8 > 512` (since the player is 8 pixels wide), meaning we want `player_y > 504.` So, we can change our code like so:

```lua
function _update()
 if player_x < 0 then
  player_x = 0
 elseif player_x > 1016 then
  player_x = 1016
 end

 if player_y < 0 then
  player_y = 0
 elseif player_y > 504 then
  player_y = 504
 end
end
```

Now it works!

However, we've got an awful lot of chickens that can still move into the void. The whole point of the last few sections was to get a lot of chickens with very little code--so having to type this out for each chicken would waste all our old work!

Let's think about how we got so many chickens with very little code. We used functions and `foreach` to do the gruntwork for us.

So, let's do that again. I'll make a function `bound` which takes in a chicken, checks if it's inside the boundaries, and moves it inside if it's not. It's almost exactly the same as the code above:

```lua
function bound(character) 
 if character.x < 0 then
  character.x = 0
 elseif character.x > 1016 then
  player_x = 1016
 end

 if character.y < 0 then
  character.y = 0
 elseif character.y > 504 then
  character.y = 504
 end
end
```

Now, we just use `foreach` like before:

```lua
function _update()
 -- move the chickens
 
 foreach(chickens, bound)
end
```

## Making the player a table

We use tables to group related variables. `player_x` and `player_y` are related, so why not group them? 

```lua
player = {}
player.x = 0 -- this will be the starting position of the player; feel free to change
player.y = 0
```

The advantage to doing this is now our `bound` function works with the player. This is because `bound` **doesn't care what type of table (chicken or player) you give it, it just cares if that table has an `x` and a `y` variable.** This means that we can use `bound` on the player, too!

```lua
function _update()
 -- move the player and chickens
 
 bound(player) -- bound the player
 foreach(chickens, bound) -- bound each chicken
end
```

## Making solid tiles: Flags

Right now, our player can walk on water (in your games, they may be able to walk through walls or on something else they shouldn't be able to). We'd like to stop this, but first we need to tell Pico which things the player can and can't walk on. 

We can do this by going back to the sprite editor and setting **flags** on our sprites.

![](assets/sprite_flags.png)

Those eight little buttons next to the sprite are all flags. By default, they all start as off and we can turn them on by clicking them.

To help Pico know which things we can and can't walk on, let's use flags: Flag 0, the leftmost flag, will represent if the tile can be walked on or not. If the flag is off, the player can walk on it, and if the flag is on, the player cannot walk on it.

In my game, I'll turn on flag 1 for my water sprite, and leave it off for the rest of them.

![](assets/toggling_flag.gif)

Now that we've set the flags, we can start programming our collision detection.

## Collision detection: Planning

Let's make a high level plan for how our collision detection should work. Say we want to move an object to some point (x, y). Before moving it, we should:

* Check if (x, y) is clear--nothing (like ocean) blocking it
* If it is, move them there
* Otherwise, don't move them

Right now, I'm only going to check if (x, y) is on a part of the map we don't want to touch, like water. Later, when we implement enemies, we'll talk about how to check if (x, y) is inside of the player or a chicken. 

I'd also like my collision detection to be in it's own general function, just like `bound` was. I'll call this function `move_to`. It will take a point and an actor, and move the actor to the point if it can.

## Collision detection, now with code!

Alright, so now let's program our `move_to` function. 

First, we need to figure out if `(x, y)` is a point we can move on or not. For this, we're going to use two functions: `mget` and `fget`. `mget` will return which type of sprite (remember sprite numbers?) is at a point on the map, and `fget` will help us figure out which flags are on or off in a sprite if we give it a sprite number. So, we use `mget` to find which sprite (x, y) is, and then use `fget` to check if that sprite is solid or not (remember how we used flag 0 to tell Pico which tiles were solid?)

So, we might try this:

```lua
function move_to(x, y, actor) -- actor could be a player, a chicken, or something else we add in the future
 sprite = mget(x, y)
 solid = fget(sprite, 0)
end
```

`solid = fget(sprite, 0)` will return `true` if flag 0 is turned on, and `false` if it is turn off. However, this code has a very subtle flaw--`(x, y)` returns to coordinates in terms of pixels. If I give you the point `(124, 312)` it means to go 124 pixels right, and then 312 pixels down. But `mget` takes map coordinates, which mean you should 124 sprites right, and then 312 sprites down--to a much different point!

In our map editor, we can view the map coordinates of each sprite at the bottom of the screen:

![](assets/map_coordinates.png)

Hmm. So, we need to convert `(x, y)` from pixel coordinates to map coordinates. How should we do this?

Let's look at an example. The pixels at `(0, 0), (4, 7), (7, 3)` are all inside the top left sprite. This is because they each tell us to move an amount of pixels right that is smaller than 8 and an amount of pixels down smaller than 8. To pass into a new tile, we'd need to move at least 8 pixels. This is key to our solution: To convert, we use `flr(x / 8)` and `flr(y / 8)`. (Remember that `flr` just means round down.) This is because `flr(x / 8)` will tell us how many groups of 8 pixels we've moved right--in other words, how many tiles right we've moved. So, we can now fix our code:

```lua
function move_to(x, y, actor)
 sprite = mget(flr(x / 8), flr(y / 8))
 solid = fget(sprite, 0)
end
```

Now, we should only move the actor if the sprite is not solid.

```lua
function move_to(x, y, actor)
 sprite = mget(flr(x / 8), flr(y / 8))
 solid = fget(sprite, 0)
 
 if not(solid) then
  -- (x, y) is fine to move too
  actor.x = x
  actor.y = y
 end
end
```

Note the `not(solid)` in the `if` statement. `not` is a function which returns the opposite of a `true` or `false` value-- so `not(true)` would be `false` and `not(false)` would be true. 

Now, we slightly tweak how movement works. For the player, our current code looks as so:
```lua
function _update()
 if btn(0) then
  player.x = player.x - 1
 end
 
 if btn(1) then
  player.x = player.x + 1
 end
 
 if btn(2) then
  player.y = player.y - 1
 end
 
 if btn(3) then
  player.y = player.y + 1
 end
end
```

Instead of doing that, I'll store a potential position, and then use `move_to` to check.

```lua
function_update()
 potentialx = player.x
 potentialy = player.y
 
 if btn(0) then
  potentialx = potentialx - 1
 end
 
 if btn(1) then
  potentialx = potentialx + 1
 end
 
 if btn(2) then
  potentialy = potentialy - 1
 end
 
 if btn(3) then
  potentialy = potentialy + 1
 end
 
 move_to(potentialx, potentialy, player)
end
```

Done! Now, the chickens. Their movement happens in a the `move_chicken` function. Try modifying that function to use `move_to` instead of moving the chicken directly.

## Let's go down!

Our collision code is done! Let's play:

![](assets/broken_collisions.gif)

Shoot. It only works half the time?

Let's think about what's happening. Remember that `player.x` and `player.y`refer to the **top left corner** of the player. So, when we say `move_to(x, y, player)` we only check if the top left corner of the player will go to a valid spot--not if every part of the player will!

To fix this, I'm going to first add a second function, `on_solid`, which checks if a point is on a solid sprite:

```lua
function on_solid(x, y)
 sprite = mget(flr(x / 8), flr(y / 8))
 solid = fget(sprite, 0)
 return solid -- give back solid to whoever uses this function
end
```

Now, I'll modify `move_to` a little. Instead of just checking `(x, y)` I will have it also check the other three corners of the actor. 

Wait, where are the other three corners of the actor, if the top left corner is `(x, y)`? Remembering that the sprites in Pico are 8 pixels by 8 pixels, it might be temping to say those other corners are at `(x + 8, y), (x + 8, y + 8),` and `(x, y + 8).` 
This is wrong for a subtle reason. Let's implement the rest of our collision code, then see why:

```lua
function move_to(x, y, actor)
 corner1 = on_solid(x, y)
 corner2 = on_solid(x + 8, y)
 corner3 = on_solid(x + 8, y + 8)
 corner3 = on_solid(x, y + 8)
 
 if not(corner1) and not(corner2) and not(corner3) and not(corner4) then
  actor.x = x
  actor.y = y
 end
end
```

The first four lines will check if each corner is solid, and store the answer in the variables `corner1` through `corner4`. Then, we check that *each corner* is not solid before moving. (Question to you: Why can I just check corners instead of having to check every single pixel of the player?)

Let's test this out:

![](assets/almost_collision.gif)

Uh oh! I can't go to the edge, just one pixel away from it. My corner locations were wrong--`(x + 8, y)` isn't actually the top right corner. To see this, note that `(x, y)` is the top left pixel of my player. So, `(x + 8, y)` is the pixel 8 pixels away from that pixel. This means that a sprite with corners at `(x, y)` and `(x + 8, y)` would be 9 pixels wide, not 8. We have 1 pixel from `(x, y)`, plus 8 more since `(x + 8, y)` is 8 pixels away, giving us 9 pixels total. So, we'd actually want to use `(x + 7, y)`. Similary we change all the 8's to 7's to get the desired effect.

Finally! Right?

## Let's go up and left!

Our code still has a small problem. Say we want to go both up and left at the same time. This is fine. But the tile to our upper left is solid, so we can't walk there--no biggy, `move_to` just won't move us. But what if the tile to our immediate left is fine? Instead of moving left, we just won't move at all. I'd like pressing left to move me left whenever possible. 

So, I'll change my collision code one last time. Instead of checking if it can move me on the x-axis and the y-axis at the same time, I'll have it first check the x-axis and then check the y-axis.

```lua
function move_to1(x, y, actor)
 corner1 = on_solid(x, y)
 corner2 = on_solid(x + 8, y)
 corner3 = on_solid(x + 8, y + 8)
 corner3 = on_solid(x, y + 8)
 
 if not(corner1) and not(corner2) and not(corner3) and not(corner4) then
  actor.x = x
  actor.y = y
 end
end

function move_to2(x, y, actor)
 move_to_partial(x, actor.y, actor)
 move_to_partial(actor.x, y, actor)
end
```

Let's see what I did here. `move_to1` is just our old `move_to` function. The new `move_to2` function will first move the actor on the x-axis using our old code, and then move them on the y-axis using our old code. This way, we can choose whether or not we want the `move_to1` style movement or the `move_to2` style movement (and it also made writing `move_to2` really quick!). I prefer using `move_to2` but feel free to use what fits best with your game. 

## Fixing chicken spawns

Chickens can still spawn on water... shit.

have `make_chickens` check if the tile it wants to spawn on is good.

## Enemies: A plan

We have friendly chickens, so why not add some unfriendly chickens?

<image of bad chicken>
  
My goal for these unfriendly chickens is the following:

* The squares randomly walk around the map until they're in the same room as the player
* If they're in the same room as the player, they move towards the player
* If they touch the player, they do a little damage to the player and then explode

Here's an example of what I want to happen:

<gif of chicken attack>

## Tables of enemies

'member chickens? that, but with enemies

## Chasing the player

first version:

```lua
function charge_player(enemy)
	if player.x > enemy.x then
		enemy.dirx = 1
	elseif player.x < enemy.x then
	 enemy.dirx = -1
	else
	 enemy.dirx = 0
	end
	
	if player.y > enemy.y then
		enemy.diry = 1
	elseif player.y < enemy.y then
	 enemy.diry = -1
	else
	 enemy.diry = 0
	end
end
```

but oh no we can get chickens stuck shit

<image of stuck chicken>

hmm how can this happen

well when the chicken has the right x-coordinate but moving y-coordinate causes him to move into obstacle (or other way)

let's fix this by telling the chicken to just move until there's no more obstacle  
  
```lua
if enemy.dirx == 0 and enemy.y != py then
 enemy.stuckx = true
 move_to(px - 1, py, enemy)
else
 enemy.stuckx = false
end
	
if enemy.diry == 0 and enemy.x != px then 
 enemy.stucky = true
 move_to(px, py - 1, enemy)
else
 enemy.stucky = false
end
```

and then

```lua
function charge_player()
 -- previous stuff
 
 if enemy.stuckx then
	 enemy.dirx = 0
 end
  
 if enemy.stucky then
	enemy.diry = 0
 end
end
```

simple, good enough... BUT SHIT

what if he can't go left no more... poor little chicken gets stuck

change the code to start moving right if left gets us stuck (same for up/down) use the fixx, fixy system


poor little chicko still can't handle corners!!!!!
(think about why our collision code can't handle them)

let's fix it (do stuckcorner thing)

## Exploding

note: 

```lua
function touching(e1, e2)
	xgood = abs(e1.x - e2.x) < 8
	ygood = abs(e1.y - e2.y) < 8
	return xgood and ygood
end
```

is probably an easy to understand collision function

## Health

## Health bar

## Flags, again: Spikes and lava
