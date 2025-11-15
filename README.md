--[[
	Script de ESP de Baú para Desenvolvedores
	- Colocar em StarterPlayer > StarterPlayerScripts
	- USA CollectionService. Certifique-se de que seus baús tenham a tag "Chest".
]]

-- Services
local RunService = game:GetService("RunService")
local CollectionService = game:GetService("CollectionService")
local Players = game:GetService("Players")

-- Configuração
local TAG_DO_BAU = "Chest" -- A tag que você usou para marcar os baús
local DISTANCIA_MAXIMA = 300 -- A distância máxima em studs para o ESP aparecer (0 = infinito)

-- Variáveis locais
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local activeEsps = {} -- Tabela para guardar os ESPs ativos: { [bauInstance] = espGui }

-- Função para criar a interface do ESP
local function createEspGui(target)
	local espGui = Instance.new("BillboardGui")
	espGui.Name = "ChestESP"
	espGui.AlwaysOnTop = true -- Faz o GUI aparecer através das paredes
	espGui.Size = UDim2.new(0, 150, 0, 40) -- Tamanho do GUI
	espGui.StudsOffset = Vector3.new(0, 2.5, 0) -- Posição acima do baú
	espGui.Adornee = target -- O objeto que o GUI vai seguir
	espGui.Parent = target -- Coloca o GUI dentro do próprio baú para organização

	local textLabel = Instance.new("TextLabel")
	textLabel.Size = UDim2.new(1, 0, 1, 0)
	textLabel.BackgroundColor3 = Color3.new(0, 0, 0)
	textLabel.BackgroundTransparency = 1
	textLabel.TextColor3 = Color3.fromRGB(255, 230, 100) -- Cor dourada
	textLabel.Font = Enum.Font.SourceSansBold
	textLabel.TextSize = 18
	textLabel.TextStrokeTransparency = 0.5 -- Contorno para melhor leitura
	textLabel.Text = target.Name
	textLabel.Parent = espGui

	return espGui
end

-- Função para atualizar o texto (distância)
local function updateEspText(character, bauInstance, esp)
	if not character or not character:FindFirstChild("HumanoidRootPart") then return end

	local rootPart = character.HumanoidRootPart
	local distance = (rootPart.Position 
