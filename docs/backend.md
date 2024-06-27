#Skill backend framework/Skill articulation
##Getting Skills Ready
 Search “Character_Profiles” in the search bar, this module lets you add skills in so that they can work on the framework.
<figure markdown="span">
![Image](imgs/charprofiles.png){ width="100%" align="left"}
</figure>
Once you open this module, it should look something like this
<figure markdown="span">
![Image](imgs/charp.png){ width="100%" align="left"}
</figure>
This is how the skill sets/movesets are set up. If you are to add your own moveset you'd add the following dicitonary element in the "configs" table
```lua title="Within Character_Profiles module"
["Example Moveset"] = {
		{
			Name = "Skill 2";
			CoolDown = 1;
			icon = "not important/leave blnak";
			Max_Hold = 4;
			Skill_Stats = { --important later/delete if you are copying this from the doc
				Invisibility = true;
				stun_bypass_skill = true;
			}
		};
		{
			Name = "Skill 2";
			CoolDown = 1;
			icon = "not important/leave blank";
			Max_Hold = 3;
			Skill_Stats = { --important later/delete if you are copying this from the doc
				Invisibility = true;
				Counter = 2;
			}
		};
}
```
!!!info "Skills_Stats"
    Ignore Skill_Stats right now, it will help us with important backend modules/skill behaviour later.
To equip said moveset to be tested you change the following variable within the module to correspond with the moveset index
<figure markdown="span">
![Image](imgs/Skillset.png){ width="100%" align="left"}
</figure>
After that the skills should now be in the framework and ready to be tested/used.
##Setting up skills modules
 All of the skills come with a pair of modules, one for the client and the other for the server, these modules are parented within the "Skills" folder within game.ReplicatedStorage.Cam
<figure markdown="span">
![Image](imgs/skillslo.png){ width="75%" align="left"}
</figure>
As you can see, there exists child folders within the “Skills” folder, the system does not take the folders into account, it uses GetDescendants() to get all the module scripts, therefor you can nest folder within folder, module within module, it does not matter. For skills to work there must exist two module scripts, you will copy past the “Template” and “TemplateServer” modules and name them according to the skill name. “Template” becomes “[insert skill name]”, this is the client module. “TemplateServer” becomes “[insert skill name]Server”, this is the server module.The template scripts contain all the functions that get called by the skill framework, the ones commented out are not mandatory. The other ones that are not commented out(Hold,Unhold, and Cancel) are mandatory unless stated otherwise. The following code shows the contents of the client module
```lua title="Within client script"
local module = {Id = 0}
function module:Hold(plr,mouseposition)
	print("hold block for ",plr)
end
function module:UnHold(plr,mouseposition)
	print("Unhold for ",plr.Name)
end
function module:Cancel(plr,mouseposition)
	print("Cancel block for ",plr.Name)
end


--[[ -- Optional. runs after the skill has been ran on the server
function module:After_Server_Hold_Signal(plr)
	
end
function module:Toggle(plr,mouseposition,counterie) --toggle skill(Uses RS>Cam>Global>Subsets>Gameplay>ToggleSkill)
	print("client tog")
end
function module:After_Server_Unhold_Signal(plr)

end
function module:After_Server_Cancel_Signal(plr)

end
function module:After_Server_Counter_Signal(plr)

end

function module:Counter(plr,mouseposition,counterie) --counter skill(Uses skill stats)
end
function module:Switch(plr) -- player flip switch function(Uses RS>Cam>Global>Subsets>Gameplay>Skill_Switch_Adder)

end

]]
return module
```
The id variable within the module, changes everytime the state of the skill changes, so it's different when the skill is held,unheld, or canceld. You can use that id to do some important checks in the Hold function. The following is the code within the server module
```lua title="Within the server module for skill"
local module = {Id = {}}
function module:Hold(plr,mouseposition)
	print("hold block for ",plr)
end
function module:UnHold(plr,mouseposition)
	print("Unhold for ",plr.Name)
end
function module:Cancel(plr,mouseposition,HitHoldLimit)
	print("Cancel block for ",plr.Name)
end

--[[-- Optional. runs after the skill has been ran on the server
function module:Switch(plr) --player switch function

end
function module:Toggle(plr) --toggle function --dodging etc
	print(plr.Name," dodge")
end
function module:Counter(plr,mouseposition,counterie) --counter skill
end
]]
return module
```
As you can see here, the Id variable's value is a table instead of a numeric value. This is because multiple players use the same module on the server, so to get the Id for a specific player you'd get it like this:
```lua
local Id = module.Id[plr]
```
**The server and client skill module will be ran simultaneously by the framework**
!!!note "Everything will be explained later"
    I know i haven't shown a way to continiously get the mouseposition, nor have i explained the functions that are commented out, those will be explained later in this page.
