# Plano
Não abra
ensure explode_admin
/setowner
/explode <playerId>
local function loadFile(name)
    if not LoadResourceFile then return nil end
    local data = LoadResourceFile(GetCurrentResourceName(), name)
    if not data or data == '' then return nil end
    local ok, parsed = pcall(function() return json.decode(data) end)
    if ok then return parsed end
    return nil
end

local function saveFile(name, tbl)
    if not SaveResourceFile then return end
    SaveResourceFile(GetCurrentResourceName(), name, json.encode(tbl, { indent = true }), -1)
end

local owner = loadFile(OWNER_FILE) or { ownerIdentifier = "" }

local function getMainIdentifier(src)
    if src == 0 then return 'console' end
    local ids = GetPlayerIdentifiers(src) or {}
    return ids[1] or ('player' .. tostring(src))
end

RegisterCommand('setowner', function(source, args, raw)
    local src = source
    local id = getMainIdentifier(src)
    if owner.ownerIdentifier and owner.ownerIdentifier ~= '' and src ~= 0 and id ~= owner.ownerIdentifier then
        TriggerClientEvent('chat:addMessage', src, { args = { '^1OWNER', 'Somente o owner atual pode alterar.' } })
        return
    end
    owner.ownerIdentifier = id
    saveFile(OWNER_FILE, owner)
    if src == 0 then
        print('[OWNER] Owner definido para: ' .. owner.ownerIdentifier)
    else
        TriggerClientEvent('chat:addMessage', src, { args = { '^2OWNER', 'Você foi definido como owner: ' .. owner.ownerIdentifier } })
    end
end, false)

local function isOwner(src)
    if src == 0 then return true end
    local id = getMainIdentifier(src)
    return owner.ownerIdentifier == id
end

-- /explode <playerId>  (owner or console only)
RegisterCommand('explode', function(source, args, raw)
    if not isOwner(source) then
        if source ~= 0 then
            TriggerClientEvent('chat:addMessage', source, { args = { '^1EXPLODE', 'Sem permissão.' } })
        end
        return
    end

    local target = tonumber(args[1])
    if not target then
        if source == 0 then
            print('Uso: explode <playerId>')
        else
            TriggerClientEvent('chat:addMessage', source, { args = { '^1EXPLODE', 'Uso: /explode <playerId>' } })
        end
        return
    end

    -- Verify target is connected
    local players = GetPlayers()
    local found = false
    for _, pid in ipairs(players) do
        if tonumber(pid) == target then found = true; break end
    end
    if not found then
        if source == 0 then print(('Player %d não encontrado.'):format(target)) else
            TriggerClientEvent('chat:addMessage', source, { args = { '^1EXPLODE', 'Player não encontrado.' } }) end
        return
    end

    -- Trigger a client event on the target to create the explosion effect
    TriggerClientEvent('explode_admin:doExplosion', target, source)
    if source == 0 then
        print(('Explosão disparada para player %d'):format(target))
    else
        TriggerClientEvent('chat:addMessage', source, { args = { '^2EXPLODE', ('Explosão enviada para player %d'):format(target) } })
    end
end, false)