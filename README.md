-- Dandy's World
local Poltergeist = loadstring(game:HttpGet("https://raw.githubusercontent.com/berhddb/Library/refs/heads/main/Main", true))()

local function sound(id)
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://"..id
    sound.Parent = workspace
    sound.Volume = 1
    sound:Play()
    sound.Ended:Connect(function()
        sound:Destroy()
    end)
end

-- Create main window
local Window = Poltergeist:CreateWindow({
    Name = "Poltergeist Hub",
    Subtitle = "Universal",
    LogoID = "101636243364181",
    LoadingEnabled = true,
    LoadingTitle = "Poltergeist Hub", 
    LoadingSubtitle = "V1.0 (BETA)",
    ConfigSettings = {
        RootFolder = "PoltergeistHubDW",
        ConfigFolder = "PoltergeistHubUniversal"
    },
    KeySystem = false
})

-- Create main tab
local Tab = Window:CreateHomeTab({
    Icon = 1,
    SupportedExecutors = {"Xeno", "Sonar", "Solara", "Swift", "Argon"},
    DiscordInvite = "c5mZwdzbHx"
})

-- Create Visual tab
local VisualTab = Window:CreateTab({
    Name = "Visual",
    Icon = "visibility",
    ImageSource = "Material",
    ShowTitle = true
})

-- Generator ESP System
local GeneratorESP = false
local GeneratorESPToggle = VisualTab:CreateToggle({
    Name = "🔧 Generator ESP",
    Description = "Highlights incomplete generators",
    CurrentValue = false,
    Callback = function(v)
        GeneratorESP = v
        if GeneratorESP then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Generator ESP has been activated"
            })
            coroutine.wrap(function()
                while GeneratorESP do
                    UpdateGeneratorESP()
                    wait(1)
                end
            end)()
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Generator ESP has been deactivated"
            })
            CleanUpGeneratorESP()
        end
    end
})

function UpdateGeneratorESP()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Name == "Generator" then
            pcall(function()
                local stats = obj:FindFirstChild("Stats")
                if stats and stats:FindFirstChild("Completed") then
                    if not stats.Completed.Value then
                        if not obj:FindFirstChild("GeneratorHighlight") then
                            local highlight = Instance.new("Highlight")
                            highlight.Name = "GeneratorHighlight"
                            highlight.FillColor = Color3.fromRGB(0, 255, 0)
                            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                            highlight.Parent = obj
                            
                            local billboard = Instance.new("BillboardGui", obj)
                            billboard.Name = "GeneratorESP"
                            billboard.Size = UDim2.new(1, 200, 1, 30)
                            billboard.AlwaysOnTop = true
                            billboard.Adornee = obj:FindFirstChild("Main") or obj.PrimaryPart
                            billboard.ExtentsOffset = Vector3.new(0, 2, 0)
                            
                            local label = Instance.new("TextLabel", billboard)
                            label.Text = "Generator"
                            label.Size = UDim2.new(1, 0, 1, 0)
                            label.Font = Enum.Font.SourceSansBold
                            label.TextSize = 12
                            label.TextColor3 = Color3.fromRGB(255, 255, 255)
                            label.BackgroundTransparency = 1
                        end
                    else
                        CleanUpGeneratorESP(obj)
                    end
                end
            end)
        end
    end
    
    if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Head") then
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("Model") and obj.Name == "Generator" and obj:FindFirstChild("GeneratorESP") then
                pcall(function()
                    local root = obj:FindFirstChild("Main") or obj.PrimaryPart
                    if root then
                        local dist = (game.Players.LocalPlayer.Character.Head.Position - root.Position).Magnitude
                        obj.GeneratorESP.TextLabel.Text = string.format("Generator [%.1fm]", dist)
                    end
                end)
            end
        end
    end
end

function CleanUpGeneratorESP(specificObj)
    if specificObj then
        local highlight = specificObj:FindFirstChild("GeneratorHighlight")
        if highlight then highlight:Destroy() end
        local esp = specificObj:FindFirstChild("GeneratorESP")
        if esp then esp:Destroy() end
    else
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("Model") and obj.Name == "Generator" then
                local highlight = obj:FindFirstChild("GeneratorHighlight")
                if highlight then highlight:Destroy() end
                local esp = obj:FindFirstChild("GeneratorESP")
                if esp then esp:Destroy() end
            end
        end
    end
end

