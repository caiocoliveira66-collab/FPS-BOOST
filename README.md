--[[ RANDOM HUB - Ultimate Optimizer | Tema: Preto / Azul / Branco ]]

print("[RANDOM HUB] iniciando...")

-- ========= SERVIÇOS =========
local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then
    print("[RANDOM HUB] aguardando LocalPlayer...")
    repeat
        task.wait(0.1)
        LocalPlayer = Players.LocalPlayer
    until LocalPlayer
end

local PlayerGui = LocalPlayer:WaitForChild("PlayerGui", 30)
if not PlayerGui then
    warn("[RANDOM HUB] PlayerGui não encontrado")
    return
end

local CoreGui
do
    local ok, cg = pcall(function() return game:GetService("CoreGui") end)
    if ok then CoreGui = cg end
end

local Stats
do
    local ok, s = pcall(function() return game:GetService("Stats") end)
    if ok then Stats = s end
end

-- ========= ESTADO =========
local RH = { Connections = {}, Original = {}, Flags = {
    FPSBoost=false, FPSMode="Off", RemoveFog=false, ReduceFog=false,
    GraphicsMode="Off", FPSCounter=false, PingMonitor=false,
    ReduceParticles=false, ReducePostFX=false, FOV=70, InterfaceScale=1,
}}

-- ========= UTILS =========
local function safe(fn, ...)
    local ok, e = pcall(fn, ...)
    if not ok then warn("[RH]", e) end
    return ok
end
local function connect(sig, fn)
    local c = sig:Connect(fn)
    table.insert(RH.Connections, c)
    return c
end
local function c3(r,g,b) return Color3.fromRGB(r,g,b) end
local function isProp(o,p) return o and pcall(function() return o[p] end) end
local function setProp(o,p,v) if o and p and isProp(o,p) then return pcall(function() o[p]=v end) end return false end
local function saveOrig(o,p)
    if o and p and isProp(o,p) then
        local k = tostring(o).."."..p
        if RH.Original[k]==nil then
            local ok, v = pcall(function() return o[p] end)
            if ok then RH.Original[k]=v end
        end
    end
end
local function restoreOrig(o,p)
    if o then
        local k = tostring(o).."."..p
        if RH.Original[k]~=nil then
            pcall(function() o[p]=RH.Original[k] end)
            RH.Original[k]=nil
        end
    end
end
local function applyGrad(g, a, b)
    local u = Instance.new("UIGradient")
    u.Color = ColorSequence.new(a,b)
    u.Rotation = 90
    u.Parent = g
    return u
end

-- ========= LIMPAR ANTIGO =========
local parentGui = CoreGui or PlayerGui
if parentGui:FindFirstChild("RandomHubUI") then parentGui.RandomHubUI:Destroy() end

-- ========= SCREEN GUI =========
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RandomHubUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder = 999
ScreenGui.Parent = parentGui

local isMobile = UIS.TouchEnabled and not UIS.KeyboardEnabled
local baseW = isMobile and 320 or 460
local baseH = isMobile and 380 or 460

-- ========= MAIN =========
local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.new(0, baseW, 0, baseH)
Main.Position = UDim2.new(0.5, -baseW/2, 0.5, -baseH/2)
Main.BackgroundColor3 = c3(10,10,14)
Main.BorderSizePixel = 0
Main.ClipsDescendants = true
Main.Active = true
Main.Parent = ScreenGui
local mc = Instance.new("UICorner"); mc.CornerRadius = UDim.new(0,10); mc.Parent = Main
local ms = Instance.new("UIStroke"); ms.Color = c3(0,140,255); ms.Thickness = 1.5; ms.Transparency = 0.2; ms.Parent = Main

-- ========= HEADER =========
local Header = Instance.new("Frame")
Header.Name = "Header"
Header.Size = UDim2.new(1,0,0,isMobile and 44 or 50)
Header.BackgroundColor3 = c3(14,18,28)
Header.BorderSizePixel = 0
Header.Parent = Main
applyGrad(Header, c3(0,90,200), c3(0,30,70))

local title = Instance.new("TextLabel")
title.Text = "RANDOM HUB"; title.Font = Enum.Font.GothamBlack
title.TextSize = isMobile and 16 or 20
title.TextColor3 = c3(255,255,255); title.BackgroundTransparency = 1
title.Size = UDim2.new(1,-110,1,0); title.Position = UDim2.new(0,14,0,0)
title.TextXAlignment = Enum.TextXAlignment.Left; title.Parent = Header

local sub = Instance.new("TextLabel")
sub.Text = " Ultimate Optimizer"; sub.Font = Enum.Font.Gotham
sub.TextSize = 11; sub.TextColor3 = c3(180,210,255); sub.BackgroundTransparency = 1
sub.Size = UDim2.new(1,-110,0,14); sub.Position = UDim2.new(0,14,1,-16)
sub.TextXAlignment = Enum.TextXAlignment.Left; sub.Parent = Header

local function mkBtn(t, x, clr, cb)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0,30,0,30); b.Position = UDim2.new(1,x,0.5,-15)
    b.BackgroundColor3 = clr; b.Text = t; b.Font = Enum.Font.GothamBold
    b.TextSize = 14; b.TextColor3 = c3(255,255,255); b.AutoButtonColor = false; b.Parent = Header
    local cc = Instance.new("UICorner"); cc.CornerRadius = UDim.new(0,6); cc.Parent = b
    b.MouseEnter:Connect(function() TweenService:Create(b, TweenInfo.new(0.15), {BackgroundTransparency=0.2}):Play() end)
    b.MouseLeave:Connect(function() TweenService:Create(b, TweenInfo.new(0.15), {BackgroundTransparency=0}):Play() end)
    b.MouseButton1Click:Connect(cb)
    return b
end

-- ========= BOTÃO FLUTUANTE =========
local FB = Instance.new("TextButton")
FB.Name = "RandomHubFloating"
FB.Size = UDim2.new(0,54,0,54); FB.Position = UDim2.new(0,20,0.5,-27)
FB.BackgroundColor3 = c3(0,120,230); FB.Text = "RH"; FB.Font = Enum.Font.GothamBlack
FB.TextSize = 16; FB.TextColor3 = c3(255,255,255); FB.AutoButtonColor = false
FB.Active = true; FB.ZIndex = 5; FB.Parent = ScreenGui
local fbc = Instance.new("UICorner"); fbc.CornerRadius = UDim.new(1,0); fbc.Parent = FB
local fbs = Instance.new("UIStroke"); fbs.Color = c3(0,180,255); fbs.Thickness = 2; fbs.Parent = FB
applyGrad(FB, c3(0,90,200), c3(0,160,255))

