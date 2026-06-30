# Implementation Patterns

## Purpose

- Use when Deepin needs to generate real code, not only checklists.
- Read with the active specialist.
- Copy the pattern, then adapt system name, folders, remotes, schema, and comment language.

## Module Shape

```luau
--!strict

export type Config = {
	Debug: boolean,
}

local Service = {}
Service.__index = Service

export type Service = typeof(setmetatable({} :: {
	_config: Config,
	_connections: { RBXScriptConnection },
}, Service))

function Service.new(config: Config): Service
	local self = setmetatable({
		_config = config,
		_connections = {},
	}, Service)

	return self
end

function Service:Init(): ()
	-- SETUP ONLY
end

function Service:Start(): ()
	-- CONNECT EVENTS
end

function Service:Destroy(): ()
	for _, connection in self._connections do
		connection:Disconnect()
	end

	table.clear(self._connections)
end

return Service
```

## Cleanup Container

```luau
--!strict

local Maid = {}
Maid.__index = Maid

export type Maid = typeof(setmetatable({} :: {
	_tasks: { any },
}, Maid))

function Maid.new(): Maid
	return setmetatable({
		_tasks = {},
	}, Maid)
end

function Maid:Give(taskObject: any): any
	table.insert(self._tasks, taskObject)
	return taskObject
end

function Maid:Clean(): ()
	for i = #self._tasks, 1, -1 do
		local taskObject = self._tasks[i]
		self._tasks[i] = nil

		if typeof(taskObject) == "RBXScriptConnection" then
			taskObject:Disconnect()
		elseif typeof(taskObject) == "Instance" then
			taskObject:Destroy()
		elseif type(taskObject) == "function" then
			taskObject()
		elseif type(taskObject) == "table" and type(taskObject.Destroy) == "function" then
			taskObject:Destroy()
		end
	end
end

return Maid
```

## Remote Map

```luau
--!strict

local Remotes = {
	Shop_Buy_RE = {
		ClassName = "RemoteEvent",
		Direction = "ClientToServer",
		Rate = 0.5,
		Payload = {
			ItemId = "string",
		},
	},
	Shop_Sync_RE = {
		ClassName = "RemoteEvent",
		Direction = "ServerToClient",
		Rate = 0.2,
		Payload = {
			Coins = "number",
			Owned = "table",
		},
	},
	Shop_GetInfo_RF = {
		ClassName = "RemoteFunction",
		Direction = "ClientToServer",
		Rate = 2,
		Payload = {},
	},
}

return Remotes
```

## Rate Limiter

```luau
--!strict

local RateLimiter = {}

local buckets: { [Player]: { [string]: number } } = {}

function RateLimiter.Allow(player: Player, key: string, cooldown: number): boolean
	local now = os.clock()
	local userBucket = buckets[player]

	if not userBucket then
		userBucket = {}
		buckets[player] = userBucket
	end

	local last = userBucket[key]
	if last and now - last < cooldown then
		return false
	end

	userBucket[key] = now
	return true
end

function RateLimiter.Clear(player: Player): ()
	buckets[player] = nil
end

return RateLimiter
```

## Payload Validation

```luau
--!strict

local Validator = {}

function Validator.String(value: any, minLength: number, maxLength: number): (boolean, string?)
	if type(value) ~= "string" then
		return false, "type"
	end

	local length = #value
	if length < minLength or length > maxLength then
		return false, "length"
	end

	return true, nil
end

function Validator.Number(value: any, minValue: number, maxValue: number): (boolean, string?)
	if type(value) ~= "number" then
		return false, "type"
	end

	if value ~= value or value == math.huge or value == -math.huge then
		return false, "finite"
	end

	if value < minValue or value > maxValue then
		return false, "range"
	end

	return true, nil
end

return Validator
```

## Remote Handler

```luau
--!strict

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local RateLimiter = require(script.Parent.RateLimiter)
local Validator = require(script.Parent.Validator)

local remotes = ReplicatedStorage:WaitForChild("ShopSystem"):WaitForChild("Remotes")
local buyRemote = remotes:WaitForChild("Shop_Buy_RE") :: RemoteEvent

local function canBuy(player: Player, itemId: string): boolean
	-- SERVER CHECK
	return itemId ~= "" and player.Parent == Players
end

buyRemote.OnServerEvent:Connect(function(player: Player, payload: any)
	if not RateLimiter.Allow(player, "Shop_Buy_RE", 0.5) then
		return
	end

	if type(payload) ~= "table" then
		return
	end

	local ok = Validator.String(payload.ItemId, 1, 64)
	if not ok then
		return
	end

	if not canBuy(player, payload.ItemId) then
		return
	end

	-- GRANT SERVER
end)

Players.PlayerRemoving:Connect(function(player: Player)
	RateLimiter.Clear(player)
end)
```

