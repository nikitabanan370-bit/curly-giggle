local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Переменные
local platToggle = false
local hookToggle = false
local cloakToggle = false
local basePos = nil
local char = nil
local rootPart = nil
local humanoid = nil
local bodyVelocity = nil

-- Ждём персонажа
local function onCharacterAdded(newChar)
    char = newChar
    rootPart = char:WaitForChild("HumanoidRootPart")
    humanoid = char:WaitForChild("Humanoid")
    if not basePos then
        basePos = rootPart.Position -- Стартовый спавн как база
    end
end
player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then
    onCharacterAdded(player.Character)
end

-- GUI (ещё красивее версия)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BrainrotCheat"
screenGui.Parent = playerGui

-- Toggle кнопка (стильная)
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 120, 0, 50)
toggleBtn.Position = UDim2.new(0, 10, 0, 10)
toggleBtn.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
toggleBtn.Text = "🛡️ Toggle Panel"
toggleBtn.TextColor3 = Color3.new(1, 1, 1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 14
local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 12)
toggleCorner.Parent = toggleBtn
local toggleStroke = Instance.new("UIStroke")
toggleStroke.Color = Color3.new(0.2, 0.8, 1)  -- Неоновый голубой
toggleStroke.Thickness = 2
toggleStroke.Parent = toggleBtn
local toggleGradient = Instance.new("UIGradient")
toggleGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.new(0.1, 0.1, 0.1)),
    ColorSequenceKeypoint.new(1, Color3.new(0.3, 0.3, 0.3))
}
toggleGradient.Rotation = 45
toggleGradient.Parent = toggleBtn
toggleBtn.Parent = screenGui

-- Hover для toggle
local tweenHoverIn = TweenService:Create(toggleBtn, TweenInfo.new(0.2), {Size = UDim2.new(0, 130, 0, 55)})
local tweenHoverOut = TweenService:Create(toggleBtn, TweenInfo.new(0.2), {Size = UDim2.new(0, 120, 0, 50)})
toggleBtn.MouseEnter:Connect(function() tweenHoverIn:Play() end)
toggleBtn.MouseLeave:Connect(function() tweenHoverOut:Play() end)

-- Main Frame (панель с заголовком)
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 220, 0, 240)
mainFrame.Position = UDim2.new(0, 10, 0, 70)
mainFrame.BackgroundColor3 = Color3.new(0.15, 0.15, 0.15)
mainFrame.Visible = false
mainFrame.Parent = screenGui
local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 16)
mainCorner.Parent = mainFrame
local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.new(0.2, 0.8, 1)
mainStroke.Thickness = 2
mainStroke.Parent = mainFrame
local mainGradient = Instance.new("UIGradient")
mainGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.new(0.15, 0.15, 0.15)),
    ColorSequenceKeypoint.new(1, Color3.new(0.05, 0.05, 0.05))
}
mainGradient.Rotation = 90
mainGradient.Parent = mainFrame

-- Заголовок панели с градиентом
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 30)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🧠 Brainrot Cheat"
titleLabel.TextColor3 = Color3.new(0.2, 0.8, 1)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 16
local titleGradient = Instance.new("UIGradient")
titleGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.new(0.2, 0.8, 1)),
    ColorSequenceKeypoint.new(1, Color3.new(0.8, 0.2, 1))
}
titleGradient.Rotation = 45
titleGradient.Parent = titleLabel
titleLabel.Parent = mainFrame

-- Кнопка закрытия (X в углу)
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 20, 0, 20)
closeBtn.Position = UDim2.new(1, -25, 0, 5)
closeBtn.BackgroundColor3 = Color3.new(0.8, 0.2, 0.2)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 14
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 8)
closeCorner.Parent = closeBtn
local closeStroke = Instance.new("UIStroke")
closeStroke.Color = Color3.new(1, 0.2, 0.2)
closeStroke.Thickness = 1
closeStroke.Parent = closeBtn
closeBtn.Parent = mainFrame
closeBtn.MouseButton1Click:Connect(function()
    local tweenClose = TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(0, 0, 0, 0)})
    tweenClose:Play()
    tweenClose.Completed:Connect(function()
        mainFrame.Visible = false
        mainFrame.Size = UDim2.new(0, 220, 0, 240)
    end)
end)

-- UIListLayout для кнопок (авто-выравнивание)
local layout = Instance.new("UIListLayout")
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 10)
layout.Parent = mainFrame