local dot = Instance.new("Frame")
dot.Size = UDim2.new(0,10,0,10); dot.Position = UDim2.new(1,-14,0,4)
dot.BackgroundColor3 = c3(90,100,120); dot.BorderSizePixel = 0; dot.ZIndex = 6; dot.Parent = FB
local dc = Instance.new("UICorner"); dc.CornerRadius = UDim.new(1,0); dc.Parent = dot

local function updStatus()
    local act = RH.Flags.FPSBoost or RH.Flags.RemoveFog or RH.Flags.ReduceFog
        or RH.Flags.FPSCounter or RH.Flags.PingMonitor or RH.Flags.ReduceParticles
        or RH.Flags.ReducePostFX or (RH.Flags.GraphicsMode ~= "Off")
    TweenService:Create(dot, TweenInfo.new(0.2), {BackgroundColor3 = act and c3(0,255,150) or c3(90,100,120)}):Play()
end

local function toggleMenu()
    Main.Visible = not Main.Visible
    if Main.Visible then
        TweenService:Create(FB, TweenInfo.new(0.2), {BackgroundColor3=c3(0,60,130)}):Play(); FB.Text = "✕"
    else
        TweenService:Create(FB, TweenInfo.new(0.2), {BackgroundColor3=c3(0,120,230)}):Play(); FB.Text = "RH"
    end
end

local fbDeb = false
FB.MouseButton1Click:Connect(function()
    if fbDeb then return end
    fbDeb = true
    toggleMenu()
    task.wait(0.15)
    fbDeb = false
end)

local fbDrag, fbStart, fbPos, fbMv
FB.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        fbDrag = true; fbMv = false; fbStart = i.Position; fbPos = FB.Position
        i.Changed:Connect(function() if i.UserInputState == Enum.UserInputState.End then fbDrag = false end end)
    end
end)
UIS.InputChanged:Connect(function(i)
    if fbDrag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        local d = i.Position - fbStart
        if math.abs(d.X) > 3 or math.abs(d.Y) > 3 then fbMv = true end
        FB.Position = UDim2.new(fbPos.X.Scale, fbPos.X.Offset+d.X, fbPos.Y.Scale, fbPos.Y.Offset+d.Y)
    end
end)
FB.MouseButton1Up:Connect(function() if fbMv then fbDeb = true; task.wait(0.1); fbDeb = false end end)

task.spawn(function()
    while ScreenGui.Parent do
        updStatus()
        task.wait(1)
    end
end)

mkBtn("–", -72, c3(30,60,110), function()
    Main.Visible = false
    TweenService:Create(FB, TweenInfo.new(0.2), {BackgroundColor3=c3(0,120,230)}):Play(); FB.Text = "RH"
end)
mkBtn("✕", -36, c3(150,30,30), function()
    safe(function()
        if RH.Flags.FPSCounter then RH:ToggleFPSCounter(false) end
        if RH.Flags.PingMonitor then RH:TogglePing(false) end
    end)
    for _, c in ipairs(RH.Connections) do pcall(function() c:Disconnect() end) end
    RH.Connections = {}
    FB:Destroy()
    ScreenGui:Destroy()
end)

-- Arrastar Main
local drag, dStart, dPos
Header.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
        drag = true; dStart = i.Position; dPos = Main.Position
        i.Changed:Connect(function() if i.UserInputState == Enum.UserInputState.End then drag = false end end)
    end
end)
UIS.InputChanged:Connect(function(i)
    if drag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
        local d = i.Position - dStart
        Main.Position = UDim2.new(dPos.X.Scale, dPos.X.Offset+d.X, dPos.Y.Scale, dPos.Y.Offset+d.Y)
    end
end)

-- ========= CONTENT =========
local Content = Instance.new("Frame")
Content.Name = "Content"
Content.Size = UDim2.new(1,-12,1,-(Header.Size.Y.Offset+60))
Content.Position = UDim2.new(0,6,0,Header.Size.Y.Offset+6)
Content.BackgroundTransparency = 1; Content.ClipsDescendants = true; Content.Parent = Main

-- ========= TABBAR =========
local TabBar = Instance.new("ScrollingFrame")
TabBar.Size = UDim2.new(1,-12,0,40); TabBar.Position = UDim2.new(0,6,1,-46)
TabBar.BackgroundColor3 = c3(16,20,30); TabBar.BorderSizePixel = 0
TabBar.ScrollBarThickness = 2; TabBar.ScrollBarImageColor3 = c3(0,140,255)
TabBar.CanvasSize = UDim2.new(0,0,0,0); TabBar.AutomaticCanvasSize = Enum.AutomaticSize.X
TabBar.ScrollingDirection = Enum.ScrollingDirection.X; TabBar.Parent = Main
local tbc = Instance.new("UICorner"); tbc.CornerRadius = UDim.new(0,8); tbc.Parent = TabBar
local tl = Instance.new("UIListLayout"); tl.FillDirection = Enum.FillDirection.Horizontal
tl.Padding = UDim.new(0,4); tl.SortOrder = Enum.SortOrder.LayoutOrder
tl.VerticalAlignment = Enum.VerticalAlignment.Center; tl.Parent = TabBar
local tp = Instance.new("UIPadding"); tp.PaddingLeft = UDim.new(0,4); tp.PaddingRight = UDim.new(0,4); tp.Parent = TabBar

-- ========= PÁGINAS =========
local Pages, TabBtns = {}, {}

local function mkPage(name)
    local pg = Instance.new("ScrollingFrame")
    pg.Name = name; pg.Size = UDim2.new(1,0,1,0); pg.BackgroundTransparency = 1
    pg.BorderSizePixel = 0; pg.ScrollBarThickness = 3; pg.ScrollBarImageColor3 = c3(0,140,255)
    pg.CanvasSize = UDim2.new(0,0,0,0); pg.AutomaticCanvasSize = Enum.AutomaticSize.Y
    pg.Visible = false; pg.Parent = Content
    local l = Instance.new("UIListLayout"); l.Padding = UDim.new(0,8); l.SortOrder = Enum.SortOrder.LayoutOrder; l.Parent = pg
    local p = Instance.new("UIPadding"); p.PaddingTop = UDim.new(0,4); p.PaddingBottom = UDim.new(0,8)
    p.PaddingLeft = UDim.new(0,4); p.PaddingRight = UDim.new(0,4); p.Parent = pg
    Pages[name] = pg
end

