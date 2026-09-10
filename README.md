
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local AUTHORIZED_USERS = {
    [123456789] = true, -- 6103207627

local function isAuthorized(player)
    return AUTHORIZED_USERS[player.UserId] == true
end

local remote = ReplicatedStorage:FindFirstChild("Lunna")

if not remote then
    remote = Instance.new("RemoteEvent")
    remote.Name = "LunnaKennyAdmin"
    remote.Parent = ReplicatedStorage
end

local function teleportPlayer(player, target)
    if not isAuthorized(player) then
        return
    end

    if not target or not target.Character then
        return
    end

    local character = player.Character
    local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
    local root = character and character:FindFirstChild("HumanoidRootPart")

    if root and targetRoot then
        root.CFrame = targetRoot.CFrame + Vector3.new(0, 3, 0)
    end
end

local function setFly(player, enabled)
    if not isAuthorized(player) then
        return
    end

    local character = player.Character
    if not character then
        return
    end

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then
        return
    end

    if enabled then
        humanoid.PlatformStand = true
    else
        humanoid.PlatformStand = false
    end
end

remote.OnServerEvent:Connect(function(player, action, data)

    if not isAuthorized(player) then
        warn(player.Name .. " tentou acessar o painel administrativo.")
        return
    end

    if action == "Teleport" then
        teleportPlayer(player, data)
    elseif action == "Fly" then
        setFly(player, data == true)

    elseif action == "ESP" then
        -- O ESP é controlado pelo LocalScript.
        -- O servidor apenas confirma a autorização.

    elseif action == "Aimbot" then
        -- O sistema de mira é destinado ao modo
        -- administrativo/treinamento do seu próprio jogo.
    end
end)

print("Lunna + Kenny Admin Panel carregado!")
:::

### 2. `StarterPlayer > StarterPlayerScripts > AdminPanelClient`

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local remote = ReplicatedStorage:WaitForChild("LunnaKennyAdmin")

local AUTHORIZED_USERS = {
    [123456789] = true,
    [987654321] = true,
}

if not AUTHORIZED_USERS[player.UserId] then
    return
end

local gui = Instance.new("ScreenGui")
gui.Name = "LunnaKennyPanel"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 330, 0, 390)
main.Position = UDim2.new(0.5, -165, 0.5, -195)
main.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
main.BorderSizePixel = 0
main.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 18)
corner.Parent = main

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(255, 105, 180)
stroke.Thickness = 3
stroke.Parent = main

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 70)
title.BackgroundColor3 = Color3.fromRGB(255, 105, 180)
title.Text = "🌸 LUNNA + KENNY 🌸"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 22
title.Font = Enum.Font.GothamBold
title.Parent = main

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 18)
titleCorner.Parent = title

local function createButton(text, y)
    local button = Instance.new("TextButton")

    button.Size = UDim2.new(0, 270, 0, 45)
    button.Position = UDim2.new(0.5, -135, 0, y)

    button.BackgroundColor3 = Color3.fromRGB(255, 225, 240)
    button.TextColor3 = Color3.fromRGB(210, 50, 130)

    button.Text = text
    button.TextSize = 16
    button.Font = Enum.Font.GothamBold

    button.AutoButtonColor = true
    button.Parent = main

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 12)
    c.Parent = button

    return button
end

local aimbotButton = createButton("🎯 Aimbot: OFF", 90)
local flyButton = createButton("🪽 Fly: OFF", 145)
local espButton = createButton("👁️ ESP: OFF", 200)
local teleportButton = createButton("📍 Teleport: Spawn", 255)

local aimbotEnabled = false
local flyEnabled = false
local espEnabled = false

