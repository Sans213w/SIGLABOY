-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "TPGui"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 220, 0, 200)
Frame.Position = UDim2.new(0, 20, 0.5, -100)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true

Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 8)

local function createButton(text, yPos)
	local button = Instance.new("TextButton", Frame)
	button.Size = UDim2.new(1, -20, 0, 30)
	button.Position = UDim2.new(0, 10, 0, yPos)
	button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	button.TextColor3 = Color3.new(1, 1, 1)
	button.Font = Enum.Font.SourceSansBold
	button.TextSize = 18
	button.Text = text
	Instance.new("UICorner", button).CornerRadius = UDim.new(0, 6)
	return button
end

local espButton = createButton("Ativar ESP", 10)
local clickTPButton = createButton("TP com Clique", 45)
local nearestTPButton = createButton("TP mais próximo", 80)
local aimbotButton = createButton("Aimbot com ESP", 115)
local getGunButton = createButton("Pegar Arma", 150)

-- ESP
local espEnabled = false
local espList = {}

local function createESP(player)
	if player == game.Players.LocalPlayer then return end
	if not player.Character then return end
	if espList[player] then return end

	local esp = Instance.new("Highlight")
	esp.FillColor = Color3.fromRGB(255, 0, 0)
	esp.OutlineColor = Color3.fromRGB(255, 255, 255)
	esp.FillTransparency = 0.5
	esp.OutlineTransparency = 0
	esp.Adornee = player.Character
	esp.Parent = player.Character
	espList[player] = esp
end

local function removeAllESP()
	for _, esp in pairs(espList) do
		if esp then esp:Destroy() end
	end
	espList = {}
end

espButton.MouseButton1Click:Connect(function()
	espEnabled = not espEnabled
	if espEnabled then
		for _, player in ipairs(game.Players:GetPlayers()) do
			createESP(player)
		end
		espButton.Text = "ESP [ON]"
	else
		removeAllESP()
		espButton.Text = "Ativar ESP"
	end
end)

game.Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function()
		wait(1)
		if espEnabled then
			createESP(player)
		end
	end)
end)

-- TP com clique (10 studs à frente do ponto clicado)
local clickTPEnabled = false
clickTPButton.MouseButton1Click:Connect(function()
	clickTPEnabled = not clickTPEnabled
	clickTPButton.Text = clickTPEnabled and "TP com Clique [ON]" or "TP com Clique"

	if clickTPEnabled then
		local mouse = game.Players.LocalPlayer:GetMouse()
		mouse.Button1Down:Connect(function()
			if not clickTPEnabled then return end
			if mouse.Target then
				local targetPos = mouse.Hit.Position + (mouse.Hit.LookVector.Unit * 10)
				game.Players.LocalPlayer.Character:MoveTo(targetPos + Vector3.new(0, 3, 0))
			end
		end)
	end
end)

-- TP para jogador mais próximo
nearestTPButton.MouseButton1Click:Connect(function()
	local lp = game.Players.LocalPlayer
	local closest, minDist = nil, math.huge
	local myPos = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") and lp.Character.HumanoidRootPart.Position
	if not myPos then return end

	for _, p in pairs(game.Players:GetPlayers()) do
		if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			local dist = (p.Character.HumanoidRootPart.Position - myPos).Magnitude
			if dist < minDist then
				minDist = dist
				closest = p
			end
		end
	end

	if closest and closest.Character then
		lp.Character:MoveTo(closest.Character.HumanoidRootPart.Position + Vector3.new(0, 3, 0))
	end
end)

-- Aimbot com ESP
local aimbotEnabled = false
aimbotButton.MouseButton1Click:Connect(function()
	aimbotEnabled = not aimbotEnabled
	aimbotButton.Text = aimbotEnabled and "Aimbot [ON]" or "Aimbot com ESP"

	local player = game.Players.LocalPlayer
	local mouse = player:GetMouse()

	game:GetService("RunService").RenderStepped:Connect(function()
		if not aimbotEnabled then return end

		local closest, dist = nil, math.huge
		for _, p in pairs(game.Players:GetPlayers()) do
			if p ~= player and p.Character and p.Character:FindFirstChild("Head") then
				local screenPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(p.Character.Head.Position)
				if onScreen then
					local mag = (Vector2.new(mouse.X, mouse.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
					if mag < dist then
						dist = mag
						closest = p
					end
				end
			end
		end

		if closest and closest.Character and closest.Character:FindFirstChild("Head") then
			workspace.CurrentCamera.CFrame = CFrame.new(
				workspace.CurrentCamera.CFrame.Position,
				closest.Character.Head.Position
			)
		end
	end)
end)

-- Arma com dano
getGunButton.MouseButton1Click:Connect(function()
	local player = game.Players.LocalPlayer
	if player.Backpack:FindFirstChild("Blaster") then return end

	local tool = Instance.new("Tool")
	tool.RequiresHandle = false
	tool.Name = "Blaster"

	tool.Activated:Connect(function()
		local character = player.Character
		if not character then return end

		local hrp = character:FindFirstChild("HumanoidRootPart")
		local mouse = player:GetMouse()

		if hrp and mouse then
			local direction = (mouse.Hit.Position - hrp.Position).Unit

			local bullet = Instance.new("Part")
			bullet.Shape = Enum.PartType.Ball
			bullet.Size = Vector3.new(0.5, 0.5, 0.5)
			bullet.BrickColor = BrickColor.new("Bright red")
			bullet.Material = Enum.Material.Neon
			bullet.CanCollide = false
			bullet.Anchored = false
			bullet.CFrame = hrp.CFrame + hrp.CFrame.LookVector * 2
			bullet.Velocity = direction * 100
			bullet.Parent = workspace

			bullet.Touched:Connect(function(hit)
				local hum = hit.Parent and hit.Parent:FindFirstChildOfClass("Humanoid")
				if hum and hit.Parent ~= character then
					hum:TakeDamage(20)
					bullet:Destroy()
				end
			end)

			game.Debris:AddItem(bullet, 5)
		end
	end)

	tool.Parent = player.Backpack
end)
