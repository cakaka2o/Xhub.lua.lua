-- Código completo para GitHub:
-- loadstring(game:HttpGet("https://raw.githubusercontent.com/Serwxl/Community/main/DiasXHUB"))()

return function()
    --[[
    ===========================================
    XHUB MM2 ULTIMATE - By María & Vxr
    Features:
    - Aimbot automático (pistola/sheriff)
    - Autofarm inteligente de coins
    - ESP de roles (Asesino/Inocente/Sheriff)
    - Optimización Raifield (max FPS)
    - Key system: MARIAXD23W
    ===========================================
    ]]

    -- Configuración
    local _XHUB = {
        KEY = "MARIAXD23W", -- Key permanente
        ESP_COLORS = {
            Murder = Color3.fromRGB(255, 0, 0),
            Sheriff = Color3.fromRGB(0, 0, 255),
            Innocent = Color3.fromRGB(0, 255, 0)
        },
        AIMBOT = {
            FOV = 50, -- Área de detección
            SMOOTHNESS = 0.2, -- Suavizado del aim
            PREDICTION = 0.136, -- Predicción de movimiento
            TRIGGER_KEY = Enum.UserInputType.MouseButton2 -- Botón derecho
        }
    }

    -- Servicios
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local UserInputService = game:GetService("UserInputService")
    local Lighting = game:GetService("Lighting")
    local Workspace = game:GetService("Workspace")

    -- Variables
    local LocalPlayer = Players.LocalPlayer
    local CurrentCamera = Workspace.CurrentCamera
    local ESP_Objects = {}
    local AimbotActive = false
    local AutoFarmActive = false

    -- Función para verificar key
    local function VerifyKey()
        return true -- Key hardcodeada (en un script real usar HTTP)
    end

    -- Optimización Raifield
    local function OptimizeGame()
        settings().Rendering.QualityLevel = 1
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 500
        for _, v in ipairs(Workspace:GetDescendants()) do
            if v:IsA("ParticleEmitter") then v.Enabled = false end
            if v:IsA("Decal") then v:Destroy() end
        end
    end

    -- ESP de roles
    local function CreateESP(player)
        local character = player.Character or player.CharacterAdded:Wait()
        local highlight = Instance.new("Highlight")
        highlight.OutlineColor = _XHUB.ESP_COLORS[player.Status.Role.Value]
        highlight.OutlineTransparency = 0
        highlight.FillTransparency = 0.8
        highlight.Parent = character
        ESP_Objects[player] = highlight
    end

    -- Aimbot avanzado
    local function AimbotThread()
        RunService.Heartbeat:Connect(function()
            if not AimbotActive then return end
            
            local closestPlayer, closestDistance = nil, _XHUB.AIMBOT.FOV
            local localCharacter = LocalPlayer.Character
            local localRoot = localCharacter and localCharacter:FindFirstChild("HumanoidRootPart")
            
            if not localRoot then return end

            -- Buscar objetivo más cercano
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    local enemyRoot = player.Character:FindFirstChild("HumanoidRootPart")
                    if enemyRoot then
                        local screenPos, onScreen = CurrentCamera:WorldToViewportPoint(enemyRoot.Position)
                        if onScreen then
                            local distance = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)).Magnitude
                            if distance < closestDistance then
                                closestPlayer = player
                                closestDistance = distance
                            end
                        end
                    end
                end
            end

            -- Aplicar aimbot
            if closestPlayer then
                local enemyRoot = closestPlayer.Character:FindFirstChild("HumanoidRootPart")
                local predictedPosition = enemyRoot.Position + (enemyRoot.Velocity * _XHUB.AIMBOT.PREDICTION)
                local cameraDirection = (predictedPosition - CurrentCamera.CFrame.Position).Unit
                CurrentCamera.CFrame = CurrentCamera.CFrame:Lerp(CFrame.new(CurrentCamera.CFrame.Position, CurrentCamera.CFrame.Position + cameraDirection), _XHUB.AIMBOT.SMOOTHNESS)
            end
        end)
    end

    -- Autofarm de coins
    local function AutoFarm()
        while AutoFarmActive do
            for _, coin in ipairs(Workspace:GetDescendants()) do
                if coin.Name == "Coin" and coin:IsA("BasePart") then
                    LocalPlayer.Character:MoveTo(coin.Position)
                    task.wait(0.3)
                end
            end
            task.wait(1)
        end
    end

    -- UI Principal
    local function CreateUI()
        local ScreenGui = Instance.new("ScreenGui")
        ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

        local MainFrame = Instance.new("Frame")
        MainFrame.Size = UDim2.new(0, 300, 0, 200)
        MainFrame.Position = UDim2.new(0.5, -150, 0.5, -100)
        MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        MainFrame.Parent = ScreenGui

        -- Botón Aimbot
        local AimbotBtn = Instance.new("TextButton")
        AimbotBtn.Text = "AIMBOT (RMB)"
        AimbotBtn.Size = UDim2.new(0.9, 0, 0, 40)
        AimbotBtn.Position = UDim2.new(0.05, 0, 0.1, 0)
        AimbotBtn.Parent = MainFrame
        AimbotBtn.MouseButton1Click:Connect(function()
            AimbotActive = not AimbotActive
        end)

        -- Botón Autofarm
        local FarmBtn = Instance.new("TextButton")
        FarmBtn.Text = "AUTOFARM COINS"
        FarmBtn.Size = UDim2.new(0.9, 0, 0, 40)
        FarmBtn.Position = UDim2.new(0.05, 0, 0.4, 0)
        FarmBtn.Parent = MainFrame
        FarmBtn.MouseButton1Click:Connect(function()
            AutoFarmActive = not AutoFarmActive
            if AutoFarmActive then
                coroutine.wrap(AutoFarm)()
            end
        end)

        -- Botón Cerrar
        local CloseBtn = Instance.new("TextButton")
        CloseBtn.Text = "CERRAR XHUB"
        CloseBtn.Size = UDim2.new(0.9, 0, 0, 40)
        CloseBtn.Position = UDim2.new(0.05, 0, 0.7, 0)
        CloseBtn.Parent = MainFrame
        CloseBtn.MouseButton1Click:Connect(function()
            ScreenGui:Destroy()
        end)
    end

    -- Inicialización
    if VerifyKey() then
        OptimizeGame()
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                coroutine.wrap(CreateESP)(player)
            end
        end
        Players.PlayerAdded:Connect(CreateESP)
        AimbotThread()
        CreateUI()
    else
        game:Shutdown()
    end
end