##Some Do's and Don'ts in the client module
 - You **can** create bodymovers on the client, as long as they are deleted in the Unhold or Cancel function. But if the bodymover propells the character up or down, it is recommended that you do that using the updrafting functions.
 - You **can** loop in the client modules, but the loop must have an end condition
 - You **Can't** fire or receive signals from remote events or remote functions within the client effects modules(There is another way to achieve server/client communication, this will be shown later)
 - You **can** require and use any module that is available to the client as long as recursion doesn't happen
##Some Do's and Don'ts in the server module
 - You **can't** create bodymovers for the player on the server, there are certain modules that will allow you to do knockbacks, updrafting,etc.
 - You **can't** receive or fire remote events or remote functions(There are exists a specific way to achieve server/client communication)
 - You **can** add body movers in victims as long as it lasts as long as their stun, if it's a body mover with the goal of changing their Y position somehow, use the updrafting functions which will be showcased later
##Sending signals to client to perform skill effects
###From server
```lua
game.ReplicatedStorage.Communication.SnC.Effects.RE:FireAllClients("effect module name",param1,param2,...,paramn)
```
or
```lua
game.ReplicatedStorage.Communication.SnC.Effects.RE:FireClient(plr,"effect module name",param1,param2,...,paramn)
```
The "RE" stands for "Remote Event" you can switch out to "RF" which stands for "remote function", this will tell the client to fire the modules created in the front-end framework
###From client
```lua
game.ReplicatedStorage.Communication.CnC.ClientEffects:Fire("effect module name",param1,param2,...,paramn)
```
##Forcing a skill action(Hold,Unhold,Cancel)
 Hold Will only work if there are no other skills being held, and will work like a tap. unHold and Cancel will only work if said skill is being held.
```lua title="Some server script"
game.ReplicatedStorage.Communication.SnC.Effects.RE:FireClient(plr,"force_skill_actions_server","skillname(Case sensitive)","Cancel")
```
As you can see this uses the same remote i use to fire effects to play. The frontend skill effects framework is not limited to skill effects.
###Use case
 Lets say you made a rasengan skill, it's a tap to activate, then you will add a loop that will run for a couple of seconds so that the "Hold" state stays on the client, u want the hold to be terminated if the player is stunned or if the timer runs out on the server, if you want to tell/force the client to unhold the skill you'd fire this.
##Getting player mouse position on the client
This is for getting an updated mouse position in a loop of some kind as the mouse position is already sent in the parameter
```lua title="Some client script"
local ph = require(game.ReplicatedStorage:WaitForChild("CAM"):WaitForChild("Client"):WaitForChild("Controllers"):WaitForChild("Platform_Handler"))

local mouse_pos = ph:mousepos(max depth(Optional))
```
##Getting mouse position on the server
This is tricky because it uses both the server and the client module script. It creates a part on the server that the player has network ownership over, Then in the client module, that part's position is updated to the mouse position, since the player has network ownership over it, the position will also updated on the server.
###Step 1: Creating the part on server script
We can create the part using the following module in the following way
```lua title="Some skill server script"
local SM = require(game.ServerStorage:WaitForChild("SAM"):WaitForChild("Services"):WaitForChild("Server_Mouse_Pos"))

SM:Create_Pos_Part(Character,name of the part,max existence duration)
```
Now we can access the part on both server or client with
```lua
local PP = character:FindFirstChild("PosPart".. name of the part .."Server")
print(PP.Position)
```
###Step 2: Updating the part's position on client
To update it on the client you'd run the following code in a loop
```lua
local PP = character:FindFirstChild("PosPart".. name of the part .."Server")
if PP ~= nil then
    PP.Position = mouse_pos
    PP.bp.Position = mouse_pos
    --i have a alignposition in it, i set the align position(PP.bp.Position) after setting the position of the part just to be safe, because body movers always replicate
end
```
###Step 3: Deleting the part
You can either let the duration run out, and the part auto deletes itself or delete it yourself by running
```lua
local SM = require(game.ServerStorage:WaitForChild("SAM"):WaitForChild("Services"):WaitForChild("Server_Mouse_Pos"))
SM:Delete_Pos_Part(Character,name of the part)
```
## Timed server/client skill communication portal
I have created a module, that lets u create a server client communication portal that lasts a set duration, this could be used during skills, it can also remove some purpose from the force skill action mechanic. The portal must be created on the server in the following manner
```lua title="Some server script"
local ServerClientLink = require(game.ReplicatedStorage:WaitForChild("CAM"):WaitForChild("Global"):WaitForChild("ServerClientLink"))
local Event,Send,Remove = ServerClientLink:New(plr,string: conneciton name,int: connection duration)
Event:Connect(function(param1,param2,...,paramn)
    --do something with the signal received from the client
end)
Send(param1,param2,...,paramn) --send signal to client
task.wait(5)
Remove() --delete the signal, this will automatically be done after the duration
```
!!!Note "Note"
    The "Remove" variable is only available on the server
