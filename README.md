print("Loading Poltergeist Hub (Dandy's World)")
-- Dandy's World
local TpWalkSpeed = 0.5
local ttg = false
local Poltergeist = loadstring(game:HttpGet("https://raw.githubusercontent.com/berhddb/Library/refs/heads/main/Main", true))()
local VirtualInputManager = game:GetService('VirtualInputManager')
local player = game.Players.LocalPlayer

-- Configuration
-- generator esp
local GeneratorEspColor = 0,255,0
local ShowGeneratorDistance = false

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
    if ShowGeneratorDistance then
    local dist = (localHead.Position - root.Position).Magnitude
    object[name.."ESP"].TextLabel.Text = string.format("%s [%.1fm]", name, dist)
    end
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
                        createESP(obj, "Generator", GeneratorEspColor)
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

local ASC2 = GeneralTab:CreateToggle({
    Name = "🏓 Auto Skill Check 2",
    Description = "Automatically completes skill check",
    Value = false,
    Callback = function(state)
        local runService = game:GetService("RunService")
        local virtualInputManager = game:GetService("VirtualInputManager")
        local player = game.Players.LocalPlayer
        local ScreenGui = player:WaitForChild("PlayerGui"):FindFirstChild("ScreenGui")
        -- Declarar conexión fuera de la función
        if not _G.connection then _G.connection = nil end

        -- Verificación inicial
        if not ScreenGui then
            warn("ScreenGui not found.")
            return
        end

        local menu = ScreenGui:FindFirstChild("Menu")
        local skillCheckFrame = menu and menu:FindFirstChild("SkillCheckFrame")
        local marker = skillCheckFrame and skillCheckFrame:FindFirstChild("Marker")
        local goldArea = skillCheckFrame and skillCheckFrame:FindFirstChild("GoldArea")
        local calibrateButton = menu and menu:FindFirstChild("Calibrate")

        if not (marker and goldArea and calibrateButton) then
            warn("Required elements not found.")
            return
        end

        local timeElapsed = 0
        local checkInterval = 0.01

        -- Función para presionar la tecla espacio
        local function pressSpace()
            if skillCheckFrame.Visible then
                virtualInputManager:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
                virtualInputManager:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
            end
        end

        -- Función para comprobar la posición de los frames
        local function checkFramesPosition()
            local frame1X = marker.AbsolutePosition.X
            local frame2X = goldArea.AbsolutePosition.X
            local minRange = frame2X
            local maxRange = frame2X + 10

            if frame1X >= minRange and frame1X <= maxRange then
                pressSpace()
            end
        end
        -- Si el toggle está activado
        if state then
             sound(8486683243)
        Poltergeist:Notification({ 
            Title = "Notification",
            Icon = "notifications_active",
            ImageSource = "Material",
            Content = "Auto Skill Check has been activated"
        })
            -- Asegurarse de que no haya conexiones previas
            if _G.connection then
                _G.connection:Disconnect()
                _G.connection = nil
            end

            -- Inicia la verificación del marcador
            _G.connection = runService.RenderStepped:Connect(function(deltaTime)
                timeElapsed = timeElapsed + deltaTime
                if timeElapsed >= checkInterval then
                    timeElapsed = 0
                    checkFramesPosition()
                end
            end)
        else
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Auto Skill Check deactivated."
            })
            if _G.connection then
                _G.connection:Disconnect()
                _G.connection = nil
            end
        end
    end,
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

