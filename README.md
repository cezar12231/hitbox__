--[[
  Hitbox Manager GUI v2.0 - Roblox
  Autor: Assistente
  Descrição: Cria uma interface arrastável para gerenciar hitboxes visuais
  em jogadores e NPCs. Acompanha o personagem mesmo após respawn.
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- =================== CONFIGURAÇÕES ===================
local CONFIG = {
	DefaultSize = 3,          -- tamanho padrão da hitbox
	DefaultTransparency = 0.5, -- transparência padrão (0 = opaco, 1 = invisível)
	Color = Color3.new(0.2, 0.6, 1), -- cor azul clara
	AutoApply = false          -- aplicar a todos automaticamente? (não usado aqui)
}

-- =================== VARIÁVEIS GLOBAIS DO SCRIPT ===================
local hitboxEnabled = false
local currentTarget = nil
local hitboxSize = CONFIG.DefaultSize
local hitboxTransparency = CONFIG.DefaultTransparency
local hitboxConnection = nil
local isMinimized = false
local guiMinimized = nil

-- =================== CRIAÇÃO DA GUI ===================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "HitboxManagerGUI"
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false

-- Frame principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 320, 0, 400)
mainFrame.Position = UDim2.new(0.5, -160, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Active = true

-- Arredondar cantos (via UI Corner)
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

-- DropShadow
local shadow = Instance.new("ImageLabel")
shadow.Name = "Shadow"
shadow.Size = UDim2.new(1, 10, 1, 10)
shadow.Position = UDim2.new(0, -5, 0, -5)
shadow.BackgroundTransparency = 1
shadow.Image = "rbxassetid://4740665940"
shadow.ImageColor3 = Color3.new(0, 0, 0)
shadow.ImageTransparency = 0.6
shadow.ScaleType = Enum.ScaleType.Slice
shadow.SliceCenter = Rect.new(10, 10, 20, 20)
shadow.Parent = mainFrame

-- Barra de título (arrastável)
local titleBar = Instance.new("Frame")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0, 30)
titleBar.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleBar

-- (apenas o canto superior fica arredondado; faremos um truque com um frame filho)
local titleTopCorner = Instance.new("UICorner")
titleTopCorner.CornerRadius = UDim.new(0, 8)
titleTopCorner.Parent = mainFrame
-- Usaremos um frame interno para "esconder" o canto inferior

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "Title"
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 10, 0, 0)
titleLabel.BackgroundTransparency = 
