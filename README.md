# Metin2 Stone-Farm Bot
A bot written in Python using image processing to auto-farm Metin stones. The bot may not be suitable for slow computers as it needs CPU power to process images and detect objects in it. Otherwise it will be painfully slow or may not even work properly.

## How it works
- Logins to your character.
- Teleports to the correct map.
- Rotates the camera looks for a Metin stone.
- Breaks the stone and collect items.
- Checks if the player is killed, stuck, or kicked.

### Notes
- Rubinum seems to have prevented such scripts from sending input to the client. So currently it seems the bot doesn't work anymore.
- The bot may not work for every server especially in its current version. It was created for Rubinum. The client must give you the ability to save your login credentials, your player must have a ring that teleports you to the place you want to farm and a map that is made specifically for stone-breaking. The bot isn't advanced enough to run around searching.
- Configuration for this bot is a bit complex so it may be difficult to understand.
- Recommended game resolution: 1024x768. 800x600 makes the UI elements overlap each other and causes problems on detection. And larger resolutions will increase the time of image processing.
- Your character should have a horse and ride it for the bot to work properly.
- The bot can handle multiple clients at the same time.
- The bot uses libraries that only work on Windows.
- The bot is still not finished. However, it may or may not be continued. There are many configurations that are hardcoded and the modularity is poor.

## Configuration
First of all, I need to explain the config files. In order to configure the bot properly, you need to understand how exactly it works.

We have 3 directories: clients, maps and relatives.

### relatives
Note: The current version doesn't support multiple resolutions as they are hardcoded. So you can skip the current guide if you want unless you are going to change the code yourself.

This file contains some screen coordinates for specific resolutions. If you want to use one of the two that are already there you can leave the file as it is. This configuration basically creates an example of a hypothetical client that is going to be used to calculate the relative position on your screen.

For example: If our hypothetical client's top left position was in (895, 270) position, where would the mini-map be? Let's say it's on (1903, 286) Where would the the stone hp be? Let's say it's on (1408, 293). So if your actual top left position is on (862, 315) the program can guess where the these positions are by finding the relation of them. This is done by the following code:

### bmath.py

    def get_relative(top_left, new_top_left, def_pos):
        dif = (def_pos[0] - top_left[0], def_pos[1] - top_left[1])

        pos = (new_top_left[0] + dif[0], new_top_left[1] + dif[1])

        return pos
        
- `top_left` is the config's file top left position of the client. It is used to find all relative positions.  
- `new_top_left` is the top left position of your actual client.  
- `def_pos` is the configs position we want to find the relative one.  
- `[0]` is the x position on screen and `[1]` is y. So by writing `new_top_left[0]` we mean the x value of your client's top left position.

So, to find the position of the mini-map, we have:

    top_left = (895, 270)  
    new_top_left = (862, 315)  
    def_position = (1903, 286)

Replacing the values:

    dif = (1903 - 895, 286 - 270) = (1008, 16)
    pos = (862 + 1008, 315 + 16) = (1870, 331)

So in this example, the mini-map's position would be at (1870, 331). This applies to all positions that are included in this configuration file.


### Create your own config of relatives

Now that we know how it works, let's see how to create our own config file for other resolutions.

