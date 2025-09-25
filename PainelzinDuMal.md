local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Fundo Killua, 75% opacidade
local backgroundImageId = "rbxassetid://101814758219491"
local lastPanelPosition = UDim2.new(0.5, -150, 0.5, -150)
local minimized = false
local guiObjects
local reopenBtn

-- Estados globais que devem ser mantidos após respawn
local aimbotEnabled = false
local circleRadius = 150
local minRadius = 50
local maxRadius = 400
local aimCircle = nil

local function removeGuiByName(name)
    local g = LocalPlayer.PlayerGui:FindFirstChild(name)
    if g then g:Destroy() end
end

function CreateReopenBtn()
    removeGuiByName("ReabrirBtn")
    reopenBtn = Instance.new("TextButton")
    reopenBtn.Name = "ReabrirBtn"
    reopenBtn.Parent = LocalPlayer.PlayerGui
    reopenBtn.Size = UDim2.new(0, 100, 0, 40)
    reopenBtn.Position = UDim2.new(0, 10, 1, -50)
    reopenBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    reopenBtn.Text = "Reabrir"
    reopenBtn.TextColor3 = Color3.new(1,1,1)
    reopenBtn.TextSize = 22
    reopenBtn.Font = Enum.Font.SourceSansBold
    reopenBtn.BorderSizePixel = 0

    local hubCorner = Instance.new("UICorner")
    hubCorner.CornerRadius = UDim.new(0, 12)
    hubCorner.Parent = reopenBtn

    reopenBtn.MouseButton1Click:Connect(function()
        minimized = false
        removeGuiByName("ReabrirBtn")
        guiObjects = CreateAimbotPanel()
        updateConnections()
        -- Atualiza visual do painel ao reabrir
        updatePanelState()
    end)
end

