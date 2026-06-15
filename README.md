-- ====================================================
--        MO + SH + BLO HUB - BY MOILA933
-- ====================================================

local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService   = game:GetService("RunService")
local UIS          = game:GetService("UserInputService")
local RS           = game:GetService("ReplicatedStorage")
local LocalPlayer  = Players.LocalPlayer

-- ريموت الشات (يظهر بالشات + HD Admin يقرأه)
local DataRemote = RS
    :WaitForChild("RemoteEvents")
    :WaitForChild("DataService")

-- ريموت HD Admin (تنفيذ مباشر بدون شات)
local HDRemote = RS
    :WaitForChild("HDAdminHDClient")
    :WaitForChild("Signals")
    :WaitForChild("RequestCommandModification")

-- إرسال رسالة واحدة كبيرة — تظهر بالشات + تنفّذ
local function sendBigMessage(bigMsg)
    task.spawn(function()
        -- عبر الشات (يظهر للكل)
        pcall(function() DataRemote:FireServer(bigMsg) end)
        -- عبر HD Admin مباشرة (للتأكد من التنفيذ)
        pcall(function() HDRemote:InvokeServer(bigMsg) end)
    end)
end

-- الحالة
local selectedPlayer = nil
local nameMode       = "full"
local spamActive     = {false, false, false}

-- الألوان
local GREEN     = Color3.fromRGB(80, 200, 100)
local GREEN_MID = Color3.fromRGB(30, 130, 55)
local WHITE     = Color3.fromRGB(255, 255, 255)

-- تنظيف نسخة قديمة
pcall(function()
    LocalPlayer:WaitForChild("PlayerGui"):FindFirstChild("MO_SH_BLO_Hub"):Destroy()
end)

-- ====== ScreenGui ======
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name           = "MO_SH_BLO_Hub"
ScreenGui.ResetOnSpawn   = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder   = 2147483647
ScreenGui.Parent         = LocalPlayer:WaitForChild("PlayerGui")

-- ====== دوال مساعدة ======
local function corner(p, r)
    local c = Instance.new("UICorner", p)
    c.CornerRadius = UDim.new(0, r or 10)
    return c
end

local function mkStroke(p, col, thick, trans)
    local s = Instance.new("UIStroke", p)
    s.Color           = col   or WHITE
    s.Thickness       = thick or 2
    s.Transparency    = trans or 0
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    return s
end

local function gradient(p, c1, c2, rot)
    local g = Instance.new("UIGradient", p)
    g.Color    = ColorSequence.new(c1, c2)
    g.Rotation = rot or 90
    return g
end

local function pulsedStroke(s, speed, minT, maxT, offset)
    offset = offset or 0
    task.spawn(function()
        while s and s.Parent do
            local t = tick()
            local p = (math.sin(t * (speed or 2) + offset) + 1) / 2
            s.Transparency = (minT or 0.1) + p * ((maxT or 0.8) - (minT or 0.1))
            task.wait(0.03)
        end
    end)
end

local function makeDraggable(frame, handle)
    local dragging, dragInput, dragStart, startPos
    handle = handle or frame
    handle.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or
           inp.UserInputType == Enum.UserInputType.Touch then
            dragging  = true
            dragStart = inp.Position
            startPos  = frame.Position
            inp.Changed:Connect(function()
                if inp.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    handle.InputChanged:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseMovement or
           inp.UserInputType == Enum.UserInputType.Touch then
            dragInput = inp
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if inp == dragInput and dragging then
            local d = inp.Position - dragStart
            frame.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + d.X,
                startPos.Y.Scale, startPos.Y.Offset + d.Y
            )
        end
    end)
end

-- ====================================================
--                  MAIN FRAME
-- ====================================================
local MainFrame = Instance.new("Frame")
MainFrame.Name                   = "MainFrame"
MainFrame.Size                   = UDim2.new(0, 520, 0, 560)
MainFrame.Position               = UDim2.new(0.5, -260, 0.5, -280)
MainFrame.BackgroundColor3       = Color3.fromRGB(10, 55, 20)
MainFrame.BackgroundTransparency = 0.08
MainFrame.BorderSizePixel        = 0
MainFrame.Active                 = true
MainFrame.Parent                 = ScreenGui
corner(MainFrame, 16)

local mainStroke = mkStroke(MainFrame, WHITE, 2.5, 0.2)

