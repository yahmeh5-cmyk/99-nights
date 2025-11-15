-- ================================================================= --
--         SCRIPT ESP DE BAÚS v2 - COM DIAGNÓSTICO E GUI             --
-- ================================================================= --
--[[
    Este script foi melhorado com mensagens de diagnóstico.
    Abra o console do seu executor para ver o que o script está fazendo.
]]

-- || CONFIGURAÇÃO - A PARTE MAIS IMPORTANTE! || --
-- EDITE ESTA LISTA com o nome EXATO do baú no jogo.
local NOMES_DOS_BAUS = {
    "Chest",
    "Bau",
    "Treasure Chest",
    "Crate",
    "SupplyCrate"
    -- Exemplo: Se o nome for "ForestChest", adicione "ForestChest" aqui.
}

-- ================================================================= --
-- ||     NÚCLEO DO SCRIPT (GUI E ESP) - NÃO PRECISA EDITAR     || --
-- ================================================================= --

-- Carregamento seguro da biblioteca da interface
local OrionLib
local success, result = pcall(function()
    OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()
end)

if not success then
    warn("Orion Library: Falha ao carregar a biblioteca de interface. Verifique a conexão com a internet.")
    return
end
print("Orion Library: Biblioteca de interface carregada com sucesso.")

-- Variáveis e Serviços
local ESP_Ativado = true
local RunService = game:GetService("RunService")
local Workspace = game:GetService("workspace")
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

if CoreGui:FindFirstChild("Baú_ESP_Container_v2") then
    CoreGui.Baú_ESP_Container_v2:Destroy()
end
local ESP_Container = Instance.new("Folder", CoreGui)
ESP_Container.Name = "Baú_ESP_Container_v2"

local BaúsAtivos = {}

-- Função para achar a parte principal de um objeto
local function findTargetPart(objeto)
    return objeto:IsA("BasePart") and objeto or (objeto:IsA("Model") and (objeto.PrimaryPart or objeto:FindFirstChildOfClass("BasePart")))
end

-- Função que cria o visual do ESP
local function criarESP(objeto)
    local part = findTargetPart(objeto)
    if not part or BaúsAtivos[part] then return end
    
    -- Mensagem de diagnóstico
    print("DIAGNÓSTICO: Encontrado objeto '" .. part.Name .. "'. Criando ESP.")

    local BillboardGui = Instance.new("BillboardGui")
    BillboardGui.AlwaysOnTop = true
    BillboardGui.Size = UDim2.new(0, 200, 0, 50)
    BillboardGui.Adornee = part
    BillboardGui.Parent = ESP_Container

    local TextLabel = Instance.new("TextLabel")
    TextLabel.Parent = BillboardGui
    TextLabel.BackgroundColor3 = Color3.new(0, 0, 0)
    TextLabel.BackgroundTransparency = 1
    TextLabel.Size = UDim2.new(1, 0, 1, 0)
    TextLabel.Font = Enum.Font.SourceSansSemibold
    TextLabel.TextSize = 19
    TextLabel.TextColor3 = Color3.fromRGB(255, 230, 100)
    TextLabel.TextStrokeTransparency = 0.5
    TextLabel.TextStrokeColor3 = Color3.new(0, 0, 0)

    BaúsAtivos[part] = BillboardGui
end

-- Função que procura por baús
local function procurarBaús()
    print("DIAGNÓSTICO: Procurando por baús no Workspace...")
    for _, objeto in ipairs(Workspace:GetDescendants()) do
        if table.find(NOMES_DOS_BAUS, objeto.Name) then
            criarESP(objeto)
        end
    end
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    if not ESP_Ativado then return end
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    
    local HRP_Position = LocalPlayer.Character.HumanoidRootPart.Position

    for part, gui in pairs(BaúsAtivos) do
        if not part or not part.Parent or not gui or not gui.Parent then
            if gui then gui:Destroy() end
            BaúsAtivos[part] = nil
        else
            local distancia = (HRP_Position - part.Position).Magnitude
            gui.TextLabel.Text = string.format("%s\n[%.0fm]", part.Name, distancia)
        end
    end
end)

-- Loop secundário para encontrar novos baús
task.spawn(function()
    while task.wait(7) do
        if ESP_Ativado then
            pcall(procurarBaús)
        end
    end
end)

-- Criação da Interface Gráfica
local Window = OrionLib:MakeWindow({Name = "99 Nights Dev Tool v2", HidePremium = true, SaveConfig = true, ConfigFolder = "OrionConfig99"})
local Tab = Window:MakeTab({Name = "ESP", Icon = "rbxassetid://4483345998"})

Tab:AddLabel("Controle o ESP de Baús")
Tab:AddToggle({
    Name = "ESP de Baús",
    Default = true,
    Callback = function(Value)
        ESP_Ativado = Value
        ESP_Container.Enabled = Value
        if Value then print("ESP ATIVADO") pcall(procurarBaús) else print("ESP DESATIVADO") end
    end
})

Tab:AddLabel("IMPORTANTE: Se nada aparecer, edite a lista 'NOMES_DOS_BAUS' no topo do script com o nome exato do baú.")

-- Inicia tudo
pcall(procurarBaús)
OrionLib:Init()
print("Interface carregada. Verifique o console para mensagens de diagnóstico.")