function CreateAimbotPanel()
    removeGuiByName("AimbotGUI")
    removeGuiByName("ReabrirBtn")

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AimbotGUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

    local panel = Instance.new("Frame")
    panel.Size = UDim2.new(0, 320, 0, 280)
    panel.Position = lastPanelPosition
    panel.BackgroundTransparency = 1
    panel.Active = true
    panel.Parent = screenGui

    -- Fundo Killua 75% opacidade
    local bg = Instance.new("ImageLabel")
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.Position = UDim2.new(0, 0, 0, 0)
    bg.Image = backgroundImageId
    bg.BackgroundTransparency = 0.25
    bg.Parent = panel

    local uicornerBG = Instance.new("UICorner")
    uicornerBG.CornerRadius = UDim.new(0, 24)
    uicornerBG.Parent = bg

    -- Título (arrastável)
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 60)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "Painelzinho do QR"
    title.TextColor3 = Color3.new(1,1,1)
    title.TextStrokeTransparency = 0.7
    title.TextStrokeColor3 = Color3.fromRGB(50,50,50)
    title.TextSize = 32
    title.Font = Enum.Font.GothamBold
    title.ZIndex = 2
    title.Parent = panel

    -- Botão de minimizar
    local btnColor = Color3.fromRGB(49, 172, 80)
    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Size = UDim2.new(0, 40, 0, 40)
    minimizeBtn.Position = UDim2.new(1, 16, 0, -6)
    minimizeBtn.AnchorPoint = Vector2.new(1, 0)
    minimizeBtn.Text = "-"
    minimizeBtn.BackgroundColor3 = btnColor
    minimizeBtn.BackgroundTransparency = 0.25
    minimizeBtn.TextColor3 = Color3.new(1,1,1)
    minimizeBtn.TextSize = 22
    minimizeBtn.Font = Enum.Font.GothamBold
    minimizeBtn.BorderSizePixel = 0
    minimizeBtn.ZIndex = 10
    minimizeBtn.Parent = panel
    local uicornerMinimize = Instance.new("UICorner")
    uicornerMinimize.CornerRadius = UDim.new(1, 0)
    uicornerMinimize.Parent = minimizeBtn

    -- SLIDER
    local sliderFrame = Instance.new("Frame")
    sliderFrame.Size = UDim2.new(0.85, 0, 0, 28)
    sliderFrame.Position = UDim2.new(0.5, 0, 0.54, 0)
    sliderFrame.AnchorPoint = Vector2.new(0.5, 0)
    sliderFrame.BackgroundTransparency = 1
    sliderFrame.ZIndex = 3
    sliderFrame.Parent = panel

    local sliderBar = Instance.new("Frame")
    sliderBar.Size = UDim2.new(1, -18, 0, 10)
    sliderBar.Position = UDim2.new(0, 9, 0.5, -5)
    sliderBar.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
    sliderBar.BorderSizePixel = 0
    sliderBar.ZIndex = 4
    sliderBar.Parent = sliderFrame
    local sliderBarCorner = Instance.new("UICorner")
    sliderBarCorner.CornerRadius = UDim.new(1,0)
    sliderBarCorner.Parent = sliderBar

    local sliderKnob = Instance.new("Frame")
    sliderKnob.Size = UDim2.new(0, 22, 0, 22)
    sliderKnob.Position = UDim2.new(0.25, -11, 0.5, -11)
    sliderKnob.BackgroundColor3 = btnColor
    sliderKnob.BorderSizePixel = 0
    sliderKnob.ZIndex = 5
    sliderKnob.Parent = sliderFrame
    local sliderKnobCorner = Instance.new("UICorner")
    sliderKnobCorner.CornerRadius = UDim.new(1,0)
    sliderKnobCorner.Parent = sliderKnob

    -- Botão "Aumentar área de tiro"
    local increaseBtn = Instance.new("TextButton")
    increaseBtn.Size = UDim2.new(0.7, 0, 0, 34)
    increaseBtn.Position = UDim2.new(0.5, 0, 0.66, 0)
    increaseBtn.AnchorPoint = Vector2.new(0.5, 0)
    increaseBtn.Text = "Aumentar área de tiro"
    increaseBtn.BackgroundColor3 = btnColor
    increaseBtn.BackgroundTransparency = 0.25
    increaseBtn.TextColor3 = Color3.new(1,1,1)
    increaseBtn.TextStrokeTransparency = 0.8
    increaseBtn.TextSize = 20
    increaseBtn.Font = Enum.Font.GothamBold
    increaseBtn.BorderSizePixel = 0
    increaseBtn.ZIndex = 6
    increaseBtn.Parent = panel
    local uicornerIncrease = Instance.new("UICorner")
    uicornerIncrease.CornerRadius = UDim.new(0, 12)
    uicornerIncrease.Parent = increaseBtn

    -- Botão "Ativar/Desativar Aimbot"
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.7, 0, 0, 34)
    button.Position = UDim2.new(0.5, 0, 0.80, 0)
    button.AnchorPoint = Vector2.new(0.5, 0)
    button.Text = "Ativar Aimbot"
    button.BackgroundColor3 = btnColor
    button.BackgroundTransparency = 0.25
    button.TextColor3 = Color3.new(1,1,1)
    button.TextStrokeTransparency = 0.7
    button.TextSize = 20
    button.Font = Enum.Font.GothamBold
    button.BorderSizePixel = 0
    button.ZIndex = 7
    button.Parent = panel
    local uicornerButton = Instance.new("UICorner")
    uicornerButton.CornerRadius = UDim.new(0, 12)
    uicornerButton.Parent = button

    -- Arrastar só pelo título
    local dragging, dragInput, dragStart, startPos
    title.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = panel.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    title.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            lastPanelPosition = panel.Position
        end
    end)

    minimizeBtn.MouseButton1Click:Connect(function()
        minimized = true
        lastPanelPosition = panel.Position
        if screenGui then screenGui:Destroy() end
        RunService.RenderStepped:Wait()
        CreateReopenBtn()
    end)

    return {
        button = button,
        increaseBtn = increaseBtn,
        sliderKnob = sliderKnob,
        sliderBar = sliderBar,
        panel = panel
    }
end

function drawCircle()
    if aimCircle then aimCircle:Remove() end
    aimCircle = Drawing.new("Circle")
    aimCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
    aimCircle.Radius = circleRadius
    aimCircle.Thickness = 2
    aimCircle.Color = Color3.fromRGB(100, 200, 255)
    aimCircle.Transparency = 1
    aimCircle.Visible = true
end

