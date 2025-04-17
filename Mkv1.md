-- Variáveis
local espEnabled = false
local players = game:GetService("Players")
local localPlayer = players.LocalPlayer

-- Função para criar ESP (caixa branca ao redor dos jogadores)
local function toggleESP(player, enable)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local adornment = player.Character:FindFirstChild("ESPBox")
        if enable and not adornment then
            local box = Instance.new("BoxHandleAdornment")
            box.Name = "ESPBox"
            box.Adornee = player.Character
            box.Size = Vector3.new(4, 6, 4)
            box.Color3 = Color3.new(1, 1, 1) -- Branco
            box.Transparency = 0.5
            box.AlwaysOnTop = true
            box.Parent = player.Character
        elseif not enable and adornment then
            adornment:Destroy()
        end
    end
end

-- GUI adaptada para celular
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

-- Função para criar botões com design arredondado
local function createButton(parent, text, position, callback)
    local button = Instance.new("TextButton", parent)
    button.Size = UDim2.new(0, 150, 0, 50)
    button.Position = position
    button.Text = text
    button.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
    button.BorderColor3 = Color3.new(1, 1, 1) -- Branco
    button.BorderSizePixel = 2
    button.TextColor3 = Color3.new(1, 1, 1) -- Texto Branco
    button.Font = Enum.Font.SourceSansBold
    button.TextScaled = true -- Texto se ajusta ao tamanho do botão
    button.AutoButtonColor = true
    
    -- Tornando o botão arredondado
    local corner = Instance.new("UICorner", button)
    corner.CornerRadius = UDim.new(0, 20) -- Ajusta o arredondamento
    
    button.MouseButton1Click:Connect(callback)
    return button
end

-- Botão ESP
local espButton = createButton(screenGui, "Ativar ESP", UDim2.new(0.1, 0, 0.1, 0), function()
    espEnabled = not espEnabled
    espButton.Text = espEnabled and "Desativar ESP" or "Ativar ESP" -- Atualiza o texto do botão
    for _, player in pairs(players:GetPlayers()) do
        if player ~= localPlayer then
            toggleESP(player, espEnabled)
        end
    end
end)

-- Atualizar ESP para novos jogadores
players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        toggleESP(player, espEnabled)
    end)
end)
