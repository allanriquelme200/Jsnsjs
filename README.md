-- Script Completo TH3 ACS - Hitbox 12 + Smooth Aim + Wallcheck + ESP 1D fininho
-- Criado por DRAKERIQ3

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local HitboxSize = 12
local AimPart = "Head"
local AimSmoothness = 0.15 -- Quanto menor, mais rápido
local FOV = 120
local WallCheck = true
local ESPThickness = 1 -- Linha bem fininha

-- Variáveis internas
local AimbotEnabled = false

-- Função para verificar parede (wallcheck)
local function IsVisible(targetPart)
    if not targetPart or not targetPart:IsA("BasePart") then return false end
    local origin = Camera.CFrame.Position
    local direction = (targetPart.Position - origin).Unit * (targetPart.Position - origin).Magnitude
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    local raycastResult = workspace:Raycast(origin, direction, raycastParams)
    if raycastResult and raycastResult.Instance then
        if raycastResult.Instance:IsDescendantOf(targetPart.Parent) then
            return true
        else
            return false
        end
    else
        return true
    end
end

-- Função para expandir hitbox (Head)
local function ExpandHitbox(character)
    if not character then return end
    local head = character:FindFirstChild("Head")
    if head then
        head.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
        head.Transparency = 0.1
        head.BrickColor = BrickColor.new("Bright red")
        head.Material = Enum.Material.Neon
        head.CanCollide = false
        head.Massless = true
    end
end

-- Função para smooth aim
local function SmoothAim(targetPos)
    local cameraCFrame = Camera.CFrame
    local targetCFrame = CFrame.new(cameraCFrame.Position, targetPos)
    local newCFrame = cameraCFrame:Lerp(targetCFrame, AimSmoothness)
    Camera.CFrame = newCFrame
end

-- Função para encontrar o melhor alvo
local function GetClosestTarget()
    local closestDist = math.huge
    local closestPlayer = nil
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(AimPart) and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local part = player.Character[AimPart]
            local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
            if onScreen then
                local mousePos = UserInputService:GetMouseLocation()
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
                if dist < FOV then
                    if WallCheck then
                        if IsVisible(part) then
                            if dist < closestDist then
                                closestDist = dist
                                closestPlayer = player
                            end
                        end
                    else
                        if dist < closestDist then
                            closestDist = dist
                            closestPlayer = player
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- ESP fininho linha 1D
local ESPTable = {}

local function CreateESP(player)
    if ESPTable[player] then return end
    local line = Drawing.new("Line")
    line.Color = Color3.new(1, 0, 0)
    line.Thickness = ESPThickness
    line.Transparency = 1
    line.Visible = false
    ESPTable[player] = line
end

local function UpdateESP()
    for player, line in pairs(ESPTable) do
        if player.Character and player.Character:FindFirstChild("Torso") then
            local torso = player.Character.Torso
            local bottomPos = torso.Position - Vector3.new(0, 3, 0)

            local torsoScreen, visTorso = Camera:WorldToViewportPoint(torso.Position)
            local bottomScreen, visBottom = Camera:WorldToViewportPoint(bottomPos)

            if visTorso and visBottom then
                line.From = Vector2.new(bottomScreen.X, bottomScreen.Y)
                line.To = Vector2.new(torsoScreen.X, torsoScreen.Y)
                line.Visible = true
            else
                line.Visible = false
            end
        else
            line.Visible = false
        end
    end
end

-- Atualiza hitbox dos players constantemente
RunService.Heartbeat:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            ExpandHitbox(player.Character)
        end
    end
end)

-- Loop principal do aimbot
RunService.RenderStepped:Connect(function()
    -- Atualiza ESP
    UpdateESP()

    if AimbotEnabled then
        local target = GetClosestTarget()
        if target and target.Character and target.Character:FindFirstChild(AimPart) then
            local targetPos = target.Character[AimPart].Position
            SmoothAim(targetPos)
        end
    end
end)

-- Cria ESP para todos jogadores
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end

-- Novo jogador, cria ESP
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end)

-- Toggle aimbot com botão esquerdo do mouse
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Enum.UserInputType.MouseButton2 then -- botão direito do mouse para ativar/desativar
        AimbotEnabled = not AimbotEnabled
    end
end)
