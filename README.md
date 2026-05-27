--[[
DELTA HUB SYSTEM - COMPLETO
Execute este script para ativar o sistema completo
Comandos: aimbot <força>, farm, teleport, speed <valor>, godmode, fly, esp
SEM VERIFICAÇÃO DE ID - FUNCIONA PARA TODOS!
--]]

local l_Players = game:GetService("Players")
local l_UserInputService = game:GetService("UserInputService")
local l_RunService = game:GetService("RunService")
local l_LocalPlayer = l_Players.LocalPlayer
local l_Camera = workspace.CurrentCamera

-- Aguardar LocalPlayer estar pronto
if not l_LocalPlayer then
    l_LocalPlayer = l_Players.LocalPlayer or l_Players:WaitForChild("LocalPlayer")
end

-- Aguardar PlayerGui estar pronto
local playerGui = l_LocalPlayer:WaitForChild("PlayerGui")

local CONFIG = {
    BG_COLOR = Color3.fromRGB(13, 14, 27),
    ACCENT = Color3.fromRGB(3, 182, 255),
    INFO_COLOR = Color3.fromRGB(180, 180, 180),
    ADMIN_COLOR = Color3.fromRGB(212, 175, 55),
    TEXT_NORMAL = Color3.fromRGB(220, 220, 220),
    FONT = Enum.Font.Code
}

local COMMANDS = {
    {name = "aimbot", info = "<força>"},
    {name = "farm", info = "(ativa/desativa)"},
    {name = "teleport", info = "<player/spawn>"},
    {name = "speed", info = "<valor>"},
    {name = "godmode", info = "(ativa/desativa)"},
    {name = "fly", info = "(ativa/desativa)"},
    {name = "esp", info = "(ativa/desativa)"},
}

-- VARIÁVEIS DOS HACKS
local aimbotActive = false
local aimbotStrength = 1
local aimbotConnection = nil
local farmActive = false
local farmConnection = nil
local godmodeActive = false
local godmodeConnection = nil
local flyActive = false
local flyConnection = nil
local flySpeed = 50
local espActive = false
local espHighlights = {}

-- ==========================================
-- FUNÇÕES DO AIMBOT
-- ==========================================

local function getClosestEnemy()
    local closestPlayer = nil
    local closestDistance = math.huge
    
    for _, player in pairs(l_Players:GetPlayers()) do
        if player ~= l_LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local distance = (player.Character.Head.Position - l_LocalPlayer.Character.Head.Position).Magnitude
            if distance < closestDistance then
                closestDistance = distance
                closestPlayer = player
            end
        end
    end
    
    return closestPlayer
end

local function enableAimbot(strength)
    if aimbotConnection then aimbotConnection:Disconnect() end
    
    aimbotStrength = tonumber(strength) or 1
    aimbotStrength = math.max(0.1, math.min(aimbotStrength, 10))
    
    aimbotConnection = l_RunService.RenderStepped:Connect(function()
        if not aimbotActive or not l_LocalPlayer.Character or not l_LocalPlayer.Character:FindFirstChild("Head") then
            return
        end
        
        local enemy = getClosestEnemy()
        if enemy and enemy.Character and enemy.Character:FindFirstChild("Head") then
            local targetPosition = enemy.Character.Head.Position
            local cameraPosition = l_Camera.CFrame.Position
            local direction = (targetPosition - cameraPosition).Unit
            local newCFrame = CFrame.new(cameraPosition, cameraPosition + direction)
            l_Camera.CFrame = l_Camera.CFrame:Lerp(newCFrame, 0.1 * aimbotStrength)
        end
    end)
end

local function disableAimbot()
    if aimbotConnection then
        aimbotConnection:Disconnect()
        aimbotConnection = nil
    end
end

-- ==========================================
-- FUNÇÕES DO FARM
-- ==========================================

