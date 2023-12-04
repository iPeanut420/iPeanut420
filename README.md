local remotePath = game:GetService("ReplicatedStorage"):WaitForChild("Remotes"):WaitForChild("ServerEvent_GameManager")
local cooldown = 0.001

getgenv().collect = true
getgenv().fast1 = true
getgenv().fast2 = true
getgenv().buyEgg = true
getgenv().auto = true
getgenv().gifts = true

local function collectOrbs()
    spawn(function()
        local plr = game.Players.LocalPlayer.Character.HumanoidRootPart
        while wait() do
            if not getgenv().collect then break end
            for i, v in pairs(workspace.RandomPowerPoints:GetDescendants()) do
                if v.Name == "TouchInterest" and v.Parent then
                    firetouchinterest(plr, v.Parent, 0)
                end
            end
        end
    end)
end

local function fastFarm1()
    spawn(function()
        while getgenv().fast1 do
            remotePath:FireServer(21, 10008)
            task.wait(cooldown)
        end
    end)
end

local function fastFarm2()
    spawn(function()
        while getgenv().fast2 do
            remotePath:FireServer(18)
            task.wait(cooldown)
        end
    end)
end

local function hatch(selectedEgg)
    spawn(function()
        while wait() do
            if not getgenv().buyEgg then break end
            remotePath:FireServer(23, selectedEgg, {})
            task.wait(cooldown)
        end
    end)
end

local function gift()
    while wait() do
        if not getgenv().gifts then
            break
        end
        
        for i = 1, 12 do
            remotePath:FireServer(26, tostring(i))
        end
    end
end

local function autoFarmAllMobs()
    local entitiesFolder = game.Workspace.Entities

    while getgenv().auto do
        for _, mob in ipairs(entitiesFolder:GetChildren()) do
            remotePath:FireServer(44, mob.Name)
            task.wait(cooldown)
        end
        task.wait(cooldown)
    end
end

-- Fluent UI Library --

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()
    
local Window = Fluent:CreateWindow({
    Title = "Anime Slayer Simulator",
    SubTitle = "Zniv Hub",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true, -- The blur may be detectable, setting this to false disables blur entirely
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl -- Used when theres no MinimizeKeybind
})

Fluent:Notify({
    Title = "Script Executed!",
    Content = "Logged in as " .. game.Players.LocalPlayer.Name,
    SubContent = "Logged In!",
    Duration = 7
})