local ntfri = false
local ntfriToggle = GeneralTab:CreateToggle({
    Name = "‼ Notify Rare Items",
    Description = "Notify you when a rare item appears",
    CurrentValue = false,
    Callback = function(v)
        ntfri = v
        if ntfri then
            sound(8486683243)  -- Notification sound when activated
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Notify Rare Items has been activated"
            })
            local notifiedItem = {}
            local function sendNotificationItem(itemName)
                Poltergeist:Notification({ 
                    Title = "Rare Item",
                    Icon = "notifications_active",
                    ImageSource = "Material",
                    Content = "A rare item has appeared! Item name: " .. itemName
                })
                sound(8486683243)  -- Notification sound when a rare item appears
            end
            local targetItems = {
                "Bandage",
                "HealthKit",
                "SmokeBomb",
                "EjectButton",
                "Valve",
                "Box chocolates",
                "AirHorn",
                "JawBreaker",
                "JumperCable",
                "PopBottle"
            }
            local currentRoom = game.Workspace:FindFirstChild("CurrentRoom")
            if currentRoom then
                while ntfri do  -- This checks if the toggle is active
                    for _, model in ipairs(currentRoom:GetChildren()) do
                        if model:IsA("Model") then
                            local itemsFolder = model:FindFirstChild("Items")
                            if itemsFolder then
                                for _, item in ipairs(itemsFolder:GetChildren()) do
                                    if table.find(targetItems, item.Name) and not notifiedItem[item.Name] then
                                        sendNotificationItem(item.Name)
                                        notifiedItem[item.Name] = true  -- Mark item as notified
                                    end
                                end
                            end
                        end
                    end
                    task.wait(5)  -- Wait before checking again
                end
            else
                warn("There is no current room in workspace!")
            end
        else
            sound(17208361335)  -- Sound when deactivated
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Notify Rare Items has been deactivated"
            })
        end
    end
})

  local function GettingChased()
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




local GTGToggle = GeneralTab:CreateButton({
    Name = "⚙️ Go to a generator",
    Description = "Makes you go to the nearest incomplete generator. (You can use noclip for better results. Don't use that on the elevator.)",
    Callback = function()
        if TTG then
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "You are already going to a generator!"
            })
            return
        end
        TTG = true
        local Players = game:GetService("Players")
        local TweenService = game:GetService("TweenService")
        local VirtualInputManager = game:GetService("VirtualInputManager")
        local player = Players.LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()
        local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
        local cr = workspace:FindFirstChild("CurrentRoom")
        if cr then
            local room = cr:FindFirstChildWhichIsA("Model")
            if room then
                local GeneratorsFolder = room:FindFirstChild("Generators")
                local Generators = GeneratorsFolder and GeneratorsFolder:GetChildren() or {}
                -- Filtra apenas os geradores incompletos
                local incompleteGenerators = {}
                for _, generator in ipairs(Generators) do
                    local stats = generator:FindFirstChild("Stats")
                    if stats then
                        local completed = stats:FindFirstChild("Completed")
                        if completed and not completed.Value then
                            table.insert(incompleteGenerators, generator)
                        end
                    end
                end
                local closestGenerator, shortestDistance
                -- Encontra o gerador mais próximo
                for _, generator in ipairs(incompleteGenerators) do
                    local prompt = generator:FindFirstChild("Prompt")
                    if prompt then
                        local distance = (humanoidRootPart.Position - prompt.Position).Magnitude
                        if not shortestDistance or distance < shortestDistance then
                            shortestDistance = distance
                            closestGenerator = generator
                        end
                    end
                end
                if closestGenerator then
                    local promptPart = closestGenerator:FindFirstChild("Prompt")
                    if promptPart then
                        local distance = (humanoidRootPart.Position - promptPart.Position).Magnitude
                        local duration = distance * 0.05 -- Quanto maior a distância, mais devagar
                        print("Nearest generator distance:", math.floor(distance), " | Duration:", string.format("%.2f", duration))
                        local tweenInfo = TweenInfo.new(
                            duration,
                            Enum.EasingStyle.Linear
                        )
                        local goal = {
                            CFrame = promptPart.CFrame + Vector3.new(0, 0, -2)
                        }
                        local tween = TweenService:Create(humanoidRootPart, tweenInfo, goal)
                        tween:Play()
                        tween.Completed:Connect(function()
			if not GettingChased() then
                            task.wait(0.3)
                            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
                            task.wait(0.1)
                            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
			    end
       TTG = false
                        end)
                    end
                else
                    GTG = false
                    Poltergeist:Notification({
                        Title = "Notification",
                        Icon = "info",
                        ImageSource = "Material",
                        Content = "No incomplete generators found!"
                    })
                end
            end
        end
        sound(8486683243)
        Poltergeist:Notification({ 
            Title = "Notification",
            Icon = "notifications_active",
            ImageSource = "Material",
            Content = "Going to the generator..."
        })
    end
})



