local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "neptune " .. Fluent.Version,
    SubTitle = "by Dot",
    TabWidth = 110,
    Size = UDim2.fromOffset(500, 300),
    Acrylic = true,
    Theme = "NSExpression",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "" }),
    Misc = Window:AddTab({ Title = "Misc", Icon = "" })
}

local Options = Fluent.Options
local Toggle = Tabs.Misc:AddToggle("MyToggle", {Title = "Range Indicator", Default = false })
local CamlockEnabled = false

Toggle:OnChanged(function()
    RangeIndicatorEnabled = Options.MyToggle.Value
end)

Options.MyToggle:SetValue(false)

local Toggle = Tabs.Main:AddToggle("CamlockToggle", { Title = "Enable Camlock", Default = false })
local CamlockEnabled = false

Toggle:OnChanged(function()
    CamlockEnabled = Options.CamlockToggle.Value
end)

-- Function to calculate Higher Arc adjustment
local function HighArc(dist)
    if dist >= 59 and dist <= 61 then
        return 30  -- Increase arc for short distances
    elseif dist >= 62 and dist <= 63 then
        return 27
    elseif dist >= 64 and dist <= 65 then
        return 24
    elseif dist >= 66 and dist <= 67 then
        return 20
    elseif dist >= 68 and dist <= 69 then
        return 17
    elseif dist >= 70 and dist <= 71 then
        return 14
    elseif dist == 72 then
        return 10
    else
        return 0  -- No arc adjustment for distances outside this range
    end
end

-- Camlock Function (With High Arc for Shooting)
local function Camlock()
    local Players = game:GetService("Players")
    local Workspace = game:GetService("Workspace")
    local Player = Players.LocalPlayer
    local Character = Player.Character or Player.CharacterAdded:Wait()
    local Camera = Workspace.CurrentCamera
    local Humanoid = Character:WaitForChild("Humanoid")
    local Ball = nil  -- Variable to store the basketball

    -- Find the nearest hoop
    local function GetNearestHoop()
        local Distance, TargetHoop = math.huge, nil
        local CharacterPosition = Character.PrimaryPart.Position

        local function CheckHoops(container)
            if not container then return end
            for _, court in ipairs(container:GetChildren()) do
                for _, Obj in ipairs(court:GetDescendants()) do
                    if Obj.Name == "Swish" and Obj.Parent:FindFirstChildOfClass("TouchTransmitter") then
                        local HoopPosition = Obj.Parent.Position
                        local Magnitude = (CharacterPosition - HoopPosition).Magnitude
                        if Magnitude < Distance then
                            Distance = Magnitude
                            TargetHoop = Obj.Parent
                        end
                    end
                end
            end
        end

        CheckHoops(Workspace:FindFirstChild("Courts"))
        CheckHoops(Workspace:FindFirstChild("PracticeArea"))

        return TargetHoop, Distance
    end

    -- Function to adjust aim (With High Arc)
    local function AimAtHoop()
        local Hoop, Distance = GetNearestHoop()
        if Hoop then
            local HoopPosition = Hoop.Position
            local PlayerPosition = Character.PrimaryPart.Position
            local ArcAdjustment = HighArc(math.floor(Distance))  -- Apply Higher Arc adjustment
            local AdjustedHoopPosition = HoopPosition + Vector3.new(0, ArcAdjustment, 0)

            -- Smoothly adjust camera towards the hoop with arc correction
            local NewCameraCFrame = CFrame.new(Camera.CFrame.Position, AdjustedHoopPosition)
            Camera.CFrame = NewCameraCFrame
        end
    end

    -- Find the basketball
    local function FindBasketball()
        -- Assuming the basketball is a part named "Basketball" in the workspace
        for _, obj in ipairs(Workspace:GetChildren()) do
            if obj.Name == "Basketball" then
                Ball = obj
                return Ball
            end
        end
        return nil
    end

    -- Function to automatically shoot the ball
    local function ShootBasketball()
        if Ball then
            local Hoop, Distance = GetNearestHoop()
            if Hoop then
                -- Calculate the shot direction and apply an upward force
                local Direction = (Hoop.Position - Ball.Position).unit
                local Force = 150  -- Increase shot strength for a better result
                local BodyVelocity = Instance.new("BodyVelocity")
                BodyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
                BodyVelocity.Velocity = Direction * Force + Vector3.new(0, 70, 0)  -- Higher upward velocity for more arc
                BodyVelocity.Parent = Ball
            end
        end
    end

    -- Detect Jumping and Landing
    Humanoid.StateChanged:Connect(function(_, NewState)
        if CamlockEnabled then
            if NewState == Enum.HumanoidStateType.Jumping then
                -- When jumping, switch to Follow mode and aim at the hoop
                Camera.CameraType = Enum.CameraType.Follow  -- Set camera to Follow mode
                AimAtHoop()  -- Focus the camera on the hoop with arc adjustment
                ShootBasketball()  -- Automatically shoot the basketball
            elseif NewState == Enum.HumanoidStateType.Landed then
                -- When the player lands, reset the camera to Custom or Scriptable
                Camera.CameraType = Enum.CameraType.Custom  -- Or use Enum.CameraType.Scriptable if you prefer
            end
        end
    end)
end

-- Activate Camlock
Camlock()
