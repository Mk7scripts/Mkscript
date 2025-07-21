-- Acessa os serviços necessários
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Player = Players.LocalPlayer
local PlayerGui = Player.PlayerGui
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Cria a ScreenGui para conter a UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CustomButtonUI"
screenGui.Parent = PlayerGui
screenGui.ResetOnSpawn = true

-- Cria o Frame para conter o botão
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 150, 0, 50) -- Tamanho do frame igual ao botão
frame.Position = UDim2.new(0.5, -75, 0.5, -25) -- Centralizado na tela
frame.BackgroundTransparency = 1 -- Fundo transparente
frame.Parent = screenGui

-- Cria o botão
local button = Instance.new("TextButton")
button.Name = "CoolButton"
button.Size = UDim2.new(0, 150, 0, 50) -- Tamanho do botão
button.Position = UDim2.new(0, 0, 0, 0) -- Alinhado ao frame
button.BackgroundColor3 = Color3.fromRGB(0, 0, 0) -- Cor preta
button.Text = "OFF" -- Texto inicial
button.TextColor3 = Color3.fromRGB(255, 255, 255) -- Texto branco
button.TextSize = 20
button.Font = Enum.Font.GothamBold -- Fonte estilizada
button.Parent = frame

-- Adiciona UICorner para cantos arredondados no botão
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 12) -- Raio dos cantos
uiCorner.Parent = button

-- Adiciona UIStroke para borda roxa no botão
local uiStroke = Instance.new("UIStroke")
uiStroke.Color = Color3.fromRGB(128, 0, 128) -- Roxo
uiStroke.Thickness = 2 -- Espessura da borda
uiStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
uiStroke.Parent = button

-- Efeito de hover (muda a transparência do fundo)
button.MouseEnter:Connect(function()
    button.BackgroundTransparency = 0.2
end)

button.MouseLeave:Connect(function()
    button.BackgroundTransparency = 0
end)

-- Animação ao clicar
local tweenInfo = TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local function animateClick()
    local tweenDown = TweenService:Create(button, tweenInfo, {Size = UDim2.new(0, 145, 0, 45)})
    local tweenUp = TweenService:Create(button, tweenInfo, {Size = UDim2.new(0, 150, 0, 50)})
    tweenDown:Play()
    tweenDown.Completed:Connect(function()
        tweenUp:Play()
    end)
end

-- Cria o círculo FOV roxo
local fovCircle = Instance.new("Frame")
fovCircle.Name = "FOVCircle"
fovCircle.Size = UDim2.new(0, 150, 0, 150) -- Tamanho reduzido do círculo (ajustável)
fovCircle.Position = UDim2.new(0.5, -75, 0.5, -75) -- Ajustado para centralizar o círculo menor
fovCircle.BackgroundColor3 = Color3.fromRGB(128, 0, 128) -- Roxo
fovCircle.BackgroundTransparency = 0.7 -- Transparência para visibilidade
fovCircle.Parent = screenGui

local uiCornerCircle = Instance.new("UICorner")
uiCornerCircle.CornerRadius = UDim.new(1, 0) -- Faz o Frame ser circular
uiCornerCircle.Parent = fovCircle

-- Inicialmente esconde o círculo
fovCircle.Visible = false

-- Variáveis para controlar o estado
local isAimlockActive = false
local aimlockConnection = nil
local currentTarget = nil

-- Função para encontrar o alvo dentro do círculo FOV
local function findTarget()
    local maxDistance = 200 -- Aumentado de 100 para 200 studs (ajustável)
    local target = nil
    local shortestDistance = math.huge

  if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return nil -- Sai se o personagem local não estiver pronto
    end

   local circleCenter = Vector2.new(
        fovCircle.AbsolutePosition.X + fovCircle.AbsoluteSize.X / 2,
        fovCircle.AbsolutePosition.Y + fovCircle.AbsoluteSize.Y / 2
    )
    local circleRadius = fovCircle.AbsoluteSize.X / 2

 for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local character = player.Character
            local humanoidRootPart = character.HumanoidRootPart
            local screenPoint, onScreen = Camera:WorldToViewportPoint(humanoidRootPart.Position)
            local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - circleCenter).Magnitude
            local playerDistance = (LocalPlayer.Character.HumanoidRootPart.Position - humanoidRootPart.Position).Magnitude

 if onScreen and distance <= circleRadius and playerDistance <= maxDistance then
                if playerDistance < shortestDistance then
                    shortestDistance = playerDistance
                    target = humanoidRootPart
                    print("Target found: " .. (player.Name or "Unknown")) -- Depuração
                end
            end
        end
    end
    return target
end

-- Função para suavizar a transição do aimlock
local function smoothAimlock(target)
    if target and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local humanoidRootPart = LocalPlayer.Character.HumanoidRootPart
        local tweenInfoSmooth = TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out) -- Transição suave
        local tweenCamera = TweenService:Create(Camera, tweenInfoSmooth, {CFrame = CFrame.new(Camera.CFrame.Position, target.Position)})
        local tweenCharacter = TweenService:Create(humanoidRootPart, tweenInfoSmooth, {CFrame = CFrame.new(humanoidRootPart.Position, target.Position)})
        tweenCamera:Play()
        tweenCharacter:Play()
    end
end

-- Função para alternar o aimlock e o círculo FOV
local function toggleAimlockAndFOV()
    if not isAimlockActive then
        -- Ativa o aimlock e o círculo FOV
        isAimlockActive = true
        button.Text = "ON"
        fovCircle.Visible = true
        
   aimlockConnection = RunService.RenderStepped:Connect(function()
            local newTarget = findTarget()
            if newTarget and newTarget ~= currentTarget then
                currentTarget = newTarget
                smoothAimlock(newTarget)
                print("Switching to new target") -- Depuração
            elseif not newTarget and currentTarget then
                currentTarget = nil
                print("No target in range") -- Depuração
            elseif currentTarget then
                smoothAimlock(currentTarget)
            end
        end)
    else
    
   isAimlockActive = false
        button.Text = "OFF"
        fovCircle.Visible = false
        currentTarget = nil
        
   if aimlockConnection then
            aimlockConnection:Disconnect()
            aimlockConnection = nil
        end
    end
end

button.MouseButton1Click:Connect(function()
    print("Botão clicado por " .. Player.Name .. "!")
    animateClick()
    toggleAimlockAndFOV()
end)

-- Funcionalidade de arrastar
local dragging = false
local dragStart = nil
local startPosition = nil

local function update(input)
    local delta = input.Position - dragStart
    frame.Position = UDim2.new(startPosition.X.Scale, startPosition.X.Offset + delta.X, startPosition.Y.Scale, startPosition.Y.Offset + delta.Y)
end

button.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPosition = frame.Position

 input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

button.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
        if dragging then
            update(input)
        end
    end
end)

-- Limita o movimento do botão para não sair da tela
local function clampPosition()
    local screenSize = screenGui.AbsoluteSize
    local frameSize = frame.AbsoluteSize
    local pos = frame.Position
    local xOffset = math.clamp(pos.X.Offset, 0, screenSize.X - frameSize.X)
    local yOffset = math.clamp(pos.Y.Offset, 0, screenSize.Y - frameSize.Y)
    frame.Position = UDim2.new(0, xOffset, 0, yOffset)
end

RunService.RenderStepped:Connect(clampPosition)
