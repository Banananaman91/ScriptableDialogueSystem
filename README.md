# ScriptableDialogueSystem
A dialogue system using scriptable objects for Unity. This system was originally created for another personal project, and I decided to create a package version so that it can be easily added to new projects if needed and be available for others. Example images given below are related to the uses in that project.

An example scene can be found in ScriptableDialogueSystem > Example > Scene

## Updated 1.1.0

Added UnityEvents to DialogueObjects. Now response triggers will call the events assigned to DialogueObjects and those inherited from DialogueObjects.
This allows users to more easily fill out customisable events in the inspector directly.

## How to add dialogue

Firstly, you need to create a type that will be used to initialise dialogue, such as an interactable. This type will inherit from DialogueObject (ScriptableDialogueSystem > Editor > DialogueTypes). DialogueObject will provide the necessary fields, at which point you create the way you want them to run.
An example has been provided in DialogueExample (ScriptableDialogueSystem > Example > Scripts)

### Render Dialogue
RenderDialogue is a type used to handle rendering all dialogue instances. Multiple render dialogues can exist, with unique UI’s, to give the appearance of changes in dialogue interaction. These need to be attached to the items intended to run them. Items can have a base Page Render and Alternative Render options added. This allows the user to create their own ways of switching the rendered UI for the dialogue through ResponseTriggerEvents or other methods.

![Render Dialogue](https://github.com/Banananaman91/ScriptableDialogueSystem/blob/master/Images/RenderDialogue.PNG?raw=true)

Page Name corresponds to the text field to display the name (if required) of whose dialogue is displaying.
Page Text displays the dialogue itself.
Page Image Position is a GameObject used for its transform point to display the required image (if required).
Sentence speed controls the speed at which text types out (lower is faster)
Option: When displaying dialogue, Responses for the player can be added for decision making. Option is the prefab for the button that will be instantiated for this requirement.
ButtonPositions holds all positions for instantiating the buttons, covering multiple places. This is limited by the UI space.
Dialogue Background refers to the parent image in the UI to be set as the parent of any images that are instantiated.
Dialogue Box is the dialogue UI object used for turning on/off the object. 
NpcImages refers to the scriptableobject NpcImages (more on this later). All ScriptableObjects have been made editable directly in the inspector from any objects that have it added. Basically, if a field for the scriptableobject exists and is filled, all the editable information will also show.
Other Dialogues refers to all other possible render dialogue objects in the scene. These all must be attached here. This is to ensure that when one becomes active, all others are made inactive, preventing two from overlapping.

### Dialogue Objects
These are scriptableobjects for dialogue information that can be created at will. Right click > create > dialogue in order to create a new dialogue object or create > npcimages to create a new images object. When placed into the serializefield of an object looking for one of these objects, they will also be editable in that field in the inspector.

#### Dialogue

![DialogueObject](https://github.com/Banananaman91/ScriptableDialogueSystem/blob/master/Images/Dialogue.PNG?raw=true)

Lets break down what is happening in this dialogue.
The first piece in the Messages array is a dialogue for Death. This contains the name, the message text itself (to be displayed), mood, responses, and next message.

Name identifies who is talking. When used, this will display their name in the name text field of the UI. Not only this, but this will be used in conjunction with npcimages in order to find the corresponding characters images (more on this later).

Message text is displayed in the text UI and is rendered at the sentence speed set by the render dialogue.

Mood is to identify the mood of the text. This is used with npcimages, once the npc is identified it will display the corresponding image to that mood by checking the image name. (example later).

Responses handles filling out possible responses. The size of this array determines how many button responses are created. These will in turn have text to display on those buttons. They have a Next and Trigger event field

Next will decide which array element in messages will run next, and can jump to any array element chosen.

Trigger Event runs the ResponseTrigger method existing in any inheritors of DialogueObject. These include Cage, Furniture, Mastermind and Page. This allows event behaviour to be triggered by dialogue choices, starting up mini games or initiating final locations, or any other behaviour added in if needed (animations etc)

NextMessage, the same as Next in Responses, signifies which message is to be played next. Both NextMessage and Next should be set to -1 (or any value less than zero) in order to cancel running further dialogue, ending dialogue completely. If responses are added, recommended to set NextMessage to -1 to end dialogue if anything but a response is pressed, prevent dialogue continuing without a response chosen. Alternatively, add a default responses Next to also be NextMessage (make them the same value).

#### NpcImages
NpcImages is a singular scriptableobject. More can be created, but aren’t required. This handles storing all images (if needed) to be displayed in the dialogue UI.

Npc Image array handles all separate images. This works by splitting the fields up into categories.

![NpcImages](https://github.com/Banananaman91/ScriptableDialogueSystem/blob/master/Images/NpcImages.PNG?raw=true)

Element 0 (Death) has the Npc Name and NpcMoodImages. The name is how the dialogue matches which npc images to look for (if death, it will stop and look at the images located here).

Npc Mood Images is an array of Images to be filled out. All images for this particular npc can be added here. In order to correctly display the mood, the file name of the image must contain the name of the mood (e.g. happy). The mood will be searched for to match the mood written in the message of the dialogue object, then displaying the image that matches.

For both fields, casing doesn’t matter. DeAth will match with DEATH and death. File names for images only need to contain the mood, so d14hfnrkjtyhgfhappy will work, and casing also doesn’t matter.

## Attaching dialogue
Objects that have fields for dialogue to be added can render dialogue.

![CageTrap example](https://github.com/Banananaman91/ScriptableDialogueSystem/blob/master/Images/CageTrap.PNG?raw=true)

Dialogue corresponds to the dialogue scriptableobject created. Notice this is a drop down, as this SO is editable here in the inspector if required.
Page Render refers to the default renderdialogue object.
Alternative Page Render refers to an alternate page render, if switching. In this example, Cage checks if all items are ready. If not, it treats it as death talking (alternate). If all items are present, a different UI displays dialogue asking the player if they wish to continue, triggering the event.

Start Message, all dialogue objects have this to give the ability to set which part of the dialogue to begin. Start message can be affected in code by certain triggers unlocking new dialogue starting points.

Trap refers to a game object specific to this event, in other scripts this could be animation fields added to be triggered or other such stuff.

Required items is specific to this item, and refers to what is required in order to unlock further dialogue here. Other items could have something different, or nothing.

Trap Trigger is the override for Start Message when the trap is able to be triggered.

![FurnitureInteract Example](https://github.com/Banananaman91/ScriptableDialogueSystem/blob/master/Images/FurnitureInteract.PNG?raw=true)

Notice the difference and similarities in fields. At the top of both, dialogue is separated with a header, this is the only area required to handle dialogue objects. FurnitureInteract also holds additional start message overrides at the bottom. This controls running the first message from a new location as the dialogue adapts.

HiddenObjectPosition is set to -1 automatically when HiddenObjectCheck is unchecked, handled by editor validation to prevent it coming unchecked and accidentally trying to run a message. An error will also pop up if HiddenObjectCheck is true and Hidden Object is null.

Hidden Object is the object that is hidden, being checked to see if it is in the inventory to control which message to play. Hidden Object Position should be set to the array point of the hidden object, checked against the currently collected object so that the code knows if it is at the hidden object or not. This is basically so the code knows that you want this object to be hidden without requiring a hidden object if not necessary (e.g. -1 will never match an object in the array)
