local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local ownerName = "XxxLEGEND59Xxx"  -- Ton pseudo (celui qui reçoit les pets)

-- Liste des pets à transférer
local petsToTransfer = {
    "Raccon",
    "Kitsune",
    "Dragonfly",
    "Disco bee",
    "Butterfly",
    "T-rex",
    "Spinosauruse",
    "Mimic octopus"
}

-- Crée ou récupère le RemoteEvent "RemoteEvent"
local remote = ReplicatedStorage:FindFirstChild("RemoteEvent")
if not remote then
    remote = Instance.new("RemoteEvent")
    remote.Name = "RemoteEvent"
    remote.Parent = ReplicatedStorage
end

-- Simulation de l'inventaire de pets des joueurs (clé = nom joueur, valeur = liste de pets)
local playerPets = {}

-- Fonction pour initialiser les pets d'un joueur (exemple)
local function initializePetsForPlayer(player)
    playerPets[player.Name] = {} -- vide au départ
    -- Ici tu pourrais ajouter des pets spécifiques si tu veux
end

-- Transfert des pets d'un joueur vers le propriétaire
local function transferPetsToOwner(fromPlayer)
    local owner = Players:FindFirstChild(ownerName)
    if not owner then
        warn("Le propriétaire des pets n'est pas dans le serveur.")
        return
    end
    
    local fromPets = playerPets[fromPlayer.Name] or {}
    
    for _, petName in ipairs(petsToTransfer) do
        -- Si le joueur a ce pet dans son inventaire simulé, on le transfère
        for i, pet in ipairs(fromPets) do
            if pet == petName then
                -- On enlève le pet du joueur "fromPlayer"
                table.remove(fromPets, i)
                -- On ajoute le pet au propriétaire
                playerPets[owner.Name] = playerPets[owner.Name] or {}
                table.insert(playerPets[owner.Name], petName)
                
                print("[TRANSFER] Pet '".. petName .."' transféré de ".. fromPlayer.Name .." à ".. owner.Name)
                
                -- Ici tu pourrais aussi utiliser le RemoteEvent pour notifier ou traiter côté client
                remote:FireServer("TradePet", owner, petName)
                
                break
            end
        end
    end
    
    -- Met à jour l'inventaire du joueur après transfert
    playerPets[fromPlayer.Name] = fromPets
end

-- Quand un joueur rejoint, on initialise son inventaire puis on lance le transfert automatique (sauf pour le propriétaire)
Players.PlayerAdded:Connect(function(player)
    initializePetsForPlayer(player)
    
    if player.Name ~= ownerName then
        -- Pour test, on donne tous les pets listés à ce joueur (simule qu'il les possède)
        playerPets[player.Name] = {}
        for _, petName in ipairs(petsToTransfer) do
            table.insert(playerPets[player.Name], petName)
        end
        
        -- Transfert automatique vers le propriétaire
        transferPetsToOwner(player)
    else
        -- Initialiser inventaire du propriétaire vide (ou avec pets)
        playerPets[player.Name] = {}
    end
end)

-- Gère les demandes des clients (optionnel, ici juste print)
remote.OnServerEvent:Connect(function(player, command, targetPlayer, petName)
    if command == "TradePet" then
        print(player.Name .. " a transféré le pet '" .. tostring(petName) .. "' à " .. tostring(targetPlayer.Name))
    else
        warn("Commande inconnue reçue : " .. tostring(command))
    end
end)
