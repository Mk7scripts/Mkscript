-- Variáveis
local espEnabled = false
local uiVisible = false
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
            newHighlight.FillTransparency = 1 -- Parte interna transparente
            newHighlight.OutlineColor = Color3.new(1, 1, 1) -- Borda branca
            newHighlight.OutlineTransparency = 0 -- Borda totalmente visível
            newHighlight.Parent = player.Character
        elseif not enable and highlight then
            highlight:Destroy()
        end
    end
end

-- Criar UI-GUI para ESP
local function createESPUI(parent)
    local espGui = Instance.new("Frame")
    espGui.Name = "ESP_UI"
    espGui.Size = UDim2.new(0, 250, 0, 70) -- Tamanho
    espGui.Position = UDim2.new(0.5, -125, 0, 10) -- Centralizado no topo
    espGui.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
    espGui.BorderColor3 = Color3.new(1, 1, 1) -- Borda branca
    espGui.BorderSizePixel = 2 -- Tamanho da borda
    espGui.Visible = false -- Inicialmente escondida
    espGui.Parent = parent

    -- Botão para ativar/desativar ESP
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 200, 0, 50)
    button.Position = UDim2.new(0.5, -100, 0.5, -25) -- Centralizado dentro da UI-GUI
    button.Text = "ESP+" -- Texto inicial
    button.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
    button.TextColor3 = Color3.new(1, 1, 1) -- Branco
    button.BorderColor3 = Color3.new(1, 1, 1) -- Borda branca
    button.BorderSizePixel = 2
    button.Font = Enum.Font.SourceSansBold
    button.TextScaled = true -- Ajuste de texto
    button.Parent = espGui

    -- Bordas arredondadas no botão
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 20)
    corner.Parent = button

    -- Lógica do botão ESP
    button.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        button.Text = espEnabled and "ESP-" or "ESP+" -- Atualiza o texto
        for _, player in pairs(players:GetPlayers()) do
            if player ~= localPlayer then
                toggleESP(player, espEnabled)
            end
        end
    end)

    return espGui
end

-- Criar botão roxo para mostrar/esconder UI-GUI
local function createToggleUIButton(parent, espUI)
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0, 50, 0, 50) -- Tamanho
    toggleButton.Position = UDim2.new(0.5, -25, 0.9, -60) -- Centralizado no meio inferior
    toggleButton.Text = "*" -- Texto do botão
    toggleButton.BackgroundColor3 = Color3.new(0.5, 0, 0.5) -- Roxo
    toggleButton.TextColor3 = Color3.new(1, 1, 1) -- Branco
    toggleButton.Font = Enum.Font.SourceSansBold
    toggleButton.TextScaled = true -- Ajuste de texto
    toggleButton.Parent = parent

    -- Bordas arredondadas no botão
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 20)
    corner.Parent = toggleButton

    -- Lógica do botão roxo
    toggleButton.MouseButton1Click:Connect(function()
        uiVisible = not uiVisible
        espUI.Visible = uiVisible -- Mostra/esconde a UI-GUI
    end)
end

-- Criar a GUI principal
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "Main_GUI"
    screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

    -- Criar UI-GUI de ESP
    local espUI = createESPUI(screenGui)

    -- Criar botão roxo para mostrar/esconder UI-GUI
    createToggleUIButton(screenGui, espUI)
end

-- Atualizar ESP para novos jogadores
players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        toggleESP(player, espEnabled)
    end)
end)

-- Garantir que GUI seja criada ao iniciar
createGUI()
