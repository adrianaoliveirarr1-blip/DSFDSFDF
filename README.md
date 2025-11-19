-- Exemplo de Kill Aura para TESTES (Love2D)
-- Uso: pressione G para alternar KillAura on/off.
-- Atenção: usar somente em ambiente de desenvolvimento/local para testes de balanceamento.

local player = {
  x = 400, y = 300,
  radius = 120,         -- alcance da aura
  damage = 25,          -- dano por ataque
  interval = 0.4,       -- segundos entre ataques
  maxTargets = 5,       -- número máximo de alvos por golpe
  team = "player"
}

-- Exemplos de inimigos (em um jogo real, use sua estrutura de entidades)
local enemies = {
  { id=1, x=500, y=300, hp=300, isBoss=true, team="enemy" },
  { id=2, x=420, y=320, hp=80, isBoss=false, team="enemy" },
  { id=3, x=300, y=280, hp=120, isBoss=false, team="enemy" },
  { id=4, x=700, y=500, hp=150, isBoss=false, team="enemy" },
}

local killAuraActive = false
local attackTimer = 0

local function distance(a,b)
  local dx = a.x - b.x
  local dy = a.y - b.y
  return math.sqrt(dx*dx + dy*dy)
end

local function findTargets()
  local candidates = {}
  for _,e in ipairs(enemies) do
    if e.hp > 0 and e.team ~= player.team then
      local d = distance(player, e)
      if d <= player.radius then
        table.insert(candidates, {ent=e, dist=d})
      end
    end
  end
  -- Prioridade: boss primeiro, depois por distância
  table.sort(candidates, function(a,b)
    if (a.ent.isBoss and not b.ent.isBoss) then return true end
    if (b.ent.isBoss and not a.ent.isBoss) then return false end
    return a.dist < b.dist
  end)
  -- pegar até maxTargets
  local targets = {}
  for i=1, math.min(#candidates, player.maxTargets) do
    table.insert(targets, candidates[i].ent)
  end
  return targets
end

local function applyDamageToTargets(targets)
  for _,t in ipairs(targets) do
    t.hp = t.hp - player.damage
    if t.hp <= 0 then
      t.hp = 0
      -- Marcar morto; em jogo real, dispare evento de morte, loot, etc.
    end
  end
end

function love.load()
  love.window.setTitle("Kill Aura - Teste (pressione G para alternar)")
  love.graphics.setFont(love.graphics.newFont(14))
end

function love.update(dt)
  -- Movimentação de exemplo: mover jogador com setas (para testar)
  local speed = 200
  if love.keyboard.isDown("left") or love.keyboard.isDown("a") then player.x = player.x - speed*dt end
  if love.keyboard.isDown("right") or love.keyboard.isDown("d") then player.x = player.x + speed*dt end
  if love.keyboard.isDown("up") or love.keyboard.isDown("w") then player.y = player.y - speed*dt end
  if love.keyboard.isDown("down") or love.keyboard.isDown("s") then player.y = player.y + speed*dt end

  if killAuraActive then
    attackTimer = attackTimer - dt
    if attackTimer <= 0 then
      local targets = findTargets()
      if #targets > 0 then
        applyDamageToTargets(targets)
      end
      attackTimer = player.interval
    end
  end
end

function love.keypressed(key)
  if key == "g" then
    killAuraActive = not killAuraActive
    if killAuraActive then
      attackTimer = 0 -- ataque imediato ao ligar
    end
  end
end

function love.draw()
  -- Desenhar jogador
  love.graphics.setColor(0.2, 0.6, 1.0)
  love.graphics.circle("fill", player.x, player.y, 12)
  -- Desenhar alcance da aura
  if killAuraActive then
    love.graphics.setColor(1,0.2,0.2,0.25)
  else
    love.graphics.setColor(0.2,0.8,0.2,0.12)
  end
  love.graphics.circle("fill", player.x, player.y, player.radius)

  -- Desenhar inimigos
  for _,e in ipairs(enemies) do
    if e.hp > 0 then
      if e.isBoss then love.graphics.setColor(0.7,0.1,0.7)
      else love.graphics.setColor(0.9,0.3,0.1) end
    else
      love.graphics.setColor(0.4,0.4,0.4) -- morto
    end
    love.graphics.rectangle("fill", e.x-10, e.y-10, 20, 20)
    -- HP bar
    love.graphics.setColor(0,0,0)
    love.graphics.rectangle("line", e.x-15, e.y-18, 30, 5)
    local pct = math.max(0, e.hp) / (e.isBoss and 300 or 150)
    love.graphics.setColor(0.2,0.9,0.2)
    love.graphics.rectangle("fill", e.x-15, e.y-18, 30 * pct, 5)
  end

  -- UI
  love.graphics.setColor(1,1,1)
  love.graphics.print(("KillAura: %s (G)"):format(killAuraActive and "ON" or "OFF"), 10, 10)
  love.graphics.print(("Dano: %d  Alcance: %d  Intervalo: %.2fs"):format(player.damage, player.radius, player.interval), 10, 30)
  love.graphics.print("Use setas/WASD para mover o jogador", 10, 50)
end