-- Кнопки в панели (по умолчанию красные OFF, с градиентом, тенью и hover)
local function createStyledButton(text, positionY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 50)
    btn.Position = UDim2.new(0, 10, 0, positionY)
    btn.BackgroundColor3 = Color3.new(1, 0, 0)  -- Красный OFF
    btn.Text = text
    btn.TextColor3 = Color3.new(0, 0, 0)  -- Чёрный текст для видимости
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 12
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = btn
    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color = Color3.new(0.5, 0.5, 0.5)  -- Тень/обводка
    btnStroke.Thickness = 1
    btnStroke.Transparency = 0.5  -- Полупрозрачная тень
    btnStroke.Parent = btn
    local btnGradient = Instance.new("UIGradient")
    btnGradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.new(1, 0, 0)),
        ColorSequenceKeypoint.new(1, Color3.new(0.8, 0, 0))
    }
    btnGradient.Rotation = 45
    btnGradient.Parent = btn
    btn.Parent = mainFrame
    
    -- Hover эффект
    local btnHoverIn = TweenService:Create(btn, TweenInfo.new(0.2), {Size = UDim2.new(1, -10, 0, 55), BackgroundTransparency = 0.1})
    local btnHoverOut = TweenService:Create(btn, TweenInfo.new(0.2), {Size = UDim2.new(1, -20, 0, 50), BackgroundTransparency = 0})
    btn.MouseEnter:Connect(function() btnHoverIn:Play() end)
    btn.MouseLeave:Connect(function() btnHoverOut:Play() end)
    
    return btn
end

local platBtn = createStyledButton("🪜 Platforms: OFF", 40)
local hookBtn = createStyledButton("🎣 Hook Fly: OFF", 100)
local cloakBtn = createStyledButton("👻 Cloak Invis: OFF", 160)

-- Toggle GUI с анимацией
toggleBtn.MouseButton1Click:Connect(function()
    if mainFrame.Visible then
        local tweenClose = TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(0, 0, 0, 0)})
        tweenClose:Play()
        tweenClose.Completed:Connect(function()
            mainFrame.Visible = false
            mainFrame.Size = UDim2.new(0, 220, 0, 240)
        end)
    else
        mainFrame.Visible = true
        mainFrame.Size = UDim2.new(0, 0, 0, 0)
        local tweenOpen = TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 220, 0, 240)})
        tweenOpen:Play()
    end
end)

-- Функция детекта Brainrot (в персонаже)
local function hasBrainrot()
    if not char then return false end
    for _, child in pairs(char:GetDescendants()) do
        if child:IsA("BasePart") and string.find(string.lower(child.Name), "brainrot") then
            return true, child
        end
    end
    return false
end

-- Функция детекта инструмента
local function hasTool(toolName)
    local backpack = player:FindFirstChild("Backpack")
    local characterBackpack = char and char:FindFirstChildOfClass("Tool")
    if backpack and backpack:FindFirstChild(toolName) then return true end
    if characterBackpack and characterBackpack.Name == toolName then return true end
    return false
end

-- 1. Platforms spam
platBtn.MouseButton1Click:Connect(function()
    platToggle = not platToggle
    platBtn.Text = "🪜 Platforms: " .. (platToggle and "ON" or "OFF")
    platBtn.BackgroundColor3 = platToggle and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
    platBtn.TextColor3 = Color3.new(0, 0, 0)  -- Чёрный текст
    local platGradient = platBtn:FindFirstChild("UIGradient")
    if platGradient then
        platGradient.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, platToggle and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)),
            ColorSequenceKeypoint.new(1, platToggle and Color3.new(0, 0.8, 0) or Color3.new(0.8, 0, 0))
        }
    end
    local platStroke = platBtn:FindFirstChild("UIStroke")
    if platStroke then
        platStroke.Color = platToggle and Color3.new(0, 0.8, 0) or Color3.new(1, 0.2, 0.2)
    end
    if platToggle then
        spawn(function()
            while platToggle and rootPart do
                local platform = Instance.new("Part")
                platform.Name = "FlyPlatform"
                platform.Size = Vector3.new(5, 1, 5)
                platform.Anchored = true
                platform.CanCollide = true
                platform.BrickColor = BrickColor.new("Bright blue")
                platform.Material = Enum.Material.Neon
                platform.Position = rootPart.Position - Vector3.new(0, 3, 0)
                platform.Parent = workspace
                local TweenService = game:GetService("TweenService")
                local tween = TweenService:Create(platform, TweenInfo.new(2, Enum.EasingStyle.Linear), {Position = platform.Position + Vector3.new(0, 10, 0)})
                tween:Play()
                wait(0.5)
                game:GetService("Debris"):AddItem(platform, 5)
            end
        end)
    end