local function mkTab(name, ord)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0,82,0,30); b.BackgroundColor3 = c3(22,30,45); b.Text = name
    b.Font = Enum.Font.GothamBold; b.TextSize = 12; b.TextColor3 = c3(200,220,255)
    b.AutoButtonColor = false; b.LayoutOrder = ord; b.Parent = TabBar
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,6); c.Parent = b
    b.MouseButton1Click:Connect(function()
        for n, p in pairs(Pages) do
            p.Visible = (n == name)
            if n == name then p.CanvasPosition = Vector2.new(0,0) end
        end
        for n, t in pairs(TabBtns) do
            TweenService:Create(t, TweenInfo.new(0.15), {BackgroundColor3 = (n==name) and c3(0,130,240) or c3(22,30,45)}):Play()
        end
    end)
    TabBtns[name] = b
end

for i, n in ipairs({"Home","FPS Boost","Graphics","Remove Fog","Stretched","FOV","Monitor","Settings"}) do
    mkPage(n)
    mkTab(n, i)
end
Pages["Home"].Visible = true
TabBtns["Home"].BackgroundColor3 = c3(0,130,240)

-- ========= HELPERS UI =========
local function mkSec(parent, t)
    local f = Instance.new("Frame"); f.Size = UDim2.new(1,0,0,30); f.BackgroundTransparency = 1; f.Parent = parent
    local l = Instance.new("TextLabel"); l.Text = t; l.Font = Enum.Font.GothamBold; l.TextSize = 13
    l.TextColor3 = c3(0,170,255); l.BackgroundTransparency = 1; l.Size = UDim2.new(1,-10,1,0)
    l.Position = UDim2.new(0,10,0,0); l.TextXAlignment = Enum.TextXAlignment.Left; l.Parent = f
end

local function mkTog(parent, lbl, def, cb)
    local f = Instance.new("Frame"); f.Size = UDim2.new(1,0,0,40); f.BackgroundColor3 = c3(18,22,32)
    f.BorderSizePixel = 0; f.Parent = parent
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,6); c.Parent = f
    local l = Instance.new("TextLabel"); l.Text = lbl; l.Font = Enum.Font.GothamSemibold; l.TextSize = 12
    l.TextColor3 = c3(230,235,245); l.BackgroundTransparency = 1; l.Size = UDim2.new(1,-80,1,0)
    l.Position = UDim2.new(0,12,0,0); l.TextXAlignment = Enum.TextXAlignment.Left; l.Parent = f
    local bg = Instance.new("Frame"); bg.Size = UDim2.new(0,40,0,20); bg.Position = UDim2.new(1,-52,0.5,-10)
    bg.BackgroundColor3 = def and c3(0,150,255) or c3(45,50,60); bg.Parent = f
    local bgc = Instance.new("UICorner"); bgc.CornerRadius = UDim.new(1,0); bgc.Parent = bg
    local kn = Instance.new("Frame"); kn.Size = UDim2.new(0,16,0,16)
    kn.Position = def and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8)
    kn.BackgroundColor3 = c3(255,255,255); kn.Parent = bg
    local kc = Instance.new("UICorner"); kc.CornerRadius = UDim.new(1,0); kc.Parent = kn
    local b = Instance.new("TextButton"); b.Size = UDim2.new(1,0,1,0); b.BackgroundTransparency = 1; b.Text = ""; b.Parent = f
    local st = def
    b.MouseButton1Click:Connect(function()
        st = not st
        TweenService:Create(bg, TweenInfo.new(0.2), {BackgroundColor3 = st and c3(0,150,255) or c3(45,50,60)}):Play()
        TweenService:Create(kn, TweenInfo.new(0.2), {Position = st and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8)}):Play()
        if cb then safe(cb, st) end
    end)
end

local function mkBtn2(parent, lbl, cb)
    local b = Instance.new("TextButton"); b.Size = UDim2.new(1,0,0,36); b.BackgroundColor3 = c3(0,100,200)
    b.Text = lbl; b.Font = Enum.Font.GothamBold; b.TextSize = 12; b.TextColor3 = c3(255,255,255)
    b.AutoButtonColor = false; b.Parent = parent
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,6); c.Parent = b
    b.MouseEnter:Connect(function() TweenService:Create(b, TweenInfo.new(0.15), {BackgroundColor3=c3(0,140,240)}):Play() end)
    b.MouseLeave:Connect(function() TweenService:Create(b, TweenInfo.new(0.15), {BackgroundColor3=c3(0,100,200)}):Play() end)
    b.MouseButton1Click:Connect(function() safe(cb) end)
end

local function mkSld(parent, lbl, mn, mx, def, cb)
    local f = Instance.new("Frame"); f.Size = UDim2.new(1,0,0,54); f.BackgroundColor3 = c3(18,22,32)
    f.BorderSizePixel = 0; f.Parent = parent
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,6); c.Parent = f
    local l = Instance.new("TextLabel"); l.Text = lbl..": "..tostring(def); l.Font = Enum.Font.GothamSemibold
    l.TextSize = 12; l.TextColor3 = c3(230,235,245); l.BackgroundTransparency = 1
    l.Size = UDim2.new(1,-20,0,20); l.Position = UDim2.new(0,12,0,4)
    l.TextXAlignment = Enum.TextXAlignment.Left; l.Parent = f
    local bar = Instance.new("Frame"); bar.Size = UDim2.new(1,-24,0,6); bar.Position = UDim2.new(0,12,0,34)
    bar.BackgroundColor3 = c3(40,45,55); bar.Parent = f
    local bc = Instance.new("UICorner"); bc.CornerRadius = UDim.new(1,0); bc.Parent = bar
    local fill = Instance.new("Frame"); fill.Size = UDim2.new((def-mn)/(mx-mn),0,1,0)
    fill.BackgroundColor3 = c3(0,150,255); fill.Parent = bar
    local fc = Instance.new("UICorner"); fc.CornerRadius = UDim.new(1,0); fc.Parent = fill
    local b = Instance.new("TextButton"); b.Size = UDim2.new(1,0,1,0); b.BackgroundTransparency = 1; b.Text = ""; b.Parent = f
    local function upd(x)
        local r = math.clamp((x-bar.AbsolutePosition.X)/bar.AbsoluteSize.X, 0, 1)
        local v = math.floor(mn+r*(mx-mn)+0.5)
        fill.Size = UDim2.new(r,0,1,0); l.Text = lbl..": "..tostring(v)
        if cb then safe(cb, v) end
    end
    b.MouseButton1Down:Connect(function()
        upd(UIS:GetMouseLocation().X)
        local mc2 = UIS.InputChanged:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then upd(i.Position.X) end
        end)
        local ec = UIS.InputEnded:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                mc2:Disconnect(); ec:Disconnect()
            end
        end)
    end)
end

