kaitun banana velocity

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")

------------------------------------------------------------------------
-- 1. CONFIGURAÇÕES PRINCIPAIS (EDITA ESTA PARTE ANTES DE USAR)
------------------------------------------------------------------------
local ChaveCorreta = "BANANA_TURBO_2026" 
local LinkLootLabs = "https://lootlabs.gg" 

_G.Webhook_Config = {
    ["Ativado"] = true,
    ["Url"] = "https://discord.com/api/webhooks/1525704012610015292/yVBRyuDsX1XvOhD481SQnU6mtIMWiKhgbUn-DsTABZF2RHmWpRkuTg_SWKHkLzbxFeYq"
}

------------------------------------------------------------------------
-- ESTADO INICIAL DAS VARIÁVEIS GLOBAIS
------------------------------------------------------------------------
_G.AutoFarm = true
_G.AutoGodhuman = true
_G.AutoNewWorld = true      
_G.AutoSpinFruit = true     
_G.BringMobs = true        
_G.FPSBoost = true          
_G.AntiBounty = true         
_G.SafeFlyBypass = true      
_G.DistanciaDoInimigo = 40
_G.FarmAteNivelMax = true

-- STATS SELECIONÁVEIS
_G.Stat_Melee = true
_G.Stat_Defense = true
_G.Stat_Fruit = true

-- NÍVEIS MÁXIMOS POR MUNDO
local NiveisMaximos = {
    ["East Blue"] = 10,
    ["Pirate Village"] = 20,
    ["Shells Town"] = 30,
    ["Baratie"] = 50,
    ["Arlong Park"] = 70,
    ["Loguetown"] = 100,
    ["Reverse Mountain"] = 120,
    ["Whiskey Peak"] = 150,
    ["Grand Line"] = 200,
    ["Ice Island"] = 250,
    ["Marineford"] = 300
}

local NomeFicheiroConfig = "BananaKaitun_Turbo_Config.txt"

------------------------------------------------------------------------
-- 2. SISTEMA DE CONFIGURAÇÃO LOCAL (SAVE / LOAD)
------------------------------------------------------------------------
local function SalvarConfiguracoes()
    local config = {
        AutoFarm = _G.AutoFarm, AutoGodhuman = _G.AutoGodhuman, AutoNewWorld = _G.AutoNewWorld,
        AutoSpinFruit = _G.AutoSpinFruit, BringMobs = _G.BringMobs, FPSBoost = _G.FPSBoost, 
        AntiBounty = _G.AntiBounty, SafeFlyBypass = _G.SafeFlyBypass, Stat_Melee = _G.Stat_Melee,
        Stat_Defense = _G.Stat_Defense, Stat_Sword = _G.Stat_Sword
    }
    local sucesso, textoJson = pcall(function() return HttpService:JSONEncode(config) end)
    if sucesso and writefile then writefile(NomeFicheiroConfig, textoJson) end
end

local function CarregarConfiguracoes()
    if readfile and isfile and isfile(NomeFicheiroConfig) then
        local sucesso, textoJson = pcall(function() return readfile(NomeFicheiroConfig) end)
        if sucesso then
            local de, configCarregada = pcall(function() return HttpService:JSONDecode(textoJson) end)
            if de and configCarregada then
                _G.AutoFarm = configCarregada.AutoFarm ?? _G.AutoFarm
                _G.AutoGodhuman = configCarregada.AutoGodhuman ?? _G.AutoGodhuman
                _G.AutoNewWorld = configCarregada.AutoNewWorld ?? _G.AutoNewWorld
                _G.AutoSpinFruit = configCarregada.AutoSpinFruit ?? _G.AutoSpinFruit
                _G.BringMobs = configCarregada.BringMobs ?? _G.BringMobs
                _G.FPSBoost = configCarregada.FPSBoost ?? _G.FPSBoost
                _G.AntiBounty = configCarregada.AntiBounty ?? _G.AntiBounty
                _G.SafeFlyBypass = configCarregada.SafeFlyBypass ?? _G.SafeFlyBypass
                _G.Stat_Melee = configCarregada.Stat_Melee ?? _G.Stat_Melee
                _G.Stat_Defense = configCarregada.Stat_Defense ?? _G.Stat_Defense
                _G.Stat_Sword = configCarregada.Stat_Sword ?? _G.Stat_Sword
            end
        end
    end