-- Twisted ESP System
local TwistedESP = false
local TwistedESPToggle = VisualTab:CreateToggle({
    Name = "🩸 Twisted ESP",
    Description = "Highlights Twisteds with a red glow",
    CurrentValue = false,
    Callback = function(v)
        TwistedESP = v
        if TwistedESP then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Twisters ESP has been activated"
            })
            coroutine.wrap(function()
                while TwistedESP do
                    UpdateTwistedESP()
                    wait(1)
                end
            end)()
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Twisters ESP has been deactivated"
            })
            CleanUpTwistedESP()
        end
    end
})

function UpdateTwistedESP()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Parent.Name == "Monsters" then
            pcall(function()
                if not obj:FindFirstChild("TwistedHighlight") then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "TwistedHighlight"
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.FillTransparency = 0.4
                    highlight.Parent = obj
                    
                    local billboard = Instance.new("BillboardGui", obj)
                    billboard.Name = "TwistedESP"
                    billboard.Size = UDim2.new(1, 200, 1, 30)
                    billboard.AlwaysOnTop = true
                    billboard.Adornee = obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart
                    billboard.ExtentsOffset = Vector3.new(0, 3, 0)
                    
                    local label = Instance.new("TextLabel", billboard)
                    label.Text = obj.Name
                    label.Size = UDim2.new(1, 0, 1, 0)
                    label.Font = Enum.Font.SourceSansBold
                    label.TextSize = 14
                    label.TextColor3 = Color3.fromRGB(255, 50, 50)
                    label.BackgroundTransparency = 1
                end
            end)
        end
    end
    
    if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Head") then
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("Model") and obj:FindFirstChild("TwistedESP") then
                pcall(function()
                    local root = obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart
                    if root then
                        local dist = (game.Players.LocalPlayer.Character.Head.Position - root.Position).Magnitude
                        obj.TwistedESP.TextLabel.Text = string.format(obj.name.." [%.1fm]", dist)
                    end
                end)
            end
        end
    end
end

function CleanUpTwistedESP()
    for _, obj in ipairs(workspace:GetDescendants()) do
        local highlight = obj:FindFirstChild("TwistedHighlight")
        if highlight then highlight:Destroy() end
        local esp = obj:FindFirstChild("TwistedESP")
        if esp then esp:Destroy() end
    end
end

-- Item ESP System
local ItemESP = false
local itemCache = {}

local function FindItemsFolder()
    if workspace:FindFirstChild("CurrentRoom") then
        for _, child in ipairs(workspace.CurrentRoom:GetDescendants()) do
            if child.Name == "Items" and child:IsA("Folder") then
                return child
            end
        end
    end
    return nil
end

local ItemEspToggle = VisualTab:CreateToggle({
    Name = "🔍 Item ESP",
    Description = "Highlights items",
    CurrentValue = false,
    Callback = function(v)
        ItemESP = v
        if ItemESP then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Item ESP has been activated"
            })
            coroutine.wrap(function()
                while ItemESP do
                    UpdateItemESP()
                    wait(1)
                end
            end)()
        else
            CleanUpItemESP()
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Item ESP has been deactivated"
            })
        end
    end
})

function UpdateItemESP()
    local itemsFolder = FindItemsFolder()
    if not itemsFolder then return end

    for item, _ in pairs(itemCache) do
        if not item.Parent then
            itemCache[item] = nil
        end
    end

    for _, obj in ipairs(itemsFolder:GetChildren()) do
        if obj:IsA("Model") and obj:FindFirstChild("Prompt") and not itemCache[obj] then
            pcall(function()
                local highlight = Instance.new("Highlight")
                highlight.Name = "ItemHighlight"
                highlight.FillColor = Color3.fromRGB(255, 215, 0)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                highlight.FillTransparency = 0.3
                highlight.Parent = obj
                
                local billboard = Instance.new("BillboardGui", obj)
                billboard.Name = "ItemESP"
                billboard.Size = UDim2.new(1, 200, 1, 30)
                billboard.AlwaysOnTop = true
                billboard.Adornee = obj:FindFirstChild("Main") or obj.PrimaryPart
                billboard.ExtentsOffset = Vector3.new(0, 2, 0)
                
                local label = Instance.new("TextLabel", billboard)
                label.Text = obj.Name
                label.Size = UDim2.new(1, 0, 1, 0)
                label.Font = Enum.Font.SourceSansBold
                label.TextSize = 12
                label.TextColor3 = Color3.fromRGB(255, 215, 0)
                label.BackgroundTransparency = 1
                
                itemCache[obj] = true
            end)
        end
    end
    
    if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Head") then
        for obj, _ in pairs(itemCache) do
            if obj.Parent and obj:FindFirstChild("ItemESP") then
                pcall(function()
                    local root = obj:FindFirstChild("Main") or obj.PrimaryPart
                    if root then
                        local dist = (game.Players.LocalPlayer.Character.Head.Position - root.Position).Magnitude
                        obj.ItemESP.TextLabel.Text = string.format("%s [%.1fm]", obj.Name, dist)
                    end
                end)
            end
        end
    end
