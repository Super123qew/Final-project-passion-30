# Source code (Roblox uses a dialect of Lua called Luau)


## Main Script


```luau

local ServerStorage = game:GetService("ServerStorage")

local bindables = ServerStorage:WaitForChild("Bindables")
local gameOverEvent = bindables:WaitForChild("GameOver")



local base = require(script.Base)
local mob= require(script.Mob)
local tower = require(script.Tower)
local map =workspace.Grassland

local gameOver = false

base.Setup(map,500)
 
 gameOverEvent.Event:Connect(function()
		gameOver = true
 end)
 
task.wait(1)
--base.UpdateHealth(50)


for wave=1, 10 do
	print("WAVE STARTING:", wave)
	
	if wave <=2 then
		mob.Spawn("Zombie", 3 * wave, map)
	elseif wave <=4 then
		for i=1,5 do
			mob.Spawn("Zombie", wave, map)
			mob.Spawn("Noob", wave, map)
		end
	elseif wave <=7 then
		for i=1,5 do
			mob.Spawn("Zombie", 3 * wave, map) 
			mob.Spawn("Noob", 3 * wave, map)
			mob.Spawn("Mech", 2 * (wave*0.5), map)
		end
	elseif wave <=9 then
		mob.Spawn("Mech", 2 * wave, map)
		mob.Spawn("Noob",wave, map)
		for i=1,5 do
			mob.Spawn("Zombie", (wave*0.5), map)
			mob.Spawn("Mech", (wave*0.5), map)
		end
	elseif wave == 10 then	
		mob.Spawn("Zombie", 100, map)
		for i=1, 5 do
			mob.Spawn("Mech", 5, map)
			mob.Spawn("Teddy", 1, map)
			mob.Spawn("Zombie", 2, map)
		end
		mob.Spawn("Teddy", 10, map)
	end
	
	repeat
		task.wait(1)
	until #workspace.Mobs:GetChildren() == 0 or gameOver
	
	if gameOver then
		print ("GAME OVER MAN")
		break
	end
	
	print("WAVE ENDED")
	task.wait(1)
end 
	
```

## Base Module

```luau
local ServerStorage = game:GetService("ServerStorage")
	
local bindables = ServerStorage:WaitForChild("Bindables")
local updateBaseHealthEvent = bindables:WaitForChild("UpdateBaseHealth")

local gameOverEvent = bindables:WaitForChild("GameOver")


local base = {}
       
function base.Setup(map, health)
	base.Model = map:WaitForChild ("Base")
	base.CurrentHealth = health
	base.MaxHealth = health
	base.UpdateHealth()
end

function base.UpdateHealth(damage)
	if damage then
		base.CurrentHealth -= damage
	end
	
	local gui= base.Model.HealthGui
	local percent = base.CurrentHealth / base.MaxHealth
	
	gui.CurrentHealth.Size = UDim2.new(percent, 0, 0.5, 0)
	
	if base.CurrentHealth <= 0 then
		gameOverEvent:Fire()
		gui.Title.Text ="Foster home lost!"
		else
		gui.Title.Text = "Base: " .. base.CurrentHealth .. "/" .. base.MaxHealth
	end
	
	
end

updateBaseHealthEvent.Event:Connect(base.UpdateHealth)

return base


```

## Mob Module

```luau

local PhysicsService = game:GetService("PhysicsService")
local ServerStorage = game:GetService("ServerStorage")
local ServerStorage = game:GetService("ServerStorage")

local bindables = ServerStorage:WaitForChild("Bindables")
local updateBaseHealthEvent = bindables:WaitForChild("UpdateBaseHealth")



local mob = {}



function mob.Move (mob, map)
	local humanoid = mob:WaitForChild("Humanoid")
	local waypoints = map.Waypoints
	
	for waypoint=1, #waypoints:GetChildren()do
		humanoid:MoveTo(waypoints[waypoint].Position)
		humanoid.MoveToFinished:Wait()
	end
	
	mob:Destroy()
    updateBaseHealthEvent:Fire(humanoid.Health)
end

function mob.Spawn(name, quantity, map)
	local mobExists = ServerStorage.Mobs:FindFirstChild(name)
	
	if mobExists then
		for i=1, quantity do
			task.wait(0.5)
			local newMob = mobExists:Clone()
			newMob.HumanoidRootPart.CFrame = map.Start.CFrame
			newMob.Parent =workspace.Mobs
			newMob.HumanoidRootPart:SetNetworkOwner(nil)
			
			for i, object in ipairs(newMob:GetDescendants()) do
				if object:IsA("BasePart") then
					object.CollisionGroup = "Mob"
				end
			end
			
			
			newMob.Humanoid.Died:Connect(function()
				task.wait(0.5)
					newMob:Destroy()
				
			end)
			
	
			coroutine.wrap(mob.Move) (newMob, map)
		end
	
	
		--move into workspace
	else
		warn ("Requested mob does not exist", name)
	end
	
	task.wait(1)
end

return mob

```

