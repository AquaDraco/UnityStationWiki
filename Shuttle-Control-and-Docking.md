## Intro
Unitystation, will implement a seperate matrix for shuttles and other seperate-from-station objects.
This means that those objects can move on their own, while keeping the relative position of objects in on it.

## Shuttle control In system
Controllable shuttles, can be controlled by entering the pilots chair and using the navigational computer. They will be controllable by the controls that are usually used to move the player.

## Moving to another system
Moving to another system, a expedition shuttle for example, happens by setting the destination in the navigational computer, moving far enough from the station, preparing jump (click) and clicking jump-now

Jump will seem like a bright flash on the outside and the inside matrix will be moved to a "In transit" scene and jump out to the destination scene with the same bright flash afterwards.

if obstructed it will jump out on another non-obstructed jumpout point.

## Landing
Landing will happen by moving the shuttle to the planet position, preparing-landing and clicking "Initiate Landing". with a firey flash the shuttle will disappear. for insiders a "deorbit" scene will be displayed after which the shuttle will land on the surface.

Prior to shuttle land (at the same time of the deorbit scene on the shuttle) the landing spot will darken, as if it is a shadow. Anything there will be obliterated

## Docking
Docking will be done by "Magnetic Docking Clamps" on both station and shuttle. When both are enabled and the shuttle's docking clamps are in a straight line with those of the station, the shuttle will slowly drift towards those of the station and clamp itself. This will also automagicaly create an airtight seal between the two objects.

## Cargo Shuttle jump with people onboard
The cargoshuttle has no "In Jump" lifesupport, so all people inside will overdose with exotic radiation within minutes, no mater what they wear, are, or inject.

## Shuttle damage
When something hits a shuttle it will get damaged, when the docking clamps are damaged, disabled or gone automated shuttles (cargo, emergency) will hover a few tiles away from the station and will require a spacewalk. Shuttles that are player controlled can, of course, get close to the docking port, but will not latch on.

Automated shuttles will, however, be very slow in movement and not stop for any object in their path. 

When the cargo shuttle is damaged and sent back to centcomm, it will take longer before it can return. But will completely be repaired when returning.

## Shuttle Hitting Station
The station will have a shield, that prevents against accidental or less accidental collisions. However, with enough force the shield will still fail (or when power is too low), leaving both the object hitting the station and the station itself damaged. The object hitting the station will always be inoperable after hitting it.