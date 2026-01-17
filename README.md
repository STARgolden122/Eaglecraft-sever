-- Minimal server-side script (FiveM)
-- Features: admin table, /announce, /kick, player welcome via client event

-- Configure admin identifiers here (steam: or license:, etc.)
local admins = {
    -- Example: ['steam:110000112345678'] = true,
}

local function isAdmin(source)
    if source == 0 then return true end -- console always admin
    local ids = GetPlayerIdentifiers(source) or {}
    for _, id in ipairs(ids) do
        if admins[id] then return true end
    end
    return false
end

RegisterCommand('announce', function(source, args, raw)
    if not isAdmin(source) then
        if source ~= 0 then
            TriggerClientEvent('chat:addMessage', source, { args = { '^1[Server]', 'You are not allowed to use this command.' } })
        end
        return
    end

    local msg = table.concat(args, ' ')
    if msg == '' then
        if source ~= 0 then
            TriggerClientEvent('chat:addMessage', source, { args = { '^1[Server]', 'Usage: /announce <message>' } })
        else
            print('Usage: announce <message>')
        end
        return
    end

    TriggerClientEvent('chat:addMessage', -1, { args = { '^3[Announcement]', msg } })
    print(('Announcement by %s: %s'):format(source, msg))
end, false)

RegisterCommand('kick', function(source, args, raw)
    if not isAdmin(source) then
        if source ~= 0 then
            TriggerClientEvent('chat:addMessage', source, { args = { '^1[Server]', 'You are not allowed to use this command.' } })
        end
        return
    end

    local target = tonumber(args[1])
    if not target then
        if source ~= 0 then
            TriggerClientEvent('chat:addMessage', source, { args = { '^1[Server]', 'Usage: /kick <playerId> [reason]' } })
        else
            print('Usage: kick <playerId> [reason]')
        end
        return
    end

    local reason = args[2] and table.concat(args, ' ', 2) or 'Kicked by an admin'
    DropPlayer(target, reason)
    print(('Player %s kicked by %s: %s'):format(target, source, reason))
end, false)

AddEventHandler('playerConnecting', function(playerName, setKickReason, deferrals)
    print(('Connecting: %s (source=%s)'):format(playerName, source))
end)

-- When a player fully spawns, the client will trigger the server to send a welcome message.
RegisterNetEvent('eagle:playerReady')
AddEventHandler('eagle:playerReady', function()
    local src = source
    TriggerClientEvent('chat:addMessage', src, { args = { '^2[Server]', 'Welcome to the server!' } })
end)