## Tower Module

```luau

local PhysicsService = game:GetService("PhysicsService")
local ServerStorage = game:GetService("ServerStorage")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local events = ReplicatedStorage:WaitForChild("events")
local spawnTowerEvent = events:WaitForChild("SpawnTower")

local tower = {}

function FindNearestTarget(newTower, range)
	local nearestTarget = nil

	for i, target in ipairs(workspace.Mobs:GetChildren()) do
		local distance = (target.HumanoidRootPart.position - newTower.HumanoidRootPart.Position).Magnitude
		if distance < range then
			nearestTarget = target
			range = distance
		end
	end

	return nearestTarget
end

function tower.Attack(newTower, player)
	local config = newTower.Config
	local target = FindNearestTarget(newTower, config.Range.Value)
	
	if target and target:FindFirstChild("Humanoid") and target.Humanoid.Health > 0 then 
		
		local targetCFrame = CFrame.lookAt(newTower.HumanoidRootPart.Position, target.HumanoidRootPart.Position)
		newTower.HumanoidRootPart.BodyGyro.CFrame = targetCFrame
		
		target.Humanoid:TakeDamage(config.Damage.Value)
		
		task.wait(config.Cooldown.Value)
		
		if target.Humanoid.Health <= 0 then
			player.Gold.Value += target.Humanoid.MaxHealth
			
		end
	end

	task.wait(0.1)
	
	tower.Attack(newTower, player)
end

function tower.Spawn(player, name, cframe)
	local towerExists = ReplicatedStorage.Towers:FindFirstChild(name)
	
	if towerExists then
		
		local newTower = towerExists:Clone()
		newTower.HumanoidRootPart.CFrame = cframe
		newTower.Parent = workspace.Towers
		newTower.HumanoidRootPart:SetNetworkOwner(nil)
		
		local bodyGyro = Instance.new("BodyGyro")
		bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
		bodyGyro.D = 0
		bodyGyro.CFrame = newTower.HumanoidRootPart.CFrame
		bodyGyro.Parent = newTower.HumanoidRootPart

		for i, object in ipairs(newTower:GetDescendants()) do
			if object:IsA("BasePart") then
				PhysicsService:SetPartCollisionGroup(object, "Tower")
				object.CollisionGroup = "Tower"
			end
		end
	
	
		coroutine.wrap(tower.Attack)(newTower, player)
		--move into workspace
	else
		warn ("Requested tower does not exist", name)
	end
end

spawnTowerEvent.OnServerEvent:Connect(tower.Spawn)

return tower

```

## GameController Local Script

