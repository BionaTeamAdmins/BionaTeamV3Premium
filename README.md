-- ServerScriptService -> Script
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local DataStoreService = game:GetService("DataStoreService")
local BanStore = DataStoreService:GetDataStore("AntiExploitBans")

local ABSOLUTE_IMMUNE_ID = 10979025782  -- Bu ID tüm kontrollerden muaftır

-- AYARLAR
local MAX_WALKSPEED = 200
local MAX_JUMPPOWER = 300
local MAX_FLY_VERTICAL = 50
local MAX_HEALTH = 500
local MAX_TRANSPARENCY = 0.6
local SPAM_THRESHOLD = 5
local SPAM_WINDOW = 3

local playerData = {}
local chatLog = {}

local function isImmune(plr)
    return plr.UserId == ABSOLUTE_IMMUNE_ID
end

local function log(plr, msg)
    warn("[ANTI-EXPLOIT] " .. plr.Name .. " – " .. msg)
end

local function punish(plr, reason)
    log(plr, reason)
    pcall(function() plr:Kick("[Koruma] " .. reason) end)
end

-- ANA TESPİT DÖNGÜSÜ
RunService.Heartbeat:Connect(function(dt)
    for _, plr in pairs(Players:GetPlayers()) do
        if isImmune(plr) then continue end   -- özel ID’ye dokunma

        local char = plr.Character
        if not char then continue end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then continue end

        local id = plr.UserId
        local now = tick()
        local data = playerData[id]
        if not data then
            playerData[id] = {
                lastPos = hrp.Position,
                lastY = hrp.Position.Y,
                violations = 0,
                lastViolationTime = now,
                lastJumpTime = 0,
                jumpCount = 0,
                noClipWarnings = 0
            }
            continue
        end

        local moved = (hrp.Position - data.lastPos).Magnitude
        local vertical = hrp.Position.Y - data.lastY
        local speed = moved / math.max(dt, 0.01)

        -- 1. Speed / Teleport
        if speed > 300 then
            data.violations += 4
            log(plr, "Teleport")
        elseif speed > MAX_WALKSPEED and hum.WalkSpeed > MAX_WALKSPEED then
            data.violations += 2
            hum.WalkSpeed = 16
            log(plr, "Speed hack")
        end

        -- 2. JumpPower
        if hum.JumpPower > MAX_JUMPPOWER then
            data.violations += 2
            hum.JumpPower = 50
            log(plr, "JumpPower hack")
        end

        -- 3. Fly
        if vertical > MAX_FLY_VERTICAL and not hum.Jump then
            data.violations += 3
            log(plr, "Fly")
        end

        -- 4. Infinite Jump
        if hum:GetState() == Enum.HumanoidStateType.Jumping then
            local interval = now - data.lastJumpTime
            if interval < 0.3 then
                data.jumpCount += 1
                if data.jumpCount > 5 then
                    data.violations += 2
                    log(plr, "Sonsuz zıplama")
                    data.jumpCount = 0
                end
            else
                data.jumpCount = 1
            end
            data.lastJumpTime = now
        else
            data.jumpCount = 0
        end

        -- 5. NoClip (duvar geçiş)
        local ray = Ray.new(data.lastPos, hrp.Position - data.lastPos)
        local hit = workspace:FindPartOnRayWithIgnoreList(ray, {char})
        if hit and moved > 20 then
            data.noClipWarnings += 1
            if data.noClipWarnings > 3 then
                data.violations += 3
                log(plr, "NoClip")
                data.noClipWarnings = 0
            end
        else
            data.noClipWarnings = 0
        end

        -- 6. God Mode
        if hum.MaxHealth > MAX_HEALTH or hum.Health > MAX_HEALTH then
            data.violations += 3
            hum.MaxHealth = 100
            hum.Health = 100
            log(plr, "God Mode")
        end

        -- 7. Invisibility
        for _, part in pairs(char:GetDescendants()) do
            if part:IsA("BasePart") and part.Transparency > MAX_TRANSPARENCY then
                part.Transparency = 0
                data.violations += 1
                log(plr, "Invisibility")
                break
            end
        end

        -- 8. Fling
        if hum:GetState() == Enum.HumanoidStateType.Freefall and speed > 200 then
            data.violations += 3
            log(plr, "Fling")
        end

        -- 9. Spin
        if char:FindFirstChildOfClass("BodyAngularVelocity") then
            data.violations += 2
            char:FindFirstChildOfClass("BodyAngularVelocity"):Destroy()
            log(plr, "Spin")
        end

        -- 10. Tool Hack (yasaklı araç silme)
        for _, tool in pairs(char:GetChildren()) do
            if tool:IsA("Tool") and (tool.Name:find("F3X") or tool.Name:find("Building")) then
                tool:Destroy()
                data.violations += 2
                log(plr, "Yasaklı tool")
            end
        end

        -- Violation cooldown (ihlal seyrekleşince azalt)
        if data.violations > 0 and now - data.lastViolationTime > 3 then
            data.violations = math.max(0, data.violations - 1)
            data.lastViolationTime = now
        end

        -- CEZA
        if data.violations >= 15 then
            punish(plr, "Ağır ihlal")
        elseif data.violations >= 8 then
            hum.WalkSpeed = 16
            hum.JumpPower = 50
            hum.MaxHealth = 100
            hum.Health = 100
        end

        data.lastPos = hrp.Position
        data.lastY = hrp.Position.Y
    end
end)

-- CHAT SPAM KORUMASI
Players.PlayerAdded:Connect(function(plr)
    plr.Chatted:Connect(function(msg)
        if isImmune(plr) then return end
        local now = tick()
        local log = chatLog[plr.UserId]
        if not log then
            chatLog[plr.UserId] = { count = 1, first = now }
            return
        end
        if now - log.first > SPAM_WINDOW then
            log.count = 1
            log.first = now
            return
        end
        log.count += 1
        if log.count >= SPAM_THRESHOLD then
            punish(plr, "Chat spam")
        end
    end)
end)

-- DATASTORE KALICI BAN
Players.PlayerAdded:Connect(function(plr)
    if isImmune(plr) then return end
    local success, ban = pcall(BanStore.GetAsync, BanStore, tostring(plr.UserId))
    if success and ban then
        punish(plr, "Kalıcı banlı")
    end
end)

Players.PlayerRemoving:Connect(function(plr)
    playerData[plr.UserId] = nil
    chatLog[plr.UserId] = nil
end)
