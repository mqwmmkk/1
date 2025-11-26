-- UH卡密验证+飞行修复+防坠落+面板改名版
local lp=game.Players.LocalPlayer
local char=lp.Character or lp.CharacterAdded:Wait()
local hum=char:WaitForChild("Humanoid")
local root=char:WaitForChild("HumanoidRootPart")
local state={fly=false,god=false,speed=25,panelOpen=false, verified=false, noFall=false}
local CORRECT_KEY="UH2025"
local flySpeed=60

-- 权限初始化
hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
hum:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
hum:SetStateEnabled(Enum.HumanoidStateType.Running, false)
hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)

-- 角色加载监听
lp.CharacterAdded:Connect(function(newChar)
    char=newChar
    hum=char:WaitForChild("Humanoid")
    root=char:WaitForChild("HumanoidRootPart")
    state.fly=false
    state.noFall=false
    hum.PlatformStand=false
    hum.AutoRotate=true
    hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
    hum:SetStateEnabled(Enum.HumanoidStateType.Jumping, false)
    hum:SetStateEnabled(Enum.HumanoidStateType.Running, false)
    hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)
    if flyBtn then flyBtn.Text="飞行 [关]" end
    if noFallBtn then noFallBtn.Text="关闭坠落 [关]" end
end)

-- 1. 卡密验证弹窗（UI优化）
local ui=Instance.new("ScreenGui",lp.PlayerGui)
ui.Name="UH_Panel"
local keyPopup=Instance.new("Frame",ui)
keyPopup.Size=UDim2.new(0,280,0,150)
keyPopup.Position=UDim2.new(0.5,-140,0.5,-75)
keyPopup.BackgroundColor3=Color3.new(0,0,0)
keyPopup.BackgroundTransparency=0.3
keyPopup.BorderSizePixel=2
keyPopup.BorderColor3=Color3.new(0.4,0.4,0.8)
keyPopup.Visible=true

local popupTitle=Instance.new("TextLabel",keyPopup)
popupTitle.Size=UDim2.new(1,0,0,30)
popupTitle.BackgroundColor3=Color3.new(0.2,0.2,0.4)
popupTitle.Text="UH面板 - 卡密验证"
popupTitle.TextColor3=Color3.new(1,1,1)
popupTitle.TextSize=16
popupTitle.Font=Enum.Font.SourceSansBold

local keyInput=Instance.new("TextBox",keyPopup)
keyInput.Size=UDim2.new(0.8,0,0,35)
keyInput.Position=UDim2.new(0.1,0,0,40)
keyInput.BackgroundColor3=Color3.new(0.15,0.15,0.15)
keyInput.TextColor3=Color3.new(1,1,1)
keyInput.PlaceholderText="请输入卡密（UH2025）"
keyInput.TextSize=14
keyInput.BorderSizePixel=1
keyInput.BorderColor3=Color3.new(0.4,0.4,0.8)

local verifyBtn=Instance.new("TextButton",keyPopup)
verifyBtn.Size=UDim2.new(0.4,0,0,30)
verifyBtn.Position=UDim2.new(0.3,0,0,90)
verifyBtn.BackgroundColor3=Color3.new(0,0.7,0)
verifyBtn.Text="验证"
verifyBtn.TextColor3=Color3.new(1,1,1)
verifyBtn.TextSize=16
verifyBtn.Font=Enum.Font.SourceSansBold
verifyBtn.BorderSizePixel=1
verifyBtn.BorderColor3=Color3.new(0,0.5,0)

local tipLabel=Instance.new("TextLabel",keyPopup)
tipLabel.Size=UDim2.new(1,0,0,20)
tipLabel.Position=UDim2.new(0,0,0,125)
tipLabel.BackgroundTransparency=1
tipLabel.Text="卡密错误将无法使用功能"
tipLabel.TextColor3=Color3.new(0.8,0.8,0.8)
tipLabel.TextSize=12
tipLabel.TextXAlignment=Enum.TextXAlignment.Center

-- 2. 可拖动UH悬浮窗（UI优化）
local floatBtn=Instance.new("Frame",ui)
floatBtn.Size=UDim2.new(0,220,0,40)
floatBtn.Position=UDim2.new(0.5,-110,0.02,0)
floatBtn.BackgroundColor3=Color3.new(0,0,0)
floatBtn.BackgroundTransparency=0.3
floatBtn.BorderSizePixel=2
floatBtn.BorderColor3=Color3.new(0.4,0.4,0.8)
floatBtn.Active=true
floatBtn.Visible=false
local isDragging=false
local dragStartPos,floatStartPos

-- 悬浮窗拖动逻辑
floatBtn.InputBegan:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.Touch or input.UserInputType==Enum.UserInputType.MouseButton1 then
        isDragging=true
        dragStartPos=input.Position
        floatStartPos=floatBtn.Position
        floatBtn.BackgroundTransparency=0.5
    end
end)

