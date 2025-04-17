-- Variáveis
local espEnabled = false
local fovEnabled = false
local players = game:GetService("Players")
local localPlayer = players.LocalPlayer

-- Função para criar ESP (caixa vermelha ao redor dos jogadores)
local function toggleESP(player, enable)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local adornment = player.Character:FindFirstChild("ESPBox")
        if enable and not adornment then
            local box = Instance.new("BoxHandleAdornment")
            box.Name = "ESPBox"
            box.Adornee = player.Character
            box.Size = Vector3.new(4, 6, 4) -- Tamanho da caixa
            box.Color3 = Color3.new(1, 0, 0) -- Vermelho
            box.Transparency = 0.5
            box.AlwaysOnTop = true
            box.Parent = player.Character
        elseif not enable and adornment then
            adornment:Destroy()
        end
    end
end

-- Função para criar FOV (linhas apontando para os jogadores)
local function toggleFOV(player, enable)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local beam = player.Character:FindFirstChild("FOVLine")
        if enable and not beam then
            local newBeam = Instance.new("Beam")
            newBeam.Name = "FOVLine"
            newBeam.Attachment0 = Instance.new("Attachment", localPlayer.Character.HumanoidRootPart)
            newBeam.Attachment1 = Instance.new("Attachment", player.Character.HumanoidRootPart)
            newBeam.Color = ColorSequence.new(Color3.new(1, 1, 1)) -- Branco
            newBeam.Width0 = 0.1
            newBeam.Width1 = 0.1
            newBeam.Transparency = NumberSequence.new(0.5)
            newBeam.Parent = player.Character
        elseif not enable and beam then
            beam:Destroy()
        end
    end
end

-- GUI adaptada para celular
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

-- Função de design do botão
local function createButton(parent, text, position, callback)
    local button = Instance.new("TextButton", parent)
    button.Size = UDim2.new(0, 120, 0, 50)
    button.Position = position
    button.Text = text
    button.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
    button.BorderColor3 = Color3.new(1, 1, 1) -- Branco
    button.BorderSizePixel = 3
    button.TextColor3 = Color3.new(1, 1, 1)
    button.MouseButton1Click:Connect(callback)
    return button
end

-- Botão ESP
createButton(screenGui, "Ativar ESP", UDim2.new(0.1, 0, 0.1, 0), function()
    espEnabled = not espEnabled
    for _, player in pairs(players:GetPlayers()) do
        if player ~= localPlayer then
            toggleESP(player, espEnabled)
        end
    end
end)

-- Botão FOV
createButton(screenGui, "Ativar FOV", UDim2.new(0.1, 0, 0.2, 0), function()
    fovEnabled = not fovEnabled
    for _, player in pairs(players:GetPlayers()) do
        if player ~= localPlayer then
            toggleFOV(player, fovEnabled)
        end
    end
end)

-- Atualizar ESP e FOV para novos jogadores
players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        toggleESP(player, espEnabled)
        toggleFOV(player, fovEnabled)
    end)
end)