local function mkLbl(parent, txt, clr, size)
    local l = Instance.new("TextLabel"); l.Text = txt; l.Font = Enum.Font.Gotham
    l.TextSize = size or 12; l.TextColor3 = clr or c3(180,200,230); l.BackgroundTransparency = 1
    l.Size = UDim2.new(1,-20,0,0); l.AutomaticSize = Enum.AutomaticSize.Y; l.TextWrapped = true
    l.TextXAlignment = Enum.TextXAlignment.Left; l.Parent = parent
    local p = Instance.new("UIPadding"); p.PaddingLeft = UDim.new(0,10); p.PaddingRight = UDim.new(0,10); p.Parent = l
end

-- ========= NOTIFICAÇÕES (2s) =========
local NC = Instance.new("Frame"); NC.Size = UDim2.new(0,260,1,0); NC.Position = UDim2.new(1,-270,0,0)
NC.BackgroundTransparency = 1; NC.Parent = ScreenGui
local nl = Instance.new("UIListLayout"); nl.Padding = UDim.new(0,6)
nl.VerticalAlignment = Enum.VerticalAlignment.Bottom; nl.HorizontalAlignment = Enum.HorizontalAlignment.Right
nl.SortOrder = Enum.SortOrder.LayoutOrder; nl.Parent = NC
local nOrd = 0

local function notify(txt)
    safe(function()
        nOrd = nOrd + 1
        local wrap = Instance.new("Frame"); wrap.Size = UDim2.new(1,0,0,0); wrap.BackgroundTransparency = 1
        wrap.LayoutOrder = nOrd; wrap.Parent = NC
        local n = Instance.new("TextLabel"); n.Size = UDim2.new(1,0,1,0); n.BackgroundColor3 = c3(12,16,24)
        n.BackgroundTransparency = 1; n.Text = "  "..txt; n.Font = Enum.Font.GothamSemibold
        n.TextSize = 12; n.TextColor3 = c3(220,235,255); n.TextTransparency = 1
        n.TextXAlignment = Enum.TextXAlignment.Left; n.Parent = wrap
        local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0,6); c.Parent = n
        local s = Instance.new("UIStroke"); s.Color = c3(0,150,255); s.Transparency = 1; s.Parent = n
        TweenService:Create(wrap, TweenInfo.new(0.2), {Size = UDim2.new(1,0,0,36)}):Play()
        TweenService:Create(n, TweenInfo.new(0.2), {BackgroundTransparency = 0, TextTransparency = 0}):Play()
        TweenService:Create(s, TweenInfo.new(0.2), {Transparency = 0.3}):Play()
        task.delay(2, function()
            if not wrap.Parent then return end
            TweenService:Create(wrap, TweenInfo.new(0.25), {Size = UDim2.new(1,0,0,0)}):Play()
            TweenService:Create(n, TweenInfo.new(0.25), {BackgroundTransparency = 1, TextTransparency = 1}):Play()
            TweenService:Create(s, TweenInfo.new(0.25), {Transparency = 1}):Play()
            task.wait(0.28)
            if wrap and wrap.Parent then wrap:Destroy() end
        end)
    end)
end

-- ========= HUD OPÇÕES ATIVAS =========
local AH = Instance.new("Frame"); AH.Name = "ActiveHUD"; AH.Size = UDim2.new(0,240,0,0)
AH.Position = UDim2.new(0,16,1,-16); AH.AnchorPoint = Vector2.new(0,1)
AH.BackgroundTransparency = 1; AH.AutomaticSize = Enum.AutomaticSize.Y; AH.Parent = ScreenGui
local hl = Instance.new("UIListLayout"); hl.Padding = UDim.new(0,6)
hl.VerticalAlignment = Enum.VerticalAlignment.Bottom; hl.SortOrder = Enum.SortOrder.LayoutOrder; hl.Parent = AH
local Entries, eOrd = {}, 0

local function animOut(entry, key)
    if not entry or not entry.frame or not entry.frame.Parent then Entries[key] = nil; return end
    if entry.exiting then return end
    entry.exiting = true
    if entry.task then pcall(task.cancel, entry.task); entry.task = nil end
    local f = entry.frame; local sp = f.Position
    TweenService:Create(f, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
        Position = UDim2.new(sp.X.Scale, sp.X.Offset-280, sp.Y.Scale, sp.Y.Offset), BackgroundTransparency = 1}):Play()
    for _, ch in ipairs(f:GetDescendants()) do
        if ch:IsA("TextLabel") then
            TweenService:Create(ch, TweenInfo.new(0.35), {TextTransparency = 1}):Play()
        elseif ch:IsA("Frame") then
            if ch.BackgroundTransparency < 1 then
                TweenService:Create(ch, TweenInfo.new(0.35), {BackgroundTransparency = 1}):Play()
            end
        elseif ch:IsA("UIStroke") then
            TweenService:Create(ch, TweenInfo.new(0.35), {Transparency = 1}):Play()
        end
    end
    task.delay(0.4, function()
        if f and f.Parent then f:Destroy() end
        if Entries[key] == entry then Entries[key] = nil end
    end)
end

-- Duração padrão 2s em todos os cards
local function setActive(key, txt, dur)
    dur = dur or 2
    if Entries[key] then
        local e = Entries[key]
        if e.exiting then e.exiting = false; e.frame.Position = UDim2.new(0,0,0,0) end
        e.label.Text = "  "..txt
        if e.task then pcall(task.cancel, e.task); e.task = nil end
        e.fill.Size = UDim2.new(1,0,1,0)
        TweenService:Create(e.fill, TweenInfo.new(dur, Enum.EasingStyle.Linear), {Size = UDim2.new(0,0,1,0)}):Play()
        e.task = task.delay(dur, function() if Entries[key] == e then animOut(e, key) end end)
        return
    end
    eOrd = eOrd + 1
    local f = Instance.new("Frame"); f.Size = UDim2.new(0,240,0,44); f.Position = UDim2.new(0,0,0,0)
    f.BackgroundColor3 = c3(12,16,24); f.BorderSizePixel = 0; f.LayoutOrder = -eOrd; f.Parent = AH
    local fc = Instance.new("UICorner"); fc.CornerRadius = UDim.new(0,6); fc.Parent = f
    local fs = Instance.new("UIStroke"); fs.Color = c3(0,150,255); fs.Transparency = 0.35; fs.Parent = f
    local bb = Instance.new("Frame"); bb.Size = UDim2.new(1,-16,0,4); bb.Position = UDim2.new(0,8,1,-8)
    bb.BackgroundColor3 = c3(30,40,55); bb.BorderSizePixel = 0; bb.Parent = f
    local bc = Instance.new("UICorner"); bc.CornerRadius = UDim.new(1,0); bc.Parent = bb
    local fill = Instance.new("Frame"); fill.Size = UDim2.new(1,0,1,0); fill.BackgroundColor3 = c3(0,150,255)
    fill.BorderSizePixel = 0; fill.Parent = bb
    local flc = Instance.new("UICorner"); flc.CornerRadius = UDim.new(1,0); flc.Parent = fill
    local dt = Instance.new("Frame"); dt.Size = UDim2.new(0,6,0,6); dt.Position = UDim2.new(0,10,0,11)
    dt.BackgroundColor3 = c3(0,255,150); dt.Parent = f
    local dtc = Instance.new("UICorner"); dtc.CornerRadius = UDim.new(1,0); dtc.Parent = dt
    local lbl = Instance.new("TextLabel"); lbl.Size = UDim2.new(1,-30,0,22); lbl.Position = UDim2.new(0,20,0,4)
    lbl.BackgroundTransparency = 1; lbl.Text = "  "..txt; lbl.Font = Enum.Font.GothamSemibold
    lbl.TextSize = 11; lbl.TextColor3 = c3(220,235,255); lbl.TextXAlignment = Enum.TextXAlignment.Left; lbl.Parent = f
    local e = {frame=f, label=lbl, fill=fill, bar=bb, task=nil, exiting=false}
    Entries[key] = e
    f.Size = UDim2.new(0,240,0,0)
    TweenService:Create(f, TweenInfo.new(0.2, Enum.EasingStyle.Back), {Size = UDim2.new(0,240,0,44)}):Play()
    TweenService:Create(fill, TweenInfo.new(dur, Enum.EasingStyle.Linear), {Size = UDim2.new(0,0,1,0)}):Play()
    e.task = task.delay(dur, function() if Entries[key] == e then animOut(e, key) end end)
