-- Dandy's World
local Poltergeist = loadstring(game:HttpGet("https://raw.githubusercontent.com/berhddb/Library/refs/heads/main/Main", true))()
local VirtualInputManager = game:GetService('VirtualInputManager')
local player = game.Players.LocalPlayer

-- Sound function
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

-- Create tabs
local VisualTab = Window:CreateTab({
    Name = "Visual",
    Icon = "visibility",
    ImageSource = "Material",
    ShowTitle = true
})

local GeneralTab = Window:CreateTab({
    Name = "General",
    Icon = "calendar_today",
    ImageSource = "Material",
    ShowTitle = true
})

local PlayerTab = Window:CreateTab({
    Name = "Player",
    Icon = "account_circle",
    ImageSource = "Material",
    ShowTitle = true
})

local OthersTab = Window:CreateTab({
    Name = "Others",
    Icon = "add_circle",
    ImageSource = "Material",
    ShowTitle = true
})

-- ESP Utility Functions
local function createESP(object, name, color, offset)
    if not object then return end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = name.."Highlight"
    highlight.FillColor = color
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.3
    highlight.Parent = object
    
    local billboard = Instance.new("BillboardGui")
    billboard.Name = name.."ESP"
    billboard.Size = UDim2.new(1, 200, 1, 30)
    billboard.AlwaysOnTop = true
    billboard.Adornee = object:FindFirstChild("HumanoidRootPart") or object:FindFirstChild("Main") or object.PrimaryPart
    billboard.ExtentsOffset = Vector3.new(0, offset or 2, 0)
    billboard.Parent = object
    
    local label = Instance.new("TextLabel")
    label.Text = name
    label.Size = UDim2.new(1, 0, 1, 0)
    label.Font = Enum.Font.SourceSansBold
    label.TextSize = 12
    label.TextColor3 = color
    label.BackgroundTransparency = 1
    label.Parent = billboard
    
    return highlight, billboard
end

local function updateDistanceESP(object, name)
    if not object or not object:FindFirstChild(name.."ESP") then return end
    
    local root = object:FindFirstChild("HumanoidRootPart") or object:FindFirstChild("Main") or object.PrimaryPart
    if not root then return end
    
    local localHead = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Head")
    if not localHead then return end
    
    local dist = (localHead.Position - root.Position).Magnitude
    object[name.."ESP"].TextLabel.Text = string.format("%s [%.1fm]", name, dist)
end

-- Generator ESP System
local GeneratorESP = false
local generatorCache = {}

local function UpdateGeneratorESP()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Name == "Generator" then
            pcall(function()
                local stats = obj:FindFirstChild("Stats")
                if stats and stats:FindFirstChild("Completed") then
                    if not stats.Completed.Value and not generatorCache[obj] then
                        createESP(obj, "Generator", Color3.fromRGB(0, 255, 0))
                        generatorCache[obj] = true
                    elseif stats.Completed.Value and generatorCache[obj] then
                        if obj:FindFirstChild("GeneratorHighlight") then obj.GeneratorHighlight:Destroy() end
                        if obj:FindFirstChild("GeneratorESP") then obj.GeneratorESP:Destroy() end
                        generatorCache[obj] = nil
                    end
                    
                    if generatorCache[obj] then
                        updateDistanceESP(obj, "Generator")
                    end
                end
            end)
        end
    end
end

local function CleanUpGeneratorESP()
    for obj in pairs(generatorCache) do
        if obj.Parent then
            if obj:FindFirstChild("GeneratorHighlight") then obj.GeneratorHighlight:Destroy() end
            if obj:FindFirstChild("GeneratorESP") then obj.GeneratorESP:Destroy() end
        end
    end
    generatorCache = {}
end

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
            
            local conn
            conn = game:GetService("RunService").Heartbeat:Connect(function()
                if not GeneratorESP then 
                    conn:Disconnect()
                    return
                end
                UpdateGeneratorESP()
            end)
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

-- Twisted ESP System
local TwistedESP = false
local twistedCache = {}