-- Player Tab Features
local tpWalking = false
local localPlayer = game:GetService("Players").LocalPlayer
local RunService = game:GetService("RunService")

local function StartTeleportWalk()
    tpWalking = true

    local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid")

    -- Desconecta anterior se houver
    if tpWalkingConnection then
        tpWalkingConnection:Disconnect()
    end

    tpWalkingConnection = RunService.Heartbeat:Connect(function(deltaTime)
        if not tpWalking or not character or not humanoid or not humanoid.Parent then
            tpWalkingConnection:Disconnect()
            return
        end

        if humanoid.MoveDirection.Magnitude > 0 then
            character:TranslateBy(humanoid.MoveDirection * TpWalkSpeed * deltaTime * 10)
        end
    end)
end


local function StopTeleportWalk()
    tpWalking = false
    if tpWalkingConnection then
        tpWalkingConnection:Disconnect()
        tpWalkingConnection = nil
    end
end

-- Slider para velocidade
local TpWalkSpeedSlider = PlayerTab:CreateSlider({
    Name = "Teleport Walk Speed",
    Description = "Adjust movement speed multiplier",
    Range = {0.1, 5},
    Increment = 0.1,
    CurrentValue = TpWalkSpeed,
    Callback = function(Value)
        TpWalkSpeed = Value
        if TpWalkToggle and TpWalkToggle.CurrentValue then
            StopTeleportWalk()
            StartTeleportWalk()
        end
    end
})

-- Toggle para ativar/desativar o modo
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
            StartTeleportWalk()
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

-- Reativar ao morrer/respawnar
localPlayer.CharacterAdded:Connect(function()
    if TpWalkToggle and TpWalkToggle.CurrentValue then
        StopTeleportWalk()
        task.wait(0.5)
        StartTeleportWalk()
    end
end)


local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")

local Noclip = false
local noclipConnection

-- Função para validar se uma part pode ser considerada "chão"
local function isValidGround(part, result, originY)
	if not part then return false end

	local normal = result.Normal
	if math.abs(normal.Y) < 0.9 then
		return false -- inclinado demais
	end

	local hitY = result.Position.Y
	if (originY - hitY) > 12 then
		return false -- muito distante abaixo do jogador
	end

	return true
end

local NoclipToggle = PlayerTab:CreateToggle({
	Name = "👻 Noclip",
	Description = "turns you into a POLTERGEIST bOooOooOoo",
	CurrentValue = false,
	Callback = function(v)
		Noclip = v

		local player = Players.LocalPlayer
		local character = player.Character or player.CharacterAdded:Wait()
		local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

		if Noclip then
			sound(8486683243)
			Poltergeist:Notification({ 
				Title = "Notification",
				Icon = "notifications_active",
				ImageSource = "Material",
				Content = "Noclip has been activated"
			})

			noclipConnection = RunService.Stepped:Connect(function()
				if not character or not character.Parent then return end
				local origin = humanoidRootPart.Position
				local direction = Vector3.new(0, -12, 0)

				local params = RaycastParams.new()
				params.FilterDescendantsInstances = {character}
				params.FilterType = Enum.RaycastFilterType.Exclude

				local result = Workspace:Raycast(origin, direction, params)
				local originY = origin.Y

				local groundedPart = (result and isValidGround(result.Instance, result, originY)) and result.Instance or nil

				for _, part in ipairs(Workspace:GetPartBoundsInBox(humanoidRootPart.CFrame, Vector3.new(20, 20, 20))) do
					if part:IsA("BasePart") and not part:IsDescendantOf(character) then
						if groundedPart and part == groundedPart then
							part.CanCollide = true
						else
							part.CanCollide = false
						end
					end
				end
			end)

		else
			sound(17208361335)
			Poltergeist:Notification({ 
				Title = "Notification",
				Icon = "notifications_active",
				ImageSource = "Material",
				Content = "Noclip has been deactivated"
			})

			for _, part in ipairs(Workspace:GetDescendants()) do
				if part:IsA("BasePart") then
					part.CanCollide = true
				end
			end

			if noclipConnection then
				noclipConnection:Disconnect()
				noclipConnection = nil
			end
		end
	end
})


