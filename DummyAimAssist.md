getgenv().LockScript = {
    Enabled = true, -- Enable or disable the lock feature
    Keybind = Enum.KeyCode.C, -- Key to toggle the lock
    TargetPart = "HumanoidRootPart", -- Part of the target to aim at
    Prediction = 0.15, -- Adjusted prediction for better aiming
    Amount = 0.2, -- Adjusted lerp amount for smoother aiming
    ESPEnabled = true, -- Enable ESP
}

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local Client = Players.LocalPlayer
local LockedPlayer = nil
local ToggleLock = false

-- Function to create an ESP box and name for a player
local function createESP(player)
    local character = player.Character or player.CharacterAdded:Wait()
    if character:FindFirstChild("HumanoidRootPart") then
        -- Create hitbox
        local box = Instance.new("BoxHandleAdornment")
        box.Size = Vector3.new(2, 5, 1) -- Size of the hitbox
        box.Color3 = Color3.fromRGB(255, 0, 0) -- Hitbox color
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Parent = character.HumanoidRootPart

        -- Create name label
        local nameLabel = Instance.new("BillboardGui")
        nameLabel.Size = UDim2.new(1, 0, 1, 0)
        nameLabel.Adornee = character.HumanoidRootPart
        nameLabel.AlwaysOnTop = true

        local nameText = Instance.new("TextLabel")
        nameText.Size = UDim2.new(1, 0, 0.5, 0)
        nameText.BackgroundTransparency = 1
        nameText.TextColor3 = Color3.fromRGB(255, 255, 255)
        nameText.TextStrokeTransparency = 0.5
        nameText.Text = player.Name
        nameText.Parent = nameLabel

        nameLabel.Parent = character.HumanoidRootPart

        -- Clean up when the character is removed
        character.AncestryChanged:Connect(function(_, parent)
            if not parent then
                box:Destroy()
                nameLabel:Destroy()
            end
        end)
    end
end

-- Function to get the closest player to the mouse cursor
local function getClosestPlayerToCursor()
    local closestDist = math.huge
    local closestPlr = nil

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= Client and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local screenPos, cameraVisible = Camera:WorldToViewportPoint(player.Character[getgenv().LockScript.TargetPart].Position)
            if cameraVisible then
                local distToMouse = (UserInputService:GetMouseLocation() - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
                if distToMouse < closestDist then
                    closestPlr = player
                    closestDist = distToMouse
                end
            end
        end
    end
    return closestPlr
end

-- Function to toggle lock on key press
local function OnKeyPress(Input, GameProcessedEvent)
    if Input.KeyCode == getgenv().LockScript.Keybind and not GameProcessedEvent then 
        ToggleLock = not ToggleLock
        if ToggleLock then
            LockedPlayer = getClosestPlayerToCursor() -- Lock onto the closest player
        else
            LockedPlayer = nil -- Unlock
        end
    end
end

UserInputService.InputBegan:Connect(OnKeyPress)

-- Main loop to update camera position for aimbot lock
RunService.RenderStepped:Connect(function()
    if LockedPlayer and LockedPlayer.Character and LockedPlayer.Character:FindFirstChild(getgenv().LockScript.TargetPart) then
        local targetPart = LockedPlayer.Character[getgenv().LockScript.TargetPart]
        local targetPosition = targetPart.Position

        -- Adjust prediction based on target's velocity for more accuracy
        local predictedPosition = targetPosition + (LockedPlayer.Character.HumanoidRootPart.Velocity * getgenv().LockScript.Prediction)

        -- Create a new camera CFrame aiming at the predicted position
        local cameraCFrame = CFrame.new(Camera.CFrame.Position, predictedPosition)

        -- Smoothly interpolate to the new camera CFrame for better aiming
        Camera.CFrame = Camera.CFrame:Lerp(cameraCFrame, getgenv().LockScript.Amount)
    end
end)

-- Initialize ESP for each player when the script is executed
for _, player in ipairs(Players:GetPlayers()) do
    createESP(player)
end

-- Connect to PlayerAdded event to create ESP for new players
Players.PlayerAdded:Connect(createESP)
