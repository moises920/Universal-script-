-- botão flutuante discreto (sempre visível)
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Button = Instance.new("TextButton", ScreenGui)

Button.Size = UDim2.new(0, 40, 0, 40)
Button.Position = UDim2.new(1, -60, 1, -60)
Button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Button.Text = "💀"
Button.TextColor3 = Color3.fromRGB(255, 255, 255)
Button.Font = Enum.Font.GothamBold
Button.TextSize = 20
Button.Active = true
Button.Draggable = true
Button.BackgroundTransparency = 0.2
Button.BorderSizePixel = 0
Button.AutoButtonColor = true

Button.MouseEnter:Connect(function() Button.BackgroundColor3 = Color3.fromRGB(70, 70, 70) end)
Button.MouseLeave:Connect(function() Button.BackgroundColor3 = Color3.fromRGB(40, 40, 40) end)

-- função principal que carrega todo o hub
local function LoadHub()
    -- carregar biblioteca Fluent
    local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

    local Window = Fluent:CreateWindow({
        Title = "Caveira Hub " .. tostring(Fluent.Version),
        TabWidth = 160, 
        Size = UDim2.fromOffset(580, 460), 
        Theme = "Dark"
    })

    local Tabs = {
        Main = Window:AddTab({ Title = "Main" }),
        Settings = Window:AddTab({ Title = "Settings", Icon = "settings" }),
        Extras = Window:AddTab({ Title = "Extras" }),
        Farm = Window:AddTab({ Title = "Farm" }),
        Visual = Window:AddTab({ Title = "Visual" }),
        Player = Window:AddTab({Title = "Player"}),
        Combat = Window:AddTab({Title="Combat"}),
        Fun = Window:AddTab({Title="Fun"}),
        Utility = Window:AddTab({Title="Utility"})
    }

    local UIS = game:GetService("UserInputService")
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local HttpService = game:GetService("HttpService")

    -- hotkey RightShift para abrir/fechar
    UIS.InputBegan:Connect(function(input, gp)
        if not gp and input.KeyCode == Enum.KeyCode.RightShift then
            Window:Toggle()
        end
    end)

    -- Main Tab
    Tabs.Main:AddParagraph({ Title = "Caveira Hub Script", Content = "Script novo do Caveira" })

    -- Infinite Jump
    Tabs.Main:AddButton({
        Title = "Infinite Jump",
        Callback = function()
            local InfiniteJumpEnabled = true
            UIS.JumpRequest:Connect(function()
                if InfiniteJumpEnabled then
                    local char = Players.LocalPlayer.Character
                    if char and char:FindFirstChildOfClass("Humanoid") then
                        char:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
                    end
                end
            end)
            Fluent:Notify({ Title = "Infinite Jump", Content = "Ativado com sucesso!" })
        end
    })

    -- JumpPower Slider
    local JumpSlider = Tabs.Main:AddSlider("pulo", {
        Title = "Ajustar Pulo",
        Description = "Muda a altura do pulo",
        Default = 50,
        Min = 0,
        Max = 200,
        Rounding = 0,
        Callback = function(Value)
            local humanoid = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.JumpPower = Value
            end
        end
    })
    JumpSlider:OnChanged(function(Value)
        print("JumpPower mudou para:", Value)
    end)

    -- WalkSpeed Slider
    local WalkSlider = Tabs.Main:AddSlider("velocidade", {
        Title = "Ajustar Velocidade",
        Description = "Muda a velocidade do jogador",
        Default = 16,
        Min = 10,
        Max = 200,
        Rounding = 0,
        Callback = function(Value)
            local humanoid = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.WalkSpeed = Value
            end
        end
    })
    WalkSlider:OnChanged(function(Value)
        print("WalkSpeed mudou para:", Value)
    end)

    -- NoClip toggle
    local noclipConnection
    Tabs.Main:AddToggle("noclip", {
        Title = "NoClip",
        Description = "Permite atravessar paredes",
        Default = false,
        Callback = function(state)
            if state then
                Fluent:Notify({ Title = "NoClip", Content = "Ativado" })
                noclipConnection = RunService.Stepped:Connect(function()
                    local char = Players.LocalPlayer.Character
                    if char then
                        for _, part in pairs(char:GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.CanCollide = false
                            end
                        end
                    end
                end)
            else
                if noclipConnection then
                    noclipConnection:Disconnect()
                    noclipConnection = nil
                end
                Fluent:Notify({ Title = "NoClip", Content = "Desativado" })
            end
        end
    })

    -- Fly Mode (Extras Tab)
    local flying = false
    local flyConnection
    Tabs.Extras:AddToggle("fly", {
        Title = "Fly Mode",
        Description = "Permite voar livremente",
        Default = false,
        Callback = function(state)
            local char = Players.LocalPlayer.Character
            local humanoidRootPart = char and char:FindFirstChild("HumanoidRootPart")
            local speed = 50

            if state and humanoidRootPart then
                flying = true
                Fluent:Notify({ Title = "Fly", Content = "Ativado" })
                flyConnection = RunService.RenderStepped:Connect(function()
                    if flying and humanoidRootPart then
                        local moveDir = Vector3.zero
                        if UIS:IsKeyDown(Enum.KeyCode.W) then moveDir += workspace.CurrentCamera.CFrame.LookVector end
                        if UIS:IsKeyDown(Enum.KeyCode.S) then moveDir -= workspace.CurrentCamera.CFrame.LookVector end
                        if UIS:IsKeyDown(Enum.KeyCode.A) then moveDir -= workspace.CurrentCamera.CFrame.RightVector end
                        if UIS:IsKeyDown(Enum.KeyCode.D) then moveDir += workspace.CurrentCamera.CFrame.RightVector end
                        if UIS:IsKeyDown(Enum.KeyCode.Space) then moveDir += Vector3.new(0,1,0) end
                        if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir -= Vector3.new(0,1,0) end
                        humanoidRootPart.Velocity = moveDir * speed
                    end
                end)
            else
                flying = false
                if flyConnection then
                    flyConnection:Disconnect()
                    flyConnection = nil
                end
                if humanoidRootPart then
                    humanoidRootPart.Velocity = Vector3.zero
                end
                Fluent:Notify({ Title = "Fly", Content = "Desativado" })
            end
        end
    })

    -- Auto Buy/Store (Farm Tab)
    Tabs.Farm:AddToggle("autobuy", {
        Title = "Auto Buy/Store",
        Description = "Compra automaticamente itens/lojas",
        Default = false,
        Callback = function(state)
            if state then
                Fluent:Notify({ Title = "Auto Buy", Content = "Ativado" })
                -- código de Auto Buy/Store vai aqui
            else
                Fluent:Notify({ Title = "Auto Buy", Content = "Desativado" })
            end
        end
    })

    -- ESP de players/NPCs (Visual Tab)
    Tabs.Visual:AddToggle("esp", {
        Title = "ESP Players/NPCs",
        Description = "Mostra jogadores e NPCs",
        Default = false,
        Callback = function(state)
            local function CreateESP(model)
                if model:IsA("Model") and model:FindFirstChild("HumanoidRootPart") then
                    local highlight = Instance.new("Highlight")
                    highlight.Adornee = model
                    highlight.FillColor = Color3.new(1,0,0)
                    highlight.OutlineColor = Color3.new(1,1,1)
                    highlight.Parent = model
                end
            end
            if state then
                for _, plr in pairs(Players:GetPlayers()) do
                    if plr ~= Players.LocalPlayer then
                        CreateESP(plr.Character or plr.CharacterAdded:Wait())
                    end
                end
                for _, npc in pairs(workspace:GetChildren()) do
                    if npc:IsA("Model") and npc:FindFirstChild("Humanoid") then
                        CreateESP(npc)
                    end
                end
            else
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("Highlight") then
                        v:Destroy()
                    end
                end
            end
        end
    })

    -- Server Hop (Settings Tab)
    Tabs.Settings:AddButton({
        Title = "Server Hop",
        Callback = function()
            local PlaceID = game.PlaceId
            local Servers = {}
            local success, data = pcall(function()
                return game:HttpGet("https://games.roblox.com/v1/games/"..PlaceID.."/servers/Public?sortOrder=Asc&limit=100")
            end)
            if success then
                local decoded = HttpService:JSONDecode(data)
                for _, s in pairs(decoded.data) do
                    if s.id ~= game.JobId and s.playing < s.maxPlayers then
                        table.insert(Servers, s.id)
                    end
                end
            end
            if #Servers > 0 then
                local RandomServer = Servers[math.random(1,#Servers)]
                game:GetService("TeleportService"):TeleportToPlaceInstance(PlaceID, RandomServer, Players.LocalPlayer)
            else
                Fluent:Notify({ Title = "Server Hop", Content = "Nenhum servidor disponível!" })
            end
        end
    })

    -- Anti-Stun (Settings Tab)
    Tabs.Settings:AddToggle("antistun", {
        Title = "Anti-Stun",
        Description = "Previne stun",
        Default = true,
        Callback = function(state)
            local char = Players.LocalPlayer.Character
            if state and char then
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.StateChanged:Connect(function(_, newState)
                        if newState == Enum.HumanoidStateType.Seated or newState == Enum.HumanoidStateType.FallingDown then
                            humanoid:ChangeState(Enum.HumanoidStateType.Running)
                        end
                    end)
                end
            end
        end
    })

    -- ==============================
    -- Player Tab
    -- ==============================
    Tabs.Player:AddDropdown("teleportTo", {
        Title = "Teleport To Player",
        Description = "Teleporta até outro jogador",
        Options = function()
            local list = {}
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= Players.LocalPlayer then
                    table.insert(list, plr.Name)
                end
            end
            return list
        end,
        Callback = function(selected)
            local target = Players:FindFirstChild(selected)
            if target and target.Character and Players.LocalPlayer.Character then
                Players.LocalPlayer.Character:MoveTo(target.Character.HumanoidRootPart.Position)
                Fluent:Notify({Title="Teleport", Content="Teleported to "..selected})
            end
        end
    })

    Tabs.Player:AddToggle("invisible", {
        Title = "Invisibility",
        Description = "Fica invisível",
        Default = false,
        Callback = function(state)
            local char = Players.LocalPlayer.Character
            if char then
                for _, part in pairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.Transparency = state and 1 or 0
                        part.CanCollide = not state
                    end
                end
            end
            Fluent:Notify({Title="Invisibility", Content=state and "Ativado" or "Desativado"})
        end
    })

    Tabs.Player:AddButton({Title="Reset Player", Callback=function()
        if Players.LocalPlayer.Character then
            Players.LocalPlayer.Character:BreakJoints()
        end
    end})

    -- ==============================
    -- Combat Tab
    -- ==============================
    Tabs.Combat:AddButton({Title="Super Punch", Callback=function()
        local char = Players.LocalPlayer.Character
        if char and char:FindFirstChildOfClass("Humanoid") then
            char:FindFirstChildOfClass("Humanoid").Damage = 100
            Fluent:Notify({Title="Combat", Content="Super Punch Ativado!"})
        end
    end})

    local killAura = false
    Tabs.Combat:AddToggle("killAura", {
        Title="Kill Aura",
        Description="Ataca NPCs próximos",
        Default=false,
        Callback=function(state)
            killAura = state
            Fluent:Notify({Title="Kill Aura", Content=state and "Ativado" or "Desativado"})
        end
    })

    RunService.RenderStepped:Connect(function()
        if killAura then
            local char = Players.LocalPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                for _, npc in pairs(workspace:GetChildren()) do
                    if npc:IsA("Model") and npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") then
                        local dist = (char.HumanoidRootPart.Position - npc.HumanoidRootPart.Position).Magnitude
                        if dist < 10 then
                            npc.Humanoid.Health = 0
                        end
                    end
                end
            end
        end
    end)

    -- ==============================
    -- Fun Tab
    -- ==============================
    Tabs.Fun:AddToggle("bigHead", {
        Title="Big Head",
        Description="Cabeças gigantes",
        Default=false,
        Callback=function(state)
            for _, plr in pairs(Players:GetPlayers()) do
                if plr.Character and plr.Character:FindFirstChild("Head") then
                    plr.Character.Head.Mesh.Scale = state and Vector3.new(5,5,5) or Vector3.new(1,1,1)
                end
            end
        end
    })

    Tabs.Fun:AddToggle("lowGravity", {
        Title="Low Gravity",
        Description="Pula mais alto e cai devagar",
        Default=false,
        Callback=function(state)
            workspace.Gravity = state and 50 or 196.2
        end
    })

    Tabs.Fun:AddButton({Title="Fire Feet", Callback=function()
        local char = Players.LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local fire = Instance.new("Fire", char.HumanoidRootPart)
            fire.Heat = 10
            fire.Size = 5
            Fluent:Notify({Title="Fun", Content="Fogo nos pés ativado!"})
        end
    end})

    -- ==============================
    -- Utility Tab
    -- ==============================
    Tabs.Utility:AddButton({Title="Anti AFK", Callback=function()
        local vu = game:GetService("VirtualUser")
        Players.LocalPlayer.Idled:Connect(function()
            vu:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
            wait(1)
            vu:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
        end)
        Fluent:Notify({Title="Utility", Content="Anti AFK ativado!"})
    end})

    Tabs.Utility:AddButton({Title="Rejoin", Callback=function()
        local ts = game:GetService("TeleportService")
        ts:Teleport(game.PlaceId, Players.LocalPlayer)
    end})

    Tabs.Utility:AddButton({Title="Clear ESP", Callback=function()
        for _, v in pairs(workspace:GetDescendants()) do
            if v:IsA("Highlight") then v:Destroy() end
        end
        Fluent:Notify({Title="Utility", Content="ESP removido!"})
    end})

end

-- botão flutuante recarrega o hub
Button.MouseButton1Click:Connect(function() LoadHub() end)

-- carrega o hub pela primeira vez
LoadHub()