-- God Mode
local GodMode = false
local GodModeToggle = PlayerTab:CreateToggle({
    Name = "🌌 God Mode",
    Description = "Makes you immune to twisteds (not all the twisteds)",
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

local FOV = PlayerTab:CreateSlider({
    Name = "Field of View",
    Description = "Adjust your fov",
    Range = {10, 120},
    Increment = 1,
    CurrentValue = 70,
    Callback = function(Value)
       workspace.Camera.FieldOfView = Value
    end
})

local MZD = PlayerTab:CreateSlider({
    Name = "Max zoom distance",
    Description = "Adjust your max zoom distance",
    Range = {3, 100},
    Increment = 1,
    CurrentValue = 50,
    Callback = function(Value)
     player.CameraMaxZoomDistance = Value
    end
})
OthersTab:CreateSection("Config")

OthersTab:CreateSection("Generator ESP")
local GeneratorEspColor = OthersTab:CreateColorPicker({
    Name = "🖋 Generator ESP color",
    Color = Color3.fromRGB(0, 255, 0),
    Flag = "generatorespcolor",
    Callback = function(Value)
    GeneratorEspColor = Value
    end
}, "generatorespcolor")

local ShowDistance = false
local ShowDistanceToggle = OthersTab:CreateToggle({
    Name = "📍 Show Distance",
    Description = "Shows the distance of the generator",
    CurrentValue = false,
    Callback = function(v)
        ShowDistance = v
        if ShowDistance then
            sound(8486683243)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Now the ESP shows the distance."
            })
          ShowGeneratorDistance = true
	   if generatorCache[obj] then
                        updateDistanceESP(obj, "Generator")
                    end
        else
           
            sound(17208361335)
            Poltergeist:Notification({ 
                Title = "Notification",
                Icon = "notifications_active",
                ImageSource = "Material",
                Content = "Now the ESP hide the distance."
		
            })
	    ShowGeneratorDistance = false
      if generatorCache[obj] then
                        updateDistanceESP(obj, "Generator")
                    end
        end
    end
})

OthersTab:CreateSection("Other Hubs")

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

local IYscript = OthersTab:CreateButton({
    Name = "Infinite Yield",
    Description = "Loads Infinite Yield (great universal script)",
    Callback = function()
        loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source'))()
        sound(8486683243)
        Poltergeist:Notification({ 
            Title = "Notification",
            Icon = "notifications_active",
            ImageSource = "Material",
            Content = "Script loaded!"
        })
    end
})

local ikHub = OthersTab:CreateButton({
    Name = "Iriska Hub",
    Description = "Dandy's World script made by hex233222 (🐱🐱🐱🐱🐱🐱)",
    Callback = function()
        loadstring(game:HttpGet("https://rawscripts.net/raw/Dandy's-World-ALPHA-Zyra-Hub-24675"))()
        sound(8486683243)
        Poltergeist:Notification({ 
            Title = "Notification",
            Icon = "notifications_active",
            ImageSource = "Material",
            Content = "Script loaded!"
        })
    end
})
