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
        Title = "caveira hub" .. tostring(Fluent.Version),
        TabWidth = 160, 
        Size = UDim2.fromOffset(580, 460), 
        Theme = "Dark"
    })

    local Tabs = {
        Main = Window:AddTab({ Title = "Main" }),
        Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
    }

    local UIS = game:GetService("UserInputService")
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")

    -- hotkey RightShift para abrir/fechar
    UIS.InputBegan:Connect(function(input, gp)
        if not gp and input.KeyCode == Enum.KeyCode.RightShift then
            Window:Toggle()
        end
    end)

    -- parágrafo
    Tabs.Main:AddParagraph({ Title = "caveira hub script", Content = "script novo do caveira " })

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

    -- AutoFarm toggle
    Tabs.Main:AddToggle("autoFarm", {
        Title = "AutoFarm",
        Description = "Ele vai farmar level",
        Default = false,
        Callback = function(state)
            if state then
                Fluent:Notify({ Title = "AutoFarm ligado", Content = "Ativo" })
                -- aqui entra seu código de autofarm
            else
                Fluent:Notify({ Title = "AutoFarm desligado", Content = "Desativado" })
            end
        end
    })

    -- Sliders
    -- JumpPower
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

    -- WalkSpeed
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

    -- Fly toggle
    local flying = false
    local flyConnection
    Tabs.Main:AddToggle("fly", {
        Title = "Fly",
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
end

-- botão flutuante recarrega o hub
Button.MouseButton1Click:Connect(function()
    LoadHub()
end)

-- carrega o hub pela primeira vez
LoadHub()
