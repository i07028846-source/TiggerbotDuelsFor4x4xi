-- Servicios esenciales de Roblox
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Variables del jugador y la cámara
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local triggerbotActivo = false 

----------------------------------------------------------------
-- INTERFAZ GRÁFICA AJUSTADA
----------------------------------------------------------------
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "DH_LegitTrigger"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 220, 0, 55)
frame.Position = UDim2.new(0.5, -110, 0, 15) 
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Parent = screenGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 6)
uiCorner.Parent = frame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 0.6, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Font = Enum.Font.SourceSansBold
statusLabel.TextSize = 16
statusLabel.TextColor3 = Color3.fromRGB(255, 50, 50) 
statusLabel.Text = "MIRA MANDO: DESACTIVADO"
statusLabel.Parent = frame

local hideHintLabel = Instance.new("TextLabel")
hideHintLabel.Size = UDim2.new(1, 0, 0.4, 0)
hideHintLabel.Position = UDim2.new(0, 0, 0.55, 0)
hideHintLabel.BackgroundTransparency = 1
hideHintLabel.Font = Enum.Font.SourceSansItalic
hideHintLabel.TextSize = 11
hideHintLabel.TextColor3 = Color3.fromRGB(150, 150, 150) 
hideHintLabel.Text = "Presiona L3 para Activar y Ocultar"
hideHintLabel.Parent = frame

local function actualizarInterfaz()
    if triggerbotActivo then
        statusLabel.Text = "MIRA MANDO: ACTIVADO"
        statusLabel.TextColor3 = Color3.fromRGB(50, 255, 50) 
        frame.Visible = false
    else
        statusLabel.Text = "MIRA MANDO: DESACTIVADO"
        statusLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
        frame.Visible = true
    end
end

----------------------------------------------------------------
-- LÓGICA DE DETECCIÓN INSTANTÁNEA GLOBAL (TODO EL CUERPO)
----------------------------------------------------------------
local function comprobarMiraMando()
    local rayOrigin = Camera.CFrame.Position
    local rayDirection = Camera.CFrame.LookVector * 400 
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    
    if LocalPlayer.Character then
        raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, workspace.CurrentCamera}
    end
    
    local raycastResult = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
    
    if raycastResult and raycastResult.Instance then
        local parteTocada = raycastResult.Instance
        -- Buscamos si la parte golpeada por el rayo pertenece al modelo de un personaje
        local character = parteTocada:FindFirstAncestorOfClass("Model")
        
        if character and character:FindFirstChild("Humanoid") then
            local enemigo = Players:GetPlayerFromCharacter(character)
            -- MODIFICADO: Se elimina el filtro estricto de nombres de extremidades.
            -- Si es un oponente válido y está vivo, devuelve positivo inmediatamente.
            if enemigo ~= LocalPlayer and character.Humanoid.Health > 0 then
                return true
            end
        end
    end
    return false
end

local function forzarDisparo()
    local character = LocalPlayer.Character
    if character then
        local armaEquipada = character:FindFirstChildOfClass("Tool")
        if armaEquipada then
            local nombreArma = armaEquipada.Name
            
            -- FILTRO ANTI-CUCHILLO Y PUÑOS REQUERIDO PARA DH
            if string.match(nombreArma, "Knife") or string.match(nombreArma, "combat") or string.match(nombreArma, "Combat") then
                return 
            end
            
            -- 0 DELAY: Disparo inmediato
            armaEquipada:Activate()
            
            local clickDetector = armaEquipada:FindFirstChild("Click") or armaEquipada:FindFirstChild("OnFire")
            if clickDetector and clickDetector:IsA("BindableEvent") then
                clickDetector:Fire()
            end
        end
    end
end

-- Bucle de fotogramas puro enganchado al refresco de la pantalla
RunService.RenderStepped:Connect(function()
    if triggerbotActivo then
        if comprobarMiraMando() then
            forzarDisparo()
        end
    end
end)

----------------------------------------------------------------
-- CONTROL UNIFICADO EN UN SOLO BOTÓN (L3)
----------------------------------------------------------------
UserInputService.InputBegan:Connect(function(input)
    if string.find(tostring(input.UserInputType), "Gamepad") then
        if input.KeyCode == Enum.KeyCode.ButtonL3 then 
            triggerbotActivo = not triggerbotActivo
            actualizarInterfaz()
        end
    end
end)
