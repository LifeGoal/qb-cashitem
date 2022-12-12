# qb-cashitem
Just some shitty code to make cash usable. No support will be given, if you have any problems, please feel free to make feature-issues to the QBCore crew to make this themselves.

## qb-core/server/player.lua
```lua
  function self.Functions.RemoveMoney(moneytype, amount, reason)
        reason = reason or 'unknown'
        moneytype = moneytype:lower()
        amount = tonumber(amount)
        if amount < 0 then return end
        if moneytype == 'cash' then
            if self.Functions.GetItemByName('cash') then
                if self.Functions.GetItemByName('cash').amount >= amount then
                    self.Functions.RemoveItem('cash', amount)
                    self.Functions.UpdatePlayerData()
                else
                    return false
                end
            else
                return false
            end
        else
            if self.PlayerData.money[moneytype] then
                for _, mtype in pairs(QBCore.Config.Money.DontAllowMinus) do
                    if mtype == moneytype then
                        if (self.PlayerData.money[moneytype] - amount) < 0 then return false end
                    end
                end
                self.PlayerData.money[moneytype] = self.PlayerData.money[moneytype] - amount
            else
                return false
            end
        end

        self.Functions.UpdatePlayerData()
        if amount > 100000 then
            TriggerEvent('qb-log:server:CreateLog', 'playermoney', 'RemoveMoney', 'red', '**' .. GetPlayerName(self.PlayerData.source) .. ' (citizenid: ' .. self.PlayerData.citizenid .. ' | id: ' .. self.PlayerData.source .. ')** $' .. amount .. ' (' .. moneytype .. ') removed, new ' .. moneytype .. ' balance: ' .. self.PlayerData.money[moneytype], true)
        else
            TriggerEvent('qb-log:server:CreateLog', 'playermoney', 'RemoveMoney', 'red', '**' .. GetPlayerName(self.PlayerData.source) .. ' (citizenid: ' .. self.PlayerData.citizenid .. ' | id: ' .. self.PlayerData.source .. ')** $' .. amount .. ' (' .. moneytype .. ') removed, new ' .. moneytype .. ' balance: ' .. self.PlayerData.money[moneytype])
        end
        TriggerClientEvent('hud:client:OnMoneyChange', self.PlayerData.source, moneytype, amount, true)
        if moneytype == 'bank' then
            TriggerClientEvent('qb-phone:client:RemoveBankMoney', self.PlayerData.source, amount)
        end
        return true
    end

    function self.Functions.SetMoney(moneytype, amount, reason)
        reason = reason or 'unknown'
        moneytype = moneytype:lower()
        amount = tonumber(amount)
        if amount < 0 then return false end
        if moneytype == 'cash' then
			if self.Functions.GetItemByName('cash') then
				local playerCash = self.Functions.GetItemByName('cash').amount
				self.Functions.RemoveItem('cash', playerCash)
				self.Functions.AddItem('cash', amount)
			else
				self.Functions.AddItem('cash', amount)
			end
		elseif self.PlayerData.money[moneytype] then
			self.PlayerData.money[moneytype] = amount
		end

        self.Functions.UpdatePlayerData()
        TriggerEvent('qb-log:server:CreateLog', 'playermoney', 'SetMoney', 'green', '**' .. GetPlayerName(self.PlayerData.source) .. ' (citizenid: ' .. self.PlayerData.citizenid .. ' | id: ' .. self.PlayerData.source .. ')** $' .. amount .. ' (' .. moneytype .. ') set, new ' .. moneytype .. ' balance: ' .. self.PlayerData.money[moneytype])
        return true
    end

    function self.Functions.GetMoney(moneytype)
        if not moneytype then return false end
        moneytype = moneytype:lower()
        if moneytype == "cash" then
            return self.Functions.GetItemByName('cash').amount
        else
            return self.PlayerData.money[moneytype]
        end
    end
```

## Code to automatically update cash from playerdata to items
```lua
Citizen.CreateThread(function()
	Wait(10000)
    exports['oxmysql']:execute('SELECT * FROM players', {}, function(result)
		for k, v in pairs(result) do
			local Player = QBCore.Functions.GetOfflinePlayerByCitizenId(v.citizenid)
			if Player.PlayerData.money.cash > 0 then
				table.insert(Player.PlayerData.items, {
					info = "",
					name = "cash",
					slot = (#Player.PlayerData.inventory + 1),
					type = "item",
					amount = math.floor(Player.PlayerData.money.cash + 0.5)
				})
				Player.PlayerData.money.cash = 0
				QBCore.Player.SaveOffline(Player.PlayerData)
			end
			Wait(100)
		end
	end)
end)
```
