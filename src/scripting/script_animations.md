# Scripted animations

____

## Theory

The first thing to know is that animations can be divided into a few categories:

### Motions

* They represent actual hands, NPCs or actor movements
* Stored in .omf files
* Unique to an object

In this article we’ll focus on hands animations which are, for example, sprinting or weapons reloading. FDDA items animations are also motions. Motions are usually unique to a specific object. For example, AK74 reload animation will only look nice when used with AK74 or any other gun that is set up for this specific motion. That means you can still play it in your script with empty hands, but if you play it while holding a PKM the machine gun won’t play any reload animation at all.

Why so many details, you may wonder? Point is, a motion can have interesting use cases. For example, you can play a pistol holstering animation to temporarily hide one or both hands. Or you can use it for gestures as well as other unarmed animations.

Also, an .omf file itself is not a motion. .omf files usually store multiple motions. If you want to add a new motion, you’ll need to either

1. put your motion into an existing .omf file or
2. make a new .omf file that will store your motions.

### .anm animations

In turn, these can be divided into two subcategories:

#### Camera animations

These are much simpler due to their nature. Camera animations represent a screen effect - for example, leaning when an actor equips a helmet or a suit, ‘kickback’ effect when enhanced recoil effects are enabled, camera animations when reloading (mostly used by weapon mods like Boomsticks and Sharpsticks; not to be confused with hands motions) etc.

#### ‘Universal’ animations

This is not an official name, of course, it’s just something I came up with to distinguish these from the simple screen effects. I’d better describe them with an example.

In Anomaly, there’s a ‘misfire’ animation. Try to shoot a gun without any ammo, and you’ll see slight hands movement. This is ‘misfire.anm’ animation.

Point is, this is not exactly a screen effect type of .anm file, and yet this is not an .omf-type motion. It can be used with any object the actor is holding, i.e. it’s not tied to a specific gun.

So technically, all the .anm files are the same and are played with the same function. I just decided there needs to be some sort of distinction.

.anm are separate files unlike motions from .omf files, and every single .anm file can be just placed into anims/ folder in gamedata and used directly.

### .ppe effects

These are camera animations with more possibilities: besides camera movements, they can also include, for example, duality effect (when drinking vodka or when there was an explosion next to the actor), grain effect in radiation zones etc. Vanilla night vision is also a .ppe effect.

.ppe files work similarly to .anm files - they’re separate and go to anims/ folder as well.

____

## Practice

Now let’s have a look at what we need to play animations in scripts.

### Motions

To play a motion, we need a few components:

1. An [.ltx](../main-folders-and-files/file-formats/ltx.md) file containing the basic parameters of the motion we’re going to use.
2. A script function to play it.

#### Setting up motion parameters

1. Create a config [.ltx](../main-folders-and-files/file-formats/ltx.md) file in **configs/items/items** and give it any name, but make sure it starts with **items_**. Example: **items_my_anims.ltx**.
2. Set the parameters. We’ll examine them by looking at this example.

```ini

_[anim_my_sec]_

    _hands_position              = 0.033, -0.155, 0.1_

    _hands_orientation           = 0, 0, 0_

    _hands_position_16x9         = 0.033, -0.155, 0.1_

    _hands_orientation_16x9      = 0, 0, 0_

    _anm_my_anim                 = name_of_motion_that_will_be_played_

```

**[anim_my_sec]** is an animation section name. It’ll be used in the script.

**hands_position**, **hands_orientation**, **hands_position_16x9** and **hands_orientation_16x9** are hands position/orientation parameters. They work similarly to weapon position parameters.

**anm_my_anim** is a motion name. From what I saw, it should always be started with **anm_** in such a config. Not to be confused with an .omf file name. For example, if your .omf file is called **my_new_file.omf** and it stores a motion called **my_motion_1**, specify **my_motion_1_** here.

To make a test placeholder, you can also open any weapon config file and choose any of the motions specified there (like, _anm_ak74_reload_, \_anm_svd_sprint _etc.)

#### Script function

The function that lets you play a motion is:

```lua

_game.play_hud_motion(hands, anim_section_name, motion_name, restart, speed)_

```

Now let’s have a closer look at its arguments.

1. (int) **hands**
    1. Specifies what hands will be used to play an animation. 1 - left hand only, 0 - right hand only, 2 - both hands.
    2. Note that you still can’t play a left-handed (like detector draw) animation for the right hand. But you still can play any two-handed animation with only one of the hands.
2. (string) **anim_section_name**
    3. This is the name of the animation section specified in the config file.
    4. In our example this is **[anim_my_sec]**.
3. (string) **motion_name**
    5. This is the name of the motion within the specified section in the config file.
    6. In our example this is **anm_my_anim**.
4. (bool) **restart**
    7. Specified whether or not an animation can be restarted after it was finished, or it should only play once. Can be True or False.
    8. Note that setting it to \_false _with a looped animation may not work. It must be stopped manually - we’ll describe it later.
    9. This one is a bit unclear - it’s missing from lua_help.script, and I’m not sure if I understood how it works correctly.
