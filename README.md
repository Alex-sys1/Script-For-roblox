local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local Drawing = Drawing or require("Drawing")

local aimLockEnabled = false
local aimLockActive = false
local targetPlayer = nil
local fovRange = 60 -- Increased field of view range in pixels
local fovCircle = Drawing.new("Circle")

-- Menu variables
local menuVisible = true
local menuDragging = false
local menuPosition = Vector2.new(300, 300)
local menuSize = Vector2.new(250, 220) -- Increased size to accommodate additional text
local dragOffset = Vector2.new(0, 0)

-- Create menu elements
local menuBackground = Drawing.new("Square")
local menuTitle = Drawing.new("Text")
local menuStatus = Drawing.new("Text")
local menuMode = Drawing.new("Text")
local menuCreator = Drawing.new("Text")
local menuControls = Drawing.new("Text") -- Added to show control keys

-- ESP variables
local espEnabled = true
local espTexts = {}

-- Define the color for enemy ESP (green fluorescent)
local enemyColor = Color3.fromRGB(0, 255, 0) -- Verde fosforescente

-- Define mode variables
local is1v1Mode = true

-- Function to determine if a player is an enemy
local function isEnemy(player)
    if is1v1Mode then
        return player ~= LocalPlayer
    else
        return player.Team ~= LocalPlayer.Team
    end
end

-- Function to create ESP text for a player
local function createESPText(player)
    if isEnemy(player) then
        local text = Drawing.new("Text")
        text.Text = player.Name
        text.Size = 18
        text.Color = enemyColor -- Color verde fosforescente para los enemigos
        text.Outline = true
        text.OutlineColor = Color3.fromRGB(0, 0, 0)
        text.Center = true
        espTexts[player] = text
    end
end

-- Function to update ESP text for a player
local function updateESPText(player)
    local character = player.Character
    if character and character:FindFirstChild("Head") and isEnemy(player) then
        local head = character.Head
        local headPosition = Camera:WorldToViewportPoint(head.Position)
        if headPosition.Z > 0 then
            espTexts[player].Visible = true
            espTexts[player].Position = Vector2.new(headPosition.X, headPosition.Y - 25)
        else
            espTexts[player].Visible = false
        end
    else
        espTexts[player].Visible = false
    end
end

-- Function to remove ESP text for a player
local function removeESPText(player)
    if espTexts[player] then
        espTexts[player]:Remove()
        espTexts[player] = nil
    end
end

-- Function to get the closest player within FOV
local function getClosestPlayerInFOV()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") and isEnemy(player) then
            local character = player.Character
            local head = character.Head
            local screenPoint = Camera:WorldToViewportPoint(head.Position)
            local onScreen = screenPoint.Z > 0
            local distanceFromCenter = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude

            if onScreen and distanceFromCenter < fovRange then
                local distance = (head.Position - LocalPlayer.Character.Head.Position).Magnitude
                if distance < shortestDistance then
                    shortestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end

    return closestPlayer
end

-- Function to lock aim
local function aimLock()
    if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
        local targetPosition = targetPlayer.Character.Head.Position
        local aimPosition = CFrame.new(Camera.CFrame.Position, targetPosition)
        Camera.CFrame = aimPosition
    end
end

-- Visualize the FOV circle
local function updateFOVCircle()
    fovCircle.Visible = aimLockEnabled
    fovCircle.Thickness = 2
    fovCircle.Color = Color3.fromRGB(144, 238, 144) -- Verde claro
    fovCircle.Filled = false
    fovCircle.Radius = fovRange
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
end