```luau

local Players = game:GetService("Players")
local PhysicsService = game:GetService("PhysicsService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local userinputservice = game:GetService("UserInputService")

local gold = Players.LocalPlayer:WaitForChild("Gold")
local events = ReplicatedStorage:WaitForChild("events")
local functions = ReplicatedStorage:WaitForChild("Functions")
local requestTowerFunction = functions:WaitForChild("RequestTower")
local towers = ReplicatedStorage:WaitForChild("Towers")
local spawnTowerEvent = events:WaitForChild("SpawnTower")
local camera = workspace.CurrentCamera
local gui = script.Parent
--local item = nil

local towerToSpawn = nil
local canPlace = false
local rotation = 0
local placedTowers = 0
local maxTowers = 10

local function updateGold ()
	gui.Gold.Text = gold.Value
end
updateGold()
gold.Changed:Connect(updateGold)

local function MouseRaycast(exclude)
	local mousePosition = userinputservice:GetMouseLocation()
	local mouseRay = camera:ViewportPointToRay(mousePosition.X, mousePosition.Y)
	local raycastParams = RaycastParams.new()
	
	raycastParams.FilterType = Enum.RaycastFilterType.Exclude
	raycastParams.FilterDescendantsInstances = exclude

	local raycastResult = workspace:Raycast(mouseRay.Origin, mouseRay.Direction * 1000, raycastParams)

	
	return raycastResult
end

local function RemovePlaceholderTower()
	if towerToSpawn then
		towerToSpawn:Destroy()
		towerToSpawn = nil
	end
end

local function AddPlaceholderTower(name)
	
	local towerExists = towers:FindFirstChild(name)
	if towerExists then
		RemovePlaceholderTower()
		towerToSpawn = towerExists:Clone()
		towerToSpawn.Parent = workspace.Towers
		
		for i, object in ipairs(towerToSpawn:GetDescendants()) do
			if object:IsA("BasePart") then
				PhysicsService:SetPartCollisionGroup(object, "Tower")
				object.Material = Enum.Material.ForceField
			end
		end
	end
end

local function colorPlaceholderTower(color)  
	for i, object in ipairs(towerToSpawn:GetDescendants()) do
		if object:IsA("BasePart") then
			object.Color = color
		end
	end
end
	
gui.Towers.Title.Text = "Towers:" .. placedTowers .. "/" .. maxTowers
for i, tower in pairs(towers:GetChildren()) do
	local button = gui.Towers.ImageButton:Clone()
	local config = tower:WaitForChild("Config")
	button.Name = tower.Name
	button.Image = config.Image.Texture
	button.Visible = true
	button.Price.Text = config.Price.Value
	button.LayoutOrder = config.Price.Value
	button.Parent = gui.Towers
	
	button.Activated:Connect(function()
		local allowedToSPawn = requestTowerFunction:InvokeServer(tower.Name)
		if allowedToSPawn then
		AddPlaceholderTower((tower.Name))
		end
	end)
end




userinputservice.InputBegan:Connect(function(input, processed)
	if processed then 
		return
	end
	
	if towerToSpawn then
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			if canPlace then
				spawnTowerEvent:FireServer(towerToSpawn.Name, towerToSpawn.PrimaryPart.CFrame)
				placedTowers += 1
				gui.Towers.Title.Text = "Towers:" .. placedTowers .. "/" .. maxTowers
				RemovePlaceholderTower()
			end	
		elseif input.KeyCode == Enum.KeyCode.R then
			rotation += 90
		end
	end
	
end)

RunService.RenderStepped:Connect(function()
	if towerToSpawn then
		local result = MouseRaycast({towerToSpawn})
		if result and result.Instance then
			if result.Instance.Parent.Name == "TowerArea" then
				canPlace = true
				colorPlaceholderTower(Color3.new(0,1,0))
			else
				canPlace = false
				colorPlaceholderTower(Color3.new(1,0,0))
			end 
 			local x = result.Position.X
			local y = result.Position.Y + towerToSpawn.Humanoid.HipHeight + (towerToSpawn.PrimaryPart.Size.Y/2) + 1.5
			local z = result.Position.Z

			local cframe = CFrame.new(x,y,z) * CFrame.Angles(0, math.rad(rotation),0)
			towerToSpawn:SetPrimaryPartCFrame(cframe)
		end
	end
end)

```

## MobAnimations Local Script

```luau

local function animateMob (object)
	local humandoid = object:WaitForChild("Humanoid")
	local  animationsFolder =object:WaitForChild("Animations")
	
	if humandoid and animationsFolder then
		local walkAnimation = animationsFolder:WaitForChild("Walk")	
		if walkAnimation then
			local animator = humandoid:FindFirstChild("Animator") or Instance.new("Animator", humandoid)
			local walkTrack = animator:LoadAnimation(walkAnimation)
			walkTrack:Play()
		
		end
		
	else
		warn("No Humanoid/Animation folder for", object.Name)
	end
	
end

workspace.Mobs.ChildAdded:Connect(animateMob)

```

DataStore Script (WIP)

```luau

```

OnPlayerAdded Script

```luau

local players = game:GetService("Players")
local PhysicsService = game:GetService("PhysicsService")

players.PlayerAdded:Connect(function(player)
	
	local gold = Instance.new("IntValue")
	gold.Name = "Gold"
	gold.Value = 250
	gold.Parent = player
	 


	
	player.CharacterAdded:Connect(function(character)
		for i, object in ipairs(character:GetDescendants()) do
			if object:IsA("BasePart") then
				object.CollisionGroup = "Player"
			end
		end
	end)
end)

```



