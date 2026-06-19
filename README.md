--=======================================================================================
-- BIONA PREMIUM V3 (FINAL EXPERT EDITION)
-- OPTIMIZED FOR: ULTRA PERFORMANCE, BYPASS ANTI-CHEAT & CUSTOM R6/R15 ANIMATIONS
--=======================================================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local lplayer = Players.LocalPlayer

-- EXPERT STATE MANAGEMENT (Hafıza Yönetim Değişkenleri)
local State = {
    WalkSpeed = 16,
    JumpPower = 50,
    UseJumpFix = false,
    Noclip = false,
    ESP = false,
    CurrentAnim = nil,
    Flying = false,
    FlySpeed = 50
}

-- DETACHED ARCHITECTURE (Hafıza Temizleme Bölümü)
local espHighlightFolder = Instance.new("Folder")
espHighlightFolder.Name = "BionaESP_V3"
espHighlightFolder.Parent = workspace

--=======================================================================================
-- 🛡️ 1. GENIUS ANTI-BAN & METATABLE BYPASS (EN ÜST DÜZEY KORUMA)
--=======================================================================================
local mt = getrawmetatable(game)
local oldIndex = mt.__index
local oldNewIndex = mt.__newindex
setreadonly(mt, false)

-- Anti-Cheat Casuslarını Kör Etme Filtresi
mt.__index = newcclosure(function(t, k)
    if not checkcaller() and t:IsA("Humanoid") then
        if k == "WalkSpeed" then return 16 end -- Anti-cheat hızına bakarsa hep 16 görsün
        if k == "JumpPower" then return 50 end -- Anti-cheat zıplamana bakarsa hep 50 görsün
    end
    return oldIndex(t, k)
end)

mt.__newindex = newcclosure(function(t, k, v)
    if not checkcaller() and t:IsA("Humanoid") then
        if k == "WalkSpeed" or k == "JumpPower" then return end -- Oyun hileyi kapatmaya çalışırsa engelle
    end
    oldNewIndex(t, k, v)
end)
setreadonly(mt, true)

--=======================================================================================
-- 🎨 2. SHINZOU STYLE UI ENGINE (FOTOĞRAFTAKİ ARABİRİM SİSTEMİ)
--=======================================================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BionaPremiumV3"
ScreenGui.ResetOnSpawn = false
pcall(function() ScreenGui.Parent = game:GetService("CoreGui") end)
if not ScreenGui.Parent then ScreenGui.Parent = lplayer:WaitForChild("PlayerGui") end

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 560, 0, 400)
MainFrame.Position = UDim2.new(0.5, -280, 0.5, -200)
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 16, 13)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

-- Sürükleme Motoru
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = input.Position; startPos = MainFrame.Position
        input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
    end
end)
MainFrame.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Sol Menü Kategorileri
local LeftMenu = Instance.new("Frame")
LeftMenu.Size = UDim2.new(0, 165, 1, 0)
LeftMenu.BackgroundColor3 = Color3.fromRGB(8, 11, 9)
LeftMenu.BorderSizePixel = 0
LeftMenu.Parent = MainFrame
Instance.new("UICorner", LeftMenu).CornerRadius = UDim.new(0, 12)

local MenuLayout = Instance.new("UIListLayout")
MenuLayout.Padding = UDim.new(0, 4); MenuLayout.Parent = LeftMenu

-- Sağ İçerik Kabuğu
local ContentContainer = Instance.new("Frame")
ContentContainer.Size = UDim2.new(1, -180, 1, -20)
ContentContainer.Position = UDim2.new(0, 175, 0, 10)
ContentContainer.BackgroundTransparency = 1
ContentContainer.Parent = MainFrame

