-- JC HUB - SINTONIA RP (Versão Completa & Corrigida)
-- ID: 125299131646017
-- Autor: Jucarmelo

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Humanoid = Character:WaitForChild("Humanoid")

-- ==========================================
-- CONFIGURAÇÕES E VARIÁVEIS GLOBAIS
-- ==========================================

local Config = {
    FlySpeed = 80, -- Velocidade máxima de voo
    WalkSpeed = 16,
    MaxWalkSpeed = 32, 
    Noclip = false,
    Fly = false,
    GodMode = false,
    AutoRevive = true,
    Color = Color3.fromRGB(0, 120, 255) -- Azul JC Hub
}

-- Conectores de Loop
local FlyConn = nil
local NoclipConn = nil
local SpeedConn = nil
local ReviveConn = nil

-- ==========================================
-- LÓGICA DE ANTI-CHEAT / MOVIMENTO
-- ==========================================

-- 1. NOCLIP ROBUSTO
NoclipConn = RunService.Stepped:Connect(function()
    if not Character then return end
    
    if Config.Noclip then
        for _, part in pairs(Character:GetChildren()) do
            if part:IsA("BasePart") and part.CanCollide == true then
                part.CanCollide = false
            end
        end
    else
        -- Restaura colisão suavemente
        for _, part in pairs(Character:GetChildren()) do
            if part:IsA("BasePart") and part.ClassName ~= "Motor6D" then
                 part.CanCollide = true 
            end
        end
    end
end)

-- 2. FLY & SPEED Otimizado (Bypass Silen)
FlyConn = RunService.RenderStepped:Connect(function(dt)
    if not Character or Character:FindFirstChild("HumanoidRootPart") == nil then return end
    
    local RootPart = Character:FindFirstChild("HumanoidRootPart")
    
    -- Se estiver voando
    if Config.Fly then
        local MoveDir = Humanoid.MoveDirection
        local CameraCFrame = workspace.CurrentCamera.CFrame
        
        -- Vetor de movimento relativo à câmera
        local Forward = CameraCFrame.RightVector:Dot(MoveDir)
        local Right = CameraCFrame.LookVector:Dot(MoveDir)
        
        -- Impulsionamento vertical suave se apertar W/S (opcional)
        local UpForce = 0
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then UpForce = 1 end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then UpForce = -1 end

        local Velocity = Vector3.new(Forward * Config.FlySpeed, UpForce * Config.FlySpeed, Right * Config.FlySpeed)
        
        -- Aplicação direta sem mudar a propriedade WalkSpeed do Humanoid
        RootPart.Velocity = Velocity
        RootPart.AssemblyLinearVelocity = Vector3.new(Velocity.X, RootPart.Velocity.Y, Velocity.Z)
        
        -- Mantém o Humanoid em uma velocidade "normal" para o anti-cheat não detectar
        Humanoid.WalkSpeed = 16 
    else
        -- Modo Normal / Super Speed
        Humanoid.WalkSpeed = Config.WalkSpeed
        
        -- Se quiser speed extra mantendo a física normal:
        -- Humanoid.MaxWalkSpeed = Config.MaxWalkSpeed 
    end
end)

-- 3. GOD MODE
RunService.Heartbeat:Connect(function()
    if Config.GodMode and Humanoid.Health < Humanoid.MaxHealth then
        Humanoid.Health = Humanoid.MaxHealth
    end
end)

-- 4. AUTO REVIVE (Lógica Específica Sintonia RP)
ReviveConn = RunService.Heartbeat:Connect(function()
    if not Config.AutoRevive then return end
    
    if not Character or not Character:FindFirstChild("Humanoid") then return end
    
    if Character.Humanoid.Health <= 0 then
        task.spawn(function()
            task.wait(1)
            local ReplicatedStorage = game:GetService("ReplicatedStorage")
            local OrpheusShared = ReplicatedStorage:FindFirstChild("OrpheusShared")
            
            if OrpheusShared then
                local DeathEvent = OrpheusShared:FindFirstChild("Death")
                if DeathEvent and DeathEvent:IsA("RemoteEvent") then
                    DeathEvent:FireServer("Action1")
                end
            end
        end)
    end
end)

