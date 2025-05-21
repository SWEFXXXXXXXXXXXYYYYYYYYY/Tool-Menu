-- Roblox LocalScript (place in StarterGui)
-- Tool Menu with Highlight, Lighting, Speed Control, and Fly

local Players     = game:GetService("Players")
local RunService  = game:GetService("RunService")
local Lighting    = game:GetService("Lighting")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local originalLighting = {
    Brightness      = Lighting.Brightness,
    Ambient         = Lighting.Ambient,
    OutdoorAmbient  = Lighting.OutdoorAmbient,
    ClockTime       = Lighting.ClockTime
}

-- GUI Setup
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ToolMenuGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 220)
frame.Position = UDim2.new(0, 20, 0, 20)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 25)
title.Position = UDim2.new(0, 0, 0, 5)
title.BackgroundTransparency = 1
title.Text = "Tool Menu"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextSize = 18
title.Font = Enum.Font.SourceSansBold
title.Parent = frame

-- Button Factory
local function createButton(text, positionY, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 25)
    btn.Position = UDim2.new(0.05, 0, 0, positionY)
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextSize = 16
    btn.Font = Enum.Font.SourceSansBold
    btn.Parent = frame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    return btn
end

-- Create Buttons
local highlightBtn = createButton("Highlight: OFF", 35, Color3.fromRGB(200, 50, 50))
local lightingBtn  = createButton("Full bright: OFF", 65, Color3.fromRGB(100, 100, 200))
local speedBtn     = createButton("Speed: 16", 95, Color3.fromRGB(100, 200, 100))
local resetSpeedBtn = createButton("Reset Speed", 125, Color3.fromRGB(200, 150, 0))
local flyBtn       = createButton("Fly: OFF", 155, Color3.fromRGB(180, 100, 255))

-- Logic Variables
local highlights = {}
local highlightEnabled = false
local lightingEnabled = false
local flying = false
local currentSpeed = 16

-- Highlight Logic
local function ensureHighlight(player)
    if player == LocalPlayer then return end
    local char = player.Character
    if not char or not char.PrimaryPart then return end

    local existing = highlights[player]
    if existing and existing.Adornee == char then return end

    if existing then existing:Destroy() end

    local hl = Instance.new("Highlight")
    hl.FillColor = Color3.fromRGB(255, 255, 0)
    hl.OutlineColor = Color3.fromRGB(255, 255, 255)
    hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    hl.Adornee = char
    hl.Parent = char
    highlights[player] = hl
end

local function removeHighlight(player)
    if highlights[player] then
        highlights[player]:Destroy()
        highlights[player] = nil
    end
end

-- Fly Logic
local function startFly()
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    bodyGyro.P = 9e4
    bodyGyro.CFrame = root.CFrame
    bodyGyro.Parent = root

    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.zero
    bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    bodyVelocity.Parent = root

    local direction = Vector3.zero

    local conn1, conn2, runConn
    conn1 = UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Keyboard then
            if input.KeyCode == Enum.KeyCode.W then direction += Vector3.new(0, 0, -1) end
            if input.KeyCode == Enum.KeyCode.S then direction += Vector3.new(0, 0, 1) end
            if input.KeyCode == Enum.KeyCode.A then direction += Vector3.new(-1, 0, 0) end
            if input.KeyCode == Enum.KeyCode.D then direction += Vector3.new(1, 0, 0) end
            if input.KeyCode == Enum.KeyCode.Space then direction += Vector3.new(0, 1, 0) end
            if input.KeyCode == Enum.KeyCode.LeftControl then direction += Vector3.new(0, -1, 0) end
        end
    end)

    conn2 = UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Keyboard then
            if input.KeyCode == Enum.KeyCode.W then direction -= Vector3.new(0, 0, -1) end
            if input.KeyCode == Enum.KeyCode.S then direction -= Vector3.new(0, 0, 1) end
            if input.KeyCode == Enum.KeyCode.A then direction -= Vector3.new(-1, 0, 0) end
            if input.KeyCode == Enum.KeyCode.D then direction -= Vector3.new(1, 0, 0) end
            if input.KeyCode == Enum.KeyCode.Space then direction -= Vector3.new(0, 1, 0) end
            if input.KeyCode == Enum.KeyCode.LeftControl then direction -= Vector3.new(0, -1, 0) end
        end
    end)

    runConn = RunService.RenderStepped:Connect(function()
        if not flying or not char or not root then
            bodyGyro:Destroy()
            bodyVelocity:Destroy()
            conn1:Disconnect()
            conn2:Disconnect()
            runConn:Disconnect()
            return
        end
        bodyGyro.CFrame = workspace.CurrentCamera.CFrame
        bodyVelocity.Velocity = workspace.CurrentCamera.CFrame:VectorToWorldSpace(direction) * 50
    end)
end

-- Update Highlights Continuously
RunService.Heartbeat:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        if highlightEnabled then
            ensureHighlight(player)
        else
            removeHighlight(player)
        end
    end
end)

-- Button Callbacks
highlightBtn.MouseButton1Click:Connect(function()
    highlightEnabled = not highlightEnabled
    highlightBtn.Text = highlightEnabled and "Highlight: ON" or "Highlight: OFF"
    highlightBtn.BackgroundColor3 = highlightEnabled and Color3.fromRGB(50, 200, 50) or Color3.fromRGB(200, 50, 50)
end)

lightingBtn.MouseButton1Click:Connect(function()
    lightingEnabled = not lightingEnabled
    lightingBtn.Text = lightingEnabled and "Full bright: ON" or "Full bright: OFF"
    lightingBtn.BackgroundColor3 = lightingEnabled and Color3.fromRGB(100, 200, 255) or Color3.fromRGB(100, 100, 200)

    if lightingEnabled then
        Lighting.Brightness = 5
        Lighting.Ambient = Color3.new(1, 1, 1)
        Lighting.OutdoorAmbient = Color3.new(1, 1, 1)
        Lighting.ClockTime = 12
    else
        for prop, val in pairs(originalLighting) do
            Lighting[prop] = val
        end
    end
end)

speedBtn.MouseButton1Click:Connect(function()
    currentSpeed = currentSpeed + 25
    if currentSpeed > 200 then currentSpeed = 0 end
    speedBtn.Text = "Speed: " .. currentSpeed

    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = currentSpeed
    end
end)

resetSpeedBtn.MouseButton1Click:Connect(function()
    currentSpeed = 16
    speedBtn.Text = "Speed: 16"
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = currentSpeed
    end
end)

flyBtn.MouseButton1Click:Connect(function()
    flying = not flying
    flyBtn.Text = flying and "Fly: ON" or "Fly: OFF"
    flyBtn.BackgroundColor3 = flying and Color3.fromRGB(100, 255, 180) or Color3.fromRGB(180, 100, 255)
    if flying then
        startFly()
    end
end)

-- Keep speed on respawn
LocalPlayer.CharacterAdded:Connect(function(char)
    char:WaitForChild("Humanoid").WalkSpeed = currentSpeed
end)

Players.PlayerRemoving:Connect(removeHighlight)