-- نبض أخضر + حواف بيضاء
task.spawn(function()
    while MainFrame and MainFrame.Parent do
        local t = tick()
        local p = (math.sin(t * 2) + 1) / 2
        MainFrame.BackgroundColor3 = Color3.fromRGB(
            math.floor(8  + p * 12),
            math.floor(50 + p * 45),
            math.floor(18 + p * 18)
        )
        mainStroke.Transparency = 0.05 + p * 0.75
        mainStroke.Thickness    = 2    + p * 1.5
        task.wait(0.03)
    end
end)

makeDraggable(MainFrame)

-- ====== شريط العنوان ======
local TitleBar = Instance.new("Frame")
TitleBar.Size                   = UDim2.new(1, 0, 0, 44)
TitleBar.BackgroundColor3       = Color3.fromRGB(5, 30, 12)
TitleBar.BackgroundTransparency = 0.2
TitleBar.BorderSizePixel        = 0
TitleBar.Parent                 = MainFrame
corner(TitleBar, 16)
gradient(TitleBar, Color3.fromRGB(15, 80, 35), Color3.fromRGB(5, 35, 15), 90)

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size               = UDim2.new(1, -50, 1, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text               = "🌿  MO + SH + BLO  🌿"
TitleLabel.Font               = Enum.Font.GothamBlack
TitleLabel.TextSize           = 19
TitleLabel.TextColor3         = GREEN
TitleLabel.Parent             = TitleBar

task.spawn(function()
    while TitleLabel and TitleLabel.Parent do
        local t = tick()
        local p = (math.sin(t * 3) + 1) / 2
        TitleLabel.TextColor3 = Color3.fromRGB(
            math.floor(80  + p * 30),
            math.floor(200 + p * 55),
            math.floor(90  + p * 40)
        )
        task.wait(0.03)
    end
end)

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size             = UDim2.new(0, 32, 0, 32)
CloseBtn.Position         = UDim2.new(1, -38, 0, 6)
CloseBtn.BackgroundColor3 = Color3.fromRGB(190, 40, 40)
CloseBtn.Text             = "✕"
CloseBtn.TextColor3       = WHITE
CloseBtn.TextSize         = 15
CloseBtn.Font             = Enum.Font.GothamBold
CloseBtn.AutoButtonColor  = false
CloseBtn.BorderSizePixel  = 0
CloseBtn.ZIndex           = 10
CloseBtn.Parent           = TitleBar
corner(CloseBtn, 7)
mkStroke(CloseBtn, WHITE, 1, 0.5)
CloseBtn.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)

-- ====================================================
--              SCROLL CONTENT (عمودي)
-- ====================================================
local Content = Instance.new("ScrollingFrame")
Content.Size                 = UDim2.new(1, -14, 1, -54)
Content.Position             = UDim2.new(0, 7, 0, 50)
Content.BackgroundTransparency = 1
Content.ScrollBarThickness   = 4
Content.ScrollBarImageColor3 = GREEN
Content.BorderSizePixel      = 0
Content.CanvasSize           = UDim2.new()
Content.AutomaticCanvasSize  = Enum.AutomaticSize.Y
Content.Parent               = MainFrame

local CL = Instance.new("UIListLayout")
CL.Padding             = UDim.new(0, 8)
CL.SortOrder           = Enum.SortOrder.LayoutOrder
CL.HorizontalAlignment = Enum.HorizontalAlignment.Center
CL.Parent              = Content

local CP = Instance.new("UIPadding")
CP.PaddingTop    = UDim.new(0, 6)
CP.PaddingBottom = UDim.new(0, 10)
CP.PaddingLeft   = UDim.new(0, 5)
CP.PaddingRight  = UDim.new(0, 5)
CP.Parent        = Content

-- ====================================================
--           1) أسماء اللاعبين
-- ====================================================
local pTitle = Instance.new("TextLabel")
pTitle.Size               = UDim2.new(1, 0, 0, 18)
pTitle.BackgroundTransparency = 1
pTitle.Text               = "👥  اختر اللاعب"
pTitle.TextColor3         = Color3.fromRGB(160, 255, 180)
pTitle.Font               = Enum.Font.GothamBold
pTitle.TextSize           = 12
pTitle.TextXAlignment     = Enum.TextXAlignment.Right
pTitle.LayoutOrder        = 1
pTitle.Parent             = Content

