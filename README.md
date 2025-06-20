-- ✅ Aimbot com Teleporte de Paredes (Anti-Raycast / Fake Delete)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local aimbotCircle = Drawing.new("Circle")
aimbotCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
aimbotCircle.Color = Color3.fromRGB(255, 0, 0)
aimbotCircle.Transparency = 1
aimbotCircle.Filled = false
aimbotCircle.Thickness = 2
aimbotCircle.Radius = 100

local function isAlive(player)
    local character = player.Character
    return character and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0
end

local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = aimbotCircle.Radius

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and isAlive(player) then
            local head = player.Character and player.Character:FindFirstChild("Head")
            if head then
                local screenPoint, onScreen = Camera:WorldToViewportPoint(head.Position)
                if onScreen then
                    local dist = (Vector2.new(screenPoint.X, screenPoint.Y) - aimbotCircle.Position).Magnitude
                    if dist < shortestDistance then
                        shortestDistance = dist
                        closestPlayer = player
                    end
                end
            end
        end
    end

    return closestPlayer
end

local movedParts = {}

local function restoreWalls()
    for part, originalCFrame in pairs(movedParts) do
        if part and part:IsDescendantOf(workspace) then
            pcall(function()
                part.CFrame = originalCFrame
            end)
        end
    end
    movedParts = {}
end

local function moveWallsOut(target)
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local origin = Camera.CFrame.Position
        local targetPos = target.Character.Head.Position
        local direction = (targetPos - origin).Unit
        local distance = (targetPos - origin).Magnitude

        local step = 1
        for i = 0, distance, step do
            local point = origin + direction * i
            local region = Region3.new(point - Vector3.new(0.5,0.5,0.5), point + Vector3.new(0.5,0.5,0.5))
            local parts = workspace:FindPartsInRegion3WithIgnoreList(region, {LocalPlayer.Character, target.Character}, 10)

            for _, part in pairs(parts) do
                if part and part:IsDescendantOf(workspace) and not movedParts[part] and part.Anchored then
                    movedParts[part] = part.CFrame
                    part.CFrame = CFrame.new(0, -9999, 0) -- Move a parede pra longe (embaixo do mapa)
                end
            end
        end
    end
end

local currentTarget = nil

RunService.RenderStepped:Connect(function()
    local target = getClosestPlayer()

    if target and target ~= currentTarget then
        restoreWalls()
        currentTarget = target
    end

    if target then
        -- Aimbot
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
        -- Move as paredes para longe
        moveWallsOut(target)
    else
        restoreWalls()
        currentTarget = nil
    end
end)

print("✅ Aimbot + Wall Teleport Fake iniciado com sucesso!")
