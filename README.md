-- Script para entrar automaticamente na equipe Pirates
task.wait(27) -- Delay de 40 segundos antes de entrar na equipe Pirates
local args = {
    [1] = "ChooseTeam",
    [2] = "Pirates"
}

game:GetService("ReplicatedStorage").BetweenSides.Remotes.Events.PlayerEvent:FireServer(unpack(args))

-- Aguarda 5 segundos e executa Refresh Tools automaticamente
task.wait(5)
pcall(function()
    if type(RefreshTools) == "function" then
        RefreshTools()
    end
end)


-- ===== Auto-load ao trocar de servidor =====
local queue_on_tp = (syn and syn.queue_on_teleport)
    or (fluxus and fluxus.queue_on_teleport)
    or (queue_on_teleport)
do
    -- Defina a URL RAW do seu script para reexecutar após o teleport.
    -- Ex.: getgenv().SCRIPT_URL = "https://raw.githubusercontent.com/usuario/repositorio/main/script.lua"
    local SCRIPT_URL = (getgenv and getgenv().SCRIPT_URL) or "REPLACE_WITH_YOUR_RAW_URL"
    if typeof(queue_on_tp) == "function" and type(SCRIPT_URL) == "string" and string.find(SCRIPT_URL, "http") then
        queue_on_tp(('loadstring(game:HttpGet("%s"))()'):format(SCRIPT_URL))
    end
end
-- ============================================

-- Anti-AFK sempre ativo
pcall(function()
    game:GetService("Players").LocalPlayer.Idled:Connect(function()
        game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(1)
        game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    end)
end)

-- Auto entrar no time Pirates
pcall(function()
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local TeamEvent = ReplicatedStorage:FindFirstChild("BetweenSides") and ReplicatedStorage.BetweenSides.Remotes.Events:FindFirstChild("TeamsEvent")
    if TeamEvent then
        TeamEvent:FireServer("JoinTeam", "Pirates")
    end
end)


--i make open source because its shit and there are few players, then u can skidd me only pls do credit :heart:
local _ENV = (getgenv or getrenv or getfenv)()
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local DialogueEvent = ReplicatedStorage.BetweenSides.Remotes.Events.DialogueEvent
local CombatEvent = ReplicatedStorage.BetweenSides.Remotes.Events.CombatEvent
local ToolEvent = ReplicatedStorage.BetweenSides.Remotes.Events.ToolsEvent
local QuestsNpcs = workspace.IgnoreList.Int.NPCs.Quests
local Enemys = workspace.Playability.Enemys
local QuestsDecriptions = require(ReplicatedStorage.MainModules.Essentials.QuestDescriptions)
local EnemiesFolders = {}
local CFrameAngle = CFrame.Angles(math.rad(- 90), 0, 0)


local function a()
    local QuestsList = {}
    local CurrentQuest = nil
    local CurrentLevel = - 1
    for _, QuestData in QuestsDecriptions do
        if QuestData.Goal <= 1 then
            continue
        end
        table.insert(QuestsList, {
            Level = QuestData.MinLevel,
            Target = QuestData.Target,
            NpcName = QuestData.Npc,
            Id = QuestData.Id
        })
    end
    table.sort(QuestsList, function(a, b)
        return a.Level > b.Level
    end)
    local function b()
        local Level = nil
        local success, result = pcall(function()
            local mainUI = Player.PlayerGui:FindFirstChild("MainUI")
            if mainUI then
                local mainFrame = mainUI:FindFirstChild("MainFrame")
                if mainFrame then
                    local statsFrame = mainFrame:FindFirstChild("StastisticsFrame") or mainFrame:FindFirstChild("StatisticsFrame")
                    if statsFrame then
                        local levelBG = statsFrame:FindFirstChild("LevelBackground")
                        if levelBG then
                            local levelText = levelBG:FindFirstChild("Level")
                            if levelText and levelText.Text then
                                return tonumber(levelText.Text)
                            end
                        end
                        for _, child in pairs(statsFrame:GetDescendants()) do
                            if child:IsA("TextLabel") and child.Text:match("^%d+$") then
                                local num = tonumber(child.Text)
                                if num and num >= 1 and num <= 2000 then
                                    return num
                                end
                            end
                        end
                    end
                end
            end
            return 1
        end)
        if success and result then
            Level = result
        else
            Level = 1
        end
        if Level == CurrentLevel then
            return CurrentQuest
        end
        for _, QuestData in QuestsList do
            if QuestData.Level <= Level then
                CurrentLevel, CurrentQuest = Level, QuestData
                return QuestData
            end
        end
        return nil
    end
    return b()
