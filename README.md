-- Variáveis
local espEnabled = false
local fovEnabled = false

-- Função para criar bordas ESP ao redor dos personagens
local function createESP(player)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local highlight = Instance.new("Highlight", player.Character)
        highlight.FillTransparency = 1
        highlight.OutlineTransparency = 0
        highlight.OutlineColor = Color3.new(1, 0, 0) -- Vermelho
        highlight.Enabled = espEnabled
    end
end

-- Função para criar setas FOV
local function createFOVArrow(enemy)
    local arrow = Instance.new("BillboardGui", enemy.Character.HumanoidRootPart)
    arrow.Size = UDim2.new(0, 50, 0, 50)
    arrow.Adornee = enemy.Character.HumanoidRootPart
    arrow.Enabled = fovEnabled
    
    local frame = Instance.new("Frame", arrow)
    frame.BackgroundColor3 = Color3.new(1, 0, 0) -- Vermelho
    frame.Size = UDim2.new(1, 0, 1, 0)
    frame.BorderSizePixel = 0
end

-- GUI adaptada para celular
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local espButton = Instance.new("TextButton", screenGui)
espButton.Size = UDim2.new(0, 100, 0, 50)
espButton.Position = UDim2.new(0.1, 0, 0.1, 0)
espButton.Text = "Ativar ESP"
espButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    espButton.Text = espEnabled and "Desativar ESP" or "Ativar ESP"
    for _, player in pairs(game.Players:GetPlayers()) do
        createESP(player)
    end
end)

local fovButton = Instance.new("TextButton", screenGui)
fovButton.Size = UDim2.new(0, 100, 0, 50)
fovButton.Position = UDim2.new(0.1, 0, 0.2, 0)
fovButton.Text = "Ativar FOV"
fovButton.MouseButton1Click:Connect(function()
    fovEnabled = not fovEnabled
    fovButton.Text = fovEnabled and "Desativar FOV" or "Ativar FOV"
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            createFOVArrow(player)
        end
    end
end)

-- Inicialização
game.Players.PlayerAdded:Connect(createESP)