end

local function clearActive(key) if Entries[key] then animOut(Entries[key], key) end end

-- ========= MÓDULOS =========
local function snapLight()
    safe(function()
        for _, p in ipairs({"Brightness","Ambient","OutdoorAmbient","GlobalShadows","EnvironmentDiffuseScale","EnvironmentSpecularScale","ShadowSoftness","ExposureCompensation"}) do
            saveOrig(Lighting, p)
        end
    end)
end

local function applyFPS(mode)
    snapLight()
    if mode == "Off" then
        for _, p in ipairs({"GlobalShadows","Brightness","EnvironmentDiffuseScale","EnvironmentSpecularScale"}) do
            restoreOrig(Lighting, p)
        end
        clearActive("fpsmode"); notify("FPS Boost desativado")
        RH.Flags.FPSMode = "Off"; RH.Flags.FPSBoost = false; updStatus(); return
    end
    if mode == "Low" then
        setProp(Lighting,"GlobalShadows",false); setProp(Lighting,"Brightness",1)
        setProp(Lighting,"EnvironmentDiffuseScale",0); setProp(Lighting,"EnvironmentSpecularScale",0)
        setActive("fpsmode","FPS Boost: Low",2); notify("FPS Boost: Low")
    elseif mode == "Medium" then
        setProp(Lighting,"GlobalShadows",true); setProp(Lighting,"Brightness",1.5)
        setProp(Lighting,"EnvironmentDiffuseScale",0.15); setProp(Lighting,"EnvironmentSpecularScale",0.15)
        setActive("fpsmode","FPS Boost: Medium",2); notify("FPS Boost: Medium")
    elseif mode == "High" then
        setProp(Lighting,"GlobalShadows",true); setProp(Lighting,"Brightness",2)
        setProp(Lighting,"EnvironmentDiffuseScale",0.4); setProp(Lighting,"EnvironmentSpecularScale",0.4)
        setActive("fpsmode","FPS Boost: High",2); notify("FPS Boost: High")
    end
    RH.Flags.FPSMode = mode; RH.Flags.FPSBoost = (mode ~= "Off"); updStatus()
end

local function setPostFX(en)
    for _, fx in ipairs(Lighting:GetChildren()) do
        if fx:IsA("BlurEffect") or fx:IsA("BloomEffect") or fx:IsA("ColorCorrectionEffect") or fx:IsA("SunRaysEffect") or fx:IsA("DepthOfFieldEffect") then
            saveOrig(fx,"Enabled"); setProp(fx,"Enabled",en)
        end
    end
end

local function setParticles(en)
    safe(function()
        for _, d in ipairs(Workspace:GetDescendants()) do
            if d:IsA("ParticleEmitter") or d:IsA("Smoke") or d:IsA("Fire") or d:IsA("Sparkles") then
                saveOrig(d,"Enabled"); setProp(d,"Enabled",en)
            end
        end
    end)
end

local function applyGraphics(mode)
    RH.Flags.GraphicsMode = mode
    if mode == "Off" then
        for _, fx in ipairs(Lighting:GetChildren()) do
            if fx:IsA("PostEffect") then restoreOrig(fx,"Enabled") end
        end
        restoreOrig(Lighting,"GlobalShadows"); restoreOrig(Lighting,"Brightness")
        clearActive("graphics"); notify("Graphics: restaurado"); updStatus(); return
    end
    if mode == "Min" then
        setPostFX(false); snapLight(); setProp(Lighting,"GlobalShadows",false); setProp(Lighting,"Brightness",1)
        setActive("graphics","Graphics: Mínimo", 2); notify("Graphics: mínimo")
    elseif mode == "Balanced" then
        setPostFX(true); snapLight(); setProp(Lighting,"GlobalShadows",true)
        setActive("graphics","Graphics: Equilibrado", 2); notify("Graphics: equilibrado")
    elseif mode == "HighPerf" then
        setPostFX(false); snapLight(); setProp(Lighting,"GlobalShadows",false); setProp(Lighting,"Brightness",0.8)
        setActive("graphics","Graphics: Alto Desempenho", 2); notify("Graphics: alto desempenho")
    end
    updStatus()
end

local function removeFog()
    snapLight()
    for _, p in ipairs({"FogStart","FogEnd","FogColor"}) do saveOrig(Lighting,p) end
    setProp(Lighting,"FogStart",0); setProp(Lighting,"FogEnd",100000)
    for _, a in ipairs(Lighting:GetChildren()) do
        if a:IsA("Atmosphere") then
            for _, p in ipairs({"Density","Haze","Glare"}) do saveOrig(a,p); setProp(a,p,0) end
        end
    end
end

local function reduceFog()
    snapLight()
    for _, p in ipairs({"FogStart","FogEnd"}) do saveOrig(Lighting,p) end
    if isProp(Lighting,"FogStart") and Lighting.FogStart then setProp(Lighting,"FogStart", math.min(Lighting.FogStart+100, 10000)) end
    if isProp(Lighting,"FogEnd") and Lighting.FogEnd then setProp(Lighting,"FogEnd", math.max(Lighting.FogEnd*2, 5000)) end
    for _, a in ipairs(Lighting:GetChildren()) do
        if a:IsA("Atmosphere") then saveOrig(a,"Density"); setProp(a,"Density", (a.Density or 0)*0.3) end
    end
