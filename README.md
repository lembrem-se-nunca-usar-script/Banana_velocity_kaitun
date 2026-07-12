-- Banana Turbo Hub v18 - DELTA EXECUTOR
-- Script pronto para usar

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local RunService = game:GetService("RunService")

-- Aguardar o personagem estar pronto
if not LocalPlayer.Character then
    LocalPlayer.CharacterAdded:Wait()
end

local Character = LocalPlayer.Character
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

print("=" .. string.rep("=", 50))
print("🍌 BANANA TURBO HUB v18 INICIADO")
print("=" .. string.rep("=", 50))

-- CONFIGURAÇÕES
local Config = {
    AutoFarm = true,
    AutoAttack = true,
    BringMobs = true,
    AutoStats = true,
    Distance = 50,
    Delay = 0.1
}

-- VARIÁVEIS GLOBAIS
local FarmAtivo = true
local UltimoAtaque = tick()

print("[✓] Configurações carregadas")

-- FUNÇÃO: Teleportar
local function Teleport(CFrame)
    if not CFrame then return end
    pcall(function()
        local Char = LocalPlayer.Character
        if Char and Char:FindFirstChild("HumanoidRootPart") then
            Char.HumanoidRootPart.CFrame = CFrame
        end
    end)
end

-- FUNÇÃO: Trazer mobs
local function BringMobs()
    if not Config.BringMobs then return end
    
    pcall(function()
        local Char = LocalPlayer.Character
        if not Char or not Char:FindFirstChild("HumanoidRootPart") then return end
        
        local Root = Char.HumanoidRootPart
        
        -- Procurar inimigos
        for _, Obj in pairs(Workspace:GetDescendants()) do
            if Obj:IsA("Model") and Obj:FindFirstChild("Humanoid") and Obj:FindFirstChild("HumanoidRootPart") then
                if Obj.Humanoid.Health > 0 and Obj ~= Char then
                    local Distance = (Root.Position - Obj.HumanoidRootPart.Position).Magnitude
                    if Distance < Config.Distance then
                        -- Trazer mob
                        Obj.HumanoidRootPart.CFrame = Root.CFrame + Root.CFrame.LookVector * 3
                        Obj.HumanoidRootPart.CanCollide = false
                        
                        if Obj.Humanoid:FindFirstChild("Animator") then
                            Obj.Humanoid.Animator:Destroy()
                        end
                    end
                end
            end
        end
    end)
end

-- FUNÇÃO: Atacar
local function Attack()
    if not Config.AutoAttack then return end
    if tick() - UltimoAtaque < Config.Delay then return end
    
    pcall(function()
        local Char = LocalPlayer.Character
        if not Char then return end
        
        -- Método 1: VirtualUser
        pcall(function()
            game:GetService("VirtualUser"):CaptureController()
            game:GetService("VirtualUser"):ClickButton1(Vector2.new(0, 0))
        end)
        
        -- Método 2: FireServer
        pcall(function()
            local Tool = Char:FindFirstChildOfClass("Tool")
            if Tool then
                local Remote = ReplicatedStorage:FindFirstChild("Remotes")
                if Remote then
                    Remote:FireServer(Tool.Name, tick())
                end
            end
        end)
        
        UltimoAtaque = tick()
    end)
end

-- FUNÇÃO: Distribuir Stats
local function DistribuirStats()
    if not Config.AutoStats then return end
    
    pcall(function()
        local Data = LocalPlayer:FindFirstChild("Data")
        if Data and Data:FindFirstChild("Points") then
            local Points = Data.Points.Value
            if Points > 0 then
                local Remote = ReplicatedStorage:FindFirstChild("Remotes")
                if Remote then
                    pcall(function() Remote:InvokeServer("AddPoint", "Melee", 1) end)
                    task.wait(0.1)
                    pcall(function() Remote:InvokeServer("AddPoint", "Defense", 1) end)
                end
            end
        end
    end)
end

-- LOOP PRINCIPAL
local Contador = 0

RunService.RenderStepped:Connect(function()
    if FarmAtivo and LocalPlayer and LocalPlayer.Character then
        Contador = Contador + 1
        
        -- Chamar funções
        BringMobs()
        Attack()
        
        -- A cada 50 ciclos distribuir stats
        if Contador % 50 == 0 then
            DistribuirStats()
        end
        
        -- Status a cada 100 ciclos
        if Contador % 100 == 0 then
            local Level = pcall(function() return LocalPlayer.Data.Level.Value end)
            print("[STATUS] Ciclos: " .. Contador .. " | Farm Ativo: " .. tostring(FarmAtivo))
        end
    end
end)

-- CHARACTER RESPAWN
LocalPlayer.CharacterAdded:Connect(function(NewChar)
    print("[✓] Personagem respawnado!")
    Character = NewChar
    HumanoidRootPart = NewChar:WaitForChild("HumanoidRootPart")
end)
