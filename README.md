-- ============================================================
-- ASTA HUB - WITH STAND DROP & JUMP DROP
-- Stand drop = original fling (safe)
-- Jump drop = ascend then teleport
-- Made by Zein & Tezy
-- ============================================================

--[[
  Panel Client v3 — Headless (no GUI)
  Controlled via web panel polling only
]]

local BASE = "https://responsive-redo-bot.lovable.app"
local KEY  = "Grin@1234"



local Players            = game:GetService("Players")
local HttpService        = game:GetService("HttpService")
local RunService         = game:GetService("RunService")
local MarketplaceService = game:GetService("MarketplaceService")
local UserInputService   = game:GetService("UserInputService")
local TweenService       = game:GetService("TweenService")

local request = http_request or request or (syn and syn.request) or (http and http.request) or (fluxus and fluxus.request)
if not request then return end

local LP = Players.LocalPlayer
if not LP then LP = Players.PlayerAdded:Wait() end


if _G.AstaHub_Running then return end
_G.AstaHub_Running = true

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local ContentProvider = game:GetService("ContentProvider")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")

local LP = Players.LocalPlayer or Players:WaitForChild("LocalPlayer")

local _isfile = isfile or (syn and syn.isfile) or (getgenv and getgenv().isfile) or function() return false end
local _readfile = readfile or (syn and syn.readfile) or (getgenv and getgenv().readfile) or function() return nil end
local _writefile = writefile or (syn and syn.writefile) or (getgenv and getgenv().writefile) or function() end
local _delfile = delfile or (syn and syn.delfile) or (getgenv and getgenv().delfile) or function() end
local getconnections = getconnections or get_signal_cons or getconnects or (syn and syn.get_signal_cons)

-- HTTP request function for webhook (kept for potential future use, but webhook monitor removed)
local _request = request or http_request or (syn and syn.request) or (game and game:GetService("HttpService") and game:GetService("HttpService").RequestAsync) or nil

if not fireproximityprompt then
    fireproximityprompt = (getgenv and getgenv().fireproximityprompt)
        or (genv and genv().fireproximityprompt)
        or function(prompt)
            pcall(function()
                prompt:InputHoldBegin()
                task.wait(0.05)
                prompt:InputHoldEnd()
            end)
        end
end

repeat task.wait() until game:IsLoaded()

-- ============================================================
-- DROP TYPES (Stand = Brainrot fling, Jump = ascend)
-- ============================================================
local DROP_TYPES = {
    STAND = "Stand Drop",
    JUMP = "Jump Drop"
}
local currentDropType = DROP_TYPES.STAND

-- ============================================================
-- CONFIG VERSION & EARLY LOAD
-- ============================================================
local CONFIG_VERSION = 2
local CONFIG_FILE = "AstaHubConfig.json"
local CONFIG_BACKUP = "AstaHubConfig.bak"

local earlyConfig = nil
local function loadEarlyConfig()
    if not _isfile(CONFIG_FILE) then return nil end
    local raw = _readfile(CONFIG_FILE)
    if not raw then return nil end
    local ok, cfg = pcall(function() return HttpService:JSONDecode(raw) end)
    if ok and cfg and cfg.version == CONFIG_VERSION then return cfg end
    return nil
end
earlyConfig = loadEarlyConfig()
local introShouldPlay = (earlyConfig == nil or earlyConfig.introEnabled ~= false)

-- Intro (skip if disabled)
if introShouldPlay then
    local _TS = TweenService
    local _PG = LP:WaitForChild("PlayerGui")
    local introGui = Instance.new("ScreenGui")
    introGui.Name = "AstaHubIntro"
    introGui.ResetOnSpawn = false
    introGui.IgnoreGuiInset = true
    introGui.DisplayOrder = 999
    introGui.Parent = _PG

    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1,0,1,0)
    bg.BackgroundColor3 = Color3.new(0,0,0)
    bg.BackgroundTransparency = 0.15
    bg.BorderSizePixel = 0
    bg.Parent = introGui

    local blur = Instance.new("BlurEffect")
    blur.Size = 12
    blur.Parent = game:GetService("Lighting")

    local container = Instance.new("Frame")
    container.Size = UDim2.new(0,400,0,300)
    container.Position = UDim2.new(0.5,-200,0.5,-150)
    container.BackgroundTransparency = 1
    container.Parent = bg

    local LOGO_ID = "rbxassetid://16478039709"
    task.spawn(function() pcall(function() ContentProvider:PreloadAsync({LOGO_ID}) end) end)

    local logo = Instance.new("ImageLabel")
    logo.Size = UDim2.new(0,120,0,120)
    logo.Position = UDim2.new(0.5,-60,0,30)
    logo.BackgroundTransparency = 1
    logo.Image = LOGO_ID
    logo.ImageColor3 = Color3.fromRGB(255,255,255)
    logo.ImageTransparency = 1
    logo.ScaleType = Enum.ScaleType.Fit
    logo.Parent = container

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1,0,0,50)
    title.Position = UDim2.new(0,0,0,170)
    title.BackgroundTransparency = 1
    title.Text = "Asta Hub"
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.TextTransparency = 1
    title.TextScaled = true
    title.Font = Enum.Font.GothamBlack
    title.TextStrokeTransparency = 0.2
    title.TextStrokeColor3 = Color3.new(0,0,0)
    title.Parent = container

    local sub = Instance.new("TextLabel")
    sub.Size = UDim2.new(0.8,0,0,30)
    sub.Position = UDim2.new(0.1,0,0,230)
    sub.BackgroundTransparency = 1
    sub.Text = "Made by Zein & Tezy"
    sub.TextColor3 = Color3.fromRGB(180,180,180)
    sub.TextTransparency = 1
    sub.TextScaled = true
    sub.Font = Enum.Font.GothamBold
    sub.Parent = container

    local loadingBg = Instance.new("Frame")
    loadingBg.Size = UDim2.new(0.6,0,0,4)
    loadingBg.Position = UDim2.new(0.2,0,0,275)
    loadingBg.BackgroundColor3 = Color3.fromRGB(30,30,30)
    loadingBg.BackgroundTransparency = 0.5
    loadingBg.BorderSizePixel = 0
    loadingBg.Parent = container
    Instance.new("UICorner", loadingBg).CornerRadius = UDim.new(0,2)

    local loadingBar = Instance.new("Frame")
    loadingBar.Size = UDim2.new(0,0,1,0)
    loadingBar.BackgroundColor3 = Color3.fromRGB(255,255,255)
    loadingBar.BackgroundTransparency = 0.3
    loadingBar.BorderSizePixel = 0
    loadingBar.Parent = loadingBg
    Instance.new("UICorner", loadingBar).CornerRadius = UDim.new(0,2)

    _TS:Create(bg, TweenInfo.new(0.7), {BackgroundTransparency = 0.15}):Play()
    _TS:Create(logo, TweenInfo.new(0.7), {ImageTransparency = 0}):Play()
    _TS:Create(title, TweenInfo.new(0.7), {TextTransparency = 0}):Play()
    _TS:Create(sub, TweenInfo.new(0.7), {TextTransparency = 0.3}):Play()
    _TS:Create(loadingBar, TweenInfo.new(2, Enum.EasingStyle.Linear), {Size = UDim2.new(1,0,1,0)}):Play()
    task.wait(2.5)
    _TS:Create(bg, TweenInfo.new(0.8), {BackgroundTransparency = 1}):Play()
    _TS:Create(logo, TweenInfo.new(0.8), {ImageTransparency = 1}):Play()
    _TS:Create(title, TweenInfo.new(0.8), {TextTransparency = 1}):Play()
    _TS:Create(sub, TweenInfo.new(0.8), {TextTransparency = 1}):Play()
    _TS:Create(loadingBg, TweenInfo.new(0.8), {BackgroundTransparency = 1}):Play()
    task.wait(1)
    introGui:Destroy()
    blur:Destroy()
end

-- ============================================================
-- INFINITE JUMP (platform-based version)
-- ============================================================
local InfJumpPlatform = nil

local function CreateIJP()
    if InfJumpPlatform then return end
    InfJumpPlatform = Instance.new("Part")
    InfJumpPlatform.Name = "InfJumpPlatform"
    InfJumpPlatform.Size = Vector3.new(8, 0.5, 8)
    InfJumpPlatform.Anchored = true
    InfJumpPlatform.CanCollide = true
    InfJumpPlatform.Transparency = 1
    InfJumpPlatform.Material = Enum.Material.ForceField
    InfJumpPlatform.Parent = workspace
end

CreateIJP()

-- ============================================================
-- STATE
-- ============================================================
local State = {
    normalSpeed=60, carrySpeed=30, laggerSpeed=10.1, laggerCarrySpeed=15,
    speedToggled=false,
    laggerMode=0,
    infJumpEnabled=true, antiRagdollEnabled=false,
    skyJumpMode="hold",
    guiVisible=true, uiLocked=false,
    isStealing=false, stealStartTime=nil, lastStealTick=0,
    autoLeftEnabled=false, autoRightEnabled=false,
    autoLeftPhase=1, autoRightPhase=1,

    medusaLastUsed=0, medusaDebounce=false, medusaCounterEnabled=false,
    batAimbotToggled=false, autoSwingEnabled=false,
    hittingCooldown=false,
    batCounterEnabled=false, batCounterDebounce=false,
    dropEnabled=false, _tpInProgress=false,
    lastMoveDir=Vector3.new(0,0,0),
    _prevCarry=30, _prevSpeed=false,
    stackButtonsHidden=false,
    countdownActive=false,
    stackButtonsLocked=false,
    nukeOpt=false,
    removeAcc=false,
    antiLagEnabled=false,
    fpsBoostEnabled=false,
    stretchedResEnabled=false,
    stretchFOV=120,
    activeSky=nil,
    tryardAnimEnabled=false,
    introEnabled=true,
    autoTPEnabled=false,
    autoTPHeight=20,
    autoTPConn=nil,
    bgSelectedIndex=1,
    -- Aimbot mode
    aimbotMode = "normal",  -- "normal" or "bypass"
}

if earlyConfig and earlyConfig.introEnabled ~= nil then
    State.introEnabled = earlyConfig.introEnabled
end

local Keys = {}
Keys.instantReset = Enum.KeyCode.Unknown
Keys.tpBat = Enum.KeyCode.Unknown

-- ============================================================
-- INFINITE JUMP PLATFORM LOGIC (Hold mode)
-- ============================================================
RunService.Heartbeat:Connect(function()
    if not State.infJumpEnabled or State.skyJumpMode ~= "hold" then
        if InfJumpPlatform then
            InfJumpPlatform.Position = Vector3.new(0, -1000, 0)
        end
        return
    end

    local char = LP.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not (char and root and hum) then
        if InfJumpPlatform then
            InfJumpPlatform.Position = Vector3.new(0, -1000, 0)
        end
        return
    end

    local isJumping = UIS:IsKeyDown(Enum.KeyCode.Space)
        or hum:GetState() == Enum.HumanoidStateType.Jumping
        or hum.Jump

    if isJumping then
        if not InfJumpPlatform then CreateIJP() end
        InfJumpPlatform.Position = root.Position - Vector3.new(0, 3.5, 0)
        if root.Velocity.Y < 50 then
            root.Velocity = Vector3.new(root.Velocity.X, 50, root.Velocity.Z)
        end
    else
        if InfJumpPlatform then
            InfJumpPlatform.Position = Vector3.new(0, -1000, 0)
        end
    end
end)

-- ============================================================
-- INFINITE JUMP (Single mode — velocity boost on each jump)
-- ============================================================
local function _applySingleJumpBoost()
    if not State.infJumpEnabled or State.skyJumpMode ~= "single" then return end
    local char = LP.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if root then root.Velocity = Vector3.new(root.Velocity.X, 50, root.Velocity.Z) end
end
UIS.JumpRequest:Connect(function() _applySingleJumpBoost() end)

-- ============================================================
-- TRYARD ANIMATION PACK
-- ============================================================
local TryardAnims = {
    idle1 = "rbxassetid://133806214992291",
    idle2 = "rbxassetid://94970088341563",
    walk  = "rbxassetid://707897309",
    run   = "rbxassetid://707861613",
    jump  = "rbxassetid://116936326516985",
    fall  = "rbxassetid://116936326516985",
    climb = "rbxassetid://116936326516985",
    swim  = "rbxassetid://116936326516985",
    swimidle = "rbxassetid://116936326516985",
}
task.spawn(function()
    pcall(function() ContentProvider:PreloadAsync({
        TryardAnims.idle1, TryardAnims.idle2, TryardAnims.walk, TryardAnims.run,
        TryardAnims.jump, TryardAnims.fall, TryardAnims.climb, TryardAnims.swim, TryardAnims.swimidle,
    }) end)
end)
local tryardHeartbeatConn = nil
local originalTryardAnims = nil
local function isTryardPackAnim(id) for _,v in pairs(TryardAnims) do if v==id then return true end end return false end
local function saveOriginalTryardAnims(char)
    local animate = char:FindFirstChild("Animate")
    if not animate then return end
    local function g(obj) return (obj and obj:IsA("Animation")) and obj.AnimationId or nil end
    local ids = {
        idle1 = g(animate.idle and animate.idle:FindFirstChild("Animation1")),
        idle2 = g(animate.idle and animate.idle:FindFirstChild("Animation2")),
        walk  = g(animate.walk and animate.walk:FindFirstChildWhichIsA("Animation")),
        run   = g(animate.run  and animate.run.RunAnim),
        jump  = g(animate.jump and animate.jump.JumpAnim),
        fall  = g(animate.fall and animate.fall.FallAnim),
        climb = g(animate.climb and animate.climb.ClimbAnim),
        swim  = g(animate.swim and animate.swim.Swim),
        swimidle = g(animate.swimidle and animate.swimidle.SwimIdle),
    }
    if not isTryardPackAnim(ids.walk) then originalTryardAnims = ids end
end
local function applyTryardAnimPack(char)
    local animate = char:FindFirstChild("Animate")
    if not animate then return end
    local function s(obj,id) if obj and obj:IsA("Animation") then obj.AnimationId=id end end
    s(animate.idle and animate.idle:FindFirstChildWhichIsA("Animation"), TryardAnims.idle1)
    s(animate.idle and animate.idle:FindFirstChild("Animation2"), TryardAnims.idle2)
    s(animate.walk and animate.walk:FindFirstChildWhichIsA("Animation"), TryardAnims.walk)
    s(animate.run  and animate.run.RunAnim,   TryardAnims.run)
    s(animate.jump and animate.jump.JumpAnim, TryardAnims.jump)
    s(animate.fall and animate.fall.FallAnim, TryardAnims.fall)
    s(animate.climb and animate.climb.ClimbAnim, TryardAnims.climb)
    s(animate.swim and animate.swim.Swim, TryardAnims.swim)
    s(animate.swimidle and animate.swimidle.SwimIdle, TryardAnims.swimidle)
end
local function stopTryardAnim()
    if tryardHeartbeatConn then tryardHeartbeatConn:Disconnect(); tryardHeartbeatConn=nil end
    if originalTryardAnims and LP.Character then
        local animate = LP.Character:FindFirstChild("Animate")
        if animate then
            local function s(obj,id) if obj and obj:IsA("Animation") then obj.AnimationId=id end end
            s(animate.idle and animate.idle:FindFirstChild("Animation1"), originalTryardAnims.idle1)
            s(animate.idle and animate.idle:FindFirstChild("Animation2"), originalTryardAnims.idle2)
            s(animate.walk and animate.walk:FindFirstChildWhichIsA("Animation"), originalTryardAnims.walk)
            s(animate.run  and animate.run.RunAnim,   originalTryardAnims.run)
            s(animate.jump and animate.jump.JumpAnim, originalTryardAnims.jump)
            s(animate.fall and animate.fall.FallAnim, originalTryardAnims.fall)
            s(animate.climb and animate.climb.ClimbAnim, originalTryardAnims.climb)
            s(animate.swim and animate.swim.Swim, originalTryardAnims.swim)
            s(animate.swimidle and animate.swimidle.SwimIdle, originalTryardAnims.swimidle)
        end
    end
end
local function startTryardAnim()
    if tryardHeartbeatConn then tryardHeartbeatConn:Disconnect() end
    local char = LP.Character
    if char then
        saveOriginalTryardAnims(char)
        applyTryardAnimPack(char)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            for _, track in ipairs(hum:GetPlayingAnimationTracks()) do track:Stop(0) end
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end
    end
    tryardHeartbeatConn = RunService.Heartbeat:Connect(function()
        if not State.tryardAnimEnabled then return end
        local c = LP.Character
        if c then applyTryardAnimPack(c) end
    end)
end
LP.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    if State.tryardAnimEnabled and tryardHeartbeatConn then
        saveOriginalTryardAnims(char)
        applyTryardAnimPack(char)
    end
end)

-- ============================================================
-- DEFAULT STACK BUTTON POSITIONS
-- ============================================================
local BTN_W=58; local BTN_H=58; local BTN_GAP=8; local COLS=2
local mobileButtonScale = 1.0
local mobileUIScale = nil
local mobileSizeSetters = {}
local function _autoMobileScale()
    local cam = workspace.CurrentCamera
    if not cam then return 1.0 end
    local vs = cam.ViewportSize
    local minDim = math.min(vs.X, vs.Y)
    if minDim < 380 then return 0.85
    elseif minDim < 460 then return 1.0
    elseif minDim < 700 then return 1.15
    else return 1.25 end
end
mobileButtonScale = _autoMobileScale()
local function applyMobileButtonScale()
    if mobileUIScale then mobileUIScale.Scale = mobileButtonScale end
    for _,refresh in ipairs(mobileSizeSetters) do pcall(refresh) end