floatBtn.InputChanged:Connect(function(input)
    if isDragging and (input.UserInputType==Enum.UserInputType.Touch or input.UserInputType==Enum.UserInputType.MouseMovement) then
        local delta=input.Position-dragStartPos
        floatBtn.Position=UDim2.new(
            floatStartPos.X.Scale,
            floatStartPos.X.Offset+delta.X,
            floatStartPos.Y.Scale,
            floatStartPos.Y.Offset+delta.Y
        )
    end
end)

floatBtn.InputEnded:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.Touch or input.UserInputType==Enum.UserInputType.MouseButton1 then
        isDragging=false
        floatBtn.BackgroundTransparency=0.3
    end
end)

-- 悬浮窗内容（优化排版）
local icon1=Instance.new("TextLabel",floatBtn)
icon1.Size=UDim2.new(0,40,1,0)
icon1.BackgroundTransparency=1
icon1.Text="🔮"
icon1.TextColor3=Color3.new(1,1,1)
icon1.TextSize=22
icon1.TextXAlignment=Enum.TextXAlignment.Center

local icon2=Instance.new("TextLabel",floatBtn)
icon2.Size=UDim2.new(0,40,1,0)
icon2.Position=UDim2.new(0.18,0,0,0)
icon2.BackgroundTransparency=1
icon2.Text="∞"
icon2.TextColor3=Color3.new(1,1,1)
icon2.TextSize=22
icon2.TextXAlignment=Enum.TextXAlignment.Center

local nameLabel=Instance.new("TextLabel",floatBtn)
nameLabel.Size=UDim2.new(0.25,0,1,0)
nameLabel.Position=UDim2.new(0.36,0,0,0)
nameLabel.BackgroundTransparency=1
nameLabel.Text="UH"
nameLabel.TextColor3=Color3.new(1,1,1)
nameLabel.TextSize=20
nameLabel.TextXAlignment=Enum.TextXAlignment.Center
nameLabel.Font=Enum.Font.SourceSansBold

-- 开/关按钮（优化样式）
local openBtn=Instance.new("TextButton",floatBtn)
openBtn.Size=UDim2.new(0.18,0,1,0)
openBtn.Position=UDim2.new(0.62,0,0,0)
openBtn.BackgroundColor3=Color3.new(0,0.7,0)
openBtn.Text="开"
openBtn.TextColor3=Color3.new(1,1,1)
openBtn.TextSize=18
openBtn.Font=Enum.Font.SourceSansBold
openBtn.BorderSizePixel=1
openBtn.BorderColor3=Color3.new(0,0.5,0)

local closeBtn=Instance.new("TextButton",floatBtn)
closeBtn.Size=UDim2.new(0.18,0,1,0)
closeBtn.Position=UDim2.new(0.8,0,0,0)
closeBtn.BackgroundColor3=Color3.new(0.7,0,0)
closeBtn.Text="关"
closeBtn.TextColor3=Color3.new(1,1,1)
closeBtn.TextSize=18
closeBtn.Font=Enum.Font.SourceSansBold
closeBtn.BorderSizePixel=1
closeBtn.BorderColor3=Color3.new(0.5,0,0)

-- 3. UI控制面板（改名+居中）
local panel=Instance.new("Frame",ui)
panel.Size=UDim2.new(0,220,0,300)
panel.Position=UDim2.new(0.5,-110,0.5,-150) -- 屏幕正中间
panel.BackgroundColor3=Color3.new(0.1,0.1,0.1)
panel.BackgroundTransparency=0.3
panel.BorderSizePixel=2
panel.BorderColor3=Color3.new(0.4,0.4,0.8)
panel.Visible=false
panel.ClipsDescendants=true

-- 面板标题栏（核心改名：我是优枪儿子）
local title=Instance.new("TextLabel",panel)
title.Size=UDim2.new(1,0,0,35)
title.BackgroundColor3=Color3.new(0.2,0.2,0.4)
title.Text="我是优枪儿子 v1.0" -- 面板名字修改完成
title.TextColor3=Color3.new(1,1,1)
title.TextSize=18
title.Font=Enum.Font.SourceSansBold
title.TextXAlignment=Enum.TextXAlignment.Center

-- 功能按钮（优化样式+交互）
local function btn(text,y,color)
    local b=Instance.new("TextButton",panel)
    b.Size=UDim2.new(0.9,0,0,40)
    b.Position=UDim2.new(0.05,0,0,y)
    b.Text=text
    b.TextColor3=Color3.new(1,1,1)
    b.BackgroundColor3=color or Color3.new(0.2,0.2,0.2)
    b.TextSize=16
    b.Font=Enum.Font.SourceSansBold
    b.BorderSizePixel=1
    b.BorderColor3=Color3.new(0.4,0.4,0.8)
    -- 按钮hover效果
    b.InputBegan:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.Touch or input.UserInputType==Enum.UserInputType.MouseMovement then
            b.BackgroundColor3=Color3.new(b.BackgroundColor3.R+0.1, b.BackgroundColor3.G+0.1, b.BackgroundColor3.B+0.1)
        end
    end)
    b.InputEnded:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.Touch or input.UserInputType==Enum.UserInputType.MouseMovement then
            b.BackgroundColor3=color or Color3.new(0.2,0.2,0.2)
        end
    end)
    return b