end

local Settings = {
    ClickV2 = false,
    TweenSpeed = 250,
    SelectedTool = "CombatType",
    BringMobDistance = 35,
    dSpeed = 0.05,
    NoClip = false,
    AutoStats = false,
    SelectedStat = "Strength"
}

local EquippedTool = nil
local CurrentTarget = nil

local conepc = _ENV.cnn or {}
_ENV.cnn = conepc

for i = 1, # conepc do
    conepc[i]:Disconnect()
end

table.clear(conepc)

local function c(Character)
    if Character then
        local Humanoid = Character:FindFirstChildOfClass("Humanoid")
        return Humanoid and Humanoid.Health > 0
    end
end

local BodyVelocity = Instance.new("BodyVelocity")
BodyVelocity.Velocity = Vector3.zero
BodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
BodyVelocity.P = 1000

if _ENV.tween_bodyvelocity then
    _ENV.tween_bodyvelocity:Destroy()
end

_ENV.tween_bodyvelocity = BodyVelocity

local CanCollideObjects = {}

local function ss(Object)
    if Object:IsA("BasePart") and Object.CanCollide then
        table.insert(CanCollideObjects, Object)
    end
end

local function rrr(BasePart)
    local index = table.find(CanCollideObjects, BasePart)
    if index then
        table.remove(CanCollideObjects, index)
    end
end

local function ne(Character)
    table.clear(CanCollideObjects)
    for _, Object in Character:GetDescendants() do
        ss(Object)
    end
    Character.DescendantAdded:Connect(ss)
    Character.DescendantRemoving:Connect(rrr)
end

table.insert(conepc, Player.CharacterAdded:Connect(ne))
task.spawn(ne, Player.Character)

local function np(Character)
    if _ENV.OnFarm then
        for i = 1, # CanCollideObjects do
            CanCollideObjects[i].CanCollide = false
        end
    elseif Character.PrimaryPart and not Character.PrimaryPart.CanCollide then
        for i = 1, # CanCollideObjects do
            CanCollideObjects[i].CanCollide = true
        end
    end
end

local function upe(Character)
    local BasePart = Character:FindFirstChild("UpperTorso")
    local Humanoid = Character:FindFirstChild("Humanoid")
    local BodyVelocity = _ENV.tween_bodyvelocity
    if _ENV.OnFarm and BasePart and Humanoid and Humanoid.Health > 0 then
        if BodyVelocity.Parent ~= BasePart then
            BodyVelocity.Parent = BasePart
        end
    elseif BodyVelocity.Parent then
        BodyVelocity.Parent = nil
    end
    if BodyVelocity.Velocity ~= Vector3.zero and (not Humanoid or not Humanoid.SeatPart or not _ENV.OnFarm) then
        BodyVelocity.Velocity = Vector3.zero
    end
end

table.insert(conepc, RunService.Stepped:Connect(function()
    local Character = Player.Character
    if c(Character) then
        upe(Character)
        np(Character)
    end
end))

local TweenCreator = {}
TweenCreator.__index = TweenCreator

local tweens = {}
local EasingStyle = Enum.EasingStyle.Linear

function TweenCreator.new(obj, time, prop, value)
    local self = setmetatable({}, TweenCreator)
    self.tween = TweenService:Create(obj, TweenInfo.new(time, EasingStyle), {
        [prop] = value
    })
    self.tween:Play()
    self.value = value
    self.object = obj
    if tweens[obj] then
        tweens[obj]:destroy()
    end
    tweens[obj] = self
    return self
end

function TweenCreator:destroy()
    self.tween:Pause()
    self.tween:Destroy()
    tweens[self.object] = nil
    setmetatable(self, nil)
end

function TweenCreator:stopTween(obj)
    if obj and tweens[obj] then
        tweens[obj]:destroy()
    end
end

local function TweenStopped()
    if not BodyVelocity.Parent and c(Player.Character) then
        TweenCreator:stopTween(Player.Character:FindFirstChild("HumanoidRootPart"))
    end
end

local lastCFrame = nil
local lastTeleport = 0

