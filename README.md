# God-hub
local espada = script.Parent  -- A referência à espada (Tool)
local dano = 25  -- O dano que a espada vai causar
local cooldown = 1  -- Tempo de recarga entre os ataques
local alcance = 5  -- Alcance do dano
local efeitoParticula = "rbxassetid://12345678"  -- Efeito visual da espada
local somAtaque = "rbxassetid://87654321"  -- Som de ataque

local players = game:GetService("Players")
local debris = game:GetService("Debris")

-- Função para dar dano
local function darDano(alvo)
    if alvo and alvo:FindFirstChild("Humanoid") then
        local humanoide = alvo:FindFirstChild("Humanoid")
        humanoide:TakeDamage(dano)
    end
end

-- Função para criar o efeito visual
local function criarEfeito(posicao)
    local part = Instance.new("Part")
    part.Shape = Enum.PartType.Ball
    part.Size = Vector3.new(1, 1, 1)
    part.Position = posicao
    part.Anchored = true
    part.CanCollide = false
    part.Material = Enum.Material.Neon
    part.Color = Color3.fromRGB(255, 0, 0)  -- Cor vermelha para o efeito
    part.Transparency = 0.5
    part.Parent = game.Workspace

    -- Efeito de partícula
    local particula = Instance.new("ParticleEmitter")
    particula.Texture = efeitoParticula
    particula.Parent = part
    debris:AddItem(part, 2)  -- Remove a partícula após 2 segundos
end

-- Função para tocar o som de ataque
local function tocarSom(posicao)
    local som = Instance.new("Sound")
    som.SoundId = somAtaque
    som.Volume = 1
    som.Parent = game.Workspace
    som.Position = posicao
    som:Play()
    debris:AddItem(som, 2)  -- Remove o som após 2 segundos
end

-- Função de ataque
local function atacar()
    local mouse = espada.Parent:WaitForChild("Humanoid").Parent:FindFirstChild("HumanoidRootPart")
    local posicaoEspada = espada.Handle.Position

    -- Visibilidade da espada para todos os jogadores
    espada.Handle.Transparency = 0  -- Torna a espada visível
    espada.Handle.CanCollide = false  -- Evita colisões com objetos

    -- Tocar o som de ataque
    tocarSom(posicaoEspada)

    -- Criar efeito visual
    criarEfeito(posicaoEspada)

    -- Verifica todos os jogadores próximos
    for _, jogador in ipairs(players:GetPlayers()) do
        if jogador.Character and jogador.Character:FindFirstChild("HumanoidRootPart") then
            local distancia = (posicaoEspada - jogador.Character.HumanoidRootPart.Position).Magnitude
            if distancia <= alcance then
                darDano(jogador.Character)
            end
        end
    end
end

-- Mecanismo de cooldown
local function iniciarCooldown()
    local cooldownFlag = false
    espada.Activated:Connect(function()
        if not cooldownFlag then
            cooldownFlag = true
            atacar()
            wait(cooldown)  -- Espera o tempo de cooldown
            cooldownFlag = false
        end
    end)
end

-- Inicia a espada com cooldown e ataque
iniciarCooldown()
