-- Variáveis
local espEnabled = false
local fovEnabled = false

-- Função para criar ESP (caixa vermelha ao redor dos jogadores)
local function createESP(player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        if not player.Character:FindFirstChild("ESPBox") then
            local box = Instance.new("BoxHandleAdornment")
            box.Name = "ESPBox"
            box.Adornee = player.Character
            box.Size = player.Character.HumanoidRootPart.Size
            box.Color3 = Color3.new(1, 0, 0) -- Vermelho
            box.Transparency = 0.5
            box.AlwaysOnTop = true
            box.ZIndex = 1
            box.Parent = player.Character
        end
        player.Character.ESPBox.Visible = espEnabled
    end
end

-- Função para criar linhas FOV (linhas apontando para os jogadores)
local function createFOVLines(player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local beam = player.Character:FindFirstChild("FOVLine") or Instance.new("Beam")
        beam.Name = "FOVLine"
        beam.Attachment0 = Instance.new("Attachment", game.Players.LocalPlayer.Character.HumanoidRootPart)
        beam.Attachment1 = Instance.new("Attachment", player.Character.HumanoidRootPart)
        beam.Color = ColorSequence.new(Color3.new(1, 1, 1)) -- Branco
        beam.Width0 = 0.1
        beam.Width1 = 0.1
        beam.Transparency = NumberSequence.new(0.5)
        beam.Parent = player.Character
        beam.Enabled = fovEnabled
    end
end

-- GUI adaptada para celular
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Botão de ESP
local espButton = Instance.new("TextButton", screenGui)
espButton.Size = UDim2.new(0, 120, 0, 50)
espButton.Position = UDim2.new(0.1, 0, 0.1, 0)
espButton.Text = "Ativar ESP"
espButton.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
espButton.BorderColor3 = Color3.new(1, 1, 1) -- Branco
espButton.BorderSizePixel = 3
espButton.TextColor3 = Color3.new(1, 1, 1)
espButton.MouseButton1Click()
    espEnabled = not espEnabled
    espButton.Text = espEnabled and "Desativar ESP" or "Ativar ESP"
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            createESP(player)
        end
    end
end)

-- Botão de FOV
local fovButton = Instance.new("TextButton", screenGui)
fovButton.Size = UDim2.new(0, 120, 0, 50)
fovButton.Position = UDim2.new(0.1, 0, 0.2, 0)
fovButton.Text = "Ativar FOV"
fovButton.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
fovButton.BorderColor3 = Color3.new(1, 1, 1) -- Branco
fovButton.BorderSizePixel = 3
fovButton.TextColor3 = Color3.new(1, 1, 1)
fovButton.MouseButton1Click:Connect(function()
    fovEnabled = not fovEnabled
    fovButton.Text = fovEnabled and "Desativar FOV" or "Ativar FOV"
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            createFOVLines(player)
        end
    end
end)

-- Atualização de ESP e FOV para jogadores que entrarem no jogo
game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if espEnabled then createESP(player) end
        if fovEnabled then createFOVLines(player) end
    end)
end)
