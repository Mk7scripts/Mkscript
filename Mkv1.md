-- Variáveis
local espEnabled = false
local fovEnabled = false

-- Função para criar linhas FOV apontando para os jogadores
local function createFOVLine(player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local line = Instance.new("Beam")
        line.Parent = player.Character.HumanoidRootPart
        line.Attachment0 = Instance.new("Attachment", game.Players.LocalPlayer.Character.HumanoidRootPart)
        line.Attachment1 = Instance.new("Attachment", player.Character.HumanoidRootPart)
        line.Color = ColorSequence.new(Color3.new(1, 0, 0)) -- Vermelho
        line.Width0 = 0.2
        line.Width1 = 0.2
    end
end

-- GUI adaptada para celular
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local espButton = Instance.new("TextButton", screenGui)
espButton.Size = UDim2.new(0, 120, 0, 50)
espButton.Position = UDim2.new(0.1, 0, 0.1, 0)
espButton.Text = "Ativar ESP"
espButton.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
espButton.BorderColor3 = Color3.new(1, 1, 1) -- Branco
espButton.BorderSizePixel = 3
espButton.TextColor3 = Color3.new(1, 1, 1) -- Texto Branco
espButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    espButton.Text = espEnabled and "Desativar ESP" or "Ativar ESP"
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            createFOVLine(player)
        end
    end
end)

local fovButton = Instance.new("TextButton", screenGui)
fovButton.Size = UDim2.new(0, 120, 0, 50)
fovButton.Position = UDim2.new(0.1, 0, 0.2, 0)
fovButton.Text = "Ativar FOV"
fovButton.BackgroundColor3 = Color3.new(0, 0, 0) -- Preto
fovButton.BorderColor3 = Color3.new(1, 1, 1) -- Branco
fovButton.BorderSizePixel = 3
fovButton.TextColor3 = Color3.new(1, 1, 1) -- Texto Branco
fovButton.MouseButton1Click:Connect(function()
    fovEnabled = not fovEnabled
    fovButton.Text = fovEnabled and "Desativar FOV" or "Ativar FOV"
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            createFOVLine(player)
        end
    end
end)