local playerScroll = Instance.new("ScrollingFrame")
playerScroll.Size                 = UDim2.new(1, 0, 0, 40)
playerScroll.BackgroundColor3     = Color3.fromRGB(5, 22, 10)
playerScroll.BackgroundTransparency = 0.25
playerScroll.ScrollBarThickness   = 3
playerScroll.ScrollBarImageColor3 = GREEN
playerScroll.BorderSizePixel      = 0
playerScroll.CanvasSize           = UDim2.new()
playerScroll.AutomaticCanvasSize  = Enum.AutomaticSize.X
playerScroll.ScrollingDirection   = Enum.ScrollingDirection.X
playerScroll.LayoutOrder          = 2
playerScroll.Parent               = Content
corner(playerScroll, 9)
local pScrollStroke = mkStroke(playerScroll, GREEN, 1.5, 0.4)
pulsedStroke(pScrollStroke, 2.2, 0.2, 0.7, 0)

local pLL = Instance.new("UIListLayout")
pLL.FillDirection       = Enum.FillDirection.Horizontal
pLL.Padding             = UDim.new(0, 5)
pLL.VerticalAlignment   = Enum.VerticalAlignment.Center
pLL.Parent              = playerScroll

local pLP = Instance.new("UIPadding")
pLP.PaddingLeft   = UDim.new(0, 5)
pLP.PaddingRight  = UDim.new(0, 5)
pLP.PaddingTop    = UDim.new(0, 4)
pLP.PaddingBottom = UDim.new(0, 4)
pLP.Parent        = playerScroll

-- شاشة المستهدف
local targetDisplay = Instance.new("TextLabel")
targetDisplay.Size                   = UDim2.new(1, 0, 0, 26)
targetDisplay.BackgroundColor3       = Color3.fromRGB(8, 38, 16)
targetDisplay.BackgroundTransparency = 0.15
targetDisplay.Text                   = "🎯  المستهدف: الكل"
targetDisplay.TextColor3             = Color3.fromRGB(140, 255, 165)
targetDisplay.Font                   = Enum.Font.GothamBold
targetDisplay.TextSize               = 12
targetDisplay.BorderSizePixel        = 0
targetDisplay.LayoutOrder            = 3
targetDisplay.Parent                 = Content
corner(targetDisplay, 7)
gradient(targetDisplay, Color3.fromRGB(12, 55, 22), Color3.fromRGB(5, 25, 10), 90)
local tdS = mkStroke(targetDisplay, GREEN, 1, 0.45)
pulsedStroke(tdS, 2.5, 0.2, 0.65, 1)

-- ====================================================
--       2) دالة الأمر + زري وضع الاسم
-- ====================================================
local prefixSec = Instance.new("Frame")
prefixSec.Size                = UDim2.new(1, 0, 0, 46)
prefixSec.BackgroundTransparency = 1
prefixSec.LayoutOrder         = 4
prefixSec.Parent              = Content

local prefixLbl = Instance.new("TextLabel")
prefixLbl.Size               = UDim2.new(0, 90, 1, 0)
prefixLbl.BackgroundTransparency = 1
prefixLbl.Text               = "دالة الأمر:"
prefixLbl.TextColor3         = Color3.fromRGB(160, 255, 180)
prefixLbl.Font               = Enum.Font.GothamBold
prefixLbl.TextSize           = 12
prefixLbl.TextXAlignment     = Enum.TextXAlignment.Right
prefixLbl.Parent             = prefixSec

local prefixBox = Instance.new("TextBox")
prefixBox.Size              = UDim2.new(0, 48, 0, 34)
prefixBox.Position          = UDim2.new(0, 0, 0.5, -17)
prefixBox.Text              = ";"
prefixBox.BackgroundColor3  = Color3.fromRGB(5, 22, 10)
prefixBox.Font              = Enum.Font.GothamBold
prefixBox.TextSize          = 20
prefixBox.TextColor3        = GREEN
prefixBox.BorderSizePixel   = 0
prefixBox.ClearTextOnFocus  = false
prefixBox.Parent            = prefixSec
corner(prefixBox, 7)
mkStroke(prefixBox, GREEN, 1.5, 0.25)