end

function CleanUpItemESP()
    for obj, _ in pairs(itemCache) do
        if obj.Parent then
            local highlight = obj:FindFirstChild("ItemHighlight")
            if highlight then highlight:Destroy() end
            local esp = obj:FindFirstChild("ItemESP")
            if esp then esp:Destroy() end
        end
    end
    itemCache = {}
end

-- Player ESP System
local PlayerEsp = false
local PlayerEspToggle = VisualTab:CreateToggle({
    Name = "👤 Player ESP",
    Description = "Highlights alive players",
    CurrentValue = false,
    Callback = function(v)
        PlayerEsp = v
        if PlayerEsp then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Player ESP has been activated"
            })
            coroutine.wrap(function()
                while PlayerEsp do
                    UpdatePlayerEsp()
                    wait(1)
                end
            end)()
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Player ESP has been deactivated"
            })
            CleanUpPlayerEsp()
        end
    end
})

function UpdatePlayerEsp()
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            local obj = player.Character
            pcall(function()
                if not obj:FindFirstChild("PlayerHighlight") then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "PlayerHighlight"
                    highlight.FillColor = Color3.fromRGB(0, 0, 255)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.Parent = obj
                    
                    local billboard = Instance.new("BillboardGui", obj)
                    billboard.Name = "PlayerEsp"
                    billboard.Size = UDim2.new(1, 200, 1, 30)
                    billboard.AlwaysOnTop = true
                    billboard.Adornee = obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart
                    billboard.ExtentsOffset = Vector3.new(0, 2, 0)
                    
                    local label = Instance.new("TextLabel", billboard)
                    label.Text = player.Name
                    label.Size = UDim2.new(1, 0, 1, 0)
                    label.Font = Enum.Font.SourceSansBold
                    label.TextSize = 12
                    label.TextColor3 = Color3.fromRGB(255, 255, 255)
                    label.BackgroundTransparency = 1
                end
            end)
        end
    end
    
    if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Head") then
        for _, player in ipairs(game.Players:GetPlayers()) do
            if player ~= game.Players.LocalPlayer and player.Character then
                local obj = player.Character
                if obj:FindFirstChild("PlayerEsp") then
                    pcall(function()
                        local root = obj:FindFirstChild("HumanoidRootPart") or obj.PrimaryPart
                        if root then
                            local dist = (game.Players.LocalPlayer.Character.Head.Position - root.Position).Magnitude
                            obj.PlayerEsp.TextLabel.Text = string.format(player.Name.." [%.1fm]", dist)
                        end
                    end)
                end
            end
        end
    end
end

function CleanUpPlayerEsp()
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player.Character then
            local obj = player.Character
            local highlight = obj:FindFirstChild("PlayerHighlight")
            if highlight then highlight:Destroy() end
            local esp = obj:FindFirstChild("PlayerEsp")
            if esp then esp:Destroy() end
        end
    end
end

NVPU = false
local NoVeePopUps = VisualTab:CreateButton({
    Name = "🖥️ No Vee pop-ups",
    Description = "Disable pop-ups of twisted Vee",
    Callback = function()
        if NVPU == false then
            NVPU = true
            if game.Players.LocalPlayer.PlayerGui:FindFirstChild("ScreenGui") then
                if game.Players.LocalPlayer.PlayerGui.ScreenGui:FindFirstChild("PopUp") then
                    game.Players.LocalPlayer.PlayerGui.ScreenGui.PopUp:Destroy()
                end
            end
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "No Vee pop-ups has been activated"
            })
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "No Vee pop-ups is already activated"
            })
        end
    end
})