local function tepe(TargetCFrame)
    while not Player.Character or not c(Player.Character) do
        task.wait(1)
        if not _ENV.OnFarm then
            return false
        end
    end
    if not Player.Character or not Player.Character.PrimaryPart then
        return false
    end
    if (tick() - lastTeleport) <= 0.3 and lastCFrame == TargetCFrame then
        return false
    end
    local Character = Player.Character
    local Humanoid = Character:FindFirstChildOfClass("Humanoid")
    local PrimaryPart = Character.PrimaryPart
    if not Humanoid or not PrimaryPart then
        return false
    end
    if Humanoid.Sit then
        Humanoid.Sit = false
        task.wait(0.1)
        return false
    end
    lastTeleport = tick()
    lastCFrame = TargetCFrame
    _ENV.OnFarm = true
    local teleportPosition = TargetCFrame.Position
    local currentPosition = PrimaryPart.Position
    local Distance = (currentPosition - teleportPosition).Magnitude
    
    TweenCreator:stopTween(PrimaryPart)
    local tween = TweenCreator.new(PrimaryPart, Distance / Settings.TweenSpeed, "CFrame", TargetCFrame)
    local connection
    connection = RunService.Heartbeat:Connect(function()
        if not c(Character) or not _ENV.OnFarm then
            TweenCreator:stopTween(PrimaryPart)
            if connection then
                connection:Disconnect()
            end
        end
    end)
    coroutine.wrap(function()
        while tween.tween.PlaybackState == Enum.PlaybackState.Playing do
            task.wait()
        end
        if connection then
            connection:Disconnect()
        end
    end)()
    return true
end

table.insert(conepc, BodyVelocity:GetPropertyChangedSignal("Parent"):Connect(TweenStopped))

local CurrentTime = workspace:GetServerTimeNow()

local function d()
    if not c(Player.Character) then
        return
    end
    local Tool = Player.Character:FindFirstChildOfClass("Tool")
    if not Tool then
        return
    end
    CurrentTime = workspace:GetServerTimeNow()
    pcall(function()
        Tool:Activate()
        local Handle = Tool:FindFirstChild("Handle")
        if Handle then
            if Handle:FindFirstChild("Cooldown") then
                Handle.Cooldown.Value = 0
            end
            if Handle:FindFirstChild("AttackCooldown") then
                Handle.AttackCooldown.Value = 0
            end
            if Handle:FindFirstChild("Debounce") then
                Handle.Debounce.Value = false
            end
            local Sound = Handle:FindFirstChildOfClass("Sound")
            if Sound then
                Sound:Play()
            end
        end
        ToolEvent:FireServer("Effects", 1)
        ToolEvent:FireServer("Activate", 1)
        if Settings.ClickV2 then
            for i = 1, 3 do
                Tool:Activate()
                task.wait(0.01)
            end
        end
    end)
end

local function faz(Enemies)
    CurrentTime = workspace:GetServerTimeNow()
    CombatEvent:FireServer("DealDamage", {
        CallTime = CurrentTime,
        DelayTime = 0,
        Combo = 1,
        Results = Enemies,
        Damage = math.random(50, 150),
        CriticalHit = math.random(1, 10) <= 3
    })
end

local function irl(Folder, EnemyName)
    local foundEnemies = {}
    for _, Enemy in pairs(Folder:GetChildren()) do
        if Enemy and Enemy.Parent then
            local enemyName = Enemy:GetAttribute("OriginalName") or Enemy:GetAttribute("EnemyName") or Enemy.Name
            if enemyName == EnemyName or string.find(enemyName, EnemyName) then
                local Humanoid = Enemy:FindFirstChild("Humanoid")
                local HumanoidRootPart = Enemy:FindFirstChild("HumanoidRootPart")
                if Humanoid and HumanoidRootPart and Humanoid.Health > 0 then
                    local isReady = true
                    if Enemy:GetAttribute("Respawned") ~= nil then
                        isReady = Enemy:GetAttribute("Respawned")
                    end
                    if Enemy:GetAttribute("Ready") ~= nil then
                        isReady = isReady and Enemy:GetAttribute("Ready")
                    end
                    if isReady then
                        table.insert(foundEnemies, Enemy)
                    end
                end
            end
        end
    end
    return foundEnemies
end

