-- BOMBE :: MM2 v19.0 (GUN FIXED + FLOATING TITLE) :: DELTA
-- Auto Grab Gun полностью переделан | Плавающая надпись BOMBE

local Player = game:GetService("Players").LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local VIM = game:GetService("VirtualInputManager")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local Workspace = workspace
local TweenService = game:GetService("TweenService")

-- ===== КОНФИГ =====
local Config = {
    EspEnabled = false,
    AutoAim = false,
    AutoCollect = false,
    AntiAFK = false,
    AutoGrabGun = false,
    KillAura = false,
    KillAuraRange = 15,
    FOV = 70,
    FOVEnabled = false,
}

-- ===== ЦВЕТА РОЛЕЙ =====
local RoleColors = {
    Murderer = Color3.fromRGB(255, 0, 0),
    Sheriff = Color3.fromRGB(0, 100, 255),
    Innocent = Color3.fromRGB(0, 255, 100),
}

-- ===== ОПРЕДЕЛЕНИЕ РОЛЕЙ =====
local function getPlayerRole(player)
    if not player or not player.Name then return "Innocent" end
    local name = player.Name:lower()
    if name:find("sheriff") or name:find("шериф") then return "Sheriff" end
    if name:find("murder") or name:find("убийц") then return "Murderer" end
    
    if player.Character then
        for _, tool in pairs(player.Character:GetChildren()) do
            if tool:IsA("Tool") then
                local tn = tool.Name:lower()
                if tn:find("knife") or tn:find("ноже") then return "Murderer" end
                if tn:find("gun") or tn:find("pistol") or tn:find("револьвер") then return "Sheriff" end
            end
        end
        local bp = player:FindFirstChild("Backpack")
        if bp then
            for _, tool in pairs(bp:GetChildren()) do
                if tool:IsA("Tool") then
                    local tn = tool.Name:lower()
                    if tn:find("knife") or tn:find("ноже") then return "Murderer" end
                    if tn:find("gun") or tn:find("pistol") or tn:find("револьвер") then return "Sheriff" end
                end
            end
        end
    end
    return "Innocent"
end

-- ===== GUI =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BOMBE_GUI"
screenGui.Parent = Player.PlayerGui
screenGui.IgnoreGuiInset = true

-- ===== ПЛАВАЮЩАЯ НАДПИСЬ "BOMBE" СВЕРХУ =====
local floatingTitle = Instance.new("TextLabel")
floatingTitle.Parent = screenGui
floatingTitle.BackgroundTransparency = 1
floatingTitle.Size = UDim2.new(0, 200, 0, 50)
floatingTitle.Position = UDim2.new(0.5, -100, 0, 20)
floatingTitle.Text = "✦ BOMBE ✦"
floatingTitle.TextColor3 = Color3.fromRGB(200, 100, 255)
floatingTitle.TextScaled = true
floatingTitle.Font = Enum.Font.GothamBold
floatingTitle.TextStrokeTransparency = 0.3
floatingTitle.TextStrokeColor3 = Color3.fromRGB(80, 0, 120)
floatingTitle.ZIndex = 10

spawn(function()
    local offset = 0
    while wait(0.05) do
        offset = offset + 0.1
        local y = 20 + math.sin(offset) * 10
        floatingTitle.Position = UDim2.new(0.5, -100, 0, y)
    end
end)

-- Главное окно
local mainFrame = Instance.new("Frame")
mainFrame.Parent = screenGui
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 0, 30)
mainFrame.BackgroundTransparency = 0.1
mainFrame.BorderColor3 = Color3.fromRGB(180, 50, 255)
mainFrame.BorderSizePixel = 2
mainFrame.Position = UDim2.new(0.5, -160, 0.5, -125)
mainFrame.Size = UDim2.new(0, 320, 0, 250)
mainFrame.Visible = true
mainFrame.Active = true
mainFrame.Draggable = false
mainFrame.ClipsDescendants = true

local mainCorner = Instance.new("UICorner")
mainCorner.Parent = mainFrame
mainCorner.CornerRadius = UDim.new(0, 12)

-- ===== СНЕЖИНКИ =====
local snowContainer = Instance.new("Frame")
snowContainer.Parent = mainFrame
snowContainer.BackgroundTransparency = 1
snowContainer.Size = UDim2.new(1, 0, 1, 0)
snowContainer.ZIndex = 0
snowContainer.ClipsDescendants = true