-- ==========================================
-- INTERFACE GRÁFICA (DESIGN AZUL MODERNO)
-- ==========================================

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "JCHubUI"
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Foreground
ScreenGui.DisplayOrder = 100

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 380, 0, 450)
MainFrame.Position = UDim2.new(0.5, -190, 0.5, -225)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 20, 30) -- Fundo escuro azulado
MainFrame.BackgroundTransparency = 0.1
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 12)
Corner.Parent = MainFrame

local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Color3.fromRGB(0, 80, 200) -- Barra azul superior
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleText = Instance.new("TextLabel")
TitleText.Name = "TitleText"
TitleText.Size = UDim2.new(1, -40, 1, 0)
TitleText.Position = UDim2.new(0, 10, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "JC HUB — SINTONIA RP"
TitleText.Font = Enum.Font.GothamBold
TitleText.TextSize = 16
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.Parent = TitleBar

local CloseBtn = Instance.new("TextButton")
CloseBtn.Name = "CloseBtn"
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -35, 0.5, -15)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)
CloseBtn.Parent = TitleBar

-- Dragging Logic
local dragging = false
local dragInput, mousePos, framePos

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        mousePos = input.Position
        framePos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - mousePos
        MainFrame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
    end
end)

-- ==========================================
-- ELEMENTOS DA GUI
-- ==========================================

local Container = Instance.new("ScrollingFrame")
Container.Name = "Container"
Container.Size = UDim2.new(1, -20, 1, -60)
Container.Position = UDim2.new(0, 10, 0, 50)
Container.BackgroundTransparency = 1
Container.ScrollBarThickness = 4
Container.Parent = MainFrame

local ListLayout = Instance.new("UIListLayout")
ListLayout.Padding = UDim.new(0, 8)
ListLayout.SortOrder = Enum.SortOrder.LayoutOrder
ListLayout.Parent = Container

-- Helper para criar botões bonitos
local function CreateButton(name, text, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.BackgroundColor3 = Color3.fromRGB(25, 35, 50)
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(200, 230, 255)
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 14
    btn.Parent = Container
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = btn
    
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color = Color3.fromRGB(0, 120, 255)
    btnStroke.Transparency = 0.5
    btnStroke.Parent = btn

    btn.MouseButton1Click:Connect(function()
        callback()
        -- Efeito visual de clique
        btn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
        task.wait(0.1)
        btn.BackgroundColor3 = Color3.fromRGB(25, 35, 50)
    end)
    
    return btn
end

CreateButton("Velocidade Turbo", "Ativar Super Speed (Bypass)", function()
    Config.WalkSpeed = 45
    Config.MaxWalkSpeed = 45
    print("[JC Hub] Speed Ativado")
end)

CreateButton("Voo Suave", "Toggle Voo", function()
    Config.Fly = not Config.Fly
    if Config.Fly then
        print("[JC Hub] Voo Ligado")
    else
        print("[JC Hub] Vvo Desligado")
    end
end)

CreateButton("Noclip Total", "Ativar/Desativar Noclip", function()
    Config.Noclip = not Config.Noclip
    print("[JC Hub] Noclip: " .. tostring(Config.Noclip))
end)

CreateButton("God Mode", "Invencibilidade Simples", function()
    Config.GodMode = not Config.GodMode
    if Config.GodMode then
        Humanoid.Health = 1000
        print("[JC Hub] God Mode ON")
    else
        Humanoid.Health = 100
        print("[JC Hub] God Mode OFF")
    end
end)

CreateButton("Auto Revive", "Reaviver Automático", function()
    Config.AutoRevive = not Config.AutoRevive
    print("[JC Hub] Auto Revive: " .. tostring(Config.AutoRevive))
end)

CreateButton("Resetar", "Voltar ao Normal", function()
    Config.Fly = false
    Config.Noclip = false
    Config.WalkSpeed = 16
    HumanoidRootPart.Velocity = Vector3.new(0,0,0)
    print("[JC Hub] Resetado")
end)

-- Ajuste final do Scroll
ListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Container.CanvasSize = UDim2.new(0, 0, 0, ListLayout.AbsoluteContentSize.Y)
end)

ScreenGui.Parent = Players.LocalPlayer.PlayerGui