local function enableFarm()
    if farmConnection then farmConnection:Disconnect() end
    
    farmConnection = l_RunService.Heartbeat:Connect(function()
        if not farmActive or not l_LocalPlayer.Character then
            return
        end
        
        for _, item in pairs(workspace:GetDescendants()) do
            if item:IsA("Part") or item:IsA("Model") then
                local name = item.Name:lower()
                
                if (name:find("coin") or name:find("moeda") or name:find("money") or name:find("cash")) then
                    if item:FindFirstChild("ClickDetector") then
                        pcall(function()
                            item.ClickDetector:FireServer()
                        end)
                    elseif item.Parent and item.Parent:FindFirstChild("ClickDetector") then
                        pcall(function()
                            item.Parent.ClickDetector:FireServer()
                        end)
                    end
                end
            end
        end
    end)
end

local function disableFarm()
    if farmConnection then
        farmConnection:Disconnect()
        farmConnection = nil
    end
end

-- ==========================================
-- FUNÇÕES DO GODMODE
-- ==========================================

local function enableGodmode()
    if godmodeConnection then godmodeConnection:Disconnect() end
    
    local character = l_LocalPlayer.Character
    if character and character:FindFirstChild("Humanoid") then
        character.Humanoid.Health = character.Humanoid.MaxHealth
    end
    
    godmodeConnection = l_RunService.Heartbeat:Connect(function()
        if not godmodeActive or not l_LocalPlayer.Character then return end
        
        if l_LocalPlayer.Character:FindFirstChild("Humanoid") then
            l_LocalPlayer.Character.Humanoid.Health = l_LocalPlayer.Character.Humanoid.MaxHealth
        end
    end)
end

local function disableGodmode()
    if godmodeConnection then
        godmodeConnection:Disconnect()
        godmodeConnection = nil
    end
end

-- ==========================================
-- FUNÇÕES DO FLY
-- =========================================

local function enableFly()
    local character = l_LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local rootPart = character.HumanoidRootPart
    
    -- Remover objetos antigos de fly
    for _, obj in pairs(rootPart:GetChildren()) do
        if obj:IsA("BodyVelocity") or obj:IsA("BodyGyro") then
            obj:Destroy()
        end
    end
    
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bodyVelocity.Parent = rootPart
    
    local bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bodyGyro.P = 10000
    bodyGyro.CFrame = rootPart.CFrame
    bodyGyro.Parent = rootPart
    
    if flyConnection then flyConnection:Disconnect() end
    
    flyConnection = l_RunService.RenderStepped:Connect(function()
        if not flyActive or not character or not rootPart.Parent then
            return
        end
        
        local moveDirection = Vector3.new(0, 0, 0)
        
        if l_UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection = moveDirection + (l_Camera.CFrame.LookVector * Vector3.new(1, 0, 1)).Unit
        end
        if l_UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - (l_Camera.CFrame.LookVector * Vector3.new(1, 0, 1)).Unit
        end
        if l_UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection = moveDirection - l_Camera.CFrame.RightVector
        end
        if l_UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + l_Camera.CFrame.RightVector
        end
        if l_UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection = moveDirection + Vector3.new(0, 1, 0)
        end
        if l_UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
            moveDirection = moveDirection - Vector3.new(0, 1, 0)
        end
        
        if moveDirection.Magnitude > 0 then
            moveDirection = moveDirection.Unit
        end
        
        bodyVelocity.Velocity = moveDirection * flySpeed
        bodyGyro.CFrame = l_Camera.CFrame
    end)
end

local function disableFly()
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    
    local character = l_LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local rootPart = character.HumanoidRootPart
        for _, obj in pairs(rootPart:GetChildren()) do
            if obj:IsA("BodyVelocity") or obj:IsA("BodyGyro") then
                obj:Destroy()
            end
        end
    end
end

-- ==========================================
-- FUNÇÕES DE TELEPORTE
-- ==========================================

local function teleportToPlayer(playerName)
    local targetPlayer = nil
    for _, player in pairs(l_Players:GetPlayers()) do
        if player.Name:lower():find(playerName:lower()) then
            targetPlayer = player
            break
        end
    end
    
    if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        if l_LocalPlayer.Character and l_LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            l_LocalPlayer.Character.HumanoidRootPart.CFrame = targetPlayer.Character.HumanoidRootPart.CFrame + Vector3.new(5, 0, 0)
            return true
        end
    end
    return false
end