local snowParticles = {}
for i = 1, 25 do
    local particle = Instance.new("Frame")
    particle.Parent = snowContainer
    particle.BackgroundColor3 = Color3.fromRGB(200, 150, 255)
    particle.BackgroundTransparency = 0.3 + math.random() * 0.4
    particle.BorderSizePixel = 0
    particle.Size = UDim2.new(0, math.random(3, 7), 0, math.random(3, 7))
    particle.Position = UDim2.new(math.random() * 0.95, 0, math.random() * 0.95, 0)
    particle.ZIndex = 0
    local corner = Instance.new("UICorner")
    corner.Parent = particle
    corner.CornerRadius = UDim.new(1, 0)
    table.insert(snowParticles, {object=particle, speed=0.3+math.random()*0.6, drift=(math.random()-0.5)*0.4, rot=(math.random()-0.5)*80})
end

RunService.RenderStepped:Connect(function()
    if not mainFrame.Visible then return end
    for _, data in pairs(snowParticles) do
        local pos = data.object.Position
        local newY = pos.Y.Scale + 0.004 * data.speed
        local newX = pos.X.Scale + 0.002 * data.drift
        if newY > 1 then newY = -0.05; newX = math.random() * 0.95 end
        if newX > 1 or newX < 0 then newX = math.random() * 0.95 end
        data.object.Position = UDim2.new(newX, 0, newY, 0)
        data.object.Rotation = data.object.Rotation + data.rot * 0.02
    end
end)

-- Заголовок в окне
local title = Instance.new("TextLabel")
title.Parent = mainFrame
title.BackgroundTransparency = 1
title.Size = UDim2.new(1, 0, 0, 28)
title.Position = UDim2.new(0, 0, 0, 2)
title.Text = "✦ BOMBE ✦"
title.TextColor3 = Color3.fromRGB(200, 100, 255)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.ZIndex = 2

-- Кнопка закрытия
local closeBtn = Instance.new("TextButton")
closeBtn.Parent = mainFrame
closeBtn.BackgroundColor3 = Color3.fromRGB(80, 0, 80)
closeBtn.BorderColor3 = Color3.fromRGB(200, 50, 255)
closeBtn.BorderSizePixel = 1
closeBtn.Position = UDim2.new(1, -30, 0, 2)
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(200, 100, 255)
closeBtn.TextScaled = true
closeBtn.Font = Enum.Font.GothamBold
closeBtn.ZIndex = 2
local closeCorner = Instance.new("UICorner")
closeCorner.Parent = closeBtn
closeCorner.CornerRadius = UDim.new(0, 6)
closeBtn.MouseButton1Click:Connect(function() mainFrame.Visible = false end)

-- ===== КНОПКИ ВКЛАДОК =====
local tabButtons = Instance.new("Frame")
tabButtons.Parent = mainFrame
tabButtons.BackgroundTransparency = 1
tabButtons.Position = UDim2.new(0, 5, 0, 35)
tabButtons.Size = UDim2.new(0, 65, 0, 200)
tabButtons.ZIndex = 2

local function createTabButton(text, yPos)
    local btn = Instance.new("TextButton")
    btn.Parent = tabButtons
    btn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
    btn.BorderColor3 = Color3.fromRGB(180, 50, 255)
    btn.BorderSizePixel = 1
    btn.Position = UDim2.new(0, 0, 0, yPos)
    btn.Size = UDim2.new(1, 0, 0, 35)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(200, 150, 255)
    btn.TextScaled = true
    btn.Font = Enum.Font.GothamBold
    btn.ZIndex = 2
    local c = Instance.new("UICorner")
    c.Parent = btn
    c.CornerRadius = UDim.new(0, 6)
    return btn
end

local espTabBtn = createTabButton("ESP", 0)
local gunTabBtn = createTabButton("GUN", 38)
local knifeTabBtn = createTabButton("KILL", 76)
local coinsTabBtn = createTabButton("COINS", 114)
local fovTabBtn = createTabButton("FOV", 152)

-- ===== ПАНЕЛИ =====
local function createPanel()
    local p = Instance.new("Frame")
    p.Parent = mainFrame
    p.BackgroundColor3 = Color3.fromRGB(25, 0, 50)
    p.BorderColor3 = Color3.fromRGB(180, 50, 255)
    p.BorderSizePixel = 1
    p.Position = UDim2.new(0, 75, 0, 35)
    p.Size = UDim2.new(1, -85, 0, 205)
    p.BackgroundTransparency = 0.2
    p.Visible = false
    p.ClipsDescendants = true
    p.ZIndex = 2
    local c = Instance.new("UICorner")
    c.Parent = p
    c.CornerRadius = UDim.new(0, 8)
    return p
end

local espPanel = createPanel()
local gunPanel = createPanel()
local knifePanel = createPanel()
local coinsPanel = createPanel()
local fovPanel = createPanel()