function getTarget()
    local camera = workspace.CurrentCamera
    local center = Vector2.new(camera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
    local closestPlayer, closestDist = nil, circleRadius

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local headPos = camera:WorldToViewportPoint(player.Character.Head.Position)
            local screenPos = Vector2.new(headPos.X, headPos.Y)
            local dist = (screenPos - center).Magnitude
            if dist < closestDist then
                closestDist = dist
                closestPlayer = player
            end
        end
    end
    return closestPlayer
end

function updateSliderKnob()
    if guiObjects and guiObjects.sliderKnob then
        local percent = (circleRadius-minRadius)/(maxRadius-minRadius)
        guiObjects.sliderKnob.Position = UDim2.new(percent, -11, 0.5, -11)
    end
end

-- Atualiza visual do painel (botão e círculo) de acordo com estado dos globais
function updatePanelState()
    if not guiObjects then return end
    if aimbotEnabled then
        drawCircle()
        guiObjects.button.Text = "Desativar Aimbot"
        guiObjects.button.BackgroundColor3 = Color3.fromRGB(200,30,30)
    else
        if aimCircle then aimCircle:Remove() aimCircle = nil end
        guiObjects.button.Text = "Ativar Aimbot"
        guiObjects.button.BackgroundColor3 = Color3.fromRGB(49, 172, 80)
    end
    updateSliderKnob()
end

function updateConnections()
    if not guiObjects then return end

    -- Remove conexões antigas se existirem
    if guiObjects._buttonConn then guiObjects._buttonConn:Disconnect() end
    if guiObjects._increaseConn then guiObjects._increaseConn:Disconnect() end
    if guiObjects._sliderBarBeginConn then guiObjects._sliderBarBeginConn:Disconnect() end
    if guiObjects._sliderBarEndConn then guiObjects._sliderBarEndConn:Disconnect() end
    if guiObjects._sliderKnobBeginConn then guiObjects._sliderKnobBeginConn:Disconnect() end
    if guiObjects._sliderKnobEndConn then guiObjects._sliderKnobEndConn:Disconnect() end

    guiObjects._buttonConn = guiObjects.button.MouseButton1Click:Connect(function()
        aimbotEnabled = not aimbotEnabled
        updatePanelState()
    end)

    guiObjects._increaseConn = guiObjects.increaseBtn.MouseButton1Click:Connect(function()
        circleRadius = math.min(circleRadius + 10, maxRadius)
        updatePanelState()
    end)

    -- Slider: clique ou arraste para alterar FOV
    local draggingSlider = false
    local function setFovFromX(x)
        local sliderAbs = guiObjects.sliderBar.AbsolutePosition.X
        local sliderW = guiObjects.sliderBar.AbsoluteSize.X
        local percent = math.clamp((x-sliderAbs)/sliderW, 0, 1)
        circleRadius = math.floor(minRadius + percent*(maxRadius-minRadius))
        updatePanelState()
    end

    guiObjects._sliderBarBeginConn = guiObjects.sliderBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            draggingSlider = true
            setFovFromX(input.Position.X)
        end
    end)
    guiObjects._sliderBarEndConn = guiObjects.sliderBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            draggingSlider = false
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if draggingSlider and input.UserInputType == Enum.UserInputType.MouseMovement then
            setFovFromX(input.Position.X)
        end
    end)
    guiObjects._sliderKnobBeginConn = guiObjects.sliderKnob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            draggingSlider = true
        end
    end)
    guiObjects._sliderKnobEndConn = guiObjects.sliderKnob.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            draggingSlider = false
        end
    end)
end

guiObjects = CreateAimbotPanel()
updateConnections()
updatePanelState()

local ShootRemote = nil
for _,v in pairs(getgc(true)) do
    if typeof(v) == "Instance" and v:IsA("RemoteEvent") and v.Name:lower():find("shoot") then
        ShootRemote = v
        break
    end
end

UIS.InputBegan:Connect(function(input, gp)
    if not aimbotEnabled then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local target = getTarget()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            if ShootRemote then
                ShootRemote:FireServer(target.Character.Head.Position)
            else
                local camera = workspace.CurrentCamera
                camera.CFrame = CFrame.new(camera.CFrame.Position, target.Character.Head.Position)
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    if aimCircle and aimbotEnabled then
        aimCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
        aimCircle.Radius = circleRadius
        aimCircle.Visible = true
    elseif aimCircle then
        aimCircle.Visible = false
    end
end)

Players.LocalPlayer.CharacterAdded:Connect(function()
    while not LocalPlayer:FindFirstChild("PlayerGui") do task.wait() end
    local panelExists = LocalPlayer.PlayerGui:FindFirstChild("AimbotGUI")
    local reopenExists = LocalPlayer.PlayerGui:FindFirstChild("ReabrirBtn")
    if minimized then
        if panelExists then panelExists:Destroy() end
        if reopenExists then reopenExists:Destroy() end
        CreateReopenBtn()
    else
        -- Só cria o painel se não existe
        if not panelExists then
            guiObjects = CreateAimbotPanel()
            updateConnections()
            updatePanelState()
        else
            -- Se já existe, só atualiza visual
            updatePanelState()
        end
    end
end)
