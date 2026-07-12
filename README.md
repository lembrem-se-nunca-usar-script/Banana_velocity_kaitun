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
    ["Url"] = "AQUI_VAI_O_TEU_LINK_DO_WEBHOOK_DO_DISCORD"
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

local function verificar_e_equipar_estilo_para_maestria()local bp = LocalPlayer:FindFirstChild("Backpack") or LocalPlayer:FindFirstChild("Data")for nome_estilo, meta in pairs(MetasMaestria) dolocal item = bp:FindFirstChild(nome_estilo)if item thenlocal lvl = item:FindFirstChild("Level") and item.Level.Value or 0if lvl < meta then return nome_estilo endendendreturn nilendlocal function obter_material_em_falta()local material_folder = LocalPlayer.Data:FindFirstChild("Materials")if not material_folder then return nil endfor _, mat in pairs(MateriaisGodhuman) dolocal qtd_atual = material_folder:FindFirstChild(mat.Nome) and material_folder[mat.Nome].Value or 0if qtd_atual < mat.Meta then return mat endendreturn nilendlocal function verificar_e_comprar_v2_imediato()if not G.AutoGodhuman then return false endlocal bp = LocalPlayer:FindFirstChild("Backpack") or LocalPlayer:FindFirstChild("Data")for v1_nome, d in pairs(EvolucaoEstilosV2) dolocal item_v1 = bp:FindFirstChild(v1_nome)local ja_tem_v2 = bp:FindFirstChild(d.V2Nome)if item_v1 and item_v1.Level.Value >= 500 and not ja_tem_v2 thenif game.PlaceId == d.MarRequisito and LocalPlayer.Data.Beli.Value >= d.PrecoBeli and LocalPlayer.Data.Fragments.Value >= d.PrecoFrag thenlocal npc = Workspace.NPCs:FindFirstChild(d.NPCNome) or Workspace:FindFirstChild(d.NPCNome)if npc thenteleportar_com_bypass(npc.HumanoidRootPart.CFrame)task.wait(0.4)local remote = ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("CommF")if remote thenremote:InvokeServer("Buy" .. d.V2Nome:gsub(" ", ""))EnviarNotificacaoDiscord("Estilo V2 Adquirido!", "Comprado automaticamente: " .. d.V2Nome, 4620411)return trueendendendendendreturn falseendToggleFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 30) ; ToggleFrame.Parent = MainFramelocal TextLabel = Instance.new("TextLabel") ; TextLabel.Size = UDim2.new(0.6, 0, 1, 0) ; TextLabel.Position = UDim2.new(0, 15, 0, 0) ; TextLabel.BackgroundTransparency = 1 ; TextLabel.Text = textoBotao ; TextLabel.TextColor3 = Color3.fromRGB(255, 255, 255) ; TextLabel.TextSize = 10 ; TextLabel.Font = Enum.Font.GothamSemibold ; TextLabel.TextXAlignment = Enum.TextXAlignment.Left ; TextLabel.Parent = ToggleFramelocal Button = Instance.new("TextButton") ; Button.Size = UDim2.new(0.25, 0, 0.6, 0) ; Button.Position = UDim2.new(0.7, 0, 0.2, 0) ; Button.BackgroundColor3 = _G[variavelGlobal] and Color3.fromRGB(46, 204, 113) or Color3.fromRGB(231, 76, 60) ; Button.Text = _G[variavelGlobal] and "ON" or "OFF" ; Button.TextColor3 = Color3.fromRGB(255, 255, 255) ; Button.Font = Enum.Font.GothamBold ; Button.Parent = ToggleFrameButton.MouseButton1Click:Connect(function()_G[variavelGlobal] = not _G[variavelGlobal]Button.BackgroundColor3 = _G[variavelGlobal] and Color3.fromRGB(46, 204, 113) or Color3.fromRGB(231, 76, 60)Button.Text = _G[variavelGlobal] and "ON" or "OFF"pcall(SalvarConfiguracoes)end)endcriarBotaoToggle("AutoFarm", UDim2.new(0, 15, 0, 50), "AutoFarm", "Auto Farm Inteligente (Mares 1-3)")criarBotaoToggle("AutoGodhuman", UDim2.new(0, 15, 0, 90), "AutoGodhuman", "Auto Godhuman Completo (Alternado)")criarBotaoToggle("AutoNewWorld", UDim2.new(0, 15, 0, 130), "AutoNewWorld", "Auto New World (Mar 2)")criarBotaoToggle("AutoSpinFruit", UDim2.new(0, 15, 0, 170), "AutoSpinFruit", "Auto Gacha & Auto Store")criarBotaoToggle("StatMelee", UDim2.new(0, 15, 0, 220), "Stat_Melee", "[STATS] Auto Soco (Melee)")criarBotaoToggle("StatDefense", UDim2.new(0, 15, 0, 260), "Stat_Defense", "[STATS] Auto Vida (Defense)")criarBotaoToggle("StatSword", UDim2.new(0, 15, 0, 300), "Stat_Sword", "[STATS] Auto Espada (Sword)")criarBotaoToggle("BringMobs", UDim2.new(0, 15, 0, 350), "BringMobs", "Turbo Fast Bring Mobs")criarBotaoToggle("AntiBounty", UDim2.new(0, 15, 0, 390), "AntiBounty", "Anti-Bounty Hunter Protection")criarBotaoToggle("SafeFlyBypass", UDim2.new(0, 15, 0, 430), "SafeFlyBypass", "Safe Fly Velocity Bypass MAX")GetKeyButton.MouseButton1Click:Connect(function() if setclipboard then setclipboard(LinkLootLabs) end end)local KeyValidada = falseCheckButton.MouseButton1Click:Connect(function() if KeyInput.Text == ChaveCorreta then KeyValidada = true ; KeyFrame:Destroy() ; MainFrame.Visible = true ; pcall(CarregarConfiguracoes) EnviarNotificacaoDiscord("Hub Autenticado!", "Licença validada com sucesso.", 3066993) end end)UserInputService.InputBegan:Connect(function(ipt, proc)if not proc and ipt.KeyCode == Enum.KeyCode.RightControl and KeyValidada then MainFrame.Visible = not MainFrame.Visible endend)
Workspace:FindFirstChild("Military Detective")if det thenteleportar_com_bypass(det.HumanoidRootPart.CFrame)task.wait(0.4)remote:InvokeServer("TravelIceIsland")endlocal admiral = Workspace.Enemies:FindFirstChild("Ice Admiral") or Workspace:FindFirstChild("Ice Admiral")if admiral and admiral:FindFirstChild("Humanoid") and admiral.Humanoid.Health > 0 thenteleportar_com_bypass(admiral.HumanoidRootPart.CFrame * CFrame.new(0, _G.DistanciaDoInimigo, 0))auto_atacar_turbo()returnendif game.PlaceId == 2747839626 thenlocal cap = Workspace.NPCs:FindFirstChild("Experienced Captain") or Workspace:FindFirstChild("Experienced Captain")if cap thenteleportar_com_bypass(cap.HumanoidRootPart.CFrame)task.wait(0.4)EnviarNotificacaoDiscord("A transitar de Mar!", "A viajar para o Second Sea.", 2306448)remote:InvokeServer("TravelToSecondSea")endendendlocal function rolar_e_guardar_gacha()local remote = ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("CommF_")if remote thenremote:InvokeServer("Cousin", "Buy")task.wait(0.4)for _, item in pairs(LocalPlayer.Backpack:GetChildren()) doif string.find(item.Name, "Fruit") thenremote:InvokeServer("StoreFruit", item.Name, item)EnviarNotificacaoDiscord("Gacha Spin!", "Fruta ganha: " .. item.Name, 15105570)endendendend-- 8. MOTOR DE PRIORIDADES GERAL (LAÇO PRINCIPAL MÁXIMA FREQUÊNCIA)spawn(function()local tempo_ultimo_spin = 0while true dotask.wait(0.01)if KeyValidada thenif _G.AntiBounty then pcall(checar_anti_bounty) endpcall(executar_auto_stats)if _G.AutoFarm thenlocal comprando_v2 = pcall(verificar_e_comprar_v2_imediato)if not comprando_v2 thenlocal estilo_pendente = _G.AutoGodhuman and verificar_e_equipar_estilo_para_maestria() or nillocal material_em_falta = _G.AutoGodhuman and obter_material_em_falta() or nilif estilo_pendente thenlocal q = obter_quest_atual()executar_ciclo_de_ataque(q.NPC, q.Quest, q.ID, q.Inimigo, estilo_pendente)elseif material_em_falta and game.PlaceId == 11413812836 thenexecutar_ciclo_de_ataque(material_em_falta.NPCQuest, material_em_falta.QuestNome, material_em_falta.ID, material_em_falta.InimigoFarm, "Combat")elselocal q = obter_quest_atual()executar_ciclo_de_ataque(q.NPC, q.Quest, q.ID, q.Inimigo, "Combat")endendelseif _G.AutoNewWorld and LocalPlayer.Data.Level.Value >= 700 then pcall(fazer_quest_transicao_sea2)elseif _G.AutoSpinFruit and (tick() - tempo_ultimo_spin > 20) thenpcall(rolar_e_guardar_gacha)tempo_ultimo_spin = tick()endendendend)