local function pgaall(EnemyName)
    local AllEnemies = {}
    local EnemyFolder = EnemiesFolders[EnemyName]
    if EnemyFolder and EnemyFolder.Parent then
        local enemies = irl(EnemyFolder, EnemyName)
        for _, enemy in ipairs(enemies) do
            table.insert(AllEnemies, enemy)
        end
    else
        if Enemys and Enemys.Parent then
            local Islands = Enemys:GetChildren()
            for i = 1, # Islands do
                local Island = Islands[i]
                if Island and Island.Parent then
                    local enemies = irl(Island, EnemyName)
                    if # enemies > 0 then
                        EnemiesFolders[EnemyName] = Island
                        for _, enemy in ipairs(enemies) do
                            table.insert(AllEnemies, enemy)
                        end
                    end
                end
            end
        end
    end
    return AllEnemies
end

local function clst(EnemyName)
    local AllEnemies = pgaall(EnemyName)
    if # AllEnemies == 0 then
        return nil
    end
    local ClosestEnemy = nil
    local ShortestDistance = math.huge
    for i, Enemy in ipairs(AllEnemies) do
        if Enemy and Enemy.Parent then
            local RootPart = Enemy:FindFirstChild("HumanoidRootPart")
            local Humanoid = Enemy:FindFirstChild("Humanoid")
            if RootPart and Humanoid and Humanoid.Health > 0 then
                local Distance = Player:DistanceFromCharacter(RootPart.Position)
                if Distance < ShortestDistance then
                    ShortestDistance = Distance
                    ClosestEnemy = Enemy
                end
            end
        end
    end
    return ClosestEnemy
end

local function brnge(EnemyName, TargetPosition)
    if not _ENV.BringMob then
        return 0
    end
    local AllEnemies = pgaall(EnemyName)
    local BroughtCount = 0
    if # AllEnemies == 0 then
        return 0
    end
    for i, Enemy in ipairs(AllEnemies) do
        if Enemy and Enemy.Parent then
            local RootPart = Enemy:FindFirstChild("HumanoidRootPart")
            local Humanoid = Enemy:FindFirstChild("Humanoid")
            if RootPart and Humanoid and Humanoid.Health > 0 then
                if not RootPart:FindFirstChild("BodyVelocity") then
                    local BV = Instance.new("BodyVelocity", RootPart)
                    BV.Velocity = Vector3.zero
                    BV.MaxForce = Vector3.one * math.huge
                end
                RootPart.CanCollide = false
                RootPart.Size = Vector3.one * Settings.BringMobDistance
                RootPart.CFrame = TargetPosition
                BroughtCount = BroughtCount + 1
            end
        end
    end
    if BroughtCount > 0 then
        pcall(sethiddenproperty, Player, "SimulationRadius", math.huge)
    end
    return BroughtCount
end

local function IsSelectedTool(Tool)
    return Tool:GetAttribute(Settings.SelectedTool)
end

local function EquipCombat(Activate)
    if not c(Player.Character) then
        return
    end
    if EquippedTool and IsSelectedTool(EquippedTool) then
        if Activate then
            if Settings.ClickV2 then
                d()
            else
                EquippedTool:Activate()
            end
        end
        if EquippedTool.Parent == Player.Backpack then
            Player.Character.Humanoid:EquipTool(EquippedTool)
        elseif EquippedTool.Parent ~= Player.Character then
            EquippedTool = nil
        end
        return
    end
    local Equipped = Player.Character:FindFirstChildOfClass("Tool")
    if Equipped and IsSelectedTool(Equipped) then
        EquippedTool = Equipped
        return
    end
    for _, Tool in Player.Backpack:GetChildren() do
        if Tool:IsA("Tool") and IsSelectedTool(Tool) then
            EquippedTool = Tool
            Player.Character.Humanoid:EquipTool(Tool)
            return
        end
    end
end

local function ten(EnemyName)
    local success, result = pcall(function()
        local QuestFrame = Player.PlayerGui.MainUI.MainFrame.CurrentQuest
        if not QuestFrame.Visible then
            return false
        end
        local questText = nil
        local goalElement = QuestFrame:FindFirstChild("Goal")
        if goalElement and goalElement.Text then
            questText = goalElement.Text
        end
        if not questText then
            for _, child in pairs(QuestFrame:GetDescendants()) do
                if child:IsA("TextLabel") and child.Text and child.Text ~= "" then
                    if string.find(child.Text, "Defeat") or string.find(child.Text, "/") then
                        questText = child.Text
                        break
                    end
                end
            end
        end
        if questText then
            local hasTargetQuest = string.find(questText, EnemyName)
            return hasTargetQuest ~= nil
        else
            return false
        end
    end)
    if success then
        return result
    else
        return false
    end
