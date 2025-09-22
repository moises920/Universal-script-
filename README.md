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

    ------------------------------------------------
    -- MAIN TAB
    ------------------------------------------------
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

    local JumpSlider = Tabs.Main:AddSlider("pulo", {
        Title = "Ajustar Pulo",
        Description = "Muda a altura do pulo",
        Default = 51,
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

    local noclipConnection
    Tabs.Main:AddToggle("noclip", {
        Title = "NoClip",
        Description = "Permite atravessar paredes",
        Default = false,
        Callback = function(state)
            if state then
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
            end
        end
    })

    ------------------------------------------------
    -- COMBAT TAB
    ------------------------------------------------
    local aimbotEnabled = false
    Tabs.Combat:AddToggle("aimbot", {
        Title="Aimbot",
        Default=false,
        Callback=function(state) aimbotEnabled = state end
    })
    RunService.RenderStepped:Connect(function()
        if aimbotEnabled then
            local cam = workspace.CurrentCamera
            local closest, dist = nil, math.huge
            for _,plr in pairs(Players:GetPlayers()) do
                if plr ~= Players.LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
                    local pos, vis = cam:WorldToViewportPoint(plr.Character.Head.Position)
                    if vis then
                        local mag = (Vector2.new(pos.X,pos.Y) - UIS:GetMouseLocation()).Magnitude
                        if mag < dist then
                            closest, dist = plr, mag
                        end
                    end
                end
            end
            if closest and closest.Character and closest.Character:FindFirstChild("Head") then
                cam.CFrame = CFrame.new(cam.CFrame.Position, closest.Character.Head.Position)
            end
        end
    end)

    local autoClick = false
    Tabs.Combat:AddToggle("autoclick", {
        Title="Auto Click",
        Default=false,
        Callback=function(state) autoClick = state end
    })
    task.spawn(function()
        local vu = game:GetService("VirtualUser")
        while task.wait(0.1) do
            if autoClick then
                pcall(function()
                    vu:Button1Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
                    task.wait()
                    vu:Button1Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
                end)
            end
        end
    end)

    local killAura = false
    Tabs.Combat:AddToggle("killAura", {
        Title="Kill Aura",
        Default=false,
        Callback=function(state) killAura = state end
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

    ------------------------------------------------
    -- FUN TAB
    ------------------------------------------------
    local rainbow = false
    Tabs.Fun:AddToggle("rainbow", {
        Title="Rainbow Character",
        Default=false,
        Callback=function(state) rainbow = state end
    })
    task.spawn(function()
        while task.wait(0.2) do
            if rainbow then
                local char = Players.LocalPlayer.Character
                if char then
                    for _,part in pairs(char:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.Color = Color3.fromHSV(tick()%5/5,1,1)
                        end
                    end
                end
            end
        end
    end)

    local spinning = false
    Tabs.Fun:AddToggle("spin", {
        Title="Spin Character",
        Default=false,
        Callback=function(state) spinning = state end
    })
    RunService.RenderStepped:Connect(function()
        if spinning and Players.LocalPlayer.Character and Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = Players.LocalPlayer.Character.HumanoidRootPart
            hrp.CFrame = hrp.CFrame * CFrame.Angles(0,math.rad(10),0)
        end
    end)

    Tabs.Fun:AddToggle("bigHead", {
        Title="Big Head",
        Default=false,
        Callback=function(state)
            for _, plr in pairs(Players:GetPlayers()) do
                if plr.Character and plr.Character:FindFirstChild("Head") then
                    local mesh = plr.Character.Head:FindFirstChildOfClass("SpecialMesh")
                    if mesh then
                        mesh.Scale = state and Vector3.new(5,5,5) or Vector3.new(1,1,1)
                    end
                end
            end
        end
    })

    Tabs.Fun:AddToggle("lowGravity", {
        Title="Low Gravity",
        Default=false,
        Callback=function(state) workspace.Gravity = state and 50 or 196.2 end
    })

    Tabs.Fun:AddButton({Title="Fire Feet", Callback=function()
        local char = Players.LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local fire = Instance.new("Fire", char.HumanoidRootPart)
            fire.Heat, fire.Size = 10, 5
        end
    end})

    ------------------------------------------------
    -- UTILITY TAB
    ------------------------------------------------
    Tabs.Utility:AddButton({Title="Anti AFK", Callback=function()
        local vu = game:GetService("VirtualUser")
        Players.LocalPlayer.Idled:Connect(function()
            vu:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
            task.wait(1)
            vu:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
        end)
    end})

    Tabs.Utility:AddButton({Title="Rejoin", Callback=function()
        game:GetService("TeleportService"):Teleport(game.PlaceId, Players.LocalPlayer)
    end})

    Tabs.Utility:AddButton({Title="Clear ESP", Callback=function()
        for _, v in pairs(workspace:GetDescendants()) do
            if v:IsA("Highlight") then v:Destroy() end
        end
    end})

end -- fim do LoadHub

-- botão flutuante recarrega o hub
Button.MouseButton1Click:Connect(function() LoadHub() end)

-- carrega o hub pela primeira vez
LoadHub()
