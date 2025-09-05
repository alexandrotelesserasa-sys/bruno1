-- Superscript.br - Fly, Corrida, Super Pulo, Chão Invisível, Fog, Velocidade customizável
local plr = game.Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")
local runService = game:GetService("RunService")
local lighting = game:GetService("Lighting")
local terrain = workspace.Terrain

-- GUI principal
local gui = Instance.new("ScreenGui", plr.PlayerGui)
gui.Name = "superscriptGUI"
gui.ResetOnSpawn = false

-- Título no canto superior esquerdo
local titulo = Instance.new("TextLabel", gui)
titulo.Text = "superscript.br"
titulo.Position = UDim2.new(0, 10, 0, 10)
titulo.Size = UDim2.new(0, 140, 0, 25)
titulo.BackgroundTransparency = 1
titulo.TextColor3 = Color3.fromRGB(200, 200, 200)
titulo.Font = Enum.Font.GothamSemibold
titulo.TextSize = 14
titulo.TextXAlignment = Enum.TextXAlignment.Left

-- Janela principal
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 210)
frame.Position = UDim2.new(0, 10, 0, 50)
frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 8)

-- Criar botão padrão
local function criarBotao(texto, y, cor)
	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(1, -20, 0, 30)
	btn.Position = UDim2.new(0, 10, 0, y)
	btn.BackgroundColor3 = cor or Color3.fromRGB(60, 120, 255)
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Font = Enum.Font.SourceSansBold
	btn.TextSize = 14
	btn.Text = texto
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
	return btn
end

-- Botões
local btnFlyCam = criarBotao("Fly Cam: ON", 10)
local btnFog = criarBotao("Remover Fog", 50)
local btnVelocidade = criarBotao("Velocidade: ON", 90)
local btnFechar = criarBotao("Fechar", 170, Color3.fromRGB(200, 50, 50))

-- Dropdown de seleção de velocidade
local velocidadeSelecionada = 300
local dropdown = Instance.new("TextButton", frame)
dropdown.Size = UDim2.new(1, -20, 0, 30)
dropdown.Position = UDim2.new(0, 10, 0, 130)
dropdown.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
dropdown.TextColor3 = Color3.new(1,1,1)
dropdown.Font = Enum.Font.SourceSansBold
dropdown.TextSize = 14
dropdown.Text = "Velocidade: 300"
Instance.new("UICorner", dropdown).CornerRadius = UDim.new(0, 6)

local opcoes = {200, 250, 300, 350}
local index = 3

dropdown.MouseButton1Click:Connect(function()
	index += 1
	if index > #opcoes then index = 1 end
	velocidadeSelecionada = opcoes[index]
	dropdown.Text = "Velocidade: " .. velocidadeSelecionada
end)

-- Ativar velocidade
local velocidadeAtiva = true

-- Corrida automática (com controle)
task.spawn(function()
	while task.wait(0.03) do
		if velocidadeAtiva and humanoid.MoveDirection.Magnitude > 0 and not humanoid.PlatformStand then
			local move = humanoid.MoveDirection
			hrp.CFrame = CFrame.new(hrp.Position + move * velocidadeSelecionada * 0.03)
		end
	end
end)

btnVelocidade.MouseButton1Click:Connect(function()
	velocidadeAtiva = not velocidadeAtiva
	btnVelocidade.Text = velocidadeAtiva and "Velocidade: ON" or "Velocidade: OFF"
	btnVelocidade.BackgroundColor3 = velocidadeAtiva and Color3.fromRGB(60,120,255) or Color3.fromRGB(80,80,80)
end)

-- Fly com câmera
local flyCam = true
local gyro, velo, con

local function ativarFlyCam()
	if flyCam then
		gyro = Instance.new("BodyGyro", hrp)
		gyro.P = 9e4
		gyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
		gyro.CFrame = hrp.CFrame

		velo = Instance.new("BodyVelocity", hrp)
		velo.MaxForce = Vector3.new(9e9, 9e9, 9e9)
		velo.Velocity = Vector3.zero

		humanoid.PlatformStand = true

		con = runService.RenderStepped:Connect(function()
			local cam = workspace.CurrentCamera
			local dir = humanoid.MoveDirection
			if dir.Magnitude > 0 then
				velo.Velocity = cam.CFrame.LookVector.Unit * velocidadeSelecionada * dir.Magnitude
			else
				velo.Velocity = Vector3.zero
			end
			gyro.CFrame = cam.CFrame
		end)
	end
end

local function desativarFlyCam()
	flyCam = false
	humanoid.PlatformStand = false
	if gyro then gyro:Destroy() end
	if velo then velo:Destroy() end
	if con then con:Disconnect() end
end

btnFlyCam.MouseButton1Click:Connect(function()
	if flyCam then
		desativarFlyCam()
		btnFlyCam.Text = "Fly Cam: OFF"
		btnFlyCam.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
	else
		flyCam = true
		ativarFlyCam()
		btnFlyCam.Text = "Fly Cam: ON"
		btnFlyCam.BackgroundColor3 = Color3.fromRGB(60, 120, 255)
	end
end)
ativarFlyCam()

-- Remover fog
btnFog.MouseButton1Click:Connect(function()
	lighting.FogStart = 999999
	lighting.FogEnd = 999999
	local atm = lighting:FindFirstChildOfClass("Atmosphere")
	if atm then atm:Destroy() end
	btnFog.Text = "Fog Removido"
	btnFog.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
	btnFog.Active = false
end)

-- Super pulo
humanoid.JumpPower = 300
humanoid.StateChanged:Connect(function(state)
	if state == Enum.HumanoidStateType.Jumping then
		humanoid.JumpPower = 300
	end
end)

-- Chão invisível sobre água (automático)
task.spawn(function()
	local resolucao = 4
	local area = Region3.new(Vector3.new(-2048, -10, -2048), Vector3.new(2048, 100, 2048)):ExpandToGrid(resolucao)
	local materiais = terrain:ReadVoxels(area, resolucao)

	local tamanho = resolucao
	local base = area.CFrame.Position - (area.Size / 2)

	for x = 1, materiais.Size.X do
		for y = 1, materiais.Size.Y do
			for z = 1, materiais.Size.Z do
				if materiais[x][y][z] == Enum.Material.Water then
					local pos = base + Vector3.new((x - 1) * tamanho, (y - 1) * tamanho, (z - 1) * tamanho)
					local parte = Instance.new("Part")
					parte.Size = Vector3.new(tamanho, 1, tamanho)
					parte.Position = Vector3.new(pos.X, pos.Y + 2, pos.Z)
					parte.Anchored = true
					parte.CanCollide = true
					parte.Transparency = 1
					parte.Name = "ChaoInvisivelAgua"
					parte.Parent = workspace
				end
			end
		end
	end
end)

-- Fechar interface
btnFechar.MouseButton1Click:Connect(function()
	gui:Destroy()
	for _, obj in pairs(workspace:GetChildren()) do
		if obj.Name == "ChaoInvisivelAgua" then
			obj:Destroy()
		end
	end
end)

-- Super pulo ao renascer
plr.CharacterAdded:Connect(function(novoChar)
	char = novoChar
	humanoid = char:WaitForChild("Humanoid")
	hrp = char:WaitForChild("HumanoidRootPart")
	humanoid.JumpPower = 300
end)
