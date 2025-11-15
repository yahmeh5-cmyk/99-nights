-- ================================================================= --
--         SCRIPT ESP DE BAÚS COM INTERFACE PARA EXECUTOR            --
--                por [Seu Nome/Nome do Grupo]                       --
-- ================================================================= --
--[[
    INSTRUÇÕES:
    1. Copie e cole este script no seu executor.
    2. Execute enquanto estiver no jogo "99 Nights in the Forest".
    3. Uma interface vai aparecer. Use o botão para ligar/desligar o ESP.
    4. Se nenhum baú aparecer, EDITE A LISTA 'NOMES_DOS_BAUS' ABAIXO com o nome correto do baú no jogo.
       (Use uma ferramenta como "Dark Dex" ou "Remote Spy" para encontrar o nome).
]]

-- || CONFIGURAÇÃO (EDITAR SE NECESSÁRIO) || --
local NOMES_DOS_BAUS = {
    "Chest",          -- Nome genérico em inglês
    "Bau",            -- Nome genérico em português
    "Treasure Chest", -- Outro nome comum
    "SupplyCrate",    -- Comum em jogos de sobrevivência
    "Crate"
}

local COR_DO_TEXTO = Color3.fromRGB(255, 230, 100) -- Dourado

-- ================================================================= --
-- ||     INTERFACE GRÁFICA (ORION LIBRARY) E CÓDIGO DO ESP     || --
-- ================================================================= --

-- Carrega a biblioteca da interface
local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

-- Variáveis Globais
local ESP_Ativado = true -- O ESP começa ligado por padrão
local RunService = game:GetService("RunService")
local Workspace = game:GetService("workspace")
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer

-- Container para os ESPs, para manter tudo organizado
local ESP_Container = Instance.new("Folder", CoreGui)
ESP_Container.Name = "Baú_ESP_Container"

local BaúsAtivos = {} -- Tabela para rastrear os baús e suas GUIs

-- Função para encontrar a parte principal do baú para anexar o ESP
local function findTargetPart(objeto)
    if objeto:IsA("BasePart") then return objeto end
    if objeto:IsA("Model") then
        return objeto.PrimaryPart or objeto:FindFirstChildOfClass("BasePart") or objeto:FindFirstChildOfClass("MeshPart")
    end
    return nil
end

-- Função que cria o visual do ESP (BillboardGui)
local function criarESP(objeto)
    local part = findTargetPart(objeto)
    if not part or BaúsAtivos[part] then return end

    local BillboardGui = Instance.new("BillboardGui")
    BillboardGui.Name = "ESP_GUI"
    BillboardGui.AlwaysOnTop = true
    BillboardGui.Size = UDim2.new(0, 200, 0, 50)
    BillboardGui.Adornee = part
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
    TextLabel.TextStrokeTransparency = 0.4
    TextLabel.TextStrokeColor3 = Color3.new(0, 0, 0)

    BaúsAtivos[part] = BillboardGui
end

-- Função que procura por novos baús no mapa
local function procurarBaús()
    for _, objeto in ipairs(Workspace:GetDescendants()) do
        if table.find(NOMES_DOS_BAUS, objeto.Name) then
            criarESP(objeto)
        end
    end
end

-- Loop principal que roda a cada frame para atualizar as informações
RunService.RenderStepped:Connect(function()
    if not ESP_Ativado then return end -- Se estiver desativado, não faz nada

    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local HRP_Position = character.HumanoidRootPart.Position

    for part, gui in pairs(BaúsAtivos) do
        -- Limpeza: remove o ESP se o baú for destruído
        if not part or not part.Parent or not gui or not gui.Parent then
            if gui then gui:Destroy() end
            BaúsAtivos[part] = nil
        else
            -- Atualiza o texto com a distância
            local distancia = (HRP_Position - part.Position).Magnitude
            gui.InfoLabel.Text = string.format("%s\n[%.0fm]", part.Name, distancia)
        end
    end
end)

-- Loop secundário que procura por novos baús a cada 5 segundos
task.spawn(function()
    while task.wait(5) do
        if ESP_Ativado then
            pcall(procurarBaús)
        end
    end
end)

-- Criação da Interface
local Window = OrionLib:MakeWindow({
    Name = "99 Nights Dev Tool",
    HidePremium = true,
    SaveConfig = true,
    ConfigFolder = "Orion_Config_99Nights"
})

local Tab = Window:MakeTab({
    Name = "ESP",
    Icon = "rbxassetid://4483345998", -- Ícone de um olho
    PremiumOnly = false
})

Tab:AddToggle({
    Name = "ESP de Baús",
    Default = true, -- O botão começa ligado
    Callback = function(Value)
        -- Esta função é chamada sempre que o botão é clicado
        ESP_Ativado = Value
        ESP_Container.Enabled = Value
        
        if ESP_Ativado then
            print("ESP de Baús ATIVADO")
            pcall(procurarBaús) -- Procura baús assim que é ativado
        else
            print("ESP de Baús DESATIVADO")
        end
    end
})

-- Execução Inicial
pcall(procurarBaús)
OrionLib:Init()
print("Interface carregada! Use o menu para controlar o ESP.")
