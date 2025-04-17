-- Variáveis
local espEnabled = false
local players = game:GetService("Players")
local localPlayer = players.LocalPlayer

-- Função para ativar/desativar ESP
local function toggleESP(player, enable)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local highlight = player.Character:FindFirstChild("ESPHighlight")
        if enable and not highlight then
            local newHighlight = Instance.new("Highlight")
            newHighlight.Name = "ESPHighlight"
            newHighlight.Adornee = player.Character
            newHighlight.FillTransparency = 1 -- Transparência total na parte interna
            newHighlight.OutlineColor = Color3.new(1, 1, 1) -- Borda branca
            newHighlight.OutlineTransparency = 0 -- Totalmente visível
            newHighlight.Parent = player.Character
        elseif not enable and highlight then
            highlight:Destroy()
        end
    end
end

-- Criar UI-GUI
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ESP_GUI"
    screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

    -- Criar botão
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 200, 0, 50) -- Tamanho
    button.Position = UDim2.new(0.5, -100, 0, 10) -- Centralizado no topo
    button.Text = "ESP+" -- Texto inicial do botão
    button.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
    button.TextColor3 = Color3.new(1, 1, 1) -- Branco
    button.BorderColor3 = Color3.new(1, 1, 1) -- Borda branca
    button.BorderSizePixel = 2 -- Tamanho da borda
    button.Font = Enum.Font.SourceSansBold
    button.TextScaled = true -- Ajuste de texto
    button.Parent = screenGui

    -- Bordas arredondadas no botão
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 20)
    corner.Parent = button

    -- Lógica do botão
    button.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        button.Text = espEnabled and "ESP-" or "ESP+" -- Atualiza o texto
        for _, player in pairs(players:GetPlayers()) do
            if player ~= localPlayer then
                toggleESP(player, espEnabled)
            end
        end
    end)
end

-- Atualizar ESP para novos jogadores
players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        toggleESP(player, espEnabled)
    end)
end)

-- Garantir que GUI seja criada ao iniciar
createGUI()