local function teleportToSpawn()
    local spawnLocation = workspace:FindFirstChild("SpawnLocation") or workspace:FindFirstChildOfClass("SpawnLocation")
    if spawnLocation then
        if l_LocalPlayer.Character and l_LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            l_LocalPlayer.Character.HumanoidRootPart.CFrame = spawnLocation.CFrame + Vector3.new(0, 5, 0)
            return true
        end
    end
    return false
end

-- ==========================================
-- FUNÇÕES DO ESP COM HIGHLIGHT
-- ==========================================

local function createESPHighlight(player)
    if not player.Character then return end
    
    local highlight = Instance.new("Highlight")
    highlight.Parent = player.Character
    highlight.FillColor = Color3.fromRGB(0, 150, 255) -- Azul
    highlight.OutlineColor = Color3.fromRGB(0, 200, 255) -- Azul claro
    highlight.FillTransparency = 0.3
    highlight.OutlineTransparency = 0
    
    espHighlights[player] = highlight
end

local function removeESPHighlight(player)
    if espHighlights[player] then
        espHighlights[player]:Destroy()
        espHighlights[player] = nil
    end
end

local function enableESP()
    -- Adicionar highlight para todos os players existentes
    for _, player in pairs(l_Players:GetPlayers()) do
        if player ~= l_LocalPlayer and player.Character then
            createESPHighlight(player)
        end
    end
    
    -- Adicionar highlight quando um novo player entra
    l_Players.PlayerAdded:Connect(function(player)
        if espActive then
            player.CharacterAdded:Connect(function(character)
                if espActive then
                    task.wait(0.1)
                    createESPHighlight(player)
                end
            end)
            
            task.wait(0.1)
            if player.Character then
                createESPHighlight(player)
            end
        end
    end)
end

local function disableESP()
    for player, highlight in pairs(espHighlights) do
        removeESPHighlight(player)
    end
    espHighlights = {}
end

-- --- UI (ESTRUTURA ORIGINAL) ---
local ScreenGui = Instance.new("ScreenGui", playerGui)
ScreenGui.Name = "Delta_Hub_System"
ScreenGui.ResetOnSpawn = false
ScreenGui.DisplayOrder = 999

local ToggleButton = Instance.new("TextButton", ScreenGui)
ToggleButton.Size = UDim2.new(0, 40, 0, 40)
ToggleButton.Position = UDim2.new(1, -50, 0, 150)
ToggleButton.BackgroundColor3 = CONFIG.BG_COLOR
ToggleButton.BackgroundTransparency = 0.2
ToggleButton.Text = ">_"
ToggleButton.TextColor3 = CONFIG.ACCENT
ToggleButton.Font = CONFIG.FONT
ToggleButton.TextSize = 18
ToggleButton.Visible = true
Instance.new("UICorner", ToggleButton).CornerRadius = UDim.new(1, 0)

local Container = Instance.new("Frame", ScreenGui)
Container.Size = UDim2.new(0.95, 0, 0, 0)
Container.Position = UDim2.new(0.5, 0, 1, -20)
Container.AnchorPoint = Vector2.new(0.5, 1)
Container.BackgroundTransparency = 1
Container.AutomaticSize = Enum.AutomaticSize.Y
Container.Visible = false

local UIList = Instance.new("UIListLayout", Container)
UIList.VerticalAlignment = Enum.VerticalAlignment.Bottom
UIList.Padding = UDim.new(0, 5)

-- CAIXA DE LOGS
local LogFrame = Instance.new("Frame", Container)
LogFrame.Size = UDim2.new(1, 0, 0, 0)
LogFrame.AutomaticSize = Enum.AutomaticSize.Y
LogFrame.BackgroundColor3 = CONFIG.BG_COLOR
LogFrame.BackgroundTransparency = 0.15
Instance.new("UICorner", LogFrame).CornerRadius = UDim.new(0, 6)
local logPadding = Instance.new("UIPadding", LogFrame)
logPadding.PaddingLeft = UDim.new(0, 15)
logPadding.PaddingTop = UDim.new(0, 10)
logPadding.PaddingBottom = UDim.new(0, 10)
local logList = Instance.new("UIListLayout", LogFrame)

