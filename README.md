-- Carrega Fluent
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- Cria a Janela principal
local Window = Fluent:CreateWindow({
    Title = "wakazinn hub 1.0 " .. Fluent.Version,
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Theme = "Dark"
})

-- Botão minimizar
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 100, 0, 30)
minimizeBtn.Position = UDim2.new(1, -110, 0, 10)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
minimizeBtn.Text = "Minimizar"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.Font = Enum.Font.SourceSans
minimizeBtn.TextSize = 20
minimizeBtn.ZIndex = 100
minimizeBtn.Parent = Window.Frame

-- Ícone para reabrir
local playerGui = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
local reopenGui = Instance.new("ScreenGui", playerGui)
reopenGui.Name = "WakazinnReopenGUI"
reopenGui.ResetOnSpawn = false
reopenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local imgButton = Instance.new("ImageButton")
imgButton.Image = "rbxassetid://17112247010"
imgButton.Size = UDim2.new(0, 80, 0, 80)
imgButton.Position = UDim2.new(1, -100, 1, -100)
imgButton.BackgroundTransparency = 1
imgButton.Visible = false
imgButton.Parent = reopenGui

minimizeBtn.MouseButton1Click:Connect(function()
    Window.Frame.Visible = false
    imgButton.Visible = true
end)

imgButton.MouseButton1Click:Connect(function()
    Window.Frame.Visible = true
    imgButton.Visible = false
end)

-- Abas
local Tabs = {
    LagServe = Window:AddTab({ Title = "Lag Serve" }),
    AntiSit = Window:AddTab({ Title = "Anti Sit" }),
    Player = Window:AddTab({ Title = "Player" }),
    Trolls = Window:AddTab({ Title = "Trolls" }),
}

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local LocalPlayer = Players.LocalPlayer

-------------------------------
-- 📦 LAG SERVER (PREFEITURA - CLICKDETECTOR)
-------------------------------
local LoopAtivo = false

local function TeleportarPara(x, y, z)
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart", 5)
    if hrp then hrp.CFrame = CFrame.new(x, y, z) end
end

local function ClicarItemPrefeitura()
    local hrp = LocalPlayer.Character:WaitForChild("HumanoidRootPart")
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ClickDetector") then
            local part = obj.Parent
            if part and part:IsA("BasePart") then
                local distancia = (part.Position - hrp.Position).Magnitude
                if distancia <= 10 then
                    fireclickdetector(obj)
                end
            end
        end
    end
end

local function LoopPrefeitura()
    while LoopAtivo and #LocalPlayer.Backpack:GetChildren() < 500 do
        ClicarItemPrefeitura()
        task.wait(0.1)
    end
end

Tabs.LagServe:AddButton({
    Title = "⚠️ Lag Prefeitura (500 Itens)",
    Description = "Teleporta na prefeitura e clica no item até 500",
    Callback = function()
        LoopAtivo = true
        TeleportarPara(-321, 8.5, -107.9) -- Coordenada da prefeitura
        task.wait(1)
        LoopPrefeitura()
        for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
            pcall(function() item.Parent = LocalPlayer.Character end)
        end
    end
})

Tabs.LagServe:AddButton({
    Title = "❌ Parar Lag",
    Callback = function()
        LoopAtivo = false
    end
})

-------------------------------
-- 🪑 ANTI SIT
-------------------------------
Tabs.AntiSit:AddButton({
    Title = "🪑 Ativar Anti Sit",
    Description = "Impede que você sente",
    Callback = function()
        RunService.Stepped:Connect(function()
            local char = LocalPlayer.Character
            if char and char:FindFirstChild("Humanoid") then
                char.Humanoid.Sit = false
            end
        end)
    end
})

-------------------------------
-- 👁️ ESP NOME 3D
-------------------------------
Tabs.Player:AddButton({
    Title = "👁️ Ativar ESP Nome",
    Description = "Mostra nome 3D nos players",
    Callback = function()
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local head = plr.Character:FindFirstChild("Head")
                if head and not head:FindFirstChild("ESP") then
                    local esp = Instance.new("BillboardGui", head)
                    esp.Name = "ESP"
                    esp.Size = UDim2.new(0, 100, 0, 20)
                    esp.AlwaysOnTop = true

                    local name = Instance.new("TextLabel", esp)
                    name.Size = UDim2.new(1, 0, 1, 0)
                    name.BackgroundTransparency = 1
                    name.Text = plr.Name
                    name.TextColor3 = Color3.new(1, 0, 0)
                    name.TextScaled = true
                end
            end
        end
    end
})

-------------------------------
-- 👤 COPIAR AVATAR POR NOME
-------------------------------
local nomeAvatar = ""

Tabs.Player:AddInput("InputAvatar", {
    Title = "Copiar Avatar de",
    Placeholder = "Ex: wakazinn",
    Default = "",
}):OnChanged(function(value)
    nomeAvatar = value
end)

Tabs.Player:AddButton({
    Title = "👤 Copiar Avatar",
    Callback = function()
        local target = Players:FindFirstChild(nomeAvatar)
        if target and target.Character then
            local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            local description = target.Character:FindFirstChildOfClass("Humanoid"):GetAppliedDescription()
            humanoid:ApplyDescription(description)
        end
    end
})

-------------------------------
-- 🚪 NOCLIP (ATRAVESSAR PAREDES)
-------------------------------
local noclipAtivo = false
local noclipConn

local function ativarNoclip()
    noclipConn = RunService.Stepped:Connect(function()
        local char = LocalPlayer.Character
        if char then
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end)
end

local function desativarNoclip()
    if noclipConn then
        noclipConn:Disconnect()
        noclipConn = nil
    end
end

Tabs.Player:AddToggle("NoclipToggle", {
    Title = "🚪 Atravessar Paredes (Noclip)",
    Default = false,
    Callback = function(value)
        noclipAtivo = value
        if noclipAtivo then
            ativarNoclip()
        else
            desativarNoclip()
        end
    end
})

-------------------------------
-- 🚓 PRENDER JOGADOR
-------------------------------
local nickParaPrender = ""

Tabs.Trolls:AddInput("InputPrender", {
    Title = "Nome do Jogador",
    Placeholder = "Ex: wakazinn",
}):OnChanged(function(value)
    nickParaPrender = value:sub(1,1) == "@" and value:sub(2) or value
end)

Tabs.Trolls:AddButton({
    Title = "🚓 Prender Jogador",
    Description = "Teleporta até o jogador e depois leva para cela",
    Callback = function()
        local target = Players:FindFirstChild(nickParaPrender)
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if myHRP then
                myHRP.CFrame = target.Character.HumanoidRootPart.CFrame + Vector3.new(0, 2, 0)
                task.delay(2, function()
                    if myHRP then
                        myHRP.CFrame = CFrame.new(95.1, 64.5, -506.0) -- Cela
                    end
                end)
            end
        else
            warn("Jogador não encontrado.")
        end
    end
})
