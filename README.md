-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local SoundService = game:GetService("SoundService")

local localPlayer = Players.LocalPlayer
local mouse = localPlayer:GetMouse()

-- Configurações
local ESPAtivo = false
local AimbotAtivo = false
local TeamCheck = true
local SmoothAmount = 0.15 -- quanto menor, mais rápido a mira se move
local AimKey = Enum.KeyCode.E -- tecla para ativar aimbot

-- Sons para botão
local clickSound = Instance.new("Sound")
clickSound.SoundId = "rbxassetid://12222216"
clickSound.Volume = 0.5
clickSound.Parent = SoundService

local hoverSound = Instance.new("Sound")
hoverSound.SoundId = "rbxassetid://2101137"
hoverSound.Volume = 0.3
hoverSound.Parent = SoundService

-- Função para verificar time
local function IsEnemy(player)
    if not TeamCheck then return true end
    if not player.Team or not localPlayer.Team then return true end
    return player.Team ~= localPlayer.Team
end

-- Criar ESP
local ESPFolder = Instance.new("Folder")
ESPFolder.Name = "ESP_Folder"
ESPFolder.Parent = localPlayer.PlayerGui

local function CreateESPForPlayer(player)
    if player == localPlayer then return end
    if not IsEnemy(player) then return end

    local char = player.Character
    if not char then return end

    local head = char:FindFirstChild("Head")
    if not head then return end

    local espTag = Instance.new("BillboardGui")
    espTag.Name = "ESPTag"
    espTag.Adornee = head
    espTag.Size = UDim2.new(0, 100, 0, 25)
    espTag.AlwaysOnTop = true

    local textLabel = Instance.new("TextLabel", espTag)
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    textLabel.TextStrokeTransparency = 0.7
    textLabel.TextScaled = true
    textLabel.Font = Enum.Font.GothamBold
    textLabel.Text = player.Name

    espTag.Parent = ESPFolder
end

local function RemoveESPForPlayer(player)
    local esp = ESPFolder:FindFirstChild(player.Name)
    if esp then esp:Destroy() end
    -- Também remove pelo nome do BillboardGui se usado
    for _, v in pairs(ESPFolder:GetChildren()) do
        if v:IsA("BillboardGui") and v.Adornee and v.Adornee.Parent == player.Character then
            v:Destroy()
        end
    end
end

local function UpdateESP()
    -- Remove esp de players que não existem mais
    for _, esp in pairs(ESPFolder:GetChildren()) do
        if esp:IsA("BillboardGui") then
            local adornee = esp.Adornee
            if not adornee or not adornee.Parent or not Players:FindFirstChild(adornee.Parent.Name) then
                esp:Destroy()
            end
        end
    end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= localPlayer then
            local existing = false
            for _, esp in pairs(ESPFolder:GetChildren()) do
                if esp.Adornee and esp.Adornee.Parent == player.Character then
                    existing = true
                    break
                end
            end

            if ESPAtivo and IsEnemy(player) then
                if not existing then
                    CreateESPForPlayer(player)
                end
            else
                -- Remove ESP se desligado ou time amigo
                for _, esp in pairs(ESPFolder:GetChildren()) do
                    if esp.Adornee and esp.Adornee.Parent == player.Character then
                        esp:Destroy()
                    end
                end
            end
        end
    end
end

RunService.RenderStepped:Connect(UpdateESP)

-- Aimbot

local function getClosestEnemyToCursor()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character and IsEnemy(player) then
            local head = player.Character:FindFirstChild("Head")
            if head then
                local screenPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
                if onScreen then
                    local distance = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                    if distance < shortestDistance then
                        shortestDistance = distance
                        closestPlayer = player
                    end
                end
            end
        end
    end

    return closestPlayer
end

-- Variável para estado da tecla de aimbot
local aiming = false

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == AimKey then
        aiming = true
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == AimKey then
        aiming = false
    end
end)

RunService.RenderStepped:Connect(function()
    if AimbotAtivo and aiming then
        local target = getClosestEnemyToCursor()
        if target and target.Character then
            local head = target.Character:FindFirstChild("Head")
            local cam = workspace.CurrentCamera
            if head and cam then
                local currentCF = cam.CFrame
                local goalCFrame = CFrame.new(currentCF.Position, head.Position)
                cam.CFrame = currentCF:Lerp(goalCFrame, SmoothAmount)
            end
        end
    end
end)

-- ===== GUI =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ESP_Aimbot_Menu"
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local circleFrame = Instance.new("Frame")
circleFrame.Size = UDim2.new(0, 240, 0, 240)
circleFrame.Position = UDim2.new(0, 20, 0, 100)
circleFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
circleFrame.BorderSizePixel = 0
circleFrame.Active = true
circleFrame.Draggable = true
circleFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = circleFrame

local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(70, 20, 20)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 10, 10)),
}
gradient.Rotation = 90
gradient.Parent = circleFrame

-- Animação abrir
circleFrame.BackgroundTransparency = 1
circleFrame.Size = UDim2.new(0, 0, 0, 0)

TweenService:Create(circleFrame, TweenInfo.new(0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {
    BackgroundTransparency = 0,
    Size = UDim2.new(0, 240, 0, 240)
}):Play()

-- Pulsar quando ESP ativado
local pulseTweenInfo = TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
local pulseTween = TweenService:Create(circleFrame, pulseTweenInfo, {BackgroundColor3 = Color3.fromRGB(90, 20, 20)})

local function updatePulse()
    if ESPAtivo then
        pulseTween:Play()
    else
        pulseTween:Pause()
        TweenService:Create(circleFrame, TweenInfo.new(0.3), {BackgroundColor3 = Color3.fromRGB(20, 20, 20)}):Play()
    end
end

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(0, 200, 0, 40)
title.Position = UDim2.new(0, 20, 0, 15)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255, 180, 180)
title.Text = "ESP + Aimbot Menu 🎯"
title.Font = Enum.Font.GothamBold
title.TextScaled = true
title.Parent = circleFrame

-- Função para criar botão com hover, som e tooltip
local function createButton(text, posY, tooltipText)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 200, 0, 45)
    btn.Position = UDim2.new(0, 20, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(50, 15, 15)
    btn.TextColor3 = Color3.fromRGB(255, 180, 180)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextScaled = true
    btn.Parent = circleFrame

    local tooltip = Instance.new("TextLabel")
    tooltip.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    tooltip.BackgroundTransparency = 0.7
    tooltip.Size = UDim2.new(0, 200, 0, 30)
    tooltip.Position = UDim2.new(1, 5, 0, 0)
    tooltip.TextColor3 = Color3.fromRGB(230, 230, 230)
    tooltip.Font = Enum.Font.SourceSans
    tooltip.TextScaled = true
    tooltip.Text = tooltipText
    tooltip.Visible = false
    tooltip.Parent = btn

    btn.MouseEnter:Connect(function()
        hoverSound:Play()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(80, 25, 25), TextSize = 20}):Play()
        tooltip.Visible = true
    end)

    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50, 15, 15), TextSize = 16}):Play()
        tooltip.Visible = false
    end)

    return btn
end

local espBtn = createButton("🟢 Ativar ESP", 70, "Mostra inimigos com tag vermelha")
local aimbotBtn = createButton("🟢 Ativar Aimbot", 130, "Mira suave na cabeça inimiga com 'E' pressionado")

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30,