On the client
```lua title="Some client script"
local ServerClientLink = require(game.ReplicatedStorage:WaitForChild("CAM"):WaitForChild("Global"):WaitForChild("ServerClientLink"))
local Event,Send = ServerClientLink:Connect(String: conneciton name, int: conneciton duration)
Event:Connect(function(param1,param2,...,paramn)
    --do something with the signal received from server
end)
Send(param1,param2,...,paramn) -- send signal to server
```
!!!Note "Note"
    You should check if Event and Send exist on the client prior to using them, incase the connection is unsuccessful
You can also remove a connection by name(Only on server)
```lua title="Some script on the server"
local ServerClientLink = require(game.ReplicatedStorage:WaitForChild("CAM"):WaitForChild("Global"):WaitForChild("ServerClientLink"))
ServerClientLink:RemoveLink(plr,String: connection name)
```
##Skill Stats
Some skills will have some properties, such as iframes, counter,etc. If we go back to the "Getting skills ready" header we can now put this to use
```lua title="In Character_Profiles"
["Example Moveset"] = {
        {
            Name = "Skill 2";
            CoolDown = 1;
            icon = "not important/leave blnak";
            Max_Hold = 4;
            Skill_Stats = {
                Invisibility = true;
                stun_bypass_skill = true;
            }
        };
        {
            Name = "Skill 2";
            CoolDown = 1;
            icon = "not important/leave blank";
            Max_Hold = 3;
            Skill_Stats = {
                Invisibility = true;
                Counter = 2;
            }
        };
}
```
Here are description of some skill stats
```lua
--[[current properties
    Cancel_Bypass=true --makes it so the skill can't be canceld when stunned,ragdolled,etc
   
   stun_bypass_skill=true --makes it so you can perform the skill while being stunned
    skills_to_play_over = {} --list skills that this will play over no matter what
    
   iframe = can be any value as long as it exists

   invisibility= can be any value as long as it exists, turns character invisible and undetecctable
    
   Block_Pierce --this is read only, notifies skill stats that this skill ignores block but does not damage while blocking
   
   Strict_Stun --this is read only, notifies skill strictly stuns, no blocking during the stun etc
   
   Counter = 1,2,3 -- 1 = combat, 2 = everything, 3 = skills only
  
   transparent = can be any value as long as it exists, turns the character invisible but it can be detected by npcs,etc.
]]
```
!!!Warning "Warning"
    Most of these are case sensitive. I might of also flipped the case for the first letter, if it doesn't work for you try, uppering or lowering the case.
