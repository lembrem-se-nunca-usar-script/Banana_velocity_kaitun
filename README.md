kaitun banana velocity - VERSÃO DELTA EXECUTOR

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")

------------------------------------------------------------------------
-- 1. SISTEMA DE LOG & DEBUG
------------------------------------------------------------------------
local LogAtivo = true
local function Log(titulo, mensagem, tipo)
    tipo = tipo or "INFO"
    if LogAtivo then
        local timestamp = os.date("%H:%M:%S")
        print("[" .. timestamp .. "] [" .. tipo .. "] " .. titulo .. ": " .. tostring(mensagem))
    end
end

------------------------------------------------------------------------
-- 2. CONFIGURAÇÕES PRINCIPAIS
------------------------------------------------------------------------
_G.AutoFarm = true
_G.AutoGodhuman = true
_G.AutoNewWorld = true      
_G.AutoSpinFruit = true     
_G.BringMobs = true        
_G.AntiBounty = true         
_G.SafeFlyBypass = true      
_G.DistanciaDoInimigo = 40
_G.FarmAteNivelMax = true

_G.Webhook_Config = {
    ["Ativado"] = false, -- Desativado por padrão no executor
    ["Url"] = "SEU_WEBHOOK_AQUI"
}

local NomeFicheiroConfig = "BananaKaitun_Turbo_Config.txt"

------------------------------------------------------------------------
-- 3. DETECÇÃO AUTOMÁTICA DE ESTRUTURA DO JOGO
------------------------------------------------------------------------
local GameStructure = {
    MobsFolder = nil,
    RemotesFolder = nil,
    NPC_Prefix = nil,
    CurrentWorld = "Unknown",
    CurrentLevel = 0
}

local function DetectarEstruturaj()
    Log("Estrutura", "Detectando estrutura do jogo...", "INFO")
    
    -- Procurar pasta de inimigos
    GameStructure.MobsFolder = Workspace:FindFirstChild("Enemies") or Workspace:FindFirstChild("Mobs") or Workspace:FindFirstChild("NPCs")
    if GameStructure.MobsFolder then
        Log("Estrutura", "Pasta de inimigos encontrada: " .. GameStructure.MobsFolder.Name, "SUCCESS")
    else
        Log("Estrutura", "Pasta de inimigos não encontrada. Procurando alternativas...", "WARN")
        -- Procurar por padrão
        for _, folder in pairs(Workspace:GetChildren()) do
            if folder:IsA("Folder") and string.lower(folder.Name):find("enemi") or string.lower(folder.Name):find("mob") then
                GameStructure.MobsFolder = folder
                Log("Estrutura", "Pasta alternativa encontrada: " .. folder.Name, "SUCCESS")
                break
            end
        end
    end
    
    -- Procurar Remotes
    GameStructure.RemotesFolder = ReplicatedStorage:FindFirstChild("Remotes") or ReplicatedStorage:FindFirstChild("RPC")
    if GameStructure.RemotesFolder then
        Log("Estrutura", "Remotes encontradas", "SUCCESS")
    else
        Log("Estrutura", "Remotes não encontradas", "WARN")
    end
    
    -- Detectar mundo atual
    pcall(function()
        if LocalPlayer:FindFirstChild("Data") then
            GameStructure.CurrentWorld = LocalPlayer.Data:FindFirstChild("WorldName") and LocalPlayer.Data.WorldName.Value or "Unknown"
            GameStructure.CurrentLevel = LocalPlayer.Data:FindFirstChild("Level") and LocalPlayer.Data.Level.Value or 0
            Log("Estrutura", "Mundo: " .. GameStructure.CurrentWorld .. " | Nível: " .. GameStructure.CurrentLevel, "INFO")
        end
    end)
end

