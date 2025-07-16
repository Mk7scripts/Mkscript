local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local AimlockEnabled = false
local TeamCheck = true
local FOV = 100
local Smoothness = 0.2 -- 0 = instantâneo, 1 = muito lento

-- Criar a GUI
local ScreenGui = Instance.new("ScreenGui")
local TextButton = Instance.new("TextButton")

ScreenGui.Name = "AimlockUI"
ScreenGui.Parent = game.CoreGui
ScreenGui.ResetOnSpawn = false

TextButton.Name = "AimlockButton"
TextButton.Parent = ScreenGui
TextButton.Size = UDim2.new(0, 100, 0, 50)
TextButton.Position = UDim2.new(0.5, -50, 0.9, -25)
TextButton.Text = "OFF"
TextButton.TextScaled = true
TextButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
TextButton.TextColor3 = Color3.fromRGB(255, 255, 255)
TextButton.Font = Enum.Font.GothamBold
TextButton.ZIndex = 10

-- Adaptar para mobile
if UserInputService.TouchEnabled then
    TextButton.Size = UDim2.new(0, 150, 0, 75)
    TextButton.Position = UDim2.new(0.5, -75, 0.9, -37.5)
    TextButton.TextSize = 28
end

-- Função para encontrar o jogador mais próximo
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = FOV
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            if TeamCheck and player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then
                continue
            end
            
            local character = player.Character
            local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
            
            if humanoidRootPart then
                local screenPoint = Camera:WorldToScreenPoint(humanoidRootPart.Position)
                if screenPoint.Z > 0 then
                    local viewportSize = Camera.ViewportSize
                    local center = Vector2.new(viewportSize.X/2, viewportSize.Y/2)
                    local distance = (center - Vector2.new(screenPoint.X, screenPoint.Y)).Magnitude
                    
                    if distance < shortestDistance then
                        closestPlayer = player
                        shortestDistance = distance
                    end
                end
            end
        end
    end
    
    return closestPlayer
end

-- Função principal do Aimlock
local function Aimlock()
    if not AimlockEnabled or not LocalPlayer.Character then return end
    
    local target = GetClosestPlayer()
    if target and target.Character then
        local humanoidRootPart = target.Character:FindFirstChild("HumanoidRootPart")
        if humanoidRootPart then
            local newCFrame = CFrame.new(Camera.CFrame.Position, humanoidRootPart.Position)
            Camera.CFrame = Camera.CFrame:Lerp(newCFrame, 1 - Smoothness)
        end
    end
end

-- Alternar Aimlock quando o botão for clicado
TextButton.MouseButton1Click:Connect(function()
    AimlockEnabled = not AimlockEnabled
    
    if AimlockEnabled then
        TextButton.Text = "ON"
        TextButton.BackgroundColor3 = Color3.fromRGB(50, 255, 50)
    else
        TextButton.Text = "OFF"
        TextButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    end
end)

-- Também funciona com toque na tela (mobile)
TextButton.TouchTap:Connect(function()
    AimlockEnabled = not AimlockEnabled
    
    if AimlockEnabled then
        TextButton.Text = "ON"
        TextButton.BackgroundColor3 = Color3.fromRGB(50, 255, 50)
    else
        TextButton.Text = "OFF"
        TextButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
    end
end)

-- Conexão com o loop do jogo
RunService.RenderStepped:Connect(Aimlock)