local fullNameBtn = Instance.new("TextButton")
fullNameBtn.Size             = UDim2.new(0, 165, 0, 38)
fullNameBtn.Position         = UDim2.new(0, 58, 0.5, -19)
fullNameBtn.Text             = "👤 الاسم الكامل للاعب"
fullNameBtn.TextColor3       = WHITE
fullNameBtn.Font             = Enum.Font.GothamBold
fullNameBtn.TextSize         = 12
fullNameBtn.AutoButtonColor  = false
fullNameBtn.BorderSizePixel  = 0
fullNameBtn.Parent           = prefixSec
corner(fullNameBtn, 9)
gradient(fullNameBtn, Color3.fromRGB(30, 155, 65), Color3.fromRGB(12, 90, 35), 90)
mkStroke(fullNameBtn, WHITE, 1.5, 0.2)

local shortNameBtn = Instance.new("TextButton")
shortNameBtn.Size            = UDim2.new(0, 160, 0, 38)
shortNameBtn.Position        = UDim2.new(0, 235, 0.5, -19)
shortNameBtn.Text            = "✂️ أول 3 أحرف للاعب"
shortNameBtn.TextColor3      = WHITE
shortNameBtn.Font            = Enum.Font.GothamBold
shortNameBtn.TextSize        = 12
shortNameBtn.AutoButtonColor = false
shortNameBtn.BorderSizePixel = 0
shortNameBtn.Parent          = prefixSec
corner(shortNameBtn, 9)
gradient(shortNameBtn, Color3.fromRGB(15, 90, 38), Color3.fromRGB(6, 50, 18), 90)
mkStroke(shortNameBtn, WHITE, 1.5, 0.45)

local function updateModeButtons()
    if nameMode == "full" then
        gradient(fullNameBtn,  Color3.fromRGB(35, 180, 75), Color3.fromRGB(15, 110, 42), 90)
        gradient(shortNameBtn, Color3.fromRGB(10, 75, 30),  Color3.fromRGB(6,  45, 18),  90)
        fullNameBtn.BackgroundTransparency  = 0
        shortNameBtn.BackgroundTransparency = 0.25
    else
        gradient(shortNameBtn, Color3.fromRGB(35, 180, 75), Color3.fromRGB(15, 110, 42), 90)
        gradient(fullNameBtn,  Color3.fromRGB(10, 75, 30),  Color3.fromRGB(6,  45, 18),  90)
        shortNameBtn.BackgroundTransparency = 0
        fullNameBtn.BackgroundTransparency  = 0.25
    end
end

fullNameBtn.MouseButton1Click:Connect(function()
    nameMode = "full"; updateModeButtons()
end)
shortNameBtn.MouseButton1Click:Connect(function()
    nameMode = "short"; updateModeButtons()
end)
updateModeButtons()

-- ====================================================
--               3) الأقسام الثلاثة
-- ====================================================
local CMDS = {
    -- نسخ عادي
    {"LOGS","NV","RE","CLOGS","LOGS","NV","RE","CLOGS",
     "LOGS","NV","RE","CLOGS","LOGS","NV","RE","CLOGS",
     "LOGS","NV","RE","CLOGS","LOGS","NV","RE","CLOGS",
     "LOGS","NV","RE","CLOGS","LOGS","NV","RE","CLOGS"},
    -- سبام قوي
    {"JAIL","ICE","NV","RE","LOGS","RES","JUMP","FLY",
     "JAIL","ICE","NV","RE","LOGS","RES","JUMP","FLY",
     "JAIL","ICE","NV","RE","LOGS","RES","JUMP","FLY",
     "JAIL","ICE","NV","RE","LOGS","RES","JUMP","FLY",
     "JAIL","ICE","NV","RE","LOGS","RES","JUMP","FLY"},
    -- سبام غامض
    {"RE","EXPLODE","RES","LOGS","WARP","NV",
     "RE","EXPLODE","RES","LOGS","WARP","NV",
     "RE","EXPLODE","RES","LOGS","WARP","NV",
     "RE","EXPLODE","RES","LOGS","WARP","NV",
     "RE","EXPLODE","RES","LOGS","WARP","NV"},
}

local SECTION_INFO = {
    {label = "⚡  نسخ عادي",   c1 = Color3.fromRGB(18, 110, 45),  c2 = Color3.fromRGB(8,  60, 22)},
    {label = "💥  سبام قوي",   c1 = Color3.fromRGB(170, 70,  18),  c2 = Color3.fromRGB(110, 38, 8)},
    {label = "👻  سبام غامض",  c1 = Color3.fromRGB(60,  18, 140),  c2 = Color3.fromRGB(35,  8, 90)},
}