end)

-- 2. Hook Fly
hookBtn.MouseButton1Click:Connect(function()
    hookToggle = not hookToggle
    hookBtn.Text = "🎣 Hook Fly: " .. (hookToggle and "ON" or "OFF")
    hookBtn.BackgroundColor3 = hookToggle and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
    hookBtn.TextColor3 = Color3.new(0, 0, 0)  -- Чёрный текст
    local hookGradient = hookBtn:FindFirstChild("UIGradient")
    if hookGradient then
        hookGradient.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, hookToggle and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)),
            ColorSequenceKeypoint.new(1, hookToggle and Color3.new(0, 0.8, 0) or Color3.new(0.8, 0, 0))
        }
    end
    local hookStroke = hookBtn:FindFirstChild("UIStroke")
    if hookStroke then
        hookStroke.Color = hookToggle and Color3.new(0, 0.8, 0) or Color3.new(1, 0.2, 0.2)
    end
end)

-- 3. Cloak Invis
cloakBtn.MouseButton1Click:Connect(function()
    cloakToggle = not cloakToggle
    cloakBtn.Text = "👻 Cloak Invis: " .. (cloakToggle and "ON" or "OFF")
    cloakBtn.BackgroundColor3 = cloakToggle and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
    cloakBtn.TextColor3 = Color3.new(0, 0, 0)  -- Чёрный текст
    local cloakGradient = cloakBtn:FindFirstChild("UIGradient")
    if cloakGradient then
        cloakGradient.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, cloakToggle and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)),
            ColorSequenceKeypoint.new(1, cloakToggle and Color3.new(0, 0.8, 0) or Color3.new(0.8, 0, 0))
        }
    end
    local cloakStroke = cloakBtn:FindFirstChild("UIStroke")
    if cloakStroke then
        cloakStroke.Color = cloakToggle and Color3.new(0, 0.8, 0) or Color3.new(1, 0.2, 0.2)
    end
end)

-- Основной цикл
RunService.Heartbeat:Connect(function()
    if not char or not rootPart then return end
   
    -- Hook Fly
    if hookToggle and hasTool("Grapple Hook") then
        local holding, brainrotPart = hasBrainrot()
        if holding and basePos then
            if not bodyVelocity then
                bodyVelocity = Instance.new("BodyVelocity")
                bodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
                bodyVelocity.Velocity = Vector3.new(0, 0, 0)
                bodyVelocity.Parent = rootPart
            end
            local direction = (basePos - rootPart.Position).Unit
            bodyVelocity.Velocity = direction * 100 -- Быстрый полёт
            humanoid.PlatformStand = true -- Чтобы не падал
            -- Стоп при прибытии
            if (rootPart.Position - basePos).Magnitude < 10 then
                bodyVelocity:Destroy()
                bodyVelocity = nil
                humanoid.PlatformStand = false
            end
        else
            if bodyVelocity then
                bodyVelocity:Destroy()
                bodyVelocity = nil
                humanoid.PlatformStand = false
            end
        end
    else
        if bodyVelocity then
            bodyVelocity:Destroy()
            bodyVelocity = nil
            humanoid.PlatformStand = false
        end
    end
   
    -- Cloak Invis
    if cloakToggle and hasTool("Invisibility Cloak") then
        local holding = hasBrainrot()
        if holding then
            -- Экипируем и активируем плащ
            local cloak = player.Backpack:FindFirstChild("Invisibility Cloak") or (char and char:FindFirstChild("Invisibility Cloak"))
            if cloak and not char:FindFirstChild("Invisibility Cloak") then
                humanoid:EquipTool(cloak)
                wait(0.1)
                cloak:Activate() -- Активация инструмента
            end
            -- Делаем невидимым (клиент + попытка реплики)
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                    part.Transparency = 1
                end
            end
            local _, brainrot = hasBrainrot()
            if brainrot then brainrot.Transparency = 1 end
            -- Noclip для stealth
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
            -- Speed boost от плаща
            humanoid.WalkSpeed = 50
        else
            -- Возврат видимости
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                    part.Transparency = 0
                end
            end
            humanoid.WalkSpeed = 16
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end
end)

print("Панелька стала ещё красивее! Чёрный текст на кнопках, hover-эффекты, тени и кнопка закрытия. Всё супер! 😎")
