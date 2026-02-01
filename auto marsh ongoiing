--==================================================
-- SERVICES
--==================================================
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local VIM = game:GetService("VirtualInputManager")
local RunService = game:GetService("RunService")
local ProximityPromptService = game:GetService("ProximityPromptService")

local player = Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")

print("player:"..player.Name)

--==================================================
-- GLOBAL STATE
--==================================================
local running = false
local deleteTool
local undoStack = {}
local instantInteract = false
local savedHold = {}

--==================================================
-- INPUT
--==================================================
local function pressE()
	VIM:SendKeyEvent(true, Enum.KeyCode.E, false, game)
	task.wait(0.05)
	VIM:SendKeyEvent(false, Enum.KeyCode.E, false, game)
end

--==================================================
-- INSTANT INTERACT (PRESS ONCE, NO HOLD)
--==================================================
ProximityPromptService.PromptShown:Connect(function(prompt)
	if not savedHold[prompt] then
		savedHold[prompt] = prompt.HoldDuration
	end
	if instantInteract then
		prompt.HoldDuration = 0
	else
		prompt.HoldDuration = savedHold[prompt]
	end
end)

ProximityPromptService.PromptHidden:Connect(function(prompt)
	if savedHold[prompt] then
		prompt.HoldDuration = savedHold[prompt]
	end
end)

--==================================================
-- INVENTORY
--==================================================
local function findTool(name)
	for _,t in ipairs(player.Backpack:GetChildren()) do
		if t:IsA("Tool") and t.Name:lower():find(name) then
			return t
		end
	end
end

local function equip(t)
	t.Parent = char
	task.wait(0.25)
end

local function unequip()
	hum:UnequipTools()
	task.wait(0.2)
end

--==================================================
-- UI
--==================================================
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "FARMHUB"

local main = Instance.new("Frame", gui)
main.Size = UDim2.fromScale(0.2,0.45)
main.Position = UDim2.fromScale(0.72,0.27)
main.BackgroundColor3 = Color3.fromRGB(25,25,25)
main.Active = true
main.Draggable = true

local function label(txt,y)
	local l = Instance.new("TextLabel", main)
	l.Size = UDim2.fromScale(1,0.07)
	l.Position = UDim2.fromScale(0,y)
	l.BackgroundTransparency = 1
	l.TextColor3 = Color3.new(1,1,1)
	l.Font = Enum.Font.GothamBold
	l.TextScaled = true
	l.Text = txt
	return l
end

label("FARMHUB",0)
local status = label("WAITING",0.07)

local function btn(t,y)
	local b = Instance.new("TextButton", main)
	b.Size = UDim2.fromScale(0.9,0.07)
	b.Position = UDim2.fromScale(0.05,y)
	b.BackgroundColor3 = Color3.fromRGB(60,60,60)
	b.TextColor3 = Color3.new(1,1,1)
	b.Font = Enum.Font.GothamBold
	b.TextScaled = true
	b.Text = t
	return b
end

local startBtn   = btn("START FARM",0.18)
local stopBtn    = btn("STOP FARM",0.26)
local addDel     = btn("ADD CLICK DELETE",0.34)
local remDel     = btn("REMOVE TOOL",0.42)
local tpVehBtn   = btn("TP TO VEH",0.50)
local tpDsBtn    = btn("TP TO DEALER",0.58)
local tpMarshBtn = btn("TP TO DEALER MARSH",0.66)
local instBtn    = btn("INSTANT INTERACT : OFF",0.74)

--==================================================
-- SHIFT + X (HAPUS UI SCRIPT)
--==================================================
UIS.InputBegan:Connect(function(i,gp)
	if gp then return end
	if i.KeyCode==Enum.KeyCode.X and UIS:IsKeyDown(Enum.KeyCode.LeftShift) then
		gui:Destroy()
	end
end)

--==================================================
-- CLICK TO DELETE + UNDO
--==================================================
addDel.MouseButton1Click:Connect(function()
	if deleteTool then return end
	deleteTool = Instance.new("Tool", player.Backpack)
	deleteTool.Name = "ClickToDelete"
	deleteTool.RequiresHandle = false
	deleteTool.Activated:Connect(function()
		local t = player:GetMouse().Target
		if t then
			table.insert(undoStack, t)
			t.Parent = nil
		end
	end)
end)

remDel.MouseButton1Click:Connect(function()
	if deleteTool then deleteTool:Destroy() deleteTool=nil end
end)

