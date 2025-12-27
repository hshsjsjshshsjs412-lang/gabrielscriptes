--!native
--!strict

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- VARIÁVEIS
local LocalPlayer = Players.LocalPlayer
local espActive: boolean = false

-- CONFIGURAÇÕES
local SETTINGS = {
    FillColor = Color3.fromRGB(0, 255, 127),
    OutlineColor = Color3.fromRGB(255, 255, 255),
    FillTransparency = 0.5,
    OutlineTransparency = 0,
    TextColor = Color3.fromRGB(255, 255, 255)
}

-- UI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui")
local ToggleButton = Instance.new("TextButton")

ScreenGui.Name = "ESP_Menu"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

ToggleButton.Size = UDim2.new(0, 150, 0, 40)
ToggleButton.Position = UDim2.new(0.05, 0, 0.05, 0)
ToggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
ToggleButton.Text = "ESP: OFF"
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.Font = Enum.Font.RobotoMono
ToggleButton.TextSize = 18
ToggleButton.Parent = ScreenGui

-- FUNÇÃO PARA CRIAR A TAG DE DISTÂNCIA
local function createDistanceTag(character: Model)
    local head = character:WaitForChild("Head", 5)
    if not head then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "DistanceTag"
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 100, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true -- Permite ver através das paredes
    billboard.Enabled = espActive

    local label = Instance.new("TextLabel")
    label.Parent = billboard
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(1, 0, 1, 0)
    label.TextColor3 = SETTINGS.TextColor
    label.TextStrokeTransparency = 0
    label.Font = Enum.Font.RobotoMono
    label.TextSize = 14
    label.Text = "Calculando..."

    billboard.Parent = character
end

-- FUNÇÃO PARA CRIAR O HIGHLIGHT (WALLHACK)
local function applyESP(character: Model)
    if not character or character == LocalPlayer.Character then return end
    
    -- Highlight
    local highlight = (character:FindFirstChild("DebugHighlight") or Instance.new("Highlight")) :: Highlight
    highlight.Name = "DebugHighlight"
    highlight.Adornee = character
    highlight.FillColor = SETTINGS.FillColor
    highlight.OutlineColor = SETTINGS.OutlineColor
    highlight.FillTransparency = SETTINGS.FillTransparency
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- AQUI ESTÁ O WALLHACK
    highlight.Enabled = espActive
    highlight.Parent = character
    
    -- Tag de Distância
    if not character:FindFirstChild("DistanceTag") then
        createDistanceTag(character)
    end
end

-- LOOP DE ATUALIZAÇÃO DE DISTÂNCIA (OTIMIZADO)
RunService.RenderStepped:Connect(function()
    if not espActive then return end
    
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    local myPos = char.HumanoidRootPart.Position

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local root = player.Character:FindFirstChild("HumanoidRootPart")
            local tag = player.Character:FindFirstChild("DistanceTag")
            
            if root and tag then
                local label = tag:FindFirstChild("TextLabel") :: TextLabel
                if label then
                    local magnitude = (myPos - root.Position).Magnitude
                    local meters = magnitude * 0.28 -- Conversão padrão de Studs para Metros
                    label.Text = string.format("[%d m]", math.floor(meters))
                end
            end
        end
    end
end)

-- CONTROLE DE ATIVAÇÃO
local function toggleVisuals()
    espActive = not espActive
    ToggleButton.Text = "ESP: " .. (espActive and "ON" or "OFF")
    ToggleButton.BackgroundColor3 = espActive and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)

    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character then
            local h = player.Character:FindFirstChild("DebugHighlight") :: Highlight
            local t = player.Character:FindFirstChild("DistanceTag") :: BillboardGui
            if h then h.Enabled = espActive end
            if t then t.Enabled = espActive end
            
            if espActive then applyESP(player.Character) end
        end
    end
end

-- EVENTOS
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        task.wait(1)
        applyESP(char)
    end)
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.T then
        toggleVisuals()
    end
end)

ToggleButton.MouseButton1Click:Connect(toggleVisuals)

-- INICIALIZAÇÃO
for _, player in ipairs(Players:GetPlayers()) do
    if player.Character then applyESP(player.Character) end
end
