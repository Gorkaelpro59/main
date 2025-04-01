if game.PlaceId == 3623096087 then

    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
    
    -- Crear ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "MiLibreriaGui"
    screenGui.Parent = PlayerGui
    
    -- Crear botón para activar/desactivar la operación de repetición
    local repButton = Instance.new("TextButton")
    repButton.Size = UDim2.new(0, 200, 0, 50)
    repButton.Position = UDim2.new(0.5, -100, 0, 50)
    repButton.Text = "Activar Repetición"
    repButton.Parent = screenGui
    
    -- Crear botón para mostrar/ocultar frames
    local toggleFramesButton = Instance.new("TextButton")
    toggleFramesButton.Size = UDim2.new(0, 200, 0, 50)
    toggleFramesButton.Position = UDim2.new(0.5, -100, 0, 110)
    toggleFramesButton.Text = "Mostrar Frames"
    toggleFramesButton.Parent = screenGui
    
    -- Crear botón para iniciar la automatización
    local automationButton = Instance.new("TextButton")
    automationButton.Size = UDim2.new(0, 200, 0, 50)
    automationButton.Position = UDim2.new(0.5, -100, 0, 170)
    automationButton.Text = "Iniciar Automatización"
    automationButton.Parent = screenGui
    
    -- Requiere la librería
    local MiLibreria = require
    
    -- mi_libreria.lua
    
    local MiLibreria = {}
    
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    
    -- Función para manejar la operación de repetición
    function MiLibreria.RepOperation(Value)
        _G.RepOP = Value
        if not Value then return end
    
        local function FastRep()
            while _G.RepOP do
                for i = 1, 400 do
                    game:GetService("Players").LocalPlayer.muscleEvent:FireServer("rep")
                end
                task.wait()
            end
        end
    
        task.spawn(FastRep)
    end
    
    -- Función para mostrar u ocultar frames de la interfaz
    function MiLibreria.ToggleFrames(bool)
        for _, frameName in ipairs({"strengthFrame", "durabilityFrame", "agilityFrame"}) do
            local frame = game:GetService("ReplicatedStorage"):FindFirstChild(frameName)
            if frame and frame:IsA("GuiObject") then
                frame.Visible = not bool
            end
        end
    end
    
    -- Unequip all pets from the player's pets folder.
    function MiLibreria.UnequipAllPets()
        local petsFolder = LocalPlayer.petsFolder
        for _, folder in pairs(petsFolder:GetChildren()) do
            if folder:IsA("Folder") then
                for _, pet in pairs(folder:GetChildren()) do
                    ReplicatedStorage.rEvents.equipPetEvent:FireServer("unequipPet", pet)
                end
            end
        end
        task.wait(0.1)
    end
    
    -- Equip a specific pet by name from the "Unique" pets folder.
    function MiLibreria.EquipPet(petName)
        MiLibreria.UnequipAllPets()
        task.wait(0.01)
        for _, pet in pairs(LocalPlayer.petsFolder.Unique:GetChildren()) do
            if pet.Name == petName then
                ReplicatedStorage.rEvents.equipPetEvent:FireServer("equipPet", pet)
            end
        end
    end
    
    -- Attempt to find a machine with the specified name.
    function MiLibreria.FindMachine(machineName)
        local machine = workspace.machinesFolder:FindFirstChild(machineName)
        if not machine then
            for _, child in pairs(workspace:GetChildren()) do
                if child:IsA("Folder") and child.Name:find("machines") then
                    machine = child:FindFirstChild(machineName)
                    if machine then
                        break
                    end
                end
            end
        end
        return machine
    end
    
    -- Simulate pressing the "E" key.
    function MiLibreria.PressEKey()
        local VirtualInputManager = game:GetService("VirtualInputManager")
        VirtualInputManager:SendKeyEvent(true, "E", false, game)
        task.wait(0.1)
        VirtualInputManager:SendKeyEvent(false, "E", false, game)
    end
    
    -- Main automation loop.
    function MiLibreria.StartAutomation()
        task.spawn(function()
            while true do
                local ok, err = pcall(function()
                    -- Validate essential properties before proceeding.
                    if not (LocalPlayer and LocalPlayer:FindFirstChild("leaderstats") and LocalPlayer:FindFirstChild("petsFolder") 
                        and LocalPlayer:FindFirstChild("ultimatesFolder") and LocalPlayer.Character) then
                        error("Missing one or more essential LocalPlayer properties.")
                    end
    
                    local currentRebirths = LocalPlayer.leaderstats.Rebirths.Value
                    local strengthThreshold = 10000 + (5000 * currentRebirths)
                    
                    if LocalPlayer.ultimatesFolder:FindFirstChild("Golden Rebirth") then
                        local goldenRebirthValue = LocalPlayer.ultimatesFolder["Golden Rebirth"].Value
                        strengthThreshold = math.floor(strengthThreshold * (1 - (goldenRebirthValue * 0.1)))
                    end
    
                    MiLibreria.UnequipAllPets()
                    task.wait(0.1)
                    MiLibreria.EquipPet("Swift Samurai")
     
                    -- Increase strength until the threshold is met.
                    while LocalPlayer.leaderstats.Strength.Value < strengthThreshold do
                        for i = 1, 10 do
                            if LocalPlayer.muscleEvent then
                                LocalPlayer.muscleEvent:FireServer("rep")
                            else
                                error("muscleEvent not found.")
                            end
                        end
                    end
                end)
    
                if not ok then
                    warn("Error in automation loop: " .. err)
                end
                task.wait(1)
            end
        end)
    end
    
end