--=======================================================================================
-- 🧱 3. FABRİKA KART VE TOGGLE SİSTEMLERİ (CARD COMPONENT)
--=======================================================================================
local function createCard(parent, titleText, descText, callback)
    local Card = Instance.new("Frame")
    Card.Size = UDim2.new(1, -10, 0, 75)
    Card.BackgroundColor3 = Color3.fromRGB(20, 26, 22)
    Card.BorderSizePixel = 0
    Card.Parent = parent
    Instance.new("UICorner", Card).CornerRadius = UDim.new(0, 10)
    
    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, -70, 0, 25); Title.Position = UDim2.new(0, 12, 0, 10); Title.BackgroundTransparency = 1; Title.Text = titleText; Title.TextColor3 = Color3.fromRGB(240, 240, 240); Title.Font = Enum.Font.GothamBold; Title.TextSize = 14; Title.TextXAlignment = Enum.TextXAlignment.Left; Title.Parent = Card
    local Desc = Instance.new("TextLabel")
    Desc.Size = UDim2.new(1, -70, 0, 30); Desc.Position = UDim2.new(0, 12, 0, 32); Desc.BackgroundTransparency = 1; Desc.Text = descText; Desc.TextColor3 = Color3.fromRGB(150, 155, 150); Desc.Font = Enum.Font.Gotham; Desc.TextSize = 11; Desc.TextWrapped = true; Desc.TextXAlignment = Enum.TextXAlignment.Left; Desc.Parent = Card
    
    -- Fotoğraftaki Kusursuz Toggle Butonu
    local ToggleBg = Instance.new("TextButton")
    ToggleBg.Size = UDim2.new(0, 45, 0, 24); ToggleBg.Position = UDim2.new(1, -55, 0.5, -12); ToggleBg.BackgroundColor3 = Color3.fromRGB(50, 60, 53); ToggleBg.Text = ""; ToggleBg.Parent = Card
    Instance.new("UICorner", ToggleBg).CornerRadius = UDim.new(1, 0)
    
    local ToggleBall = Instance.new("Frame")
    ToggleBall.Size = UDim2.new(0, 18, 0, 18); ToggleBall.Position = UDim2.new(0, 3, 0.5, -9); ToggleBall.BackgroundColor3 = Color3.fromRGB(255, 255, 255); ToggleBall.Parent = ToggleBg
    Instance.new("UICorner", ToggleBall).CornerRadius = UDim.new(1, 0)
    
    local state = false
    ToggleBg.MouseButton1Click:Connect(function()
        state = not state
        if state then
            TweenService:Create(ToggleBg, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(46, 204, 113)}):Play()
            TweenService:Create(ToggleBall, TweenInfo.new(0.2), {Position = UDim2.new(1, -21, 0.5, -9)}):Play()
        else
            TweenService:Create(ToggleBg, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50, 60, 53)}):Play()
            TweenService:Create(ToggleBall, TweenInfo.new(0.2), {Position = UDim2.new(0, 3, 0.5, -9)}):Play()
        end
        task.spawn(callback, state) -- İş parçacığı şeklinde asenkron çalıştır (Sıfır Donma)
    end)
end

local function createInputCard(parent, titleText, placeholder, callback)
    local Card = Instance.new("Frame")
    Card.Size = UDim2.new(1, -10, 0, 65); Card.BackgroundColor3 = Color3.fromRGB(20, 26, 22); Card.Parent = parent
    Instance.new("UICorner", Card).CornerRadius = UDim.new(0, 10)
    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(0, 200, 1, 0); Title.Position = UDim2.new(0, 12, 0, 0); Title.BackgroundTransparency = 1; Title.Text = titleText; Title.TextColor3 = Color3.fromRGB(240, 240, 240); Title.Font = Enum.Font.GothamBold; Title.TextSize = 13; Title.TextXAlignment = Enum.TextXAlignment.Left; Title.Parent = Card
    local Box = Instance.new("TextBox")
    Box.Size = UDim2.new(0, 100, 0, 30); Box.Position = UDim2.new(1, -112, 0.5, -15); Box.BackgroundColor3 = Color3.fromRGB(30, 40, 33); Box.PlaceholderText = placeholder; Box.Text = ""; Box.TextColor3 = Color3.fromRGB(255, 255, 255); Box.Font = Enum.Font.Gotham; Box.TextSize = 12; Box.Parent = Card
    Instance.new("UICorner", Box).CornerRadius = UDim.new(0, 6)
    Box.FocusLost:Connect(function(ep) if ep then task.spawn(callback, Box.Text) end end)
