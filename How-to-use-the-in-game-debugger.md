In order to facilitate debugging in the game, without the help of the unity editor, we have a slightly modified version of yasirkula's debugger. (https://github.com/yasirkula/UnityIngameDebugConsole). It was designed to show debug log during the game, but you can also run custom commands on the debug console.

# How to use the debugger
Once in game, press F5 and a pop-up window should appear in the lower left corner that shows the number of Debug log entries that have been written since the last time you opened the console.

Click on the icon and it will expand to the top of the screen. You can scroll through the log entries, and click on one of them to expand the details. 

Press F5 again, or the red X in the top right corner 

## Debug commands
yasirkula's debugger only comes with two built in commands, help and sysinfo. 

`help` will return all available commands and their signatures. However, the debugger will collapse log entries by default so you will have to click on the log entry and then the arrow to expand it and scroll down. This will list the all the commands, their custom help message, and what function is actually called.

If the function has arguments, then they are space delimited. For instance, `damage-self` takes three arguments, a body part, the amount of burn damage, and the amount of burn damage. It would be invoked like this

`damage-self Head 50 30`