end

local function restoreFog(silent)
    for _, p in ipairs({"FogStart","FogEnd","FogColor"}) do restoreOrig(Lighting,p) end
    for _, a in ipairs(Lighting:GetChildren()) do
        if a:IsA("Atmosphere") then
            for _, p in ipairs({"Density","Haze","Glare"}) do restoreOrig(a,p) end
        end
    end
    clearActive("fog_remove"); clearActive("fog_reduce")
    if not silent then notify("Neblina restaurada") end
end

local function setFOV(v)
    local cam = Workspace.CurrentCamera
    if not cam then return end
    v = math.clamp(v, 40, 120)
    if isProp(cam,"FieldOfView") then cam.FieldOfView = v; RH.Flags.FOV = v end
end

-- ========= FPS COUNTER =========
local FPSGui = Instance.new("TextLabel"); FPSGui.Name = "RandomHubFPS"
FPSGui.Size = UDim2.new(0,130,0,28); FPSGui.Position = UDim2.new(0,10,0,10)
FPSGui.BackgroundColor3 = c3(0,0,0); FPSGui.BackgroundTransparency = 0.35
FPSGui.TextColor3 = c3(0,200,255); FPSGui.Font = Enum.Font.Code; FPSGui.TextSize = 14
FPSGui.Text = "FPS: --"; FPSGui.Visible = false; FPSGui.Parent = ScreenGui
local fgc = Instance.new("UICorner"); fgc.CornerRadius = UDim.new(0,6); fgc.Parent = FPSGui
local fpsConn
local function startFPS()
    if fpsConn then return end
    local fr, acc = 0, 0
    fpsConn = RunService.RenderStepped:Connect(function(dt)
        fr = fr + 1; acc = acc + dt
        if acc >= 1 then FPSGui.Text = "FPS: "..fr; fr, acc = 0, 0 end
    end)
end
local function stopFPS()
    if fpsConn then fpsConn:Disconnect(); fpsConn = nil end
    FPSGui.Text = "FPS: --"
end

-- ========= PING =========
local PingGui = Instance.new("TextLabel"); PingGui.Name = "RandomHubPing"
PingGui.Size = UDim2.new(0,130,0,28); PingGui.Position = UDim2.new(0,10,0,44)
PingGui.BackgroundColor3 = c3(0,0,0); PingGui.BackgroundTransparency = 0.35
PingGui.TextColor3 = c3(0,255,150); PingGui.Font = Enum.Font.Code; PingGui.TextSize = 14
PingGui.Text = "Ping: --"; PingGui.Visible = false; PingGui.Parent = ScreenGui
local pgc = Instance.new("UICorner"); pgc.CornerRadius = UDim.new(0,6); pgc.Parent = PingGui
local pingConn
local function getPing()
    if not Stats then return nil end
    local ok, v = pcall(function()
        local n = Stats.Network
        if n and n.ServerStatsItem then
            local i = n.ServerStatsItem["Data Ping"]
            if i then return i:GetValue() end
        end
    end)
    if ok and v then return v end
    local ok2, p = pcall(function()
        local pf = Stats.PerformanceStats
        if pf and pf.Ping then return pf.Ping:GetValue() end
    end)
    return ok2 and p or nil
end
local function startPing()
    if pingConn then return end
    local t = 0
    pingConn = RunService.Heartbeat:Connect(function()
        t = t + 1
        if t < 60 then return end
        t = 0
        local p = getPing()
        if p and p > 0 then
            PingGui.Text = string.format("Ping: %d ms", math.floor(p))
            if p < 80 then PingGui.TextColor3 = c3(0,255,150)
            elseif p < 150 then PingGui.TextColor3 = c3(255,220,0)
            else PingGui.TextColor3 = c3(255,80,80) end
        else
            PingGui.Text = "Ping: indisponível"
            PingGui.TextColor3 = c3(180,180,180)
        end
    end)
end
local function stopPing()
    if pingConn then pingConn:Disconnect(); pingConn = nil end
    PingGui.Text = "Ping: --"
end

function RH:ToggleFPSCounter(on)
    RH.Flags.FPSCounter = on
    FPSGui.Visible = on
    if on then
        startFPS()
        setActive("fpscounter","Contador de FPS: ON",2)
        notify("Contador de FPS ativado")
    else
        stopFPS()
        clearActive("fpscounter")
        notify("Contador de FPS desativado")
    end
    updStatus()
end
function RH:TogglePing(on)
    RH.Flags.PingMonitor = on
    PingGui.Visible = on
    if on then
        startPing()
        setActive("pingmon","Monitor de Ping: ON",2)
        notify("Monitor de Ping ativado")
    else
        stopPing()
        clearActive("pingmon")
        notify("Monitor de Ping desativado")
    end
    updStatus()
end

-- ========= PÁGINA HOME =========
safe(function()
    local home = Pages["Home"]
    local t = Instance.new("TextLabel"); t.Text = "RANDOM HUB"; t.Font = Enum.Font.GothamBlack
    t.TextSize = 24; t.TextColor3 = c3(0,170,255); t.BackgroundTransparency = 1
    t.Size = UDim2.new(1,0,0,34); t.Parent = home
    mkLbl(home,"Ultimate Optimizer — preto, azul e branco.", c3(180,200,230), 12)
    mkLbl(home,"Ative recursos individualmente. Valores originais podem ser restaurados em Settings.", c3(150,170,200), 11)
    mkSec(home,"Status")
    local st = Instance.new("TextLabel"); st.Font = Enum.Font.Code; st.TextSize = 11
    st.TextColor3 = c3(0,200,255); st.BackgroundColor3 = c3(14,18,28)
    st.Size = UDim2.new(1,0,0,100); st.TextWrapped = true; st.TextXAlignment = Enum.TextXAlignment.Left
    st.TextYAlignment = Enum.TextYAlignment.Top; st.Parent = home
    local sc = Instance.new("UICorner"); sc.CornerRadius = UDim.new(0,6); sc.Parent = st
    local sp = Instance.new("UIPadding"); sp.PaddingLeft = UDim.new(0,10); sp.PaddingTop = UDim.new(0,8); sp.Parent = st
    task.spawn(function()
        while ScreenGui.Parent do
            st.Text = string.format("FPS Boost: %s\nGraphics: %s\nFog: %s\nFOV: %d\nFPS Counter: %s\nPing: %s",
                RH.Flags.FPSBoost and RH.Flags.FPSMode or "Off", RH.Flags.GraphicsMode,
                RH.Flags.RemoveFog and "Removida" or (RH.Flags.ReduceFog and "Reduzida" or "Off"),
                RH.Flags.FOV, RH.Flags.FPSCounter and "ON" or "OFF", RH.Flags.PingMonitor and "ON" or "OFF")
            task.wait(0.5)
        end
    end)
    mkBtn2(home,"Aplicar configurações recomendadas", function()
        applyFPS("Medium")
        applyGraphics("Balanced")
        RH:ToggleFPSCounter(true)
        RH:TogglePing(true)
        setFOV(75)
        setActive("fov","FOV: 75",2)
        notify("Configurações recomendadas aplicadas")
    end)
end)

