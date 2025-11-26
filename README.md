--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

--// Variáveis
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

local MaxDistance = 100
local AimSpeed = 0.3
local IsAiming = false

--// Função para encontrar o jogador mais próximo
local function GetNearestPlayer()
    local nearestPlayer = nil
    local nearestDistance = MaxDistance

    for i, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local targetRootPart = player.Character.HumanoidRootPart
            local distance = (RootPart.Position - targetRootPart.Position).Magnitude

            if distance < nearestDistance then
                nearestDistance = distance
                nearestPlayer = player
            end
        end
    end

    return nearestPlayer
end

--// Função para calcular a direção para a cabeça
local function GetDirectionToHead(targetPlayer)
    local head = targetPlayer.Character:FindFirstChild("Head")
    if not head then return nil end

    return (head.Position - RootPart.Position).Unit
end

--// Função para verificar a visibilidade do alvo
local function IsTargetVisible(targetPlayer)
    local head = targetPlayer.Character:FindFirstChild("Head")
    if not head then return false end

    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist

    local direction = (head.Position - RootPart.Position).Unit
    local raycastResult = workspace:Raycast(RootPart.Position, direction * MaxDistance, raycastParams)

    return raycastResult and raycastResult.Instance == head
end

--// Função para ajustar a mira
local function AimBot()
    if not IsAiming then return end

    local targetPlayer = GetNearestPlayer()

    if targetPlayer and IsTargetVisible(targetPlayer) then
        local direction = GetDirectionToHead(targetPlayer)

        if direction then
            local targetCFrame = CFrame.lookAt(Camera.CFrame.Position, Camera.CFrame.Position + direction)
            Camera.CFrame = Camera.CFrame:Lerp(targetCFrame, AimSpeed)
        end
    end
end

--// Evento de Input para ativar/desativar o AimBot
UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
    if gameProcessedEvent then return end

    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        IsAiming = true
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessedEvent)
    if gameProcessedEvent then return end

    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        IsAiming = false
    end
end)

--// Loop principal
RunService.RenderStepped:Connect(AimBot)
# morenexx
