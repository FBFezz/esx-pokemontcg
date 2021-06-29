## Must have below added to qb-inventory for the box to have its own stash (This stash holds anything with 0 weight so make sure to have anything else that weighs 0 = 1 or something.)

## Client/Main.lua ##

# Under (around ln 35) # 
```lua

 if name ~= CurrentStash or CurrentStash == nil then
            TriggerServerEvent('inventory:server:SetIsOpenState', false, type, id)
        end

--# Add this #


    elseif type == "pokeBox" then
        if name ~= CurrentStash or CurrentStash == nil then
            TriggerServerEvent('inventory:server:SetIsOpenState', false, type, id)
        end

--################################################################

--# under this (around LN 736)#


    CurrentStash = stash
end)

--# Add this # 


RegisterNetEvent("inventory:client:SetCurrentPokeBox")
AddEventHandler("inventory:client:SetCurrentPokeBox", function(stash)
    CurrentStash = pokeBox
end)











```



##### Server/main.lua ###################
```lua
--# Under this (around line 67)


	if not IsOpen then
		if type == "stash" then
			Stashes[id].isOpen = false

--# add this # 


		elseif type == "pokeBox" then
			Stashes[id].isOpen = false

--# so it should look like this after -#


		elseif type == "pokeBox" then
			Stashes[id].isOpen = false
		elseif type == "trunk" then
			Trunks[id].isOpen = false
		elseif type == "glovebox" then

--###########################################################

-- Under this (line 134ish)

						Stashes[id].label = secondInv.label
					end
				end


-- add this # 


			elseif name == "pokeBox" then
				if Stashes[id] ~= nil then
					if Stashes[id].isOpen then
						local Target = QBCore.Functions.GetPlayer(Stashes[id].isOpen)
						if Target ~= nil then
							TriggerClientEvent('inventory:client:CheckOpenState', Stashes[id].isOpen, name, id, Stashes[id].label)
						else
							Stashes[id].isOpen = false
						end
					end
				end
				local maxweight = 1
				local slots = 160
				if other ~= nil then 
					maxweight = other.maxweight ~= nil and other.maxweight or 1
					slots = other.slots ~= nil and other.slots or 160
				end
				secondInv.name = "stash-"..id
				secondInv.label = "Stash-"..id
				secondInv.maxweight = maxweight
				secondInv.inventory = {}
				secondInv.slots = slots
				if Stashes[id] ~= nil and Stashes[id].isOpen then
					secondInv.name = "none-inv"
					secondInv.label = "Stash-None"
					secondInv.maxweight = 1
					secondInv.inventory = {}
					secondInv.slots = 0
				else
					local stashItems = GetStashItems(id)
					if next(stashItems) ~= nil then
						secondInv.inventory = stashItems
						Stashes[id] = {}
						Stashes[id].items = stashItems
						Stashes[id].isOpen = src
						Stashes[id].label = secondInv.label
					else
						Stashes[id] = {}
						Stashes[id].items = {}
						Stashes[id].isOpen = src
						Stashes[id].label = secondInv.label
					end
				end

--#################################

-- Under this (line 344ish)
	end
	elseif type == "stash" then
		SaveStashItems(id, Stashes[id].items)

-- add this 
	elseif type == "pokeBox" then
		SaveStashItems(id, Stashes[id].items)


--#########################

-- under this (line 508ish)
				end
				local itemInfo = QBCore.Shared.Items[fromItemData.name:lower()]
				AddToStash(stashId, toSlot, fromSlot, itemInfo["name"], fromAmount, fromItemData.info)


-- add this 


			elseif QBCore.Shared.SplitStr(toInventory, "-")[1] == "pokeBox" then  --- PokeBox
				local stashId = QBCore.Shared.SplitStr(toInventory, "-")[2]
				local toItemData = Stashes[stashId].items[toSlot]
				local IsItemValid = exports['pokeBox']:CanItemBeSaled(fromItemData.name:lower())
				if IsItemValid then
					Player.Functions.RemoveItem(fromItemData.name, fromAmount, fromSlot)
					TriggerClientEvent("inventory:client:CheckWeapon", src, fromItemData.name)
					--Player.PlayerData.items[toSlot] = fromItemData
					if toItemData ~= nil then
						--Player.PlayerData.items[fromSlot] = toItemData
						local itemInfo = QBCore.Shared.Items[toItemData.name:lower()]
						local toAmount = tonumber(toAmount) ~= nil and tonumber(toAmount) or toItemData.amount
						if toItemData.name ~= fromItemData.name then
							RemoveFromStash(stashId, fromSlot, itemInfo["name"], toAmount)
							Player.Functions.AddItem(toItemData.name, toAmount, fromSlot, toItemData.info)
							TriggerEvent("qb-log:server:CreateLog", "stash", "Swapped Item", "orange", "**".. GetPlayerName(src) .. "** (citizenid: *"..Player.PlayerData.citizenid.."* | id: *"..src.."*) swapped item; name: **"..itemInfo["name"].."**, amount: **" .. toAmount .. "** with name: **" .. fromItemData.name .. "**, amount: **" .. fromAmount .. "** - stash: *" .. stashId .. "*")
						end
					else
						local itemInfo = QBCore.Shared.Items[fromItemData.name:lower()]
						TriggerEvent("qb-log:server:CreateLog", "stash", "Dropped Item", "red", "**".. GetPlayerName(src) .. "** (citizenid: *"..Player.PlayerData.citizenid.."* | id: *"..src.."*) dropped new item; name: **"..itemInfo["name"].."**, amount: **" .. fromAmount .. "** - stash: *" .. stashId .. "*")
					end
					local itemInfo = QBCore.Shared.Items[fromItemData.name:lower()]
					AddToStash(stashId, toSlot, fromSlot, itemInfo["name"], fromAmount, fromItemData.info)
				else 
					TriggerClientEvent('QBCore:Notify', src, "You can\'t Store this item in here..", 'error')

				end

--- #########################################

-- Under this (line 774ish)
		else
			TriggerClientEvent("QBCore:Notify", src, "Item doesn\'t exist??", "error")
		end

-- add this 
elseif QBCore.Shared.SplitStr(fromInventory, "-")[1] == "pokeBox" then
		local stashId = QBCore.Shared.SplitStr(fromInventory, "-")[2]
		local fromItemData = Stashes[stashId].items[fromSlot]
		local fromAmount = tonumber(fromAmount) ~= nil and tonumber(fromAmount) or fromItemData.amount
		if fromItemData ~= nil and fromItemData.amount >= fromAmount then
			local itemInfo = QBCore.Shared.Items[fromItemData.name:lower()]
			if toInventory == "player" or toInventory == "hotbar" then
				local toItemData = Player.Functions.GetItemBySlot(toSlot)
				RemoveFromStash(stashId, fromSlot, itemInfo["name"], fromAmount)
				if toItemData ~= nil then
					local itemInfo = QBCore.Shared.Items[toItemData.name:lower()]
					local toAmount = tonumber(toAmount) ~= nil and tonumber(toAmount) or toItemData.amount
					if toItemData.name ~= fromItemData.name then
						Player.Functions.RemoveItem(toItemData.name, toAmount, toSlot)
						AddToStash(stashId, fromSlot, toSlot, itemInfo["name"], toAmount, toItemData.info)
						TriggerEvent("qb-log:server:CreateLog", "stash", "Swapped Item", "orange", "**".. GetPlayerName(src) .. "** (citizenid: *"..Player.PlayerData.citizenid.."* | id: *"..src.."*) swapped item; name: **"..toItemData.name.."**, amount: **" .. toAmount .. "** with item; name: **"..fromItemData.name.."**, amount: **" .. fromAmount .. "** stash: *" .. stashId .. "*")
					else
						TriggerEvent("qb-log:server:CreateLog", "stash", "Stacked Item", "orange", "**".. GetPlayerName(src) .. "** (citizenid: *"..Player.PlayerData.citizenid.."* | id: *"..src.."*) stacked item; name: **"..toItemData.name.."**, amount: **" .. toAmount .. "** from stash: *" .. stashId .. "*")
					end
				else
					TriggerEvent("qb-log:server:CreateLog", "stash", "Received Item", "green", "**".. GetPlayerName(src) .. "** (citizenid: *"..Player.PlayerData.citizenid.."* | id: *"..src.."*) reveived item; name: **"..fromItemData.name.."**, amount: **" .. fromAmount.. "** stash: *" .. stashId .. "*")
				end
				SaveStashItems(stashId, Stashes[stashId].items)
				Player.Functions.AddItem(fromItemData.name, fromAmount, toSlot, fromItemData.info)
			else
				local toItemData = Stashes[stashId].items[toSlot]
				RemoveFromStash(stashId, fromSlot, itemInfo["name"], fromAmount)
				--Player.PlayerData.items[toSlot] = fromItemData
				if toItemData ~= nil then
					local itemInfo = QBCore.Shared.Items[toItemData.name:lower()]
					--Player.PlayerData.items[fromSlot] = toItemData
					local toAmount = tonumber(toAmount) ~= nil and tonumber(toAmount) or toItemData.amount
					if toItemData.name ~= fromItemData.name then
						local itemInfo = QBCore.Shared.Items[toItemData.name:lower()]
						RemoveFromStash(stashId, toSlot, itemInfo["name"], toAmount)
						AddToStash(stashId, fromSlot, toSlot, itemInfo["name"], toAmount, toItemData.info)
					end
				else
					--Player.PlayerData.items[fromSlot] = nil
				end
				local itemInfo = QBCore.Shared.Items[fromItemData.name:lower()]
				AddToStash(stashId, toSlot, fromSlot, itemInfo["name"], fromAmount, fromItemData.info)
			end
		else
			TriggerClientEvent("QBCore:Notify", src, "Item doesn\'t exist??", "error")
		end


---###########################################################
 

 -- Underthis (line 1642ish )
 			if Stashes[invId] ~= nil then 
				Stashes[invId].isOpen = false
			end
-- add this 
		elseif invType == "pokeBox" then
			if Stashes[invId] ~= nil then 
				Stashes[invId].isOpen = false
			end


```
## for shared.lua ##
```lua
["boosterbox"] 					 = {["name"] = "boosterbox",		  	  		["label"] = "Boosterbox", 				["weight"] = 200, 		["type"] = "item", 		["image"] = "boosterBox.png", 			["unique"] = false, 	["useable"] = true, 	["shouldClose"] = true,	   ["combinable"] = nil,   ["description"] = "Box Of Card Packs"},
["boosterpack"] 				 = {["name"] = "boosterpack", 			  	  	["label"] = "Boosterpack", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "boosterPack.png", 			["unique"] = false, 	["useable"] = true, 	["shouldClose"] = true,   ["combinable"] = nil,   ["description"] = "Pack of Cards"},
["abra"] 						 = {["name"] = "abra", 			  			  	["label"] = "Abra", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "abra.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "3/6 for Marsh Badge"},
["aerodactyl"] 					 = {["name"] = "aerodactyl", 			  	  	["label"] = "Aerodactyl", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "aerodactyl.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Ultra Rare"},
["alakazam"] 					 = {["name"] = "alakazam", 					 	["label"] = "Alakazam", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "alakazam.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "1/6 for Marsh Badge"},
["arbok"] 						 = {["name"] = "arbok", 			  	  		["label"] = "Arbok", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "arbok.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = ""},
["arcanine"] 					 = {["name"] = "arcanine", 				  	  	["label"] = "Arcanine", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "arcanine.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = "1/6 for Volcano Badge"},
["articuno"] 					 = {["name"] = "articuno", 			  		 	["label"] = "Articuno", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "articuno.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = "Ultra Rare"},
["beedrill"] 					 = {["name"] = "beedrill", 			  	  		["label"] = "Beedrill", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "beedrill.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = ""},
["bellsprout"] 					 = {["name"] = "bellsprout", 			  	  	["label"] = "Bellsprout", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "bellsprout.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "5/6 for Rainbow Badge"},
["blastoise"] 					 = {["name"] = "blastoise", 			  	  	["label"] = "Blastoise", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "blastoise.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "1/6 for Cascade Badge - Ultra Rare"},
["boulderbadge"] 				 = {["name"] = "boulderbadge", 			 		["label"] = "Boulderbadge", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "boulderBadge.png", 		["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "1/8 for Trophy Badge"},
["butterfree"] 					 = {["name"] = "butterfree", 			  		["label"] = "Butterfree", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "butterfree.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["cascadeBadge"] 				 = {["name"] = "cascadeBadge", 			  		["label"] = "CascadeBadge", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "cascadeBadge.png", 		["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "2/8 for Trophy Badge"},
["caterpie"] 				     = {["name"] = "caterpie", 			  			["label"] = "Caterpie", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "caterpie.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["chansey"] 				 	 = {["name"] = "chansey", 			  			["label"] = "Chansey", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "chansey.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["charizard"] 				 	 = {["name"] = "charizard", 			  		["label"] = "Charizard", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "charizard.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "5/6 for Volcano Badge - Ultra Rare"},
["charmander"] 				 	 = {["name"] = "charmander", 			  		["label"] = "Charmander", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "charmander.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["charmeleon"] 				 	 = {["name"] = "charmeleon", 			  		["label"] = "Charmeleon", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "charmeleon.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["clefable"] 					 = {["name"] = "clefable", 			  		 	["label"] = "Clefable", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "clefable.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["clefairy"] 				 	 = {["name"] = "clefairy", 			  	  		["label"] = "Clefairy", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "clefairy.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["cloyster"] 				 	 = {["name"] = "cloyster", 			  	  		["label"] = "Cloyster", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "cloyster.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["cubone"] 						 = {["name"] = "cubone", 			  	  		["label"] = "Cubone", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "cubone.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["dewgong"] 				 	 = {["name"] = "dewgong", 			  	  		["label"] = "Dewgong", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "dewgong.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["diglett"] 					 = {["name"] = "diglett", 			 	  	  	["label"] = "Diglett", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "diglett.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["ditto"] 						 = {["name"] = "ditto", 			 			["label"] = "Ditto", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "ditto.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Rare"},
["dodrio"] 						 = {["name"] = "dodrio", 			 			["label"] = "Dodrio", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "dodrio.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["doduo"] 						 = {["name"] = "doduo", 						["label"] = "Doduo", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "doduo.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["dragonair"] 					 = {["name"] = "dragonair", 			 	  	["label"] = "Dragonair", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "dragonair.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Ultra Rare"},
["dragonite"] 					 = {["name"] = "dragonite", 				    ["label"] = "Dragonite", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "dragonite.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "Ultra Rare"},
["dratini"] 					 = {["name"] = "dratini", 			    		["label"] = "Dratini", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "dratini.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "Ultra Rare"},
["drowzee"] 					 = {["name"] = "drowzee", 			 			["label"] = "Drowzee", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "drowzee.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["dugtrio"] 				  	 = {["name"] = "dugtrio", 			 			["label"] = "Dugtrio", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "dugtrio.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "5/6 for Earth Badge"},
["earthBadge"] 					 = {["name"] = "earthBadge", 					["label"] = "EarthBadge", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "earthBadge.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "8/8 for Trophy Badge"},
["eevee"] 						 = {["name"] = "eevee", 			 			["label"] = "Eevee", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "eevee.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "Rare"},
["ekans"] 						 = {["name"] = "ekans", 			 			["label"] = "Ekans", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "ekans.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["electabuzz"] 				     = {["name"] = "electabuzz", 			 		["label"] = "Electabuzz", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "electabuzz.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "4/6 for Thunder Badge"},
["electrode"] 					 = {["name"] = "electrode", 					["label"] = "Electrode", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "electrode.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "5/6 for Thunder Badge"},
["exeggcute"] 					 = {["name"] = "exeggcute", 				    ["label"] = "Exeggcute", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "exeggcute.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["exeggutor"] 					 = {["name"] = "exeggutor", 					["label"] = "Exeggutor", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "exeggutor.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["farfetchd"] 					 = {["name"] = "farfetchd", 					["label"] = "Farfetchd", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "farfetchd.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["fearow"] 						 = {["name"] = "fearow", 						["label"] = "Fearow", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "fearow.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["flareon"] 					 = {["name"] = "flareon", 					    ["label"] = "Flareon", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "flareon.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Rare"},
["gastly"] 						 = {["name"] = "gastly", 					    ["label"] = "Gastly", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "gastly.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["gengar"] 						 = {["name"] = "gengar", 					    ["label"] = "Gengar", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "gengar.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["geodude"] 				 	 = {["name"] = "geodude", 				    	["label"] = "Geodude", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "geodude.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "6/6 for Boulder Badge"},
["gloom"] 						 = {["name"] = "gloom", 						["label"] = "Gloom", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "gloom.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["golbat"] 						 = {["name"] = "golbat", 						["label"] = "Golbat", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "golbat.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "6/6 for Soul Badge"},
["goldeen"] 					 = {["name"] = "goldeen", 						["label"] = "Goldeen", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "goldeen.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["golduck"] 		 			 = {["name"] = "golduck", 						["label"] = "Golduck", 					["weight"] = 0, 	    ["type"] = "item", 		["image"] = "golduck.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "5/6 for Cascade Badge"},
["golem"] 		 				 = {["name"] = "golem", 						["label"] = "Golem", 					["weight"] = 0, 	    ["type"] = "item", 		["image"] = "golem.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["graveler"] 		 			 = {["name"] = "graveler", 						["label"] = "Graveler", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "graveler.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "1/6 for Boulder Badge"},
["grimer"] 		 				 = {["name"] = "grimer", 						["label"] = "Grimer", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "grimer.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["growlithe"] 		 			 = {["name"] = "growlithe", 					["label"] = "Growlithe", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "growlithe.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["gyarados"] 				 	 = {["name"] = "gyarados", 			  	  		["label"] = "Gyarados", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "gyarados.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Rare"},
["haunter"] 					 = {["name"] = "haunter", 						["label"] = "Haunter", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "haunter.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["hitmonchan"] 				 	 = {["name"] = "hitmonchan", 			  	  	["label"] = "Hitmonchan", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "hitmonchan.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Rare"},
["hitmonlee"] 				 	 = {["name"] = "hitmonlee", 			   		["label"] = "Hitmonlee", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "hitmonlee.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Rare"},
["horsea"] 						 = {["name"] = "horsea", 			  		 	["label"] = "Horsea", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "horsea.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["hypno"] 				 		 = {["name"] = "hypno", 			  	  		["label"] = "Hypno", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "hypno.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "5/6 for Marsh Badge"},
["ivysaur"] 					 = {["name"] = "ivysaur", 			  			["label"] = "Ivysaur", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "ivysaur.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["jigglypuff"] 					 = {["name"] = "jigglypuff", 			  	  	["label"] = "Jigglypuff", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "jigglypuff.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Rare"},
["jolteon"] 					 = {["name"] = "jolteon", 					  	["label"] = "Jolteon", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "jolteon.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "3/6 for Thunder Badge - Rare"},
["jynx"] 						 = {["name"] = "jynx", 						  	["label"] = "Jynx", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "jynx.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "6/6 for Marsh Badge - Rare"},
["kabuto"] 						 = {["name"] = "kabuto", 					  	["label"] = "Kabuto", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "kabuto.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Ultra Rare"},
["kabutops"] 			 		 = {["name"] = "kabutops", 				 		["label"] = "Kabutops", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "kabutops.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "4/6 for Boulder Badge - Ultra Rare"},
["kadabra"] 			 	 	 = {["name"] = "kadabra", 			  			["label"] = "Kadabra", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "kadabra.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "4/6 for Marsh Badge - Rare"},
["kaknua"] 			 	 		 = {["name"] = "kaknua", 			  			["label"] = "Kaknua", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "kaknua.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = ""},
["kangaskhan"] 			 	 	 = {["name"] = "kangaskhan", 			  		["label"] = "Kangaskhan", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "kangaskhan.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,   ["combinable"] = nil,   ["description"] = "Rare"},
["kingler"] 			 	 	 = {["name"] = "kingler", 			  			["label"] = "Kingler", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "kingler.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = ""},
["koffing"] 			 	 	 = {["name"] = "koffing", 			  			["label"] = "Koffing", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "koffing.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = "1/6 for Soul Badge"},
["krabby"] 				 	 	 = {["name"] = "krabby", 			  			["label"] = "Krabby", 					["weight"] = 0, 	    ["type"] = "item", 		["image"] = "krabby.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = ""},
["lapras"] 			 			 = {["name"] = "lapras", 			  			["label"] = "Lapras", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "lapras.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = "4/6 for Cascade Badge - Rare"},
["lickitung"] 			 		 = {["name"] = "lickitung", 			  		["label"] = "Lickitung", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "lickitung.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = ""},
["machamp"] 					 = {["name"] = "machamp", 					  	["label"] = "Machamp", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "machamp.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = "Rare"},
["machoke"] 					 = {["name"] = "machoke", 						["label"] = "Machoke", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "machoke.png",				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = ""},
["machop"] 			 			 = {["name"] = "machop", 						["label"] = "Machop", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "machop.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = ""},
["magikarp"] 			 		 = {["name"] = "magikarp", 						["label"] = "Magikarp", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "magikarp.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = ""},
["magmar"] 					     = {["name"] = "magmar", 						["label"] = "Magmar", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "magmar.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = "4/6 for Volcano Badge"},
["magnemite"] 			   		 = {["name"] = "magnemite", 					["label"] = "Magnemite", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "magnemite.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,    ["combinable"] = nil,   ["description"] = ""},
["magneton"] 					 = {["name"] = "magneton", 			  	  		["label"] = "Magneton", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "magneton.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "6/6 for Thunder Badge"},
["mankey"] 						 = {["name"] = "mankey", 					 	["label"] = "Mankey", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "mankey.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["marowak"] 					 = {["name"] = "marowak", 			  	  		["label"] = "Marowak", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "marowak.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["marshBadge"] 					 = {["name"] = "marshBadge", 				  	["label"] = "MarshBadge", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "marshBadge.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "6/8 for Trophy Badge"},
["meowth"] 						 = {["name"] = "meowth", 			  		 	["label"] = "Meowth", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "meowth.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["metapod"] 					 = {["name"] = "metapod", 			  	  		["label"] = "Metapod", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "metapod.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["mew"] 						 = {["name"] = "mew", 			  				["label"] = "Mew", 						["weight"] = 0, 		["type"] = "item", 		["image"] = "mew.png", 					["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Ultra Rare"},
["mewtwo"] 						 = {["name"] = "mewtwo", 			  			["label"] = "Mewtwo", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "mewtwo.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "4/6 for Earth Badge - Ultra Rare"},
["moltres"] 					 = {["name"] = "moltres", 			 			["label"] = "Moltres", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "moltres.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "6/6 for Volcano Badge - Ultra Rare"},
["mr_mime"] 					 = {["name"] = "mr_mime", 			  			["label"] = "Mr_mime", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "mr_mime.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "2/6 for Marsh Badge"},
["muk"] 						 = {["name"] = "muk", 			  				["label"] = "Muk", 						["weight"] = 0, 		["type"] = "item", 		["image"] = "muk.png", 					["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["nidoking"] 				     = {["name"] = "nidoking", 			  			["label"] = "Nidoking", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "nidoking.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "2/6 for Earth Badge - Rare"},
["nidoqueen"] 				 	 = {["name"] = "nidoqueen", 			  		["label"] = "Nidoqueen", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "nidoqueen.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "3/6 for Earth Badge"},
["nidoran"] 				 	 = {["name"] = "nidoran", 			  			["label"] = "Nidoran", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "nidoran.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["nidorina"] 				 	 = {["name"] = "nidorina", 			  			["label"] = "Nidorina", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "nidorina.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["nidorino"] 				 	 = {["name"] = "nidorino", 			  			["label"] = "Nidorino", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "nidorino.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["ninetails"] 					 = {["name"] = "ninetails", 			  		["label"] = "Ninetails", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "ninetails.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "3/6 for Volcano badge"},
["oddish"] 				 		 = {["name"] = "oddish", 			  	  		["label"] = "Oddish", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "oddish.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["omanyte"] 				 	 = {["name"] = "omanyte", 			  	  		["label"] = "Omanyte", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "omanyte.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Ultra Rare"},
["omastar"] 					 = {["name"] = "omastar", 			  	  		["label"] = "Omastar", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "omastar.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "3/6 for Boulder Badge - Ultra Rare"},
["onix"] 				 		 = {["name"] = "onix", 			  	  			["label"] = "Onix", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "onix.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "5/6 for Boulder Badge - Rare"},
["paras"] 						 = {["name"] = "paras", 			 	  	  	["label"] = "Paras", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "paras.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["parasect"] 					 = {["name"] = "parasect", 			 			["label"] = "Parasect", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "parasect.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["persian"] 					 = {["name"] = "persian", 			 			["label"] = "Persian", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "persian.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "6/6 for Earth Badge"},
["pidgeotto"] 					 = {["name"] = "pidgeotto", 					["label"] = "Pidgeotto", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "pidgeotto.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["pidgey"] 						 = {["name"] = "pidgey", 			 			["label"] = "Pidgey", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "pidgey.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["pikachu"] 					 = {["name"] = "pikachu", 						["label"] = "Pikachu", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "pikachu.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "2/6 for Thunder Badge"},
["pinsir"] 						 = {["name"] = "pinsir", 			    		["label"] = "Pinsir", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "pinsir.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["poliwag"] 					 = {["name"] = "poliwag", 			 			["label"] = "Poliwag", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "poliwag.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["polywhirl"] 				  	 = {["name"] = "polywhirl", 			 		["label"] = "Polywhirl", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "polywhirl.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["poliwrath"] 					 = {["name"] = "poliwrath", 					["label"] = "Poliwrath", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "poliwrath.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["ponyta"] 						 = {["name"] = "ponyta", 			 			["label"] = "Ponyta", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "ponyta.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["porygon"] 					 = {["name"] = "porygon", 			 			["label"] = "Porygon", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "porygon.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Ultra Rare"},
["primeape"] 				     = {["name"] = "primeape", 			 			["label"] = "Primeape", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "primeape.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["psyduck"] 					 = {["name"] = "psyduck", 						["label"] = "Psyduck", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "psyduck.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "3/6 for Cascade Badge"},
["raichu"] 						 = {["name"] = "raichu", 						["label"] = "Raichu", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "raichu.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Rare"},
["rainbowBadge"] 				 = {["name"] = "rainbowBadge", 					["label"] = "RainbowBadge", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "rainbowBadge.png", 		["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "4/6 for Rainbow Badge"},
["rapidash"] 					 = {["name"] = "rapidash", 						["label"] = "Rapidash", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "rapidash.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "2/6 for Volcano Badge"},
["raticate"] 					 = {["name"] = "raticate", 						["label"] = "Raticate", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "raticate.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["rattata"] 					 = {["name"] = "rattata", 					    ["label"] = "Rattata", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "rattata.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["rhydon"] 						 = {["name"] = "rhydon", 					    ["label"] = "Rhydon", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "rhydon.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "1/6 for Earth Badge"},
["rhyhorn"] 					 = {["name"] = "rhyhorn", 					    ["label"] = "Rhyhorn", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "rhyhorn.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "2/6 for Boulder Badge"},
["sandshrew"] 				 	 = {["name"] = "sandshrew", 				    ["label"] = "Sandshrew", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "sandshrew.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["sandslash"] 					 = {["name"] = "sandslash", 					["label"] = "Sandslash", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "sandslash.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["scyther"] 					 = {["name"] = "scyther", 						["label"] = "Scyther", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "scyther.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "4/6 for Rainbow Badge - Rare"},
["seadra"] 						 = {["name"] = "seadra", 						["label"] = "Seadra", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "seadra.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["seaking"] 		 			 = {["name"] = "seaking", 						["label"] = "Seaking", 					["weight"] = 0, 	    ["type"] = "item", 		["image"] = "seaking.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["seel"] 		 				 = {["name"] = "seel", 							["label"] = "Seel", 					["weight"] = 0, 	    ["type"] = "item", 		["image"] = "seel.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["shellder"] 		 			 = {["name"] = "shellder", 						["label"] = "Shellder", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "shellder.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["slowbro"] 		 			 = {["name"] = "slowbro", 						["label"] = "Slowbro", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "slowbro.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["slowpoke"] 		 			 = {["name"] = "slowpoke", 						["label"] = "Slowpoke", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "slowpoke.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["snorlax"] 				 	 = {["name"] = "snorlax", 			  	  		["label"] = "Snorlax", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "snorlax.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "3/6 for Soul Badge - Ultra Rare"},
["soulBadge"] 					 = {["name"] = "soulBadge", 					["label"] = "SoulBadge", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "soulBadge.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "5/8 for Trophy Badge"},
["spearow"] 				 	 = {["name"] = "spearow", 			  	  		["label"] = "Spearow", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "spearow.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["squirtle"] 				 	 = {["name"] = "squirtle", 			   			["label"] = "Squirtle", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "squirtle.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["starmie"] 					 = {["name"] = "starmie", 			  		 	["label"] = "Starmie", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "starmie.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "2/6 for Cascade Badge - Rare"},
["staryu"] 				 		 = {["name"] = "staryu", 			  	  		["label"] = "Staryu", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "staryu.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["tangela"] 					 = {["name"] = "tangela", 			  			["label"] = "Tangela", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "tangela.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "2/6 for Rainbow Badge"},
["tauros"] 						 = {["name"] = "tauros", 			  	  		["label"] = "Tauros", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "tauros.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["tentacool"] 					 = {["name"] = "tentacool", 					["label"] = "Tentacool", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "tentacool.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["tentacruel"] 					 = {["name"] = "tentacruel", 					["label"] = "Tentacruel", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "tentacruel.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = ""},
["thunderBadge"] 				 = {["name"] = "thunderBadge", 					["label"] = "ThunderBadge", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "thunderBadge.png", 		["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "3/8 for Trophy Badge"},
["togepi"] 			 			 = {["name"] = "togepi", 				 		["label"] = "Togepi", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "togepi.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "6/6 for Cascade Badge - Rare"},
["trophyBadge"] 			 	 = {["name"] = "trophyBadge", 			  		["label"] = "TrophyBadge", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "trophyBadge.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "OwO You have a Trophy"},
["vaporeon"] 			 	 	 = {["name"] = "vaporeon", 			  			["label"] = "Vaporeon", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "vaporeon.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "Rare"},
["venomoth"] 			 	 	 = {["name"] = "venomoth", 			  			["label"] = "Venomoth", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "venomoth.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "4/6 for Soul Badge"},
["venonat"] 			 	 	 = {["name"] = "venonat", 			  			["label"] = "Venonat", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "venonat.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "5/6 for Soul Badge"},
["venusaur"] 			 	 	 = {["name"] = "venusaur", 			  			["label"] = "Venusaur", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "venusaur.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "6/6 for Rainbow Badge - Rare"},
["victreebel"] 				 	 = {["name"] = "victreebel", 			  		["label"] = "Victreebel", 				["weight"] = 0, 	    ["type"] = "item", 		["image"] = "victreebel.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "1/6 for Rainbow Badge"},
["vileplume"] 		 			 = {["name"] = "vileplume", 			  		["label"] = "Vileplume", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "vileplume.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "3/6 for Rainbow Badge"},
["volcanoBadge"] 		 		 = {["name"] = "volcanoBadge", 			  		["label"] = "VolcanoBadge", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "volcanoBadge.png", 		["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "7/8 for Trophy Badge"},
["voltorb"] 					 = {["name"] = "voltorb", 					  	["label"] = "Voltorb", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "voltorb.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["vulpix"] 						 = {["name"] = "vulpix", 						["label"] = "Vulpix", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "vulpix.png",				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["wartortle"] 		 			 = {["name"] = "wartortle", 					["label"] = "Wartortle", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "wartortle.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["weedle"] 				 		 = {["name"] = "weedle", 						["label"] = "Weedle", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "weedle.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["weepinbell"] 				     = {["name"] = "weepinbell", 					["label"] = "Weepinbell", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "weepinbell.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["weezing"] 			   		 = {["name"] = "weezing", 						["label"] = "Weezing", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "weezing.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "2/6 for Soul Badge"},
["wigglytuff"] 			 		 = {["name"] = "wigglytuff", 					["label"] = "Wigglytuff", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "wigglytuff.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Rare"},
["zapdos"] 					     = {["name"] = "zapdos", 						["label"] = "Zapdos", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "zapdos.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "1/6 for Thunder Badge - Ultra Rare"},
["zubat"] 				   		 = {["name"] = "zubat", 						["label"] = "Zubat", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "zubat.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["bulbasaur"] 			   		 = {["name"] = "bulbasaur", 					["label"] = "Bulbasaur", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "bulbasaur.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = ""},
["pidgeot"]                      = {["name"] = "pidgeot", 						["label"] = "Pidgeot", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "pidgeot.png", 				["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false, 	   ["combinable"] = nil,   ["description"] = "6/6 for Earth Badge"},
["blastoisev"] 			 	 	 = {["name"] = "blastoisev", 			  		["label"] = "Blastoise V", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "blastoisev.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,	   ["combinable"] = nil,   ["description"] = "V Card"},
["charizardv"] 				 	 = {["name"] = "charizardv", 			  		["label"] = "Charizard V", 				["weight"] = 0, 	    ["type"] = "item", 		["image"] = "charizardv.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "V Card"},
["mewv"] 		 				 = {["name"] = "mewv", 			  				["label"] = "Mew V", 					["weight"] = 0, 		["type"] = "item", 		["image"] = "mewv.png", 			    ["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "V Card"},
["pikachuv"] 		 		 	 = {["name"] = "pikachuv", 			  		    ["label"] = "Pikachu V", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "pikachuv.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "V Card"},
["snorlaxv"] 					 = {["name"] = "snorlaxv", 					  	["label"] = "Snorlax V", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "snorlaxv.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "V Card"},
["venusaurv"] 					 = {["name"] = "venusaurv", 				    ["label"] = "Venusaur V", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "venusaurv.png",			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "V Card"},
["blastoisevmax"] 		 	     = {["name"] = "blastoisevmax", 			    ["label"] = "Blastoise Vmax", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "blastoisevmax.png", 		["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Vmax Card"},
["mewtwogx"] 				 	 = {["name"] = "mewtwogx", 						["label"] = "Mewtwo Vmax", 				["weight"] = 0, 		["type"] = "item", 		["image"] = "mewtwogx.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Vmax Card"},
["snorlaxvmax"] 				 = {["name"] = "snorlaxvmax", 					["label"] = "Snorlax Vmax", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "snorlaxvmax.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Vmax Card"},
["venusaurvmax"] 			   	 = {["name"] = "venusaurvmax", 					["label"] = "Venusaur Vmax", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "venusaurvmax.png", 		["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Vmax Card"},
["vmaxcharizard"] 			 	 = {["name"] = "vmaxcharizard", 				["label"] = "Charizard Vmax", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "vmaxcharizard.png", 		["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Vmax Card"},
["vmaxpikachu"] 			     = {["name"] = "vmaxpikachu", 					["label"] = "Pikachu Vmax", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "vmaxpikachu.png", 			["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Vmax Card"},
["rainbowmewtwogx"] 			 = {["name"] = "rainbowmewtwogx", 				["label"] = "Rainbow Mewtwo", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "rainbowmewtwogx.png", 		["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Rainbow Card"},
["rainbowvmaxcharizard"] 		 = {["name"] = "rainbowvmaxcharizard", 			["label"] = "Rainbow Charizard", 		["weight"] = 0, 		["type"] = "item", 		["image"] = "rainbowvmaxcharizard.png", ["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Rainbow Card"},
["rainbowvmaxpikachu"] 			 = {["name"] = "rainbowvmaxpikachu", 			["label"] = "Rainbow Pikachu", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "rainbowvmaxpikachu.png", 	["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Rainbow Card"},
["snorlaxvmaxrainbow"] 			 = {["name"] = "snorlaxvmaxrainbow", 			["label"] = "Rainbox Snorlax", 			["weight"] = 0, 		["type"] = "item", 		["image"] = "snorlaxvmaxrainbow.png", 	["unique"] = false, 	["useable"] = false, 	["shouldClose"] = false,       ["combinable"] = nil,   ["description"] = "Rainbow Card"},
["pokebox"] 					 = {["name"] = "pokebox", 						["label"] = "Pokemon TCG Box", 			["weight"] = 50, 		["type"] = "item", 		["image"] = "pokebox.png", 				["unique"] = true, 		["useable"] = true, 	["shouldClose"] = true, 	   ["combinable"] = nil,  ["description"] = "Pokemon TCG Storage Box"},


## you may change shop location coords or the maps that i use for the shops are linked below

https://modit.store/collections/gtav-maps-mlo-shell/products/pawn-and-jewelry-mlo-interior?variant=38162504024247#

https://www.youtube.com/watch?v=KSfuP583gV8&feature=youtu.be

I'd Like to also point out that I am not to be recognized as sole creator of this project.
I am still learning and got tons of help from a great team of guys!
Big thanks to anyone that helped me if you want any sort of individual recognition say the word!

kamui_kody for support on script join https://discord.gg/3j9b439TeY