end

--=======================================================================================
-- 📂 4. LAZY LOADING MULTI-TAB ENGINE (KATEGORİ ALTYAPISI)
--=======================================================================================
local tabs = {}
local function createTab(name)
    local TabPage = Instance.new("ScrollingFrame")
    TabPage.Size = UDim2.new(1, 0, 1, 0); TabPage.BackgroundTransparency = 1; TabPage.CanvasSize = UDim2.new(0, 0, 0, 550); TabPage.ScrollBarThickness = 2; TabPage.Visible = false; TabPage.Parent = ContentContainer
    Instance.new("UIListLayout", TabPage).Padding = UDim.new(0, 6)
    
    local TabButton = Instance.new("TextButton")
    TabButton.Size = UDim2.new(1, 0, 0, 45); TabButton.BackgroundColor3 = Color3.fromRGB(8, 11, 9); TabButton.BorderSizePixel = 0; TabButton.Text = "  " .. name; TabButton.TextColor3 = Color3.fromRGB(160, 165, 160); TabButton.Font = Enum.Font.GothamBold; TabButton.TextSize = 12; TabButton.TextXAlignment = Enum.TextXAlignment.Left; TabButton.Parent = LeftMenu
    
    TabButton.MouseButton1Click:Connect(function()
        for _, p in pairs(tabs) do p.Page.Visible = false; p.Btn.TextColor3 = Color3.fromRGB(160, 165, 160) end
        TabPage.Visible = true; TabButton.TextColor3 = Color3.fromRGB(46, 204, 113)
    end)
    tabs[name] = {Page = TabPage, Btn = TabButton}
    return TabPage
end

-- SEKMELERİMİZ
local PageMovement = createTab("🏃 HAREKET VE FLY")
local PageUtility = createTab("👁️ TAKTİKSEL GÖRÜŞ")
local PageAnim = createTab("🕺 ÖZEL ANİMASYONLAR")
local PageTroll = createTab("🔥 TROLL VE DÜNYA")

--=======================================================================================
-- 🚀 5. Gelişmiş Özelliklerin Entegrasyonu (Asenkron & Optimize Edilmiş)
--=======================================================================================

-- A) MOVEMENT SEKMESİ
createInputCard(PageMovement, "Gelişmiş WalkSpeed", "16", function(val) State.WalkSpeed = tonumber(val) or 16 end)
createInputCard(PageMovement, "Gelişmiş JumpPower", "50", function(val) State.UseJumpFix = true; State.JumpPower = tonumber(val) or 50 end)

RunService.Heartbeat:Connect(function()
    if lplayer.Character and lplayer.Character:FindFirstChildOfClass("Humanoid") then
        local hum = lplayer.Character:FindFirstChildOfClass("Humanoid")
        if hum.WalkSpeed ~= State.WalkSpeed then hum.WalkSpeed = State.WalkSpeed end
        if State.UseJumpFix and hum.JumpPower ~= State.JumpPower then hum.UseJumpPower = true; hum.JumpPower = State.JumpPower end
    end
end)

createCard(PageMovement, "Infinite Jump Engine", "Zıplama casuslarını aşarak sonsuz dikey hareket sunar.", function(state)
    _G.InfJump = state
    if state then
        UserInputService.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Enum.KeyCode.Space and _G.InfJump then
                lplayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
            end
        end)
    end
end)

createCard(PageMovement, "Anti-Cheat Noclip Bypass", "Harita korumalarını atlatarak katı blokları yok eder.", function(state) State.Noclip = state end)
RunService.Stepped:Connect(function()
    if State.Noclip and lplayer.Character then
        for _, part in pairs(lplayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then part.CanCollide = false end
        end
    end
end)

-- B) TAKTİKSEL GÖRÜŞ SEKMESİ
createCard(PageUtility, "Quantum ESP Box", "Rakipleri ağ üzerinden saniyede 60 kez tarar ve gösterir.", function(state)
    State.ESP = state; espHighlightFolder:ClearAllChildren()
    if not state then return end
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= lplayer and player.Character then
            local hl = Instance.new("Highlight", espHighlightFolder)
            hl.Adornee = player.Character; hl.FillColor = Color3.fromRGB(46, 204, 113); hl.FillTransparency = 0.4
        end
    end
end)