-- ===== ТОГГЛЫ =====
local function createToggle(parent, yPos, labelText, configKey)
    local frame = Instance.new("Frame")
    frame.Parent = parent
    frame.BackgroundColor3 = Color3.fromRGB(15, 0, 30)
    frame.BorderColor3 = Color3.fromRGB(150, 40, 220)
    frame.BorderSizePixel = 1
    frame.Position = UDim2.new(0.05, 0, 0, yPos)
    frame.Size = UDim2.new(0.9, 0, 0, 28)
    frame.ZIndex = 2
    local c = Instance.new("UICorner")
    c.Parent = frame
    c.CornerRadius = UDim.new(0, 5)

    local label = Instance.new("TextLabel")
    label.Parent = frame
    label.BackgroundTransparency = 1
    label.Position = UDim2.new(0, 5, 0, 0)
    label.Size = UDim2.new(0, 100, 1, 0)
    label.Text = labelText
    label.TextColor3 = Color3.fromRGB(200, 150, 255)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextScaled = true
    label.Font = Enum.Font.Gotham
    label.ZIndex = 2

    local btn = Instance.new("TextButton")
    btn.Parent = frame
    btn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
    btn.BorderColor3 = Color3.fromRGB(180, 50, 255)
    btn.BorderSizePixel = 1
    btn.Position = UDim2.new(1, -45, 0, 2)
    btn.Size = UDim2.new(0, 40, 0, 23)
    btn.Text = "OFF"
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextScaled = true
    btn.Font = Enum.Font.GothamBold
    btn.ZIndex = 2
    local bc = Instance.new("UICorner")
    bc.Parent = btn
    bc.CornerRadius = UDim.new(0, 5)

    btn.MouseButton1Click:Connect(function()
        Config[configKey] = not Config[configKey]
        btn.BackgroundColor3 = Config[configKey] and Color3.fromRGB(100, 0, 200) or Color3.fromRGB(60, 0, 80)
        btn.Text = Config[configKey] and "ON" or "OFF"
    end)
    return frame
end

-- ===== ВКЛАДКА ESP =====
local espTitle = Instance.new("TextLabel")
espTitle.Parent = espPanel
espTitle.BackgroundTransparency = 1
espTitle.Size = UDim2.new(1, 0, 0, 20)
espTitle.Position = UDim2.new(0, 0, 0, 2)
espTitle.Text = "👁 ESP"
espTitle.TextColor3 = Color3.fromRGB(200, 100, 255)
espTitle.TextScaled = true
espTitle.Font = Enum.Font.GothamBold
espTitle.ZIndex = 2

createToggle(espPanel, 25, "Аура", "EspEnabled")

-- ===== ВКЛАДКА GUN =====
local gunTitle = Instance.new("TextLabel")
gunTitle.Parent = gunPanel
gunTitle.BackgroundTransparency = 1
gunTitle.Size = UDim2.new(1, 0, 0, 20)
gunTitle.Position = UDim2.new(0, 0, 0, 2)
gunTitle.Text = "🔫 ОРУЖИЕ"
gunTitle.TextColor3 = Color3.fromRGB(200, 100, 255)
gunTitle.TextScaled = true
gunTitle.Font = Enum.Font.GothamBold
gunTitle.ZIndex = 2

createToggle(gunPanel, 25, "Aimbot", "AutoAim")
createToggle(gunPanel, 55, "Auto Grab", "AutoGrabGun")

local gunStatus = Instance.new("TextLabel")
gunStatus.Parent = gunPanel
gunStatus.BackgroundTransparency = 1
gunStatus.Position = UDim2.new(0.05, 0, 0, 90)
gunStatus.Size = UDim2.new(0.9, 0, 0, 20)
gunStatus.Text = "⚪ OFF"
gunStatus.TextColor3 = Color3.fromRGB(200, 150, 255)
gunStatus.TextScaled = true
gunStatus.Font = Enum.Font.Gotham
gunStatus.ZIndex = 2

-- ===== ВКЛАДКА KILL AURA =====
local knifeTitle = Instance.new("TextLabel")
knifeTitle.Parent = knifePanel
knifeTitle.BackgroundTransparency = 1
knifeTitle.Size = UDim2.new(1, 0, 0, 20)
knifeTitle.Position = UDim2.new(0, 0, 0, 2)
knifeTitle.Text = "🔪 KILL AURA"
knifeTitle.TextColor3 = Color3.fromRGB(200, 100, 255)
knifeTitle.TextScaled = true
knifeTitle.Font = Enum.Font.GothamBold
knifeTitle.ZIndex = 2

createToggle(knifePanel, 25, "Kill Aura", "KillAura")