-- ========= PÁGINA FPS BOOST =========
safe(function()
    local p = Pages["FPS Boost"]
    mkLbl(p,"Escolha um modo de desempenho. As alterações afetam apenas o seu cliente.", c3(150,170,200), 11)
    mkSec(p,"Modos")
    mkBtn2(p,"Low Performance", function() applyFPS("Low") end)
    mkBtn2(p,"Medium Performance", function() applyFPS("Medium") end)
    mkBtn2(p,"High Performance", function() applyFPS("High") end)
    mkBtn2(p,"Desativar FPS Boost", function() applyFPS("Off") end)
    mkSec(p,"Restauração")
    mkBtn2(p,"Restaurar configurações originais", function()
        for _, k in ipairs({"Brightness","Ambient","OutdoorAmbient","GlobalShadows","EnvironmentDiffuseScale","EnvironmentSpecularScale","ShadowSoftness","ExposureCompensation"}) do
            restoreOrig(Lighting,k)
        end
        RH.Flags.FPSBoost = false; RH.Flags.FPSMode = "Off"; clearActive("fpsmode"); updStatus()
        notify("Configurações de iluminação restauradas")
    end)
end)

-- ========= PÁGINA GRAPHICS =========
safe(function()
    local p = Pages["Graphics"]
    mkSec(p,"Modos Gráficos")
    mkBtn2(p,"Gráfico Mínimo", function() applyGraphics("Min") end)
    mkBtn2(p,"Gráfico Equilibrado", function() applyGraphics("Balanced") end)
    mkBtn2(p,"Alto Desempenho", function() applyGraphics("HighPerf") end)
    mkBtn2(p,"Restaurar gráficos", function() applyGraphics("Off") end)
    mkSec(p,"Efeitos")
    mkTog(p,"Reduzir partículas", false, function(on)
        RH.Flags.ReduceParticles = on; setParticles(not on)
        if on then setActive("particles","Partículas reduzidas",2); notify("Partículas reduzidas")
        else clearActive("particles"); notify("Partículas restauradas") end
        updStatus()
    end)
    mkTog(p,"Reduzir pós-processamento", false, function(on)
        RH.Flags.ReducePostFX = on; setPostFX(not on)
        if on then setActive("postfx","Pós-processamento reduzido",2); notify("Pós-processamento reduzido")
        else clearActive("postfx"); notify("Pós-processamento restaurado") end
        updStatus()
    end)
    mkTog(p,"Reduzir sombras", false, function(on)
        snapLight(); setProp(Lighting,"GlobalShadows", not on)
        if on then setActive("shadows","Sombras reduzidas",2); notify("Sombras reduzidas")
        else clearActive("shadows"); notify("Sombras restauradas") end
        updStatus()
    end)
end)

-- ========= PÁGINA REMOVE FOG =========
safe(function()
    local p = Pages["Remove Fog"]
    mkLbl(p,"Controle de neblina local e atmosfera.", c3(150,170,200), 11)
    mkTog(p,"Remover neblina", false, function(on)
        RH.Flags.RemoveFog = on
        if on then removeFog(); setActive("fog_remove","Neblina removida",2); notify("Neblina removida")
        else RH.Flags.RemoveFog = false; restoreFog(true); notify("Neblina restaurada") end
        updStatus()
    end)
    mkTog(p,"Reduzir intensidade da neblina", false, function(on)
        RH.Flags.ReduceFog = on
        if on then reduceFog(); setActive("fog_reduce","Neblina reduzida",2); notify("Intensidade da neblina reduzida")
        else RH.Flags.ReduceFog = false; clearActive("fog_reduce"); notify("Intensidade da neblina restaurada") end
        updStatus()
    end)
    mkBtn2(p,"Restaurar neblina", function()
        RH.Flags.RemoveFog = false; RH.Flags.ReduceFog = false
        restoreFog(false); updStatus()
    end)
end)

-- ========= PÁGINA STRETCHED =========
safe(function()
    local p = Pages["Stretched"]
    mkLbl(p,"Ajustes de tela e visualização.", c3(150,170,200), 11)
    mkSec(p,"Interface")
    mkSld(p,"Escala da interface", 70, 130, 100, function(v)
        RH.Flags.InterfaceScale = v/100
        Main.Size = UDim2.new(0, baseW*(v/100), 0, baseH*(v/100))
        Main.Position = UDim2.new(0.5, -baseW*(v/100)/2, 0.5, -baseH*(v/100)/2)
        setActive("uiscale", string.format("Escala UI: %d%%", v), 2)
        notify(string.format("Escala da interface: %d%%", v))
    end)
    mkBtn2(p,"Visualização compacta", function()
        Main.Size = UDim2.new(0, baseW*0.7, 0, baseH*0.7)
        Main.Position = UDim2.new(0.5, -baseW*0.35, 0.5, -baseH*0.35)
        setActive("view","Visualização compacta",2); notify("Visualização compacta aplicada")
    end)
    mkBtn2(p,"Visualização expandida", function()
        Main.Size = UDim2.new(0, baseW*1.3, 0, baseH*1.3)
        Main.Position = UDim2.new(0.5, -baseW*0.65, 0.5, -baseH*0.65)
        setActive("view","Visualização expandida",2); notify("Visualização expandida aplicada")
    end)
    mkBtn2(p,"Restaurar tamanho padrão", function()
        Main.Size = UDim2.new(0, baseW, 0, baseH)
        Main.Position = UDim2.new(0.5, -baseW/2, 0.5, -baseH/2)
        clearActive("view"); notify("Tamanho padrão restaurado")
    end)
end)