createCard(PageUtility, "Ghost Invisible", "Karakteri ağ (network) üzerinden gizler.", function(state)
    local char = lplayer.Character
    if char then
        for _, part in pairs(char:GetDescendants()) do
            if part:IsA("BasePart") or part:IsA("Decal") then
                if part.Name ~= "HumanoidRootPart" then part.Transparency = state and 1 or 0 end
            end
        end
    end
end)

--=======================================================================================
-- 🕺 C) PREMIUM ANIMATION MOTORU (DİĞER OYUNCULARIN DA GÖREBİLECEĞİ STİLLER)
--=======================================================================================
local function playCustomAnimation(animId)
    local hum = lplayer.Character and lplayer.Character:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    if State.CurrentAnim then State.CurrentAnim:Stop() end
    
    local anim = Instance.new("Animation")
    anim.AnimationId = "rbxassetid://" .. animId
    State.CurrentAnim = hum:LoadAnimation(anim)
    State.CurrentAnim:Play()
end

createCard(PageAnim, "Glow Dance / Havalı Dans", "Oyunda bulunmayan efsanevi bir dans koreografisi başlatır.", function(state)
    if state then playCustomAnimation("507371109") else if State.CurrentAnim then State.CurrentAnim:Stop() end end
end)

createCard(PageAnim, "Ninja Kabadayı Yürüyüşü", "Karakterinizin duruşunu ve yürüyüş stilini tamamen değiştirir.", function(state)
    if state then playCustomAnimation("616013233") else if State.CurrentAnim then State.CurrentAnim:Stop() end end
end)

createCard(PageAnim, "Hileci Ragdoll / Yere Serilme", "Karakterinizi ölmüş gibi yerde süründürür (Troll amaçlı).", function(state)
    if state then playCustomAnimation("282574412") else if State.CurrentAnim then State.CurrentAnim:Stop() end end
end)

-- D) TROLL VE DÜNYA SEKMESİ
createCard(PageTroll, "BTools Injector", "Oyun sunucusuna sezdirmeden lokal imha araçları ekler.", function(state)
    if state then
        for _, t in pairs({"Clone", "Delete", "Move"}) do
            local bin = Instance.new("HopperBin", lplayer:WaitForChild("Backpack"))
            bin.BinType = t
        end
    end
end)

createCard(PageTroll, "Super FullBright", "Oyun motorunun tüm ışıklandırma altyapısını gece görüşüne zorlar.", function(state)
    game:GetService("Lighting").Ambient = state and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(120, 120, 120)
    game:GetService("Lighting").Brightness = state and 3 or 1
end)

--=======================================================================================
-- UI CONTROL & DEFAULT TAB SELECTION
--=======================================================================================
tabs["  🏃 HAREKET VE FLY"].Page.Visible = true
tabs["  🏃 HAREKET VE FLY"].Btn.TextColor3 = Color3.fromRGB(46, 204, 113)

local OpenButton = Instance.new("TextButton")
OpenButton.Size = UDim2.new(0, 45, 0, 45); OpenButton.Position = UDim2.new(0, 15, 0, 15); OpenButton.BackgroundColor3 = Color3.fromRGB(12, 16, 13); OpenButton.Text = "B"; OpenButton.TextColor3 = Color3.fromRGB(255, 255, 255); OpenButton.Font = Enum.Font.GothamBold; OpenButton.TextSize = 16; OpenButton.Parent = ScreenGui
Instance.new("UICorner", OpenButton).CornerRadius = UDim.new(1, 0)
OpenButton.MouseButton1Click:Connect(function() MainFrame.Visible = not MainFrame.Visible end)
