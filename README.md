--[[
    FPS BOOSTER AUTOMÁTICO - RONIX EXECUTOR
    Este script será executado automaticamente ao entrar em qualquer jogo
]]

-- Configurações do FPS Booster
local settings = {
    Graphics = {
        QualityLevel = 1,  -- 1 = Mínimo, 2-3 = Baixo, 4 = Médio, 5+ = Alto
        TextureQuality = 1, -- 1-3 (1 = mais baixo)
        ShadowQuality = 1,  -- 1-3 (1 = desligado/mínimo)
        LightingQuality = 1,-- 1-3 (1 = mínima)
        RenderDistance = 500, -- Distância de renderização
        EffectsQuality = 1, -- 1-3 (1 = mínimo)
    },
    
    AutoOptimize = true,    -- Otimização automática
    ShowFPS = true,        -- Mostrar contador de FPS
    FPSLimit = 60,         -- Limite de FPS (0 = sem limite)
}

-- Função principal de otimização
local function OptimizeGame()
    -- Ajustar configurações gráficas
    if settings.AutoOptimize then
        -- Qualidade global
        if settings.Graphics.QualityLevel then
            settings.Graphics.QualityLevel = math.clamp(settings.Graphics.QualityLevel, 1, 10)
        end
        
        -- Otimizações de renderização
        local RunService = game:GetService("RunService")
        local Lighting = game:GetService("Lighting")
        local Workspace = game:GetService("Workspace")
        local Players = game:GetService("Players")
        
        -- Configurações de iluminação
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 500
        Lighting.Brightness = 1
        Lighting.OutdoorAmbient = Color3.new(0.5, 0.5, 0.5)
        
        -- Qualidade das texturas
        if settings.Graphics.TextureQuality then
            settings().Rendering.TextureQuality = settings.Graphics.TextureQuality
        end
        
        -- Qualidade das sombras
        if settings.Graphics.ShadowQuality then
            settings().Rendering.ShadowQuality = settings.Graphics.ShadowQuality
        end
        
        -- Distância de renderização
        Workspace.DescendantAdded:Connect(function(part)
            if part:IsA("BasePart") and not part:IsA("Terrain") then
                if part.Position.Magnitude > settings.Graphics.RenderDistance then
                    part.Transparency = 1
                end
            end
        end)
        
        -- Otimizar partículas e efeitos
        for _, v in pairs(Workspace:GetDescendants()) do
            if v:IsA("ParticleEmitter") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
                v.Enabled = false
            end
            
            if v:IsA("Decal") or v:IsA("Texture") then
                v.Transparency = 1
            end
        end
    end
    
    -- Desabilitar animações desnecessárias
    for _, player in pairs(game:GetService("Players"):GetPlayers()) do
        if player.Character then
            for _, part in pairs(player.Character:GetDescendants()) do
                if part:IsA("Motor6D") then
                    part.MaxVelocity = 0
                end
            end
        end
    end
end

-- Contador de FPS
local function CreateFPSCounter()
    if not settings.ShowFPS then return end
    
    local ScreenGui = Instance.new("ScreenGui")
    local Frame = Instance.new("Frame")
    local TextLabel = Instance.new("TextLabel")
    local UICorner = Instance.new("UICorner")
    
    ScreenGui.Parent = game:GetService("CoreGui")
    ScreenGui.Name = "FPSBoosterGUI"
    ScreenGui.ResetOnSpawn = false
    
    Frame.Parent = ScreenGui
    Frame.Size = UDim2.new(0, 100, 0, 40)
    Frame.Position = UDim2.new(0, 10, 0, 10)
    Frame.BackgroundColor3 = Color3.new(0, 0, 0)
    Frame.BackgroundTransparency = 0.5
    Frame.BorderSizePixel = 0
    
    UICorner.Parent = Frame
    UICorner.CornerRadius = UDim.new(0, 8)
    
    TextLabel.Parent = Frame
    TextLabel.Size = UDim2.new(1, 0, 1, 0)
    TextLabel.BackgroundTransparency = 1
    TextLabel.TextColor3 = Color3.new(0, 1, 0)
    TextLabel.TextScaled = true
    TextLabel.Font = Enum.Font.GothamBold
    TextLabel.Text = "FPS: 0"
    
    local frameCount = 0
    local lastTime = tick()
    
    game:GetService("RunService").RenderStepped:Connect(function()
        frameCount = frameCount + 1
        
        local currentTime = tick()
        if currentTime - lastTime >= 1 then
            local fps = math.floor(frameCount / (currentTime - lastTime))
            TextLabel.Text = "FPS: " .. fps
            frameCount = 0
            lastTime = currentTime
        end
    end)
end

-- Limitar FPS
local function LimitFPS()
    if settings.FPSLimit and settings.FPSLimit > 0 then
        local RunService = game:GetService("RunService")
        local frameTime = 1 / settings.FPSLimit
        local lastTime = tick()
        
        RunService.RenderStepped:Connect(function()
            local currentTime = tick()
            if currentTime - lastTime < frameTime then
                wait(frameTime - (currentTime - lastTime))
            end
            lastTime = tick()
        end)
    end
end

-- Limpar memória periodicamente
local function CleanMemory()
    while true do
        wait(300) -- A cada 5 minutos
        collectgarbage()
        collectgarbage("collect")
    end
end

-- Inicialização automática
local function AutoExecute()
    print("FPS Booster iniciado automaticamente!")
    
    -- Aguardar o jogo carregar completamente
    wait(3)
    
    -- Aplicar otimizações
    OptimizeGame()
    
    -- Criar contador de FPS
    CreateFPSCounter()
    
    -- Limitar FPS
    LimitFPS()
    
    -- Limpeza de memória
    spawn(CleanMemory)
    
    print("FPS Booster configurado com sucesso!")
end

-- Executar automaticamente
AutoExecute()

-- Reconectar automaticamente se reiniciar
game:GetService("Players").LocalPlayer.CharacterAdded:Connect(function()
    wait(1)
    OptimizeGame()
end)