end

-- 飞行按钮（专属颜色）
local flyBtn=btn("飞行 [关]",45, Color3.new(0.2,0.3,0.4))
flyBtn.MouseButton1Click:Connect(function()
    if not state.verified then return end
    state.fly=not state.fly
    hum.PlatformStand=state.fly
    hum.AutoRotate=not state.fly
    hum.WalkSpeed=0
    hum.JumpPower=0
    if state.fly then
        root.CFrame=root.CFrame+Vector3.new(0,3,0)
        flyBtn.Text="飞行 [开]"
    else
        hum.WalkSpeed=state.speed
        hum.JumpPower=50
        hum.AutoRotate=true
        flyBtn.Text="飞行 [关]"
    end
end)

-- 无敌按钮（专属颜色）
local godBtn=btn("无敌 [关]",90, Color3.new(0.3,0.2,0.4))
godBtn.MouseButton1Click:Connect(function()
    if not state.verified then return end
    state.god=not state.god
    hum.MaxHealth=state.god and 99999 or 100
    hum.Health=state.god and 99999 or 100
    godBtn.Text="无敌 ["..(state.god and "开" or "关").."]"
end)

-- 关闭坠落按钮（专属颜色）
local noFallBtn=btn("关闭坠落 [关]",135, Color3.new(0.2,0.4,0.3))
noFallBtn.MouseButton1Click:Connect(function()
    if not state.verified then return end
    state.noFall=not state.noFall
    noFallBtn.Text="关闭坠落 ["..(state.noFall and "开" or "关").."]"
    hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, not state.noFall)
    hum.FloorMaterial=state.noFall and Enum.Material.Air or Enum.Material.Plastic
end)

-- 速度调节按钮（专属颜色）
local speedUp=btn("速度+ (当前:25)",180, Color3.new(0.4,0.3,0.2))
speedUp.MouseButton1Click:Connect(function()
    if not state.verified then return end
    state.speed=math.min(state.speed+10,100)
    flySpeed=state.speed*1.2
    if not state.fly then hum.WalkSpeed=state.speed end
    speedUp.Text="速度+ (当前:"..state.speed..")"
end)

local speedDown=btn("速度- (当前:25)",225, Color3.new(0.4,0.3,0.2))
speedDown.MouseButton1Click:Connect(function()
    if not state.verified then return end
    state.speed=math.max(state.speed-10,10)
    flySpeed=state.speed*1.2
    if not state.fly then hum.WalkSpeed=state.speed end
    speedDown.Text="速度- (当前:"..state.speed..")"
end)

-- 卡密验证逻辑
verifyBtn.MouseButton1Click:Connect(function()
    local inputKey=keyInput.Text
    if inputKey==CORRECT_KEY then
        state.verified=true
        keyPopup.Visible=false
        floatBtn.Visible=true
        tipLabel.Text="验证成功！"
        tipLabel.TextColor3=Color3.new(0,1,0)
    else
        tipLabel.Text="卡密错误，请重新输入！"
        tipLabel.TextColor3=Color3.new(1,0,0)
        keyInput.Text=""
    end
end)

-- 面板控制逻辑
openBtn.MouseButton1Click:Connect(function()
    if not state.verified then return end
    panel.Visible=true
end)

closeBtn.MouseButton1Click:Connect(function()
    panel.Visible=false
end)

-- 核心功能逻辑（稳定生效）
game.RunService.RenderStepped:Connect(function()
    if not state.verified then return end
    
    -- 飞行逻辑
    if state.fly then
        local dir=Vector3.new()
        local cam=workspace.CurrentCamera
        local input=game.UserInputService
        if input:IsKeyDown(Enum.KeyCode.W) then dir=dir+cam.CFrame.LookVector end
        if input:IsKeyDown(Enum.KeyCode.S) then dir=dir-cam.CFrame.LookVector end
        if input:IsKeyDown(Enum.KeyCode.A) then dir=dir-cam.CFrame.RightVector end
        if input:IsKeyDown(Enum.KeyCode.D) then dir=dir+cam.CFrame.RightVector end
        if input:IsKeyDown(Enum.KeyCode.Space) then dir.Y=1 end
        if input:IsKeyDown(Enum.KeyCode.LeftShift) then dir.Y=-1 end
        dir=dir.Unit
        root.Velocity=dir*flySpeed
    end
    
    -- 关闭坠落逻辑
    if state.noFall then
        hum.Health=hum.MaxHealth
        root.Velocity=Vector3.new(root.Velocity.X, math.max(root.Velocity.Y, -10), root.Velocity.Z)
    end
end)