-- Create General tab
local GeneralTab = Window:CreateTab({
    Name = "General",
    Icon = "calendar_today",
    ImageSource = "Material",
    ShowTitle = true
})

local ASCValue = false
local AutoSkillCheck = GeneralTab:CreateButton({
    Name = "🤹‍♀️ Auto Skill Check",
    Description = "Automatically completes skill checks (the skill check dont appear)",
    Callback = function()
        if ASCValue == false then
            ASCValue = true
            game.ReplicatedStorage.Events.SkillcheckUpdate.OnClientInvoke = function() return "supercomplete" end
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Auto Skill Check has been activated"
            })
            task.wait(0.2)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "The skill check will not appear!"
            })
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Auto Skill Check is already activated"
            })
        end
    end
})

local FullBright = GeneralTab:CreateButton({
    Name = "💡 Fullbright",
    Description = "Remove game darkness",
    Callback = function()
        if game.Lighting.Brightness ~= 2 then 
            local Lighting = game:GetService("Lighting")
            Lighting.Brightness = 2
            Lighting.ClockTime = 14
            Lighting.FogEnd = 100000
            Lighting.GlobalShadows = false
            Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Fullbright has been activated"
            })
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Fullbright is already activated"
            })
        end
    end
})

local disFB = GeneralTab:CreateButton({
    Name = "❌💡 Disable fullbright",
    Description = "Disable the fullbright effect",
    Callback = function()
        if game.Lighting.Brightness == 2 then 
            local Lighting = game:GetService("Lighting")
            Lighting.Brightness = 3
            Lighting.ClockTime = 0
            Lighting.FogEnd = 250
            Lighting.GlobalShadows = true
            Lighting.OutdoorAmbient = Color3.fromRGB(0, 0, 0)
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Fullbright disabled"
            })
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Fullbright is not activated"
            })
        end
    end
})

local function fbwbodetect()
    if workspace.Info.BlackOut.Value == true then
        local Lighting = game:GetService("Lighting")
        Lighting.Brightness = 2
        Lighting.ClockTime = 14
        Lighting.FogEnd = 100000
        Lighting.GlobalShadows = false
        Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
    end
end

local FBwBO = false
local FBwBOToggle = GeneralTab:CreateToggle({
    Name = "🔦 Fullbright when blackout",
    Description = "Automatically toggles fullbright when blackout detected",
    CurrentValue = false,
    Callback = function(v)
        FBwBO = v
        if FBwBO then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Fullbright when blackout has been activated"
            })
            coroutine.wrap(function()
                while FBwBO do
                    fbwbodetect()
                    wait(1)
                end
            end)()
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Fullbright when blackout has been deactivated"
            })
        end
    end
})

local AutoGTE = false
local AutoGTEToggle = GeneralTab:CreateToggle({
    Name = "🏃 Auto GTE",
    Description = "Automatically teleports you to elevator when panic mode",
    CurrentValue = false,
    Callback = function(v)
        AutoGTE = v
        if AutoGTE then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Auto GTE has been activated"
            })
            coroutine.wrap(function()
                local workspace = game:GetService("Workspace")
                local players = game:GetService("Players")
                local infoFolder = workspace:FindFirstChild("Info")
                if infoFolder then
                    local panicBool = infoFolder:FindFirstChild("Panic")
                    if panicBool and panicBool:IsA("BoolValue") then
                        while AutoGTE do
                            if panicBool.Value == true then       
                                local elevatorsFolder = workspace:FindFirstChild("Elevators")              
                                if elevatorsFolder then 
                                    local elevatorModel = elevatorsFolder:FindFirstChildWhichIsA("Model")
                                    if elevatorModel then  
                                        local monsterBlocker = elevatorModel:FindFirstChild("MonsterBlocker")    
                                        if monsterBlocker and monsterBlocker:IsA("Part") then
                                            local character = game.Players.LocalPlayer.Character
                                            if character and character:FindFirstChild("HumanoidRootPart") then
                                                character.HumanoidRootPart.CFrame = monsterBlocker.CFrame
                                            end
                                        end
                                    end
                                end
                            end
                            wait(0.5)
                        end
                    end
                end
            end)()
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Auto GTE has been deactivated"
            })
        end
    end
})

-- Create Player tab
local PlayerTab = Window:CreateTab({
    Name = "Player",
    Icon = "account_circle",
    ImageSource = "Material",
    ShowTitle = true
})

local tpWalking = false
local localPlayer = game:GetService("Players").LocalPlayer

