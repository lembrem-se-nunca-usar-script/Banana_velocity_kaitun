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

-- STATS SELECIONÁVEIS
_G.Stat_Melee = true
_G.Stat_Defense = true
_G.Stat_Fruit = true

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
            task.wait(0.01) -- Velocidade limite mecânica adaptativa
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
-- 5. MECÂNICA DE GODHUMAN INTELIGENTE E COMPRA DE V2 AUTOMÁTICA
------------------------------------------------------------------------
local MetasMaestria = {
    ["Dark Step"] = 500, ["Electric"] = 500, ["Water Kung Fu"] = 500, ["Dragon Breath"] = 500,
    ["Superhuman"] = 400, ["Death Step"] = 400, ["Sharkman Karate"] = 400, ["Electric Claw"] = 400, ["Dragon Talon"] = 400
}

local EvolucaoEstilosV2 = {
    ["Dark Step"] = {V2Nome = "Death Step", PrecoBeli = 2500000, PrecoFrag = 5000, NPCNome = "Phoenis", MarRequisito = 4442272163},
    ["Electric"] = {V2Nome = "Electric Claw", PrecoBeli = 3000000, PrecoFrag = 5000, NPCNome = "Previous Hero", MarRequisito = 11413812836},
    ["Water Kung Fu"] = {V2Nome = "Sharkman Karate", PrecoBeli = 2500000, PrecoFrag = 5000, NPCNome = "Daigrock the Sharkman", MarRequisito = 4442272163},
    ["Dragon Breath"] = {V2Nome = "Dragon Talon", PrecoBeli = 3000000, PrecoFrag = 5000, NPCNome = "Uzoth", MarRequisito = 11413812836}
}

local MateriaisGodhuman = {
    {Nome = "Fish Tail",       Meta = 20, InimigoFarm = "Fishman Raider",      NPCQuest = "Turtle Quest Giver", QuestNome = "FloatingTurtleQuest1", ID = 1},
    {Nome = "Magma Ore",       Meta = 20, InimigoFarm = "Lab Subordinate",     NPCQuest = "Hot and Cold Quest Giver", QuestNome = "FireQuest", ID = 1},
    {Nome = "Dragon Scale",    Meta = 10, InimigoFarm = "Dragon Crew Warrior", NPCQuest = "Hydra Quest Giver", QuestNome = "HydraIslandQuest", ID = 1},
    {Nome = "Mystic Droplet",  Meta = 10, InimigoFarm = "Water Captain",       NPCQuest = "Forgotten Quest Giver", QuestNome = "ForgottenQuest", ID = 2}
}

local function verificar_e_equipar_estilo_para_maestria()
    local bp = LocalPlayer:FindFirstChild("Backpack") or LocalPlayer:FindFirstChild("Data")
    for nome_estilo, meta in pairs(MetasMaestria) do
        local item = bp:FindFirstChild(nome_estilo)
        if item then
            item.Parent = LocalPlayer.Character
            task.wait(0.2)
        end
    end
end

local function farm_material_godhuman(material)
    if not _G.AutoGodhuman then return end
    local inimigo = material.InimigoFarm
    local posicaoFarm = Workspace:FindFirstChild(inimigo)
    
    if posicaoFarm and posicaoFarm:FindFirstChild("HumanoidRootPart") then
        local npcQuest = Workspace:FindFirstChild(material.NPCQuest)
        if npcQuest then
            teleportar_com_bypass(npcQuest.HumanoidRootPart.CFrame + Vector3.new(5, 0, 0))
            task.wait(0.5)
            
            local remote = ReplicatedStorage:FindFirstChild("Remotes")
            if remote then
                remote:FireServer("TalkToNPC", material.NPCQuest, material.QuestNome)
                task.wait(0.3)
                remote:FireServer("StartQuest", material.QuestNome)
            end
        end
    end
end

local function comprar_v2_automatico(estilo)
    if not _G.AutoGodhuman then return end
    local evolucao = EvolucaoEstilosV2[estilo]
    if not evolucao then return end
    
    local npc = Workspace:FindFirstChild(evolucao.NPCNome)
    if npc and npc:FindFirstChild("HumanoidRootPart") then
        teleportar_com_bypass(npc.HumanoidRootPart.CFrame + Vector3.new(5, 0, 0))
        task.wait(0.4)
        
        local remote = ReplicatedStorage:FindFirstChild("Remotes")
        if remote then
            remote:FireServer("BuyAbility", evolucao.V2Nome, evolucao.PrecoBeli)
            task.wait(0.3)
            EnviarNotificacaoDiscord("V2 Desbloqueado", "Você desbloqueou: " .. evolucao.V2Nome, 65280)
        end
    end
end

------------------------------------------------------------------------
-- 6. SISTEMA DE AUTO-FARM INTEGRADO
------------------------------------------------------------------------
local function executar_auto_farm()
    if not _G.AutoFarm then return end
    
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local root = char:WaitForChild("HumanoidRootPart")
    
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
end

------------------------------------------------------------------------
-- 7. LOOP PRINCIPAL
------------------------------------------------------------------------
local function main_loop()
    CarregarConfiguracoes()
    
    while true do
        if LocalPlayer and LocalPlayer.Character then
            if _G.AutoFarm then executar_auto_farm() end
            if _G.AutoSpinFruit then executar_auto_stats() end
            if _G.AutoGodhuman then
                for _, material in pairs(MateriaisGodhuman) do
                    farm_material_godhuman(material)
                    task.wait(0.5)
                end
            end
            if _G.AntiBounty then checar_anti_bounty() end
        end
        
        SalvarConfiguracoes()
        task.wait(1)
    end
end

------------------------------------------------------------------------
-- 8. INICIALIZAÇÃO
------------------------------------------------------------------------
EnviarNotificacaoDiscord("Script Iniciado", "Banana Turbo Hub v18 foi ativado com sucesso!", 3066993)
main_loop()