local function UpdateTwistedESP()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj.Parent and obj.Parent.Name == "Monsters" and not twistedCache[obj] then
            pcall(function()
                createESP(obj, obj.Name, Color3.fromRGB(255, 0, 0), 3)
                twistedCache[obj] = true
            end)
        end
        
        if twistedCache[obj] then
            updateDistanceESP(obj, obj.Name)
        end
    end
end

local function CleanUpTwistedESP()
    for obj in pairs(twistedCache) do
        if obj.Parent then
            if obj:FindFirstChild(obj.Name.."Highlight") then obj[obj.Name.."Highlight"]:Destroy() end
            if obj:FindFirstChild(obj.Name.."ESP") then obj[obj.Name.."ESP"]:Destroy() end
        end
    end
    twistedCache = {}
end

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
                Content = "Twisted ESP has been activated"
            })
            
            local conn
            conn = game:GetService("RunService").Heartbeat:Connect(function()
                if not TwistedESP then 
                    conn:Disconnect()
                    return
                end
                UpdateTwistedESP()
            end)
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Twisted ESP has been deactivated"
            })
            CleanUpTwistedESP()
        end
    end
})

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

local function UpdateItemESP()
    local itemsFolder = FindItemsFolder()
    if not itemsFolder then return end

    for item in pairs(itemCache) do
        if not item.Parent then
            itemCache[item] = nil
        end
    end

    for _, obj in ipairs(itemsFolder:GetChildren()) do
        if obj:IsA("Model") and obj:FindFirstChild("Prompt") and not itemCache[obj] then
            pcall(function()
                createESP(obj, obj.Name, Color3.fromRGB(255, 215, 0))
                itemCache[obj] = true
            end)
        end
        
        if itemCache[obj] then
            updateDistanceESP(obj, obj.Name)
        end
    end
end

local function CleanUpItemESP()
    for obj in pairs(itemCache) do
        if obj.Parent then
            if obj:FindFirstChild(obj.Name.."Highlight") then obj[obj.Name.."Highlight"]:Destroy() end
            if obj:FindFirstChild(obj.Name.."ESP") then obj[obj.Name.."ESP"]:Destroy() end
        end
    end
    itemCache = {}
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
            
            local conn
            conn = game:GetService("RunService").Heartbeat:Connect(function()
                if not ItemESP then 
                    conn:Disconnect()
                    return
                end
                UpdateItemESP()
            end)
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

-- Player ESP System
local PlayerEsp = false
local playerCache = {}

local function UpdatePlayerEsp()
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character and not playerCache[player] then
            pcall(function()
                createESP(player.Character, player.Name, Color3.fromRGB(0, 0, 255))
                playerCache[player] = true
            end)
        end
        
        if playerCache[player] and player.Character then
            updateDistanceESP(player.Character, player.Name)
        end
    end
end

local function CleanUpPlayerEsp()
    for player in pairs(playerCache) do
        if player.Character then
            if player.Character:FindFirstChild(player.Name.."Highlight") then 
                player.Character[player.Name.."Highlight"]:Destroy() 
            end
            if player.Character:FindFirstChild(player.Name.."ESP") then 
                player.Character[player.Name.."ESP"]:Destroy() 
            end
        end
    end
    playerCache = {}
end

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
            
            local conn
            conn = game:GetService("RunService").Heartbeat:Connect(function()
                if not PlayerEsp then 
                    conn:Disconnect()
                    return
                end
                UpdatePlayerEsp()
            end)
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

-- No Vee Pop-ups
local NVPU = false
local NoVeePopUps = VisualTab:CreateButton({
    Name = "🖥️ No Vee pop-ups",
    Description = "Disable pop-ups of twisted Vee",
    Callback = function()
        if NVPU then
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "No Vee pop-ups is already activated"
            })
            return
        end
        
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
    end
})

-- Auto Skill Check System
local ASCValue = false
local AutoSkillCheckBtn = GeneralTab:CreateButton({
    Name = "🤹‍♀️ Auto Skill Check",
    Description = "Automatically completes skill checks (the skill check dont appear)",
    Callback = function()
        if ASCValue then
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Auto Skill Check is already activated"
            })
            return
        end
        
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
    end
})