local function StartTeleportWalk(speedMultiplier)
    tpWalking = true
    
    coroutine.wrap(function()
        local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
        local humanoid = character:WaitForChild("Humanoid")
        
        while tpWalking and character and humanoid and humanoid.Parent do
            local delta = game:GetService("RunService").Heartbeat:Wait()
            if humanoid.MoveDirection.Magnitude > 0 then
                character:TranslateBy(humanoid.MoveDirection * speedMultiplier * delta * 10)
            end
        end
    end)()
end

local function StopTeleportWalk()
    tpWalking = false
end

local TpWalkSpeedSlider = PlayerTab:CreateSlider({
    Name = "Teleport Walk Speed",
    Description = "Adjust movement speed multiplier",
    Range = {0.1, 5},
    Increment = 0.1,
    CurrentValue = 0.5,
    Callback = function(Value)
        if TpWalkToggle and TpWalkToggle.CurrentValue then
            StopTeleportWalk()
            StartTeleportWalk(Value)
        end
    end
})

local TpWalkToggle = PlayerTab:CreateToggle({
    Name = "🚀 Teleport Walk",
    Description = "Enhanced movement speed",
    CurrentValue = false,
    Callback = function(Value)
        if Value then
            sound(8486683243)
            Poltergeist:Notification({
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Teleport Walk activated"
            })
            StartTeleportWalk(TpWalkSpeedSlider.CurrentValue)
        else
            sound(17208361335)
            Poltergeist:Notification({
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Teleport Walk deactivated"
            })
            StopTeleportWalk()
        end
    end
})

localPlayer.CharacterAdded:Connect(function()
    if TpWalkToggle and TpWalkToggle.CurrentValue then
        StopTeleportWalk()
        wait(0.5)
        StartTeleportWalk(TpWalkSpeedSlider.CurrentValue)
    end
end)

local Noclip = false
local NoclipToggle = PlayerTab:CreateToggle({
    Name = "👻 Noclip",
    Description = "turns you into a POLTERGEIST bOooOooOoo",
    CurrentValue = false,
    Callback = function(v)
        Noclip = v
        if Noclip then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Noclip has been activated"
            })
            local player = game.Players.LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

            local function onTouched(part)
                if part:IsA("BasePart") and part.CanCollide then
                    if part:IsDescendantOf(character) then
                        return
                    end

                    local playerPosition = humanoidRootPart.Position
                    local partPosition = part.Position

                    if partPosition.Y < playerPosition.Y - humanoidRootPart.Size.Y / 2 then
                        return
                    end
                    if Noclip == true then
                        local noclipBool = Instance.new("BoolValue")
                        noclipBool.Name = "NoclipBool"
                        noclipBool.Parent = part
                        part.CanCollide = false
                    end
                end
            end
            humanoidRootPart.Touched:Connect(onTouched)
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Noclip has been deactivated"
            })
            for i, v in ipairs(workspace:GetDescendants()) do
                if v.Name == "NoclipBool" then
                    v.Parent.CanCollide = true
                    Noclip = false
                    v:Destroy() 
                end
            end
        end
    end
})

local GodMode = false
local GodModeToggle = PlayerTab:CreateToggle({
    Name = "🌌 God Mode",
    Description = "Makes you immune to twisteds (thx G0byD0llan for the inspiration to make this function)",
    CurrentValue = false,
    Callback = function(v)
        GodMode = v
        if GodMode then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "God Mode has been activated"
            })
            local function LPGettingChased()
                local currentRoom = game.Workspace:FindFirstChild("CurrentRoom")
                if not currentRoom then 
                    warn("CurrentRoom not found in Workspace")
                    return false
                end
                for _, child in ipairs(currentRoom:GetChildren()) do
                    if child:IsA("Model") then
                        local monstersContainer = child:FindFirstChild("Monsters")
                        if monstersContainer then
                            for _, monster in ipairs(monstersContainer:GetChildren()) do
                                local chasingValue = monster:FindFirstChild("ChasingValue")
                                if chasingValue and chasingValue:IsA("ObjectValue") then
                                    local player = game.Players.LocalPlayer
                                    if chasingValue.Value == player or chasingValue.Value == player.Character then
                                        return true
                                    end
                                end
                            end
                        end
                    end
                end
                return false
            end
            local function CHH(height)
                local player = game.Players.LocalPlayer
                local character = player.Character or player.CharacterAdded:Wait()
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.HipHeight = height
                end
            end
                    coroutine.wrap(function()
                while GodMode do
                    if LPGettingChased() then
                     local currentRoom = game.Workspace:WaitForChild("CurrentRoom")
                    if currentRoom.Monsters:FindFirstChild("DandyMonster") or currentRoom.Monsters:FindFirstChild("VeeMonster") or currentRoom.Monsters:FindFirstChild("SproutMonster") or currentRoom.Monsters:FindFirstChild("ShellyMonster") or currentRoom.Monsters:FindFirstChild("PebbleMonster") then
                     CHH(7)
                    else
                                        CHH(5.5)
                                        end
                    else
                        CHH(1.41)
                    end
                    task.wait(0.1)
                end
            end)()
        else
            local function CHH(height)
                local player = game.Players.LocalPlayer
                local character = player.Character or player.CharacterAdded:Wait()
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.HipHeight = height
                end
            end
            CHH(1.41)
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "God Mode has been deactivated"
            })
        end
    end
})

