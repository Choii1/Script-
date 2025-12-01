--[[ 
===================================================================================
 🌀 CHOII HUB V.3 - Universal AutoFarm + AutoQuest (Rayfield)
 Optimized continuous farming: moves smoothly between nearest alive NPCs.
 Includes smart enemy detection + auto quest refresh.
 ✅ Fixed Key System (no GUI duplication)
 For Educational / Local Testing Only
===================================================================================
]]

-- ⚙️ Services & Rayfield
local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local workspace = game:GetService("Workspace")
local lp = Players.LocalPlayer

-- ✅ MAIN WINDOW (Key System Enabled)
local Window = Rayfield:CreateWindow({
   Name = "CHOII HUB V.3",
   Icon = 0,
   LoadingTitle = "CHOII HUB V.3",
   LoadingSubtitle = "by Reiner",
   Theme = "Default",
   ToggleUIKeybind = Enum.KeyCode.K,
   DisableRayfieldPrompts = false,
   DisableBuildWarnings = true,

   ConfigurationSaving = {
      Enabled = true,
      FolderName = "CHOII_HUB",
      FileName = "CHOII_V3_Config"
   },

   Discord = {
      Enabled = true,
      Invite = "gPYKfyn4", -- Correct invite code only
      RememberJoins = true
   },

   KeySystem = true, -- ✅ ENABLED
   KeySettings = {
      Title = "CHOII HUB Key System",
      Subtitle = "Verification Required",
      Note = "Join Discord (discord.gg/gPYKfyn4) to get your access key.",
      FileName = "CHOII_KEY",
      SaveKey = true,
      GrabKeyFromSite = false, -- Local validation only
      Key = {"Choyax_1314"} -- ✅ Accepted Key(s)
   }
})

-- 📍 Target Positions
local TargetPositions = {
    ["Criminal"] = Vector3.new(808.185, 330.235, 295.545),
    ["Weak Villain"] = Vector3.new(1248.301, 330.474, 145.102),
    ["Villain"] = Vector3.new(-145.176, 330.464, 948.465),
    ["Weak Nomu"] = Vector3.new(665.465, 330.466, 3123.402),
    ["High End"] = Vector3.new(24.542, 329.967, 3976.233),
    ["Tomura"] = Vector3.new(1419.275, 330.473, -380.593),
    ["Noumu"] = Vector3.new(785.753, 330.472, 951.200),
    ["Overhaul"] = Vector3.new(-741.451, 330.462, 1089.418),
    ["Muscular"] = Vector3.new(3069.172, 328.974, 2.696),
    ["Dabi"] = Vector3.new(2684.314, 328.974, 616.486),
    ["Gigantomachia"] = Vector3.new(2871.423, 328.974, 960.359),
    ["AllForOne"] = Vector3.new(852.494, 330.462, 3735.928),
    ["Awakened Tomura"] = Vector3.new(1044.694, 329.967, 4847.814),
    ["Police"] = Vector3.new(147.809, 329.298, 310.692),
    ["Hero"] = Vector3.new(300.072, 329.528, 174.840),
    ["UA Student"] = Vector3.new(486.568, 329.479, -570.322),
    ["Forest Beast"] = Vector3.new(2707.844, 328.037, 37.286),
    ["Pro Hero"] = Vector3.new(-226.541, 329.030, 3626.969),
    ["Present Mic"] = Vector3.new(844.265, 329.628, -796.399),
    ["Midnight"] = Vector3.new(176.390, 329.628, -803.946),
    ["Gang Orca"] = Vector3.new(1420.182, 330.475, 591.197),
    ["Mount Lady"] = Vector3.new(-495.443, 330.462, 624.299),
    ["Endeavor"] = Vector3.new(-512.884, 330.466, -281.769),
    ["Hawks"] = Vector3.new(-489.134, 330.365, 4331.164),
    ["Deku"] = Vector3.new(751.982, 329.967, 4363.521),
}

-- 📜 Quest Map
local QuestMap = {
    ["Injured Man"]     = {questName = "QUEST_INJURED MAN_1", target = "Criminal"},
    ["Aizawa"]          = {questName = "QUEST_AIZAWA_1", target = "Weak Villain"},
    ["Hero"]            = {questName = "QUEST_HERO_1", target = "Villain"},
    ["Jeanist"]         = {questName = "QUEST_JEANIST_1", target = "Weak Nomu"},
    ["Mirko"]           = {questName = "QUEST_MIRKO_1", target = "High End"},
    ["Gang Member"]     = {questName = "QUEST_GANG MEMBER_1", target = "Police"},
    ["Super Villain"]   = {questName = "QUEST_SUPER VILLAIN_1", target = "Hero"},
    ["Suspicious Char"] = {questName = "QUEST_SUSPICIOUS CHARACTER_1", target = "UA Student"},
    ["Twice"]           = {questName = "QUEST_TWICE_1", target = "Forest Beast"},
    ["Toga"]            = {questName = "QUEST_TOGA_1", target = "Pro Hero"},
}

-- 🔐 Safe Quest Start
local function safeFireStartQuest(qname)
    pcall(function()
        local questing = ReplicatedStorage:FindFirstChild("Questing")
        if questing and questing:FindFirstChild("Networking") then
            local net = questing.Networking:FindFirstChild("Remotes")
            if net and net:FindFirstChild("QUESTING_START_QUEST") then
                net.QUESTING_START_QUEST:FireServer(qname)
            end
        end
    end)