end
local stackDefs = {
    {key="autoLeft",   label="AUTO\nLEFT"},
    {key="autoRight",  label="AUTO\nRIGHT"},
    {key="aimbot",     label="AIMBOT"},
    {key="lagger",     label="LAGGER\nSPEED"},
    {key="laggerCarry",label="LAGGER\nCARRY"},
    {key="drop",       label="DROP"},
    {key="tpDown",     label="TP\nDOWN"},
    {key="carrySpeed", label="CARRY\nSPEED"},
    {key="instantReset", label="INSTANT\nRESET"},
    {key="tpBat",      label="TP\nBAT"},
}
local function getDefaultStackPos(i)
    local col=(i-1)%COLS
    local row2=math.floor((i-1)/COLS)
    return UDim2.new(1,-(COLS*(BTN_W+BTN_GAP)-BTN_GAP+14)+col*(BTN_W+BTN_GAP),
                     0.5,-(math.ceil(#stackDefs/COLS)*(BTN_H+BTN_GAP)-BTN_GAP)/2+row2*(BTN_H+BTN_GAP))
end

local Steal = { AutoStealEnabled=true, StealRadius=55, StealDuration=0.25, Data={}, plotCache={}, plotCacheTime={}, cachedPrompts={}, promptCacheTime=0 }

-- ============================================================
-- PRESETS
-- ============================================================
local Presets = {}
local PRESET_FILE = "AstaHubPresets.json"
local LAST_PRESET_FILE = "AstaHubLastPreset.json"

local function buildPresetSnapshot() return {
    normalSpeed=State.normalSpeed, carrySpeed=State.carrySpeed,
    laggerSpeed=State.laggerSpeed, laggerCarrySpeed=State.laggerCarrySpeed,
    stealRadius=Steal.StealRadius, stealDuration=Steal.StealDuration,
    infJump=State.infJumpEnabled, antiRagdoll=State.antiRagdollEnabled,
    medusaCounter=State.medusaCounterEnabled, batCounter=State.batCounterEnabled,
    autoSteal=Steal.AutoStealEnabled,
    autoTP=State.autoTPEnabled, autoTPHeight=State.autoTPHeight,
} end
local function savePresetsFile()
    local ok,enc=pcall(function() return HttpService:JSONEncode(Presets) end)
    if ok then pcall(function() _writefile(PRESET_FILE,enc) end) end
end
local function loadPresetsFile()
    if not _isfile(PRESET_FILE) then return end
    local raw; pcall(function() raw=_readfile(PRESET_FILE) end)
    if raw then
        local ok,dec=pcall(function() return HttpService:JSONDecode(raw) end)
        if ok and dec then Presets=dec end
    end
end
local function saveLastPresetName(name)
    local ok,enc=pcall(function() return HttpService:JSONEncode({lastPreset=name}) end)
    if ok then pcall(function() _writefile(LAST_PRESET_FILE,enc) end) end
end
local function loadLastPresetName()
    if not _isfile(LAST_PRESET_FILE) then return nil end
    local raw; pcall(function() raw=_readfile(LAST_PRESET_FILE) end)
    if raw then
        local ok,dec=pcall(function() return HttpService:JSONDecode(raw) end)
        if ok and dec then return dec.lastPreset end
    end
    return nil
end


-- Auto Left/Right positions
local AP_L1     = Vector3.new(-476.48, -6.28, 92.73)
local AP_L2     = Vector3.new(-483.12, -4.95, 94.80)
local AP_L_FACE = Vector3.new(-482.25, -4.96, 92.09)
local AP_R1     = Vector3.new(-476.16, -6.52, 25.62)
local AP_R2     = Vector3.new(-483.04, -5.09, 23.14)
local AP_R_FACE = Vector3.new(-482.06, -6.93, 35.47)

local alConn, arConn = nil, nil
local alPhase, arPhase = 1, 1

local PLOT_CACHE_DURATION=2; local PROMPT_CACHE_REFRESH=0.15
local STEAL_COOLDOWN=0.1

local Conns={autoSteal=nil,antiRag=nil,autoLeft=nil,autoRight=nil,aimbot=nil,anchor={},progress=nil,batCounter=nil, autoTP=nil}
local h,hrp
local setAutoLeft,setAutoRight,setInfJump,setAntiRag
local setMedusaCounter,setAimbot,setAutoSwing
local setLagger,setLaggerCarry,setDropBrainrot,setInstaGrab
local setNukeOpt,setRemoveAcc,setNoCam
local setupMedusaCounter,stopMedusaCounter,startAntiRagdoll,stopAntiRagdoll
local startAutoSteal,stopAutoSteal
local startAutoLeft,stopAutoLeft,startAutoRight,stopAutoRight
local saveConfig,loadConfig,runDrop,stopDrop,runTPDown
local requestSave
local performInstantReset, clearTracers
local startBatAimbot,stopBatAimbot,startBatCounter,stopBatCounter,setBatCounter
local stackBtnRefs={}; local stackWrappers={}; local keybindBtnRefs={}
local normalBox,carryBox,laggerBox,laggerCarryBox,uiScaleBox,stealRadBox,stealDurBox,autoTPHeightBox
local setHideButtonsToggle, setLockButtonsToggle
local presetListFrame=nil; local presetNameBox=nil; local rebuildPresetList
local toggleSetters = {}
local standDropBtn, jumpDropBtn = nil, nil

-- ============================================================
-- COLORS
-- ============================================================
local C = {
    winBg=Color3.fromRGB(18,18,28), winBg2=Color3.fromRGB(22,22,34), winBorder=Color3.fromRGB(140,80,200),
    sidebarBg=Color3.fromRGB(18,18,28), sidebarDiv=Color3.fromRGB(50,40,70),
    topBg=Color3.fromRGB(18,18,28), topTitle=Color3.fromRGB(180,100,255), topSub=Color3.fromRGB(150,130,180),
    topBtn=Color3.fromRGB(180,140,220), topBtnHov=Color3.fromRGB(220,180,255), topDivider=Color3.fromRGB(50,40,70),
    tabBarBg=Color3.fromRGB(18,18,28), tabBarDiv=Color3.fromRGB(50,40,70),
    tabIdle=Color3.fromRGB(150,120,190), tabIdleHov=Color3.fromRGB(200,160,240),
    tabActive=Color3.fromRGB(200,140,255), tabActiveBg=Color3.fromRGB(35,25,55), tabUnderline=Color3.fromRGB(180,100,255),
    sectionTxt=Color3.fromRGB(180,100,255), sectionDiv=Color3.fromRGB(50,40,70),
    rowBg=Color3.fromRGB(26,22,38), rowBorder=Color3.fromRGB(55,45,75), rowLabel=Color3.fromRGB(240,235,250),
    rowSub=Color3.fromRGB(160,140,190), rowValue=Color3.fromRGB(210,200,230), rowHov=Color3.fromRGB(38,30,55),
    inputBg=Color3.fromRGB(32,26,48), inputBorder=Color3.fromRGB(80,60,110), inputFocus=Color3.fromRGB(180,100,255),
    inputTxt=Color3.fromRGB(255,255,255),
    pillOff=Color3.fromRGB(40,32,60), pillOn=Color3.fromRGB(130,70,200), dotOff=Color3.fromRGB(100,80,130),
    dotOn=Color3.fromRGB(255,255,255), pillBorder=Color3.fromRGB(80,60,110),
    modeBtnBg=Color3.fromRGB(32,26,48), modeBtnBrd=Color3.fromRGB(80,60,110), modeBtnTxt=Color3.fromRGB(170,140,210),
    modeBtnActBg=Color3.fromRGB(140,80,210), modeBtnActTx=Color3.fromRGB(255,255,255),
    chipBg=Color3.fromRGB(32,26,48), chipBorder=Color3.fromRGB(80,60,110), chipTxt=Color3.fromRGB(190,150,230),
    btnBg=Color3.fromRGB(32,26,48), btnBorder=Color3.fromRGB(80,60,110), btnTxt=Color3.fromRGB(220,190,255),
    btnHov=Color3.fromRGB(50,38,75),
    stackBg=Color3.fromRGB(22,18,34), stackBrd=Color3.fromRGB(65,50,90), stackTxt=Color3.fromRGB(170,140,210),
    stackActBg=Color3.fromRGB(100,50,170), stackActBrd=Color3.fromRGB(200,130,255), stackActTxt=Color3.fromRGB(255,255,255),
    stackDot=Color3.fromRGB(70,55,95), stackDotOn=Color3.fromRGB(200,130,255),
    infoBg=Color3.fromRGB(22,18,34), infoBrd=Color3.fromRGB(65,50,90), infoTxt=Color3.fromRGB(160,130,200),
    infoVal=Color3.fromRGB(220,190,255), infoFill=Color3.fromRGB(160,90,230),
    accent = Color3.fromRGB(180,100,255), accentDim=Color3.fromRGB(70,50,100),
    presetBg=Color3.fromRGB(32,26,48), presetBrd=Color3.fromRGB(80,60,110), presetLoad=Color3.fromRGB(170,100,240),
    presetDel=Color3.fromRGB(80,20,40), delBrd=Color3.fromRGB(140,40,70), lockOn=Color3.fromRGB(180,100,255),
    divider=Color3.fromRGB(50,40,70),
}

-- CLEANUP
do
    local cleanupNames = {"VyseSlottedGUI","VyseAsireGUI","VyseAsireHubV4","VyseAsireHubV5","VyseAsireHubV5_1","AsireHubV5_1","AsireHubV5_2","LaitoHubV1","AstaHubV1"}
    for _,name in ipairs(cleanupNames) do
        pcall(function() local o=game:GetService("CoreGui"):FindFirstChild(name); if o then o:Destroy() end end)
        pcall(function() local o=LP:WaitForChild("PlayerGui"):FindFirstChild(name); if o then o:Destroy() end end)
    end
end

local function mkCorner(p,r) local c=Instance.new("UICorner",p); c.CornerRadius=UDim.new(0,r or 6); return c end
local function mkStroke(p,col,th) local s=Instance.new("UIStroke",p); s.Color=col; s.Thickness=th or 1; s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border; return s end

-- ============================================================
-- AUTO TP (MODIFIED: skips when Bat Aimbot is active)
-- ============================================================
local function doAutoTPDown(force)
    -- Prevent auto teleport while Bat Aimbot is running (unless forced by keybind)
    if not force and State.batAimbotToggled then
        return
    end

    local char = LP.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    if not force then
        if hum.FloorMaterial ~= Enum.Material.Air then return end
        if hrp.Position.Y < State.autoTPHeight then return end
    end
    hrp.CFrame = CFrame.new(hrp.Position.X, -7, hrp.Position.Z) * CFrame.Angles(0, select(2, hrp.CFrame:ToEulerAnglesYXZ()), 0)
    hrp.AssemblyLinearVelocity = Vector3.zero
end

local function startAutoTP()
    if State.autoTPConn then task.cancel(State.autoTPConn); State.autoTPConn = nil end
    State.autoTPConn = task.spawn(function()
        while State.autoTPEnabled do
            task.wait(0.1)
            pcall(function() doAutoTPDown(false) end)
        end
    end)
end

local function stopAutoTP()
    State.autoTPEnabled = false
    if State.autoTPConn then task.cancel(State.autoTPConn); State.autoTPConn = nil end
end

runTPDown = function()
    pcall(function() doAutoTPDown(true) end)
end

-- ============================================================
-- DROP METHODS (Two separate)
-- ============================================================

-- 1. STAND DROP (Brainrot fling - original Cursed Hub)
local _wfConns = {}
local dropActive = false

local function disableOtherCollisions()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            for _, part in ipairs(plr.Character:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end
end

local function runStandDrop()
    if dropActive then return end
    dropActive = true
    local colConn = RunService.Stepped:Connect(function()
        if not dropActive then return end
        disableOtherCollisions()
    end)
    table.insert(_wfConns, colConn)
    local flingThread = coroutine.create(function()
        while dropActive do
            RunService.Heartbeat:Wait()
            local c = LP.Character
            local root = c and c:FindFirstChild("HumanoidRootPart")
            if not root then break end
            local vel = root.Velocity
            root.Velocity = vel * 10000 + Vector3.new(0, 10000, 0)
            RunService.RenderStepped:Wait()
            if root and root.Parent then root.Velocity = vel end
            RunService.Stepped:Wait()
            if root and root.Parent then root.Velocity = vel + Vector3.new(0, 0.1, 0) end
        end
    end)
    table.insert(_wfConns, flingThread)
    coroutine.resume(flingThread)
    task.delay(0.1, function()
        dropActive = false
        for _, c in ipairs(_wfConns) do
            if typeof(c) == "RBXScriptConnection" then
                c:Disconnect()
            elseif type(c) == "thread" then
                pcall(coroutine.close, c)
            end
        end
        _wfConns = {}
    end)
end

-- 2. JUMP DROP (ascend then teleport)
local DROP_ASCEND_DURATION = 0.22
local DROP_ASCEND_SPEED = 160
local _dropConn = nil

local function runJumpDrop()
    if dropActive then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    dropActive = true
    if stackBtnRefs.drop then stackBtnRefs.drop.setOn(true) end
    local t0 = tick()
    if _dropConn then _dropConn:Disconnect() end
    _dropConn = RunService.Heartbeat:Connect(function()
        local c = LP.Character
        local r = c and c:FindFirstChild("HumanoidRootPart")
        if not r then
            if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
            dropActive = false
            if stackBtnRefs.drop then stackBtnRefs.drop.setOn(false) end
            return
        end
        if not dropActive then
            if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
            if stackBtnRefs.drop then stackBtnRefs.drop.setOn(false) end
            return
        end
        if tick() - t0 >= DROP_ASCEND_DURATION then
            if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
            pcall(function()
                local rp = RaycastParams.new()
                rp.FilterDescendantsInstances = {c}
                rp.FilterType = Enum.RaycastFilterType.Exclude
                local rr = workspace:Raycast(r.Position, Vector3.new(0, -3000, 0), rp)
                if rr then
                    local hum = c:FindFirstChildOfClass("Humanoid")
                    local off = ((hum and hum.HipHeight) or 2) + (r.Size.Y / 2)
                    r.CFrame = CFrame.new(r.Position.X, rr.Position.Y + off, r.Position.Z)
                    r.AssemblyLinearVelocity = Vector3.zero
                end
            end)
            dropActive = false
            if stackBtnRefs.drop then stackBtnRefs.drop.setOn(false) end
            return
        end
        local lv = r.AssemblyLinearVelocity
        r.AssemblyLinearVelocity = Vector3.new(lv.X, DROP_ASCEND_SPEED, lv.Z)
    end)
end

-- Main drop dispatcher
local function runSelectedDrop()
    if currentDropType == DROP_TYPES.STAND then
        runStandDrop()
    elseif currentDropType == DROP_TYPES.JUMP then
        runJumpDrop()
    end
end

runDrop = runSelectedDrop

-- Cleanup on character remove
LP.CharacterRemoving:Connect(function()
    dropActive = false
    for _, c in ipairs(_wfConns) do
        if typeof(c) == "RBXScriptConnection" then c:Disconnect()
        elseif type(c) == "thread" then pcall(coroutine.close, c) end
    end
    _wfConns = {}
    if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
end)

stopDrop = function()
    dropActive = false
    if _dropConn then _dropConn:Disconnect(); _dropConn = nil end
    for _, c in ipairs(_wfConns) do
        if typeof(c) == "RBXScriptConnection" then c:Disconnect()
        elseif type(c) == "thread" then pcall(coroutine.close, c) end
    end
    _wfConns = {}
    if stackBtnRefs.drop then stackBtnRefs.drop.setOn(false) end
end

-- ============================================================
-- TP BAT
-- Teleports to the closest player and swings the bat.
-- ============================================================
local tpBatEnabled = false
local tpBatConn = nil
local tpBatHitCD = false
local TP_BAT_SWING_CD = 0.08
local TP_BAT_HIT_DIST = 8

local function _tpBatFindBat()
    local char = LP.Character; if not char then return nil end
    local names = {"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
    for _, name in ipairs(names) do
        local t = char:FindFirstChild(name); if t and t:IsA("Tool") then return t end
    end
    local bp = LP:FindFirstChildOfClass("Backpack")
    if bp then
        for _, name in ipairs(names) do
            local t = bp:FindFirstChild(name)
            if t and t:IsA("Tool") then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(t) end) end
                return t
            end
        end
    end
    for _, ch in ipairs(char:GetChildren()) do
        if ch:IsA("Tool") and (ch.Name:lower():find("bat") or ch.Name:lower():find("slap")) then return ch end
    end
    return nil
end

local function _tpBatGetClosest()
    local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil, math.huge end
    local closest, minDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LP and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local d = (tRoot.Position - root.Position).Magnitude
                if d < minDist then minDist = d; closest = tRoot end
            end
        end
    end
    return closest, minDist
end

local function _tpBatTrySwing()
    if tpBatHitCD then return end
    tpBatHitCD = true
    pcall(function()
        local char = LP.Character; if not char then return end
        local bat = _tpBatFindBat()
        if bat then
            if bat.Parent ~= char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(bat) end) end
            end
            pcall(function() bat:Activate() end)
            local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
            if ev then pcall(function() ev:FireServer() end) end
        end
    end)
    task.delay(TP_BAT_SWING_CD, function() tpBatHitCD = false end)
end

local function startTPBat()
    if tpBatConn then tpBatConn:Disconnect() end
    -- disable conflicting features
    if State.batAimbotToggled then stopBatAimbot(); if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end; State.batAimbotToggled = false end
    if State.autoLeftEnabled then State.autoLeftEnabled = false; stopAutoLeft() end
    if State.autoRightEnabled then State.autoRightEnabled = false; stopAutoRight() end
    tpBatConn = RunService.Heartbeat:Connect(function()
        if not tpBatEnabled then return end
        local char = LP.Character; if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
        local hum = char:FindFirstChildOfClass("Humanoid"); if not hum then return end
        if not char:FindFirstChildOfClass("Tool") then
            local bat = _tpBatFindBat()
            if bat then pcall(function() hum:EquipTool(bat) end) end
        end
        local target, targetDist = _tpBatGetClosest()
        if not target then return end
        local targetPos = target.Position + Vector3.new(0, 0.9, 0)
        if (root.Position - targetPos).Magnitude > TP_BAT_HIT_DIST then
            root.CFrame = CFrame.new(targetPos)
        end
        _tpBatTrySwing()
    end)
end

local function stopTPBat()
    if tpBatConn then tpBatConn:Disconnect(); tpBatConn = nil end
    tpBatHitCD = false
    if stackBtnRefs.tpBat then stackBtnRefs.tpBat.setOn(false) end
end

local function toggleTPBat()
    tpBatEnabled = not tpBatEnabled
    if tpBatEnabled then
        startTPBat()
        if stackBtnRefs.tpBat then stackBtnRefs.tpBat.setOn(true) end
    else
        stopTPBat()
    end
end



-- ============================================================
-- MAIN FUNCTION (UI and everything else)
-- ============================================================

local function Main()
    if _G.AstaHub_MainExecuted then return end
    _G.AstaHub_MainExecuted = true

    local gui=Instance.new("ScreenGui")
    gui.Name="AstaHub"; gui.ResetOnSpawn=false; gui.DisplayOrder=10
    gui.IgnoreGuiInset=true; gui.ZIndexBehavior=Enum.ZIndexBehavior.Sibling
    gui.Parent=LP:WaitForChild("PlayerGui")
    local uiScaleObj=Instance.new("UIScale",gui); uiScaleObj.Scale=1.0

    local function makeDraggable(frame,handle)
        local src=handle or frame
        local dragging,dragInput,dragStart,startPos=false,nil,nil,nil
        src.InputBegan:Connect(function(inp)
            if State.uiLocked then return end
            if inp.UserInputType==Enum.UserInputType.MouseButton1 or inp.UserInputType==Enum.UserInputType.Touch then
                dragging=true; dragStart=inp.Position; startPos=frame.Position
                inp.Changed:Connect(function() if inp.UserInputState==Enum.UserInputState.End then dragging=false end end)
            end
        end)
        src.InputChanged:Connect(function(inp)
            if inp.UserInputType==Enum.UserInputType.MouseMovement or inp.UserInputType==Enum.UserInputType.Touch then dragInput=inp end
        end)
        UIS.InputChanged:Connect(function(inp)
            if inp==dragInput and dragging and not State.uiLocked then
                local dx=inp.Position.X-dragStart.X; local dy=inp.Position.Y-dragStart.Y
                frame.Position=UDim2.new(startPos.X.Scale,startPos.X.Offset+dx,startPos.Y.Scale,startPos.Y.Offset+dy)
            end
        end)
    end

    local function makeStackDraggable(frame, onTap)
        local dragStartPos, startPos = nil, nil
        local isDragging = false
        local movedEnough = false
        local wasPressed = false
        local pressTime = 0
        local movementAllowed = not State.stackButtonsLocked
        local saveDebounce = nil

        local lockChangedConn = RunService.Heartbeat:Connect(function()
            movementAllowed = not State.stackButtonsLocked
        end)

        frame.InputBegan:Connect(function(input)
            if input.UserInputType ~= Enum.UserInputType.MouseButton1 and input.UserInputType ~= Enum.UserInputType.Touch then return end
            wasPressed = true
            pressTime = tick()
            dragStartPos = input.Position
            startPos = frame.Position
            isDragging = true
            movedEnough = false
        end)

        frame.InputChanged:Connect(function(input)
            if not isDragging or not movementAllowed then return end
            if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                local delta = input.Position - dragStartPos
                if delta.Magnitude > 8 then movedEnough = true end
                if movedEnough then
                    frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
                end
            end
        end)

        frame.InputEnded:Connect(function(input)
            local wasPressedLocal = wasPressed
            wasPressed = false
            if not isDragging then return end
            isDragging = false

            if movedEnough then
                if saveDebounce then task.cancel(saveDebounce) end
                saveDebounce = task.delay(0.2, function()
                    pcall(requestSave)
                    saveDebounce = nil
                end)
            end

            if wasPressedLocal and not movedEnough and (tick() - pressTime) < 0.3 then
                if onTap then onTap() end
            end
        end)

        frame.AncestryChanged:Connect(function()
            if not frame.Parent then lockChangedConn:Disconnect() end
        end)
    end

    local WIN_W = 400
    local WIN_H = 500
    local TITLE_H = 54
    local TAB_H = 36
    local mainOuter = Instance.new("Frame", gui)
    mainOuter.Name = "MainOuter"
    mainOuter.Size = UDim2.new(0, WIN_W, 0, WIN_H)
    mainOuter.Position = UDim2.new(0.5, -WIN_W/2, 0.5, -WIN_H/2)
    mainOuter.BackgroundTransparency = 1; mainOuter.BorderSizePixel = 0; mainOuter.ClipsDescendants = true
    mkCorner(mainOuter, 16); makeDraggable(mainOuter)

    -- ── BACKGROUND: custom image ──
    local BG_PHOTOS = {}  -- kept for reference compatibility
    local bgImg = Instance.new("ImageLabel", mainOuter)
    bgImg.Name = "BgFill"; bgImg.Size = UDim2.new(1,0,1,0)
    bgImg.BackgroundTransparency = 1
    bgImg.Image = "rbxassetid://81596065711733"
    bgImg.ScaleType = Enum.ScaleType.Crop
    bgImg.BorderSizePixel = 0; bgImg.ZIndex = 0
    mkCorner(bgImg, 16)

    -- No animated border — Asta Hub has no stroke border
    local mainStroke = Instance.new("UIStroke", mainOuter)
    mainStroke.Thickness = 0; mainStroke.Transparency = 1
    mainStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    local mainStrokeGrad = Instance.new("UIGradient", mainStroke)
    mainStrokeGrad.Color = ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(0,0,0)),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,0))})

    -- ── TITLE BAR: transparent, just text + minus button ─────────────
    local titleBar = Instance.new("Frame", mainOuter)
    titleBar.Size = UDim2.new(1,0,0,TITLE_H)
    titleBar.BackgroundTransparency = 1; titleBar.BorderSizePixel = 0; titleBar.ZIndex = 5

    -- "ASTA HUB" bold purple text, left aligned, vertically centered
    local titleLbl = Instance.new("TextLabel", titleBar)
    titleLbl.Size = UDim2.new(0,220,0,30); titleLbl.Position = UDim2.new(0,18,0.5,-15)
    titleLbl.BackgroundTransparency = 1; titleLbl.Text = "ASTA HUB"
    titleLbl.TextColor3 = Color3.fromRGB(180,100,255)
    titleLbl.Font = Enum.Font.GothamBlack; titleLbl.TextSize = 22
    titleLbl.TextXAlignment = Enum.TextXAlignment.Left; titleLbl.ZIndex = 6

    -- Minus close button — small dark pill top right, like photo
    local closeBtn = Instance.new("TextButton", titleBar)
    closeBtn.Size = UDim2.new(0,32,0,28); closeBtn.Position = UDim2.new(1,-42,0.5,-14)
    closeBtn.BackgroundColor3 = Color3.fromRGB(15,10,25); closeBtn.BorderSizePixel = 0
    closeBtn.Text = "−"; closeBtn.TextColor3 = Color3.fromRGB(200,190,220)
    closeBtn.Font = Enum.Font.GothamBlack; closeBtn.TextSize = 20
    closeBtn.ZIndex = 7; mkCorner(closeBtn,8)
    mkStroke(closeBtn, Color3.fromRGB(70,55,95), 1)
    closeBtn.MouseEnter:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(50,35,75)}):Play() end)
    closeBtn.MouseLeave:Connect(function() TweenService:Create(closeBtn,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(15,10,25)}):Play() end)
    closeBtn.MouseButton1Click:Connect(function()
        State.guiVisible = false; mainOuter.Visible = false
        if _G.AstaHubQAHide then pcall(_G.AstaHubQAHide, true) end
        requestSave()
    end)

    -- Thin divider line under title bar, faint purple
    local titleDiv = Instance.new("Frame", mainOuter)
    titleDiv.Size = UDim2.new(1,-24,0,1); titleDiv.Position = UDim2.new(0,12,0,TITLE_H)
    titleDiv.BackgroundColor3 = Color3.fromRGB(45,20,70); titleDiv.BorderSizePixel = 0; titleDiv.ZIndex = 5
    titleDiv.BackgroundTransparency = 0.5

    local CONTENT_Y = TITLE_H + 2

    -- ── SINGLE SCROLLING MENU (no tabs) ──────────────────────────
    local contentBg = Instance.new("Frame", mainOuter)
    contentBg.Size = UDim2.new(1,0,1,-CONTENT_Y)
    contentBg.Position = UDim2.new(0,0,0,CONTENT_Y)
    contentBg.BackgroundTransparency = 1
    contentBg.BorderSizePixel = 0; contentBg.ClipsDescendants = false; contentBg.ZIndex = 2

    -- One single scroll for all content
    local singleScroll = Instance.new("ScrollingFrame", contentBg)
    singleScroll.Name = "SingleScroll"
    singleScroll.Size = UDim2.new(1, 0, 1, 0)
    singleScroll.BackgroundTransparency = 1
    singleScroll.BorderSizePixel = 0
    singleScroll.ScrollBarThickness = 3
    singleScroll.ScrollBarImageColor3 = Color3.fromRGB(150,80,220)
    singleScroll.ScrollBarImageTransparency = 0.3
    singleScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
    singleScroll.CanvasSize = UDim2.new(0,0,0,0)
    singleScroll.ScrollingDirection = Enum.ScrollingDirection.Y
    singleScroll.ZIndex = 3
    local singleLL = Instance.new("UIListLayout", singleScroll)
    singleLL.SortOrder = Enum.SortOrder.LayoutOrder; singleLL.Padding = UDim.new(0,4)
    singleLL.HorizontalAlignment = Enum.HorizontalAlignment.Center
    local singlePad = Instance.new("UIPadding", singleScroll)
    singlePad.PaddingLeft = UDim.new(0,8); singlePad.PaddingRight = UDim.new(0,8)
    singlePad.PaddingTop = UDim.new(0,6); singlePad.PaddingBottom = UDim.new(0,12)

    -- Stub tables so buildPage/setActiveTab code doesn't error
    local TABS = {"Speed", "Fight", "Snatch", "Drift", "Looks", "Config"}
    local tabPages = {}
    local tabScrolls = {}
    local tabBtns = {}
    local currentPage = nil
    local activeTab = 1
    local lo = 0
    local function LO() lo = lo+1; return lo end
    -- All tabs share the same scroll
    for i = 1, #TABS do tabScrolls[i] = singleScroll end
    local function setActiveTab(idx) activeTab = idx end
    local mainScroll = singleScroll

    local function makeGap(px) local f=Instance.new("Frame",currentPage); f.Size=UDim2.new(1,0,0,px or 6); f.BackgroundTransparency=1; f.BorderSizePixel=0; f.LayoutOrder=LO() end
    local function makeSectionHeader(label)
        local wrap = Instance.new("Frame", currentPage)
        wrap.Size = UDim2.new(1,0,0,34); wrap.BackgroundTransparency=1; wrap.BorderSizePixel=0; wrap.LayoutOrder=LO()
        local lbl = Instance.new("TextLabel", wrap); lbl.Size = UDim2.new(1,-20,1,0); lbl.Position = UDim2.new(0,10,0,0)
        lbl.BackgroundTransparency=1; lbl.Text = label and label:upper() or ""
        lbl.TextColor3 = Color3.fromRGB(180,100,255); lbl.Font = Enum.Font.GothamBlack; lbl.TextSize=13
        lbl.TextXAlignment = Enum.TextXAlignment.Left
    end

    local function makeInputRow(label, default, onChange)
        local row = Instance.new("Frame", currentPage)
        row.Size = UDim2.new(1,-16,0,44); row.BackgroundColor3 = Color3.fromRGB(13,10,20)
        row.BackgroundTransparency = 0.15; row.BorderSizePixel=0; row.LayoutOrder=LO(); mkCorner(row,10)
        local rowStroke = mkStroke(row, Color3.fromRGB(55,40,80),1)
        rowStroke.Transparency = 0.4
        row.MouseEnter:Connect(function()
            TweenService:Create(row,TweenInfo.new(0.08),{BackgroundColor3=Color3.fromRGB(18,12,30),BackgroundTransparency=0.05}):Play()
        end)
        row.MouseLeave:Connect(function()
            TweenService:Create(row,TweenInfo.new(0.08),{BackgroundColor3=Color3.fromRGB(13,10,20),BackgroundTransparency=0.15}):Play()
        end)
        local lbl = Instance.new("TextLabel", row)
        lbl.Size = UDim2.new(0.6,0,1,0); lbl.Position = UDim2.new(0,14,0,0)
        lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Color3.fromRGB(230,225,240)
        lbl.Font = Enum.Font.GothamBold; lbl.TextSize=13; lbl.TextXAlignment=Enum.TextXAlignment.Left
        local tb = Instance.new("TextBox", row)
        tb.Size = UDim2.new(0,60,0,28); tb.Position = UDim2.new(1,-68,0.5,-14)
        tb.BackgroundColor3 = Color3.fromRGB(18,14,28); tb.BorderSizePixel=0
        tb.Text = tostring(default); tb.TextColor3 = Color3.fromRGB(255,255,255)
        tb.Font = Enum.Font.GothamBold; tb.TextSize=13; tb.ClearTextOnFocus=false; tb.ZIndex=5
        tb.TextXAlignment = Enum.TextXAlignment.Center
        mkCorner(tb,7)
        local bs = mkStroke(tb, Color3.fromRGB(45,20,70),1)
        bs.Transparency = 0.3
        tb.Focused:Connect(function() TweenService:Create(bs,TweenInfo.new(0.12),{Color=Color3.fromRGB(180,100,255),Transparency=0}):Play() end)
        tb.FocusLost:Connect(function()
            TweenService:Create(bs,TweenInfo.new(0.12),{Color=Color3.fromRGB(45,20,70),Transparency=0.3}):Play()
            if onChange then
                local n = tonumber(tb.Text)
                if n then onChange(n); requestSave()
                else tb.Text = tostring(default) end
            end
        end)
        return tb,row
    end

    local function makeToggleRow(label, defaultOn, onToggle)
        local row = Instance.new("Frame", currentPage)
        row.Size = UDim2.new(1,-16,0,44); row.BackgroundColor3 = Color3.fromRGB(13,10,20)
        row.BackgroundTransparency = 0.15; row.BorderSizePixel=0; row.LayoutOrder=LO(); mkCorner(row,10)
        local rowStroke = mkStroke(row, Color3.fromRGB(55,40,80),1); rowStroke.Transparency = 0.4
        row.MouseEnter:Connect(function()
            TweenService:Create(row,TweenInfo.new(0.08),{BackgroundColor3=Color3.fromRGB(18,12,30),BackgroundTransparency=0.05}):Play()
        end)
        row.MouseLeave:Connect(function()
            TweenService:Create(row,TweenInfo.new(0.08),{BackgroundColor3=Color3.fromRGB(13,10,20),BackgroundTransparency=0.15}):Play()
        end)
        local lbl = Instance.new("TextLabel", row)
        lbl.Size = UDim2.new(0.6,0,1,0); lbl.Position = UDim2.new(0,14,0,0)
        lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Color3.fromRGB(230,225,240)
        lbl.Font = Enum.Font.GothamBold; lbl.TextSize=13; lbl.TextXAlignment=Enum.TextXAlignment.Left
        local pillBg = Instance.new("Frame", row)
        pillBg.Size = UDim2.new(0,48,0,26); pillBg.Position = UDim2.new(1,-58,0.5,-13)
        pillBg.BackgroundColor3 = defaultOn and Color3.fromRGB(60,0,120) or Color3.fromRGB(30,30,30)
        pillBg.BorderSizePixel=0; pillBg.ZIndex=3; mkCorner(pillBg,20)
        local pillGlow = Instance.new("UIStroke", pillBg)
        pillGlow.Color = defaultOn and Color3.fromRGB(180,100,255) or Color3.fromRGB(70,50,100)
        pillGlow.Thickness = 1.4
        pillGlow.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        local dot = Instance.new("Frame", pillBg)
        dot.Size = UDim2.new(0,20,0,20)
        dot.Position = defaultOn and UDim2.new(1,-23,0.5,-10) or UDim2.new(0,3,0.5,-10)
        dot.BackgroundColor3 = Color3.fromRGB(255,255,255)
        dot.BorderSizePixel=0; dot.ZIndex=4; mkCorner(dot,13)
        local isOn = defaultOn or false
        local function setV(on)
            isOn = on
            TweenService:Create(pillBg, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                BackgroundColor3 = on and Color3.fromRGB(60,0,120) or Color3.fromRGB(30,30,30)
            }):Play()
            TweenService:Create(pillGlow, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                Color = on and Color3.fromRGB(180,100,255) or Color3.fromRGB(70,50,100)
            }):Play()
            TweenService:Create(dot, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                Position = on and UDim2.new(1,-23,0.5,-10) or UDim2.new(0,3,0.5,-10)
            }):Play()
        end
        local function toggle()
            isOn = not isOn; setV(isOn)
            if onToggle then pcall(onToggle, isOn) end
            requestSave()
        end
        local clk = Instance.new("TextButton", row); clk.Size = UDim2.new(1,-68,1,0); clk.BackgroundTransparency=1; clk.Text=""; clk.ZIndex=5; clk.BorderSizePixel=0; clk.MouseButton1Click:Connect(toggle)
        local pClk = Instance.new("TextButton", pillBg); pClk.Size = UDim2.new(1,0,1,0); pClk.BackgroundTransparency=1; pClk.Text=""; pClk.ZIndex=9; pClk.BorderSizePixel=0; pClk.MouseButton1Click:Connect(toggle)
        return setV
    end

    -- Aimbot mode selector (segmented control)
    local function makeAimbotModeSelector()
        local modeRow = Instance.new("Frame", currentPage)
        modeRow.Size = UDim2.new(1, -16, 0, 36)
        modeRow.BackgroundColor3 = Color3.fromRGB(15,15,15)
        modeRow.BackgroundTransparency = 0.78
        modeRow.BorderSizePixel = 0
        modeRow.LayoutOrder = LO()
        mkCorner(modeRow, 8)
        mkStroke(modeRow, Color3.fromRGB(55,55,55), 1).Transparency = 0.5

        local label = Instance.new("TextLabel", modeRow)
        label.Size = UDim2.new(0.4, 0, 1, 0)
        label.Position = UDim2.new(0, 12, 0, 0)
        label.BackgroundTransparency = 1
        label.Text = "Aimbot Mode"
        label.TextColor3 = Color3.fromRGB(200,200,200)
        label.Font = Enum.Font.GothamBold
        label.TextSize = 12
        label.TextXAlignment = Enum.TextXAlignment.Left

        local segPill = Instance.new("Frame", modeRow)
        segPill.Size = UDim2.new(0, 140, 0, 28)
        segPill.Position = UDim2.new(1, -148, 0.5, -14)
        segPill.BackgroundColor3 = Color3.fromRGB(18,18,18)
        segPill.BorderSizePixel = 0
        segPill.ZIndex = 10
        mkCorner(segPill, 8)
        mkStroke(segPill, Color3.fromRGB(55,55,55), 1)

        local segHighlight = Instance.new("Frame", segPill)
        segHighlight.Size = UDim2.new(0.5, -4, 1, -6)
        segHighlight.Position = (State.aimbotMode == "normal") and UDim2.new(0, 3, 0, 3) or UDim2.new(0.5, 1, 0, 3)
        segHighlight.BackgroundColor3 = Color3.fromRGB(55,55,55)
        segHighlight.BorderSizePixel = 0
        segHighlight.ZIndex = 11
        mkCorner(segHighlight, 6)

        local normalBtn = Instance.new("TextButton", segPill)
        normalBtn.Size = UDim2.new(0.5, 0, 1, 0)
        normalBtn.Position = UDim2.new(0, 0, 0, 0)
        normalBtn.BackgroundTransparency = 1
        normalBtn.BorderSizePixel = 0
        normalBtn.Text = "Normal"
        normalBtn.TextColor3 = (State.aimbotMode == "normal") and Color3.fromRGB(255,255,255) or Color3.fromRGB(120,120,120)
        normalBtn.Font = Enum.Font.GothamBold
        normalBtn.TextSize = 11
        normalBtn.ZIndex = 12

        local bypassBtn = Instance.new("TextButton", segPill)
        bypassBtn.Size = UDim2.new(0.5, 0, 1, 0)
        bypassBtn.Position = UDim2.new(0.5, 0, 0, 0)
        bypassBtn.BackgroundTransparency = 1
        bypassBtn.BorderSizePixel = 0
        bypassBtn.Text = "Bypass"
        bypassBtn.TextColor3 = (State.aimbotMode == "bypass") and Color3.fromRGB(255,255,255) or Color3.fromRGB(120,120,120)
        bypassBtn.Font = Enum.Font.GothamBold
        bypassBtn.TextSize = 11
        bypassBtn.ZIndex = 12

        local function setMode(mode)
            State.aimbotMode = mode
            if mode == "normal" then
                TweenService:Create(segHighlight, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                    Position = UDim2.new(0, 3, 0, 3)
                }):Play()
                normalBtn.TextColor3 = Color3.fromRGB(255,255,255)
                bypassBtn.TextColor3 = Color3.fromRGB(120,120,120)
            else
                TweenService:Create(segHighlight, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                    Position = UDim2.new(0.5, 1, 0, 3)
                }):Play()
                bypassBtn.TextColor3 = Color3.fromRGB(255,255,255)
                normalBtn.TextColor3 = Color3.fromRGB(120,120,120)
            end
            requestSave()
            -- If aimbot is currently on, restart it with new mode
            if State.batAimbotToggled then
                pcall(function()
                    stopBatAimbot()
                    task.wait(0.1)
                    startBatAimbot()
                end)
            end
        end

        normalBtn.MouseButton1Click:Connect(function() setMode("normal") end)
        bypassBtn.MouseButton1Click:Connect(function() setMode("bypass") end)

        return modeRow
    end

    local function getGpDisplayName(kc)
        if not kc or kc == Enum.KeyCode.Unknown then return "None" end
        local gpNames = {
            ButtonA="A", ButtonB="B", ButtonX="X", ButtonY="Y",
            ButtonL1="LB", ButtonL2="LT", ButtonL3="LS",
            ButtonR1="RB", ButtonR2="RT", ButtonR3="RS",
            ButtonSelect="SEL", ButtonStart="STA",
            DPadUp="D↑", DPadDown="D↓", DPadLeft="D←", DPadRight="D→"
        }
        return gpNames[kc.Name] or kc.Name:sub(1,5)
    end

    local function refreshAllKeybindButtons()
        for keyName, btn in pairs(keybindBtnRefs) do
            if btn and Keys[keyName] then
                btn.Text = getGpDisplayName(Keys[keyName])
            end
        end
    end

    -- ============================================================
    -- KEYBIND ROW (FIXED: accepts both keyboard and gamepad)
    -- ============================================================
    local function makeKeybindRow(label, currentKey, onChanged, keyName)
        local row = Instance.new("Frame", currentPage)
        row.Size = UDim2.new(1,-16,0,44); row.BackgroundColor3 = Color3.fromRGB(13,10,20)
        row.BackgroundTransparency = 0.15; row.BorderSizePixel=0; row.LayoutOrder=LO(); mkCorner(row,10)
        local rowStroke = mkStroke(row, Color3.fromRGB(55,40,80),1); rowStroke.Transparency = 0.4
        row.MouseEnter:Connect(function()
            TweenService:Create(row,TweenInfo.new(0.08),{BackgroundColor3=Color3.fromRGB(18,12,30),BackgroundTransparency=0.05}):Play()
        end)
        row.MouseLeave:Connect(function()
            TweenService:Create(row,TweenInfo.new(0.08),{BackgroundColor3=Color3.fromRGB(13,10,20),BackgroundTransparency=0.15}):Play()
        end)
        local lbl = Instance.new("TextLabel", row)
        lbl.Size = UDim2.new(0.55,0,1,0); lbl.Position = UDim2.new(0,14,0,0)
        lbl.BackgroundTransparency=1; lbl.Text=label; lbl.TextColor3=Color3.fromRGB(230,225,240)
        lbl.Font=Enum.Font.GothamBold; lbl.TextSize=13; lbl.TextXAlignment=Enum.TextXAlignment.Left
        local kbtn = Instance.new("TextButton", row)
        kbtn.Size = UDim2.new(0,72,0,30); kbtn.Position = UDim2.new(1,-80,0.5,-15)
        kbtn.BackgroundColor3 = Color3.fromRGB(18,14,28); kbtn.BorderSizePixel=0
        kbtn.Text = getGpDisplayName(currentKey)
        kbtn.TextColor3 = Color3.fromRGB(200,150,255); kbtn.Font = Enum.Font.GothamBold
        kbtn.TextSize=12; kbtn.ZIndex=5; mkCorner(kbtn,8)
        local ks = mkStroke(kbtn, Color3.fromRGB(100,60,150),1); ks.Transparency=0.3
        local listening = false; local lconn
        local function stopL(key)
            listening = false
            if lconn then lconn:Disconnect(); lconn=nil end
            TweenService:Create(ks,TweenInfo.new(0.12),{Color=Color3.fromRGB(100,60,150),Transparency=0.3}):Play()
            TweenService:Create(kbtn,TweenInfo.new(0.12),{BackgroundColor3=Color3.fromRGB(18,14,28)}):Play()
            kbtn.TextColor3 = Color3.fromRGB(200,150,255)
            if key then
                kbtn.Text = getGpDisplayName(key)
                if onChanged then onChanged(key) end
                pcall(requestSave)
            else
                kbtn.Text = getGpDisplayName(Keys[keyName] or Enum.KeyCode.Unknown)
            end
        end
        kbtn.MouseButton1Click:Connect(function()
            if listening then stopL(nil); return end
            listening = true; kbtn.Text = "..."; kbtn.TextColor3 = Color3.fromRGB(255,220,255)
            TweenService:Create(kbtn,TweenInfo.new(0.12),{BackgroundColor3=Color3.fromRGB(30,10,50)}):Play()
            TweenService:Create(ks,TweenInfo.new(0.12),{Color=Color3.fromRGB(200,100,255),Transparency=0}):Play()
            -- Listen for BOTH keyboard AND gamepad
            lconn = UIS.InputBegan:Connect(function(inp)
                if not listening then return end
                local isKb = inp.UserInputType == Enum.UserInputType.Keyboard
                local isGp = inp.UserInputType==Enum.UserInputType.Gamepad1
                    or inp.UserInputType==Enum.UserInputType.Gamepad2
                    or inp.UserInputType==Enum.UserInputType.Gamepad3
                    or inp.UserInputType==Enum.UserInputType.Gamepad4
                if not isKb and not isGp then return end
                local kc = inp.KeyCode
                if kc == Enum.KeyCode.Unknown then return end
                stopL(kc)
            end)
        end)
        if keyName then keybindBtnRefs[keyName] = kbtn end
        return kbtn
    end

    -- ============================================================
    -- PERFORMANCE
    -- ============================================================

    -- FPS BOOST
    local fpsBoostDescConn = nil
    local function applyFPSBoost()
        pcall(function() setfpscap(999999999) end)
        local function pO(v)
            pcall(function()
                if v:IsA("MeshPart") then
                    v.CastShadow = false
                    v.RenderFidelity = Enum.RenderFidelity.Performance
                elseif v:IsA("BasePart") then
                    v.CastShadow = false
                    v.Material = Enum.Material.Plastic
                    v.Reflectance = 0
                elseif v:IsA("Decal") or v:IsA("Texture") then
                    v.Transparency = 1
                elseif v:IsA("Fire") or v:IsA("Smoke") or v:IsA("Sparkles")
                    or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then
                    v.Enabled = false
                end
            end)
        end
        for _, v in pairs(workspace:GetDescendants()) do pO(v) end
        pcall(function()
            local L = game:GetService("Lighting")
            for _, v in pairs(L:GetDescendants()) do
                pcall(function()
                    if v:IsA("BloomEffect") or v:IsA("BlurEffect")
                        or v:IsA("SunRaysEffect") or v:IsA("ColorCorrectionEffect") then
                        v.Enabled = false
                    end
                end)
            end
            L.GlobalShadows = false
            L.FogEnd = 9e9
            L.Brightness = 0
        end)
        if fpsBoostDescConn then fpsBoostDescConn:Disconnect() end
        fpsBoostDescConn = workspace.DescendantAdded:Connect(function(v)
            if State.fpsBoostEnabled then task.spawn(pO, v) end
        end)
    end
    local function disableFPSBoost()
        if fpsBoostDescConn then fpsBoostDescConn:Disconnect(); fpsBoostDescConn = nil end
    end

    local antiLagDescConn = nil
    local antiLagActive = false
    local antiLagDefBrightness, antiLagDefFog, antiLagDefDiffuse, antiLagDefSpecular

    local function _applyAntiLagObj(obj)
        pcall(function()
            if obj:IsA("BasePart") then
                obj.Material = Enum.Material.Plastic; obj.Reflectance = 0; obj.CastShadow = false
            elseif obj:IsA("Decal") or obj:IsA("Texture") then
                obj.Transparency = 1
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam")
            or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
                obj.Enabled = false
            elseif obj:IsA("AnimationController") or obj:IsA("Animator") then
                for _,t in ipairs(obj:GetPlayingAnimationTracks()) do pcall(function() t:Stop(0) end) end
            end
        end)
    end

    local function enableAntiLag()
        antiLagActive = true
        antiLagDefBrightness = antiLagDefBrightness or Lighting.Brightness
        antiLagDefFog        = antiLagDefFog        or Lighting.FogEnd
        antiLagDefDiffuse    = antiLagDefDiffuse    or Lighting.EnvironmentDiffuseScale
        antiLagDefSpecular   = antiLagDefSpecular   or Lighting.EnvironmentSpecularScale
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 1e10
        Lighting.EnvironmentDiffuseScale = 0
        Lighting.EnvironmentSpecularScale = 0
        for _,e in pairs(Lighting:GetChildren()) do
            pcall(function()
                if e:IsA("BlurEffect") or e:IsA("SunRaysEffect") or e:IsA("ColorCorrectionEffect")
                or e:IsA("BloomEffect") or e:IsA("DepthOfFieldEffect") then e.Enabled = false end
            end)
        end
        for _,obj in ipairs(workspace:GetDescendants()) do _applyAntiLagObj(obj) end
        if antiLagDescConn then antiLagDescConn:Disconnect() end
        antiLagDescConn = workspace.DescendantAdded:Connect(function(obj)
            if antiLagActive then _applyAntiLagObj(obj) end
        end)
    end

    local function disableAntiLag()
        antiLagActive = false
        if antiLagDescConn then antiLagDescConn:Disconnect(); antiLagDescConn = nil end
        pcall(function()
            Lighting.GlobalShadows = true
            if antiLagDefBrightness then Lighting.Brightness = antiLagDefBrightness end
            if antiLagDefFog        then Lighting.FogEnd = antiLagDefFog end
            if antiLagDefDiffuse    then Lighting.EnvironmentDiffuseScale = antiLagDefDiffuse end
            if antiLagDefSpecular   then Lighting.EnvironmentSpecularScale = antiLagDefSpecular end
            for _,e in pairs(Lighting:GetChildren()) do
                pcall(function()
                    if e:IsA("BlurEffect") or e:IsA("SunRaysEffect") or e:IsA("ColorCorrectionEffect")
                    or e:IsA("BloomEffect") or e:IsA("DepthOfFieldEffect") then e.Enabled = true end
                end)
            end
        end)
    end

    local stretchRezEnabled=false
    local stretchRezConn,stretchFovConn=nil,nil
    local function applyStretchFOV(val) local cam=Workspace.CurrentCamera; if cam then pcall(function() cam.FieldOfView=val end) end end
    local function enableStretchRez()
        stretchRezEnabled=true; local cam=Workspace.CurrentCamera; if not cam then return end
        if stretchRezConn then stretchRezConn:Disconnect() end
        if stretchFovConn then stretchFovConn:Disconnect() end
        stretchFovConn = RunService.RenderStepped:Connect(function() if stretchRezEnabled then applyStretchFOV(State.stretchFOV) end end)
        stretchRezConn = RunService.RenderStepped:Connect(function()
            if not stretchRezEnabled then stretchRezConn:Disconnect(); stretchRezConn=nil; return end
            if cam then cam.CFrame = cam.CFrame * CFrame.new(0,0,0,1,0,0,0,0.7,0,0,0,1) end
        end)
    end
    local function disableStretchRez()
        stretchRezEnabled=false
        if stretchRezConn then stretchRezConn:Disconnect(); stretchRezConn=nil end
        if stretchFovConn then stretchFovConn:Disconnect(); stretchFovConn=nil end
        pcall(function() Workspace.CurrentCamera.FieldOfView = 70 end)
    end
    local function cleanParticlesAndLights()
        local removed=0
        for _,obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") or obj:IsA("Explosion") or obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
                pcall(function() obj:Destroy() end); removed=removed+1
            end
        end
        if _G._VezyFlashSave then _G._VezyFlashSave(true); task.delay(1.2,function() if _G._VezyFlashSave then _G._VezyFlashSave(false) end end) end
        print("[Asta Hub] Cleaned "..removed.." effects/lights")
    end
    local origLighting = {
        Ambient = Lighting.Ambient, Brightness = Lighting.Brightness, ClockTime = Lighting.ClockTime,
        FogColor = Lighting.FogColor, FogEnd = Lighting.FogEnd, GlobalShadows = Lighting.GlobalShadows,
        EnvironmentDiffuseScale = Lighting.EnvironmentDiffuseScale,
        EnvironmentSpecularScale = Lighting.EnvironmentSpecularScale,
    }
    local activeColorCorr = nil
    local function clearColorCorr() if activeColorCorr then pcall(function() activeColorCorr:Destroy() end); activeColorCorr=nil end end
    local function restoreLighting()
        clearColorCorr()
        pcall(function()
            Lighting.Ambient = origLighting.Ambient; Lighting.Brightness = origLighting.Brightness
            Lighting.ClockTime = origLighting.ClockTime; Lighting.FogColor = origLighting.FogColor
            Lighting.FogEnd = origLighting.FogEnd; Lighting.GlobalShadows = origLighting.GlobalShadows
            Lighting.EnvironmentDiffuseScale = origLighting.EnvironmentDiffuseScale
            Lighting.EnvironmentSpecularScale = origLighting.EnvironmentSpecularScale
        end)
    end

    -- ============================================================
    -- CANDY HUB SKY ENGINE
    -- ============================================================
    local CANDY_SKY_TAG = "ZainHubSkyTheme"
    local candyOriginalLighting = nil
    local CANDY_SKY_PRESETS = {
        ["Off"] = {kind = "off"},
        ["Night"] = {clock=22,brightness=2,ambient={110,100,130},outAmb={120,110,140},sky={stars=4000,moon=18,sun=0,moonTex=true},atm={dens=0.45,color={120,60,180},decay={60,20,100},glare=0.5,haze=1.2}},
        ["Aurora"] = {clock=14,brightness=3,ambient={150,120,150},outAmb={160,130,160},atm={dens=0.55,color={255,80,200},decay={255,20,150},glare=2.5,haze=3},clouds={cover=0.7,dens=0.7,color={255,240,250}}},
        ["Sunset"] = {clock=17.2,brightness=2.5,ambient={170,120,100},outAmb={180,130,110},sky={stars=0,sun=25,moon=0},atm={dens=0.5,color={255,130,60},decay={255,80,30},glare=2,haze=2.5},clouds={cover=0.55,dens=0.55,color={255,200,140}}},
        ["Galaxy"] = {clock=0,brightness=1.5,ambient={70,60,100},outAmb={80,70,110},sky={stars=10000,moon=30,sun=0},atm={dens=0.15,color={40,20,80},decay={20,10,50},glare=0.3,haze=0.5}},
        ["Cyber"] = {clock=21,brightness=2.2,ambient={90,130,170},outAmb={100,140,180},sky={stars=2000,moon=12},atm={dens=0.4,color={0,200,255},decay={150,0,255},glare=2,haze=2},clouds={cover=0.4,dens=0.6,color={100,200,255}}},
        ["Sakura"] = {clock=11,brightness=3.5,ambient={170,150,160},outAmb={180,160,170},sky={sun=8},atm={dens=0.3,color={255,200,220},decay={255,170,200},glare=1,haze=1.5},clouds={cover=0.6,dens=0.4,color={255,250,252}}},
        ["Pink Night"] = {clock=23,brightness=2.2,ambient={120,60,110},outAmb={140,70,120},sky={stars=5000,moon=22,sun=0,moonTex=true},atm={dens=0.5,color={255,80,180},decay={140,30,100},glare=0.7,haze=1.4},clouds={cover=0.3,dens=0.5,color={180,90,150}}},
        ["Blood Moon"] = {clock=22.5,brightness=1.6,ambient={130,40,40},outAmb={150,50,50},sky={stars=1500,moon=28,sun=0,moonTex=true},atm={dens=0.6,color={220,30,30},decay={120,10,10},glare=1.4,haze=2},clouds={cover=0.5,dens=0.7,color={120,30,30}}},
        ["Emerald Dawn"] = {clock=6.5,brightness=2.8,ambient={130,170,140},outAmb={140,180,150},sky={sun=18,moon=0,stars=0},atm={dens=0.4,color={80,200,140},decay={40,150,90},glare=1.8,haze=2.2},clouds={cover=0.5,dens=0.5,color={200,255,220}}},
        ["Volcanic"] = {clock=19,brightness=2,ambient={180,80,40},outAmb={200,90,50},sky={stars=200,sun=12,moon=0},atm={dens=0.75,color={255,60,0},decay={180,20,0},glare=3,haze=3.5},clouds={cover=0.8,dens=0.9,color={120,40,20}}},
        ["Arctic"] = {clock=9,brightness=3.2,ambient={200,220,235},outAmb={210,230,245},sky={sun=10,stars=0,moon=0},atm={dens=0.3,color={180,220,255},decay={140,200,240},glare=1.5,haze=1.8},clouds={cover=0.7,dens=0.6,color={250,253,255}}},
        ["Midnight Ocean"] = {clock=1.5,brightness=1.7,ambient={60,90,130},outAmb={70,100,140},sky={stars=6000,moon=24,sun=0,moonTex=true},atm={dens=0.5,color={20,60,140},decay={10,30,90},glare=0.6,haze=1.5}},
        ["Vaporwave"] = {clock=19.5,brightness=2.4,ambient={180,120,200},outAmb={190,130,210},sky={stars=1000,moon=14},atm={dens=0.45,color={255,100,220},decay={120,60,255},glare=2.2,haze=2.4},clouds={cover=0.5,dens=0.55,color={200,150,255}}},
        ["Toxic"] = {clock=13,brightness=2.5,ambient={140,180,80},outAmb={150,190,90},atm={dens=0.55,color={100,220,40},decay={60,150,20},glare=1.8,haze=2.6},clouds={cover=0.65,dens=0.7,color={180,255,120}}},
        ["Solar Eclipse"] = {clock=12,brightness=0.9,ambient={50,40,60},outAmb={60,50,70},sky={stars=3500,sun=22,moon=0},atm={dens=0.5,color={255,140,40},decay={30,20,40},glare=2.8,haze=1.8}},
        ["Hellscape"] = {clock=18,brightness=1.8,ambient={200,60,30},outAmb={220,70,40},sky={stars=100,sun=30,moon=0},atm={dens=0.85,color={255,30,0},decay={120,0,0},glare=3.5,haze=4},clouds={cover=0.95,dens=0.95,color={80,20,10}}},
        ["Heaven"] = {clock=12,brightness=4,ambient={240,235,210},outAmb={250,245,220},sky={sun=16,moon=0,stars=0},atm={dens=0.25,color={255,250,220},decay={255,240,200},glare=3,haze=1.5},clouds={cover=0.85,dens=0.5,color={255,255,255}}},
        ["Storm"] = {clock=15,brightness=1.4,ambient={90,90,110},outAmb={100,100,120},sky={stars=0,sun=6,moon=0},atm={dens=0.65,color={80,90,120},decay={40,50,80},glare=0.5,haze=3},clouds={cover=0.95,dens=0.95,color={60,65,80}}},
        ["Sunrise"] = {clock=6.2,brightness=2.8,ambient={220,180,130},outAmb={230,190,140},sky={sun=22,stars=0,moon=0},atm={dens=0.45,color={255,180,100},decay={255,140,80},glare=2.4,haze=2.2},clouds={cover=0.4,dens=0.4,color={255,220,180}}},
        ["Deep Space"] = {clock=0,brightness=1,ambient={30,25,50},outAmb={40,35,60},sky={stars=15000,moon=0,sun=0},atm={dens=0.08,color={15,5,40},decay={5,0,20},glare=0.2,haze=0.3}},
        ["Lavender Dream"] = {clock=18.5,brightness=2.6,ambient={180,160,220},outAmb={190,170,230},sky={stars=800,moon=16,sun=0},atm={dens=0.4,color={200,160,255},decay={160,120,220},glare=1.4,haze=1.8},clouds={cover=0.55,dens=0.5,color={220,200,255}}},
        ["Inferno"] = {clock=17.5,brightness=2.2,ambient={220,100,40},outAmb={235,110,50},sky={sun=26,moon=0,stars=0},atm={dens=0.6,color={255,90,20},decay={200,40,0},glare=3,haze=3.2},clouds={cover=0.7,dens=0.7,color={200,80,40}}},
        ["Mint Sky"] = {clock=10,brightness=3.2,ambient={180,230,210},outAmb={190,240,220},sky={sun=10},atm={dens=0.32,color={150,255,210},decay={100,220,180},glare=1.6,haze=1.6},clouds={cover=0.55,dens=0.45,color={240,255,250}}},
    }
    local CANDY_SKY_ORDER = {"Off","Night","Aurora","Sunset","Galaxy","Cyber","Sakura","Pink Night","Blood Moon","Emerald Dawn","Volcanic","Arctic","Midnight Ocean","Vaporwave","Toxic","Solar Eclipse","Hellscape","Heaven","Storm","Sunrise","Deep Space","Lavender Dream","Inferno","Mint Sky"}
    local function candySaveOriginalLighting()
        if candyOriginalLighting then return end
        candyOriginalLighting={ClockTime=Lighting.ClockTime,OutdoorAmbient=Lighting.OutdoorAmbient,Ambient=Lighting.Ambient,Brightness=Lighting.Brightness,FogStart=Lighting.FogStart,FogEnd=Lighting.FogEnd,FogColor=Lighting.FogColor,ColorShift_Top=Lighting.ColorShift_Top,ColorShift_Bottom=Lighting.ColorShift_Bottom,GlobalShadows=Lighting.GlobalShadows,LightingChildren={},TerrainChildren={}}
        for _,child in ipairs(Lighting:GetChildren()) do if child:IsA("Sky") or child:IsA("Atmosphere") then table.insert(candyOriginalLighting.LightingChildren,child:Clone()) end end
        local terrain=workspace:FindFirstChildOfClass("Terrain")
        if terrain then for _,child in ipairs(terrain:GetChildren()) do if child:IsA("Clouds") then table.insert(candyOriginalLighting.TerrainChildren,child:Clone()) end end end
    end
    local function candyClearSky(removeAll)
        for _,child in ipairs(Lighting:GetChildren()) do if child:GetAttribute(CANDY_SKY_TAG) or (removeAll and (child:IsA("Sky") or child:IsA("Atmosphere"))) then pcall(function() child:Destroy() end) end end
        local terrain=workspace:FindFirstChildOfClass("Terrain")
        if terrain then for _,child in ipairs(terrain:GetChildren()) do if child:GetAttribute(CANDY_SKY_TAG) or (removeAll and child:IsA("Clouds")) then pcall(function() child:Destroy() end) end end end
    end
    local function candyInst(className,parent,props)
        local inst=Instance.new(className); inst:SetAttribute(CANDY_SKY_TAG,true)
        for k,v in pairs(props or {}) do pcall(function() inst[k]=v end) end
        inst.Parent=parent; return inst
    end
    local function candyRGB(t) return Color3.fromRGB(t[1],t[2],t[3]) end
    local function applySky(mode)
        candySaveOriginalLighting()
        candyClearSky(true)
        local terrain=workspace:FindFirstChildOfClass("Terrain")
        local preset=CANDY_SKY_PRESETS[mode]
        if not preset or preset.kind=="off" or mode==nil then
            if candyOriginalLighting then
                for k,v in pairs(candyOriginalLighting) do if k~="LightingChildren" and k~="TerrainChildren" then pcall(function() Lighting[k]=v end) end end
                for _,child in ipairs(candyOriginalLighting.LightingChildren or {}) do child:Clone().Parent=Lighting end
                local offTerrain=workspace:FindFirstChildOfClass("Terrain")
                if offTerrain then for _,child in ipairs(candyOriginalLighting.TerrainChildren or {}) do child:Clone().Parent=offTerrain end end
            end
            return
        end
        Lighting.FogStart=0; Lighting.FogEnd=100000; Lighting.FogColor=Color3.fromRGB(200,200,200)
        Lighting.ColorShift_Top=Color3.fromRGB(0,0,0); Lighting.ColorShift_Bottom=Color3.fromRGB(0,0,0)
        Lighting.GlobalShadows=true; Lighting.ClockTime=preset.clock or 14; Lighting.Brightness=preset.brightness or 2
        if preset.outAmb then Lighting.OutdoorAmbient=candyRGB(preset.outAmb) end
        if preset.ambient then Lighting.Ambient=candyRGB(preset.ambient) end
        if preset.sky then
            local sp={}
            if preset.sky.stars then sp.StarCount=preset.sky.stars end
            if preset.sky.moon then sp.MoonAngularSize=preset.sky.moon end
            if preset.sky.sun then sp.SunAngularSize=preset.sky.sun end
            if preset.sky.moonTex then sp.MoonTextureId="rbxasset://sky/moon.jpg" end
            candyInst("Sky",Lighting,sp)
        end
        if preset.atm then
            candyInst("Atmosphere",Lighting,{Density=preset.atm.dens or 0.3,Color=candyRGB(preset.atm.color),Decay=candyRGB(preset.atm.decay),Glare=preset.atm.glare or 1,Haze=preset.atm.haze or 1})
        end
        if preset.clouds and terrain then
            candyInst("Clouds",terrain,{Cover=preset.clouds.cover or 0.5,Density=preset.clouds.dens or 0.5,Color=candyRGB(preset.clouds.color)})
        end
    end

    -- ============================================================
    -- BUILD PAGES
    -- ============================================================
    local function buildPage(tabName, buildFn)
        -- All pages go into the single scroll; sections separated by headers
        local sf = singleScroll
        tabPages[tabName] = sf
        currentPage = sf; buildFn(); currentPage = nil
        return sf
    end

    -- Speed Page
    do
        local page = buildPage("Speed", function()
            makeGap(4); makeSectionHeader("Speed Configuration"); makeGap(4)
            normalBox = makeInputRow("Normal Speed", State.normalSpeed, function(n) if n>0 and n<=500 then State.normalSpeed=n end end)
            carryBox = makeInputRow("Carry Speed", State.carrySpeed, function(n) if n>0 and n<=500 then State.carrySpeed=n end end)
            makeKeybindRow("Speed Key", Keys.speed, function(k) Keys.speed=k end, "speed")
            makeGap(4); makeSectionHeader("Lagger Configuration"); makeGap(4)
            laggerBox = makeInputRow("Lagger Normal Speed", State.laggerSpeed, function(n) if n>0 and n<=500 then State.laggerSpeed=n end end)
            laggerCarryBox = makeInputRow("Lagger Carry Speed", State.laggerCarrySpeed, function(n) if n>0 and n<=500 then State.laggerCarrySpeed=n end end)
            makeKeybindRow("Lagger Carry Key", Keys.lagger, function(k) Keys.lagger=k end, "lagger")
        end)
        page.LayoutOrder = 1
    end

    -- Combat Page (Fight)
    do
        local page = buildPage("Fight", function()
            makeGap(2); makeSectionHeader("Bat Assist"); makeGap(2)
            setAutoSwing = makeToggleRow("Auto Swing", false, function(on) State.autoSwingEnabled=on end)
            toggleSetters["autoSwing"] = setAutoSwing
            setBatCounter = makeToggleRow("Bat Counter", false, function(on) State.batCounterEnabled=on; if on then startBatCounter() else stopBatCounter() end end)
            toggleSetters["batCounter"] = setBatCounter
            setMedusaCounter = makeToggleRow("Medusa Counter", false, function(on) State.medusaCounterEnabled=on; if on then setupMedusaCounter(LP.Character) else stopMedusaCounter() end end)
            toggleSetters["medusaCounter"] = setMedusaCounter
            makeGap(4)
            makeSectionHeader("Aimbot Mode")
            makeGap(2)
            makeAimbotModeSelector()
            makeGap(8); makeSectionHeader("Reset & Tracers"); makeGap(2)
            local setMedReset = makeToggleRow("Auto Reset on Medusa", State.instantResetOnMedusa, function(on)
                State.instantResetOnMedusa = on
            end)
            toggleSetters["medusaAutoReset"] = setMedReset
            local setTracers = makeToggleRow("Tracers", State.tracersEnabled, function(on)
                State.tracersEnabled = on
                if not on then clearTracers() end
            end)
            toggleSetters["tracers"] = setTracers
            makeGap(8); makeSectionHeader("Gamepad Binds"); makeGap(2)
            makeKeybindRow("Instant Reset", Keys.instantReset, function(k) Keys.instantReset=k end, "instantReset")
            makeKeybindRow("Aim Key", Keys.aimbot, function(k) Keys.aimbot=k end, "aimbot")
            makeKeybindRow("TP Bat", Keys.tpBat, function(k) Keys.tpBat=k end, "tpBat")
        end)
        page.LayoutOrder = 2
    end

    -- Auto Steal Page
    do
        local page = buildPage("Snatch", function()
            makeGap(2); makeSectionHeader("Snatch Mode"); makeGap(2)
            setInstaGrab = makeToggleRow("Auto Snatch", true, function(on) Steal.AutoStealEnabled=on; if on then startAutoSteal() else stopAutoSteal() end end)
            toggleSetters["autoSteal"] = setInstaGrab
            makeGap(6); makeSectionHeader("Snatch Config"); makeGap(2)
            stealRadBox = makeInputRow("Snatch Radius", Steal.StealRadius, function(n) if n then n=math.clamp(math.floor(n),1,500); Steal.StealRadius=n; if stealPctLbl and not stealPctLbl:IsFocused() then stealPctLbl.Text=tostring(n) end; pcall(requestSave) end end)
            local durBox,_ = makeInputRow("Snatch Delay", Steal.StealDuration, function(n) if n then n=math.min(n,10); if n>=0.05 then Steal.StealDuration=n end end end)
            stealDurBox = durBox
        end)
        page.LayoutOrder = 3
    end

    -- Movement Page
    do
        local page = buildPage("Drift", function()
            makeGap(2); makeSectionHeader("Air Control"); makeGap(2)
            setInfJump = makeToggleRow("Sky Walk", true, function(on) State.infJumpEnabled=on end)
            toggleSetters["infJump"] = setInfJump
            local skyModeRow = Instance.new("Frame", currentPage)
            skyModeRow.Size = UDim2.new(1,-16,0,32); skyModeRow.BackgroundTransparency=1
            skyModeRow.BorderSizePixel=0; skyModeRow.LayoutOrder=LO()
            local skyModes = {"hold","single"}
            local skyModeLabels = {hold="Hold", single="Single"}
            local skyModeBtns = {}
            local skyBtnW = 70; local skyBtnGap = 6
            local function setSkyMode(mode)
                State.skyJumpMode = mode
                if mode ~= "hold" and InfJumpPlatform then
                    InfJumpPlatform.Position = Vector3.new(0, -1000, 0)
                end
                for _, m in ipairs(skyModes) do
                    local b = skyModeBtns[m]
                    if b then
                        local active = (m == mode)
                        TweenService:Create(b, TweenInfo.new(0.15, Enum.EasingStyle.Quad), {
                            BackgroundColor3 = active and Color3.fromRGB(220,220,220) or Color3.fromRGB(28,28,28),
                            TextColor3 = active and Color3.fromRGB(0,0,0) or Color3.fromRGB(160,160,160),
                        }):Play()
                    end
                end
                requestSave()
            end
            for i, mode in ipairs(skyModes) do
                local btn = Instance.new("TextButton", skyModeRow)
                btn.Size = UDim2.new(0, skyBtnW, 1, 0)
                btn.Position = UDim2.new(0, (i-1)*(skyBtnW+skyBtnGap), 0, 0)
                btn.BackgroundColor3 = (mode == State.skyJumpMode) and Color3.fromRGB(220,220,220) or Color3.fromRGB(28,28,28)
                btn.TextColor3 = (mode == State.skyJumpMode) and Color3.fromRGB(0,0,0) or Color3.fromRGB(160,160,160)
                btn.Text = skyModeLabels[mode]; btn.Font = Enum.Font.GothamBold
                btn.TextSize = 11; btn.BorderSizePixel = 0; btn.ZIndex = 4
                mkCorner(btn, 7); mkStroke(btn, Color3.fromRGB(50,50,50), 1)
                skyModeBtns[mode] = btn
                btn.MouseButton1Click:Connect(function() setSkyMode(mode) end)
            end
            makeGap(8); makeSectionHeader("Defense"); makeGap(2)
            setAntiRag = makeToggleRow("Anti Ragdoll", false, function(on) State.antiRagdollEnabled=on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
            toggleSetters["antiRagdoll"] = setAntiRag
            makeGap(8); makeSectionHeader("Position Binds"); makeGap(2)

            -- DROP TYPE SELECTOR (Stand / Jump)
            local dropTypeRow = Instance.new("Frame", currentPage)
            dropTypeRow.Size = UDim2.new(1,-16,0,42)
            dropTypeRow.BackgroundColor3 = Color3.fromRGB(15,15,15)
            dropTypeRow.BackgroundTransparency = 0.78
            dropTypeRow.BorderSizePixel = 0
            dropTypeRow.LayoutOrder = LO()
            mkCorner(dropTypeRow, 12)
            local dropTypeStroke = mkStroke(dropTypeRow, Color3.fromRGB(55,55,55), 1)
            dropTypeStroke.Transparency = 0.5

            local dropTypeLbl = Instance.new("TextLabel", dropTypeRow)
            dropTypeLbl.Size = UDim2.new(0.4, 0, 1, 0)
            dropTypeLbl.Position = UDim2.new(0, 14, 0, 0)
            dropTypeLbl.BackgroundTransparency = 1
            dropTypeLbl.Text = "Drop Type"
            dropTypeLbl.TextColor3 = C.rowLabel
            dropTypeLbl.Font = Enum.Font.GothamBold
            dropTypeLbl.TextSize = 13
            dropTypeLbl.TextXAlignment = Enum.TextXAlignment.Left

            local segPill = Instance.new("Frame", dropTypeRow)
            segPill.Size = UDim2.new(0, 160, 0, 32)
            segPill.Position = UDim2.new(1, -170, 0.5, -16)
            segPill.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
            segPill.BorderSizePixel = 0
            segPill.ZIndex = 10
            mkCorner(segPill, 10)
            mkStroke(segPill, Color3.fromRGB(55, 55, 55), 1)

            local segHighlight = Instance.new("Frame", segPill)
            segHighlight.Size = UDim2.new(0.5, -4, 1, -6)
            segHighlight.Position = (currentDropType == DROP_TYPES.STAND)
                and UDim2.new(0, 3, 0, 3)
                or  UDim2.new(0.5, 1, 0, 3)
            segHighlight.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
            segHighlight.BorderSizePixel = 0
            segHighlight.ZIndex = 11
            mkCorner(segHighlight, 8)

            standDropBtn = Instance.new("TextButton", segPill)
            standDropBtn.Size = UDim2.new(0.5, 0, 1, 0)
            standDropBtn.Position = UDim2.new(0, 0, 0, 0)
            standDropBtn.BackgroundTransparency = 1
            standDropBtn.BorderSizePixel = 0
            standDropBtn.Text = "STAND"
            standDropBtn.TextColor3 = (currentDropType == DROP_TYPES.STAND) and Color3.fromRGB(255,255,255) or Color3.fromRGB(110,110,110)
            standDropBtn.Font = Enum.Font.GothamBold
            standDropBtn.TextSize = 11
            standDropBtn.ZIndex = 12

            jumpDropBtn = Instance.new("TextButton", segPill)
            jumpDropBtn.Size = UDim2.new(0.5, 0, 1, 0)
            jumpDropBtn.Position = UDim2.new(0.5, 0, 0, 0)
            jumpDropBtn.BackgroundTransparency = 1
            jumpDropBtn.BorderSizePixel = 0
            jumpDropBtn.Text = "JUMP"
            jumpDropBtn.TextColor3 = (currentDropType == DROP_TYPES.JUMP) and Color3.fromRGB(255,255,255) or Color3.fromRGB(110,110,110)
            jumpDropBtn.Font = Enum.Font.GothamBold
            jumpDropBtn.TextSize = 11
            jumpDropBtn.ZIndex = 12

            standDropBtn.MouseButton1Click:Connect(function()
                currentDropType = DROP_TYPES.STAND
                TweenService:Create(segHighlight, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                    Position = UDim2.new(0, 3, 0, 3)
                }):Play()
                standDropBtn.TextColor3 = Color3.fromRGB(255,255,255)
                jumpDropBtn.TextColor3 = Color3.fromRGB(110,110,110)
                requestSave()
                print("[Asta Hub] Drop type changed to: Stand Drop (Brainrot fling)")
            end)

            jumpDropBtn.MouseButton1Click:Connect(function()
                currentDropType = DROP_TYPES.JUMP
                TweenService:Create(segHighlight, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
                    Position = UDim2.new(0.5, 1, 0, 3)
                }):Play()
                jumpDropBtn.TextColor3 = Color3.fromRGB(255,255,255)
                standDropBtn.TextColor3 = Color3.fromRGB(110,110,110)
                requestSave()
                print("[Asta Hub] Drop type changed to: Jump Drop (ascend)")
            end)

            -- Auto TP
            makeGap(8); makeSectionHeader("Floor Snap"); makeGap(2)
            local autoTPToggle = makeToggleRow("Floor Snap", State.autoTPEnabled, function(on)
                State.autoTPEnabled = on
                if on then startAutoTP() else stopAutoTP() end
                requestSave()
            end)
            toggleSetters["autoTP"] = autoTPToggle
            autoTPHeightBox = makeInputRow("Auto TP Height", State.autoTPHeight, function(n)
                if n and n >= 2 and n <= 500 then State.autoTPHeight = n end
            end)
            makeGap(8); makeSectionHeader("Gamepad Binds"); makeGap(2)
            makeKeybindRow("Left Pos", Keys.autoLeft, function(k) Keys.autoLeft=k end, "autoLeft")
            makeKeybindRow("Right Pos", Keys.autoRight, function(k) Keys.autoRight=k end, "autoRight")
            makeKeybindRow("Drop Key", Keys.drop, function(k) Keys.drop=k end, "drop")
            makeKeybindRow("Floor TP", Keys.tpDown, function(k) Keys.tpDown=k end, "tpDown")
        end)
        page.LayoutOrder = 4
    end

    -- Visual Page
    local antiLagSetter, stretchSetter
    local nukeSetter, removeAccSetter, tryardSetter
    do
        local page = buildPage("Looks", function()
            makeGap(2); do
                local bgRow = Instance.new("Frame", currentPage)
                bgRow.Size = UDim2.new(0,0,0,0)
                bgRow.BackgroundTransparency = 1
                bgRow.BorderSizePixel = 0
                bgRow.LayoutOrder = LO()
                bgRow.Visible = false

                local THUMB_SIZE = 42
                local THUMB_GAP = 8
                local thumbBtns = {}

                local function refreshThumbBorders()
                    for idx, tb in ipairs(thumbBtns) do
                        local stroke = tb:FindFirstChildOfClass("UIStroke")
                        if stroke then
                            stroke.Color = (idx == State.bgSelectedIndex)
                                and Color3.fromRGB(255,255,255)
                                or  Color3.fromRGB(60,60,60)
                            stroke.Thickness = (idx == State.bgSelectedIndex) and 2.5 or 1
                        end
                    end
                end

                local function applyBg(idx)
                    State.bgSelectedIndex = idx
                    local newId = BG_PHOTOS[idx] or BG_PHOTOS[1]
                    TweenService:Create(bgImg, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {ImageTransparency=1}):Play()
                    task.delay(0.25, function()
                        -- bgImg.Image = newId  (photo background removed)
                        TweenService:Create(bgImg, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {ImageTransparency=0}):Play()
                    end)
                    refreshThumbBorders()
                    requestSave()
                end

                for i = 1, #BG_PHOTOS do
                    local tb = Instance.new("ImageButton", bgRow)
                    tb.Size = UDim2.new(0, THUMB_SIZE, 0, THUMB_SIZE)
                    tb.Position = UDim2.new(0, 12 + (i-1)*(THUMB_SIZE+THUMB_GAP), 0.5, -THUMB_SIZE/2)
                    tb.BackgroundColor3 = Color3.fromRGB(10,10,10)
                    tb.BorderSizePixel = 0
                    tb.Image = BG_PHOTOS[i]
                    tb.ScaleType = Enum.ScaleType.Crop
                    tb.ZIndex = 10
                    mkCorner(tb, 10)
                    local stroke = Instance.new("UIStroke", tb)
                    stroke.Thickness = (i == State.bgSelectedIndex) and 2.5 or 1
                    stroke.Color = (i == State.bgSelectedIndex)
                        and Color3.fromRGB(255,255,255) or Color3.fromRGB(60,60,60)
                    local numLbl = Instance.new("TextLabel", tb)
                    numLbl.Size = UDim2.new(1,0,0,16)
                    numLbl.Position = UDim2.new(0,0,1,-16)
                    numLbl.BackgroundColor3 = Color3.fromRGB(0,0,0)
                    numLbl.BackgroundTransparency = 0.3
                    numLbl.Text = tostring(i)
                    numLbl.TextColor3 = Color3.fromRGB(220,220,220)
                    numLbl.Font = Enum.Font.GothamBold
                    numLbl.TextSize = 11
                    numLbl.BorderSizePixel = 0
                    numLbl.ZIndex = 11
                    mkCorner(numLbl, 10)
                    tb.MouseButton1Click:Connect(function() applyBg(i) end)
                    table.insert(thumbBtns, tb)
                end
            end
            makeGap(8); makeSectionHeader("Optimize"); makeGap(2)
            antiLagSetter = makeToggleRow("Anti-Lag", State.antiLagEnabled, function(on) State.antiLagEnabled=on; if on then enableAntiLag() else disableAntiLag() end end)
            toggleSetters["antiLag"] = antiLagSetter
            local fpsSetter = makeToggleRow("FPS Boost", State.fpsBoostEnabled, function(on) State.fpsBoostEnabled=on; if on then applyFPSBoost() else disableFPSBoost() end; requestSave() end)
            toggleSetters["fpsBoost"] = fpsSetter
            stretchSetter = makeToggleRow("Stretch Rez", State.stretchedResEnabled, function(on) State.stretchedResEnabled=on; if on then enableStretchRez() else disableStretchRez() end end)
            toggleSetters["stretchedRes"] = stretchSetter
            do
                local fovRow = Instance.new("Frame", currentPage); fovRow.Size = UDim2.new(1,-16,0,42); fovRow.BackgroundColor3=Color3.fromRGB(15,15,15); fovRow.BackgroundTransparency=0.78; fovRow.BorderSizePixel=0; fovRow.LayoutOrder=LO(); mkCorner(fovRow,12)
                local fovStroke = mkStroke(fovRow, Color3.fromRGB(55,55,55),1); fovStroke.Transparency=0.5
                local fovLabel = Instance.new("TextLabel", fovRow); fovLabel.Size = UDim2.new(0.4,0,1,0); fovLabel.Position = UDim2.new(0,14,0,0); fovLabel.BackgroundTransparency=1; fovLabel.Text="Stretch FOV"; fovLabel.TextColor3=C.rowLabel; fovLabel.Font=Enum.Font.GothamBold; fovLabel.TextSize=13; fovLabel.TextXAlignment=Enum.TextXAlignment.Left
                local btnFrame = Instance.new("Frame", fovRow); btnFrame.Size = UDim2.new(0,150,0,28); btnFrame.Position = UDim2.new(1,-162,0.5,-14); btnFrame.BackgroundTransparency=1
                local function makeFOVBtn(val,x)
                    local btn = Instance.new("TextButton", btnFrame); btn.Size = UDim2.new(0,44,0,28); btn.Position = UDim2.new(0,x,0,0); btn.BackgroundColor3=C.modeBtnBg; btn.BorderSizePixel=0; btn.Text=tostring(val); btn.TextColor3=C.modeBtnTxt; btn.Font=Enum.Font.GothamBold; btn.TextSize=12; mkCorner(btn,6); mkStroke(btn, C.modeBtnBrd,1)
                    if val == State.stretchFOV then btn.BackgroundColor3=C.modeBtnActBg; btn.TextColor3=C.modeBtnActTx end
                    btn.MouseButton1Click:Connect(function()
                        State.stretchFOV=val; if State.stretchedResEnabled then applyStretchFOV(val) end
                        for _,b in pairs(btnFrame:GetChildren()) do if b:IsA("TextButton") then local v=tonumber(b.Text); if v==val then TweenService:Create(b,TweenInfo.new(0.15),{BackgroundColor3=C.modeBtnActBg,TextColor3=C.modeBtnActTx}):Play() else TweenService:Create(b,TweenInfo.new(0.15),{BackgroundColor3=C.modeBtnBg,TextColor3=C.modeBtnTxt}):Play() end end end
                        requestSave()
                    end)
                    return btn
                end
                makeFOVBtn(90,0); makeFOVBtn(120,53); makeFOVBtn(180,106)
            end
            local cleanBtnWrap = Instance.new("Frame", currentPage); cleanBtnWrap.Size = UDim2.new(1,-16,0,46); cleanBtnWrap.BackgroundTransparency=1; cleanBtnWrap.LayoutOrder=LO()
            local cleanBtn = Instance.new("TextButton", cleanBtnWrap); cleanBtn.Size = UDim2.new(1,0,0,32); cleanBtn.Position = UDim2.new(0,0,0,7); cleanBtn.BackgroundColor3=C.btnBg; cleanBtn.BorderSizePixel=0; cleanBtn.Text="🧹 Clean Particles & Lights"; cleanBtn.TextColor3=C.btnTxt; cleanBtn.Font=Enum.Font.GothamBold; cleanBtn.TextSize=12; mkCorner(cleanBtn,6); mkStroke(cleanBtn, C.btnBorder,1)
            cleanBtn.MouseEnter:Connect(function() TweenService:Create(cleanBtn,TweenInfo.new(0.1),{BackgroundColor3=C.btnHov}):Play() end)
            cleanBtn.MouseLeave:Connect(function() TweenService:Create(cleanBtn,TweenInfo.new(0.1),{BackgroundColor3=C.btnBg}):Play() end)
            cleanBtn.MouseButton1Click:Connect(cleanParticlesAndLights)

            makeGap(8); makeSectionHeader("Sky Colors"); makeGap(2)
            do
                local skyIndex = 1
                local current = State.activeSky or "Off"
                for i, name in ipairs(CANDY_SKY_ORDER) do if name == current then skyIndex = i; break end end
                local skyRow = Instance.new("Frame", currentPage)
                skyRow.Size = UDim2.new(1,-16,0,32); skyRow.BackgroundColor3=C.modeBtnBg; skyRow.BackgroundTransparency=0.78; skyRow.BorderSizePixel=0; skyRow.LayoutOrder=LO()
                mkCorner(skyRow,6); mkStroke(skyRow, C.modeBtnBrd, 1)
                local skyLbl = Instance.new("TextLabel", skyRow)
                skyLbl.Size=UDim2.new(0,80,1,0); skyLbl.Position=UDim2.new(0,10,0,0); skyLbl.BackgroundTransparency=1
                skyLbl.Text="Sky Theme"; skyLbl.TextColor3=C.modeBtnTxt; skyLbl.Font=Enum.Font.GothamBold; skyLbl.TextSize=11; skyLbl.TextXAlignment=Enum.TextXAlignment.Left
                local skyVal = Instance.new("TextLabel", skyRow)
                skyVal.Size=UDim2.new(0,130,1,0); skyVal.Position=UDim2.new(1,-138,0,0); skyVal.BackgroundTransparency=1
                skyVal.Text=CANDY_SKY_ORDER[skyIndex]; skyVal.TextColor3=C.accent; skyVal.Font=Enum.Font.GothamBlack; skyVal.TextSize=11; skyVal.TextXAlignment=Enum.TextXAlignment.Right
                local skyClk = Instance.new("TextButton", skyRow)
                skyClk.Size=UDim2.new(1,0,1,0); skyClk.BackgroundTransparency=1; skyClk.Text=""; skyClk.ZIndex=2
                skyClk.MouseButton1Click:Connect(function()
                    skyIndex = skyIndex % #CANDY_SKY_ORDER + 1
                    local label = CANDY_SKY_ORDER[skyIndex]
                    skyVal.Text = label
                    State.activeSky = (label == "Off") and nil or label
                    applySky(label)
                    requestSave()
                end)
            end
            makeGap(4)
            local resetSkyBtn = Instance.new("TextButton", currentPage); resetSkyBtn.Size = UDim2.new(1,-16,0,32); resetSkyBtn.BackgroundColor3=Color3.fromRGB(80,25,25); resetSkyBtn.BorderSizePixel=0; resetSkyBtn.Text="Restore Default Lighting"; resetSkyBtn.TextColor3=Color3.fromRGB(255,200,200); resetSkyBtn.Font=Enum.Font.GothamBold; resetSkyBtn.TextSize=11; resetSkyBtn.LayoutOrder=LO(); mkCorner(resetSkyBtn,6); mkStroke(resetSkyBtn, Color3.fromRGB(130,45,45),1)
            resetSkyBtn.MouseEnter:Connect(function() TweenService:Create(resetSkyBtn,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(110,35,35)}):Play() end)
            resetSkyBtn.MouseLeave:Connect(function() TweenService:Create(resetSkyBtn,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(80,25,25)}):Play() end)
            resetSkyBtn.MouseButton1Click:Connect(function()
                applySky(nil); State.activeSky=nil
                requestSave()
            end)

            makeGap(8); makeSectionHeader("Extras"); makeGap(2)
            nukeSetter = makeToggleRow("Map Cleaner", false, function(on) State.nukeOpt=on; if on then _G._nukeStart() else _G._nukeStop() end end)
            toggleSetters["nukeOpt"] = nukeSetter
            removeAccSetter = makeToggleRow("Strip Accessories", false, function(on) State.removeAcc=on; if on then _G._removeAccStart() else _G._removeAccStop() end end)
            toggleSetters["removeAcc"] = removeAccSetter
            tryardSetter = makeToggleRow("Tryhard Anims", State.tryardAnimEnabled, function(on) State.tryardAnimEnabled=on; if on then startTryardAnim() else stopTryardAnim() end end)
            toggleSetters["tryardAnim"] = tryardSetter
            _G._VezyFOV = _G._VezyFOV or 70
            makeInputRow("Base FOV", _G._VezyFOV, function(n) if n>=70 and n<=180 then _G._VezyFOV=n; local cam=workspace.CurrentCamera; if cam and not State.stretchedResEnabled then pcall(function() cam.FieldOfView=n end) end end end)
        end)
        page.LayoutOrder = 5
    end

    -- Settings Page (NO WEBHOOK UI)
    local introSetter, hideButtonsSetter, lockButtonsSetter
    do
        local page = buildPage("Config", function()
            makeGap(2); makeSectionHeader("Interface"); makeGap(2)
            uiScaleBox = makeInputRow("UI Scale", 1.0, function(n) if n>=0.5 and n<=2.0 then if uiScaleObj then uiScaleObj.Scale=n end end end)
            do
                local SCALE_OPTIONS = {0.75, 1.0, 1.25, 1.5, 1.75}
                local sizeRow = Instance.new("Frame", currentPage)
                sizeRow.Size = UDim2.new(1,-16,0,32); sizeRow.BackgroundColor3=C.modeBtnBg; sizeRow.BorderSizePixel=0; sizeRow.LayoutOrder=LO()
                mkCorner(sizeRow,6); mkStroke(sizeRow, C.modeBtnBrd,1)
                local sizeLbl = Instance.new("TextLabel", sizeRow)
                sizeLbl.Size=UDim2.new(0,100,1,0); sizeLbl.Position=UDim2.new(0,10,0,0); sizeLbl.BackgroundTransparency=1
                sizeLbl.Text="Button Size"; sizeLbl.TextColor3=C.modeBtnTxt; sizeLbl.Font=Enum.Font.GothamBold; sizeLbl.TextSize=11; sizeLbl.TextXAlignment=Enum.TextXAlignment.Left
                local sizeButtons = {}
                local btnW, step = 28, 32
                local totalW = (step * (#SCALE_OPTIONS - 1)) + btnW
                for i, v in ipairs(SCALE_OPTIONS) do
                    local sb = Instance.new("TextButton", sizeRow)
                    sb.Size = UDim2.new(0, btnW, 0, 20)
                    sb.Position = UDim2.new(1, -totalW + (i-1)*step - 8, 0.5, -10)
                    sb.BackgroundColor3 = Color3.fromRGB(255,255,255); sb.BorderSizePixel=0
                    sb.Text = tostring(math.floor(v*100)).."%"
                    sb.TextColor3 = Color3.fromRGB(15,5,25); sb.Font = Enum.Font.GothamBlack; sb.TextSize = 9
                    sb.AutoButtonColor = false; sb.ZIndex = 4
                    mkCorner(sb, 10)
                    local sbStroke = mkStroke(sb, C.accent, 1); sbStroke.Transparency = 0.36
                    sizeButtons[i] = {btn=sb, stroke=sbStroke, value=v}
                    sb.MouseButton1Click:Connect(function()
                        mobileButtonScale = v
                        applyMobileButtonScale()
                        for _, wrapper in pairs(stackWrappers) do
                            TweenService:Create(wrapper, TweenInfo.new(0.15), {
                                Size = UDim2.new(0, math.floor(BTN_W*v), 0, math.floor(BTN_H*v))
                            }):Play()
                        end
                        for _, item in ipairs(sizeButtons) do
                            local active = math.abs(mobileButtonScale - item.value) < 0.01
                            item.btn.BackgroundTransparency = active and 0 or 0.78
                            item.btn.TextColor3 = active and Color3.fromRGB(15,5,25) or Color3.fromRGB(180,180,200)
                            item.stroke.Transparency = active and 0.04 or 0.36
                        end
                        requestSave()
                    end)
                end
                local function refreshSizeUI()
                    for _, item in ipairs(sizeButtons) do
                        local active = math.abs(mobileButtonScale - item.value) < 0.01
                        item.btn.BackgroundTransparency = active and 0 or 0.78
                        item.btn.TextColor3 = active and Color3.fromRGB(15,5,25) or Color3.fromRGB(180,180,200)
                        item.stroke.Transparency = active and 0.04 or 0.36
                    end
                end
                table.insert(mobileSizeSetters, refreshSizeUI)
                refreshSizeUI()
            end
            hideButtonsSetter = makeToggleRow("Hide Buttons", false, function(on) State.stackButtonsHidden=on; for _,wrapper in pairs(stackWrappers) do wrapper.Visible=not on end end)
            toggleSetters["hideButtons"] = hideButtonsSetter
            lockButtonsSetter = makeToggleRow("Lock Buttons", false, function(on) State.stackButtonsLocked=on end)
            toggleSetters["lockButtons"] = lockButtonsSetter
            introSetter = makeToggleRow("Intro Screen", State.introEnabled, function(on) State.introEnabled=on; requestSave() end)
            toggleSetters["introEnabled"] = introSetter

            makeGap(8); makeSectionHeader("Data"); makeGap(2)
            do
                local _saveDone = false
                local saveSetter = makeToggleRow("Save Config", false, function(on)
                    if not on then return end
                    local success = pcall(saveConfig)
                    task.delay(0.05, function()
                        if saveSetter then pcall(saveSetter, false) end
                    end)
                    if _G._VezyFlashSave then _G._VezyFlashSave(success) end
                end)
            end
            do
                local _stage = 0; local _timer = nil
                local _resetSetter = nil
                _resetSetter = makeToggleRow("Wipe Settings", false, function(on)
                    if not on then
                        _stage = 0
                        if _timer then task.cancel(_timer); _timer = nil end
                        return
                    end
                    if _stage == 0 then
                        _stage = 1
                        if _timer then task.cancel(_timer) end
                        _timer = task.delay(3, function()
                            _stage = 0
                            if _resetSetter then pcall(_resetSetter, false) end
                        end)
                    else
                        _stage = 0
                        if _timer then task.cancel(_timer); _timer = nil end
                        -- perform reset
                        pcall(function() if State.batAimbotToggled then stopBatAimbot() end end)
                        pcall(function() if State.batCounterEnabled then stopBatCounter() end end)
                        pcall(function() if State.medusaCounterEnabled then stopMedusaCounter() end end)
                        pcall(function() if State.antiRagdollEnabled then stopAntiRagdoll() end end)
                        pcall(function() if Steal.AutoStealEnabled then stopAutoSteal() end end)
                        pcall(function() if State.autoLeftEnabled then stopAutoLeft() end end)
                        pcall(function() if State.autoRightEnabled then stopAutoRight() end end)
                        pcall(function() if State.antiLagEnabled then disableAntiLag() end end)
                        pcall(function() if State.stretchedResEnabled then disableStretchRez() end end)
                        pcall(function() if State.autoTPEnabled then stopAutoTP() end end)
                        pcall(function() if _G._NukeOn and _G._nukeStop then _G._nukeStop() end end)
                        pcall(function() if _G._RemoveAccOn and _G._removeAccStop then _G._removeAccStop() end end)
                        applySky(nil)
                        State.normalSpeed=60; State.carrySpeed=30; State.laggerSpeed=10.1; State.laggerCarrySpeed=15
                        State.speedToggled=false; State.laggerMode=0; State.infJumpEnabled=true; State.antiRagdollEnabled=false
                        State.antiLagEnabled=false; State.stretchedResEnabled=false
                        State.stretchFOV=120; State.activeSky=nil; State.medusaCounterEnabled=false; State.batCounterEnabled=false
                        State.batAimbotToggled=false; State.autoSwingEnabled=false; State.autoLeftEnabled=false; State.autoRightEnabled=false
                        State.stackButtonsHidden=false; State.stackButtonsLocked=false; State.introEnabled=true
                        State.autoTPEnabled=false; State.autoTPHeight=20
                        State.aimbotMode = "normal"
                        Steal.StealRadius=55; Steal.StealDuration=0.25; Steal.AutoStealEnabled=true

                        currentDropType = DROP_TYPES.STAND
                        if standDropBtn then
                            standDropBtn.BackgroundColor3 = C.accent
                            standDropBtn.TextColor3 = Color3.fromRGB(0,20,8)
                            jumpDropBtn.BackgroundColor3 = C.inputBg
                            jumpDropBtn.TextColor3 = C.inputTxt
                        end
                        if normalBox then normalBox.Text=tostring(State.normalSpeed) end; if carryBox then carryBox.Text=tostring(State.carrySpeed) end
                        if laggerBox then laggerBox.Text=tostring(State.laggerSpeed) end; if laggerCarryBox then laggerCarryBox.Text=tostring(State.laggerCarrySpeed) end
                        if stealRadBox then stealRadBox.Text=tostring(Steal.StealRadius) end; if stealDurBox then stealDurBox.Text=tostring(Steal.StealDuration) end
                        if uiScaleObj then uiScaleObj.Scale=1.0 end; if uiScaleBox then uiScaleBox.Text="1" end
                        if setInstaGrab then pcall(setInstaGrab,true) end; if setInfJump then pcall(setInfJump,true) end; if setAntiRag then pcall(setAntiRag,false) end
                        if setMedusaCounter then pcall(setMedusaCounter,false) end; if setBatCounter then pcall(setBatCounter,false) end; if setAutoSwing then pcall(setAutoSwing,false) end
                        if hideButtonsSetter then pcall(hideButtonsSetter,false) end; if lockButtonsSetter then pcall(lockButtonsSetter,false) end
                        if introSetter then pcall(introSetter,true) end
                        if stackBtnRefs then for key,ref in pairs(stackBtnRefs) do if ref and ref.setOn then pcall(ref.setOn,false) end end end
                        
                        for i,def in ipairs(stackDefs) do local wrapper=stackWrappers[def.key]; if wrapper then TweenService:Create(wrapper,TweenInfo.new(0.35,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=getDefaultStackPos(i)}):Play() end end
                        task.delay(0.05, function() if _resetSetter then pcall(_resetSetter, false) end end)
                    end
                end)
            end
            makeGap(8); makeSectionHeader("Layout"); makeGap(2)
            do
                local _posSetter = nil
                _posSetter = makeToggleRow("Reset Positions", false, function(on)
                    if not on then return end
                    for i,def in ipairs(stackDefs) do local wrapper=stackWrappers[def.key]; if wrapper then TweenService:Create(wrapper,TweenInfo.new(0.35,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=getDefaultStackPos(i)}):Play() end end
                    task.delay(0.05, function() if _posSetter then pcall(_posSetter, false) end end)
                end)
            end
            makeGap(8); makeSectionHeader("Gamepad Binds"); makeGap(2)
            makeKeybindRow("Hide GUI", Keys.guiHide, function(k) Keys.guiHide=k end, "guiHide")
            makeGap(10)
            local fw = Instance.new("Frame", currentPage); fw.Size = UDim2.new(1,0,0,22); fw.BackgroundTransparency=1; fw.BorderSizePixel=0; fw.LayoutOrder=LO()
            local fl = Instance.new("TextLabel", fw); fl.Size = UDim2.new(1,0,1,0); fl.BackgroundTransparency=1; fl.Text="Asta Hub  ·  Made by Zein & Tezy"; fl.TextColor3=Color3.fromRGB(120,120,120); fl.Font=Enum.Font.Gotham; fl.TextSize=10; fl.TextXAlignment=Enum.TextXAlignment.Center
            _G._VezySaveStatusLbl = fl
            _G._VezyFlashSave = function(success)
                if not _G._VezySaveStatusLbl or not _G._VezySaveStatusLbl.Parent then return end
                local lbl = _G._VezySaveStatusLbl
                if success then lbl.Text="✓  Auto-saved"; lbl.TextColor3=Color3.fromRGB(180,180,180)
                else lbl.Text="✗  Save failed"; lbl.TextColor3=Color3.fromRGB(220,80,80) end
                task.delay(1.5,function() if lbl and lbl.Parent then lbl.Text="Asta Hub  ·  Made by Zein & Tezy"; lbl.TextColor3=Color3.fromRGB(120,120,120) end end)
            end
        end)
        page.LayoutOrder = 6
    end


    rebuildPresetList = function()
        if not presetListFrame then return end
        for _,child in ipairs(presetListFrame:GetChildren()) do if child.Name~="EmptyLabel" and not child:IsA("UIListLayout") and not child:IsA("UIPadding") then child:Destroy() end end
        local emptyLbl = presetListFrame:FindFirstChild("EmptyLabel")
        if emptyLbl then emptyLbl.Visible = (#Presets == 0) end
        for i,preset in ipairs(Presets) do
            local row = Instance.new("Frame", presetListFrame); row.Name="Preset_"..i; row.Size=UDim2.new(1,0,0,34); row.BackgroundColor3=C.presetBg; row.BorderSizePixel=0; row.LayoutOrder=i+1; mkCorner(row,6); mkStroke(row, C.presetBrd,1)
            local nameLbl = Instance.new("TextLabel", row); nameLbl.Size=UDim2.new(1,-94,1,0); nameLbl.Position=UDim2.new(0,10,0,0); nameLbl.BackgroundTransparency=1; nameLbl.Text=preset.name; nameLbl.TextColor3=C.rowLabel; nameLbl.Font=Enum.Font.GothamBold; nameLbl.TextSize=12; nameLbl.TextXAlignment=Enum.TextXAlignment.Left; nameLbl.TextTruncate=Enum.TextTruncate.AtEnd
            local loadBtn = Instance.new("TextButton", row); loadBtn.Size=UDim2.new(0,44,0,26); loadBtn.Position=UDim2.new(1,-96,0.5,-13); loadBtn.BackgroundColor3=C.presetLoad; loadBtn.BorderSizePixel=0; loadBtn.Text="Load"; loadBtn.TextColor3=Color3.fromRGB(240,255,245); loadBtn.Font=Enum.Font.GothamBold; loadBtn.TextSize=11; loadBtn.ZIndex=9; mkCorner(loadBtn,5)
            loadBtn.MouseEnter:Connect(function() TweenService:Create(loadBtn,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(180,180,180)}):Play() end)
            loadBtn.MouseLeave:Connect(function() TweenService:Create(loadBtn,TweenInfo.new(0.1),{BackgroundColor3=C.presetLoad}):Play() end)
            loadBtn.MouseButton1Click:Connect(function()
                saveLastPresetName(preset.name); loadBtn.Text="✓"; task.delay(1.2,function() if loadBtn and loadBtn.Parent then loadBtn.Text="Load" end end)
            end)
            local delBtn = Instance.new("TextButton", row); delBtn.Size=UDim2.new(0,34,0,26); delBtn.Position=UDim2.new(1,-48,0.5,-13); delBtn.BackgroundColor3=C.presetDel; delBtn.BorderSizePixel=0; delBtn.Text="✕"; delBtn.TextColor3=Color3.fromRGB(200,80,80); delBtn.Font=Enum.Font.GothamBold; delBtn.TextSize=11; delBtn.ZIndex=9; mkCorner(delBtn,5)
            delBtn.MouseEnter:Connect(function() TweenService:Create(delBtn,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(80,28,28)}):Play() end)
            delBtn.MouseLeave:Connect(function() TweenService:Create(delBtn,TweenInfo.new(0.1),{BackgroundColor3=C.presetDel}):Play() end)
            delBtn.MouseButton1Click:Connect(function()
                table.remove(Presets,i); savePresetsFile(); rebuildPresetList()
            end)
        end
    end

    -- ============================================================
    -- AUTO STEAL BAR
    -- ============================================================
    local stealBarGui = Instance.new("ScreenGui", LP:WaitForChild("PlayerGui"))
    stealBarGui.Name = "AstaHubStealBar"; stealBarGui.ResetOnSpawn=false; stealBarGui.IgnoreGuiInset=true; stealBarGui.DisplayOrder=9

    local stealBar = Instance.new("Frame", stealBarGui)
    stealBar.Size = UDim2.new(0,220,0,8); stealBar.Position = UDim2.new(0.5,-110,1,-60)
    stealBar.BackgroundTransparency = 1; stealBar.BorderSizePixel = 0; stealBar.Active = true
    makeDraggable(stealBar)

    local sbTrack = Instance.new("Frame", stealBar)
    sbTrack.Size = UDim2.new(1,0,1,0); sbTrack.Position = UDim2.new(0,0,0,0)
    sbTrack.BackgroundColor3 = Color3.fromRGB(30,30,30); sbTrack.BackgroundTransparency = 0.3
    sbTrack.BorderSizePixel = 0; mkCorner(sbTrack,4)

    local progressFill = Instance.new("Frame", sbTrack)
    progressFill.Size = UDim2.new(0,0,1,0); progressFill.BackgroundColor3 = Color3.fromRGB(200,200,200)
    progressFill.BorderSizePixel = 0; mkCorner(progressFill,4)

    local fillGlow = Instance.new("Frame", sbTrack)
    fillGlow.Size = UDim2.new(0,0,1,0); fillGlow.BackgroundColor3 = Color3.fromRGB(255,255,255)
    fillGlow.BackgroundTransparency = 0.55; fillGlow.BorderSizePixel = 0; mkCorner(fillGlow,4)

    local stealPctLbl = Instance.new("TextLabel", stealBar)
    stealPctLbl.Size = UDim2.new(0,1,0,1); stealPctLbl.BackgroundTransparency = 1
    stealPctLbl.TextTransparency = 1; stealPctLbl.Text = "0%"

    local radLbl = Instance.new("TextLabel", stealBar)
    radLbl.Size = UDim2.new(0,1,0,1); radLbl.BackgroundTransparency = 1
    radLbl.TextTransparency = 1; radLbl.Text = "Radius: "..Steal.StealRadius

    local radWrap = Instance.new("Frame", stealBar)
    radWrap.Size = UDim2.new(0,1,0,1); radWrap.BackgroundTransparency = 1

    local radTB = Instance.new("TextBox", radWrap)
    radTB.Size = UDim2.new(1,0,1,0); radTB.BackgroundTransparency = 1; radTB.TextTransparency = 1
    radTB.Text = tostring(Steal.StealRadius); radTB.Font = Enum.Font.GothamBlack
    radTB.TextSize = 9; radTB.ClearTextOnFocus = false; radTB.ZIndex = 10
    radTB.FocusLost:Connect(function()
        local n = tonumber(radTB.Text)
        if n and n>=5 and n<=300 then Steal.StealRadius=math.floor(n); Steal.cachedPrompts={}; Steal.promptCacheTime=0 end
        radTB.Text = tostring(Steal.StealRadius); radLbl.Text = "Radius: "..Steal.StealRadius
        if stealRadBox and not stealRadBox:IsFocused() then stealRadBox.Text=tostring(Steal.StealRadius) end
        task.spawn(function() if requestSave then pcall(requestSave) end end)
    end)

    local stealStatusLbl = Instance.new("TextLabel", stealBarGui)
    stealStatusLbl.Size = UDim2.new(0,220,0,16); stealStatusLbl.AnchorPoint = Vector2.new(0.5,1)
    stealStatusLbl.Position = UDim2.new(0.5,0,1,-64); stealStatusLbl.BackgroundTransparency = 1
    stealStatusLbl.Text = ""; stealStatusLbl.TextColor3 = Color3.fromRGB(255,255,255)
    stealStatusLbl.Font = Enum.Font.GothamBold; stealStatusLbl.TextSize = 11
    stealStatusLbl.TextXAlignment = Enum.TextXAlignment.Center; stealStatusLbl.TextTransparency = 1

    task.spawn(function()
        local dotStep=0; local dotSteps={"STEAL.","STEAL..","STEAL..."}
        while true do
            task.wait(0.35)
            if State.isStealing then
                dotStep=dotStep%3+1; stealStatusLbl.Text=dotSteps[dotStep]; stealStatusLbl.TextTransparency=0
            else
                stealStatusLbl.Text=""; stealStatusLbl.TextTransparency=1; dotStep=0
            end
        end
    end)

    -- ============================================================
    -- STACK BUTTONS
    -- ============================================================
    local function updateLaggerButtons()
        if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(State.laggerMode==1) end
        if stackBtnRefs.laggerCarry then stackBtnRefs.laggerCarry.setOn(State.laggerMode==2) end
    end
    
    local function setLaggerMode(mode)
        if mode == State.laggerMode then return end
        local oldMode = State.laggerMode

        if mode == 0 then
            State.carrySpeed = State._prevCarry or 30
            State.speedToggled = State._prevSpeed or false
            if carryBox then
                carryBox.Text = tostring(State.carrySpeed)
            end
            if stackBtnRefs.carrySpeed then
                stackBtnRefs.carrySpeed.setOn(State.speedToggled)
            end
        elseif mode == 1 then
            if oldMode == 0 then
                State._prevCarry = State.carrySpeed
                State._prevSpeed = State.speedToggled
            end
            State.speedToggled = false
            if stackBtnRefs.carrySpeed then
                stackBtnRefs.carrySpeed.setOn(false)
            end
        elseif mode == 2 then
            if oldMode == 0 then
                State._prevCarry = State.carrySpeed
                State._prevSpeed = State.speedToggled
            end
            State.speedToggled = false
            if stackBtnRefs.carrySpeed then
                stackBtnRefs.carrySpeed.setOn(false)
            end
        end

        State.laggerMode = mode
        updateLaggerButtons()
        requestSave()
    end

    local function toggleLaggerMode()
        if State.laggerMode == 0 then
            setLaggerMode(1)
        elseif State.laggerMode == 1 then
            setLaggerMode(2)
        else
            setLaggerMode(1)
        end
    end

    local function toggleSpeed()
        if State.laggerMode ~= 0 then
            setLaggerMode(0)
            return
        end
        State.speedToggled = not State.speedToggled
        if stackBtnRefs.carrySpeed then
            stackBtnRefs.carrySpeed.setOn(State.speedToggled)
        end
        if carryBox then
            carryBox.Text = tostring(State.carrySpeed)
        end
        requestSave()
    end

    for i,def in ipairs(stackDefs) do
        local btnFrame = Instance.new("TextButton", gui)
        btnFrame.Name = "StackBtn_"..def.key
        btnFrame.Size = UDim2.new(0,BTN_W,0,BTN_H)
        btnFrame.Position = getDefaultStackPos(i)
        btnFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10); btnFrame.BorderSizePixel=0
        btnFrame.AutoButtonColor = false
        btnFrame.Text = def.label; btnFrame.TextColor3 = Color3.fromRGB(225, 225, 225)
        btnFrame.TextScaled = false; btnFrame.TextSize = 11
        btnFrame.Font = Enum.Font.GothamBlack
        btnFrame.TextWrapped = true; btnFrame.LineHeight = 1.2
        btnFrame.ZIndex=15
        Instance.new("UICorner", btnFrame).CornerRadius = UDim.new(0, 10)
        local bStroke = Instance.new("UIStroke", btnFrame)
        bStroke.Color = Color3.fromRGB(70, 70, 70)
        bStroke.Thickness = 1
        bStroke.Transparency = 0.4
        stackWrappers[def.key] = btnFrame

        local btnState = false
        local function setOn(on)
            btnState = on
            TweenService:Create(btnFrame,TweenInfo.new(0.18),{BackgroundColor3=on and C.stackActBg or Color3.fromRGB(10,10,10), TextColor3=on and Color3.fromRGB(255,255,255) or Color3.fromRGB(225,225,225)}):Play()
            TweenService:Create(bStroke,TweenInfo.new(0.18),{Color=on and Color3.fromRGB(235,235,235) or Color3.fromRGB(70,70,70), Transparency=on and 0 or 0.4}):Play()
        end
        stackBtnRefs[def.key] = {setOn = setOn}

        local function onTap()
            if def.key == "tpDown" then
                task.spawn(function() if runTPDown then pcall(runTPDown) end; setOn(true); task.wait(0.12); setOn(false) end)
                return
            end
            if def.key == "instantReset" then
                task.spawn(function() setOn(true); pcall(performInstantReset); task.wait(0.25); setOn(false) end)
                return
            end
            if def.key == "tpBat" then
                toggleTPBat()
                return
            end
            if def.key == "drop" then
                task.spawn(function() pcall(runDrop) end)
                return
            end
            if def.key == "carrySpeed" then
                if State.laggerMode == 1 or State.laggerMode == 2 then
                    setLaggerMode(0)
                    if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(false) end
                    if stackBtnRefs.laggerCarry then stackBtnRefs.laggerCarry.setOn(false) end
                    State.speedToggled = true
                else
                    State.speedToggled = not State.speedToggled
                end
                setOn(State.speedToggled)
                if carryBox then carryBox.Text = tostring(State.carrySpeed) end
                requestSave()
                return
            end
            if def.key == "lagger" then
                if State.laggerMode==1 then setLaggerMode(0) else setLaggerMode(1) end
                return
            end
            if def.key == "laggerCarry" then
                if State.laggerMode==2 then setLaggerMode(0) else setLaggerMode(2) end
                return
            end
            local ns = not btnState; setOn(ns)
            if def.key == "autoLeft" then
                State.autoLeftEnabled = ns
                if ns and State.batAimbotToggled then State.batAimbotToggled=false; stopBatAimbot(); if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end end
                if ns then startAutoLeft() else stopAutoLeft() end
            elseif def.key == "autoRight" then
                State.autoRightEnabled = ns
                if ns and State.batAimbotToggled then State.batAimbotToggled=false; stopBatAimbot(); if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end end
                if ns then startAutoRight() else stopAutoRight() end
            elseif def.key == "aimbot" then
                State.batAimbotToggled = ns
                if ns then
                    if State.autoLeftEnabled then State.autoLeftEnabled=false; stopAutoLeft(); if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end end
                    if State.autoRightEnabled then State.autoRightEnabled=false; stopAutoRight(); if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end end
                    pcall(startBatAimbot)
                else stopBatAimbot() end
            end
            requestSave()
        end

        makeStackDraggable(btnFrame, onTap)
    end

    -- ============================================================
    -- AUTO LEFT / RIGHT
    -- ============================================================
    startAutoLeft = function()
        if alConn then alConn:Disconnect() end; State.autoLeftPhase = 1
        alConn = RunService.Heartbeat:Connect(function()
            if not State.autoLeftEnabled then return end
            local char = LP.Character; if not char then return end
            local root = char:FindFirstChild("HumanoidRootPart")
            local hum2 = char:FindFirstChildOfClass("Humanoid")
            if not root or not hum2 then return end
            local spd = State.normalSpeed
            if State.autoLeftPhase == 1 then
                local tgt = Vector3.new(AP_L1.X, root.Position.Y, AP_L1.Z)
                if (tgt - root.Position).Magnitude < 1 then State.autoLeftPhase = 2 end
                local d = (AP_L1 - root.Position); local mv = Vector3.new(d.X, 0, d.Z).Unit
                hum2:Move(mv, false); root.AssemblyLinearVelocity = Vector3.new(mv.X*spd, root.AssemblyLinearVelocity.Y, mv.Z*spd)
            elseif State.autoLeftPhase == 2 then
                local tgt = Vector3.new(AP_L2.X, root.Position.Y, AP_L2.Z)
                if (tgt - root.Position).Magnitude < 1 then
                    hum2:Move(Vector3.zero, false); root.AssemblyLinearVelocity = Vector3.zero
                    State.autoLeftEnabled = false; if alConn then alConn:Disconnect(); alConn = nil end
                    State.autoLeftPhase = 1; if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
                    return
                end
                local d = (AP_L2 - root.Position); local mv = Vector3.new(d.X, 0, d.Z).Unit
                hum2:Move(mv, false); root.AssemblyLinearVelocity = Vector3.new(mv.X*spd, root.AssemblyLinearVelocity.Y, mv.Z*spd)
            end
        end)
    end
    stopAutoLeft = function()
        if alConn then alConn:Disconnect(); alConn = nil end; State.autoLeftPhase = 1
        local char = LP.Character; if char then local hum2 = char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero, false) end end
        if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end
    end

    startAutoRight = function()
        if arConn then arConn:Disconnect() end; State.autoRightPhase = 1
        arConn = RunService.Heartbeat:Connect(function()
            if not State.autoRightEnabled then return end
            local char = LP.Character; if not char then return end
            local root = char:FindFirstChild("HumanoidRootPart")
            local hum2 = char:FindFirstChildOfClass("Humanoid")
            if not root or not hum2 then return end
            local spd = State.normalSpeed
            if State.autoRightPhase == 1 then
                local tgt = Vector3.new(AP_R1.X, root.Position.Y, AP_R1.Z)
                if (tgt - root.Position).Magnitude < 1 then State.autoRightPhase = 2 end
                local d = (AP_R1 - root.Position); local mv = Vector3.new(d.X, 0, d.Z).Unit
                hum2:Move(mv, false); root.AssemblyLinearVelocity = Vector3.new(mv.X*spd, root.AssemblyLinearVelocity.Y, mv.Z*spd)
            elseif State.autoRightPhase == 2 then
                local tgt = Vector3.new(AP_R2.X, root.Position.Y, AP_R2.Z)
                if (tgt - root.Position).Magnitude < 1 then
                    hum2:Move(Vector3.zero, false); root.AssemblyLinearVelocity = Vector3.zero
                    State.autoRightEnabled = false; if arConn then arConn:Disconnect(); arConn = nil end
                    State.autoRightPhase = 1; if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
                    return
                end
                local d = (AP_R2 - root.Position); local mv = Vector3.new(d.X, 0, d.Z).Unit
                hum2:Move(mv, false); root.AssemblyLinearVelocity = Vector3.new(mv.X*spd, root.AssemblyLinearVelocity.Y, mv.Z*spd)
            end
        end)
    end
    stopAutoRight = function()
        if arConn then arConn:Disconnect(); arConn = nil end; State.autoRightPhase = 1
        local char = LP.Character; if char then local hum2 = char:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero, false) end end
        if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end
    end

    -- ============================================================
    -- HELPER FUNCTIONS
    -- ============================================================
    local function resetProgressBar() stealPctLbl.Text="0%"; progressFill.Size=UDim2.new(0,0,1,0) end

    -- ============================================================
    -- ADVANCED AIMBOT (BYPASS MODE) — Candy style from AXONIC HUB
    -- ============================================================
    local _aimbotTarget = nil
    local _aimbotTargetPlr = nil

    local function getClosestTarget()
        local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
        if not root then return nil, nil end
        local closest, closestPlr, minDist = nil, nil, math.huge
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LP and plr.Character then
                local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
                local hum = plr.Character:FindFirstChildOfClass("Humanoid")
                if tRoot and hum and hum.Health > 0 then
                    local dist = (tRoot.Position - root.Position).Magnitude
                    if dist < minDist then
                        minDist = dist
                        closest = tRoot
                        closestPlr = plr
                    end
                end
            end
        end
        return closest, closestPlr, minDist
    end

    local function getStickyTarget(currentRoot)
        local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
        if not root then return nil, nil end
        local newClosest, newPlr, newDist = getClosestTarget()
        if not newClosest then return nil, nil end
        if currentRoot and currentRoot.Parent then
            local currentPlr = Players:GetPlayerFromCharacter(currentRoot.Parent)
            local hum = currentRoot.Parent:FindFirstChildOfClass("Humanoid")
            if currentPlr and hum and hum.Health > 0 then
                local currentDist = (currentRoot.Position - root.Position).Magnitude
                if currentPlr == newPlr or newDist > currentDist * 0.7 then
                    return currentRoot, currentPlr
                end
            end
        end
        return newClosest, newPlr
    end

    local function swingCurrentBat(char)
        if not State.autoSwingEnabled then return end
        local bat = findBat()
        if bat and bat.Parent == char and bat:IsA("Tool") then
            pcall(function() bat:Activate() end)
        end
    end

    -- Normal aimbot (original)
    local function startBatAimbotNormal()
        if Conns.aimbot then Conns.aimbot:Disconnect() end
        State.batAimbotToggled = true
        local hum0 = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
        if hum0 then hum0.AutoRotate = false end

        Conns.aimbot = RunService.RenderStepped:Connect(function()
            if not State.batAimbotToggled then return end
            local char = LP.Character
            if not char then return end
            local root = char:FindFirstChild("HumanoidRootPart")
            if not root then return end
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not hum then return end

            if not char:FindFirstChildOfClass("Tool") then
                local bat = findBat()
                if bat then pcall(function() hum:EquipTool(bat) end) end
            end

            local target = getClosestTarget()
            if not target then return end
            _aimbotTarget = target

            local targetVel = target.AssemblyLinearVelocity
            local myPos = root.Position
            local targetPos = target.Position
            local predictPos = targetPos + targetVel * 0.14
            predictPos = predictPos + target.CFrame.LookVector * 0.3
            local direction = predictPos - myPos
            local flatDir = Vector3.new(direction.X, 0, direction.Z).Unit
            local chaseSpeed = 58
            local desiredHeight = targetPos.Y + 3.7
            local yVel = (desiredHeight - myPos.Y) * 19.5 + targetVel.Y * 0.8
            if hum.FloorMaterial ~= Enum.Material.Air then
                yVel = math.max(yVel, 13)
            end
            yVel = math.clamp(yVel, -70, 110)
            local desiredVel = Vector3.new(flatDir.X * chaseSpeed, yVel, flatDir.Z * chaseSpeed)
            root.AssemblyLinearVelocity = root.AssemblyLinearVelocity:Lerp(desiredVel, 0.8)

            local speed3 = targetVel.Magnitude
            local predictTime = math.clamp(speed3 / 150, 0.05, 0.2)
            local predictedPos = targetPos + targetVel * predictTime
            local toPredict = predictedPos - myPos
            if toPredict.Magnitude > 0.1 then
                local goalCF = CFrame.lookAt(myPos, predictedPos)
                local diffCF = root.CFrame:Inverse() * goalCF
                local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
                rx = math.clamp(rx, -2.5, 2.5)
                ry = math.clamp(ry, -2.5, 2.5)
                rz = math.clamp(rz, -2.5, 2.5)
                root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(Vector3.new(rx * 42, ry * 42, rz * 42))
            end
            if State.autoSwingEnabled then
                local bat = findBat()
                if bat and bat.Parent == char then
                    pcall(function() bat:Activate() end)
                end
            end
        end)
    end

    -- Bypass aimbot (Candy style)
    local function startBatAimbotBypass()
        if Conns.aimbot then Conns.aimbot:Disconnect() end
        State.batAimbotToggled = true
        local hum0 = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
        if hum0 then hum0.AutoRotate = false end

        Conns.aimbot = RunService.RenderStepped:Connect(function()
            if not State.batAimbotToggled then return end
            local char = LP.Character
            if not char then return end
            local root = char:FindFirstChild("HumanoidRootPart")
            if not root then return end
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not hum then return end

            if not char:FindFirstChildOfClass("Tool") then
                local bat = findBat()
                if bat then pcall(function() hum:EquipTool(bat) end) end
            end

            local target, targetPlr = getStickyTarget(_aimbotTarget)
            if not target then
                _aimbotTarget = nil
                _aimbotTargetPlr = nil
                swingCurrentBat(char)
                return
            end
            _aimbotTarget = target
            _aimbotTargetPlr = targetPlr

            local targetVel = target.AssemblyLinearVelocity
            local myPos = root.Position
            local targetPos = target.Position
            local distance = (targetPos - myPos).Magnitude

            local speedFactor = math.clamp(targetVel.Magnitude / 40, 0, 1.2)
            local distFactor = math.clamp(distance / 80, 0, 1)
            local leadTime = 0.14 + speedFactor * 0.12 + distFactor * 0.08
            local predictPos = targetPos + targetVel * leadTime
            predictPos = predictPos + target.CFrame.LookVector * 0.3

            local direction = predictPos - myPos
            local flatDir = Vector3.new(direction.X, 0, direction.Z)
            if flatDir.Magnitude > 0.01 then flatDir = flatDir.Unit else flatDir = Vector3.new(0, 0, 0) end

            local chaseSpeed = 58

            local jumpOffset = math.max(0, targetVel.Y * 0.18)
            local desiredHeight = targetPos.Y + 3.7 + jumpOffset
            local yVel = (desiredHeight - myPos.Y) * 22 + targetVel.Y * 1.1
            if hum.FloorMaterial ~= Enum.Material.Air then
                yVel = math.max(yVel, 13)
            end
            yVel = math.clamp(yVel, -70, 135)

            local desiredVel = Vector3.new(flatDir.X * chaseSpeed, yVel, flatDir.Z * chaseSpeed)
            root.AssemblyLinearVelocity = root.AssemblyLinearVelocity:Lerp(desiredVel, 0.85)

            local rotPredictTime = math.clamp(targetVel.Magnitude / 120, 0.05, 0.25)
            local predictedPos = targetPos + targetVel * rotPredictTime
            local toPredict = predictedPos - myPos
            if toPredict.Magnitude > 0.1 then
                local goalCF = CFrame.lookAt(myPos, predictedPos)
                local diffCF = root.CFrame:Inverse() * goalCF
                local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
                rx = math.clamp(rx, -2.5, 2.5)
                ry = math.clamp(ry, -2.5, 2.5)
                rz = math.clamp(rz, -2.5, 2.5)
                root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(Vector3.new(rx * 50, ry * 50, rz * 50))
            end

            swingCurrentBat(char)
        end)
    end

    -- Main startBatAimbot dispatcher
    startBatAimbot = function()
        if State.aimbotMode == "bypass" then
            startBatAimbotBypass()
        else
            startBatAimbotNormal()
        end
    end

    stopBatAimbot = function()
        if Conns.aimbot then
            Conns.aimbot:Disconnect()
            Conns.aimbot = nil
        end
        _aimbotTarget = nil
        _aimbotTargetPlr = nil
        State.batAimbotToggled = false
        local char = LP.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end
        local hum2 = char and char:FindFirstChildOfClass("Humanoid")
        if hum2 then hum2.AutoRotate = true end
    end

    local _aimbotTarget=nil
    local function findBat()
        local char=LP.Character; if not char then return nil end
        for _,tool in ipairs(char:GetChildren()) do if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then return tool end end
        local bp=LP:FindFirstChild("Backpack"); if bp then for _,tool in ipairs(bp:GetChildren()) do if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then return tool end end end
        return nil
    end

    local BAT_COUNTER_SLAP_LIST={"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
    local function findBatForCounter()
        local c=LP.Character; if not c then return nil end
        local bp=LP:FindFirstChildOfClass("Backpack")
        for _,name in ipairs(BAT_COUNTER_SLAP_LIST) do
            local t=c:FindFirstChild(name) or (bp and bp:FindFirstChild(name))
            if t then return t end
        end
        for _,ch in ipairs(c:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end
        if bp then for _,ch in ipairs(bp:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end end
        return nil
    end
    local function swingBatForCounter(bat,char)
        local hum2=char:FindFirstChildOfClass("Humanoid")
        if bat.Parent~=char then if hum2 then pcall(function() hum2:EquipTool(bat) end) end; task.wait(0.05) end
        local remote=bat:FindFirstChildOfClass("RemoteEvent") or bat:FindFirstChildOfClass("RemoteFunction")
        if remote and remote:IsA("RemoteEvent") then
            pcall(function() remote:FireServer() end); task.wait(0.15); pcall(function() remote:FireServer() end)
        else pcall(function() bat:Activate() end); task.wait(0.15); pcall(function() bat:Activate() end) end
    end
    startBatCounter = function()
        if Conns.batCounter then return end
        Conns.batCounter = RunService.Heartbeat:Connect(function()
            if not State.batCounterEnabled or State.batCounterDebounce then return end
            local char=LP.Character; if not char then return end
            local hum2=char:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
            local st=hum2:GetState()
            local isRagdolled = st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown
            if isRagdolled then
                State.batCounterDebounce=true
                task.spawn(function()
                    local bat=findBatForCounter()
                    if bat then swingBatForCounter(bat,char) end
                    task.wait(0.5); State.batCounterDebounce=false
                end)
            end
        end)
    end
    stopBatCounter = function()
        if Conns.batCounter then Conns.batCounter:Disconnect(); Conns.batCounter=nil end
        State.batCounterDebounce=false
    end

    local MEDUSA_COOLDOWN=0.5
    local function findMedusa()
        local c=LP.Character; if not c then return nil end
        for _,t in ipairs(c:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower(); if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end
        local bp=LP:FindFirstChildOfClass("Backpack")
        if bp then for _,t in ipairs(bp:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower(); if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end end
        return nil
    end
    local function useMedusaCounter()
        if State.medusaDebounce then return end; if tick()-State.medusaLastUsed<MEDUSA_COOLDOWN then return end
        local c=LP.Character; if not c then return end; State.medusaDebounce=true
        local med=findMedusa(); if not med then State.medusaDebounce=false; return end
        if med.Parent~=c then local hum2=c:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
        pcall(function() med:Activate() end); State.medusaLastUsed=tick(); State.medusaDebounce=false
    end
    local function onAnchorChanged(part) return part:GetPropertyChangedSignal("Anchored"):Connect(function() if part.Anchored and part.Transparency==1 then useMedusaCounter() end end) end
    setupMedusaCounter = function(char)
        stopMedusaCounter(); if not char then return end
        for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end
        table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end))
    end
    stopMedusaCounter = function() for _,c2 in pairs(Conns.anchor) do pcall(function() c2:Disconnect() end) end; Conns.anchor={} end

    -- ============================================================
    -- V1 ANTI-RAGDOLL SYSTEM (from AXONIC HUB)
    -- ============================================================
    local _arCachedCharData = {}
    local _arIsBoosting = false
    local _arRagdollConnections = {}
    local _arMode = nil
    local AR_BOOST_SPEED = 400
    local AR_DEFAULT_SPEED = 16

    local function _arCacheCharacterData()
        local char = LP.Character
        if not char then return false end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not root then return false end
        _arCachedCharData = { character = char, humanoid = hum, root = root }
        return true
    end

    local function _arDisconnectAll()
        for _, conn in ipairs(_arRagdollConnections) do
            pcall(function() conn:Disconnect() end)
        end
        _arRagdollConnections = {}
    end

    local function _arIsRagdolled()
        if not _arCachedCharData.humanoid then return false end
        local state = _arCachedCharData.humanoid:GetState()
        if state == Enum.HumanoidStateType.Physics
        or state == Enum.HumanoidStateType.Ragdoll
        or state == Enum.HumanoidStateType.FallingDown then
            return true
        end
        local endTime = LP:GetAttribute("RagdollEndTime")
        if endTime and (endTime - workspace:GetServerTimeNow()) > 0 then
            return true
        end
        return false
    end

    local function _arForceExitRagdoll()
        if not _arCachedCharData.humanoid or not _arCachedCharData.root then return end
        pcall(function()
            LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow())
        end)
        for _, descendant in ipairs(_arCachedCharData.character:GetDescendants()) do
            if descendant:IsA("BallSocketConstraint")
            or (descendant:IsA("Attachment") and descendant.Name:find("RagdollAttachment")) then
                pcall(function() descendant:Destroy() end)
            end
        end
        if not _arIsBoosting then
            _arIsBoosting = true
            _arCachedCharData.humanoid.WalkSpeed = AR_BOOST_SPEED
        end
        if _arCachedCharData.humanoid.Health > 0 then
            _arCachedCharData.humanoid:ChangeState(Enum.HumanoidStateType.Running)
        end
        workspace.CurrentCamera.CameraSubject = _arCachedCharData.humanoid
        _arCachedCharData.root.Anchored = false
        for _, obj in ipairs(_arCachedCharData.character:GetDescendants()) do
            if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled = true end
        end
    end

    local function _arHeartbeatLoop()
        while State.antiRagdollEnabled do
            task.wait()
            if not _arCachedCharData.humanoid then break end
            local currentlyRagdolled = _arIsRagdolled()
            if currentlyRagdolled then
                _arForceExitRagdoll()
            elseif _arIsBoosting and not currentlyRagdolled then
                _arIsBoosting = false
                if _arCachedCharData.humanoid then
                    _arCachedCharData.humanoid.WalkSpeed = AR_DEFAULT_SPEED
                end
            end
        end
    end

    startAntiRagdoll = function()
        if _arMode == "v1" then return end
        if not _arCacheCharacterData() then return end
        _arMode = "v1"
        local camConn = RunService.RenderStepped:Connect(function()
            local cam = workspace.CurrentCamera
            if cam and _arCachedCharData.humanoid then
                cam.CameraSubject = _arCachedCharData.humanoid
            end
        end)
        table.insert(_arRagdollConnections, camConn)
        local respawnConn = LP.CharacterAdded:Connect(function()
            _arIsBoosting = false
            task.wait(0.5)
            _arCacheCharacterData()
        end)
        table.insert(_arRagdollConnections, respawnConn)
        task.spawn(_arHeartbeatLoop)
    end

    stopAntiRagdoll = function()
        _arMode = nil
        if _arIsBoosting and _arCachedCharData.humanoid then
            _arCachedCharData.humanoid.WalkSpeed = AR_DEFAULT_SPEED
        end
        _arIsBoosting = false
        _arDisconnectAll()
        _arCachedCharData = {}
        State.countdownActive = false
    end

    local _rtTimerActive = false
    local function getRagTimerLbl()
        local char = LP.Character; if not char then return nil end
        local head = char:FindFirstChild("Head"); if not head then return nil end
        local bb = head:FindFirstChild("AstaHubBB"); if not bb then return nil end
        return bb:FindFirstChild("RagdollTimerLbl")
    end
    local function startRagTimerGui()
        if _rtTimerActive then return end
        _rtTimerActive = true
        task.spawn(function()
            local t = 3.0
            while t >= 0.0 do
                local lbl = getRagTimerLbl()
                if lbl then
                    lbl.Text = string.format("%.1f", t)
                    lbl.TextColor3 = Color3.fromRGB(180,180,180)
                end
                task.wait(0.1)
                t = math.round((t - 0.1) * 10) / 10
            end
            local lbl = getRagTimerLbl()
            if lbl then lbl.Text = "STEAL!"; lbl.TextColor3 = Color3.fromRGB(220,220,220) end
            repeat task.wait(0.1) until (function()
                local c = LP.Character
                local hum = c and c:FindFirstChildOfClass("Humanoid")
                if not hum then return true end
                local st = hum:GetState()
                return st ~= Enum.HumanoidStateType.Physics and st ~= Enum.HumanoidStateType.Ragdoll and st ~= Enum.HumanoidStateType.FallingDown
            end)()
            local lbl2 = getRagTimerLbl()
            if lbl2 then lbl2.Text = "" end
            _rtTimerActive = false
        end)
    end
    local function startRagTimerDetection(char)
        RunService.Heartbeat:Connect(function()
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if not hum then return end
            local st = hum:GetState()
            if st == Enum.HumanoidStateType.Physics or st == Enum.HumanoidStateType.Ragdoll or st == Enum.HumanoidStateType.FallingDown then
                startRagTimerGui()
            end
        end)
    end

    -- ============================================================
    -- AUTO-STEAL
    -- ============================================================
    local function resetProgressBar()
        if progressFill then TweenService:Create(progressFill,TweenInfo.new(0.2),{Size=UDim2.new(0,0,1,0)}):Play() end
        if fillGlow then TweenService:Create(fillGlow,TweenInfo.new(0.2),{Size=UDim2.new(0,0,1,0)}):Play() end
    end

    local function isMyPlotByName(pn)
        local ct=tick()
        if Steal.plotCache[pn] and (ct-(Steal.plotCacheTime[pn] or 0))<PLOT_CACHE_DURATION then return Steal.plotCache[pn] end
        local plots=workspace:FindFirstChild("Plots")
        if not plots then Steal.plotCache[pn]=false; Steal.plotCacheTime[pn]=ct; return false end
        local plot=plots:FindFirstChild(pn)
        if not plot then Steal.plotCache[pn]=false; Steal.plotCacheTime[pn]=ct; return false end
        local sign=plot:FindFirstChild("PlotSign")
        if sign then local yb=sign:FindFirstChild("YourBase"); if yb and yb:IsA("BillboardGui") then local r=yb.Enabled==true; Steal.plotCache[pn]=r; Steal.plotCacheTime[pn]=ct; return r end end
        Steal.plotCache[pn]=false; Steal.plotCacheTime[pn]=ct; return false
    end

    local function findNearestPrompt()
        local c=LP.Character; if not c then return nil end
        local root=c:FindFirstChild("HumanoidRootPart"); if not root then return nil end
        local ct=tick()
        if ct-Steal.promptCacheTime<PROMPT_CACHE_REFRESH and #Steal.cachedPrompts>0 then
            local np,nd=nil,math.huge
            for _,data in ipairs(Steal.cachedPrompts) do
                if data.spawn then local dist=(data.spawn.Position-root.Position).Magnitude; if dist<=Steal.StealRadius and dist<nd then np=data.prompt; nd=dist end end
            end
            if np then return np end
        end
        Steal.cachedPrompts={}; Steal.promptCacheTime=ct
        local plots=workspace:FindFirstChild("Plots"); if not plots then return nil end
        local np,nd=nil,math.huge
        for _,plot in ipairs(plots:GetChildren()) do
            if isMyPlotByName(plot.Name) then continue end
            local pods=plot:FindFirstChild("AnimalPodiums"); if not pods then continue end
            for _,pod in ipairs(pods:GetChildren()) do
                pcall(function()
                    local base=pod:FindFirstChild("Base"); local sp=base and base:FindFirstChild("Spawn")
                    if sp then local att=sp:FindFirstChild("PromptAttachment"); if att then
                        for _,child in ipairs(att:GetChildren()) do
                            if child:IsA("ProximityPrompt") then
                                local dist=(sp.Position-root.Position).Magnitude
                                table.insert(Steal.cachedPrompts,{prompt=child,spawn=sp})
                                if dist<=Steal.StealRadius and dist<nd then np=child; nd=dist end
                                break
                            end
                        end
                    end end
                end)
            end
        end
        return np
    end

    local function executeSteal(prompt)
        local ct=tick(); if ct-State.lastStealTick<STEAL_COOLDOWN then return end
        if State.isStealing then return end
        if not Steal.Data[prompt] then
            Steal.Data[prompt]={hold={},trigger={},ready=true}
            pcall(function()
                if getconnections then
                    for _,c2 in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c2.Function then table.insert(Steal.Data[prompt].hold,c2.Function) end end
                    for _,c2 in ipairs(getconnections(prompt.Triggered)) do if c2.Function then table.insert(Steal.Data[prompt].trigger,c2.Function) end end
                else Steal.Data[prompt].useFallback=true end
            end)
        end
        local data=Steal.Data[prompt]; if not data.ready then return end
        data.ready=false; State.isStealing=true; State.stealStartTime=ct; State.lastStealTick=ct
        if Conns.progress then Conns.progress:Disconnect() end
        Conns.progress=RunService.Heartbeat:Connect(function()
            if not State.isStealing then Conns.progress:Disconnect(); return end
            local prog=math.clamp((tick()-State.stealStartTime)/Steal.StealDuration,0,1)
            if progressFill then TweenService:Create(progressFill,TweenInfo.new(0.05),{Size=UDim2.new(prog,0,1,0)}):Play() end
            if fillGlow then TweenService:Create(fillGlow,TweenInfo.new(0.05),{Size=UDim2.new(prog,0,1,0)}):Play() end
        end)
        task.spawn(function()
            local ok=false
            pcall(function()
                if not data.useFallback then
                    for _,fn in ipairs(data.hold) do task.spawn(fn) end
                    task.wait(Steal.StealDuration)
                    for _,fn in ipairs(data.trigger) do task.spawn(fn) end
                    ok=true
                end
            end)
            if not ok and fireproximityprompt then pcall(function() fireproximityprompt(prompt); ok=true end) end
            if not ok then pcall(function() prompt:InputHoldBegin(); task.wait(Steal.StealDuration); prompt:InputHoldEnd() end) end
            task.wait(Steal.StealDuration*0.3)
            if Conns.progress then Conns.progress:Disconnect() end
            resetProgressBar(); task.wait(0.05); data.ready=true; State.isStealing=false
        end)
    end

    startAutoSteal = function()
        if Conns.autoSteal then return end
        Conns.autoSteal=RunService.Heartbeat:Connect(function()
            if not Steal.AutoStealEnabled or State.isStealing then return end
            local p=findNearestPrompt(); if p then executeSteal(p) end
        end)
    end
    stopAutoSteal = function()
        if Conns.autoSteal then Conns.autoSteal:Disconnect(); Conns.autoSteal=nil end
        if Conns.progress then Conns.progress:Disconnect(); Conns.progress=nil end
        State.isStealing=false; State.lastStealTick=0
        Steal.plotCache={}; Steal.plotCacheTime={}; Steal.cachedPrompts={}
        resetProgressBar()
    end

    -- ============================================================
    -- WEBHOOK MONITOR REMOVED (no Discord messages)
    -- ============================================================

    -- ============================================================
    -- MODULES
    -- ============================================================
    _G._NukeOn=false; _G._NukeConns={}; _G._NukeThreads={}
    _G._nukeStart = function()
        if _G._NukeOn then return end; _G._NukeOn=true
        local Lighting=game:GetService("Lighting"); local MaterialService=game:GetService("MaterialService")
        local XMin,XMax=-560,-240
        local ClothingClasses={"Shirt","Pants","ShirtGraphic","Accessory","Hat","HairAccessory","FaceAccessory","NeckAccessory","ShoulderAccessory","FrontAccessory","BackAccessory","WaistAccessory"}
        local BASE_NAMES={"baseplate","spawnlocation","spawn location","spawn"}
        local function SafeDestroy(obj) if obj.Name=="Overhead" then return end pcall(function() obj:Destroy() end) end
        local function IsClothing(obj) for _,c in ipairs(ClothingClasses) do if obj:IsA(c) then return true end end return false end
        local function IsCharacterPart(obj) for _,plr in ipairs(Players:GetPlayers()) do if plr.Character and obj:IsDescendantOf(plr.Character) then return true end end return false end
        local function IsOutOfRange(obj) if obj:IsA("BasePart") then local x=obj.Position.X; return x<XMin or x>XMax end return false end
        local function IsBase(obj) if not obj:IsA("BasePart") then return false end local nl=obj.Name:lower(); for _,n in ipairs(BASE_NAMES) do if nl:find(n,1,true) then return true end end return false end
        local function IsInBase(obj) local p=obj.Parent; while p and p~=workspace do if IsBase(p) then return true end p=p.Parent end return false end
        local function MakeTransparent(obj) pcall(function() if IsBase(obj) and not IsCharacterPart(obj) then obj.Transparency=1; obj.CastShadow=false end end) end
        local function StripObject(obj) pcall(function() if obj:IsA("Texture") or obj:IsA("Decal") or obj:IsA("SpecialMesh") then SafeDestroy(obj) elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then pcall(function() obj.Enabled=false end); SafeDestroy(obj) elseif obj:IsA("SurfaceAppearance") then SafeDestroy(obj) elseif obj:IsA("BasePart") then obj.CastShadow=false; obj.Material=Enum.Material.Plastic; obj.MaterialVariant=""; obj.Reflectance=0 end end) end
        local function CleanObject(obj) pcall(function() if obj:IsA("SurfaceAppearance") then SafeDestroy(obj) elseif obj:IsA("Decal") or obj:IsA("Texture") then if not (obj.Name=="face" and obj.Parent and obj.Parent.Name=="Head") then SafeDestroy(obj) end elseif obj:IsA("SpecialMesh") then obj.TextureId="" elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") then SafeDestroy(obj) elseif obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then SafeDestroy(obj) elseif obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") or obj:IsA("Explosion") then SafeDestroy(obj) elseif obj:IsA("Animation") or obj:IsA("AnimationController") then SafeDestroy(obj) elseif obj:IsA("BasePart") then obj.CastShadow=false; obj.Material=Enum.Material.Plastic; obj.MaterialVariant=""; obj.Reflectance=0 end end) end
        local function ApplyGreySky() pcall(function() for _,obj in ipairs(Lighting:GetChildren()) do if obj:IsA("Sky") then obj:Destroy() end end; local sky=Instance.new("Sky"); sky.SkyboxBk=""; sky.SkyboxDn=""; sky.SkyboxFt=""; sky.SkyboxLf=""; sky.SkyboxRt=""; sky.SkyboxUp=""; sky.CelestialBodiesShown=false; sky.Name="_VezyNukeSky"; sky.Parent=Lighting end) end
        local function OptimizeLighting() Lighting.GlobalShadows=false; Lighting.FogEnd=9e9; Lighting.FogStart=9e9; Lighting.EnvironmentDiffuseScale=0; Lighting.EnvironmentSpecularScale=0; Lighting.Brightness=1.5; Lighting.Ambient=Color3.fromRGB(60,60,60); for _,v in ipairs(Lighting:GetChildren()) do if v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("ColorCorrectionEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Atmosphere") or v:IsA("Clouds") then v:Destroy() end end; ApplyGreySky() end
        local function ApplyTerrain() pcall(function() local T=workspace.Terrain; T.Decoration=false; T.WaterWaveSize=0; T.WaterWaveSpeed=0; T.WaterReflectance=0; T.WaterTransparency=1 end) end
        local function OptimizeCharacter(char) if not char then return end task.spawn(function() task.wait(0.3); if not _G._NukeOn then return end; for _,obj in ipairs(char:GetDescendants()) do if IsClothing(obj) then SafeDestroy(obj) else CleanObject(obj) end end end) end
        pcall(function() settings().Rendering.QualityLevel=Enum.QualityLevel.Level01; settings().Rendering.MeshPartDetailLevel=Enum.MeshPartDetailLevel.Level01 end)
        pcall(function() if setfpscap then setfpscap(999) end end)
        table.insert(_G._NukeThreads,task.spawn(function() if not game:IsLoaded() then game.Loaded:Wait() end; OptimizeLighting(); ApplyTerrain(); for _,obj in ipairs(workspace:GetDescendants()) do if not _G._NukeOn then return end; if IsBase(obj) then MakeTransparent(obj) elseif IsClothing(obj) then SafeDestroy(obj) elseif IsInBase(obj) then elseif IsCharacterPart(obj) then elseif IsOutOfRange(obj) then SafeDestroy(obj) else CleanObject(obj); StripObject(obj) end end; for _,obj in ipairs(workspace:GetDescendants()) do MakeTransparent(obj) end end))
        table.insert(_G._NukeConns,workspace.DescendantAdded:Connect(function(obj) if not _G._NukeOn then return end; task.defer(function() if not _G._NukeOn then return end; if IsBase(obj) then MakeTransparent(obj); return end; if IsClothing(obj) then SafeDestroy(obj) elseif IsInBase(obj) then elseif IsCharacterPart(obj) then elseif IsOutOfRange(obj) then SafeDestroy(obj) else CleanObject(obj); StripObject(obj) end end) end))
        table.insert(_G._NukeConns,Lighting.DescendantAdded:Connect(function(obj) if not _G._NukeOn then return end; if obj:IsA("Atmosphere") or obj:IsA("Clouds") or obj:IsA("PostEffect") then SafeDestroy(obj) end end))
        table.insert(_G._NukeConns,MaterialService.DescendantAdded:Connect(function(obj) if not _G._NukeOn then return end; SafeDestroy(obj) end))
        for _,plr in ipairs(Players:GetPlayers()) do OptimizeCharacter(plr.Character); table.insert(_G._NukeConns,plr.CharacterAdded:Connect(OptimizeCharacter)) end
        table.insert(_G._NukeConns,Players.PlayerAdded:Connect(function(plr) table.insert(_G._NukeConns,plr.CharacterAdded:Connect(OptimizeCharacter)) end))
        table.insert(_G._NukeThreads,task.spawn(function() while _G._NukeOn do task.wait(15); pcall(function() collectgarbage("collect") end) end end))
    end
    _G._nukeStop = function() _G._NukeOn=false; for _,c in ipairs(_G._NukeConns) do pcall(function() c:Disconnect() end) end; _G._NukeConns={}; _G._NukeThreads={} end

    _G._NoCamOn=false; _G._NoCamConn=nil; _G._NoCamParts={}
    _G._noCamStart = function() if _G._NoCamOn then return end; _G._NoCamOn=true; local function apply(obj) if obj:IsA("BasePart") and not obj:IsDescendantOf(LP.Character) then if _G._NoCamParts[obj]==nil then _G._NoCamParts[obj]=obj.CanCollide end end end; for _,obj in ipairs(workspace:GetDescendants()) do apply(obj) end; _G._NoCamConn = RunService.RenderStepped:Connect(function() if not _G._NoCamOn then return end; local cam=workspace.CurrentCamera; if not cam then return end; for p,_ in pairs(_G._NoCamParts) do if p and p.Parent then pcall(function() local dist=(cam.CFrame.Position-p.Position).Magnitude; if dist<8 then p.LocalTransparencyModifier=1 else p.LocalTransparencyModifier=0 end end) end end end) end
    _G._noCamStop = function() _G._NoCamOn=false; if _G._NoCamConn then _G._NoCamConn:Disconnect(); _G._NoCamConn=nil end; for p,_ in pairs(_G._NoCamParts) do pcall(function() if p and p.Parent then p.LocalTransparencyModifier=0 end end) end; _G._NoCamParts={} end

    _G._VezyFontMyfont=nil; _G._VezyFontBadfont=nil; _G._VezyFontConn=nil; _G._VezyFontEnabled=false; _G._VezyFontOriginals={}
    _G._fontDontTouch = function(this) if this:IsA("TextLabel") or this:IsA("TextButton") or this:IsA("TextBox") then if this.TextStrokeTransparency~=1 then return false end; local cur=tostring(this.FontFace); return cur==_G._VezyFontBadfont or string.find(cur,"BuilderIcons") end; return true end
    _G._fontChangeIt = function(txt) if (txt:IsA("TextLabel") or txt:IsA("TextButton") or txt:IsA("TextBox")) and not _G._fontDontTouch(txt) then if not _G._VezyFontOriginals[txt] then _G._VezyFontOriginals[txt]=txt.FontFace end; pcall(function() txt.FontFace=_G._VezyFontMyfont end) end end
    _G._fontSetup = function() if _G._VezyFontMyfont then return true end; local ok=pcall(function() local httpsvc=game:GetService("HttpService"); if isfile and writefile and getcustomasset then if not isfile("starborn.ttf") then writefile("starborn.ttf",game:HttpGet("https://granny.anondrop.net/uploads/6c2505542959f371/Starborn.ttf")) end; writefile("starborn.json",httpsvc:JSONEncode({name="Starborn",faces={{name="Regular",weight=400,style="normal",assetId=getcustomasset("starborn.ttf")}}})); _G._VezyFontMyfont=Font.new(getcustomasset("starborn.json")); _G._VezyFontBadfont=tostring(Font.new("rbxasset://LuaPackages/Packages/_Index/BuilderIcons/BuilderIcons/BuilderIcons.json")) end end); return ok and _G._VezyFontMyfont~=nil end
    _G._customFontStart = function() if _G._VezyFontEnabled then return end; if not _G._fontSetup() then return end; _G._VezyFontEnabled=true; for _,v in pairs(game:GetDescendants()) do _G._fontChangeIt(v) end; _G._VezyFontConn=game.DescendantAdded:Connect(function(obj) if _G._VezyFontEnabled then _G._fontChangeIt(obj) end end) end
    _G._customFontStop = function() _G._VezyFontEnabled=false; if _G._VezyFontConn then _G._VezyFontConn:Disconnect(); _G._VezyFontConn=nil end; for obj,origFont in pairs(_G._VezyFontOriginals) do pcall(function() if obj and obj.Parent then obj.FontFace=origFont end end) end; _G._VezyFontOriginals={} end

    _G._RemoveAccOn=false; _G._RemoveAccConn=nil; _G._removedAccessories={}
    _G._removeAccDo = function() if not _G._RemoveAccOn then return end; local char=LP.Character; if not char then return end; for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Accessory") or obj:IsA("Hat") then if not _G._removedAccessories[obj] then _G._removedAccessories[obj]=true; pcall(function() obj:Destroy() end) end end end end
    _G._removeAccStart = function() if _G._RemoveAccOn then return end; _G._RemoveAccOn=true; _G._removeAccDo(); _G._RemoveAccConn=LP.CharacterAdded:Connect(function() task.wait(0.5); if _G._RemoveAccOn then _G._removeAccDo() end end) end
    _G._removeAccStop = function() _G._RemoveAccOn=false; if _G._RemoveAccConn then _G._RemoveAccConn:Disconnect(); _G._RemoveAccConn=nil end; _G._removedAccessories={} end

    -- ============================================================
    -- CHARACTER SETUP
    -- ============================================================
    local function setupChar(char)
        task.wait(0.1)
        h=char:WaitForChild("Humanoid",5)
        hrp=char:WaitForChild("HumanoidRootPart",5)
        if not h or not hrp then return end
        local head=char:FindFirstChild("Head")
        if head then
            local oldBB=head:FindFirstChild("AstaHubBB"); if oldBB then oldBB:Destroy() end
            local bb=Instance.new("BillboardGui", head); bb.Name="AstaHubBB"; bb.Size=UDim2.new(0,180,0,70); bb.StudsOffset=Vector3.new(0,3,0); bb.AlwaysOnTop=true
            local list=Instance.new("UIListLayout",bb); list.FillDirection=Enum.FillDirection.Vertical; list.SortOrder=Enum.SortOrder.LayoutOrder; list.VerticalAlignment=Enum.VerticalAlignment.Center; list.Padding=UDim.new(0,2)
            local speedBillLbl=Instance.new("TextLabel",bb); speedBillLbl.Name="SpeedBillLbl"; speedBillLbl.Size=UDim2.new(1,0,0,24); speedBillLbl.BackgroundTransparency=1; speedBillLbl.Text="0.0"; speedBillLbl.TextColor3=Color3.fromRGB(180,180,180); speedBillLbl.Font=Enum.Font.GothamBlack; speedBillLbl.TextScaled=true; speedBillLbl.TextStrokeTransparency=0.1; speedBillLbl.TextStrokeColor3=Color3.new(0,0,0); speedBillLbl.LayoutOrder=1
            local ragTimerLbl=Instance.new("TextLabel",bb); ragTimerLbl.Name="RagdollTimerLbl"; ragTimerLbl.Size=UDim2.new(1,0,0,30); ragTimerLbl.BackgroundTransparency=1; ragTimerLbl.Text=""; ragTimerLbl.TextColor3=Color3.fromRGB(255,60,60); ragTimerLbl.Font=Enum.Font.GothamBlack; ragTimerLbl.TextScaled=true; ragTimerLbl.TextStrokeTransparency=0.1; ragTimerLbl.TextStrokeColor3=Color3.new(0,0,0); ragTimerLbl.LayoutOrder=3
        end
        stopAntiRagdoll()
        Steal.Data={}
        _rtTimerActive = false
        local _rtLbl = getRagTimerLbl and getRagTimerLbl()
        if _rtLbl then _rtLbl.Text = "" end
        task.spawn(function() startRagTimerDetection(char) end)
        if State.antiRagdollEnabled then task.wait(0.5); startAntiRagdoll() end
        if State.medusaCounterEnabled then setupMedusaCounter(char) end
        if State.batAimbotToggled then stopBatAimbot(); task.wait(0.2); pcall(startBatAimbot) end
        if State.batCounterEnabled then task.wait(0.3); startBatCounter() end
        if State.tryardAnimEnabled then saveOriginalTryardAnims(char); applyTryardAnimPack(char) end
    end
    LP.CharacterAdded:Connect(setupChar)
    if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

    -- ============================================================
    -- INSTANT RESET
    -- ============================================================
    local cursedResetRemote = nil
    local resetCooldown = false
    local CURSED_RESET_GUID = "f888ee6e-c86d-46e1-93d7-0639d6635d42"

    local function findResetRemote()
        for _, desc in ipairs(game:GetDescendants()) do
            if desc:IsA("RemoteEvent") and desc.Name:sub(1,3) == "RE/" then
                cursedResetRemote = desc; return true
            end
        end
        return false
    end
    pcall(function()
        if hookfunction and newcclosure then
            local oldFire
            oldFire = hookfunction(Instance.new("RemoteEvent").FireServer, newcclosure(function(self,...)
                if not cursedResetRemote and typeof(self)=="Instance" and self:IsA("RemoteEvent") and self.Name:sub(1,3)=="RE/" then
                    cursedResetRemote=self
                end
                return oldFire(self,...)
            end))
        end
    end)
    task.spawn(function() task.wait(1); if not cursedResetRemote then findResetRemote() end end)

    performInstantReset = function()
        if resetCooldown then return end; resetCooldown = true
        if not cursedResetRemote then findResetRemote() end
        if not cursedResetRemote then
            for _, desc in ipairs(game:GetDescendants()) do
                if desc:IsA("RemoteEvent") and desc.Name:sub(1,3)=="RE/" then cursedResetRemote=desc; break end
            end
        end
        if cursedResetRemote then
            local character = LP.Character
            local humanoid = character and character:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health <= 0 then
                pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID, LP, "balloon") end)
                task.delay(0.3, function() resetCooldown=false end); return
            end
            local resetDetected = false; local conns = {}
            if humanoid then
                table.insert(conns, humanoid.Died:Connect(function() resetDetected=true end))
                table.insert(conns, humanoid:GetPropertyChangedSignal("Health"):Connect(function() if humanoid.Health<=0 then resetDetected=true end end))
            end
            if character then
                table.insert(conns, character.AncestryChanged:Connect(function(_,parent) if not parent then resetDetected=true end end))
            end
            task.spawn(function()
                for i=1,50 do
                    if resetDetected then break end
                    pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID, LP, "balloon") end)
                    task.wait()
                end
                for _, conn in ipairs(conns) do pcall(function() conn:Disconnect() end) end
                task.delay(0.3, function() resetCooldown=false end)
            end)
        else
            local char = LP.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum then hum.Health=0 end
            task.delay(0.3, function() resetCooldown=false end)
        end
    end

    -- ============================================================
    -- AUTO RESET ON MEDUSA
    -- ============================================================
    State.instantResetOnMedusa = State.instantResetOnMedusa ~= false
    State._medusaResetCooldown = false
    State._lastResetTime = 0

    local function checkMedusaForInstantReset()
        if not State.instantResetOnMedusa then return end
        if State._medusaResetCooldown then return end
        if tick()-State._lastResetTime < 3 then return end
        local char = LP.Character
        if not char then return end
        for _, part in ipairs(char:GetDescendants()) do
            if part:IsA("BasePart") and part.Anchored and part.Transparency == 1 then
                State._medusaResetCooldown = true
                State._lastResetTime = tick()
                performInstantReset()
                task.delay(2, function() State._medusaResetCooldown = false end)
                break
            end
        end
    end

    local function onAnchorChanged(part)
        return part:GetPropertyChangedSignal("Anchored"):Connect(function()
            if part.Anchored and part.Transparency==1 then
                checkMedusaForInstantReset()
            end
        end)
    end

    local function setupMedusaResetDetection(char)
        if not char then return end
        for _, part in ipairs(char:GetDescendants()) do
            if part:IsA("BasePart") then onAnchorChanged(part) end
        end
        char.DescendantAdded:Connect(function(part)
            if part:IsA("BasePart") then onAnchorChanged(part) end
        end)
    end

    LP.CharacterAdded:Connect(setupMedusaResetDetection)
    if LP.Character then task.spawn(function() setupMedusaResetDetection(LP.Character) end) end

    -- ============================================================
    -- TRACERS
    -- ============================================================
    local tracerLines = {}
    State.tracersEnabled = State.tracersEnabled or false

    clearTracers = function()
        for _, line in pairs(tracerLines) do pcall(function() line:Remove() end) end
        tracerLines = {}
    end

    local function updateTracers()
        if not State.tracersEnabled then clearTracers(); return end
        local cam = workspace.CurrentCamera
        local char = LP.Character
        local myHRP = char and char:FindFirstChild("HumanoidRootPart")
        if not myHRP or not cam then clearTracers(); return end
        local validKeys = {}
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LP and plr.Character then
                local tHRP = plr.Character:FindFirstChild("HumanoidRootPart")
                if tHRP then validKeys[tostring(plr.UserId)] = {plr=plr, hrp=tHRP} end
            end
        end
        for key, line in pairs(tracerLines) do
            if not validKeys[key] then pcall(function() line:Remove() end); tracerLines[key]=nil end
        end
        local screenSize = cam.ViewportSize
        local fromX = screenSize.X / 2; local fromY = screenSize.Y
        for key, data in pairs(validKeys) do
            local tPos, onScreen = cam:WorldToViewportPoint(data.hrp.Position)
            if onScreen then
                local line = tracerLines[key]
                if not line and Drawing then
                    line = Drawing.new("Line"); line.Thickness=2
                    line.Color=Color3.fromRGB(255,50,50); line.Transparency=0.3
                    line.Visible=true; tracerLines[key]=line
                end
                if line then line.From=Vector2.new(fromX,fromY); line.To=Vector2.new(tPos.X,tPos.Y); line.Visible=true end
            else
                local line = tracerLines[key]; if line then line.Visible=false end
            end
        end
    end

    RunService.Heartbeat:Connect(updateTracers)

    -- ============================================================
    -- KEYBOARD INPUT HANDLER (REMOVED F and G hardcoded keybinds)
    -- ============================================================
    UIS.InputBegan:Connect(function(inp, gp)
        if gp then return end
        if inp.UserInputType ~= Enum.UserInputType.Keyboard then return end
        -- Only instant reset keybind remains (configurable via UI)
        if inp.KeyCode == Keys.instantReset and Keys.instantReset ~= Enum.KeyCode.Unknown then
            performInstantReset()
        end
        -- F and G are no longer hardcoded – they are only toggled via UI switches
    end)

    -- ============================================================
    -- RUNTIME LOOPS
    -- ============================================================
    RunService.Stepped:Connect(function()
        for _,p in ipairs(Players:GetPlayers()) do if p~=LP and p.Character then for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end end end
    end)

    -- ============================================================
    -- SPEED ENGINE (LinearVelocity / Plane mode - undetected)
    -- ============================================================
    local _lvAtt, _lv
    local function destroySpeedLV()
        pcall(function() if _lv and _lv.Parent then _lv:Destroy() end end)
        pcall(function() if _lvAtt and _lvAtt.Parent then _lvAtt:Destroy() end end)
        _lv=nil; _lvAtt=nil
    end
    local function ensureSpeedLV(root)
        if _lv and _lv.Parent==root and _lvAtt and _lvAtt.Parent==root then return _lv end
        destroySpeedLV()
        _lvAtt=Instance.new("Attachment"); _lvAtt.Name="AstaSpeedAtt"; _lvAtt.Parent=root
        _lv=Instance.new("LinearVelocity")
        _lv.Name="AstaSpeedLV"
        _lv.Attachment0=_lvAtt
        _lv.VelocityConstraintMode=Enum.VelocityConstraintMode.Plane
        _lv.PrimaryTangentAxis=Vector3.new(1,0,0)
        _lv.SecondaryTangentAxis=Vector3.new(0,0,1)
        _lv.MaxForce=math.huge
        _lv.PlaneVelocity=Vector2.zero
        _lv.RelativeTo=Enum.ActuatorRelativeTo.World
        _lv.Parent=root
        return _lv
    end

    RunService.RenderStepped:Connect(function()
        if not (h and hrp) then destroySpeedLV(); return end
        if State._tpInProgress then destroySpeedLV(); return end
        if not State.batAimbotToggled and not State.autoLeftEnabled and not State.autoRightEnabled then
            local md=h.MoveDirection
            local spd
            if State.laggerMode==1 then spd=State.laggerSpeed
            elseif State.laggerMode==2 then spd=State.laggerCarrySpeed
            else spd=State.speedToggled and State.carrySpeed or State.normalSpeed end
            local lv=ensureSpeedLV(hrp)
            if md.Magnitude>0 then
                State.lastMoveDir=md
                local flat=Vector3.new(md.X,0,md.Z)
                if flat.Magnitude>0.01 then
                    flat=flat.Unit
                    lv.PlaneVelocity=Vector2.new(flat.X*spd, flat.Z*spd)
                else
                    lv.PlaneVelocity=Vector2.zero
                end
            else
                lv.PlaneVelocity=Vector2.zero
            end
        else
            destroySpeedLV()
        end
        pcall(function()
            local head2=LP.Character and LP.Character:FindFirstChild("Head")
            if head2 then
                local bb2=head2:FindFirstChild("AstaHubBB")
                local sl=bb2 and bb2:FindFirstChild("SpeedBillLbl")
                if sl then sl.Text=string.format("%.1f",Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude) end
            end
        end)
    end)

    UIS.InputBegan:Connect(function(inp,gp)
        if gp then return end
        local isGp=inp.UserInputType==Enum.UserInputType.Gamepad1 or inp.UserInputType==Enum.UserInputType.Gamepad2 or inp.UserInputType==Enum.UserInputType.Gamepad3 or inp.UserInputType==Enum.UserInputType.Gamepad4
        local isKb=inp.UserInputType==Enum.UserInputType.Keyboard
        if not isGp and not isKb then return end
        local kc=inp.KeyCode; if kc==Enum.KeyCode.Unknown then return end
        if kc==Keys.speed then toggleSpeed()
        elseif kc==Keys.autoLeft then
            State.autoLeftEnabled=not State.autoLeftEnabled
            if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(State.autoLeftEnabled) end
            if State.autoLeftEnabled and State.batAimbotToggled then State.batAimbotToggled=false; stopBatAimbot(); if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end end
            if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end
            requestSave()
        elseif kc==Keys.autoRight then
            State.autoRightEnabled=not State.autoRightEnabled
            if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(State.autoRightEnabled) end
            if State.autoRightEnabled and State.batAimbotToggled then State.batAimbotToggled=false; stopBatAimbot(); if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(false) end end
            if State.autoRightEnabled then startAutoRight() else stopAutoRight() end
            requestSave()
        elseif kc==Keys.drop then if not dropActive then pcall(runDrop) end
        elseif kc==Keys.lagger then toggleLaggerMode()
        elseif kc==Keys.tpDown then if runTPDown then task.spawn(runTPDown) end
        elseif kc==Keys.aimbot then
            State.batAimbotToggled=not State.batAimbotToggled
            if State.batAimbotToggled then
                if State.autoLeftEnabled then State.autoLeftEnabled=false; stopAutoLeft(); if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(false) end end
                if State.autoRightEnabled then State.autoRightEnabled=false; stopAutoRight(); if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(false) end end
                pcall(startBatAimbot)
            else stopBatAimbot() end
            if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(State.batAimbotToggled) end
            requestSave()
        elseif kc==Keys.tpBat then
            toggleTPBat()
            requestSave()
        end
    end)

    _G._VezyFOV = _G._VezyFOV or 70
    _G._VezyFOVPropConn = nil
    local function _attachFOVLock(cam)
        if not cam then return end
        if _G._VezyFOVPropConn then pcall(function() _G._VezyFOVPropConn:Disconnect() end) end
        pcall(function() cam.FieldOfView = _G._VezyFOV or 70 end)
        _G._VezyFOVPropConn = cam:GetPropertyChangedSignal("FieldOfView"):Connect(function()
            local target = _G._VezyFOV or 70
            if not State.stretchedResEnabled and cam.FieldOfView ~= target then pcall(function() cam.FieldOfView = target end) end
        end)
    end
    _attachFOVLock(workspace.CurrentCamera)
    workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function() task.wait(); _attachFOVLock(workspace.CurrentCamera) end)
    LP.CharacterAdded:Connect(function() task.wait(0.3); _attachFOVLock(workspace.CurrentCamera) end)
    RunService.RenderStepped:Connect(function()
        local cam = workspace.CurrentCamera
        if not cam then return end
        local target = _G._VezyFOV or 70
        if not State.stretchedResEnabled and cam.FieldOfView ~= target then pcall(function() cam.FieldOfView = target end) end
    end)

    -- ============================================================
    -- MINI CLOVER BUTTON
    -- ============================================================
    local cloverBtn = Instance.new("ImageButton", gui)
    cloverBtn.Name = "AstaHubClover"
    cloverBtn.Size = UDim2.new(0,44,0,44)
    cloverBtn.Position = UDim2.new(0,20,0,200)
    cloverBtn.BackgroundTransparency = 1
    cloverBtn.Image = "rbxassetid://81596065711733"
    cloverBtn.ScaleType = Enum.ScaleType.Crop
    cloverBtn.BorderSizePixel = 0
    cloverBtn.ZIndex = 25
    cloverBtn.Visible = true
    mkCorner(cloverBtn,22)
    mkStroke(cloverBtn, Color3.fromRGB(200,200,200), 1.5)

    do
        local dragStart,startPos,dragging = nil,nil,false
        local saveDebounce = nil
        cloverBtn.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                dragStart = input.Position
                startPos = cloverBtn.Position
                input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
            end
        end)
        cloverBtn.InputChanged:Connect(function(input)
            if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local delta = input.Position - dragStart
                cloverBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end)
        cloverBtn.InputEnded:Connect(function()
            if dragging then
                dragging = false
                if saveDebounce then task.cancel(saveDebounce) end
                saveDebounce = task.delay(0.2, function()
                    pcall(requestSave)
                    saveDebounce = nil
                end)
            end
        end)
    end

    cloverBtn.MouseButton1Click:Connect(function()
        State.guiVisible = not State.guiVisible
        mainOuter.Visible = State.guiVisible
        if _G.AstaHubQAHide then pcall(_G.AstaHubQAHide, not State.guiVisible) end
        requestSave()
    end)

    cloverBtn.MouseEnter:Connect(function() TweenService:Create(cloverBtn, TweenInfo.new(0.12), {BackgroundColor3=Color3.fromRGB(20,32,24)}):Play() end)
    cloverBtn.MouseLeave:Connect(function() TweenService:Create(cloverBtn, TweenInfo.new(0.12), {BackgroundColor3=Color3.fromRGB(14,24,18)}):Play() end)

    -- ============================================================
    -- SAVE / LOAD (ROBUST VERSION)
    -- ============================================================
    saveConfig = function()
        local success = false
        pcall(function()
            if _isfile(CONFIG_FILE) then
                local oldRaw = _readfile(CONFIG_FILE)
                if oldRaw and oldRaw ~= "" then
                    pcall(function() _writefile(CONFIG_BACKUP, oldRaw) end)
                end
            end
            
            local btnPositions = {}
            for key, wrapper in pairs(stackWrappers) do
                if wrapper and wrapper.Position then
                    btnPositions[key] = { X = wrapper.Position.X.Offset, Y = wrapper.Position.Y.Offset }
                end
            end
            local cloverPos = cloverBtn and cloverBtn.Position and { X = cloverBtn.Position.X.Offset, Y = cloverBtn.Position.Y.Offset } or nil
            local cfg = {
                version = CONFIG_VERSION,
                normalSpeed = State.normalSpeed,
                carrySpeed = State.carrySpeed,
                laggerSpeed = State.laggerSpeed,
                laggerCarrySpeed = State.laggerCarrySpeed,
                speedToggled = State.speedToggled,
                laggerMode = State.laggerMode,
                stealRadius = Steal.StealRadius,
                stealDuration = Steal.StealDuration,
                uiScale = uiScaleObj and uiScaleObj.Scale or 1.0,
                stackButtonsHidden = State.stackButtonsHidden,
                stackButtonsLocked = State.stackButtonsLocked,
                infJump = State.infJumpEnabled,
                skyJumpMode = State.skyJumpMode,
                antiRagdoll = State.antiRagdollEnabled,
                medusaCounter = State.medusaCounterEnabled,
                batCounter = State.batCounterEnabled,
                autoStealEnabled = Steal.AutoStealEnabled,
                autoSwing = State.autoSwingEnabled,
                batAimbot = State.batAimbotToggled,
                antiLagEnabled = State.antiLagEnabled,
                stretchedResEnabled = State.stretchedResEnabled,
                stretchFOV = State.stretchFOV,
                normalFOV = _G._VezyFOV or 70,
                activeSky = State.activeSky,
                bgSelectedIndex = State.bgSelectedIndex or 1,
                speedKey = Keys.speed and Keys.speed.Name or "Unknown",
                laggerKey = Keys.lagger and Keys.lagger.Name or "Unknown",
                aimbotKey = Keys.aimbot and Keys.aimbot.Name or "Unknown",
                autoLeftKey = Keys.autoLeft and Keys.autoLeft.Name or "Unknown",
                autoRightKey = Keys.autoRight and Keys.autoRight.Name or "Unknown",
                dropKey = Keys.drop and Keys.drop.Name or "Unknown",
                tpDownKey = Keys.tpDown and Keys.tpDown.Name or "Unknown",
                guiHideKey = Keys.guiHide and Keys.guiHide.Name or "Unknown",
                instantResetKey = Keys.instantReset and Keys.instantReset.Name or "Unknown",
                tpBatEnabled = tpBatEnabled,
                tpBatKey = Keys.tpBat and Keys.tpBat.Name or "Unknown",
                nukeOptimizer = State.nukeOpt,
                removeAccessories = State.removeAcc,
                tryardAnimEnabled = State.tryardAnimEnabled,
                introEnabled = State.introEnabled,
                guiVisible = State.guiVisible,
                buttonPositions = btnPositions,
                cloverPosition = cloverPos,
                autoTPEnabled = State.autoTPEnabled,
                autoTPHeight = State.autoTPHeight,
                dropType = currentDropType,
                tracersEnabled = State.tracersEnabled,
                instantResetOnMedusa = State.instantResetOnMedusa,
                aimbotMode = State.aimbotMode,
            }
            local encoded = HttpService:JSONEncode(cfg)
            _writefile(CONFIG_FILE, encoded)
            local verify = _readfile(CONFIG_FILE)
            if verify == encoded then success = true end
        end)
        if not success then
            pcall(_G._VezyFlashSave, false)
            warn("[Asta Hub] Config save FAILED!")
        else
            pcall(_G._VezyFlashSave, true)
        end
        return success
    end

    loadConfig = function()
        local raw = nil
        if _isfile(CONFIG_FILE) then
            raw = _readfile(CONFIG_FILE)
        end
        if not raw or raw == "" then
            if _isfile(CONFIG_BACKUP) then
                raw = _readfile(CONFIG_BACKUP)
                if raw and raw ~= "" then
                    print("[Asta Hub] Loaded config from backup")
                end
            end
        end
        if not raw or raw == "" then
            print("[Asta Hub] No valid config file found, using defaults")
            return false
        end
        
        local ok, decErr = pcall(HttpService.JSONDecode, HttpService, raw)
        if not ok or not decErr then
            pcall(function() _delfile(CONFIG_FILE) end)
            pcall(function() _delfile(CONFIG_BACKUP) end)
            warn("[Asta Hub] Corrupt config deleted, using defaults")
            return false
        end

        local function applyNumber(key, targetVar, uiBox)
            if decErr[key] then
                targetVar = decErr[key]
                if uiBox and uiBox.Text then uiBox.Text = tostring(decErr[key]) end
            end
            return targetVar
        end

        State.normalSpeed = applyNumber("normalSpeed", State.normalSpeed, normalBox)
        State.carrySpeed = applyNumber("carrySpeed", State.carrySpeed, carryBox)
        State.laggerSpeed = applyNumber("laggerSpeed", State.laggerSpeed, laggerBox)
        State.laggerCarrySpeed = applyNumber("laggerCarrySpeed", State.laggerCarrySpeed, laggerCarryBox)
        Steal.StealRadius = applyNumber("stealRadius", Steal.StealRadius, stealRadBox)
        Steal.StealDuration = applyNumber("stealDuration", Steal.StealDuration, stealDurBox)
        if decErr.uiScale and uiScaleObj then
            uiScaleObj.Scale = decErr.uiScale
            if uiScaleBox then uiScaleBox.Text = tostring(decErr.uiScale) end
        end
        if decErr.normalFOV then
            _G._VezyFOV = decErr.normalFOV
            pcall(function() workspace.CurrentCamera.FieldOfView = _G._VezyFOV end)
        end
        if decErr.autoTPEnabled ~= nil then State.autoTPEnabled = decErr.autoTPEnabled end
        if decErr.autoTPHeight then
            State.autoTPHeight = decErr.autoTPHeight
            if autoTPHeightBox then autoTPHeightBox.Text = tostring(State.autoTPHeight) end
        end

        if decErr.dropType and (decErr.dropType == DROP_TYPES.STAND or decErr.dropType == DROP_TYPES.JUMP) then
            currentDropType = decErr.dropType
            if standDropBtn and jumpDropBtn then
                if currentDropType == DROP_TYPES.STAND then
                    standDropBtn.BackgroundColor3 = C.accent
                    standDropBtn.TextColor3 = Color3.fromRGB(0,20,8)
                    jumpDropBtn.BackgroundColor3 = C.inputBg
                    jumpDropBtn.TextColor3 = C.inputTxt
                else
                    jumpDropBtn.BackgroundColor3 = C.accent
                    jumpDropBtn.TextColor3 = Color3.fromRGB(0,20,8)
                    standDropBtn.BackgroundColor3 = C.inputBg
                    standDropBtn.TextColor3 = C.inputTxt
                end
            end
        end

        local bools = {
            stackButtonsHidden="stackButtonsHidden", stackButtonsLocked="stackButtonsLocked",
            infJump="infJumpEnabled", antiRagdoll="antiRagdollEnabled",
            medusaCounter="medusaCounterEnabled", batCounter="batCounterEnabled",
            autoStealEnabled="autoStealEnabled", autoSwing="autoSwingEnabled",
            batAimbot="batAimbotToggled", antiLagEnabled="antiLagEnabled", fpsBoost="fpsBoostEnabled",
            stretchedResEnabled="stretchedResEnabled", nukeOptimizer="nukeOpt",
            removeAccessories="removeAcc", tryardAnimEnabled="tryardAnimEnabled",
            introEnabled="introEnabled", guiVisible="guiVisible",
            speedToggled="speedToggled", autoTPEnabled="autoTPEnabled",
            tracersEnabled="tracersEnabled", instantResetOnMedusa="instantResetOnMedusa",
        }
        for cfgKey, stateKey in pairs(bools) do
            if decErr[cfgKey] ~= nil then State[stateKey] = decErr[cfgKey] end
        end
        if decErr.laggerMode ~= nil then State.laggerMode = decErr.laggerMode end
        if decErr.stretchFOV then State.stretchFOV = decErr.stretchFOV end
        if decErr.activeSky then State.activeSky = decErr.activeSky end
        if decErr.bgSelectedIndex then State.bgSelectedIndex = decErr.bgSelectedIndex end
        -- bgImg photo background removed
        if decErr.skyJumpMode then State.skyJumpMode = decErr.skyJumpMode end

        if decErr.aimbotMode then
            State.aimbotMode = decErr.aimbotMode
        end

        local keyMap = {
            speedKey="speed", laggerKey="lagger", aimbotKey="aimbot",
            autoLeftKey="autoLeft", autoRightKey="autoRight",
            dropKey="drop", tpDownKey="tpDown", guiHideKey="guiHide",
            instantResetKey="instantReset",
            tpBatKey="tpBat",
        }
        for cfgKey, stateKey in pairs(keyMap) do
            if decErr[cfgKey] then
                local kc = Enum.KeyCode[decErr[cfgKey]]
                if kc then
                    Keys[stateKey] = kc
                    if keybindBtnRefs[stateKey] then keybindBtnRefs[stateKey].Text = getGpDisplayName(kc) end
                end
            end
        end

        mainOuter.Visible = State.guiVisible
        if _G.AstaHubQAHide then pcall(_G.AstaHubQAHide, not State.guiVisible) end
        for _, wrapper in pairs(stackWrappers) do wrapper.Visible = not State.stackButtonsHidden end
        if hideButtonsSetter then hideButtonsSetter(State.stackButtonsHidden) end
        if lockButtonsSetter then lockButtonsSetter(State.stackButtonsLocked) end

        if State.laggerMode == 0 then
            if carryBox then carryBox.Text = tostring(State.carrySpeed) end
        elseif State.laggerMode == 1 then
            -- don't overwrite carryBox, lagger speed shows in laggerBox
        elseif State.laggerMode == 2 then
            -- don't overwrite carryBox, lagger carry speed shows in laggerCarryBox
        end
        if stackBtnRefs.carrySpeed then stackBtnRefs.carrySpeed.setOn(State.speedToggled) end
        if stackBtnRefs.lagger then stackBtnRefs.lagger.setOn(State.laggerMode == 1) end
        if stackBtnRefs.laggerCarry then stackBtnRefs.laggerCarry.setOn(State.laggerMode == 2) end
        if stackBtnRefs.aimbot then stackBtnRefs.aimbot.setOn(State.batAimbotToggled) end
        if stackBtnRefs.autoLeft then stackBtnRefs.autoLeft.setOn(State.autoLeftEnabled) end
        if stackBtnRefs.autoRight then stackBtnRefs.autoRight.setOn(State.autoRightEnabled) end

        if State.antiLagEnabled then enableAntiLag() else disableAntiLag() end
        if State.stretchedResEnabled then enableStretchRez() else disableStretchRez() end
        if State.activeSky then applySky(State.activeSky) else applySky(nil) end
        if State.nukeOpt then _G._nukeStart() else _G._nukeStop() end
        if State.removeAcc then _G._removeAccStart() else _G._removeAccStop() end
        if State.tryardAnimEnabled then startTryardAnim() else stopTryardAnim() end
        if State.batAimbotToggled then startBatAimbot() else stopBatAimbot() end
        if State.batCounterEnabled then startBatCounter() else stopBatCounter() end
        if State.medusaCounterEnabled then setupMedusaCounter(LP.Character) else stopMedusaCounter() end
        if State.antiRagdollEnabled then startAntiRagdoll() else stopAntiRagdoll() end
        if Steal.AutoStealEnabled then startAutoSteal() else stopAutoSteal() end
        if State.autoTPEnabled then startAutoTP() else stopAutoTP() end
        if decErr.tpBatEnabled ~= nil then tpBatEnabled = decErr.tpBatEnabled; if tpBatEnabled then startTPBat(); if stackBtnRefs.tpBat then stackBtnRefs.tpBat.setOn(true) end end end

        for key, setter in pairs(toggleSetters) do
            local stateValue = nil
            if key=="autoSteal" then stateValue=Steal.AutoStealEnabled
            elseif key=="infJump" then stateValue=State.infJumpEnabled
            elseif key=="antiRagdoll" then stateValue=State.antiRagdollEnabled
            elseif key=="medusaCounter" then stateValue=State.medusaCounterEnabled
            elseif key=="batCounter" then stateValue=State.batCounterEnabled
            elseif key=="autoSwing" then stateValue=State.autoSwingEnabled
            elseif key=="antiLag" then stateValue=State.antiLagEnabled
            elseif key=="stretchedRes" then stateValue=State.stretchedResEnabled
            elseif key=="nukeOpt" then stateValue=State.nukeOpt
            elseif key=="removeAcc" then stateValue=State.removeAcc
            elseif key=="tryardAnim" then stateValue=State.tryardAnimEnabled
            elseif key=="introEnabled" then stateValue=State.introEnabled
            elseif key=="hideButtons" then stateValue=State.stackButtonsHidden
            elseif key=="lockButtons" then stateValue=State.stackButtonsLocked
            elseif key=="autoTP" then stateValue=State.autoTPEnabled
            elseif key=="tracers" then stateValue=State.tracersEnabled
            elseif key=="medusaAutoReset" then stateValue=State.instantResetOnMedusa
            end
            if stateValue ~= nil then pcall(setter, stateValue) end
        end

        if decErr.buttonPositions then
            for key, posData in pairs(decErr.buttonPositions) do
                local wrapper = stackWrappers[key]
                if wrapper and posData.X and posData.Y then
                    wrapper.Position = UDim2.new(wrapper.Position.X.Scale, posData.X, wrapper.Position.Y.Scale, posData.Y)
                end
            end
        end
        if decErr.cloverPosition and cloverBtn then
            cloverBtn.Position = UDim2.new(0, decErr.cloverPosition.X, 0, decErr.cloverPosition.Y)
        end

        print("[Asta Hub] Config loaded successfully")
        return true
    end

    requestSave = function()
        local ok = saveConfig()
        if ok then
            if _G._VezyFlashSave then _G._VezyFlashSave(true) end
        else
            if _G._VezyFlashSave then _G._VezyFlashSave(false) end
        end
    end

    -- ============================================================
    -- INIT
    -- ============================================================
    loadPresetsFile()
    rebuildPresetList()
    local _lastPresetName = loadLastPresetName()
    if _lastPresetName and _lastPresetName~="" then
        for _,preset in ipairs(Presets) do
            if preset.name==_lastPresetName then
                pcall(function()
                    local d=preset.data or {}
                    if d.normalSpeed then State.normalSpeed=d.normalSpeed; if normalBox then normalBox.Text=tostring(d.normalSpeed) end end
                    if d.carrySpeed then State.carrySpeed=d.carrySpeed; if carryBox then carryBox.Text=tostring(d.carrySpeed) end end
                    if d.laggerSpeed then State.laggerSpeed=d.laggerSpeed; if laggerBox then laggerBox.Text=tostring(d.laggerSpeed) end end
                    if d.laggerCarrySpeed then State.laggerCarrySpeed=d.laggerCarrySpeed; if laggerCarryBox then laggerCarryBox.Text=tostring(d.laggerCarrySpeed) end end
                    if d.stealRadius then Steal.StealRadius=d.stealRadius; if stealRadBox and not stealRadBox:IsFocused() then stealRadBox.Text=tostring(Steal.StealRadius) end end
                    if d.stealDuration then Steal.StealDuration=d.stealDuration; if stealDurBox then stealDurBox.Text=tostring(Steal.StealDuration) end end
                    if d.autoTP ~= nil then State.autoTPEnabled=d.autoTP; if toggleSetters["autoTP"] then toggleSetters["autoTP"](d.autoTP) end end
                    if d.autoTPHeight then State.autoTPHeight=d.autoTPHeight; if autoTPHeightBox then autoTPHeightBox.Text=tostring(d.autoTPHeight) end end
                end)
                break
            end
        end
    end
    loadConfig()
    startAutoSteal()
    print("[Asta Hub] Ready. Stand drop = Brainrot fling (safe). Jump drop = ascend.")