-- TEXTBOX
local TextBox = Instance.new("TextBox", Container)
TextBox.Size = UDim2.new(1, 0, 0, 40)
TextBox.BackgroundColor3 = CONFIG.BG_COLOR
TextBox.BackgroundTransparency = 0.15
TextBox.TextColor3 = CONFIG.TEXT_NORMAL
TextBox.Font = CONFIG.FONT
TextBox.TextSize = 16
TextBox.TextXAlignment = Enum.TextXAlignment.Left
TextBox.PlaceholderText = "Enter your command"
TextBox.Text = ""
TextBox.Visible = true
Instance.new("UICorner", TextBox).CornerRadius = UDim.new(0, 6)
Instance.new("UIPadding", TextBox).PaddingLeft = UDim.new(0, 15)

local SuggestionFrame = Instance.new("Frame", TextBox)
SuggestionFrame.Size = UDim2.new(0, 250, 0, 0)
SuggestionFrame.Position = UDim2.new(0, -15, 0, -5)
SuggestionFrame.AnchorPoint = Vector2.new(0, 1)
SuggestionFrame.BackgroundColor3 = CONFIG.BG_COLOR
SuggestionFrame.Visible = false
SuggestionFrame.BorderSizePixel = 0
local suggestionList = Instance.new("UIListLayout", SuggestionFrame)
suggestionList.SortOrder = Enum.SortOrder.LayoutOrder

-- --- LÓGICA ---

local simbolos = {"-", "/", "|", "\\"}

local function addLog(txt, col)
    local l = Instance.new("TextLabel", LogFrame)
    l.BackgroundTransparency = 1
    l.Size = UDim2.new(1, 0, 0, 20)
    l.Font = CONFIG.FONT
    l.TextSize = 14
    l.TextColor3 = col or CONFIG.TEXT_NORMAL
    l.Text = txt
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.RichText = true
    l.TextWrapped = true
end

local function showInfo()
    addLog("Delta Hub System v2.0", CONFIG.INFO_COLOR)
    addLog("Copyright (c) Delta Hub", CONFIG.INFO_COLOR)
    addLog("Run 'close-ui' to close this UI.", CONFIG.ADMIN_COLOR)
end

showInfo()