for i = 1, 3 do
    local info = SECTION_INFO[i]
    local cmds = CMDS[i]

    local sec = Instance.new("Frame")
    sec.Size                   = UDim2.new(1, 0, 0, 82)
    sec.BackgroundColor3       = Color3.fromRGB(6, 28, 12)
    sec.BackgroundTransparency = 0.15
    sec.BorderSizePixel        = 0
    sec.LayoutOrder            = 4 + i
    sec.Parent                 = Content
    corner(sec, 11)
    gradient(sec, info.c1, info.c2, 140)
    local secS = mkStroke(sec, WHITE, 1.5, 0.35)
    pulsedStroke(secS, 2, 0.15, 0.7, i * 1.1)

    local secLbl = Instance.new("TextLabel")
    secLbl.Size               = UDim2.new(1, -10, 0, 26)
    secLbl.Position           = UDim2.new(0, 5, 0, 6)
    secLbl.BackgroundTransparency = 1
    secLbl.Text               = info.label
    secLbl.TextColor3         = WHITE
    secLbl.Font               = Enum.Font.GothamBlack
    secLbl.TextSize           = 14
    secLbl.TextXAlignment     = Enum.TextXAlignment.Right
    secLbl.Parent             = sec

    local startBtn = Instance.new("TextButton")
    startBtn.Size             = UDim2.new(0.48, -4, 0, 40)
    startBtn.Position         = UDim2.new(0, 4, 0, 36)
    startBtn.Text             = "▶  تشغيل"
    startBtn.TextColor3       = WHITE
    startBtn.Font             = Enum.Font.GothamBold
    startBtn.TextSize         = 14
    startBtn.AutoButtonColor  = false
    startBtn.BorderSizePixel  = 0
    startBtn.BackgroundColor3 = Color3.fromRGB(18, 155, 55)
    startBtn.Parent           = sec
    corner(startBtn, 9)
    gradient(startBtn, Color3.fromRGB(25, 175, 65), Color3.fromRGB(10, 110, 38), 90)
    mkStroke(startBtn, WHITE, 1, 0.35)

    local stopBtn = Instance.new("TextButton")
    stopBtn.Size             = UDim2.new(0.48, -4, 0, 40)
    stopBtn.Position         = UDim2.new(0.5, 2, 0, 36)
    stopBtn.Text             = "⏹  إيقاف"
    stopBtn.TextColor3       = WHITE
    stopBtn.Font             = Enum.Font.GothamBold
    stopBtn.TextSize         = 14
    stopBtn.AutoButtonColor  = false
    stopBtn.BorderSizePixel  = 0
    stopBtn.BackgroundColor3 = Color3.fromRGB(175, 38, 38)
    stopBtn.Parent           = sec
    corner(stopBtn, 9)
    gradient(stopBtn, Color3.fromRGB(195, 45, 45), Color3.fromRGB(120, 20, 20), 90)
    mkStroke(stopBtn, WHITE, 1, 0.35)

    -- ✅ كل أمر يُرسل لحاله عبر InvokeServer مع فاصل 0.3 ثانية
    local idx = i
    startBtn.MouseButton1Click:Connect(function()
        if spamActive[idx] then return end
        spamActive[idx] = true
        startBtn.Text = "⏳  يعمل..."
        task.spawn(function()
            while spamActive[idx] do
                local prefix = (prefixBox.Text ~= "" and prefixBox.Text or ";")
                local target = nil
                if selectedPlayer then
                    target = (nameMode == "full")
                        and selectedPlayer
                        or  string.sub(selectedPlayer, 1, 3)
                end
                -- بناء رسالة واحدة كبيرة بكل الأوامر
                local parts = {}
                for _, cmd in ipairs(cmds) do
                    local finalCmd = prefix .. cmd
                    if target then finalCmd = finalCmd .. " " .. target end
                    table.insert(parts, finalCmd)
                end
                local bigMsg = table.concat(parts, " ")
                -- ترسل رسالة واحدة — تظهر بالشات + تنفّذ
                sendBigMessage(bigMsg)
                task.wait(1)
            end
            startBtn.Text = "▶  تشغيل"
        end)
    end)

    stopBtn.MouseButton1Click:Connect(function()
        spamActive[idx] = false
        startBtn.Text = "▶  تشغيل"
    end)
end

