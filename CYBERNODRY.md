--[[
    HUB: CF HUB ANCOLIDE (MODIFICADO)
    FUNÇÕES: GOD, AUTO-BACK, JUMP, NOCLIP, SPEED SELECTOR, MINIMIZAR
--]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- ESTADOS
local noclipActive, godModeActive, autoBackActive, jumpActive = false, false, false, false
local lastPosSaved = nil
local isTeleporting = false
local minimized = false
local walkSpeedValue = 16

-- 1. INTERFACE
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "CF_Hub_Ancolide"
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 450, 0, 320)
mainFrame.Position = UDim2.new(0.5, -225, 0.5, -160)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 10)

-- TÍTULO
local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, -100, 0, 40)
title.Position = UDim2.new(0, 15, 0, 5)
title.Text = "CF Hub Ancolide - Speed Edition"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextXAlignment = Enum.TextXAlignment.Left

-- BOTÕES DE TOPO
local closeBtn = Instance.new("TextButton", mainFrame)
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -40, 0, 10)
closeBtn.Text = "X"
closeBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
closeBtn.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", closeBtn)

local minBtn = Instance.new("TextButton", mainFrame)
minBtn.Size = UDim2.new(0, 30, 0, 30)
minBtn.Position = UDim2.new(1, -75, 0, 10)
minBtn.Text = "-"
minBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
minBtn.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", minBtn)

local content = Instance.new("Frame", mainFrame)
content.Size = UDim2.new(1, 0, 1, -40)
content.Position = UDim2.new(0, 0, 0, 40)
content.BackgroundTransparency = 1

-- COLUNAS
local function createContainer(name, pos)
    local lbl = Instance.new("TextLabel", content)
    lbl.Text = name
    lbl.Position = pos
    lbl.Size = UDim2.new(0.45, 0, 0, 20)
    lbl.TextColor3 = Color3.fromRGB(180, 180, 180)
    lbl.Font = Enum.Font.GothamBold
    lbl.BackgroundTransparency = 1
    
    local f = Instance.new("Frame", content)
    f.Size = UDim2.new(0.45, 0, 0.65, 0)
    f.Position = pos + UDim2.new(0, 0, 0, 25)
    f.BackgroundTransparency = 1
    return f
end

local leftCol = createContainer("Hubs Principais", UDim2.new(0.03, 0, 0.1, 0))
local rightCol = createContainer("Ajustar Velocidade", UDim2.new(0.52, 0, 0.1, 0))

local function createBtn(text, parent, y)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, 0, 0, 35)
    btn.Position = UDim2.new(0, 0, 0, y)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.Gotham
    Instance.new("UICorner", btn)
    return btn
end

-- BOTÕES ESQUERDA
local noclipBtn = createBtn("NOCLIP: OFF", leftCol, 0)
local godBtn = createBtn("GOD MODE: OFF", leftCol, 45)
local backBtn = createBtn("AUTO-BACK: OFF", leftCol, 90)
local jumpBtn = createBtn("PULO INF: OFF", leftCol, 135)

-- CONFIGURAÇÃO DE VELOCIDADE (DIREITA)
local speedInput = Instance.new("TextBox", rightCol)
speedInput.Size = UDim2.new(1, 0, 0, 40)
speedInput.Position = UDim2.new(0, 0, 0, 0)
speedInput.PlaceholderText = "Digite a velocidade..."
speedInput.Text = "16"
speedInput.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
speedInput.TextColor3 = Color3.new(1, 1, 1)
speedInput.Font = Enum.Font.GothamBold
Instance.new("UICorner", speedInput)

local setSpeedBtn = createBtn("APLICAR SPEED", rightCol, 50)
setSpeedBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)

-- 2. LÓGICA MINIMIZAR
minBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    content.Visible = not minimized
    mainFrame:TweenSize(minimized and UDim2.new(0, 450, 0, 45) or UDim2.new(0, 450, 0, 320), "Out", "Quad", 0.3, true)
    minBtn.Text = minimized and "+" or "-"
end)

-- 3. MOTOR UNIFICADO
RunService.Stepped:Connect(function()
    if player.Character then
        -- NOCLIP
        if noclipActive then
            for _, v in pairs(player.Character:GetDescendants()) do 
                if v:IsA("BasePart") then v.CanCollide = false end 
            end
        end
        
        local hum = player.Character:FindFirstChildOfClass("Humanoid")
        if hum then
            -- GOD MODE
            if godModeActive then hum.Health = 9e9 end
            -- SPEED CONSTANTE
            hum.WalkSpeed = walkSpeedValue
        end
        
        -- AUTO-BACK
        local hrp = player.Character:FindFirstChild("HumanoidRootPart")
        if autoBackActive and hrp and hum and hum.Health > 0 and not isTeleporting then 
            lastPosSaved = hrp.CFrame 
        end
    end
end)

-- LÓGICA DO SPEED
setSpeedBtn.MouseButton1Click:Connect(function()
    local newSpeed = tonumber(speedInput.Text)
    if newSpeed then
        walkSpeedValue = newSpeed
        setSpeedBtn.Text = "SPEED: " .. newSpeed
        task.wait(0.5)
        setSpeedBtn.Text = "APLICAR SPEED"
    else
        speedInput.Text = "Valor Inválido"
    end
end)

-- PULO / BACK / BOTOES
UserInputService.JumpRequest:Connect(function() 
    if jumpActive and player.Character then 
        player.Character.Humanoid:ChangeState("Jumping") 
    end 
end)

player.CharacterAdded:Connect(function(char)
    if autoBackActive and lastPosSaved then
        isTeleporting = true
        local hrp = char:WaitForChild("HumanoidRootPart", 10)
        task.wait(0.2)
        for i = 1, 15 do hrp.CFrame = lastPosSaved task.wait(0.05) end
        isTeleporting = false
    end
end)

noclipBtn.MouseButton1Click:Connect(function() 
    noclipActive = not noclipActive 
    noclipBtn.BackgroundColor3 = noclipActive and Color3.fromRGB(0, 100, 0) or Color3.fromRGB(40,40,40) 
    noclipBtn.Text = noclipActive and "NOCLIP: ON" or "NOCLIP: OFF"
end)

godBtn.MouseButton1Click:Connect(function() 
    godModeActive = not godModeActive 
    godBtn.BackgroundColor3 = godModeActive and Color3.fromRGB(0, 0, 100) or Color3.fromRGB(40,40,40) 
    godBtn.Text = godModeActive and "GOD MODE: ON" or "GOD MODE: OFF"
end)

backBtn.MouseButton1Click:Connect(function() 
    autoBackActive = not autoBackActive 
    backBtn.BackgroundColor3 = autoBackActive and Color3.fromRGB(100, 0, 100) or Color3.fromRGB(40,40,40) 
    backBtn.Text = autoBackActive and "AUTO-BACK: ON" or "AUTO-BACK: OFF"
end)

jumpBtn.MouseButton1Click:Connect(function() 
    jumpActive = not jumpActive 
    jumpBtn.BackgroundColor3 = jumpActive and Color3.fromRGB(0, 100, 100) or Color3.fromRGB(40,40,40) 
    jumpBtn.Text = jumpActive and "PULO INF: ON" or "PULO INF: OFF"
end)

closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)