-- Дистанция
local rangeFrame = Instance.new("Frame")
rangeFrame.Parent = knifePanel
rangeFrame.BackgroundColor3 = Color3.fromRGB(15, 0, 30)
rangeFrame.BorderColor3 = Color3.fromRGB(150, 40, 220)
rangeFrame.BorderSizePixel = 1
rangeFrame.Position = UDim2.new(0.05, 0, 0, 58)
rangeFrame.Size = UDim2.new(0.9, 0, 0, 35)
rangeFrame.ZIndex = 2
local rangeCorner = Instance.new("UICorner")
rangeCorner.Parent = rangeFrame
rangeCorner.CornerRadius = UDim.new(0, 5)

local rangeLabel = Instance.new("TextLabel")
rangeLabel.Parent = rangeFrame
rangeLabel.BackgroundTransparency = 1
rangeLabel.Position = UDim2.new(0, 5, 0, 0)
rangeLabel.Size = UDim2.new(0, 60, 1, 0)
rangeLabel.Text = "Дист.:"
rangeLabel.TextColor3 = Color3.fromRGB(200, 150, 255)
rangeLabel.TextXAlignment = Enum.TextXAlignment.Left
rangeLabel.TextScaled = true
rangeLabel.Font = Enum.Font.Gotham
rangeLabel.ZIndex = 2

local rangeValue = Instance.new("TextLabel")
rangeValue.Parent = rangeFrame
rangeValue.BackgroundTransparency = 1
rangeValue.Position = UDim2.new(0.35, 0, 0, 0)
rangeValue.Size = UDim2.new(0, 40, 1, 0)
rangeValue.Text = "15"
rangeValue.TextColor3 = Color3.fromRGB(255, 200, 100)
rangeValue.TextScaled = true
rangeValue.Font = Enum.Font.GothamBold
rangeValue.ZIndex = 2

local minusBtn = Instance.new("TextButton")
minusBtn.Parent = rangeFrame
minusBtn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
minusBtn.BorderColor3 = Color3.fromRGB(180, 50, 255)
minusBtn.BorderSizePixel = 1
minusBtn.Position = UDim2.new(0.65, 0, 0, 3)
minusBtn.Size = UDim2.new(0, 28, 0, 28)
minusBtn.Text = "−"
minusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minusBtn.TextScaled = true
minusBtn.Font = Enum.Font.GothamBold
minusBtn.ZIndex = 2
local minusCorner = Instance.new("UICorner")
minusCorner.Parent = minusBtn
minusCorner.CornerRadius = UDim.new(0, 5)

local plusBtn = Instance.new("TextButton")
plusBtn.Parent = rangeFrame
plusBtn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
plusBtn.BorderColor3 = Color3.fromRGB(180, 50, 255)
plusBtn.BorderSizePixel = 1
plusBtn.Position = UDim2.new(0.8, 0, 0, 3)
plusBtn.Size = UDim2.new(0, 28, 0, 28)
plusBtn.Text = "+"
plusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
plusBtn.TextScaled = true
plusBtn.Font = Enum.Font.GothamBold
plusBtn.ZIndex = 2
local plusCorner = Instance.new("UICorner")
plusCorner.Parent = plusBtn
plusCorner.CornerRadius = UDim.new(0, 5)

minusBtn.MouseButton1Click:Connect(function()
    Config.KillAuraRange = math.max(5, Config.KillAuraRange - 1)
    rangeValue.Text = tostring(Config.KillAuraRange)
end)
plusBtn.MouseButton1Click:Connect(function()
    Config.KillAuraRange = math.min(30, Config.KillAuraRange + 1)
    rangeValue.Text = tostring(Config.KillAuraRange)
end)

-- ===== ВКЛАДКА COINS =====
local coinsTitle = Instance.new("TextLabel")
coinsTitle.Parent = coinsPanel
coinsTitle.BackgroundTransparency = 1
coinsTitle.Size = UDim2.new(1, 0, 0, 20)
coinsTitle.Position = UDim2.new(0, 0, 0, 2)
coinsTitle.Text = "🪙 МОНЕТЫ"
coinsTitle.TextColor3 = Color3.fromRGB(200, 100, 255)
coinsTitle.TextScaled = true
coinsTitle.Font = Enum.Font.GothamBold
coinsTitle.ZIndex = 2

createToggle(coinsPanel, 25, "Auto Collect", "AutoCollect")