-- Update menu elements
local function updateMenu()
    if menuVisible then
        -- Update menu background
        menuBackground.Visible = true
        menuBackground.Position = menuPosition
        menuBackground.Size = menuSize
        menuBackground.Color = Color3.fromRGB(50, 50, 50) -- Fondo del menú
        menuBackground.Transparency = 0.4
        menuBackground.Filled = true

        -- Update menu title
        menuTitle.Visible = true
        menuTitle.Text = "Menú de Aimlock"
        menuTitle.Position = menuPosition + Vector2.new(10, 10)
        menuTitle.Size = 24
        menuTitle.Color = Color3.fromRGB(255, 215, 0) -- Dorado
        menuTitle.Outline = true
        menuTitle.OutlineColor = Color3.fromRGB(0, 0, 0)

        -- Update menu status
        menuStatus.Visible = true
        menuStatus.Text = "Aimlock: " .. (aimLockEnabled and "Habilitado" or "Deshabilitado")
        menuStatus.Position = menuPosition + Vector2.new(10, 50)
        menuStatus.Size = 20
        menuStatus.Color = aimLockEnabled and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0) -- Green if enabled, red if disabled
        menuStatus.Outline = true
        menuStatus.OutlineColor = Color3.fromRGB(0, 0, 0)

        -- Update menu mode
        menuMode.Visible = true
        menuMode.Text = "Modo: " .. (is1v1Mode and "1v1" or "Equipo")
        menuMode.Position = menuPosition + Vector2.new(10, 90)
        menuMode.Size = 20
        menuMode.Color = Color3.fromRGB(0, 255, 255) -- Cyan
        menuMode.Outline = true
        menuMode.OutlineColor = Color3.fromRGB(0, 0, 0)

        -- Update menu creator
        menuCreator.Visible = true
        menuCreator.Text = "Creador: Vixx77"
        menuCreator.Position = menuPosition + Vector2.new(10, 130)
        menuCreator.Size = 20
        menuCreator.Color = Color3.fromRGB(255, 0, 0) -- Rojo
        menuCreator.Outline = true
        menuCreator.OutlineColor = Color3.fromRGB(0, 0, 0)
        
        -- New controls information
        menuControls.Visible = true
        menuControls.Text = "Controles:\nX - Activar Aimlock\nZ - Cambiar Modo\nF1 - Ocultar/Mostrar Menú"
        menuControls.Position = menuPosition + Vector2.new(10, 160)
        menuControls.Size = 16
        menuControls.Color = Color3.fromRGB(255, 165, 0) -- Naranja brillante
        menuControls.Outline = true
        menuControls.OutlineColor = Color3.fromRGB(0, 0, 0)
    else
        menuBackground.Visible = false
        menuTitle.Visible = false
        menuStatus.Visible = false
        menuMode.Visible = false
        menuCreator.Visible = false
        menuControls.Visible = false
    end
end

-- Handle menu dragging
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton1 and not gameProcessed then
        if Mouse.X >= menuPosition.X and Mouse.X <= menuPosition.X + menuSize.X and
           Mouse.Y >= menuPosition.Y and Mouse.Y <= menuPosition.Y + menuSize.Y then
            menuDragging = true
            dragOffset = Vector2.new(Mouse.X, Mouse.Y) - menuPosition
        end
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton1 and not gameProcessed then
        menuDragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseMovement and menuDragging and not gameProcessed then
        menuPosition = Vector2.new(Mouse.X, Mouse.Y) - dragOffset
    end
end)

-- Toggle aim lock activation with "X" key
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.X and not gameProcessed then
        aimLockEnabled = not aimLockEnabled
    end
end)

-- Toggle 1v1 and team mode with "Z" key
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.Z and not gameProcessed then
        is1v1Mode = not is1v1Mode
    end
end)

-- Toggle menu visibility with "F1" key
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.F1 and not gameProcessed then
        menuVisible = not menuVisible
    end
end)

-- Mouse button right down
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton2 and not gameProcessed and aimLockEnabled then
        aimLockActive = true
        targetPlayer = getClosestPlayerInFOV()
    end
end)

-- Mouse button right up
UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton2 and not gameProcessed then
        aimLockActive = false
        targetPlayer = nil
    end
end)

-- Remove ESP text when player leaves
Players.PlayerRemoving:Connect(function(player)
    removeESPText(player)
end)

-- RunService to update aim lock, FOV circle, ESP texts, and menu
RunService.RenderStepped:Connect(function()
    if aimLockEnabled and aimLockActive then
        targetPlayer = getClosestPlayerInFOV()
        if targetPlayer then
            aimLock()
        end
    end

    for _, player in pairs(Players:GetPlayers()) do
        if espEnabled and player ~= LocalPlayer and isEnemy(player) then
            if not espTexts[player] then
                createESPText(player)
            end
            updateESPText(player)
        elseif espTexts[player] then
            removeESPText(player)
        end
    end

    updateFOVCircle()
    updateMenu()
end)
