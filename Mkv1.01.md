-- Variáveis
local espEnabled = false
local players = game:GetService("Players")
local localPlayer = players.LocalPlayer

-- Função para criar ESP (caixa branca ao redor dos jogadores, visível através das paredes)
local function toggleESP(player, enable)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local adornment = player.Character:FindFirstChild("ESPBox")
        if enable and not adornment then
            local box = Instance.new("BoxHandleAdornment")
            box.Name = "ESPBox"
            box.Adornee = player.Character.HumanoidRootPart -- Focado na HumanoidRootPart
            box.Size = Vector3.new(4, 6, 4)
            box.Color3 = Color3.new(1, 1, 1) -- Branco
            box.Transparency = 0.5
            box.AlwaysOnTop = true -- Garante visibilidade através das paredes
            box.Parent = player.Character.HumanoidRootPart
        elseif not enable and adornment then
            adornment:Destroy()
        end
    end
end

-- GUI adaptada para celular
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESP_GUI"
    screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

    -- Função para criar botões arredondados
    local function createButton(parent, text, position, callback)
        local button = Instance.new("TextButton", parent)
        button.Size = UDim2.new(0, 150, 0, 50) -- Tamanho
        button.Position = position -- Posição na tela
        button.Text = text -- Texto inicial do botão
        button.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
        button.BorderColor3 = Color3.new(1, 1, 1) -- Branco
        button.BorderSizePixel = 2
        button.TextColor3 = Color3.new(1, 1, 1) -- Texto branco
        button.Font = Enum.Font.SourceSansBold
        button.TextScaled = true -- Ajusta o texto ao tamanho do botão
        
        -- Tornando o botão arredondado
        local corner = Instance.new("UICorner", button)
        corner.CornerRadius = UDim.new(0, 20) -- Define o arredondamento
        
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
end

-- Criar GUI ao iniciar e recriá-la após morrer
local function ensureGUI()
    if not localPlayer:FindFirstChild("PlayerGui"):FindFirstChild("ESP_GUI") then
        createGUI()
    end
end

-- Atualizar ESP para novos jogadores e personagens
players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        toggleESP(player, espEnabled)
    end)
end)

-- Detectar morte do jogador e recriar GUI
localPlayer.CharacterAdded:Connect(function()
    ensureGUI() -- Certificar-se de que a GUI esteja sempre recriada
end)

-- Inicializar GUI ao iniciar o script
ensureGUI()