local coinsStatus = Instance.new("TextLabel")
coinsStatus.Parent = coinsPanel
coinsStatus.BackgroundTransparency = 1
coinsStatus.Position = UDim2.new(0.05, 0, 0, 60)
coinsStatus.Size = UDim2.new(0.9, 0, 0, 20)
coinsStatus.Text = "⚪ OFF"
coinsStatus.TextColor3 = Color3.fromRGB(200, 150, 255)
coinsStatus.TextScaled = true
coinsStatus.Font = Enum.Font.Gotham
coinsStatus.ZIndex = 2

-- ===== ВКЛАДКА FOV =====
local fovTitle = Instance.new("TextLabel")
fovTitle.Parent = fovPanel
fovTitle.BackgroundTransparency = 1
fovTitle.Size = UDim2.new(1, 0, 0, 20)
fovTitle.Position = UDim2.new(0, 0, 0, 2)
fovTitle.Text = "🎥 FOV"
fovTitle.TextColor3 = Color3.fromRGB(200, 100, 255)
fovTitle.TextScaled = true
fovTitle.Font = Enum.Font.GothamBold
fovTitle.ZIndex = 2

createToggle(fovPanel, 25, "FOV Enabled", "FOVEnabled")

local fovFrame = Instance.new("Frame")
fovFrame.Parent = fovPanel
fovFrame.BackgroundColor3 = Color3.fromRGB(15, 0, 30)
fovFrame.BorderColor3 = Color3.fromRGB(150, 40, 220)
fovFrame.BorderSizePixel = 1
fovFrame.Position = UDim2.new(0.05, 0, 0, 60)
fovFrame.Size = UDim2.new(0.9, 0, 0, 45)
fovFrame.ZIndex = 2
local fovCorner = Instance.new("UICorner")
fovCorner.Parent = fovFrame
fovCorner.CornerRadius = UDim.new(0, 5)

local fovLabel = Instance.new("TextLabel")
fovLabel.Parent = fovFrame
fovLabel.BackgroundTransparency = 1
fovLabel.Position = UDim2.new(0, 5, 0, 0)
fovLabel.Size = UDim2.new(0, 80, 0, 20)
fovLabel.Text = "FOV:"
fovLabel.TextColor3 = Color3.fromRGB(200, 150, 255)
fovLabel.TextXAlignment = Enum.TextXAlignment.Left
fovLabel.TextScaled = true
fovLabel.Font = Enum.Font.Gotham
fovLabel.ZIndex = 2

local fovValue = Instance.new("TextLabel")
fovValue.Parent = fovFrame
fovValue.BackgroundTransparency = 1
fovValue.Position = UDim2.new(0.3, 0, 0, 0)
fovValue.Size = UDim2.new(0, 50, 0, 20)
fovValue.Text = "70"
fovValue.TextColor3 = Color3.fromRGB(255, 200, 100)
fovValue.TextScaled = true
fovValue.Font = Enum.Font.GothamBold
fovValue.ZIndex = 2

local fovBar = Instance.new("Frame")
fovBar.Parent = fovFrame
fovBar.BackgroundColor3 = Color3.fromRGB(40, 0, 60)
fovBar.BorderColor3 = Color3.fromRGB(180, 50, 255)
fovBar.BorderSizePixel = 1
fovBar.Position = UDim2.new(0.05, 0, 0, 25)
fovBar.Size = UDim2.new(0.9, 0, 0, 12)
fovBar.ZIndex = 2
local fovBarCorner = Instance.new("UICorner")
fovBarCorner.Parent = fovBar
fovBarCorner.CornerRadius = UDim.new(0, 6)

local fovFill = Instance.new("Frame")
fovFill.Parent = fovBar
fovFill.BackgroundColor3 = Color3.fromRGB(150, 50, 255)
fovFill.BorderSizePixel = 0
fovFill.Position = UDim2.new(0, 0, 0, 0)
fovFill.Size = UDim2.new(0.33, 0, 1, 0)
fovFill.ZIndex = 2
local fovFillCorner = Instance.new("UICorner")
fovFillCorner.Parent = fovFill
fovFillCorner.CornerRadius = UDim.new(0, 6)

local fovMinusBtn = Instance.new("TextButton")
fovMinusBtn.Parent = fovFrame
fovMinusBtn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
fovMinusBtn.BorderColor3 = Color3.fromRGB(180, 50, 255)
fovMinusBtn.BorderSizePixel = 1
fovMinusBtn.Position = UDim2.new(0.7, 0, 0, 3)
fovMinusBtn.Size = UDim2.new(0, 25, 0, 25)
fovMinusBtn.Text = "−"
fovMinusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
fovMinusBtn.TextScaled = true
fovMinusBtn.Font = Enum.Font.GothamBold
fovMinusBtn.ZIndex = 2
local fovMinusCorner = Instance.new("UICorner")
fovMinusCorner.Parent = fovMinusBtn
fovMinusCorner.CornerRadius = UDim.new(0, 5)

