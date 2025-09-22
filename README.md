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

Button.MouseEnter:Connect(function()
    Button.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
end)
Button.MouseLeave:Connect(function()
    Button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
end)

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

    -- criar abas
    local Tabs = {
        Main = Window:AddTab({ Title = "Main" }),
        Settings = Window:AddTab({ Title = "Settings", Icon = "settings" }),
        Extras = Window:AddTab({ Title = "Extras" }),
        Farm = Window:AddTab({ Title = "Farm" }),
        Visual = Window:AddTab({ Title = "Visual" }),
        Player = Window:AddTab({ Title = "Player" }),
        Combat = Window:AddTab({ Title = "Combat" }),
        Fun = Window:AddTab({ Title = "Fun" }),
        Utility = Window:AddTab({ Title = "Utility" })
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

    ---------------------------
    -- MAIN TAB
    ---------------------------
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

    local JumpSlider = Tabs.Main:AddSlider({
        Title = "Ajustar Pulo",
        Description = "Muda a altura do pulo",
        Default = 50,
        Min = 0,
        Max = 200,
        Callback = function(val)
            local humanoid = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then humanoid.JumpPower = val end
        end
    })

    local WalkSlider = Tabs.Main:AddSlider({
        Title = "Ajustar Velocidade",
        Description = "Muda a velocidade do jogador",
        Default = 16,
        Min = 10,
        Max = 200,
        Callback = function(val)
            local humanoid = Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then humanoid.WalkSpeed = val end
        end
    })

    local noclipConnection
    Tabs.Main:AddToggle({
        Title = "NoClip",
        Description = "Permite atravessar paredes",
        Default = false,
        Callback = function(state)
            if state then
                noclipConnection = RunService.Stepped:Connect(function()
                    local char = Players.LocalPlayer.Character
                    if char then
                        for _, part in pairs(char:GetDescendants()) do
                            if part:IsA("BasePart") then part.CanCollide = false end
                        end
                    end
                end)
            else
                if noclipConnection then noclipConnection:Disconnect() noclipConnection = nil end
            end
        end
    })

    ---------------------------
    -- COMBAT TAB
    ---------------------------
    local aimbotEnabled = false
    Tabs.Combat:AddToggle({
        Title = "Aimbot",
        Default = false,
        Callback = function(state) aimbotEnabled = state end
    })

    local autoClick = false
    Tabs.Combat:AddToggle({
        Title = "Auto Click",
        Default = false,
        Callback = function(state) autoClick = state end
    })

    local killAura = false
    Tabs.Combat:AddToggle({
        Title = "Kill Aura",
        Default = false,
        Callback = function(state) killAura = state end
    })

    -- Aimbot e Auto Click loop
    task.spawn(function()
        local vu = game:GetService("VirtualUser")
        RunService.RenderStepped:Connect(function()
            -- Aimbot
            if aimbotEnabled then
                local cam = workspace.CurrentCamera
                local closest, dist = nil, math.huge
                for _, plr in pairs(Players:GetPlayers()) do
                    if plr ~= Players.LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
                        local pos, vis = cam:WorldToViewportPoint(plr.Character.Head.Position)
                        if vis then
                            local mag = (Vector2.new(pos.X,pos.Y) - UIS:GetMouseLocation()).Magnitude
                            if mag < dist then closest, dist = plr, mag end
                        end
                    end
                end
                if closest and closest.Character and closest.Character:FindFirstChild("Head") then
                    cam.CFrame = CFrame.new(cam.CFrame.Position, closest.Character.Head.Position)
                end
            end

            -- Kill Aura
            if killAura then
                local char = Players.LocalPlayer.Character
                if char and char:FindFirstChild("HumanoidRootPart") then
                    for _, npc in pairs(workspace:GetChildren()) do
                        if npc:IsA("Model") and npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") then
                            local dist = (char.HumanoidRootPart.Position - npc.HumanoidRootPart.Position).Magnitude
                            if dist < 10 then npc.Humanoid.Health = 0 end
                        end
                    end
                end
            end

            -- Auto Click
            if autoClick then
                pcall(function()
                    vu:Button1Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
                    task.wait()
                    vu:Button1Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
                end)
            end
        end)
    end)

    ---------------------------
    -- FUN TAB
    ---------------------------
    local rainbow = false
    Tabs.Fun:AddToggle({
        Title = "Rainbow Character",
        Default = false,
        Callback = function(state) rainbow = state end
    })

    local spinning = false
    Tabs.Fun:AddToggle({
        Title = "Spin Character",
        Default = false,
        Callback = function(state) spinning = state end
    })

    Tabs.Fun:AddToggle({
        Title = "Big Head",
        Default = false,
        Callback = function(state)
            for _, plr in pairs(Players:GetPlayers()) do
                if plr.Character and plr.Character:FindFirstChild("Head") then
                    local mesh = plr.Character.Head:FindFirstChildOfClass("SpecialMesh")
                    if mesh then mesh.Scale = state and Vector3.new(5,5,5) or Vector3.new(1,1,1) end
                end
            end
        end
    })

    Tabs.Fun:AddToggle({
        Title = "Low Gravity",
        Default = false,
        Callback = function(state) workspace.Gravity = state and 50 or 196.2 end
    })

    Tabs.Fun:AddButton({
        Title = "Fire Feet",
        Callback = function()
            local char = Players.LocalPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                local fire = Instance.new("Fire", char.HumanoidRootPart)
                fire.Heat, fire.Size = 10, 5
            end
        end
    })

    -- Rainbow e Spin loop
    task.spawn(function()
        while task.wait(0.2) do
            local char = Players.LocalPlayer.Character
            if char then
                -- Rainbow
                if rainbow then
                    for _, part in pairs(char:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.Color = Color3.fromHSV(tick()%5/5,1,1)
                        end
                    end
                end
                -- Spin
                if spinning and char:FindFirstChild("HumanoidRootPart") then
                    char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame * CFrame.Angles(0, math.rad(10), 0)
                end
            end
        end
    end)

    ---------------------------
    -- PLAYER TAB
    ---------------------------
    Tabs.Player:AddDropdown({
        Title = "Teleport To Player",
        Description = "Teleporta até outro jogador",
        Options = function()
            local list = {}
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= Players.LocalPlayer then table.insert(list, plr.Name) end
            end
            return list
        end,
        Callback = function(selected)
            local target = Players:FindFirstChild(selected)
            if target and target.Character and Players.LocalPlayer.Character then
                Players.LocalPlayer.Character:MoveTo(target.Character.HumanoidRootPart.Position)
            end
        end
    })

    Tabs.Player:AddToggle({
        Title = "Invisibility",
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
        end
    })

    Tabs.Player:AddButton({
        Title = "Reset Player",
        Callback = function()
            if Players.LocalPlayer.Character then Players.LocalPlayer.Character:BreakJoints() end
        end
    })

    ---------------------------
    -- EXTRAS TAB
    ---------------------------
    local flying = false
    local flyConnection
    Tabs.Extras:AddToggle({
        Title = "Fly Mode",
        Default = false,
        Callback = function(state)
            local char = Players.LocalPlayer.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local speed = 50
            if state and hrp then
                flying = true
                flyConnection = RunService.RenderStepped:Connect(function()
                    if flying then
                        local moveDir = Vector3.zero
                        if UIS:IsKeyDown(Enum.KeyCode.W) then moveDir += workspace.CurrentCamera.CFrame.LookVector end
                        if UIS:IsKeyDown(Enum.KeyCode.S) then moveDir -= workspace.CurrentCamera.CFrame.LookVector end
                        if UIS:IsKeyDown(Enum.KeyCode.A) then moveDir -= workspace.CurrentCamera.CFrame.RightVector end
                        if UIS:IsKeyDown(Enum.KeyCode.D) then moveDir += workspace.CurrentCamera.CFrame.RightVector end
                        if UIS:IsKeyDown(Enum.KeyCode.Space) then moveDir += Vector3.new(0,1,0) end
                        if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir -= Vector3.new(0,1,0) end
                        hrp.Velocity = moveDir * speed
                    end
                end)
            else
                flying = false
                if flyConnection then flyConnection:Disconnect() flyConnection = nil end
                if hrp then hrp.Velocity = Vector3.zero end
            end
        end
    })

    ---------------------------
    -- FARM TAB
    ---------------------------
    Tabs.Farm:AddToggle({
        Title = "Auto Buy/Store",
        Default = false,
        Callback = function(state)
            -- aqui vai o código do Auto Buy/Store
        end
    })

    ---------------------------
    -- VISUAL TAB
    ---------------------------
    Tabs.Visual:AddToggle({
        Title = "ESP Players/NPCs",
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
                    if plr ~= Players.LocalPlayer then CreateESP(plr.Character or plr.CharacterAdded:Wait()) end
                end
                for _, npc in pairs(workspace:GetChildren()) do
                    if npc:IsA("Model") and npc:FindFirstChild("Humanoid") then CreateESP(npc) end
                end
            else
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("Highlight") then v:Destroy() end
                end
            end
        end
    })

    ---------------------------
    -- SETTINGS TAB
    ---------------------------
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
            end
        end
    })

    Tabs.Settings:AddToggle({
        Title = "Anti-Stun",
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

    ---------------------------
    -- UTILITY TAB
    ---------------------------
    Tabs.Utility:AddButton({
        Title = "Anti AFK",
        Callback = function()
            local vu = game:GetService("VirtualUser")
            Players.LocalPlayer.Idled:Connect(function()
                vu:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
                task.wait(1)
                vu:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
            end)
        end
    })

    Tabs.Utility:AddButton({
        Title = "Rejoin",
        Callback = function()
            game:GetService("TeleportService"):Teleport(game.PlaceId, Players.LocalPlayer)
        end
    })

    Tabs.Utility:AddButton({
        Title = "Clear ESP",
        Callback = function()
            for _, v in pairs(workspace:GetDescendants()) do
                if v:IsA("Highlight") then v:Destroy() end
            end
        end
    })

end -- fim do LoadHub

-- botão flutuante recarrega o hub
Button.MouseButton1Click:Connect(function() LoadHub() end)

-- carrega o hub pela primeira vez
LoadHub()