5. (float) **speed**
    10. Sets the playback speed of an animation. Example: 0.5, 1,2, 5.5 etc.

#### Supplementary functions

```lua

_game.hud_motion_allowed()_

```

Checks if a hands motion can be played at the moment. Useful as an additional safety check. Returns True or False.

```lua

_game.get_motion_length(anim_section_name, motion_name, speed)_

```

Gets the specified motion length. The parameters are identical to those in `_play_hud_motion_`.

Note that it returns time in milliseconds, so in order to get the actual length you’ll need to divide the result by 1000. Useful to know when the motion is over.

```lua

_game.stop_hud_motion()_

```

It will stop any hands motion that is currently playing (since it can’t accept a specific motion as an argument), not only the one you’re playing in your script.

#### Things to consider

1. It’s safer to always stop the looped animations explicitly. This can be done by:
    1. playing a non-looped and non-restart animation - the previous one will be stopped.
    2. calling `_game.stop_hud_motion()_`.
2. Consider adding more protection from breaking something.
    1. Know when to stop an animation and how not to make it looped when it’s not wanted. Remember to check the necessary conditions to make sure it makes sense to play it at all at the current moment.
    2. Know the callbacks you’re using to play animations. This may affect the animation behavior. Incorrect callback usage may lead to unwanted results, softlocks or sometimes even crashes.
    3. Call `_game.only_allow_movekeys(true)_` if you don’t want the user to break an animation by pressing any action keys (like item quick use), and then restore the controls with `_game.only_allow_movekeys(false)_`.
    4. `_hide_hud_inventory()_` and `_db.actor:activate_slot(0)_` can be used to close the backpack inventory or force hide an active weapon if needed. Detectors can be manipulated in a similar way with:

```lua

    _local det = db.actor:active_detector() or nil_


    _det:switch_state(2)_


    and then restored with _det:switch_state(1)_.

```

### .anm animations

It’s simpler to play these. Here we don’t need any configs. The files are played directly. The function is `_game.play_hud_anm(path, hands, speed, power, looped, restart)_`.

In this case I assume the played animation is “misfire.anm” mentioned earlier - I used this function for it in my addon. You can try it with different animations.

The arguments are:

1. (string) **path**
    1. Path to the .anm file.
    2. It is predefined that a file is in the **gamedata/anims** folder. That being said, the path should look like this, for example: `_"camera_effects\\actor_move\\strafe_left.anm"_`.
    3. The real path is `_gamedata/anims/camera_effects/actor_move/strafe_left.anm._`
    4. If the file is in the root **anims/** folder, just specify the file name.
2. (int) **hands**
    5. Specifies what hands will be used to play an animation. 1 - left hand only, 0 - right hand only, 2 - both hands.
3. (float) **speed**
    6. Sets the playback speed of an animation. Example: 0.5, 1,2, 5.5 etc.
4. (float) **power**
    7. Sets the ‘amplitude’ of an animation. The more it is, the more noticeable the animation will be. When set too low, it may be too ‘faint’. Example: 0.51, 1,65, 5.1 etc.
5. (bool) **looped**
    8. Defines if an animation will loop until stopped by any means. Can be True or False.
6. (bool) **restart**
    9. Specified whether or not an animation can be restarted after it was finished, or it should only play once. Can be True or False.

#### Supplementary functions

```lua

_game.stop_hud_anm(path, force)_

```

Makes it possible to stop a specific .anm from playing. _path_ is identical to the one above. _force_ is a boolean parameter which makes it possible to force stop it - a bit more ‘harsh’ method.

```lua

_game.stop_all_hud_anms(boolean)_

```

Stops all .anm’s that are currently played.

```lua

_game.set_hud_anm_time(path, time)_

```

#### Things to consider

1. Unlike motions, .anm’s can be stopped directly by calling `_game.stop_hud_anm_`.
2. .anm’s can be ‘layered’ - i.e. it’s possible to play two or more .anm files at the same time. This can provide interesting and good-looking results.
3. However, it may be tricky to play a few .anm’s with the same name at the same time because the function relies on the file name (which is already played in this case). If you want to play the same effect but with different parameters, consider making a copy of the .anm you use, give it another name and call them with two functions with different arguments.

### .ppe effects

These are the functions for them:

1. `_level.add_pp_effector(path,id,looped)_`
2. `_level.set_pp_effector_factor(id,power)_`
3. `_level.remove_pp_effector(id)_`

The first one is used to start a .ppe effect. Arguments:

1. (string) **path**
    1. It works absolutely the same way the **path** argument from `_game.play_hud_anm_` does.
2. (int) **id**
    1. An ID of the effect. Can be any number.
    2. Used for removal of the effect later on.
3. (bool) **looped**
    1. Defines if the effect will loop until stopped by any means. Can be True or False.

The second one is used to alter an already existing effect added by `_add_pp_effector_`. Arguments:

1. (int) **id**
    1. The ID of the previously created effect.
2. (float) **power**
    1. Effect intensity factor from 0 to 1.

The third one stops and removes the added effect. The only argument is **id** - use the same one you specified when adding the effect with `_add_pp_effector_`.