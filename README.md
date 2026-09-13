

-- Servidores y servicios requeridos
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")

-- Limpiar interfaz anterior si existe
if CoreGui:FindFirstChild("ChinoxFontUI") then
    CoreGui.ChinoxFontUI:Destroy()
end

-- Crear ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ChinoxFontUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui

-- Crear botón principal
local FontButton = Instance.new("TextButton")
FontButton.Name = "FontButton"
FontButton.Size = UDim2.new(0, 140, 0, 45)
FontButton.Position = UDim2.new(0.5, -70, 0.45, -22)
FontButton.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
FontButton.Text = "Font"
FontButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FontButton.TextSize = 18
FontButton.Font = Enum.Font.GothamBold
FontButton.AutoButtonColor = false
FontButton.ClipsDescendants = true
FontButton.Parent = ScreenGui

-- Esquinas redondeadas
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = FontButton

-- Borde fino con degradado elegante
local UIStroke = Instance.new("UIStroke")
UIStroke.Thickness = 1.5
UIStroke.Color = Color3.fromRGB(60, 60, 75)
UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
UIStroke.Parent = FontButton

-- Efecto Hover / Click
FontButton.MouseEnter:Connect(function()
    TweenService:Create(FontButton, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(35, 35, 45)}):Play()
    TweenService:Create(UIStroke, TweenInfo.new(0.2), {Color = Color3.fromRGB(90, 90, 120)}):Play()
end)

FontButton.MouseLeave:Connect(function()
    TweenService:Create(FontButton, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(25, 25, 30)}):Play()
    TweenService:Create(UIStroke, TweenInfo.new(0.2), {Color = Color3.fromRGB(60, 60, 75)}):Play()
end)

-- Sistema Draggable (Arrastrar y Soltar)
local dragging, dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    FontButton.Position = UDim2.new(
        startPos.X.Scale,
        startPos.X.Offset + delta.X,
        startPos.Y.Scale,
        startPos.Y.Offset + delta.Y
    )
end

FontButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = FontButton.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

FontButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- Función ejecutable de la fuente
local function executeFontScript()
    if not isfile("starborn.ttf") then
        writefile("starborn.ttf", game:HttpGet("https://granny.anondrop.net/uploads/6c2505542959f371/Starborn.ttf"))
    end

    writefile("starborn.json", HttpService:JSONEncode({
        name = "Starborn",
        faces = {{name = "Regular", weight = 400, style = "normal", assetId = getcustomasset("starborn.ttf")}}
    }))

    local myfont = Font.new(getcustomasset("starborn.json"))
    local badfont = tostring(Font.new("rbxasset://LuaPackages/Packages/_Index/BuilderIcons/BuilderIcons/BuilderIcons.json"))

    local function donttouch(this)
        if this.TextStrokeTransparency ~= 1 then return false end
        local cur = tostring(this.FontFace)
        return cur == badfont or string.find(cur, "BuilderIcons")
    end

    local function changeit(txt)
        if txt:IsA("TextLabel") or txt:IsA("TextButton") or txt:IsA("TextBox") then
            if not donttouch(txt) then
                txt.FontFace = myfont
            end
        end
    end

    for _, v in pairs(game:GetDescendants()) do
        task.spawn(function() changeit(v) end)
    end

    game.DescendantAdded:Connect(function(obj)
        task.spawn(function() changeit(obj) end)
    end)
end

-- Ejecución al presionar el botón
local executed = false
FontButton.MouseButton1Click:Connect(function()
    if not executed then
        executed = true
        FontButton.Text = "Loading..."
        
        local success, err = pcall(executeFontScript)
        
        if success then
            FontButton.Text = "Font Loaded!"
            TweenService:Create(UIStroke, TweenInfo.new(0.3), {Color = Color3.fromRGB(80, 200, 120)}):Play()
        else
            FontButton.Text = "Error"
            warn("Font Error:", err)
            task.wait(1.5)
            FontButton.Text = "Font"
            executed = false
        end
    end
end)