local JumpValue = false
local JumpBTN = PlayerTab:CreateButton({
    Name = "🦘 Jump button",
    Description = "Creates a bypass jump button (not made by me, created by FoxcatLol on Discord.)",
    Callback = function()
        if JumpValue == false then
            JumpValue = true
            local UIS = game:GetService("UserInputService")
            local Players = game:GetService("Players")

            local player = Players.LocalPlayer
            local char = player.Character or player.CharacterAdded:Wait()
            local humanoidRootPart = char:WaitForChild("HumanoidRootPart")
            local humanoid = char:WaitForChild("Humanoid")

            local button = Instance.new("TextButton")
            button.Size = UDim2.new(0, 75, 0, 75)
            button.Position = UDim2.new(0.5, -50, 0.8, 0)
            button.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            button.Text = "Jump"
            button.TextColor3 = Color3.fromRGB(255, 255, 255)
            button.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui"):FindFirstChildOfClass("ScreenGui") or Instance.new("ScreenGui", game.Players.LocalPlayer.PlayerGui)

            local dragging, dragInput, dragStart, startPos

            button.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                    dragging = true
                    dragStart = input.Position
                    startPos = button.Position
                    input.Changed:Connect(function()
                        if input.UserInputState == Enum.UserInputState.End then
                            dragging = false
                        end
                    end)
                end
            end)

            button.InputChanged:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                    dragInput = input
                end
            end)

            UIS.InputChanged:Connect(function(input)
                if input == dragInput and dragging then
                    local delta = input.Position - dragStart
                    button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
                end
            end)

            local function isOnGround()
                return humanoid:GetState() == Enum.HumanoidStateType.Seated or 
                       humanoid:GetState() == Enum.HumanoidStateType.Running or 
                       humanoid:GetState() == Enum.HumanoidStateType.Landed
            end

            button.MouseButton1Click:Connect(function()
                if isOnGround() then
                    humanoidRootPart.AssemblyLinearVelocity = Vector3.new(0, 75, 0)
                    task.wait(0.1)
                end
            end)
            
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Jump button has been activated"
            })
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Jump button is already activated"
            })
        end
    end
})

-- Create Others tab
local OthersTab = Window:CreateTab({
    Name = "Others",
    Icon = "add_circle",
    ImageSource = "Material",
    ShowTitle = true
})

local GDscript = OthersTab:CreateButton({
    Name = "G0bbyD0llan script",
    Description = "Loads G0bbyD0llan's script (Great script!! ,'D)",
    Callback = function()
          loadstring(game:HttpGet("https://pastebin.com/raw/FBRnb7S7"))()
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Script loaded!"
            })
        
    end
})

local GACscript = OthersTab:CreateButton({
    Name = "Glisten's animation closet",
    Description = "Loads Glisten's animation closet (dw animations XD)",
    Callback = function()
         loadstring(game:HttpGet("https://raw.githubusercontent.com/RodeStriker/TheDandyHelper/refs/heads/main/GAC"))()
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Script loaded!"
            })
        
    end
})

local Nscript = OthersTab:CreateButton({
    Name = "Noxious Hub",
    Description = "Loads Noxious Hub (Made by Unable :o)",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/NXSendeavor/endeavor/refs/heads/main/DSWDendeavor"))()
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Script loaded!"
            })
        
    end
})

