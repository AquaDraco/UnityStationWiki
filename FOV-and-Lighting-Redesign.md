On the roadmap past 0.3 you will find the goal to redesign lighting and FOV.
Our previous attempt at lighting was to heavy, to messy and not really adaptable, also that plugin (Light2D) is an unsupported mess.

For FOV we want to start using the following algorithm:
https://www.redblobgames.com/articles/visibility/

For lighting we would want to use the same thing, but in inverse:
Raycasting from the lightsource to detect where it reaches, in a efficient maner.

In regards to processing FOV and Lighting, one can think of merging them into one "Visibility' layer, or stack FOV ontop of lighting.