end

local function ete(QuestName, QuestId)
    local Npc = QuestsNpcs:FindFirstChild(QuestName, true)
    local RootPart = Npc and Npc.PrimaryPart
    if RootPart then
        DialogueEvent:FireServer("Quests", {
            ["NpcName"] = QuestName,
            ["QuestName"] = QuestId
        })
        tepe(RootPart.CFrame * CFrame.new(0, 0, 15))
        task.wait(2)
    end
end

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Window = Fluent:CreateWindow({
    Title = "Gostz",
    SubTitle = "by shaka",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = false,
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.LeftShift
})

local Tabs = {
    Main = Window:AddTab({
        Title = "Main",
        Icon = "rbxassetid://131580529707278"
    }),
    Misc = Window:AddTab({
        Title = "Misc",
        Icon = "rbxassetid://139867952423882"
    }),
    SettingsTab = Window:AddTab({
        Title = "Settings",
        Icon = "rbxassetid://113400358595552"
    })
}

Tabs.Main:AddSection("Auto Farm")
Tabs.Main:AddToggle("AutoFarm", {
    Title = "Auto Farm With Quests",
    Default = true,
    Callback = function(Value)
        _ENV.OnFarm = Value
        if Value then
            task.spawn(function()
                while task.wait(Settings.dSpeed) and _ENV.OnFarm do
                    if not c(Player.Character) then
                        repeat
                            task.wait(0.5)
                        until c(Player.Character)
                        task.wait(0.2)
                        continue
                    end
                    local CurrentQuest = a()
                    if not CurrentQuest then
                        continue
                    end
                    if not ten(CurrentQuest.Target) then
                        ete(CurrentQuest.NpcName, CurrentQuest.Id)
                        continue
                    end
                    local Enemy = clst(CurrentQuest.Target)
                    if not Enemy then
                        continue
                    end
                    CurrentTarget = Enemy
                    local HumanoidRootPart = Enemy:FindFirstChild("HumanoidRootPart")
                    local Humanoid = Enemy:FindFirstChild("Humanoid")
                    if HumanoidRootPart and Humanoid and Humanoid.Health > 0 then
                        if _ENV.BringMob then
                            brnge(CurrentQuest.Target, HumanoidRootPart.CFrame)
                        end
                        local targetCFrame = HumanoidRootPart.CFrame * CFrame.new(0, 7.5, 0) * CFrameAngle
                        
                        tepe(targetCFrame)
                        task.wait(0)
                        local AllQuestEnemies = pgaall(CurrentQuest.Target)
                        if # AllQuestEnemies > 0 then
                            faz(AllQuestEnemies)
                        end
                        d()
                    end
                end
            end)
        end
    end
})
Tabs.Main:AddToggle("BringMob", {
    Title = "Bring Mob (Current Quest)",
    Default = true,
    Callback = function(Value)
        _ENV.BringMob = Value
    end
})
Tabs.Main:AddSection("Tools")
local ToolDropdown = Tabs.Main:AddDropdown("ToolDropdown", {
    Title = "Select Tool",
    Values = {},
    Multi = false,
    Default = nil
})
local ToolToggle = Tabs.Main:AddToggle("ToolToggle", {
    Title = "Auto Equip Tool",
    Default = true
})
local equipping = false
local function EquipSelectedTool()
    if not ToolToggle.Value then
        return
    end
    if equipping then
        return
    end
    equipping = true
    local selectedTool = ToolDropdown.Value
    if selectedTool then
        local tool = Player.Backpack:FindFirstChild(selectedTool)
        if tool and Player.Character and Player.Character:FindFirstChild("Humanoid") then
            pcall(function()
                Player.Character.Humanoid:EquipTool(tool)
            end)
        end
    end
    equipping = false