local fovPlusBtn = Instance.new("TextButton")
fovPlusBtn.Parent = fovFrame
fovPlusBtn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
fovPlusBtn.BorderColor3 = Color3.fromRGB(180, 50, 255)
fovPlusBtn.BorderSizePixel = 1
fovPlusBtn.Position = UDim2.new(0.85, 0, 0, 3)
fovPlusBtn.Size = UDim2.new(0, 25, 0, 25)
fovPlusBtn.Text = "+"
fovPlusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
fovPlusBtn.TextScaled = true
fovPlusBtn.Font = Enum.Font.GothamBold
fovPlusBtn.ZIndex = 2
local fovPlusCorner = Instance.new("UICorner")
fovPlusCorner.Parent = fovPlusBtn
fovPlusCorner.CornerRadius = UDim.new(0, 5)

fovMinusBtn.MouseButton1Click:Connect(function()
    Config.FOV = math.max(40, Config.FOV - 5)
    fovValue.Text = tostring(Config.FOV)
    fovFill.Size = UDim2.new((Config.FOV - 40) / 80, 0, 1, 0)
end)
fovPlusBtn.MouseButton1Click:Connect(function()
    Config.FOV = math.min(120, Config.FOV + 5)
    fovValue.Text = tostring(Config.FOV)
    fovFill.Size = UDim2.new((Config.FOV - 40) / 80, 0, 1, 0)
end)

-- ===== ПЕРЕКЛЮЧЕНИЕ ВКЛАДОК =====
local function switchTab(activePanel, activeBtn)
    espPanel.Visible = false
    gunPanel.Visible = false
    knifePanel.Visible = false
    coinsPanel.Visible = false
    fovPanel.Visible = false
    
    espTabBtn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
    gunTabBtn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
    knifeTabBtn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
    coinsTabBtn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
    fovTabBtn.BackgroundColor3 = Color3.fromRGB(60, 0, 80)
    
    if activePanel then activePanel.Visible = true end
    if activeBtn then activeBtn.BackgroundColor3 = Color3.fromRGB(80, 0, 120) end
end

espTabBtn.MouseButton1Click:Connect(function() switchTab(espPanel, espTabBtn) end)
gunTabBtn.MouseButton1Click:Connect(function() switchTab(gunPanel, gunTabBtn) end)
knifeTabBtn.MouseButton1Click:Connect(function() switchTab(knifePanel, knifeTabBtn) end)
coinsTabBtn.MouseButton1Click:Connect(function() switchTab(coinsPanel, coinsTabBtn) end)
fovTabBtn.MouseButton1Click:Connect(function() switchTab(fovPanel, fovTabBtn) end)

switchTab(espPanel, espTabBtn)

-- ===== КНОПКА ОТКРЫТИЯ =====
local openBtn = Instance.new("TextButton")
openBtn.Parent = screenGui
openBtn.BackgroundColor3 = Color3.fromRGB(15, 0, 30)
openBtn.BorderColor3 = Color3.fromRGB(180, 50, 255)
openBtn.BorderSizePixel = 2
openBtn.Position = UDim2.new(0, 5, 1, -45)
openBtn.Size = UDim2.new(0, 70, 0, 40)
openBtn.Text = "✦ B"
openBtn.TextColor3 = Color3.fromRGB(200, 100, 255)
openBtn.TextScaled = true
openBtn.Font = Enum.Font.GothamBold
openBtn.ZIndex = 2
local openCorner = Instance.new("UICorner")
openCorner.Parent = openBtn
openCorner.CornerRadius = UDim.new(0, 10)

openBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = not mainFrame.Visible
end)

-- ===== ESP =====
local espObjects = {}