local function AutoSkillCheck()
    local screenGui = player.PlayerGui:FindFirstChild("ScreenGui")
    if not screenGui then return end

    local menu = screenGui:FindFirstChild("Menu")
    if not menu then return end

    local skillCheckFrame = menu:FindFirstChild("SkillCheckFrame")
    if not skillCheckFrame then return end

    -- Constants
    local TOLERANCE = 5 -- Adjust this value as needed

    -- Function to perform the check when skill check frame is visible
    local function onVisibilityChanged()
        if skillCheckFrame.Visible then
            local marker = skillCheckFrame:FindFirstChild("Marker")
            local goldArea = skillCheckFrame:FindFirstChild("GoldArea")

            if marker and goldArea then
                local markerPosition = marker.AbsolutePosition
                local goldAreaPosition = goldArea.AbsolutePosition
                local goldAreaSize = goldArea.AbsoluteSize

                -- Check if the Marker is within the bounds of the GoldArea
                if markerPosition.X >= goldAreaPosition.X and 
                   markerPosition.X <= (goldAreaPosition.X + goldAreaSize.X) + TOLERANCE then
                    -- Send spacebar press event
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
                    task.wait(0.1)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
                end
            end
        end
    end

    -- Connect to the VisibilityChanged event
    skillCheckFrame:GetPropertyChangedSignal("Visible"):Connect(onVisibilityChanged)

    -- Detect changes in Marker and GoldArea positions
    local marker = skillCheckFrame:FindFirstChild("Marker")
    local goldArea = skillCheckFrame:FindFirstChild("GoldArea")

    if marker then
        marker:GetPropertyChangedSignal("AbsolutePosition"):Connect(onVisibilityChanged)
    end

    if goldArea then
        goldArea:GetPropertyChangedSignal("AbsolutePosition"):Connect(onVisibilityChanged)
        goldArea:GetPropertyChangedSignal("AbsoluteSize"):Connect(onVisibilityChanged)
    end
end

-- Auto skill check 2 Feature
local SC2A = false
local SC2 = GeneralTab:CreateButton({
    Name = "🏓 Auto Skill Check 2",
    Description = "Automatically completes skill checks",
    Callback = function()
        if SC2A then
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Auto Skill Check is already activated"
            })
            return
        end
        
        SC2A = true
        AutoSkillCheck()
        sound(8486683243)
        Poltergeist:Notification({ 
            Title = "Notification",
            Icon = "notifications_active",
            ImageSource = "Material",
            Content = "Auto Skill Check has been activated."
        })
    end
})