end

------------------------------------------------------------------------
-- 3. NOTIFICADOR WEBHOOK DISCORD
------------------------------------------------------------------------
local function EnviarNotificacaoDiscord(titulo, mensagem, corHex)
    if not _G.Webhook_Config["Ativado"] or _G.Webhook_Config["Url"] == "" or _G.Webhook_Config["Url"] == "AQUI_VAI_O_TEU_LINK_DO_WEBHOOK_DO_DISCORD" then return end
    local request = syn and syn.request or http_request or http and http.request or request
    if request then
        local dadosEmbed = {
            ["title"] = titulo, ["description"] = mensagem, ["color"] = corHex or 24119615,
            ["fields"] = {
                {["name"] = "Utilizador:", ["value"] = LocalPlayer.Name, ["inline"] = true},
                {["name"] = "Nível:", ["value"] = tostring(LocalPlayer.Data.Level.Value), ["inline"] = true},
                {["name"] = "Beli:", ["value"] = "$" .. tostring(LocalPlayer.Data.Beli.Value), ["inline"] = true}
            },
            ["footer"] = {["text"] = "Banana Turbo Hub v18"}
        }
        pcall(function()
            request({Url = _G.Webhook_Config["Url"], Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = HttpService:JSONEncode({embeds = {dadosEmbed}})})
        end)
    end
end

------------------------------------------------------------------------
-- 4. TURBO ENGINE ENGINE, MOVIMENTAÇÃO & BYPASS MAX SPEED
------------------------------------------------------------------------
local function teleportar_com_bypass(cframe_destino)
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
            task.wait(0.01)
            bv:Destroy()
        else
            root.CFrame = cframe_destino
        end
    end
end

local function auto_atacar_turbo()
    game:GetService("VirtualUser"):CaptureController()
    game:GetService("VirtualUser"):ClickButton1(Vector2.new(0, 0))
    pcall(function()
        local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
        if tool then ReplicatedStorage.Remotes.Validator:FireServer(tool.Name, tick()) end
    end)
end

local function executar_bring_mobs(nome_inimigo, posicao_alvo)
    if not _G.BringMobs then return end
    for _, v in pairs(Workspace.Enemies:GetChildren()) do
        if v.Name == nome_inimigo and v:FindFirstChild("HumanoidRootPart") and v.Humanoid.Health > 0 then
            v.HumanoidRootPart.CFrame = posicao_alvo
            v.HumanoidRootPart.CanCollide = false 
            if v.Humanoid:FindFirstChild("Animator") then v.Humanoid.Animator:Destroy() end 
        end
    end
end

local function executar_auto_stats()
    local pts = LocalPlayer.Data.Points.Value
    if pts > 0 then
        local rm = ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("CommF_")
        if rm then
            if _G.Stat_Melee then rm:InvokeServer("AddPoint", "Melee", 1) end
            if _G.Stat_Defense then rm:InvokeServer("AddPoint", "Defense", 1) end
            if _G.Stat_Sword then rm:InvokeServer("AddPoint", "Sword", 1) end
        end
    end
end