end

-- 🧭 Utility
local function teleportToVector(vec)
    if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        lp.Character:MoveTo(vec)
    end
end

local function clickMouse()
    local char = lp.Character
    if char and workspace:FindFirstChild(lp.Name) then
        local main = workspace[lp.Name]:FindFirstChild("Main")
        if main and main:FindFirstChild("Swing") then
            pcall(function() main.Swing:FireServer() end)
        end
    end
end

-- 🧠 NPC Detection
local NPC_FOLDERS = {"NPCs","Enemies","Mobs","Monsters","Living","Bots","Creatures"}

local function getAllEnemies()
    local list = {}
    for _, fname in ipairs(NPC_FOLDERS) do
        local folder = workspace:FindFirstChild(fname)
        if folder then
            for _, v in pairs(folder:GetChildren()) do
                if v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") then
                    if v.Humanoid.Health > 0 and not Players:FindFirstChild(v.Name) then
                        table.insert(list, v)
                    end
                end
            end
        end
    end
    return list
end

local function matchesPatterns(npcName, patterns)
    for _, p in ipairs(patterns) do
        if npcName == p or string.find(npcName, p) then return true end
    end
    return false
end

local function findClosestEnemy(patterns, maxDist)
    local hrp = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    local enemies = getAllEnemies()
    local best, bestDist = nil, maxDist or 3000
    for _, e in ipairs(enemies) do
        if matchesPatterns(e.Name, patterns) then
            local dist = (e.HumanoidRootPart.Position - hrp.Position).Magnitude
            if dist < bestDist then
                best, bestDist = e, dist
            end
        end
    end
    return best
end

-- 🕊️ Fly Tab
local FlyTab = Window:CreateTab("🕊️ Fly Options")
FlyTab:CreateParagraph({ Title = "Teleport Shortcuts", Content = "Teleport to any enemy area." })
for name, vec in pairs(TargetPositions) do
    FlyTab:CreateButton({ Name = "Fly to " .. name, Callback = function() teleportToVector(vec) end })
end

-- ⚔️ AutoFarm
local MainTab = Window:CreateTab("⚔️ AutoFarm")

local function createAutoFarm(displayName, patterns, position)
    local key = "Farm_" .. displayName:gsub("%s","_")
    _G[key] = false
    MainTab:CreateToggle({
        Name = "Auto Farm: " .. displayName,
        CurrentValue = false,
        Callback = function(state)
            _G[key] = state
            task.spawn(function()
                while _G[key] do
                    local target = findClosestEnemy(patterns, 3000)
                    if target then
                        local hrp = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
                        if hrp then
                            TweenService:Create(hrp, TweenInfo.new(0.3), {CFrame = target.HumanoidRootPart.CFrame * CFrame.new(0,5,0)}):Play()
                            repeat clickMouse(); task.wait(0.12)
                            until not target:FindFirstChild("Humanoid") or target.Humanoid.Health <= 0 or not _G[key]
                        end
                    else
                        teleportToVector(position)
                        task.wait(2)
                    end
                    task.wait(0.1)
                end
            end)
        end
    })
end

local function mkPatterns(name)
    return {name, name .. " 1", name .. " 2", name .. " 3"}
end

-- 🌟 FARM LIST
for name, pos in pairs(TargetPositions) do
    createAutoFarm(name, mkPatterns(name), pos)
end

-- 📜 Auto Quest
local QuestTab = Window:CreateTab("📜 Auto Quest")
_G["AutoQuest_Master"] = false
QuestTab:CreateToggle({
    Name = "Auto Accept Quests (Master)",
    CurrentValue = false,
    Callback = function(s) _G["AutoQuest_Master"] = s end
})

for giver, data in pairs(QuestMap) do
    local flag = "AutoQuest_" .. giver:gsub("%s","_")
    _G[flag] = false
    QuestTab:CreateToggle({
        Name = "Auto Quest: " .. giver,
        CurrentValue = false,
        Callback = function(state)
            _G[flag] = state
            if state then
                safeFireStartQuest(data.questName)
                task.spawn(function()
                    while _G[flag] do
                        safeFireStartQuest(data.questName)
                        task.wait(5)
                    end
                end)
            end
        end
    })
end

-- ⚙️ Settings
local SettingsTab = Window:CreateTab("⚙️ Settings")
SettingsTab:CreateButton({
    Name = "📉 Boost FPS / Reduce Graphics",
    Callback = function()
        pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)
        local Lighting = game:GetService("Lighting")
        Lighting.GlobalShadows = false
        Lighting.Brightness = 0
        Lighting.FogEnd = 9e9
        for _,v in pairs(workspace:GetDescendants()) do
            if v:IsA("ParticleEmitter") or v:IsA("Trail") then v.Enabled = false end
            if v:IsA("Decal") or v:IsA("Texture") then v.Transparency = 1 end
            if v:IsA("BasePart") then v.Material = Enum.Material.Plastic end
        end
    end
})

SettingsTab:CreateButton({
    Name = "🔄 Rejoin New Server",
    Callback = function()
        local s, res = pcall(function()
            return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/"..game.PlaceId.."/servers/Public?limit=100"))
        end)
        if s and res and res.data then
            for _, server in ipairs(res.data) do
                if server.playing < server.maxPlayers and server.id ~= game.JobId then
                    TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, lp)
                    break
                end
            end
        end
    end
})

print("[CHOII HUB V.3] ✅ Loaded Successfully with Key System")