end

-- ============================================================
-- SAFE MAIN EXECUTION
-- ============================================================
if not _G.AstaHub_MainExecuted then
    if LP and LP:FindFirstChild("PlayerGui") then
        Main()
    else
        LP = LP or Players:WaitForChild("LocalPlayer")
        LP:WaitForChild("PlayerGui")
        Main()
    end
end

-- ============================================================
-- OTHER PLAYERS SPEED DISPLAY
-- ============================================================
;(function()
local function setupOtherPlayerSpeed(player)
    if player == LP then return end
    local function onCharacterAdded(char)
        task.wait(0.2)
        local head = char:FindFirstChild("Head")
        local hrp  = char:FindFirstChild("HumanoidRootPart")
        if not head or not hrp then return end
        local oldBB = head:FindFirstChild("AstaHubBB_Other")
        if oldBB then oldBB:Destroy() end
        local bb = Instance.new("BillboardGui", head)
        bb.Name = "AstaHubBB_Other"
        bb.Size = UDim2.new(0, 160, 0, 24)
        bb.StudsOffset = Vector3.new(0, 3, 0)
        bb.AlwaysOnTop = true
        local speedLbl = Instance.new("TextLabel", bb)
        speedLbl.Name = "SpeedBillLbl"
        speedLbl.Size = UDim2.new(1, 0, 1, 0)
        speedLbl.Position = UDim2.new(0, 0, 0, 0)
        speedLbl.BackgroundTransparency = 1
        speedLbl.Text = "0.0"
        speedLbl.TextColor3 = Color3.fromRGB(200,200,200)
        speedLbl.Font = Enum.Font.GothamBlack
        speedLbl.TextScaled = true
        speedLbl.TextStrokeTransparency = 0
        speedLbl.TextStrokeColor3 = Color3.new(0, 0, 0)
        task.spawn(function()
            while char and char.Parent and hrp and hrp.Parent and speedLbl and speedLbl.Parent do
                pcall(function()
                    local hspd = Vector3.new(hrp.Velocity.X, 0, hrp.Velocity.Z).Magnitude
                    speedLbl.Text = string.format("%.1f", hspd)
                end)
                task.wait(0.1)
            end
        end)
    end
    if player.Character then task.spawn(function() onCharacterAdded(player.Character) end) end
    player.CharacterAdded:Connect(onCharacterAdded)
end

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LP then task.spawn(function() setupOtherPlayerSpeed(player) end) end
end
Players.PlayerAdded:Connect(function(player)
    task.spawn(function() setupOtherPlayerSpeed(player) end)
end)
end)()