-- ========= PÁGINA FOV =========
safe(function()
    local p = Pages["FOV"]
    if not Workspace.CurrentCamera then
        mkLbl(p,"Câmera indisponível.", c3(255,100,100), 12)
        return
    end
    saveOrig(Workspace.CurrentCamera,"FieldOfView")
    local def = Workspace.CurrentCamera.FieldOfView
    RH.Flags.FOV = def
    mkSld(p,"Field of View", 40, 120, def, function(v)
        setFOV(v); setActive("fov", string.format("FOV: %d", v), 2)
        notify("FOV ajustado para "..tostring(v))
    end)
    mkBtn2(p,"Aumentar FOV (+5)", function()
        setFOV(RH.Flags.FOV+5); setActive("fov", string.format("FOV: %d", RH.Flags.FOV), 2)
        notify("FOV aumentado para "..tostring(RH.Flags.FOV))
    end)
    mkBtn2(p,"Diminuir FOV (-5)", function()
        setFOV(RH.Flags.FOV-5); setActive("fov", string.format("FOV: %d", RH.Flags.FOV), 2)
        notify("FOV diminuído para "..tostring(RH.Flags.FOV))
    end)
    mkBtn2(p,"Restaurar FOV padrão", function()
        setFOV(def); setActive("fov", string.format("FOV: %d", def), 2)
        notify("FOV restaurado para "..tostring(def))
    end)
end)

-- ========= PÁGINA MONITOR =========
safe(function()
    local p = Pages["Monitor"]
    mkTog(p,"Contador de FPS", false, function(on) RH:ToggleFPSCounter(on) end)
    mkTog(p,"Monitor de Ping", false, function(on) RH:TogglePing(on) end)
    mkLbl(p,"O ping depende de Stats.Network. Se indisponível, mostra 'indisponível'.", c3(150,170,200), 11)
    mkSec(p,"Posição do indicador")
    for _, info in ipairs({
        {"Topo Esquerdo", UDim2.new(0,10,0,10), UDim2.new(0,10,0,44)},
        {"Topo Direito", UDim2.new(1,-140,0,10), UDim2.new(1,-140,0,44)},
        {"Baixo Esquerdo", UDim2.new(0,10,1,-74), UDim2.new(0,10,1,-40)},
        {"Baixo Direito", UDim2.new(1,-140,1,-74), UDim2.new(1,-140,1,-40)},
    }) do
        mkBtn2(p, info[1], function()
            FPSGui.Position = info[2]; PingGui.Position = info[3]
            notify("Posição do indicador: "..info[1])
        end)
    end
end)

-- ========= PÁGINA SETTINGS =========
safe(function()
    local p = Pages["Settings"]
    mkSec(p,"Preferências")
    mkBtn2(p,"Aplicar configurações recomendadas", function()
        applyFPS("Medium")
        applyGraphics("Balanced")
        RH:ToggleFPSCounter(true)
        RH:TogglePing(true)
        setFOV(75)
        setActive("fov","FOV: 75",2)
        notify("Configurações recomendadas aplicadas")
    end)
    mkBtn2(p,"Restaurar configurações padrão", function()
        applyFPS("Off"); applyGraphics("Off")
        RH.Flags.RemoveFog = false; RH.Flags.ReduceFog = false; restoreFog(true)
        if Workspace.CurrentCamera then
            local k = tostring(Workspace.CurrentCamera)..".FieldOfView"
            if RH.Original[k] then Workspace.CurrentCamera.FieldOfView = RH.Original[k] end
        end
        RH:ToggleFPSCounter(false); RH:TogglePing(false)
        local ks = {}
        for k in pairs(Entries) do table.insert(ks, k) end
        for _, k in ipairs(ks) do clearActive(k) end
        updStatus(); notify("Configurações padrão restauradas")
    end)
    mkSec(p,"Restauração Completa")
    mkBtn2(p,"Restaurar TODAS as alterações", function()
        safe(function()
            for _, k in ipairs({"Brightness","Ambient","OutdoorAmbient","GlobalShadows","EnvironmentDiffuseScale","EnvironmentSpecularScale","ShadowSoftness","ExposureCompensation","FogStart","FogEnd","FogColor"}) do
                restoreOrig(Lighting, k)
            end
            for _, fx in ipairs(Lighting:GetChildren()) do
                if fx:IsA("PostEffect") then restoreOrig(fx,"Enabled") end
                if fx:IsA("Atmosphere") then
                    for _, p2 in ipairs({"Density","Haze","Glare"}) do restoreOrig(fx, p2) end
                end
            end
            if Workspace.CurrentCamera then
                local k = tostring(Workspace.CurrentCamera)..".FieldOfView"
                if RH.Original[k] then
                    Workspace.CurrentCamera.FieldOfView = RH.Original[k]
                    RH.Original[k] = nil
                end
            end
            RH.Flags.FPSBoost = false; RH.Flags.FPSMode = "Off"; RH.Flags.GraphicsMode = "Off"
            RH.Flags.RemoveFog = false; RH.Flags.ReduceFog = false
            local ks = {}
            for k in pairs(Entries) do table.insert(ks, k) end
            for _, k in ipairs(ks) do clearActive(k) end
            updStatus(); notify("Todas as alterações foram restauradas")
        end)
    end)
    mkBtn2(p,"Fechar RANDOM HUB", function()
        safe(function()
            RH:ToggleFPSCounter(false)
            RH:TogglePing(false)
        end)
        for _, c in ipairs(RH.Connections) do pcall(function() c:Disconnect() end) end
        RH.Connections = {}
        FB:Destroy()
        ScreenGui:Destroy()
    end)
    mkSec(p,"Informação")
    mkLbl(p,"Recursos dependem do jogo, dispositivo e executor.", c3(150,170,200), 11)
end)

-- ========= RESPONSIVO =========
local function updateResp()
    local vp = Workspace.CurrentCamera and Workspace.CurrentCamera.ViewportSize or Vector2.new(1280,720)
    local w = math.clamp(vp.X*0.9, 280, baseW)
    local h = math.clamp(vp.Y*0.8, 320, baseH)
    Main.Size = UDim2.new(0, w, 0, h)
    Main.Position = UDim2.new(0.5, -w/2, 0.5, -h/2)
end
if Workspace.CurrentCamera then
    connect(Workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"), updateResp)
    updateResp()
end

-- ========= ATALHO =========
connect(UIS.InputBegan, function(i, gp)
    if gp then return end
    if i.KeyCode == Enum.KeyCode.RightShift then toggleMenu() end
end)

-- ========= LIMPEZA =========
pcall(function()
    game:BindToClose(function()
        safe(function()
            RH:ToggleFPSCounter(false); RH:TogglePing(false)
            for _, c in ipairs(RH.Connections) do pcall(function() c:Disconnect() end) end
        end)
    end)
end)

print("[RANDOM HUB] carregado com sucesso")
notify("RANDOM HUB carregado com sucesso")
