local OrionLib = loadstring(game:HttpGet(("https://raw.githubusercontent.com/BlizTBr/scripts/main/Orion%20X")))()

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer


_G.LoopKill = false
_G.PlayerToAddList = {}
_G.LoopKillConnection = nil
_G.SavedCFrame = nil
_G.KillDelay = 0
_G.KillOffset = Vector3.new(0, -6, 0) 
_G.MaxTargets = 25


function _G.DisableCollisions(Char)
    if not Char then return end

    for _, part in pairs(Char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end
end

function _G.ReturnToSavedPosition()
    if _G.SavedCFrame and LocalPlayer.Character then
        LocalPlayer.Character:PivotTo(_G.SavedCFrame)
    end
end

function _G.IsAboveLimit(Player)
    return #_G.PlayerToAddList > _G.MaxTargets
end

function _G.ForceDeath(Root, Humanoid)
    pcall(function()
        Humanoid.Health = 0
    end)
end


local Window = OrionLib:MakeWindow({
    Name = "Loop Kill",
    HidePremium = false,
    SaveConfig = false,
    ConfigFolder = "Kill de caca cookie",
    IntroEnabled = true,
    IntroText = "Chargement caca hub..."
})

local SettingsTab = Window:MakeTab({
    Name = " Settings",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local LoopPlayersTab = Window:MakeTab({
    Name = "Loop Players",
    Icon = "rbxassetid://6034170707", 
    PremiumOnly = false
})



function _G.ExecuteKill(Player)
    if not _G.LoopKill or not Player or not Player.Character then return end
    if Workspace.PlotItems and Workspace.PlotItems.PlayersInPlots:FindFirstChild(Player.Name) then return end

    local Char = Player.Character
    local Root = Char:FindFirstChild("HumanoidRootPart")
    local Head = Char:FindFirstChild("Head")
    local Humanoid = Char:FindFirstChild("Humanoid")
    if not (Root and Head and Humanoid) or Humanoid.Health <= 0 or _G.IsAboveLimit(Player) then return end

    local SelfChar = Players.LocalPlayer.Character
    if not SelfChar or not SelfChar:FindFirstChild("HumanoidRootPart") then return end

    pcall(function()
        if not _G.LoopKill then return end
        
        SelfChar:PivotTo(CFrame.new(Root.Position + _G.KillOffset))
        
        if not _G.LoopKill then return end
        
        _G.DisableCollisions(Char)
        
        if not _G.LoopKill then return end
        
        ReplicatedStorage.GrabEvents.SetNetworkOwner:FireServer(Root, Root.CFrame)
        task.wait()
        

        _G.ReturnToSavedPosition()
        task.wait(0)
        
        if not _G.LoopKill then return end
        

        ReplicatedStorage.GrabEvents.DestroyGrabLine:FireServer(Root)
        task.wait(0)
        

        if Head:FindFirstChild("PartOwner") and Head.PartOwner.Value == Players.LocalPlayer.Name then
            _G.ForceDeath(Root, Humanoid)
            OrionLib:MakeNotification({
                Name = "Cible Touchée",
                Content = Player.DisplayName.." a été forcé à mourir.",
                Image = "rbxassetid://4483345998",
                Time = 3
            })
        end
    end)

    task.wait(_G.KillDelay)
end

function _G.LoopKillFunction()
    local Char = Players.LocalPlayer.Character
    if not Char or not Char:FindFirstChild("HumanoidRootPart") then return end
    

    _G.SavedCFrame = Char:GetPivot()

    for _, Name in ipairs(_G.PlayerToAddList) do
        if not _G.LoopKill then break end
        local Player = Players:FindFirstChild(Name)
        if Player then
            _G.ExecuteKill(Player)
        end
    end
end

function _G.StartLoopKill()
    if _G.LoopKillConnection then _G.LoopKillConnection:Disconnect() end
    _G.LoopKillConnection = RunService.Heartbeat:Connect(function()
        if _G.LoopKill then
            _G.LoopKillFunction()
        else
            _G.StopLoopKill() 
        end
    end)
    OrionLib:MakeNotification({
        Name = "Loop Kill Activé",
        Content = "La boucle de destruction est lancée.",
        Image = "rbxassetid://4483345998",
        Time = 3
    })
end

function _G.StopLoopKill()
    _G.LoopKill = false
    if _G.LoopKillConnection then
        _G.LoopKillConnection:Disconnect()
        _G.LoopKillConnection = nil
    end
    _G.SavedCFrame = nil
    OrionLib:MakeNotification({
        Name = "Loop Kill Désactivé",
        Content = "La boucle de destruction est arrêtée.",
        Image = "rbxassetid://4483345998",
        Time = 3
    })
end



SettingsTab:AddSection({
    Name = "Loop Kill Activation"
})

SettingsTab:AddToggle({
    Name = "Activer Loop Kill",
    Default = false,
    Callback = function(Value)
        _G.LoopKill = Value
        if Value then
            _G.StartLoopKill()
        else
            _G.StopLoopKill()
        end
    end
})

SettingsTab:AddSlider({
    Name = "Délai de Kill (sec)",
    Default = 0.5,
    Min = 0.1,
    Max = 2,
    Rounding = 1,
    Callback = function(Value)
        _G.KillDelay = Value
    end
})

SettingsTab:AddSection({
    Name = "Options Avancées"
})

SettingsTab:AddSlider({
    Name = "Décalage de TP Y",
    Default = -6,
    Min = -20,
    Max = 20,
    Rounding = 1,
    Callback = function(Value)
        _G.KillOffset = Vector3.new(0, Value, 0)
    end
})


local DropdownOptions = {} 
local CurrentTargetName = nil 
local PlayerListLabel 
local DropdownControl 


local function refreshPlayerListLabel()
    local text = ""
    if #_G.PlayerToAddList == 0 then
        text = "Liste vide."
    else
        for i, name in ipairs(_G.PlayerToAddList) do
            text = text .. i .. ". " .. name .. "\n"
        end
    end
    PlayerListLabel:Set(text)
end


local function refreshDropdown()
    DropdownOptions = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            table.insert(DropdownOptions, player.Name)
        end
    end
    if DropdownControl then
        DropdownControl:Refresh(DropdownOptions)
        CurrentTargetName = DropdownOptions[1] or nil
    end
    if not CurrentTargetName then
         OrionLib:MakeNotification({
            Name = "Actualisation",
            Content = "Aucun joueur trouvé à cibler.",
            Image = "rbxassetid://4483345998",
            Time = 3
        })
    end
end

LoopPlayersTab:AddSection({
    Name = "Gestion des Cibles"
})

LoopPlayersTab:AddButton({
    Name = "Actualiser la Liste des Joueurs",
    Callback = function()
        refreshDropdown()
        OrionLib:MakeNotification({
            Name = "Actualisation",
            Content = "La liste des joueurs a été mise à jour.",
            Image = "rbxassetid://4483345998",
            Time = 3
        })
    end
})

DropdownControl = LoopPlayersTab:AddDropdown({
    Name = "Sélectionner Cible",
    Default = "Actualiser la liste...",
    Options = DropdownOptions,
    Callback = function(Value)
        CurrentTargetName = Value
    end
})

LoopPlayersTab:AddButton({
    Name = "Ajouter à la Liste de Kill",
    Callback = function()
        if CurrentTargetName and CurrentTargetName ~= "Actualiser la liste..." then
            if not table.find(_G.PlayerToAddList, CurrentTargetName) then
                table.insert(_G.PlayerToAddList, CurrentTargetName)
                refreshPlayerListLabel()
                OrionLib:MakeNotification({
                    Name = "Ajout Cible",
                    Content = CurrentTargetName.." ajouté(e) à la liste de kill.",
                    Image = "rbxassetid://4483345998",
                    Time = 3
                })
            else
                OrionLib:MakeNotification({
                    Name = "Attention",
                    Content = CurrentTargetName.." est déjà dans la liste.",
                    Image = "rbxassetid://4483345998",
                    Time = 3
                })
            end
        end
    end
})

LoopPlayersTab:AddSection({
    Name = "Joueurs Ciblés ("..#_G.PlayerToAddList..")"
})

PlayerListLabel = LoopPlayersTab:AddLabel("Liste vide.")

LoopPlayersTab:AddButton({
    Name = "Supprimer de la Liste de Kill",
    Callback = function()
        if CurrentTargetName and CurrentTargetName ~= "Actualiser la liste..." then
            local index = table.find(_G.PlayerToAddList, CurrentTargetName)
            if index then
                table.remove(_G.PlayerToAddList, index)
                refreshPlayerListLabel()
                OrionLib:MakeNotification({
                    Name = "Suppression Cible",
                    Content = CurrentTargetName.." supprimé(e) de la liste.",
                    Image = "rbxassetid://4483345998",
                    Time = 3
                })
            else
                OrionLib:MakeNotification({
                    Name = "Attention",
                    Content = CurrentTargetName.." n'est pas dans la liste.",
                    Image = "rbxassetid://4483345998",
                    Time = 3
                })
            end
        end
    end
})

local CustomLineTab = Window:MakeTab({
    Name = "Lag all",
    Icon = "rbxassetid://6034170707", 
    PremiumOnly = false
})

Players = game:GetService("Players")
ReplicatedStorage = game:GetService("ReplicatedStorage")
RunService = game:GetService("RunService")
Workspace = game:GetService("Workspace")

_G.LagServer = false
_G.LagIntensity = 100
_G.LagConnection = nil

CustomLineTab:AddToggle({
    Name = "Lag All",
    Default = false,
    Callback = function(state)
        _G.LagServer = state
        if state then
            _G.LagConnection = RunService.Stepped:Connect(function()
                if not _G.LagServer then return end
                local spawn = Workspace:FindFirstChild("SpawnLocation", true)
                if not spawn then return end
                for _ = 1, _G.LagIntensity do
                    task.spawn(function()
                        ReplicatedStorage.GrabEvents.CreateGrabLine:FireServer(
                            spawn, 
                            CFrame.new(spawn.Position.X, 1e9, spawn.Position.Z)
                        )
                    end)
                end
            end)
        else
            if _G.LagConnection then
                _G.LagConnection:Disconnect()
                _G.LagConnection = nil
            end
        end
    end
})

CustomLineTab:AddSlider({
    Name = "Lag Intensity",
    Min = 1,
    Max = 400,
    Default = 400,
    Increment = 1,
    ValueName = "Lines",
    Callback = function(value)
        _G.LagIntensity = value
    end
})

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

local CreateGrabLine = ReplicatedStorage.GrabEvents.CreateGrabLine

_G.CrazyLine = false
_G.CrazyConnection = nil

CustomLineTab:AddToggle({
    Name = "Crazy Line (Light Crash Delay)",
    Default = false,
    Callback = function(state)
        _G.CrazyLine = state
        if state then
            _G.CrazyConnection = RunService.Heartbeat:Connect(function()
                if not _G.CrazyLine then return end
                for _, player in ipairs(Players:GetPlayers()) do
                    local character = player.Character
                    if character then
                        local hrp = character:FindFirstChild("HumanoidRootPart")
                        if hrp then
                            for _ = 1, 10 do
                                task.spawn(function()
                                    CreateGrabLine:FireServer(hrp, hrp.CFrame * CFrame.new(0, math.random(300, 700), 0))
                                end)
                            end
                            task.wait(1.2)
                        end
                    end
                end
            end)
        else
            if _G.CrazyConnection then
                _G.CrazyConnection:Disconnect()
                _G.CrazyConnection = nil
            end
        end
    end
})

_G.InvisibleConnection = nil


task.spawn(refreshDropdown)



local TextChatService = game:GetService("TextChatService")
local player = Players.LocalPlayer

task.wait(0) 

if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
    local channel = TextChatService:FindFirstChild("TextChannels") and TextChatService.TextChannels:FindFirstChild("RBXGeneral")
    if channel then
        channel:SendAsync("👑 King Hub Loaded 👑 By @SuPraa006")
    else
        warn("[Caca Hub] Impossible de trouver le canal de chat.")
    end
else
    game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
        Text = "👑 King Hub Loaded 👑 By @SuPraa006",
        Color = Color3.fromRGB(255, 200, 0)
    })
end