local function updateESP()
    if not Config.EspEnabled then
        for _, obj in pairs(espObjects) do pcall(function() obj:Remove() end) end
        espObjects = {}
        return
    end

    for _, obj in pairs(espObjects) do pcall(function() obj:Remove() end) end
    espObjects = {}

    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= Player and plr.Character then
            local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    local role = getPlayerRole(plr)
                    local color = RoleColors[role] or Color3.fromRGB(200, 200, 200)
                    
                    local parts = {}
                    for _, part in pairs(plr.Character:GetChildren()) do
                        if part:IsA("BasePart") then table.insert(parts, part) end
                    end
                    
                    if #parts > 0 then
                        local minX, maxX = math.huge, -math.huge
                        local minY, maxY = math.huge, -math.huge
                        for _, part in pairs(parts) do
                            local partPos, partOn = Camera:WorldToViewportPoint(part.Position)
                            if partOn then
                                if partPos.X < minX then minX = partPos.X end
                                if partPos.X > maxX then maxX = partPos.X end
                                if partPos.Y < minY then minY = partPos.Y end
                                if partPos.Y > maxY then maxY = partPos.Y end
                            end
                        end
                        
                        if minX ~= math.huge and maxX ~= -math.huge then
                            local width = maxX - minX
                            local height = maxY - minY
                            local centerX = (minX + maxX) / 2
                            local centerY = (minY + maxY) / 2
                            
                            local auraWidth = width * 1.8
                            local auraHeight = height * 1.6
                            
                            local aura = Drawing.new("Square")
                            aura.Size = Vector2.new(auraWidth, auraHeight)
                            aura.Position = Vector2.new(centerX - auraWidth/2, centerY - auraHeight/2)
                            aura.Color = color
                            aura.Thickness = 0
                            aura.Filled = true
                            aura.Transparency = 0.35
                            aura.Visible = true
                            table.insert(espObjects, aura)
                            
                            local border = Drawing.new("Square")
                            border.Size = Vector2.new(auraWidth, auraHeight)
                            border.Position = Vector2.new(centerX - auraWidth/2, centerY - auraHeight/2)
                            border.Color = color
                            border.Thickness = 3
                            border.Filled = false
                            border.Transparency = 0.1
                            border.Visible = true
                            table.insert(espObjects, border)
                            
                            if role == "Murderer" then
                                local c1 = Drawing.new("Line")
                                c1.From = Vector2.new(centerX - 20, centerY - height * 0.5)
                                c1.To = Vector2.new(centerX + 20, centerY + height * 0.5)
                                c1.Color = Color3.fromRGB(255, 255, 255)
                                c1.Thickness = 3
                                c1.Visible = true
                                table.insert(espObjects, c1)
                                
                                local c2 = Drawing.new("Line")
                                c2.From = Vector2.new(centerX + 20, centerY - height * 0.5)
                                c2.To = Vector2.new(centerX - 20, centerY + height * 0.5)
                                c2.Color = Color3.fromRGB(255, 255, 255)
                                c2.Thickness = 3
                                c2.Visible = true
                                table.insert(espObjects, c2)
                            elseif role == "Sheriff" then
                                for j = 1, 5 do
                                    local a1 = (j / 5) * math.pi * 2
                                    local a2 = ((j + 1) / 5) * math.pi * 2
                                    local sl = Drawing.new("Line")
                                    sl.From = Vector2.new(centerX + math.cos(a1) * 25, centerY + math.sin(a1) * 25)
                                    sl.To = Vector2.new(centerX + math.cos(a2) * 25, centerY + math.sin(a2) * 25)
                                    sl.Color = Color3.fromRGB(255, 255, 255)
                                    sl.Thickness = 3
                                    sl.Visible = true
                                    table.insert(espObjects, sl)
                                end
                            else
                                local head = plr.Character:FindFirstChild("Head")
                                if head then
                                    local headPos, headOn = Camera:WorldToViewportPoint(head.Position)
                                    if headOn then
                                        local dot = Drawing.new("Circle")
                                        dot.Radius = 14
                                        dot.Position = Vector2.new(headPos.X, headPos.Y - 25)
                                        dot.Color = Color3.fromRGB(0, 255, 100)
                                        dot.Filled = true
                                        dot.Thickness = 0
                                        dot.Visible = true
                                        table.insert(espObjects, dot)
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end

-- ===== AIMBOT =====
local function updateAimbot()
    if not Config.AutoAim then return end
    local target = nil
    local minDist = math.huge
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= Player and plr.Character then
            local head = plr.Character:FindFirstChild("Head")
            if head and getPlayerRole(plr) == "Murderer" then
                local dist = (head.Position - Camera.CFrame.Position).Magnitude
                if dist < minDist then minDist = dist; target = plr end
            end
        end
    end
    if target and target.Character then
        local head = target.Character.Head
        if head then
            local dir = (head.Position - Camera.CFrame.Position).unit
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + dir * 10)
            gunStatus.Text = "🎯 " .. target.Name
            gunStatus.TextColor3 = Color3.fromRGB(255, 50, 50)
        end
    else
        gunStatus.Text = "⚪ NO MURDERER"
        gunStatus.TextColor3 = Color3.fromRGB(200, 200, 100)
    end
end

-- ===== KILL AURA =====
local function updateKillAura()
    if not Config.KillAura then return end
    local char = Player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= Player and plr.Character then
            local th = plr.Character:FindFirstChild("HumanoidRootPart")
            if th and (th.Position - hrp.Position).Magnitude <= Config.KillAuraRange then
                local hum = plr.Character:FindFirstChildOfClass("Humanoid")
                if hum and hum.Health > 0 then hum.Health = 0 end
            end
        end
    end
