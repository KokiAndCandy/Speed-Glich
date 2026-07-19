local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer

-- --- UI BẬT TẮT & ẨN HIỆN ---
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "GlitchUI"

-- --- TEXTO "MADE BY KOKI AND CANDY" COLORIDO ---
local textLabel = Instance.new("TextLabel", gui)
textLabel.Size = UDim2.new(0, 280, 0, 40)
textLabel.Position = UDim2.new(0.5, -140, 0.5, -20)
textLabel.BackgroundTransparency = 1
textLabel.Text = "MADE BY KOKI AND CANDY"
textLabel.TextSize = 20
textLabel.TextScaled = false
textLabel.Font = Enum.Font.GothamBold
textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
textLabel.TextStrokeTransparency = 0
textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
textLabel.Visible = true

-- --- FUNÇÃO PARA TEXTO ARCO-ÍRIS ---
local colors = {
    Color3.fromRGB(255, 0, 0),
    Color3.fromRGB(255, 127, 0),
    Color3.fromRGB(255, 255, 0),
    Color3.fromRGB(0, 255, 0),
    Color3.fromRGB(0, 0, 255),
    Color3.fromRGB(75, 0, 130),
    Color3.fromRGB(148, 0, 211)
}
local colorIndex = 1

local function animateText()
    while textLabel.Visible do
        textLabel.TextColor3 = colors[colorIndex]
        colorIndex = colorIndex + 1
        if colorIndex > #colors then
            colorIndex = 1
        end
        task.wait(0.15)
    end
end

coroutine.wrap(function()
    textLabel.Visible = true
    animateText()
end)()

task.delay(4, function()
    textLabel.Visible = false
end)

-- --- BOTÃO ON/OFF ---
local btn = Instance.new("TextButton", gui)
btn.Size = UDim2.new(0, 100, 0, 50)
btn.Position = UDim2.new(0.5, -50, 0.7, 0)
btn.Text = "OFF"
btn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
btn.Draggable = true
btn.TextColor3 = Color3.fromRGB(255, 255, 255)
btn.TextSize = 18
btn.Font = Enum.Font.GothamBold

local enabled = false
local glitchActive = false
local speedIndex = 1
local hasToolEquipped = false
local ladderTriggered = false -- TRUE se já encostou na escada

btn.MouseButton1Click:Connect(function()
    enabled = not enabled
    btn.Text = enabled and "ON" or "OFF"
    btn.BackgroundColor3 = enabled and Color3.fromRGB(50, 255, 50) or Color3.fromRGB(255, 50, 50)
    if not enabled then
        local char = player.Character
        if char then
            local oldGhost = char:FindFirstChild("Skibidi_Ghost_Active")
            if oldGhost then oldGhost:Destroy() end
        end
        glitchActive = false
        speedIndex = 1
        hasToolEquipped = false
        ladderTriggered = false
    end
end)

UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.RightControl then
        gui.Enabled = not gui.Enabled
    end
end)

-- --- DETECÇÃO DE ITEM EQUIPADO ---
local function checkToolEquipped()
    local char = player.Character
    if not char then return false end
    
    for _, child in ipairs(char:GetChildren()) do
        if child:IsA("Tool") then
            return true
        end
    end
    return false
end

-- --- VELOCIDADE (MAIS LENTA) ---
local speedList = {8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28}

local function getNextSpeed()
    local speed = speedList[speedIndex]
    speedIndex = speedIndex + 1
    if speedIndex > #speedList then
        speedIndex = 1
    end
    return speed
end

-- --- SCRIPT DO EFEITO ---
local currentOffset = CFrame.new(0, 0, 0)
local activeGhost, activeWeld = nil, nil
local cachedRoot, cachedTorso = nil, nil

