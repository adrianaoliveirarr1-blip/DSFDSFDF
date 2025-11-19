
``lua name=ServerScriptService/DevKillAuraServer.lua
-- Script (coloque em ServerScriptService)
-- Recebe o toggle do cliente e aplica dano nos NPCs dentro de uma pasta específica.
-- Ajuste `ENEMY_FOLDER` para a pasta onde seus NPCs ficam (ex: Workspace.TestEnemies).

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")

-- CONFIGURAÇÃO
local REMOTE_NAME = "DevKillAuraEvent"
local ENEMY_FOLDER = Workspace:FindFirstChild("TestEnemies") or Workspace:FindFirstChild("Enemies") -- ajuste conforme seu jogo
local WHITELIST = { -- coloque aqui os UserIds dos desenvolvedores/testers
	[12345678] = true, -- exemplo: substitua com seu UserId
	-- [98765432] = true,
}
local ONLY_IN_STUDIO = true -- se true, só permite quando RunService:IsStudio() ou player na whitelist

local AURA_CONFIG = {
	radius = 20,       -- alcance (studs)
	damage = 25,       -- dano por ataque
	interval = 0.4,    -- segundos entre ataques
	maxTargets = 5,    -- max alvos por ataque
}

-- cria o RemoteEvent caso não exista
local remote = ReplicatedStorage:FindFirstChild(REMOTE_NAME)
if not remote then
	remote = Instance.new("RemoteEvent")
	remote.Name = REMOTE_NAME
	remote.Parent = ReplicatedStorage
	print("[DevKillAura] RemoteEvent criado em ReplicatedStorage.")
end

-- estado por jogador
local activePlayers = {} -- [player] = { on = bool, nextTick = os/time }

local function isAuthorized(player)
	-- Autoriza se estiver em Studio e ONLY_IN_STUDIO, ou se estiver na whitelist
	if WHITELIST[player.UserId] then return true end
	if RunService:IsStudio() and ONLY_IN_STUDIO then return true end
	-- adicional: autorizar o criador do jogo
	if game.CreatorType ~= Enum.Creator