First of all, you need to take a couple of screenshots. One of your login screen, one of your game screen while having a mob or a stone as a target so the health bar at the top is visible and one while being dead. You also need an image editor or any other software that you can find cordinates by pointing your cursor in the image. [paint.net](https://www.getpaint.net/index.html) is good enough and also free. Now let's explain some things:

- `top-left` is of course your screen's top left position. Not the window but the very top left pixel of your game screen. It is recommended to get it right so it doesn't cause miscalculations.
- `window-size` is your client's window size. Basically the game resolution however, the height should be the without including the bottom bar. Give it some extra 50-100 pixels offset as well because you won't need that part of your screen the image process. See the already existing values to get an idea.
- `stone-bar-close` is the x button on the health bar. This is basically the main issue with the 800x600 resolution as it may overlap with the event buttons next to it.
- `mini-map` is the button that closes the minimap at the top right of your screen.

For the stone hp the bot creates a rectangle around the hp value and detects the text inside it. So in order to take the value we need a top left position and the rectangle size:
- `hp` is the top left position.
- `hp-rectangle` is the rectangle size. This may not be necessary to change it.

The same method applies here. The bot detects if the player died by trying to find the revive here button.
- `revive-here` is the top left position of the revive button.
- `revive-rectangle` is the rectangle size.

- `account1-12` is the saved accounts on the login screen. If the client doesn't support saved accounts you may put some safe value that will not affect anything on click. The bot, by the way, will get stuck.
- `channel1-8` is for channel selection. Add whateer values if the channels are less. Add the emptier channels if the channels are more.

The following ones are for Rubinum2 only. They are used on map configuration to set the navigation of that specific map through the menu. If you can make it work somehow on other servers, good.
- `next` is the next page option.
- `ork` and `red-forest` are map names. Self explanatory.
- `area1-8` are specific areas on each map.

### maps
This file contains map settings. This file is much simpler.

- `name` is the name of the map. This is used to set which map a specific client will go to search for stones. See clients.
- `stone-sample` contains the directory of stone pictures which will be used to detect the Metin stone you want to search for. You can add to that directory as many samples as you want. In the current version, you have to screenshot the name of the stone and not the texture. If you want to detect the texture, make sure to get only small samples for more accuracy. Look the content of the `icons` folder to get the idea. Furthermore, make sure to change the code accordingly: open `actions.py` and find the function `select_target`. There's a line `loc = (loc[0], loc[1] + 80)`. 80 is the offset of the click downwards. If you're using the name as the target keep it as it is or change it to a different offset if 80 doesn't work on you. Put 0 if you are using textures as samples. Note: The sub-folders that exist in the folder are not being used. You can use them to store more samples while not using them. At the current version, you cannot use the name in one case and texture in another.
- `object-detection` currently doesn't work. However, it would be used to change the object detection method in case one doesn't work. This is a bit of an advanced option so keep it as it.
- `hl` basically means high-low. Some object detection methods give high values to best matches and some the opposite. So this says the bot which of these cases to look for. Advanced option.
- `threshold` is a threshold that helps the bot determine if the best match of the detection is actually the object being searched. Again, advanced option. Leave it as it is.

- `navigation` is the clicks you have to do to navigate to the specific map. You can add as many as you want and also name them whatever you want. It doesn't matter.

### clients
This is the config files for the clients you want to use. Each file is used for a different client.

- `account` includes some settings for each account. If enabled is fales the bot ignores this client. Id is the client's position on the saved logins. channel is the channel you want to login. username is only used in the console to let you know which client is being used at the moment. map is the map you want to go.
- `position` is used to tell the bot where your player is being placed in character selection. Currently doesn't work so make sure it's at the front.
- `navigation` has some settings to help your bot navigate through your client. task-bar is the position of the client's window on the Windows task bar. Make sure to not combine the same windows into one in the task bar settings. top-left is your clients top left position. Use the guide at the relatives to see how to get it (Create your own config of relatives). ring is the position of the map menu at the bottom bar. Supported values (`1`, `2`, `3`, `4`, `f1`, `f2`, `f3`, `f4`).
- `skills` is the same as ring but for skills. Add the skills like Aura of Sword or buffs or whatever that you need to have your character prepared. Add as many as you want. Also, the name doesn't matter. It's just an identifier.
- `horse-skills` is, of course, for horse skills. Currently only slash is supported that is used to unstuck your character if they're being surrounded.

## Run

### Requirements
- You need to install Python 3.7 or newer.
- Open cmd and using `pip`, install the following libraries:
 
    - pillow
    - cv2
    - opencv-python
    - pyautogui
    - numpy
    - keyboard
    - imutils
    - pytesseract

### Start the bot
Before you run, make sure you you have opened the clients you want to use in their login screens. Run the bot by typing `python main.py`. After that, don't touch your keyboard and don't click anywhere. Do it only if you want to stop the bot. To do so, quickly open the cmd window and press `Ctrl` + `C`.

The bot will run for 5000 times in total. That means each time it catches a player farming, it counts +1. In `main.py` change the `max_loops` value if you want to change the value or replace the `while loop < max_loops:` line with `while True:` to run the program infinitely.

There are also two extra scripts:
- `cords.py` which you can use to click anywhere on your screen and get the coordinates. It uses only the `pyautogui` library.
- `xp.py` is a very simple bot that automates xp farming by automatically pressing `space` and every 10 seconds `2` to activate a cape. I uses only the `keyboard` library. You can change the button if you want inside the code:
    - `binput.press_button('2')` is to press the cape. Supported values (`1`, `2`, `3`, `4`, `f1`, `f2`, `f3`, `f4`).
    - `sleep(10)` is the frequency for each cape in seconds.
    - `task_bar = (1291, 1062)` is the position of your client in the task bar. It is used for the bot to bring the window at the top.