-- Fullbright Feature
local FullBright = GeneralTab:CreateButton({
    Name = "💡 Fullbright",
    Description = "Remove game darkness",
    Callback = function()
        local Lighting = game:GetService("Lighting")
        if Lighting.Brightness ~= 2 then 
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


-- Auto GTE
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
            
            local conn
            conn = game:GetService("RunService").Heartbeat:Connect(function()
                if not AutoGTE then 
                    conn:Disconnect()
                    return
                end
                
                local infoFolder = workspace:FindFirstChild("Info")
                if not infoFolder then return end
                
                local panicBool = infoFolder:FindFirstChild("Panic")
                if not panicBool or not panicBool:IsA("BoolValue") then return end
                
                if panicBool.Value then       
                    local elevatorsFolder = workspace:FindFirstChild("Elevators")              
                    if not elevatorsFolder then return end
                    
                    local elevatorModel = elevatorsFolder:FindFirstChildWhichIsA("Model")
                    if not elevatorModel then return end
                    
                    local monsterBlocker = elevatorModel:FindFirstChild("MonsterBlocker")    
                    if not monsterBlocker or not monsterBlocker:IsA("Part") then return end
                    
                    local character = game.Players.LocalPlayer.Character
                    if not character then return end
                    
                    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                    if not humanoidRootPart then return end
                    
                    humanoidRootPart.CFrame = monsterBlocker.CFrame
                end
            end)
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

local ntfrm = false
local ntfrmToggle = GeneralTab:CreateToggle({
    Name = "❗ Notify Rare Monsters",
    Description = "Notify you when a rare monster appears",
    CurrentValue = false,
    Callback = function(v)
        ntfrm = v
        if ntfrm then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Notify Rare Monsters has been activated"
            })

            local notifiedMonsters = {}
            
            local function sendNotification(monsterName)
                Poltergeist:Notification({ 
                    Title = "Rare Monster",
                    Icon = "notifications_active",
                    ImageSource = "Material",
                    Content = "A rare monster has appeared! Monster name: " .. monsterName
                })
                sound(8486683243)
            end

            local targetMonsters = {
                "AstroMonster",
                "VeeMonster",
                "SproutMonster",
                "PebbleMonster",
                "ShellyMonster",
                "DandyMonster"
            }

            local currentRoom = game.Workspace:FindFirstChild("CurrentRoom")
            if currentRoom then
                -- Continuously check for monsters
                while ntfrm do
                    for _, model in ipairs(currentRoom:GetChildren()) do
                        if model:IsA("Model") then
                            local monstersFolder = model:FindFirstChild("Monsters")
                            if monstersFolder then
                                -- Check the monsters within Monsters folder
                                for _, monster in ipairs(monstersFolder:GetChildren()) do
                                    if table.find(targetMonsters, monster.Name) and not notifiedMonsters[monster.Name] then
                                        sendNotification(monster.Name) -- Send the notification
                                        notifiedMonsters[monster.Name] = true -- Mark the monster as notified
                                    end
                                end
                            end
                        end
                    end
                    
                    -- Clean up monsters that are no longer present
                    for monsterName, _ in pairs(notifiedMonsters) do
                        local stillExists = false
                        for _, model in ipairs(currentRoom:GetChildren()) do
                            local monstersFolder = model:FindFirstChild("Monsters")
                            if monstersFolder and monstersFolder:FindFirstChild(monsterName) then
                                stillExists = true
                                break
                            end
                        end
                        if not stillExists then
                            notifiedMonsters[monsterName] = nil -- Remove monsters that are no longer present
                        end
                    end
                    task.wait(5)
                end
            else
                warn("There is no current room in workspace!")
            end
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Notify Rare Monsters has been deactivated"
            })
        end
    end
})


-- Player Tab Features
local tpWalking = false
local localPlayer = game:GetService("Players").LocalPlayer

local function StartTeleportWalk(speedMultiplier)
    tpWalking = true
    
    local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid")
    
    local conn
    conn = game:GetService("RunService").Heartbeat:Connect(function()
        if not tpWalking or not character or not humanoid or not humanoid.Parent then 
            conn:Disconnect()
            return
        end
        
        local delta = game:GetService("RunService").Heartbeat:Wait()
        if humanoid.MoveDirection.Magnitude > 0 then
            character:TranslateBy(humanoid.MoveDirection * speedMultiplier * delta * 10)
        end
    end)
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
        task.wait(0.5)
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

-- God Mode
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
                if not currentRoom then return false end

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
            
            local conn
            conn = game:GetService("RunService").Heartbeat:Connect(function()
                if not GodMode then 
                    conn:Disconnect()
                    return
                end
                
                if LPGettingChased() then
                    CHH(5.5)
                else
                    CHH(1.41)
                end
            end)
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

-- Jump Button
local JumpValue = false
local JumpBTN = PlayerTab:CreateButton({
    Name = "🦘 Jump button",
    Description = "Creates a bypass jump button (not made by me, created by FoxcatLol on Discord.)",
    Callback = function()
        if JumpValue then
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Jump button is already activated"
            })
            return
        end
        
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
    end
})

-- Spin
local Spin = false
local SpinToggle = PlayerTab:CreateToggle({
    Name = "🌀 Spin",
    Description = "Makes you sPiNnNnNnNnNnNnNnNnNnN",
    CurrentValue = false,
    Callback = function(v)
        Spin = v
        if Spin then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Spin has been activated"
            })
            local Spin = Instance.new("BodyAngularVelocity")
	Spin.Name = "Spinning"
	Spin.Parent = player.Character.HumanoidRootPart
	Spin.MaxTorque = Vector3.new(0, math.huge, 0)
	Spin.AngularVelocity = Vector3.new(0,50,0)
        
        else
            for i,v in pairs(player.Character.HumanoidRootPart:GetChildren()) do
		if v.Name == "Spinning" then
			v:Destroy()
		end
	end
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Spin has been deactivated"
            })
        end
    end
})

-- Other Scripts
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
