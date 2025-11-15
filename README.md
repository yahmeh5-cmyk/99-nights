-- ================================================================= --
--                   SCRIPT ESP DE BAÚS (EXECUTOR)                   --
-- ================================================================= --

-- || CONFIGURAÇÃO || --

-- Adicione ou remova nomes da lista abaixo. O script vai procurar por QUALQUER parte/modelo com esses nomes.
-- Você pode precisar descobrir o nome exato dos baús no jogo. Nomes comuns são: "Chest", "Bau", "Treasure", "Crate", "Loot".
local NOMES_DOS_BAUS = {
    "Chest", -- Nome mais comum em jogos em inglês
    "Bau",
    "Treasure Chest",
    "Crate",
    "SupplyCrate"
}

-- Mude para 'false' para desligar o ESP
local ESP_ATIVADO = true

-- Cor do texto do ESP (em RGB, 0 a 255)
local COR_DO_TEXTO = Color3.fromRGB(255, 255, 0) -- Amarelo

-- || NÃO MEXA ABAIXO DISSO || --

-- Serviços e Variáveis
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("workspace")
local CoreGui = game:GetService("CoreGui") -- Usamos CoreGui para evitar que scripts do jogo interfiram

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

local ESP_Container = Instance.new("Folder", CoreGui)
ESP_Container.Name = "ESP_Container_" .. math.random(1, 1000)

local BaúsAtivos = {}

-- Função para criar a interface do ESP
local function criarESP(objeto)
    if not objeto or BaúsAtivos[objeto] then return end

    local BillboardGui = Instance.new("BillboardGui")
    BillboardGui.Name = "ESP_GUI"
    BillboardGui.AlwaysOnTop = true
    BillboardGui.Size = UDim2.new(0, 200, 0, 50)
    BillboardGui.Adornee = objeto
    BillboardGui.Parent = ESP_Container

    local TextLabel = Instance.new("TextLabel")
    TextLabel.Name = "InfoLabel"
    TextLabel.Parent = BillboardGui
    TextLabel.BackgroundColor3 = Color3.new(0, 0, 0)
    TextLabel.BackgroundTransparency = 1
    TextLabel.Size = UDim2.new(1, 0, 1, 0)
    TextLabel.Font = Enum.Font.SourceSansBold
    TextLabel.TextSize = 18
    TextLabel.TextColor3 = COR_DO_TEXTO
    TextLabel.TextStrokeTransparency = 0.5
    
    BaúsAtivos[objeto] = BillboardGui
end

-- Função para atualizar o texto (distância) e verificar validade
local function atualizarESPs()
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    
    local HRP = LocalPlayer.Character.HumanoidRootPart

    for objeto, gui in pairs(BaúsAtivos) do
        -- Se o objeto foi destruído, remove o ESP
        if not objeto or not objeto.Parent then
            gui:Destroy()
            BaúsAtivos[objeto] = nil
        else
            -- Calcula a distância e atualiza o texto
            local distancia = (HRP.Position - objeto.Position).Magnitude
            gui.InfoLabel.Text = string.format("%s\n[%.0fm]", objeto.Name, distancia)
        end
    end
end

-- Função para procurar por baús
local function procurarBaús()
    for _, objeto in pairs(Workspace:GetDescendants()) do
        if table.find(NOMES_DOS_BAUS, objeto.Name) then
            -- Tenta usar a parte principal se for um modelo, senão usa o próprio objeto
            local parteParaAdornar = objeto:IsA("Model") and (objeto.PrimaryPart or objeto:FindFirstChildOfClass("BasePart")) or objeto
            if parteParaAdornar then
                criarESP(parteParaAdornar)
            end
        end
    end
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    if not ESP_ATIVADO then 
        if ESP_Container.Parent then ESP_Container.Parent = nil end -- Desativa o ESP
        return 
    else
        if not ESP_Container.Parent then ESP_Container.Parent = CoreGui end -- Reativa o ESP
    end
    
    atualizarESPs()
end)

-- Procura baús a cada 5 segundos para encontrar novos que possam ter aparecido
task.spawn(function()
    while ESP_ATIVADO and task.wait(5) do
        pcall(procurarBaús)
    end
end)

-- Execução inicial
pcall(procurarBaús)
print("ESP de Baús carregado. Se nada aparecer, verifique a lista 'NOMES_DOS_BAUS' no script.")

-- Limpeza ao destruir o script (útil em alguns executores)
if getgenv then
    getgenv().ESP_Container = ESP_Container
end
script.Destroying:Connect(function()
    ESP_Container:Destroy()
    ESP_ATIVADO = false
end)
