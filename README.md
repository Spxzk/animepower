local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer
local pastaInimigos = workspace:WaitForChild("spawners"):WaitForChild("tower")

local destinoFinal = CFrame.new(
    -2039.2738, 57.8565941, -14847.2793,
     0.844751239, 8.07595413e-08, -0.535159171,
    -1.04574049e-07, 1, -1.41631045e-08,
     0.535159171, 6.79280632e-08, 0.844751239
)

local autoTowerAtivo = false

-- Hora de Brasília
local function getBrasiliaTime()
    local utc = os.time(os.date("!*t"))
    return os.date("*t", utc - 3 * 3600)
end

local function segundosAteProximoHorario()
    local t = getBrasiliaTime()
    local min = t.min
    local sec = t.sec
    local proximo = (min < 30) and 30 or 60
    return (proximo - min) * 60 - sec
end

local function teleportarParaIlha()
    ReplicatedStorage.Shared.events.RemoteEvent:FireServer("teleportToWorld", "town of beginnings")
end

local function teleportarParaDestino()
    local character = player.Character or player.CharacterAdded:Wait()
    local hrp = character:WaitForChild("HumanoidRootPart")
    hrp.CFrame = destinoFinal
end

local function cicloTeleport()
    while autoTowerAtivo do
        local s = segundosAteProximoHorario()
        if s > 5 then task.wait(5) end
        if not autoTowerAtivo then break end

        teleportarParaIlha()
        task.wait(5)

        if not autoTowerAtivo then break end
        teleportarParaDestino()
        task.wait(1)
    end
end

local function teleportarParaNPC(npc)
    local character = player.Character or player.CharacterAdded:Wait()
    local hrp = character:WaitForChild("HumanoidRootPart")
    if npc then
        hrp.CFrame = npc.CFrame + Vector3.new(0, 3, 0)
    end
end

local function cicloAutoFarm()
    while autoTowerAtivo do
        for _, npc in pairs(workspace.spawners.tower:GetChildren()) do
                teleportarParaNPC(npc)
                repeat
                    task.wait(0.2)
             until not npc:IsDescendantOf(workspace)
            end
            task.wait(1)
        end
    end 

-- UI Fluent
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Window = Fluent:CreateWindow({
    Title = "Guest Exploit",
    SubTitle = "byy Guest",
    TabWidth = 160,
    Size = UDim2.fromOffset(500, 400),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "info" }),
    Setup = Window:AddTab({ Title = "Setup", Icon = "activity" }),
    Tower = Window:AddTab({ Title = "Tower", Icon = "swords" }),
    Stars = Window:AddTab({ Title = "Stars", Icon = "star" }),
}


Tabs.Tower:AddToggle("AutoTower", {
    Title = "Auto Tower",
    Default = false,
    Callback = function(value)
        autoTowerAtivo = value
        if value then
            task.spawn(cicloTeleport)
            task.spawn(cicloAutoFarm)
        end
    end
})

-- Toggle Auto Leave
Tabs.Tower:AddToggle("AutoLeaveToggle", {
    Title = "Auto Leave Floor",
    Default = false,
    Callback = function(state)
        autoLeaveAtivo = state
        if state then
            task.spawn(loopAutoLeave)
        end
    end
})

-- Função de Auto Leave
function loopAutoLeave()
    while autoLeaveAtivo do
        local label = player:FindFirstChild("PlayerGui") and player.PlayerGui:FindFirstChild("Interface")
            and player.PlayerGui.Interface:FindFirstChild("tower")
            and player.PlayerGui.Interface.tower:FindFirstChild("floor")
            and player.PlayerGui.Interface.tower.floor:FindFirstChild("label")

        if label and label:IsA("TextLabel") then
            local floorAtual = tonumber(label.Text:match("%d+"))
            if floorAtual and floorAtual >= floorLimite then
                autoFarmAtivo = false
                game:GetService("ReplicatedStorage").Shared.events.RemoteEvent:FireServer("teleportToWorld", "town of beginnings")
                break
            end
        else
            warn("Label da floor não encontrada!")
        end
        task.wait(1)
    end
end

-- Input para definir a floor limite
Tabs.Tower:AddInput("FloorInput", {
    Title = "Floor to leave",
    Default = "100",
    Placeholder = "",
    Numeric = true,
    Callback = function(value)
        local num = tonumber(value)
        if num and num >= 1 and num <= 100 then
            floorLimite = math.floor(num)
        else
        end
    end
})

local Dropdown = Tabs.Stars:AddDropdown("Dropdown", {
    Title = "Select Type to Roll",
    Values = {"one", "multi"},
    Multi = false,
    Default = "multi"
})

local ToggleAutoClick = Tabs.Setup:AddToggle("AutoClick", {Title = "Auto Click", Default = false})
ToggleAutoClick:OnChanged(function()
    while ToggleAutoClick.Value do wait(0.001)
        game:GetService("ReplicatedStorage").Shared.events.RemoteEvent:FireServer("attack")                
    end
end)

local ToggleAutoSpin = Tabs.Setup:AddToggle("Auto Spin", {Title = "Auto Spin Wheel", Default = false})
ToggleAutoSpin:OnChanged(function()
    while ToggleAutoSpin.Value do wait(1)
        game:GetService("ReplicatedStorage").Shared.events.RemoteEvent:FireServer("spinWheel", "normal")
    end
end)

local ToggleAutoRank = Tabs.Setup:AddToggle("Auto Rank", {Title = "Auto Rank Up", Default = false})
ToggleAutoRank:OnChanged(function()
    while ToggleAutoRank.Value do wait(5)
    game:GetService("ReplicatedStorage").Shared.events.RemoteEvent:FireServer("rankup")
    end
end)

local ToggleAutoSkill = Tabs.Setup:AddToggle("Auto Skill", {Title = "Auto Skill", Default = false})
ToggleAutoSkill:OnChanged(function()
    while ToggleAutoSkill.Value do wait(0.5)
        game:GetService("ReplicatedStorage").Shared.events.RemoteEvent:FireServer("useSkill", "power")
    end
end)

local ToggleAutoOpen = Tabs.Stars:AddToggle("Auto Open", {Title = "Auto Open", Default = false})
ToggleAutoOpen:OnChanged(function()
    while ToggleAutoOpen.Value do wait(0.1)
        local mapaAtual = game.Workspace.currentWorld:GetChildren()[1]

        local args = {
            "rollChampion",
            Dropdown.Value,
            mapaAtual.Name
        }
        game:GetService("ReplicatedStorage"):WaitForChild("Shared"):WaitForChild("events"):WaitForChild("RemoteEvent"):FireServer(unpack(args))
    end
end)