end
Player.CharacterAdded:Connect(function(character)
    character:WaitForChild("Humanoid")
    if ToolToggle.Value then
        task.wait(1)
        EquipSelectedTool()
    end
end)
ToolToggle:OnChanged(function(Value)
    if Value then
        coroutine.wrap(function()
            while ToolToggle.Value do
                EquipSelectedTool()
                task.wait(0.1)
            end
        end)()
    else
        if Player.Character then
            local currentTool = Player.Character:FindFirstChildOfClass("Tool")
            if currentTool then
                currentTool.Parent = Player.Backpack
            end
        end
    end
end)
local function RefreshTools()
    local tools = {}
    if Player and Player:FindFirstChild("Backpack") then
        for _, tool in ipairs(Player.Backpack:GetChildren()) do
            if tool:IsA("Tool") then
                table.insert(tools, tool.Name)
            end
        end
    end
    ToolDropdown:SetValues(tools)
end

-- === Force "Combat" tool selected & auto-equip ===
local function findCombatToolName()
    local function hasCombatAttr(tool)
        local ok, val = pcall(function() return tool:GetAttribute("CombatType") end)
        return ok and (val == true or val == "true")
    end
    -- search character first
    if Player.Character then
        for _, t in ipairs(Player.Character:GetChildren()) do
            if t:IsA("Tool") and (hasCombatAttr(t) or string.lower(t.Name):find("combat") or string.lower(t.Name):find("combate")) then
                return t.Name
            end
        end
    end
    -- then backpack
    if Player and Player:FindFirstChild("Backpack") then
        for _, t in ipairs(Player.Backpack:GetChildren()) do
            if t:IsA("Tool") and (hasCombatAttr(t) or string.lower(t.Name):find("combat") or string.lower(t.Name):find("combate")) then
                return t.Name
            end
        end
    end
    return nil
end

local ForceCombatSelected = true
task.spawn(function()
    while ForceCombatSelected do
        local ct = findCombatToolName()
        if ct then
            pcall(function()
                ToolDropdown:SetValue(ct)
            end)
        end
        task.wait(0.5)
    end
end)
-- ================================================
local RefreshButton = Tabs.Main:AddButton({
    Title = "Refresh Tools",
    Callback = RefreshTools
})
task.spawn(function()
    task.wait(2)
    RefreshTools()
end)
Tabs.Main:AddSection("Auto Stats")
local StatDropdown = Tabs.Main:AddDropdown("StatDropdown", {
    Title = "Select Stat to Upgrade",
    Values = {
        "Strength",
        "Defense",
        "Sword",
        "Gun",
        "DevilFruit"
    },
    Multi = false,
    Default = "Strength",
    Callback = function(Value)
        Settings.SelectedStat = Value
    end
})
local AutoStatsToggle = Tabs.Main:AddToggle("AutoStatsToggle", {
    Title = "Auto Stats Selected",
    Default = true,
    Callback = function(Value)
        Settings.AutoStats = Value
        if Value then
            coroutine.wrap(function()
                while Settings.AutoStats do
                    local remote = ReplicatedStorage.BetweenSides.Remotes.Events.StatsEvent
                    if remote then
                        local args = {
                            "UpgradeStat",
                            {
                                Defense = Settings.SelectedStat == "Defense" and 1 or 0,
                                Sword = Settings.SelectedStat == "Sword" and 1 or 0,
                                Gun = Settings.SelectedStat == "Gun" and 1 or 0,
                                Strength = Settings.SelectedStat == "Strength" and 1 or 0,
                                DevilFruit = Settings.SelectedStat == "DevilFruit" and 1 or 0
                            }
                        }
                        pcall(function()
                            remote:FireServer(unpack(args))
                        end)
                    end
                    task.wait(0.1)
                end
            end)()
        end
    end
})
local char = game:GetService("Players").LocalPlayer.Character or game:GetService("Players").LocalPlayer.CharacterAdded:Wait()
local fireTouchEnabled = false
local function fireTouch(part)
    if not fireTouchEnabled or not part:IsA("BasePart") then
        return
    end
    for _, ti in ipairs(part:GetChildren()) do
        if ti:IsA("TouchTransmitter") then
            for _, myPart in ipairs(char:GetDescendants()) do
                if myPart:IsA("BasePart") then
                    firetouchinterest(myPart, part, 0)
                    task.wait()
                    firetouchinterest(myPart, part, 1)
                end
            end
        end
    end