end

-- ===== AUTO GRAB GUN =====
local grabActive = false

local function forceGrabGun()
    if grabActive then return end
    grabActive = true
    
    local char = Player.Character
    if not char then grabActive = false; return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then grabActive = false; return end
    
    local gun = nil
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("Tool") then
            local n = obj.Name:lower()
            if n:find("gun") or n:find("pistol") or n:find("револьвер") then
                gun = obj
                break
            end
        end
        if obj:IsA("BasePart") then
            local n = obj.Name:lower()
            if n:find("gun") or n:find("pistol") then
                gun = obj
                break
            end
        end
    end
    
    if not gun then
        gunStatus.Text = "⚪ NO GUN"
        gunStatus.TextColor3 = Color3.fromRGB(200, 200, 100)
        grabActive = false
        return
    end
    
    gunStatus.Text = "🔫 FOUND! TELEPORTING..."
    gunStatus.TextColor3 = Color3.fromRGB(200, 100, 255)
    
    local gPos = gun.Position
    if gun:IsA("Tool") and gun:FindFirstChild("Handle") then
        gPos = gun.Handle.Position
    end
    hrp.CFrame = CFrame.new(gPos + Vector3.new(0, 2, 0))
    wait(0.1)
    
    if gun:IsA("Tool") then
        for _, tool in pairs(char:GetChildren()) do
            if tool:IsA("Tool") then
                tool.Parent = nil
            end
        end
        gun.Parent = char
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum:EquipTool(gun) end
        gunStatus.Text = "✅ GUN EQUIPPED!"
        gunStatus.TextColor3 = Color3.fromRGB(0, 255, 0)
    else
        firetouchinterest(hrp, gun, 0)
        firetouchinterest(hrp, gun, 1)
        wait(0.3)
        for _, tool in pairs(char:GetChildren()) do
            if tool:IsA("Tool") then
                local n = tool.Name:lower()
                if n:find("gun") or n:find("pistol") then
                    gunStatus.Text = "✅ GUN EQUIPPED!"
                    gunStatus.TextColor3 = Color3.fromRGB(0, 255, 0)
                    break
                end
            end
        end
    end
    
    wait(1)
    grabActive = false
end

local function isSheriffDead()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= Player and getPlayerRole(plr) == "Sheriff" then
            if plr.Character and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
                return false
            end
        end
    end
    return true
end

local function updateGrabGun()
    if not Config.AutoGrabGun then
        gunStatus.Text = "⚪ OFF"
        gunStatus.TextColor3 = Color3.fromRGB(200, 100, 100)
        return
    end
    if not isSheriffDead() then
        gunStatus.Text = "⚪ SHERIFF ALIVE"
        gunStatus.TextColor3 = Color3.fromRGB(0, 255, 100)
        return
    end
    forceGrabGun()
end

-- ===== AUTO COLLECT =====
local function updateAutoCollect()
    if not Config.AutoCollect then
        coinsStatus.Text = "⚪ OFF"
        coinsStatus.TextColor3 = Color3.fromRGB(200, 100, 100)
        return
    end
    local char = Player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local count = 0
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Name:lower():find("coin") then
            if (obj.Position - hrp.Position).Magnitude < 12 then
                firetouchinterest(hrp, obj, 0)
                firetouchinterest(hrp, obj, 1)
                count = count + 1
            end
        end
    end
    coinsStatus.Text = count > 0 and ("🪙 +" .. count) or "🪙 WAITING"
    coinsStatus.TextColor3 = count > 0 and Color3.fromRGB(255, 200, 100) or Color3.fromRGB(200, 150, 255)
end

-- ===== FOV =====
local function updateFOV()
    if Config.FOVEnabled then
        Camera.FieldOfView = Config.FOV
    end
end

-- ===== ANTI-AFK =====
spawn(function()
    while wait(45) do
        if Config.AntiAFK then
            VIM:SendKeyEvent("w", true, false)
            wait(0.1)
            VIM:SendKeyEvent("w", false, false)
        end
    end
end)

-- ===== ГЛАВНЫЙ ЦИКЛ =====
RunService.RenderStepped:Connect(function()
    updateESP()
    updateAimbot()
    updateKillAura()
    updateAutoCollect()
    updateGrabGun()
    updateFOV()
end)

print("✦ BOMBE v19.0 ЗАГРУЖЕН!")
print("✦ Auto Grab Gun переделан — телепорт + подбор")
print("✦ Плавающая надпись BOMBE сверху")
