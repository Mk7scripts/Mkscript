-- Variáveis
local espEnabled = false
local players = game:GetService("Players")
local localPlayer = players.LocalPlayer

-- Função para criar ESP (borda vermelha ao redor dos jogadores)
local function toggleESP(player, enable)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local highlight = player.Character:FindFirstChild("ESPHighlight")
        if enable and not highlight then
            local newHighlight = Instance.new("Highlight")
            newHighlight.Name = "ESPHighlight"
            newHighlight.Adornee = player.Character -- Aplica ao personagem inteiro
            newHighlight.FillTransparency = 1 -- Deixa a parte interna transparente
            newHighlight.OutlineColor = Color3.new(1, 0, 0) -- Borda vermelha
            newHighlight.OutlineTransparency = 0 -- Totalmente visível
            newHighlight.Parent = player.Character
        elseif not enable and highlight then
            highlight:Destroy()
        end
    end
end

-- GUI adaptada para celular
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESP_GUI"
    screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

    -- Função para criar botões arredondados
    local function createButton(parent, text, position, callback)
        local button = Instance.new("TextButton", parent)
        button.Size = UDim2.new(0, 200, 0, 50) -- Tamanho maior para centralização
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
    local espButton = createButton(screenGui, "Ativar ESP", UDim2.new(0.5, -100, 0.05, 0), function() -- Centralizado
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
