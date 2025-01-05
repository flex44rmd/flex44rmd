local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- اللاعب المحلي
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- إعدادات Aim Assist
local aimEnabled = false
local aimCircle = Drawing.new("Circle")

-- إعداد الدائرة
aimCircle.Visible = true
aimCircle.Thickness = 2
aimCircle.Color = Color3.fromRGB(0, 255, 0)
aimCircle.Radius = 100
aimCircle.Filled = false
aimCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)

-- العثور على أقرب لاعب داخل الدائرة وفي نطاق 100 متر
local function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = aimCircle.Radius
    local maxDistance = 100 -- الحد الأقصى للمسافة بالأمتار

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") and player.Character:FindFirstChild("Humanoid") then
            local head = player.Character.Head
            local humanoid = player.Character.Humanoid

            -- تحقق من أن اللاعب ليس ميتًا
            if humanoid.Health > 0 then
                -- تحقق من أن اللاعب ليس من فريقك
                if player.Team ~= LocalPlayer.Team then
                    local headPosition = head.Position
                    local screenPos, onScreen = workspace.CurrentCamera:WorldToScreenPoint(headPosition)

                    if onScreen then
                        local distanceToCircle = (Vector2.new(screenPos.X, screenPos.Y) - aimCircle.Position).Magnitude
                        local distanceToPlayer = (LocalPlayer.Character.Head.Position - headPosition).Magnitude

                        -- تحقق من أن اللاعب داخل الدائرة وضمن المسافة المحددة
                        if distanceToCircle <= aimCircle.Radius and distanceToPlayer <= maxDistance and distanceToCircle < shortestDistance then
                            closestPlayer = player
                            shortestDistance = distanceToCircle
                        end
                    end
                end
            end
        end
    end

    return closestPlayer
end

-- تفعيل Aim Assist
RunService.RenderStepped:Connect(function()
    if aimEnabled then
        local closestPlayer = getClosestPlayer()
        if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("Head") then
            local head = closestPlayer.Character.Head
            workspace.CurrentCamera.CFrame = CFrame.new(
                workspace.CurrentCamera.CFrame.Position,
                head.Position
            )
        end
    end
end)

-- وظيفة لإضافة ESP لأي لاعب
local function addESP(player)
    -- تحقق من أن اللاعب لديه شخصية ورأس
    player.CharacterAdded:Connect(function(character)
        character:WaitForChild("Head") -- انتظر الرأس
        local head = character.Head

        -- إنشاء مكعب ESP
        local box = Instance.new("BoxHandleAdornment")
        box.Adornee = head
        box.Size = Vector3.new(2, 2, 2) -- حجم المكعب
        box.Color3 = Color3.fromRGB(255, 0, 0) -- لون المكعب (أحمر)
        box.AlwaysOnTop = true
        box.ZIndex = 0
        box.Transparency = 0.5
        box.Name = "ESPBox"
        box.Parent = head
    end)

    -- إذا كان اللاعب لديه شخصية بالفعل
    if player.Character and player.Character:FindFirstChild("Head") then
        local head = player.Character.Head

        -- إنشاء مكعب ESP
        local box = Instance.new("BoxHandleAdornment")
        box.Adornee = head
        box.Size = Vector3.new(2, 2, 2) -- حجم المكعب
        box.Color3 = Color3.fromRGB(255, 0, 0) -- لون المكعب (أحمر)
        box.AlwaysOnTop = true
        box.ZIndex = 0
        box.Transparency = 0.5
        box.Name = "ESPBox"
        box.Parent = head
    end
end

-- إنشاء واجهة المستخدم GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimAssistESP_GUI"
screenGui.Parent = PlayerGui

-- زر Aim Assist
local aimButton = Instance.new("TextButton")
aimButton.Parent = screenGui
aimButton.Size = UDim2.new(0, 200, 0, 50)
aimButton.Position = UDim2.new(0.5, -100, 0.8, -60)
aimButton.Text = "Aim Assist: OFF"
aimButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
aimButton.TextColor3 = Color3.fromRGB(255, 255, 255)
aimButton.TextScaled = true

-- زر Refresh ESP
local refreshButton = Instance.new("TextButton")
refreshButton.Size = UDim2.new(0, 200, 0, 50)
refreshButton.Position = UDim2.new(0.5, -100, 0.8, 0)
refreshButton.Text = "Refresh ESP"
refreshButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
refreshButton.TextColor3 = Color3.fromRGB(255, 255, 255)
refreshButton.TextScaled = true
refreshButton.Parent = screenGui

-- أحداث الأزرار
aimButton.MouseButton1Click:Connect(function()
    aimEnabled = not aimEnabled
    aimButton.Text = aimEnabled and "Aim Assist: ON" or "Aim Assist: OFF"
    aimCircle.Color = aimEnabled and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(0, 255, 0)
end)

-- وظيفة لتحديث الـ ESP عند الضغط على الزر
local function refreshESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            -- تحقق إذا كان الـ ESP موجودًا بالفعل
            local existingESP = player.Character and player.Character:FindFirstChild("Head") and player.Character.Head:FindFirstChild("ESPBox")
            if not existingESP then
                addESP(player) -- إضافة ESP إذا لم يكن موجودًا
            end
        end
    end
end

-- ربط الزر مع وظيفة التحديث
refreshButton.MouseButton1Click:Connect(function()
    refreshESP()
end)

-- تشغيل Aim Assist باستخدام المفاتيح
UserInputService.InputBegan:Connect(function(input, isProcessed)
    if isProcessed then return end -- تجاهل الإدخالات عند استخدام GUI
 
    -- تفعيل / إيقاف Aim Assist باستخدام زر **Ctrl**
    if input.KeyCode == Enum.KeyCode.LeftControl or input.KeyCode == Enum.KeyCode.RightControl then
        aimEnabled = not aimEnabled
        aimButton.Text = aimEnabled and "Aim Assist: ON" or "Aim Assist: OFF"
        aimCircle.Color = aimEnabled and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(0, 255, 0)
    end

    -- تفعيل ESP باستخدام زر **Alt**
    if input.KeyCode == Enum.KeyCode.LeftAlt or input.KeyCode == Enum.KeyCode.RightAlt then
        refreshESP() -- تحديث الـ ESP عند الضغط على Alt
    end
end)

-- إضافة ESP للاعبين الجدد
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        addESP(player)
    end
end)
