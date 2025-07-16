local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações iniciais
local AimlockEnabled = false
local TeamCheck = false -- Desativado para testes
local currentFOV = 100
local maxFOV = 300
local minFOV = 50
local Smoothness = 0.2

-- Criar a GUI completa
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AimlockUI"
ScreenGui.Parent = game:GetService("CoreGui")
ScreenGui.ResetOnSpawn = false

-- Container principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.Size = UDim2.new(0, 120, 0, 80)
MainFrame.Position = UDim2.new(1, -125, 0, 5)
MainFrame.AnchorPoint = Vector2.new(1, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BackgroundTransparency = 0.5
MainFrame.BorderSizePixel = 0

-- Botão ON/OFF
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Parent = MainFrame
ToggleButton.Size = UDim2.new(0.9, 0, 0.4, 0)
ToggleButton.Position = UDim2.new(0.05, 0, 0.05, 0)
ToggleButton.Text = "OFF"
ToggleButton.TextSize = 14
ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.ZIndex = 10
ToggleButton.BorderSizePixel = 0

-- Barra de regulagem do FOV
local FOVSlider = Instance.new("Frame")
FOVSlider.Name = "FOVSlider"
FOVSlider.Parent = MainFrame
FOVSlider.Size = UDim2.new(0.9, 0, 0.1, 0)
FOVSlider.Position = UDim2.new(0.05, 0, 0.55, 0)
FOVSlider.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
FOVSlider.BorderSizePixel = 0

local SliderFill = Instance.new("Frame")
SliderFill.Name = "SliderFill"
SliderFill.Parent = FOVSlider
SliderFill.Size = UDim2.new((currentFOV - minFOV)/(maxFOV - minFOV), 1, 1, 0)
SliderFill.Position = UDim2.new(0, 0, 0, 0)
SliderFill.BackgroundColor3 = Color3.fromRGB(0, 162, 255)
SliderFill.BorderSizePixel = 0

local SliderButton = Instance.new("TextButton")
SliderButton.Name = "SliderButton"
SliderButton.Parent = FOVSlider
SliderButton.Size = UDim2.new(0, 10, 1.5, 0)
SliderButton.Position = UDim2.new((currentFOV - minFOV)/(maxFOV - minFOV), -5, 0, -2)
SliderButton.Text = ""
SliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
SliderButton.BorderSizePixel = 0
SliderButton.ZIndex = 11

-- Texto do valor do FOV
local FOVText = Instance.new("TextLabel")
FOVText.Name = "FOVText"
FOVText.Parent = MainFrame
FOVText.Size = UDim2.new(0.9, 0, 0.2, 0)
FOVText.Position = UDim2.new(0.05, 0, 0.7, 0)
FOVText.Text = "FOV: "..currentFOV
FOVText.TextSize = 12
FOVText.BackgroundTransparency = 1
FOVText.TextColor3 = Color3.fromRGB(255, 255, 255)
FOVText.Font = Enum.Font.Gotham
FOVText.TextXAlignment = Enum.TextXAlignment.Left

-- Círculo de visualização do FOV
local FOVCircle = Instance.new("Frame")
FOVCircle.Name = "FOVCircle"
FOVCircle.Parent = ScreenGui
FOVCircle.Size = UDim2.new(0, currentFOV*2, 0, currentFOV*2)
FOVCircle.Position = UDim2.new(0.5, -currentFOV, 0.5, -currentFOV)
FOVCircle.BackgroundTransparency = 1
FOVCircle.Visible = false

local CircleOutline = Instance.new("UICorner")
CircleOutline.CornerRadius = UDim.new(1, 0)
CircleOutline.Parent = FOVCircle

local CircleBorder = Instance.new("UIStroke")
CircleBorder.Parent = FOVCircle
CircleBorder.Color = Color3.fromRGB(255, 0, 0)
CircleBorder.Thickness = 1
CircleBorder.Transparency = 0.7

-- Função para atualizar o FOV
local function UpdateFOV(newFOV)
    currentFOV = math.clamp(newFOV, minFOV, maxFOV)
    FOVText.Text = "FOV: "..math.floor(currentFOV)
    SliderFill.Size = UDim2.new((currentFOV - minFOV)/(maxFOV - minFOV), 0, 1, 0)
    SliderButton.Position = UDim2.new((currentFOV - minFOV)/(maxFOV - minFOV), -5, 0, -2)
    
    if FOVCircle.Visible then
        FOVCircle.Size = UDim2.new(0, currentFOV*2, 0, currentFOV*2)
        FOVCircle.Position = UDim2.new(0.5, -currentFOV, 0.5, -currentFOV)
    end
end

-- Controle do slider
local sliding = false

local function StartSlide(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        sliding = true
        UpdateSlide(input)
    end
end

local function EndSlide(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        sliding = false
    end
end

local function UpdateSlide(input)
    if sliding then
        local relativeX = (input.Position.X - FOVSlider.AbsolutePosition.X) / FOVSlider.AbsoluteSize.X
        relativeX = math.clamp(relativeX, 0, 1)
        local newFOV = minFOV + (maxFOV - minFOV) * relativeX
        UpdateFOV(newFOV)
    end
end

SliderButton.InputBegan:Connect(StartSlide)
SliderButton.InputEnded:Connect(EndSlide)
UserInputService.InputChanged:Connect(function(input)
    if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        UpdateSlide(input)
    end
end)

-- Função para alternar o aimlock e o círculo
local function ToggleAimlock()
    AimlockEnabled = not AimlockEnabled
    
    if AimlockEnabled then
        ToggleButton.Text = "ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(50, 255, 50)
        FOVCircle.Visible = true
        print("Aimlock ATIVADO")
    else
        ToggleButton.Text = "OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
        FOVCircle.Visible = false
        print("Aimlock DESATIVADO")
    end
end

ToggleButton.MouseButton1Click:Connect(ToggleAimlock)
if UserInputService.TouchEnabled then
    ToggleButton.TouchTap:Connect(ToggleAimlock)
end

-- Função para encontrar o jogador mais próximo
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = currentFOV
    
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

-- Conexão com o loop do jogo
RunService.RenderStepped:Connect(Aimlock)

-- Atualização inicial
UpdateFOV(currentFOV)