local function executar_server_hop()
    local servidores = HttpService:JSONDecode(game:HttpGet("https://roblox.com" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"))
    for _, s in pairs(servidores.data) do
        if s.playing < s.maxPlayers and s.id ~= game.JobId then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, s.id, LocalPlayer)
            break
        end
    end
end

local function checar_anti_bounty()
    if not _G.AntiBounty then return end
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        local tag = char.Humanoid:FindFirstChild("creator")
        if tag and tag.Value and tag.Value:IsA("Player") then
            if char.Humanoid.Health < (char.Humanoid.MaxHealth * 0.5) then
                _G.AutoFarm = false
                executar_server_hop()
            end
        end
    end
end

------------------------------------------------------------------------
-- 5. BANCO DE DADOS GODHUMAN COMPLETO (ESTILOS + QUESTS + ITENS + FRAGMENTOS)
------------------------------------------------------------------------
local EstilosGodhuman = {
    -- DARK STEP
    {
        Nome = "Dark Step",
        NPC = "Nico Robin",
        Preco = 750000,
        FragmentosNecess = 0,
        Quest = "Dark Step Quest",
        ItemsNecess = {},
        FarmLocalInimigo = "Bandit",
        FarmLocalNPC = "Dojo Instructor",
        Maestria = 500,
        V2Nome = "Death Step",
        V2Preco = 2500000,
        V2Fragmentos = 5000,
        V2NPC = "Phoenis",
        V2Mar = 4442272163
    },
    -- ELECTRIC
    {
        Nome = "Electric",
        NPC = "Enel",
        Preco = 900000,
        FragmentosNecess = 0,
        Quest = "Electric Quest",
        ItemsNecess = {},
        FarmLocalInimigo = "Electric Pirate",
        FarmLocalNPC = "Electric Master",
        Maestria = 500,
        V2Nome = "Electric Claw",
        V2Preco = 3000000,
        V2Fragmentos = 5000,
        V2NPC = "Previous Hero",
        V2Mar = 11413812836
    },
    -- WATER KUNG FU
    {
        Nome = "Water Kung Fu",
        NPC = "Fishman Instructor",
        Preco = 800000,
        FragmentosNecess = 0,
        Quest = "Water Kung Fu Quest",
        ItemsNecess = {"Fish Tail"},
        FarmLocalInimigo = "Fishman Raider",
        FarmLocalNPC = "Turtle Quest Giver",
        Maestria = 500,
        V2Nome = "Sharkman Karate",
        V2Preco = 2500000,
        V2Fragmentos = 5000,
        V2NPC = "Daigrock the Sharkman",
        V2Mar = 4442272163
    },
    -- DRAGON BREATH
    {
        Nome = "Dragon Breath",
        NPC = "Dragon Master",
        Preco = 1000000,
        FragmentosNecess = 0,
        Quest = "Dragon Breath Quest",
        ItemsNecess = {"Dragon Scale"},
        FarmLocalInimigo = "Dragon Crew Warrior",
        FarmLocalNPC = "Hydra Quest Giver",
        Maestria = 500,
        V2Nome = "Dragon Talon",
        V2Preco = 3000000,
        V2Fragmentos = 5000,
        V2NPC = "Uzoth",
        V2Mar = 11413812836
    },
    -- SUPERHUMAN
    {
        Nome = "Superhuman",
        NPC = "Superhuman Master",
        Preco = 1200000,
        FragmentosNecess = 200,
        Quest = "Superhuman Quest",
        ItemsNecess = {"Magma Ore"},
        FarmLocalInimigo = "Lab Subordinate",
        FarmLocalNPC = "Hot and Cold Quest Giver",
        Maestria = 400,
        V2Nome = nil,
        V2Preco = 0,
        V2Fragmentos = 0,
        V2NPC = nil,
        V2Mar = 0
    },
    -- DEATH STEP (V2 Dark Step)
    {
        Nome = "Death Step",
        NPC = "Phoenis",
        Preco = 2500000,
        FragmentosNecess = 5000,
        Quest = "Death Step Quest",
        ItemsNecess = {},
        FarmLocalInimigo = "Advanced Enemy",
        FarmLocalNPC = "Martial Arts Master",
        Maestria = 400
    },
    -- SHARKMAN KARATE (V2 Water Kung Fu)
    {
        Nome = "Sharkman Karate",
        NPC = "Daigrock the Sharkman",
        Preco = 2500000,
        FragmentosNecess = 5000,
        Quest = "Sharkman Quest",
        ItemsNecess = {"Mystic Droplet"},
        FarmLocalInimigo = "Water Captain",
        FarmLocalNPC = "Forgotten Quest Giver",
        Maestria = 400
    },
    -- ELECTRIC CLAW (V2 Electric)
    {
        Nome = "Electric Claw",
        NPC = "Previous Hero",
        Preco = 3000000,
        FragmentosNecess = 5000,
        Quest = "Electric Claw Quest",
        ItemsNecess = {},
        FarmLocalInimigo = "Enemy",
        FarmLocalNPC = "Master",
        Maestria = 400
    },
    -- DRAGON TALON (V2 Dragon Breath)
    {
        Nome = "Dragon Talon",
        NPC = "Uzoth",
        Preco = 3000000,
        FragmentosNecess = 5000,
        Quest = "Dragon Talon Quest",
        ItemsNecess = {},
        FarmLocalInimigo = "Dragon Enemy",
        FarmLocalNPC = "Dragon Master",
        Maestria = 400
    }
}

------------------------------------------------------------------------
-- 6. SISTEMA COMPLETO DE FARM GODHUMAN
------------------------------------------------------------------------
local function ganhar_dinheiro_para_estilo(estilo)
    if not _G.AutoGodhuman then return end
    
    local beliNecessario = estilo.Preco
    local beliAtual = LocalPlayer.Data.Beli.Value
    
    if beliAtual < beliNecessario then
        local diferencaBeli = beliNecessario - beliAtual
        EnviarNotificacaoDiscord("Farmando Beli", "Necessário: $" .. diferencaBeli .. " para " .. estilo.Nome, 16776960)
        
        -- Farm de inimigos normais para ganhar dinheiro
        local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local root = char:WaitForChild("HumanoidRootPart")
        
        while LocalPlayer.Data.Beli.Value < beliNecessario do
            for _, inimigo in pairs(Workspace.Enemies:GetChildren()) do
                if inimigo:FindFirstChild("HumanoidRootPart") and inimigo.Humanoid.Health > 0 then
                    local distancia = (root.Position - inimigo.HumanoidRootPart.Position).Magnitude
                    if distancia < _G.DistanciaDoInimigo then
                        executar_bring_mobs(inimigo.Name, root.CFrame + root.CFrame.LookVector * 5)
                        auto_atacar_turbo()
                        task.wait(0.1)
                    end
                end
            end
            task.wait(0.5)
        end
    end
end

local function ganhar_itens_para_estilo(estilo)
    if not _G.AutoGodhuman or #estilo.ItemsNecess == 0 then return end
    
    EnviarNotificacaoDiscord("Farmando Itens", "Itens necessários para " .. estilo.Nome .. ": " .. table.concat(estilo.ItemsNecess, ", "), 16776960)
    
    local npcFarm = Workspace:FindFirstChild(estilo.FarmLocalNPC)
    if npcFarm and npcFarm:FindFirstChild("HumanoidRootPart") then
        teleportar_com_bypass(npcFarm.HumanoidRootPart.CFrame + Vector3.new(5, 0, 0))
        task.wait(0.5)
        
        local remote = ReplicatedStorage:FindFirstChild("Remotes")
        if remote then
            remote:FireServer("TalkToNPC", estilo.FarmLocalNPC, estilo.Quest)
            task.wait(0.3)
            remote:FireServer("StartQuest", estilo.Quest)
            task.wait(0.3)
            
            -- Farm enquanto completa quest
            local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
            local root = char:WaitForChild("HumanoidRootPart")
            
            while true do
                local temTodosItens = true
                for _, item in pairs(estilo.ItemsNecess) do
                    if not LocalPlayer.Backpack:FindFirstChild(item) and not LocalPlayer.Character:FindFirstChild(item) then
                        temTodosItens = false
                        break
                    end
                end
                
                if temTodosItens then break end
                
                for _, inimigo in pairs(Workspace.Enemies:GetChildren()) do
                    if inimigo:FindFirstChild("HumanoidRootPart") and inimigo.Humanoid.Health > 0 then
                        local distancia = (root.Position - inimigo.HumanoidRootPart.Position).Magnitude
                        if distancia < _G.DistanciaDoInimigo then
                            executar_bring_mobs(inimigo.Name, root.CFrame + root.CFrame.LookVector * 5)
                            auto_atacar_turbo()
                            task.wait(0.1)
                        end
                    end
                end
                task.wait(0.5)
            end
            
            remote:FireServer("CompleteQuest", estilo.Quest)
        end
    end
end

local function ganhar_fragmentos_para_estilo(estilo)
    if not _G.AutoGodhuman or estilo.FragmentosNecess == 0 then return end
    
    local fragmentosNecessarios = estilo.FragmentosNecess
    local fragmentosAtual = LocalPlayer.Data.Fragments.Value or 0
    
    if fragmentosAtual < fragmentosNecessarios then
        local diferencaFragmentos = fragmentosNecessarios - fragmentosAtual
        EnviarNotificacaoDiscord("Farmando Fragmentos", "Necessários: " .. diferencaFragmentos .. " para " .. estilo.Nome, 16776960)
        
        local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        local root = char:WaitForChild("HumanoidRootPart")
        
        while (LocalPlayer.Data.Fragments.Value or 0) < fragmentosNecessarios do
            for _, inimigo in pairs(Workspace.Enemies:GetChildren()) do
                if inimigo:FindFirstChild("HumanoidRootPart") and inimigo.Humanoid.Health > 0 then
                    local distancia = (root.Position - inimigo.HumanoidRootPart.Position).Magnitude
                    if distancia < _G.DistanciaDoInimigo then
                        executar_bring_mobs(inimigo.Name, root.CFrame + root.CFrame.LookVector * 5)
                        auto_atacar_turbo()
                        task.wait(0.1)
                    end
                end
            end
            task.wait(0.5)
        end
    end
end

local function comprar_estilo(estilo)
    if not _G.AutoGodhuman then return end
    
    local npc = Workspace:FindFirstChild(estilo.NPC)
    if npc and npc:FindFirstChild("HumanoidRootPart") then
        teleportar_com_bypass(npc.HumanoidRootPart.CFrame + Vector3.new(5, 0, 0))
        task.wait(0.5)
        
        local remote = ReplicatedStorage:FindFirstChild("Remotes")
        if remote then
            remote:FireServer("BuyAbility", estilo.Nome, estilo.Preco)
            task.wait(0.3)
            EnviarNotificacaoDiscord("✅ Estilo Desbloqueado", "Você desbloqueou: " .. estilo.Nome, 65280)
        end
    end
end

local function farm_completo_estilo(estilo)
    if not _G.AutoGodhuman then return end
    
    EnviarNotificacaoDiscord("🎯 Farm GodhHuman Iniciado", "Farmando: " .. estilo.Nome, 3066993)
    
    -- 1. Ganhar dinheiro
    ganhar_dinheiro_para_estilo(estilo)
    task.wait(1)
    
    -- 2. Ganhar itens
    ganhar_itens_para_estilo(estilo)
    task.wait(1)
    
    -- 3. Ganhar fragmentos
    ganhar_fragmentos_para_estilo(estilo)
    task.wait(1)
    
    -- 4. Comprar estilo
    comprar_estilo(estilo)
    task.wait(1)
    
    -- 5. Começar a farmar com o novo estilo
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local root = char:WaitForChild("HumanoidRootPart")
    
    local item = LocalPlayer.Backpack:FindFirstChild(estilo.Nome)
    if item then
        item.Parent = LocalPlayer.Character
        task.wait(0.3)
    end
    
    EnviarNotificacaoDiscord("🚀 Farmando com Estilo", "Farmando com: " .. estilo.Nome, 65280)
end

------------------------------------------------------------------------
-- 7. SISTEMA DE MUDANÇA DE MUNDO
------------------------------------------------------------------------
local function obter_mundo_atual()
    return LocalPlayer.Data.WorldName.Value or "East Blue"
end

local function fazer_quest_mudanca_mundo()
    if not _G.AutoNewWorld then return end
    
    local remote = ReplicatedStorage:FindFirstChild("Remotes")
    if remote then
        remote:FireServer("StartWorldQuest")
        task.wait(1)
        remote:FireServer("CompleteWorldQuest")
        task.wait(1)
        EnviarNotificacaoDiscord("Mundo Trocado", "Progredindo para o próximo mundo!", 65280)
    end
end

local function entrar_marinha()
    if not _G.AutoNewWorld then return end
    
    local marineSpawn = Workspace:FindFirstChild("MarineSpawn")
    if marineSpawn and marineSpawn:FindFirstChild("HumanoidRootPart") then
        teleportar_com_bypass(marineSpawn.HumanoidRootPart.CFrame)
        task.wait(0.5)
        
        local remote = ReplicatedStorage:FindFirstChild("Remotes")
        if remote then
            remote:FireServer("JoinMarine")
            task.wait(1)
            EnviarNotificacaoDiscord("Marinha", "Você entrou na Marinha!", 3066993)
        end
    end
end

------------------------------------------------------------------------
-- 8. SISTEMA DE AUTO-FARM INTEGRADO COM PROGRESSÃO
------------------------------------------------------------------------
local function executar_auto_farm()
    if not _G.AutoFarm then return end
    
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local root = char:WaitForChild("HumanoidRootPart")
    local levelAtual = LocalPlayer.Data.Level.Value
    local mundoAtual = obter_mundo_atual()
    
    -- Distribuir stats enquanto faz farm
    if _G.AutoSpinFruit and LocalPlayer.Data.Points.Value > 0 then
        executar_auto_stats()
    end
    
    -- Farm de GodhHuman completo
    if _G.AutoGodhuman and levelAtual > 50 then
        for _, estilo in pairs(EstilosGodhuman) do
            if estilo.Nome and not LocalPlayer.Backpack:FindFirstChild(estilo.Nome) and not LocalPlayer.Character:FindFirstChild(estilo.Nome) then
                farm_completo_estilo(estilo)
                task.wait(2)
                break -- Farm um estilo por vez
            end
        end
    end
    
    -- Loop de combate
    for _, inimigo in pairs(Workspace.Enemies:GetChildren()) do
        if inimigo:FindFirstChild("HumanoidRootPart") and inimigo.Humanoid.Health > 0 then
            local distancia = (root.Position - inimigo.HumanoidRootPart.Position).Magnitude
            if distancia < _G.DistanciaDoInimigo then
                executar_bring_mobs(inimigo.Name, root.CFrame + root.CFrame.LookVector * 5)
                auto_atacar_turbo()
                task.wait(0.1)
            end
        end
    end
    
    -- Verificar mudança de mundo
    if _G.AutoNewWorld and _G.FarmAteNivelMax then
        local nivelMax = NiveisMaximos[mundoAtual] or 300
        if levelAtual >= nivelMax then
            EnviarNotificacaoDiscord("🚀 Nível Máximo Atingido", "Nível: " .. levelAtual .. " | Mundo: " .. mundoAtual, 16776960)
            fazer_quest_mudanca_mundo()
            task.wait(2)
            entrar_marinha()
            task.wait(3)
        end
    end
end

------------------------------------------------------------------------
-- 9. LOOP PRINCIPAL
------------------------------------------------------------------------
local function main_loop()
    CarregarConfiguracoes()
    EnviarNotificacaoDiscord("Script Iniciado", "Banana Turbo Hub v18 foi ativado com sucesso!", 3066993)
    
    while true do
        if LocalPlayer and LocalPlayer.Character then
            pcall(function()
                executar_auto_farm()
                checar_anti_bounty()
            end)
        end
        
        SalvarConfiguracoes()
        task.wait(0.5)
    end
end

------------------------------------------------------------------------
-- 10. INICIALIZAÇÃO
------------------------------------------------------------------------
main_loop()