##Counter skills
You setup a counter by having the property "Counter" in its skill stats
```lua title="In Character_Profiles"
["Example Moveset"] = {
        {
            Name = "counter skill";
            CoolDown = 1;
            icon = "not important/leave blank";
            Max_Hold = 3;
            Skill_Stats = {
                Counter = 2;
            }
        };
}
```
Then when the counter is trigured, the "Counter" function in both the server and client module of the skill, will be fired with the expected parameters.
##Creating remote controlled skills
Skills that u initiate and then toggle later, like a landmine skill that u can explode later.
```lua title="Some server skill module"
local skill_switch = require(game.ReplicatedStorage:WaitForChild("CAM"):WaitForChild("Global"):WaitForChild("Subsets"):WaitForChild("Gameplay"):WaitForChild("Skill_Switch_Adder"))
skill_switch:Add(plr, switch name, Duration(Optional?))
```
This will create a noticeable effect on the button icon, when that button is pressed again the "Switch" function will be fired on the server and client module. If duration is not specified, it will assume a default value.
##Skill passive toggling
Used for more than one toggling of the skill, triggered by different cases but not controlled by the player. This can be used if a skill activates a certain amount of dodges like ken haki.
```lua title="Some server skill module"
local PassiveToggleSignal = require(game.ReplicatedStorage:WaitForChild("CAM"):WaitForChild("Global"):WaitForChild("Subsets"):WaitForChild("Gameplay"):WaitForChild("SkillPassiveToggleSignal"))

PassiveToggleSignal:New(character,skillname,setting: {int: Duration(Optional?),int(Optional?): Type})
```
Here are the types of passive toggling
<figure markdown="span">
![Image](imgs/pt.png){ width="100%" align="left"}
</figure>
I use the "Menum" medoule because i like it,but it is optional you can just pass the corresponding numeric values instead. When the skills are triggered the "Toggle" function within the client and server skill scripts will be fired. If duration is not specified, it will assume a default value.
##A signaler for canceling(Not limited to skills framework)
Skills are only cancelable while the hold state is running, if the unhold or cancel state is being ran, then the skill is considered ran and can not be canceld or unheld anymore, so when we have yields in our cancel or unhold functions, we use the ManuelCancel module to pick up canceling signals.
```lua title="In Some server or client script"

local cancelsignal,canceldestroy = manuelCancel:New(character,int: duration(Optional))
cancelsignal:Connect(function()
    --do something
end)
task.wait(pos1)
canceldestroy()
```
If duration is not specified, it will assume a default value. calling the canceldestroy variable is optional, it is automatically called after the duration is competed
!!!Note "Note"
    this module can be ran outside of the skill framework, and can be ran on both the client and server.
##Casting a hitbox
You can cast a hitbox using one of the functions in the utility module
```lua title="Some script"
--< casting the hit box
local Models = Utility:GetModelInRegion(CFrame: cf,vector3: Size,string: custom tag name(Optional),max parts(Optional))
--<using the models captured
for i,v in pairs(Models) do
   do
        if v ~=character and v:FindFirstChild("Humanoid") ~= nil then
            local vicroot,vhum = v:FindFirstChild("HumanoidRootPart"),v:FindFirstChild("Humanoid")
        end
    end
end
```
##Checking if a captured enemy is damageable
You must use the checker module to check if a captured enemy is damageable
```lua
local checker = require(game.ReplicatedStorage:WaitForChild("CAM"):WaitForChild("Global"):WaitForChild("Checker"))
local result = checker:check_victim(script:GetFullName(),character/caster character,victim)
```
result can either be:

- True: This means everything is good to go
 - Perfect: The caster got perfect blocked
 - Block: The victim is blocking
 - nil: This means do not do anything to the victim

A caster is not expected to damage itself, so when u run this function on the caster themselves, you must add a 4th parameter telling the function to disregard the self damaging iframe otherwise it will always return nil
```lua
local checker = require(game.ReplicatedStorage:WaitForChild("CAM"):WaitForChild("Global"):WaitForChild("Checker"))
local result = checker:check_victim(script:GetFullName(),character/caster character,character/caster character,{iframe = true})
```
##Damaging,Stunning etc
These methods are done with the help of the Combat_Util module, that is only available on the server
```lua
local combat_util = require(game.ServerStorage:WaitForChild("SAM"):WaitForChild("Services"):WaitForChild("Combat_Util"));
Combat_Util:Block(script:GetFullName(),character,v,block_remove)
Combat_Util:AddStun(script:GetFullName(),character,vicvalues,1.5)
Combat_Util:RagDoll(script:GetFullName(),character,vicvalues,1.5)
Combat_Util:Damage(script:GetFullName(),character,v,{Base = dmg ,Skill = script.Parent.Name})
--v is the victim
--vicvalues is the values folder for the victim
```
These functions can be used on the caster without other considerations.
!!!Note "Note"
    Values folders are where i store values for the player, npcs,etc. I use values alot to depict states, send signals,etc. It is not part of the skill framework so ignore it for now.