------------------------------------------------------------------------
-- 4. FUNÇÕES SEGURAS COM TRATAMENTO DE ERRO
------------------------------------------------------------------------
local function TeleportarComSeguranca(cframe_destino)
    if not cframe_destino then
        Log("Teleporte", "CFrame inválido", "ERROR")
        return false
    end
    
    local sucesso, erro = pcall(function()
        local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local root = char:WaitForChild("HumanoidRootPart", 5)
        
        if root then
            if _G.SafeFlyBypass and (root.Position - cframe_destino.Position).Magnitude > 300 then
                local bv = Instance.new("BodyVelocity")
                bv.Name = "TurboVelocity"
                bv.Velocity = Vector3.new(0,0,0)
                bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
                bv.Parent = root
                root.CFrame = cframe_destino
                task.wait(0.1)
                bv:Destroy()
            else
                root.CFrame = cframe_destino
            end
            return true
        end
    end)
    
    if not sucesso then
        Log("Teleporte", "Erro: " .. tostring(erro), "ERROR")
        return false
    end
    return true
end

local function AtacarInimigo()
    local sucesso, erro = pcall(function()
        local char = LocalPlayer.Character
        if not char then return false end
        
        -- Tentar vários métodos de ataque
        pcall(function() game:GetService("VirtualUser"):CaptureController() end)
        pcall(function() game:GetService("VirtualUser"):ClickButton1(Vector2.new(0, 0)) end)
        
        local tool = char:FindFirstChildOfClass("Tool")
        if tool and GameStructure.RemotesFolder then
            pcall(function()
                GameStructure.RemotesFolder:FireServer(tool.Name, tick())
            end)
        end
        
        return true
    end)
    
    if not sucesso then
        Log("Ataque", "Erro ao atacar: " .. tostring(erro), "WARN")
    end
    return sucesso
end

local function BuscarInimigos()
    local inimigos = {}
    
    local sucesso, erro = pcall(function()
        if GameStructure.MobsFolder then
            for _, inimigo in pairs(GameStructure.MobsFolder:GetChildren()) do
                if inimigo:FindFirstChild("HumanoidRootPart") and inimigo:FindFirstChild("Humanoid") then
                    if inimigo.Humanoid.Health > 0 then
                        table.insert(inimigos, inimigo)
                    end
                end
            end
        else
            -- Fallback: procurar em todo o Workspace
            for _, obj in pairs(Workspace:GetChildren()) do
                if obj:FindFirstChild("HumanoidRootPart") and obj:FindFirstChild("Humanoid") and obj ~= LocalPlayer.Character then
                    if obj.Humanoid.Health > 0 then
                        table.insert(inimigos, obj)
                    end
                end
            end
        end
    end)
    
    if not sucesso then
        Log("Busca de Inimigos", "Erro: " .. tostring(erro), "WARN")
    end
    
    return inimigos
end

local function TrazerMobsParaPerto(inimigos, posicao_alvo)
    if not _G.BringMobs or #inimigos == 0 then return end
    
    pcall(function()
        for _, inimigo in pairs(inimigos) do
            if inimigo:FindFirstChild("HumanoidRootPart") and inimigo.Humanoid.Health > 0 then
                inimigo.HumanoidRootPart.CFrame = posicao_alvo
                inimigo.HumanoidRootPart.CanCollide = false
                
                if inimigo.Humanoid:FindFirstChild("Animator") then
                    inimigo.Humanoid.Animator:Destroy()
                end
            end
        end
    end)
end

local function DistribuirStats()
    if not _G.AutoSpinFruit then return end
    
    pcall(function()
        if LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Points") then
            local pts = LocalPlayer.Data.Points.Value
            if pts > 0 and GameStructure.RemotesFolder then
                -- Tentar encontrar remote de stats
                local statsRemote = GameStructure.RemotesFolder:FindFirstChild("AddStat") or 
                                   GameStructure.RemotesFolder:FindFirstChild("UpgradeStat") or
                                   GameStructure.RemotesFolder:FindFirstChild("CommF_")
                
                if statsRemote then
                    pcall(function() statsRemote:InvokeServer("AddPoint", "Melee", 1) end)
                    task.wait(0.1)
                    pcall(function() statsRemote:InvokeServer("AddPoint", "Defense", 1) end)
                    task.wait(0.1)
                    pcall(function() statsRemote:InvokeServer("AddPoint", "Sword", 1) end)
                end
            end
        end
    end)
end

