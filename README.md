-- JC HUB - SINTONIA RP
-- Design: Blue Modern
-- Author: Jucarmelo
-- Optimized for Silen Anti-Cheat Bypass

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Humanoid = Character:WaitForChild("Humanoid")

-- Configurações Globais
local Config = {
    FlySpeed = 50,
    WalkSpeed = 16,
    MaxWalkSpeed = 32, -- Limitado para não ser óbvio pro anti-cheat
    Noclip = false,
    Fly = false,
    GodMode = false,
    Color = Color3.fromRGB(0, 120, 255) -- Azul JC Hub
}

-- ==========================================
-- SISTEMA DE GUI (DESIGN AZUL MODERNO)
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
-- FUNÇÕES DO JOGO (BYPASS SILEN/ANTICHEAT)
-- ==========================================

-- 1. NOCLIP ROBUSTO
RunService.Stepped:Connect(function()
    if not Character or Character:FindFirstChild("HumanoidRootPart") == nil then return end
    
    if Config.Noclip then
        for _, part in pairs(Character:GetChildren()) do
            if part:IsA("BasePart") and part.CanCollide == true then
                part.CanCollide = false
            end
        end
    else
        -- Restaura colisão suavemente se desligar
        for _, part in pairs(Character:GetChildren()) do
            if part:IsA("BasePart") and part.ClassName ~= "Motor6D" then
                 -- Pequeno delay ou lógica simples para não travar
                 part.CanCollide = true 
            end
        end
    end
end)

-- 2. FLY & SPEED Otimizado (Anti-Silent Kick)
local FlyConn = RunService.RenderStepped:Connect(function(dt)
    if not Config.Fly then return end
    
    local RootPart = Character:FindFirstChild("HumanoidRootPart")
    if not RootPart then return end

    local MoveDir = Humanoid.MoveDirection
    local CameraCFrame = workspace.CurrentCamera.CFrame
    
    -- Vetor de movimento relativo à câmera
    local Forward = CameraCFrame.RightVector:Dot(MoveDir)
    local Right = CameraCFrame.LookVector:Dot(MoveDir)
    
    local Velocity = Vector3.new(Forward, 0, Right) * Config.FlySpeed
    
    -- Impulsionamento vertical suave
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then -- W para subir (exemplo)
         Velocity = Vector3.new(Forward * Config.FlySpeed, Config.FlySpeed, Right * Config.FlySpeed)
    end

    -- Aplicação suave sem mudar a velocidade máxima do Humanoid (burla check básico)
    RootPart.Velocity = Velocity
    RootPart.AssemblyLinearVelocity = Vector3.new(Velocity.X, RootPart.Velocity.Y, Velocity.Z)
end)

-- 3. SUPER WALKSPEED (Modificado para passar despercebido)
Humanoid.WalkSpeed = Config.WalkSpeed
Humanoid.MaxWalkSpeed = Config.MaxWalkSpeed

function SetWalkSpeed(speed)
    Config.WalkSpeed = speed
    Humanoid.WalkSpeed = speed
    Humanoid.MaxWalkSpeed = math.min(speed, 32) -- Truque: limita o Max para o anticheat não pegar
end

-- ==========================================
-- INTERFACE GRÁFICA (CONTROLES)
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
    SetWalkSpeed(Config.WalkSpeed)
    print("[JC Hub] Speed Ativado")
end)

CreateButton("Voo Suave", "Toggle Voo", function()
    Config.Fly = not Config.Fly
    if Config.Fly then
        print("[JC Hub] Voo Ligado")
    else
        FlyConn:Disconnect()
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
        Humanoid.MaxHealth = 1000
        print("[JC Hub] God Mode ON")
    else
        Humanoid.Health = 100
        print("[JC Hub] God Mode OFF")
    end
end)

CreateButton("Resetar", "Voltar ao Normal", function()
    Config.Fly = false
    Config.Noclip = false
    Config.WalkSpeed = 16
    SetWalkSpeed(16)
    FlyConn:Disconnect()
    HumanoidRootPart.Velocity = Vector3.new(0,0,0)
    print("[JC Hub] Resetado")
end)

-- Ajuste final do Scroll
ListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    Container.CanvasSize = UDim2.new(0, 0, 0, ListLayout.AbsoluteContentSize.Y)
end)

ScreenGui.Parent = Players.LocalPlayer.PlayerGui
