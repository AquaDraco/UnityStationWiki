**Quick rundown of how current system works and what changes need to be done to get rid of it:**

Unicloth is a "base" prefab that has no specific item information. We currently have scripts that parse the information for any given item out of a bunch of files and then apply that information to the Unicloth item. The files in question:

[The item hierarchy list](https://github.com/unitystation/unitystation/blob/develop/UnityProject/Assets/Resources/Metadata/hier.txt) this simply lists all available items. It is how the parser knows what information to apply to the unicloth. This string is also used to determine what type of item it is. For example, if the string does not contain "clothing" or "headset" it is NOT applied to the unicloth, as its probably an item or object. (That last part is particularly bad, considering the naming conventions in hiers are NOT consistent).

[The I-don't-know-what-it's-actually-called-but-I-call-it-item-properties list](https://raw.githubusercontent.com/unitystation/unitystation/develop/UnityProject/Assets/Resources/Metadata/dm.json) contains properties to be applied to any specific hier. So things like name, description, various attributes that we don't necessarily use, the icon image string to be used, that sort of thing.

[The image-related list](https://raw.githubusercontent.com/unitystation/unitystation/develop/UnityProject/Assets/Resources/Metadata/dmi.json) matches the icon image string from the 2nd file and specifies things like offset on the spritesheet for the item image on the player character, or in each hand or whatever.

The parser takes the hier you specified to your unicloth and goes through all of the above files to set the correct values for the ItemAttributes component attached to the unicloth. This means that if you already know the parameters, you could just copy the unicloth prefab, set the ItemAttributes manually, and the item would spawn correctly in game! We can use this to have unity generate prefabs automatically, by simply going through all the hiers. It won't be perfect, but it will match MOST things correctly, and we can edit them by hand if we find any issues.

The next issue however, is that players loadouts are currently specified as hiers. So ears slot, load this hier. Chest slot, load that hier, etc. We need to re-write this to accept item prefabs, instead of hier strings. Should not be too hard (famous last words).

The unicloth system would be fine to use as is, however it has one glaring flaw: unity does not support adding components to networked behaviors at runtime. Which means we can't add components to unicloth. (Unless we add the components to ALL unicloth, which doesn't work for us obviously).

Good luck and may whatever diety you believe in have mercy on you.