------------------------------------------------------------------------
-- 5. AUTO-FARM PRINCIPAL
------------------------------------------------------------------------
local function ExecutarAutoFarm()
    if not _G.AutoFarm then return end
    
    pcall(function()
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then return end
        
        local root = char.HumanoidRootPart
        local inimigos = BuscarInimigos()
        
        if #inimigos > 0 then
            for _, inimigo in pairs(inimigos) do
                if not inimigo or not inimigo:FindFirstChild("HumanoidRootPart") then continue end
                
                local distancia = (root.Position - inimigo.HumanoidRootPart.Position).Magnitude
                
                if distancia < _G.DistanciaDoInimigo then
                    TrazerMobsParaPerto({inimigo}, root.CFrame + root.CFrame.LookVector * 5)
                    AtacarInimigo()
                    task.wait(0.05)
                end
            end
        end
        
        -- Distribuir stats a cada 10 ciclos
        if math.random(1, 10) == 1 then
            DistribuirStats()
        end
        
        -- Atualizar estrutura
        pcall(function()
            GameStructure.CurrentLevel = LocalPlayer.Data.Level.Value
        end)
        
    end)
end

------------------------------------------------------------------------
-- 6. ANTI-BOUNTY
------------------------------------------------------------------------
local function VerificarAntiBounty()
    if not _G.AntiBounty then return end
    
    pcall(function()
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            local tag = char.Humanoid:FindFirstChild("creator")
            if tag and tag.Value and tag.Value:IsA("Player") then
                if char.Humanoid.Health < (char.Humanoid.MaxHealth * 0.5) then
                    Log("Anti-Bounty", "Bounty detectado! Trocando servidor...", "WARN")
                    _G.AutoFarm = false
                    
                    pcall(function()
                        local servidores = HttpService:JSONDecode(game:HttpGet("https://roblox.com/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"))
                        if servidores and servidores.data then
                            for _, s in pairs(servidores.data) do
                                if s.playing < s.maxPlayers and s.id ~= game.JobId then
                                    TeleportService:TeleportToPlaceInstance(game.PlaceId, s.id, LocalPlayer)
                                    break
                                end
                            end
                        end
                    end)
                end
            end
        end
    end)
end

------------------------------------------------------------------------
-- 7. LOOP PRINCIPAL ROBUSTO
------------------------------------------------------------------------
local function MainLoop()
    Log("Sistema", "Iniciando Banana Turbo Hub v18 - Delta Executor", "SUCCESS")
    
    DetectarEstruturaj()
    task.wait(1)
    
    local ciclo = 0
    
    while true do
        ciclo = ciclo + 1
        
        if LocalPlayer and LocalPlayer.Character then
            pcall(function()
                ExecutarAutoFarm()
                VerificarAntiBounty()
            end)
        end
        
        -- Log a cada 100 ciclos
        if ciclo % 100 == 0 then
            Log("Status", "Ciclo: " .. ciclo .. " | Nível: " .. GameStructure.CurrentLevel .. " | Mundo: " .. GameStructure.CurrentWorld, "INFO")
        end
        
        task.wait(0.3)
    end
end

------------------------------------------------------------------------
-- 8. LISTENER PARA CHARACTER RESPAWN
------------------------------------------------------------------------
LocalPlayer.CharacterAdded:Connect(function(newChar)
    Log("Sistema", "Personagem respawnado", "INFO")
    newChar:WaitForChild("HumanoidRootPart")
    Log("Sistema", "Pronto para continuar farmando!", "SUCCESS")
end)

------------------------------------------------------------------------
-- 9. INICIALIZAÇÃO
------------------------------------------------------------------------
Log("Sistema", "Banana Turbo Hub v18 - Delta Executor Edition", "SUCCESS")
Log("Sistema", "=== FUNCIONALIDADES ATIVAS ===", "INFO")
Log("Sistema", "AutoFarm: " .. tostring(_G.AutoFarm), "INFO")
Log("Sistema", "AutoGodhuman: " .. tostring(_G.AutoGodhuman), "INFO")
Log("Sistema", "AutoNewWorld: " .. tostring(_G.AutoNewWorld), "INFO")
Log("Sistema", "AniBounty: " .. tostring(_G.AntiBounty), "INFO")
Log("Sistema", "=============================", "INFO")

MainLoop()