end
Tabs.Misc:AddSection("Teleports")
local islands = {}
local map = workspace:FindFirstChild("Map")
if map then
    for _, island in ipairs(map:GetChildren()) do
        if island:FindFirstChild("Base") then
            table.insert(islands, island.Name)
        end
    end
end
local IslandDropdown = Tabs.Misc:AddDropdown("IslandDropdown", {
    Title = "Select Island",
    Values = islands,
    Multi = false,
    Default = islands[1] or nil
})
Tabs.Misc:AddButton({
    Title = "Teleport to Island",
    Callback = function()
        local selectedIsland = IslandDropdown.Value
        if selectedIsland and map then
            local island = map:FindFirstChild(selectedIsland)
            if island and island:FindFirstChild("Base") then
                -- Reiniciar personagem antes de teleportar
                if Player.Character and Player.Character:FindFirstChild("Humanoid") then
                    Player.Character.Humanoid.Health = 0
                end

                -- Esperar respawn
                local newChar = Player.CharacterAdded:Wait()

                -- Garantir que carregou completamente
                newChar:WaitForChild("HumanoidRootPart", 5)

                -- Agora teleportar
                local base = island.Base
                if base:FindFirstChild("HumanoidRootPart") then
                    newChar:MoveTo(base.HumanoidRootPart.Position)
                else
                    newChar:MoveTo(base.WorldPivot.Position + Vector3.new(0, 5, 0))
                end
            end
        end
    end
})
Tabs.Misc:AddSection("Others")
Tabs.Misc:AddToggle("FireTouchToggle", {
    Title = "Auto Chest And Fruit Spawned",
    Default = false,
    Callback = function(value)
        fireTouchEnabled = value
        if value then
            for _, v in ipairs(workspace:GetDescendants()) do
                fireTouch(v)
            end
        end
    end
})
local isRunning = false
local function storeFruit()
    while isRunning do
        local fruitArgs = {
            "StoreFruit"
        }
        game:GetService("ReplicatedStorage"):WaitForChild("BetweenSides"):WaitForChild("Remotes"):WaitForChild("Events"):WaitForChild("ToolsEvent"):FireServer(unpack(fruitArgs))
        task.wait(0.25)
    end
end
Tabs.Misc:AddToggle("AutoStoreToggle", {
    Title = "Auto Store Fruits",
    Default = false,
    Callback = function(value)
        isRunning = value
        if value then
            coroutine.wrap(storeFruit)()
        end
    end
})
-- ===== Server Hop (Misc) - LOW POP =====
do
    local TeleportService = game:GetService("TeleportService")
    local HttpService = game:GetService("HttpService")
    local Players = game:GetService("Players")
    local PlaceId = game.PlaceId
    local CurrentJobId = game.JobId

    -- CONFIG: máximo de players que consideramos "pouco cheio"
    local TARGET_MAX = 5         -- mude para 3/8/etc se quiser
    local INCLUDE_EMPTY = true   -- permitir servidores vazios (0 players)

    local function tryTeleport(jobId)
        if jobId and jobId ~= CurrentJobId then
            TeleportService:TeleportToPlaceInstance(PlaceId, jobId, Players.LocalPlayer)
            return true
        end
        return false
    end

    local function HopLowPop()
        local cursor = ""
        -- percorre algumas páginas para achar instância com pouca gente
        for _ = 1, 8 do
            local url = ("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100%s")
                :format(PlaceId, (cursor ~= "" and ("&cursor=" .. cursor) or ""))
            local ok, body = pcall(function() return game:HttpGet(url) end)
            if not ok or not body then break end

            local data = nil
            local okDecode, decoded = pcall(function() return HttpService:JSONDecode(body) end)
            if okDecode then data = decoded end
            if not data or not data.data then break end

            -- 1) tentar 1..TARGET_MAX players
            for _, server in ipairs(data.data) do
                local p = tonumber(server.playing) or 0
                local cap = tonumber(server.maxPlayers) or 0
                if server.id ~= CurrentJobId and p > 0 and p <= TARGET_MAX and p < cap then
                    if tryTeleport(server.id) then return end
                end
            end

            -- 2) opcional: tentar vazios
            if INCLUDE_EMPTY then
                for _, server in ipairs(data.data) do
                    local p = tonumber(server.playing) or 0
                    local cap = tonumber(server.maxPlayers) or 0
                    if server.id ~= CurrentJobId and p == 0 and p < cap then
                        if tryTeleport(server.id) then return end
                    end
                end
            end

            cursor = data.nextPageCursor or ""
            if cursor == "" or cursor == nil then break end
        end

        -- Fallback: qualquer servidor diferente (Roblox decide)
        TeleportService:Teleport(PlaceId, Players.LocalPlayer)
    end

    -- Adiciona o botão "Hop Server" na aba Misc (se ainda não existir)
    if Tabs and Tabs.Misc then
        local okSection = true
        pcall(function() Tabs.Misc:AddSection("Server") end)
        pcall(function()
            Tabs.Misc:AddButton({
                Title = "Hop Server (Low Pop)",
                Description = "Ir para um servidor com pouca gente",
                Callback = HopLowPop
            })
        end)
    end
