# SpawnNoob
For HiddenDevs

local ServerScriptService = game:GetService("ServerScriptService")
local Workspace = game:GetService("Workspace")

local SpawnNoobModule = require(ServerScriptService.Modules.SpawnNoobModule)
local RaritySpawnModule = require(ServerScriptService.Modules.RNGChancesModule)
local LeaderMemory = require(ServerScriptService.Modules.LeaderMemory)



local Charm = 1--math.random(1,3)
local SpawnNPCTime = 2

task.spawn(function()
	while true do
		Charm = 1
		wait(60 * 2)
		Charm = 3
		print("charm now")
		wait(30)
	end
end)

local Key = "SpawnNoob"

function Format(Int)
	return string.format("%02i", Int)
end

function convertToHMS(Seconds)
	local Minutes = (Seconds - Seconds%60)/60
	Seconds = Seconds - Minutes*60
	return Format(Minutes)..":"..Format(Seconds)
end

local LegendaryInterval = 60 * 5
local LastLegendaryTime = os.time()

local function StoreItem(npcName)
	LeaderMemory:Store(npcName, SpawnNPCTime, (LastLegendaryTime + LegendaryInterval) - os.time())
end

-- 🖥️ Real-time UI Timer Update
task.spawn(function()
	while true do
		local nextLegendaryIn = math.max(0, (LastLegendaryTime + LegendaryInterval) - os.time())
		Workspace.TimerStand.SurfaceGui.TextLabel.Text = convertToHMS(nextLegendaryIn)
		task.wait() -- update fast (10x per second)
	end
end)

-- 🤖 NPC Spawning Loop (every SpawnNPCTime seconds)
task.spawn(function()
	while true do
		local currentTime = os.time()
		local isLeader = LeaderMemory:IsLeader(Key)
		-- 💎 Legendary spawn
		if isLeader and currentTime - LastLegendaryTime >= LegendaryInterval then
			LastLegendaryTime = currentTime
			local ForcedLegendary = RaritySpawnModule:getRandomByRarity("Legendary")
			SpawnNoobModule:SpawnNoob(ForcedLegendary)
			StoreItem(ForcedLegendary)

			-- 👤 Normal spawn
		elseif isLeader then
			local npcName = RaritySpawnModule:selectRandom(Charm)
			SpawnNoobModule:SpawnNoob(npcName)
			StoreItem(npcName)
			--local memory = SpawnNoobModule.SafeGetMemory()
			--print(memory.Item)

		else
			local memory = SpawnNoobModule.SafeGetMemory(Key)
			if memory and memory.Item then
				SpawnNoobModule:SpawnNoob(memory.Item)
				local nextLegendaryIn = memory.NextLeg
				local nextLegendaryTimestamp = os.time() + nextLegendaryIn
				LastLegendaryTime = os.time() - (LegendaryInterval - memory.NextLeg)

			else
				warn("SafeGetMemory returned nil")
			end
		end

		task.wait(SpawnNPCTime)
	end
end)