## Profile Schema

```luau
--!strict

local CURRENT_VERSION = 3

export type ProfileData = {
	DataVersion: number,
	Coins: number,
	Inventory: { string },
	Receipts: { [string]: boolean },
}

local DEFAULT_DATA: ProfileData = {
	DataVersion = CURRENT_VERSION,
	Coins = 0,
	Inventory = {},
	Receipts = {},
}

local function cloneDefaults(): ProfileData
	return {
		DataVersion = DEFAULT_DATA.DataVersion,
		Coins = DEFAULT_DATA.Coins,
		Inventory = table.clone(DEFAULT_DATA.Inventory),
		Receipts = table.clone(DEFAULT_DATA.Receipts),
	}
end

local function reconcile(data: any): ProfileData
	local output = cloneDefaults()

	if type(data) == "table" then
		if type(data.DataVersion) == "number" then
			output.DataVersion = data.DataVersion
		end
		if type(data.Coins) == "number" then
			output.Coins = math.max(0, math.floor(data.Coins))
		end
		if type(data.Inventory) == "table" then
			output.Inventory = data.Inventory
		end
		if type(data.Receipts) == "table" then
			output.Receipts = data.Receipts
		end
	end

	if output.DataVersion < 2 then
		output.Inventory = output.Inventory or {}
	end

	if output.DataVersion < 3 then
		output.Receipts = output.Receipts or {}
	end

	output.DataVersion = CURRENT_VERSION
	return output
end

return {
	CurrentVersion = CURRENT_VERSION,
	Default = DEFAULT_DATA,
	Reconcile = reconcile,
}
```

## Receipt Idempotency

```luau
--!strict

local MarketplaceService = game:GetService("MarketplaceService")

local function grantProduct(player: Player, productId: number): boolean
	-- SERVER GRANT
	return productId > 0 and player.Parent ~= nil
end

MarketplaceService.ProcessReceipt = function(receiptInfo)
	local player = game.Players:GetPlayerByUserId(receiptInfo.PlayerId)
	if not player then
		return Enum.ProductPurchaseDecision.NotProcessedYet
	end

	local profile = nil -- get active profile
	if not profile then
		return Enum.ProductPurchaseDecision.NotProcessedYet
	end

	local purchaseId = tostring(receiptInfo.PurchaseId)
	profile.Data.Receipts = profile.Data.Receipts or {}

	if profile.Data.Receipts[purchaseId] then
		return Enum.ProductPurchaseDecision.PurchaseGranted
	end

	local granted = grantProduct(player, receiptInfo.ProductId)
	if not granted then
		return Enum.ProductPurchaseDecision.NotProcessedYet
	end

	profile.Data.Receipts[purchaseId] = true
	return Enum.ProductPurchaseDecision.PurchaseGranted
end
```

## Server Authority Gameplay

```luau
--!strict

local ATTACK_RANGE = 12
local ATTACK_COOLDOWN = 0.65

local lastAttack: { [Player]: number } = {}

local function getRoot(character: Model?): BasePart?
	if not character then
		return nil
	end

	return character:FindFirstChild("HumanoidRootPart") :: BasePart?
end

local function canAttack(player: Player, target: Model): boolean
	local now = os.clock()
	if lastAttack[player] and now - lastAttack[player] < ATTACK_COOLDOWN then
		return false
	end

	local attackerRoot = getRoot(player.Character)
	local targetRoot = getRoot(target)
	if not attackerRoot or not targetRoot then
		return false
	end

	if (attackerRoot.Position - targetRoot.Position).Magnitude > ATTACK_RANGE then
		return false
	end

	lastAttack[player] = now
	return true
end

local function applyDamage(player: Player, target: Model, amount: number): ()
	if not canAttack(player, target) then
		return
	end

	local humanoid = target:FindFirstChildOfClass("Humanoid")
	if not humanoid then
		return
	end

	humanoid:TakeDamage(math.clamp(amount, 1, 25))
end
```

## Round State Machine

```luau
--!strict

local Round = {}

type State = "Intermission" | "Countdown" | "Playing" | "Results"

local state: State = "Intermission"

local transitions = {
	Intermission = "Countdown",
	Countdown = "Playing",
	Playing = "Results",
	Results = "Intermission",
}

function Round.GetState(): State
	return state
end

function Round.Next(): State
	state = transitions[state] :: State
	return state
end

function Round.CanJoin(player: Player): boolean
	return state == "Intermission" or state == "Countdown"
end

return Round
```

## Usage Rule

- Prefer these patterns over ad hoc code.
- Keep examples adapted, not copied blindly.
- Preserve current project architecture.
- Keep server authority and runtime validation.