local function getNearestPlayer()
    local character = player.Character
    if not character then
        return nil
    end

    local root = character:FindFirstChild("HumanoidRootPart")
    if not root then
        return nil
    end

    local nearest = nil
    local distance = math.huge

    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player and target.Character then

            local targetRoot =
                target.Character:FindFirstChild("HumanoidRootPart")

            local humanoid =
                target.Character:FindFirstChildOfClass("Humanoid")

            if targetRoot and humanoid and humanoid.Health > 0 then

                local d =
                    (root.Position - targetRoot.Position).Magnitude

                if d < distance then
                    distance = d
                    nearest = target
                end
            end
        end
    end

    return nearest
end

aimbotButton.MouseButton1Click:Connect(function()

    aimbotEnabled = not aimbotEnabled

    if aimbotEnabled then
        aimbotButton.Text = "🎯 Aimbot: ON"
        aimbotButton.BackgroundColor3 =
            Color3.fromRGB(255, 150, 205)
    else
        aimbotButton.Text = "🎯 Aimbot: OFF"
        aimbotButton.BackgroundColor3 =
            Color3.fromRGB(255, 225, 240)
    end

    remote:FireServer("Aimbot", aimbotEnabled)
end)

RunService.RenderStepped:Connect(function()

    if not aimbotEnabled then
        return
    end

    local target = getNearestPlayer()

    if target and target.Character then

        local head = target.Character:FindFirstChild("Head")
        local camera = workspace.CurrentCamera

        if head and camera then
            -- Suavemente direciona a câmera para o alvo.
            camera.CFrame = CFrame.lookAt(
                camera.CFrame.Position,
                head.Position
            )
        end
    end
end)

flyButton.MouseButton1Click:Connect(function()

    flyEnabled = not flyEnabled

    if flyEnabled then
        flyButton.Text = "🪽 Fly: ON"
        flyButton.BackgroundColor3 =
            Color3.fromRGB(255, 150, 205)
    else
        flyButton.Text = "🪽 Fly: OFF"
        flyButton.BackgroundColor3 =
            Color3.fromRGB(255, 225, 240)
    end

    remote:FireServer("Fly", flyEnabled)
end)

local highlights = {}

local function addESP(target)

    if target == player then
        return
    end

    if not target.Character then
        return
    end

    if highlights[target] then
        return
    end

    local highlight = Instance.new("Highlight")
    highlight.Name = "LunnaKennyESP"
    highlight.FillTransparency = 0.65
    highlight.OutlineTransparency = 0
    highlight.FillColor = Color3.fromRGB(255, 105, 180)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.Parent = target.Character

    highlights[target] = highlight
end

local function removeESP()

    for target, highlight in pairs(highlights) do
        if highlight then
            highlight:Destroy()
        end

        highlights[target] = nil
    end
end

espButton.MouseButton1Click:Connect(function()

    espEnabled = not espEnabled

    if espEnabled then

        espButton.Text = "👁️ ESP: ON"
        espButton.BackgroundColor3 =
            Color3.fromRGB(255, 150, 205)

        for _, target in ipairs(Players:GetPlayers()) do
            addESP(target)
        end

    else

        espButton.Text = "👁️ ESP: OFF"
        espButton.BackgroundColor3 =
            Color3.fromRGB(255, 225, 240)

        removeESP()
    end

    remote:FireServer("ESP", espEnabled)
end)

Players.PlayerAdded:Connect(function(target)

    target.CharacterAdded:Connect(function()

        task.wait(1)

        if espEnabled then
            addESP(target)
        end
    end)
end)

teleportButton.MouseButton1Click:Connect(function()

    local character = player.Character
    if not character then
        return
    end

    local root = character:FindFirstChild("HumanoidRootPart")

    if root then
        -- Teleporta alguns studs para frente,
        -- útil como comando administrativo de teste.
        root.CFrame =
            root.CFrame + root.CFrame.LookVector * 15
    end
end)

local dragging = false
local dragStart
local startPosition

title.InputBegan:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1 then

        dragging = true
        dragStart = input.Position
        startPosition = main.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)

    if not dragging then
        return
    end

    if input.UserInputType == Enum.UserInputType.MouseMovement then

        local delta = input.Position - dragStart

        main.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end