UIS.InputBegan:Connect(function(i,gp)
	if gp then return end
	if UIS:IsKeyDown(Enum.KeyCode.LeftControl) and i.KeyCode==Enum.KeyCode.Z then
		local last = table.remove(undoStack)
		if last then last.Parent = workspace end
	end
end)

--==================================================
-- FARM LOGIC (TIDAK DIUBAH)
--==================================================
task.spawn(function()
	while true do
		task.wait(0.3)
		if not running then continue end

		local water = findTool("water")
		if not water then status.Text="water not found" running=false continue end
		status.Text="WATER"
		equip(water) pressE()
		task.wait(25)

		local sugar = findTool("sugar")
		if not sugar then status.Text="sugar not found" running=false continue end
		status.Text="SUGAR"
		equip(sugar) pressE()
		task.wait(1.5)

		local gel = findTool("gelatin")
		if not gel then status.Text="gelatin not found" running=false continue end
		status.Text="GELATIN"
		equip(gel) pressE()

		task.wait(50)

		local bag = findTool("empty")
		if not bag then status.Text="empty bag not found" running=false continue end
		status.Text="EMPTY BAG"
		equip(bag) pressE()
		unequip()
		task.wait(2)
	end
end)

startBtn.MouseButton1Click:Connect(function()
	running = true
	status.Text="RUNNING"
end)

stopBtn.MouseButton1Click:Connect(function()
	running = false
	status.Text="STOPPED"
end)

--==================================================
-- TP TO VEH (ANTI MANTUL)
--==================================================
tpVehBtn.MouseButton1Click:Connect(function()
	for _,m in ipairs(workspace:GetChildren()) do
		if m:IsA("Model") and m.Name:lower():find(player.Name:lower()) then
			local seat = m:FindFirstChildWhichIsA("VehicleSeat",true)
			if seat and char:FindFirstChild("HumanoidRootPart") then
				char.HumanoidRootPart.CFrame = seat.CFrame * CFrame.new(-4,0,0)
				return
			end
		end
	end
	status.Text="vehicle not found"
end)

--==================================================
-- VEHICLE TP (SAFEZONE ANTI-KICK)
--==================================================
local function tpVehicleTo(modelName)
	local seat = hum.SeatPart
	if not seat or not seat:IsA("VehicleSeat") then
		status.Text="enter vehicle first"
		return
	end

	local car = seat:FindFirstAncestorOfClass("Model")
	if not car then status.Text="vehicle model not found" return end

	local map = workspace:FindFirstChild("Map") or workspace
	for _,m in ipairs(map:GetDescendants()) do
		if m:IsA("Model") and m.Name==modelName then
			local p = m.PrimaryPart or m:FindFirstChildWhichIsA("BasePart")
			if p then
				car.PrimaryPart = car.PrimaryPart or seat
				-- initial TP
				car:SetPrimaryPartCFrame(p.CFrame * CFrame.new(0, p.Size.Y + 18, 0))
				-- micro stabilize (anti kick)
				for i=1,15 do
					RunService.Heartbeat:Wait()
					car:SetPrimaryPartCFrame(car.PrimaryPart.CFrame * CFrame.new(0,0.05,0))
				end
				status.Text="TP SUCCESS"
				return
			end
		end
	end
	status.Text=modelName.." not found"
end

tpDsBtn.MouseButton1Click:Connect(function()
	tpVehicleTo("Dealership")
end)

tpMarshBtn.MouseButton1Click:Connect(function()
	tpVehicleTo("Dealer Basketball Court")
end)

--==================================================
-- INSTANT INTERACT TOGGLE
--==================================================
instBtn.MouseButton1Click:Connect(function()
	instantInteract = not instantInteract
	instBtn.Text = "INSTANT INTERACT : "..(instantInteract and "ON" or "OFF")
	print("InstantInteract:",instantInteract)
end)

--==================================================
-- COMING SOON LABEL
--==================================================
local comingSoon = Instance.new("TextLabel", main)
comingSoon.Size = UDim2.fromScale(1,0.05)
comingSoon.Position = UDim2.fromScale(0,0.85) -- di bawah semua button
comingSoon.BackgroundTransparency = 1
comingSoon.TextColor3 = Color3.fromRGB(255,255,255)
comingSoon.Font = Enum.Font.GothamBold
comingSoon.TextScaled = true
comingSoon.Text = "Auto sell? Coming soon"