end
-- ============================================

-- ===== Hop automático se alguém ficar perto por 50s =====
do
    local ProximityHopEnabled = true -- ligado por padrão
    local HOP_CHANCE = 0.5 -- 50% de chance de executar o hop
    local NEAR_DISTANCE = 30         -- distância para considerar "perto" (studs)
    local REQUIRED_SECONDS = 2       -- tempo contínuo com alguém perto
    
    local nearStart = nil
    local lastJobId = game.JobId

    -- expõe função HopLowPop() se estiver local ao escopo
    local function _tryHop()
        if typeof(HopLowPop) == "function" then
            HopLowPop()
        else
            -- fallback: Teleport para qualquer outro servidor
            local TeleportService = game:GetService("TeleportService")
            local Players = game:GetService("Players")
            TeleportService:Teleport(game.PlaceId, Players.LocalPlayer)
        end
    end

    -- UI (Aba Misc)
    if Tabs and Tabs.Misc then
        Tabs.Misc:AddSection("Segurança")
        Tabs.Misc:AddToggle("ProximityHopToggle", {
            Title = "Hop se alguém perto (50s)",
            Default = true,
            Callback = function(v) ProximityHopEnabled = v end
        })
        Tabs.Misc:AddSlider("ProxDist", {
            Title = "Distância de detecção (studs)",
            Default = NEAR_DISTANCE, Min = 10, Max = 200, Rounding = 0,
            Callback = function(val) NEAR_DISTANCE = val end
        })
    end

    -- Monitoramento contínuo
    task.spawn(function()
        while true do
            task.wait(0.2)
            if not ProximityHopEnabled then
                nearStart = nil
                continue
            end
            local myChar = Player.Character
            if not myChar or not myChar.PrimaryPart then
                nearStart = nil
                continue
            end
            local myPos = myChar.PrimaryPart.Position
            local someoneNear = false
            for _, pl in ipairs(Players:GetPlayers()) do
                if pl ~= Player then
                    local ch = pl.Character
                    local hum = ch and ch:FindFirstChildOfClass("Humanoid")
                    local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
                    if hum and hum.Health > 0 and hrp then
                        if (hrp.Position - myPos).Magnitude <= NEAR_DISTANCE then
                            someoneNear = true
                            break
                        end
                    end
                end
            end
            if someoneNear then
                if not nearStart then nearStart = time() end
                if time() - nearStart >= REQUIRED_SECONDS then
                    nearStart = nil
                    if math.random() < HOP_CHANCE then
            _tryHop()
        end
                end
            else
                nearStart = nil
            end
        end
    end)
end
-- =========================================================

-- === Minimize/Restore toggle (starts minimized) ===
local interfaceMinimized = true

-- Start minimized right after UI is created
task.spawn(function()
    task.wait(0.2)
    pcall(function() Window:Minimize() end)
end)

if Tabs and Tabs.SettingsTab then
    pcall(function() Tabs.SettingsTab:AddSection("Interface") end)
    pcall(function()
        Tabs.SettingsTab:AddButton({
            Title = "Mostrar/Ocultar Interface",
            Description = "Alterna entre minimizar e restaurar a janela",
            Callback = function()
                if interfaceMinimized then
                    pcall(function() Window:Restore() end)
                    interfaceMinimized = false
                else
                    pcall(function() Window:Minimize() end)
                    interfaceMinimized = true
                end
            end
        })
    end)
end
-- ================================================
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({})
InterfaceManager:BuildInterfaceSection(Tabs.SettingsTab)
SaveManager:BuildConfigSection(Tabs.SettingsTab)
Window:SelectTab(1)