local function runLoadingInInput(seconds)
    local startTime = tick()
    local i = 1
    TextBox.TextEditable = false
    while tick() - startTime < seconds do
        TextBox.Text = simbolos[i] .. " " .. string.format("%.1f", tick() - startTime) .. "s"
        i = (i % #simbolos) + 1
        task.wait(0.1)
    end
    TextBox.Text = ""
    TextBox.TextEditable = true
end

-- Sugestões
TextBox:GetPropertyChangedSignal("Text"):Connect(function()
    if not TextBox.TextEditable then return end
    for _, v in pairs(SuggestionFrame:GetChildren()) do 
        if v:IsA("TextButton") then v:Destroy() end 
    end
    
    local text = TextBox.Text
    if text == "" then 
        SuggestionFrame.Visible = false 
        return 
    end
    
    local args = string.split(text, " ")
    local cmdPart = args[1]:lower()
    local count = 0
    
    if #args == 1 then
        for _, data in pairs(COMMANDS) do
            if data.name:sub(1, #cmdPart) == cmdPart and text:lower() ~= data.name then
                count = count + 1
                local b = Instance.new("TextButton", SuggestionFrame)
                b.Size = UDim2.new(1, 0, 0, 25)
                b.BackgroundTransparency = 1
                b.Text = "  " .. data.name
                b.TextColor3 = CONFIG.ACCENT
                b.Font = CONFIG.FONT
                b.TextSize = 14
                b.TextXAlignment = Enum.TextXAlignment.Left
                b.MouseButton1Click:Connect(function() 
                    TextBox.Text = data.name .. " "
                    TextBox:CaptureFocus() 
                end)
            end
        end
    end
    
    SuggestionFrame.Visible = (count > 0)
    SuggestionFrame.Size = UDim2.new(0, 250, 0, count * 25)
end)

-- PROCESSAMENTO
TextBox.FocusLost:Connect(function(enter)
    if enter and TextBox.Text ~= "" then
        local raw = TextBox.Text
        local args = string.split(raw:gsub("^%s*(.-)%s*$", "%1"), " ")
        local cmdName = args[1]:lower()
        
        SuggestionFrame.Visible = false
        
        task.spawn(function()
            local found = false
            for _, c in pairs(COMMANDS) do 
                if c.name == cmdName then 
                    found = true 
                    break 
                end 
            end
            
            runLoadingInInput(found and 1 or 1)
            
            if found then
                addLog('<font color="#03B6FF">command "' .. cmdName .. '" executes</font>', Color3.new(1,1,1))
                
                if cmdName == "aimbot" then
                    aimbotActive = not aimbotActive
                    local strength = tonumber(args[2]) or 1
                    if aimbotActive then
                        enableAimbot(strength)
                        addLog('<font color="#00FF00">Aimbot ATIVADO! (Força: ' .. strength .. ')</font>', Color3.new(0, 1, 0))
                    else
                        disableAimbot()
                        addLog('<font color="#FF6464">Aimbot DESATIVADO!</font>', Color3.fromRGB(255, 100, 100))
                    end
                
                elseif cmdName == "farm" then
                    farmActive = not farmActive
                    if farmActive then
                        enableFarm()
                        addLog('<font color="#00FF00">Farm ATIVADO!</font>', Color3.new(0, 1, 0))
                    else
                        disableFarm()
                        addLog('<font color="#FF6464">Farm DESATIVADO!</font>', Color3.fromRGB(255, 100, 100))
                    end
                
                elseif cmdName == "teleport" then
                    local target = args[2] or "spawn"
                    local success = false
                    if target:lower() == "spawn" then
                        success = teleportToSpawn()
                    else
                        success = teleportToPlayer(target)
                    end
                    if success then
                        addLog('<font color="#00FF00">Teleportado para ' .. target .. '!</font>', Color3.new(0, 1, 0))
                    else
                        addLog('<font color="#FF6464">Falha ao teleportar!</font>', Color3.fromRGB(255, 100, 100))
                    end
                
                elseif cmdName == "speed" then
                    local speed = tonumber(args[2]) or 16
                    if l_LocalPlayer.Character and l_LocalPlayer.Character:FindFirstChild("Humanoid") then
                        l_LocalPlayer.Character.Humanoid.WalkSpeed = speed
                        addLog('<font color="#00FF00">Velocidade: ' .. speed .. '</font>', Color3.new(0, 1, 0))
                    end
                
                elseif cmdName == "godmode" then
                    godmodeActive = not godmodeActive
                    if godmodeActive then
                        enableGodmode()
                        addLog('<font color="#00FF00">Godmode ATIVADO!</font>', Color3.new(0, 1, 0))
                    else
                        disableGodmode()
                        addLog('<font color="#FF6464">Godmode DESATIVADO!</font>', Color3.fromRGB(255, 100, 100))
                    end
                
                elseif cmdName == "fly" then
                    flyActive = not flyActive
                    if flyActive then
                        enableFly()
                        addLog('<font color="#00FF00">Fly ATIVADO! (WASD+SPACE+CTRL)</font>', Color3.new(0, 1, 0))
                    else
                        disableFly()
                        addLog('<font color="#FF6464">Fly DESATIVADO!</font>', Color3.fromRGB(255, 100, 100))
                    end
                
                elseif cmdName == "esp" then
                    espActive = not espActive
                    if espActive then
                        enableESP()
                        addLog('<font color="#00FF00">ESP ATIVADO! (Highlight Azul)</font>', Color3.new(0, 1, 0))
                    else
                        disableESP()
                        addLog('<font color="#FF6464">ESP DESATIVADO!</font>', Color3.fromRGB(255, 100, 100))
                    end
                
                elseif cmdName == "close-ui" then
                    Container.Visible = false
                end
            else
                addLog('The command "' .. cmdName .. '" does not exist.', Color3.fromRGB(255, 80, 80))
            end
        end)
        
        TextBox.Text = ""
    end
end)

local function toggleUI()
    Container.Visible = not Container.Visible
    if Container.Visible then 
        TextBox:CaptureFocus() 
    end
end

ToggleButton.MouseButton1Click:Connect(toggleUI)

l_UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.X then 
        toggleUI() 
    end
end)

print("✓ Delta Hub System v2.0 iniciado!")
print("✓ Pressione X ou clique em >_ para abrir a UI")
print("✓ Comandos: aimbot <força>, farm, teleport, speed, godmode, fly, esp")