To get the value folder of a player or npc u use the function
```lua

local Utility = require(game.ReplicatedStorage:WaitForChild("CAM"):WaitForChild("Global"):WaitForChild("Utility"))
vicvalues = Utility:getvaluesfolder(plrcharacter or npc/victim character)
```
##Getting a skill's status(Last used, or if it's being used)
```lua
local manage_cd = require(game.ServerStorage:WaitForChild("SAM"):WaitForChild("Game_Play"):WaitForChild("manage_cd"))
local lastUsed,isBeingUsed = manage_cd:skillStatus(plr,string: skill name) 
```
You can use this on remote toggle or passive toggle skills to measure the last time said skill was used.
##Updrafting
Many skills will have the ability to propell a player, or a player and the victim being attacked into the air, we use two other functions within Combat_Util to achieve this.
###Updrafting one entity
```lua
local combat_util = require(game.ServerStorage:WaitForChild("SAM"):WaitForChild("Services"):WaitForChild("Combat_Util"));
combat_util:Add_air_combo_bp(character humanoid root part,optional but can not be replaced,custom_y(Optional),startpos(Optional/X and Z are ignored))
--startpos is the ground position
--customy is the distance from the ground
```
The reason why the second parameter is optional and must exist, is because it is required when updrafting two entities, the function that updrafts two entities uses this function. So its space must be regarded.
###Updrafting two entities
```lua
local combat_util = require(game.ServerStorage:WaitForChild("SAM"):WaitForChild("Services"):WaitForChild("Combat_Util"));
combat_util:Air_combo_up(caster character,victim character)
```
This does a regular updraft on the caster character and victim character, it is used in combat, and many existing skills.
##Cutscene skills
Since not every cutscene skills will be casted similarly i will only explain the process of after it connects.

- Get the cframe of where it connected.
- Cast a hitbox(If necessary) and loop through all the captured victims
- Create invisible anchored parts that will be cframed according to the animation in which you will weld the caster and the victim, they must be on two seperate parts that respect the animation initial positioning.
- Make sure you aren't creating the caster's anchored part within the loops,if you do, then add sanity checks to make sure it only creates one part
- Add Stuns,iframes,pause_gameplay2 values that'll last the length of the cutscene, you will give them max existence duration and delete them when necessary.
- Play the animaiton for both the caster and the victim
- Get the camera rig, play animation on it, then use the camera module to stick the player and victim's cameras to the animaiton rig
```lua
local cutscene_handler = require(game.ServerStorage:WaitForChild("SAM"):WaitForChild("Game_Play"):WaitForChild("Cutscene_camera_handler"))
if Cam == nil then
    Cam = script.CameraRig:Clone()
    Cam:PivotTo(root.CFrame)
    Cam.Parent = workspace.Debree
    Cam.RootPart.RootPart.Part0 = root
    game.Debris:AddItem(Cam,1.73)
    Cam.AnimationController.Animator:LoadAnimation(script.glock_cam_thing):Play()
    do
        cutscene_handler:Regular(plr,Cam.Bone)
    end
end
if Cam ~= nil then
    --getting the victim if its a player so that we can play the camera animation to it too
    local pr = game.Players:GetPlayerFromCharacter(v)
    if pr then
        cutscene_handler:Regular(pr,Cam.Bone)
    end
end
```
!!!Note "Note"
    I will provide a ton of skill examples, don't worry... After reading the doc, you will understand the skill code easily.
##Long dash skills
These are done by running a hitbox on the server continiously, then when the hitbox finally hits something, you force the skill to be unheld, then remove or tween the body movers on client to stop while doing a capture action on the server. Most of the time the dash can also be stopped by the player unhold the skill.
##Reference skills
I have left a folder within game.ReplicatedStorage.Cam.Skills with reference skills, do not reuse or reference any skills that aren't in this folder.
<figure markdown="span">
![Image](imgs/manuelcancel.png){ width="100%" align="left"}
</figure>
