local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local name = LocalPlayer.Name

local function SafeArea()
    local spawn = workspace.Lobby.SpawnLocations:GetChildren()[14]
    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = spawn.CFrame
end

local function CollectCoins()
    for _, coin in workspace.GameObjects:GetChildren() do
        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, coin, 0)
        firetouchinterest(game.Players.LocalPlayer.Character.HumanoidRootPart, coin, 1)
    end
end

local function Hop()
    local PlaceID = game.PlaceId
    local AllIDs = {}
    local foundAnything = ""
    local actualHour = os.date("!*t").hour
    local Deleted = false

    function TPReturner()
        local Site
        if foundAnything == "" then
            Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100'))
        else
            Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100&cursor=' .. foundAnything))
        end
        
        -- Debugging: Print the response
        print("API Response:", Site)

        if not Site or not Site.data then
            warn("Failed to retrieve valid server list!")
            return
        end

        if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
            foundAnything = Site.nextPageCursor
        else
            foundAnything = nil
        end

        for _, v in pairs(Site.data) do
            if tonumber(v.maxPlayers) > tonumber(v.playing) then
                local ID = tostring(v.id)

                if not table.find(AllIDs, ID) then
                    table.insert(AllIDs, ID)

                    -- Attempt to teleport
                    local success, err = pcall(function()
                        game:GetService("TeleportService"):TeleportToPlaceInstance(PlaceID, ID, game.Players.LocalPlayer)
                    end)

                    if not success then
                        warn("Teleport failed:", err)
                    else
                        print("Teleporting to Server:", ID)
                    end

                    wait(4) -- Prevent excessive requests
                    return
                end
            end
        end
    end

    function Teleport()
        while wait(5) do -- Add a small delay to prevent spamming
            local success, err = pcall(TPReturner)
            if not success then
                warn("Error in TPReturner:", err)
            end
        end
    end

    Teleport()
end

-- Function to teleport to all players
local function teleportToAllPlayers()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character.HumanoidRootPart.CFrame = player.Character.HumanoidRootPart.CFrame
            wait(0.1) -- Small delay for smooth teleporting
        end
    end
end

-- Notify user on script load
Fluent:Notify({
    Title = "Monkey Hub",
    Content = "Loading",
    SubContent = "Loading Monkey Hub",
    Duration = 5
})

-- Create main UI window
local Window = Fluent:CreateWindow({
    Title = "Monkey Hub | Hide and Sike Extreme",
    SubTitle = "by one Monkey",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Farme = Window:AddTab({ Title = "Auto Farm", Icon = "" }),
    Main = Window:AddTab({ Title = "Player", Icon = "" }),
    Siker = Window:AddTab({ Title = "Siker", Icon = "" }),
    Server = Window:AddTab({ Title = "Server", Icon = "" }),
}

-- Teleport to all players button

Tabs.Server:AddButton({
    Title = "Server Hop",
    Callback = function()
        Hop()
    end
})

Tabs.Farme:AddButton({
    Title = "Teleporte to safe area",
    Callback = function()
        SafeArea()
    end
})

Tabs.Farme:AddButton({
    Title = "Collect all coins",
    Callback = function()
        CollectCoins()
    end
})

Tabs.Siker:AddButton({
    Title = "Seek all players",
    Callback = function()
        teleportToAllPlayers()
        Fluent:Notify({
            Title = "Monaco Hub",
            Content = "Seking all player",
            Duration = 3
        })
    end
})

-- Speed Slider (Optimized)
Tabs.Main:AddSlider("SpeedSlider", {
    Title = "Player Speed",
    Description = "Change the player speed",
    Default = 1,
    Min = 1,
    Max = 10,
    Rounding = 1,
    Callback = function(Value)
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = Value * 16
        end
    end
})

-- Jump Height Slider (Optimized)
Tabs.Main:AddSlider("JumpSlider", {
    Title = "Player Jump",
    Description = "Change the player jump height",
    Default = 1,
    Min = 1,
    Max = 10,
    Rounding = 1,
    Callback = function(Value)
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.JumpHeight = Value * 16
        end
    end
})

-- Load saved configurations
SaveManager:LoadAutoloadConfig()