-- ====================================================
--             4) تحديث قائمة اللاعبين
-- ====================================================
local function flashTarget()
    TweenService:Create(targetDisplay, TweenInfo.new(0.12),
        {BackgroundColor3 = Color3.fromRGB(40, 180, 80)}):Play()
    task.delay(0.25, function()
        TweenService:Create(targetDisplay, TweenInfo.new(0.3),
            {BackgroundColor3 = Color3.fromRGB(8, 38, 16)}):Play()
    end)
end

local function refreshPlayers()
    for _, v in pairs(playerScroll:GetChildren()) do
        if v:IsA("TextButton") then v:Destroy() end
    end

    local allBtn = Instance.new("TextButton")
    allBtn.Size             = UDim2.new(0, 64, 0, 30)
    allBtn.Text             = "🌐 الكل"
    allBtn.Font             = Enum.Font.GothamBold
    allBtn.TextSize         = 11
    allBtn.AutoButtonColor  = false
    allBtn.BorderSizePixel  = 0
    allBtn.BackgroundColor3 = GREEN_MID
    allBtn.TextColor3       = WHITE
    allBtn.LayoutOrder      = 0
    allBtn.Parent           = playerScroll
    corner(allBtn, 7)
    mkStroke(allBtn, WHITE, 1.2, 0.3)

    task.spawn(function()
        while allBtn and allBtn.Parent do
            local t = tick()
            local p = (math.sin(t * 3) + 1) / 2
            local bri = math.floor(150 + p * 105)
            allBtn.TextColor3       = Color3.fromRGB(bri, bri, bri)
            allBtn.BackgroundColor3 = Color3.fromRGB(
                math.floor(25 + p * 35),
                math.floor(115 + p * 65),
                math.floor(45  + p * 35)
            )
            task.wait(0.04)
        end
    end)

    allBtn.MouseButton1Click:Connect(function()
        selectedPlayer = nil
        targetDisplay.Text = "🎯  المستهدف: الكل"
        flashTarget()
    end)

    for i, p in ipairs(Players:GetPlayers()) do
        local w   = math.max(72, #p.Name * 8 + 22)
        local btn = Instance.new("TextButton")
        btn.Size             = UDim2.new(0, w, 0, 30)
        btn.Text             = "👤 " .. p.Name
        btn.Font             = Enum.Font.GothamBold
        btn.TextSize         = 10
        btn.AutoButtonColor  = false
        btn.BorderSizePixel  = 0
        btn.BackgroundColor3 = Color3.fromRGB(6, 28, 12)
        btn.TextColor3       = WHITE
        btn.LayoutOrder      = i
        btn.Parent           = playerScroll
        corner(btn, 7)
        mkStroke(btn, WHITE, 1.2, 0.4)

        -- نبض أبيض/أسود لأسماء اللاعبين
        task.spawn(function()
            while btn and btn.Parent do
                local t  = tick()
                local pp = (math.sin(t * 3.5 + i * 0.9) + 1) / 2
                local bri = math.floor(80 + pp * 175)
                btn.TextColor3       = Color3.fromRGB(bri, bri, bri)
                btn.BackgroundColor3 = Color3.fromRGB(4, math.floor(22 + pp * 18), 8)
                task.wait(0.04)
            end
        end)

        btn.MouseButton1Click:Connect(function()
            selectedPlayer = p.Name
            targetDisplay.Text = "🎯  المستهدف: " .. p.Name
            flashTarget()
        end)
    end
end

refreshPlayers()
Players.PlayerAdded:Connect(refreshPlayers)
Players.PlayerRemoving:Connect(refreshPlayers)

-- ====== زر الدائرة الخضراء (للتحكم) ======
local ToggleCircle = Instance.new("TextButton", ScreenGui)
ToggleCircle.Size             = UDim2.new(0, 60, 0, 60)
ToggleCircle.Position         = UDim2.new(0.85, 0, 0.8, 0)
ToggleCircle.BackgroundColor3 = Color3.fromRGB(100, 255, 120)
ToggleCircle.Text             = "🌿"
ToggleCircle.TextColor3       = WHITE
ToggleCircle.TextSize         = 25
ToggleCircle.Font             = Enum.Font.GothamBold
ToggleCircle.AutoButtonColor  = false
ToggleCircle.ZIndex           = 100
corner(ToggleCircle, 999)
mkStroke(ToggleCircle, WHITE, 3, 0)
makeDraggable(ToggleCircle)

ToggleCircle.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

print("[MO+SH+BLO] ✅ جاهز — الريموت: HDAdminHDClient > RequestCommandModification")