local function deployGhostGlitch()
    if not enabled then return end
    local char = player.Character
    if not char then return end
    
    cachedRoot = char:FindFirstChild("HumanoidRootPart")
    cachedTorso = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
    
    if not cachedTorso or not cachedRoot then return end

    hasToolEquipped = checkToolEquipped()
    
    -- Só ativa se: encostou na escada E tem item equipado
    if ladderTriggered and hasToolEquipped then
        glitchActive = true
    else
        glitchActive = false
        -- Remove o ghost se não tiver as condições
        local oldGhost = char:FindFirstChild("Skibidi_Ghost_Active")
        if oldGhost then oldGhost:Destroy() end
        activeGhost = nil
        activeWeld = nil
        return
    end
    
    local speed = getNextSpeed()
    local offsetMultiplier = -speed -- NEGATIVO para trás
    
    local rawCF = cachedRoot.CFrame:ToObjectSpace(cachedTorso.CFrame)
    currentOffset = CFrame.new(rawCF.Position * (20 * offsetMultiplier)) * rawCF.Rotation

    local oldGhost = char:FindFirstChild("Skibidi_Ghost_Active")
    if oldGhost then
        oldGhost:Destroy() 
    end

    local newGhost = Instance.new("Part")
    newGhost.Name = "Skibidi_Ghost_Active"
    newGhost.Size = Vector3.new(1, 1, 1)
    newGhost.CFrame = cachedTorso.CFrame 
    newGhost.Transparency = 1 
    newGhost.CanCollide = false
    newGhost.Parent = char

    activeWeld = Instance.new("Weld")
    activeWeld.Part0 = cachedTorso
    activeWeld.Part1 = newGhost
    activeWeld.C0 = currentOffset
    activeWeld.Parent = newGhost

    activeGhost = newGhost
end

local function setupListeners(char)
    if not char then return end
    
    local humanoid = char:FindFirstChild("Humanoid")
    if humanoid then
        humanoid.Died:Connect(function()
            if not enabled then return end
            activeGhost = nil
            activeWeld = nil
            cachedRoot = nil
            cachedTorso = nil
            glitchActive = false
            speedIndex = 1
            hasToolEquipped = false
            ladderTriggered = false
        end)
        
        -- Detecta quando entra na escada
        humanoid.StateChanged:Connect(function(oldState, newState)
            if not enabled then return end
            
            if newState == Enum.HumanoidStateType.Climbing then
                -- ENCOSTOU NA ESCADA! Ativa o trigger
                ladderTriggered = true
                -- Se já tiver item equipado, ativa na hora
                if checkToolEquipped() then
                    deployGhostGlitch()
                end
            end
        end)
    end
    
    task.defer(deployGhostGlitch)
    
    -- Detecta quando equipa item
    char.ChildAdded:Connect(function(child)
        if not enabled then return end
        if child:IsA("Tool") then
            hasToolEquipped = true
            -- Se já encostou na escada, ativa
            if ladderTriggered then
                deployGhostGlitch()
            end
        end
    end)
    
    -- Detecta quando remove item (DESATIVA)
    char.ChildRemoved:Connect(function(child)
        if not enabled then return end
        if child:IsA("Tool") then
            hasToolEquipped = false
            glitchActive = false
            speedIndex = 1
            -- Remove o ghost
            local oldGhost = char:FindFirstChild("Skibidi_Ghost_Active")
            if oldGhost then oldGhost:Destroy() end
            activeGhost = nil
            activeWeld = nil
            -- Reseta o trigger da escada? (opcional)
            -- ladderTriggered = false
        end
    end)
end

if player.Character then setupListeners(player.Character) end
player.CharacterAdded:Connect(setupListeners)

RunService.Heartbeat:Connect(function()
    if not enabled then return end
    
    hasToolEquipped = checkToolEquipped()
    
    -- Se perdeu o item, desativa
    if not hasToolEquipped then
        glitchActive = false
        return
    end
    
    -- Se não encostou na escada, não ativa
    if not ladderTriggered then
        glitchActive = false
        return
    end
    
    -- Atualiza offset
    if glitchActive then
        local char = player.Character
        if char then
            local root = char:FindFirstChild("HumanoidRootPart")
            local torso = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
            if root and torso then
                local speed = getNextSpeed()
                local rawCF = root.CFrame:ToObjectSpace(torso.CFrame)
                currentOffset = CFrame.new(rawCF.Position * (20 * -speed)) * rawCF.Rotation
            end
        end
    end
    
    if activeWeld then
        activeWeld.C0 = currentOffset
    else
        if player.Character and ladderTriggered and hasToolEquipped then
            deployGhostGlitch()
        end
    end
end)