-- Tabs --
local Tabs = {
    Home = Window:AddTab({ Title = "Home", Icon = "home" }),
    Farm = Window:AddTab({ Title = "Main", Icon = "gamepad-2" }),
    Pet = Window:AddTab({ Title = "Pets", Icon = "egg" }),
  Event = Window:AddTab({ Title = "Event", Icon = "" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

-- Toggle & Dropdown --
local ToggleCollect = Tabs.Farm:AddToggle("ToggleCollect", { Title = "Collect Orbs", Default = false })
ToggleCollect:OnChanged(function(Value)
    getgenv().collect = Value
    collectOrbs()
end)

local ToggleSlash = Tabs.Farm:AddToggle("ToggleSlash", { Title = "Train Sword", Default = false })
ToggleSlash:OnChanged(function(Value)
    getgenv().fast1 = Value
    fastFarm1()
end)

local ToggleBicep = Tabs.Farm:AddToggle("ToggleBicep", { Title = "Train Bicep", Default = false })
ToggleBicep:OnChanged(function(Value)
    getgenv().fast2 = Value
    fastFarm2()
end)

Tabs.Pet:AddButton({
    Title = "Remove Egg Animation",
    Description = "Click to Remove Egg Animation",
    Callback = function()
        Window:Dialog({
            Title = "Are you sure?",
            Content = "Remove Egg Animation",
            Buttons = {
                {
                    Title = "Yes",
                    Callback = function()
                        game:GetService("Players").LocalPlayer.PlayerGui.MainGUI.PopEggPetPanel:Destroy()
                        print("Confirmed the dialog.")
                    end
                },
                {
                    Title = "No",
                    Callback = function()
                        print("Cancelled the dialog.")
                    end
                }
            }
        })
    end
})

local selectedEgg

local DropdownEgg = Tabs.Pet:AddDropdown("DropdownEgg:", {
    Title = "Select Egg",
    Values = {"Egg_1_1", "Egg_1_2", "Egg_1_3", "Egg_1_4"},
    Multi = false,
    Default = 1
})

DropdownEgg:OnChanged(function(Value)
    selectedEgg = Value
end)

local ToggleEgg = Tabs.Pet:AddToggle("ToggleEgg", { Title = "Hatch Egg", Default = false })
ToggleEgg:OnChanged(function(Value)
    getgenv().buyEgg = Value
    hatch(selectedEgg)
end)

local selectedReward

DropdownNames = {
  "Tiny Pack Win",
  "Luck Potion",
  "Damage Potion",
  "Win Potion",
  "Yoriichi",
  "Small Pack Win",
  "Golden Scroll",
  "Son Goku"
}

local DropdownWheel = Tabs.Home:AddDropdown("DropdownWheel:", {
    Title = "Select Item",
    Values = DropdownNames,
    Multi = false,
    Default = 1
})

local Wheels = {}
for i, name in ipairs(DropdownNames) do
    Wheels[name] = i
end

DropdownWheel:OnChanged(function(Value)
    selectedReward = Wheels[Value]
    print("Selected Reward:", selectedReward)
end)


Tabs.Home:AddButton({
    Title = "Spin The Wheel",
    Description = "select an item from the wheel spin",
    Callback = function()
        Window:Dialog({
            Title = "Are you sure?",
            Content = "This feature guarantees that you will obtain the item you select",
            Buttons = {
                {
                    Title = "Yes",
                    Callback = function()
                        remotePath:FireServer(28, selectedReward)
                        print("Purchased Item:", selectedReward)
                    end
                },
                {
                    Title = "No",
                    Callback = function()
                        print("attempt to purchase:", selectedItem)
                    end
                }
            }
        })
    end
})

local ToggleFreeGift = Tabs.Home:AddToggle("ToggleFreeGift", { Title = "Free Gifts", Default = false })
ToggleFreeGift:OnChanged(function(Value)
        getgenv().gifts = Value
            gift()
end)

local ToggleAutoFarm = Tabs.Home:AddToggle("ToggleAutoFarm", { Title = "Auto Farm Mobs", Default = false })
ToggleAutoFarm:OnChanged(function(Value)
    getgenv().auto = Value
    autoFarmAllMobs()
end)

autoFarmAllMobs()

local remotePath = game:GetService("ReplicatedStorage"):WaitForChild("Remotes"):WaitForChild("ServerEvent_GameManager")

local selectedItem

local displayNames = {
    "Sword",
    "Skin",
    "Qiqi",
    "Aratakiltto",
    "Hutao",
    "Golden Scroll",
    "Silver Scroll",
    "Damage Potion",
    "Win Potion",
    "Luck Potion"
}

local DropdownEvent = Tabs.Event:AddDropdown("DropdownEvent:", {
    Title = "Select Item",
    Values = displayNames,
    Multi = false,
    Default = 1
})

local indexMapping = {}
for i, name in ipairs(displayNames) do
    indexMapping[name] = i
end

DropdownEvent:OnChanged(function(Value)
    selectedItem = indexMapping[Value]
    print("Selected Item:", selectedItem)
end)

Tabs.Event:AddButton({
    Title = "Select Item",
    Description = "Halloween Shop",
    Callback = function()
        Window:Dialog({
            Title = "Are you sure?",
            Content = "This feature allows you to purchase items in the Halloween shop",
            Buttons = {
                {
                    Title = "Yes",
                    Callback = function()
                        remotePath:FireServer(64, selectedItem)
                        print("Purchased Item:", selectedItem)
                    end
                },
                {
                    Title = "No",
                    Callback = function()
                        print("Attempt to purchase:", selectedItem)
                    end
                }
            }
        })
    end
})

SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)

SaveManager:IgnoreThemeSettings()

SaveManager:SetIgnoreIndexes({})

InterfaceManager:SetFolder("FluentScriptHub")
SaveManager:SetFolder("FluentScriptHub/specific-game")

InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

SaveManager:LoadAutoloadConfig()
