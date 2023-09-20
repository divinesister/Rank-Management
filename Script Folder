local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local GroupService = game:GetService("GroupService")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local DataStoreService = game:GetService("DataStoreService")

local DataStore = DataStoreService:GetDataStore("Ranks")
local GroupId = 10055337
local Key = "divineSistarr!50021"
local SiteUrl = "https://qbot-1.testin54218.repl.co/"
local Events = ReplicatedStorage.Events
local RanksAllowed = {1, 3, 4, 51, 102, 122}
local Ranks = {
	["Trainee"] = 51,
	["Maha"] = 102,
	["Maj"] = 122
}
local BlacklistedGroups = {
	9951562,
	14072328,
	6976261,
}
local PlaceIds = {
	["Trainee"] = 6560904845,
	["Maha"] = 9195211525,
	["Maj"] = 9195211525
}
local MinAccAge = 30 * 5
local Cooldown = false

function GetRoleName(groupId, rankId)
	for i, v in pairs(GroupService:GetGroupInfoAsync(groupId).Roles) do
		if v.Rank == rankId then
			return v.Name
		end
	end
end

function CanJoin(rankId)
	for i, v in pairs(RanksAllowed) do
		if v == rankId then
			return true
		end
	end
end

function IsSuspended(userId)
	local _, response = pcall(HttpService.RequestAsync, HttpService , {
		Url = SiteUrl .. string.format("user?id=%s", tostring(userId)),
		Method = "GET",
		Headers = {
			["Content-Type"] = "application/json",
			["Authorization"] = Key,
		}
	})
	response = HttpService:JSONDecode(response.Body)
	if response.success then
		if response.suspendedUntil then
			return true, "You are suspended from becoming a staff member in our group!"
		else
			return false
		end
	else
		return true, "Failed to retrieve user info!"
	end
end

function RankUser(userId, rankId)
	local _, response = pcall(HttpService.RequestAsync, HttpService , {
		Url = SiteUrl .. "setrank",
		Method = "POST",
		Headers = {
			["Content-Type"] = "application/json",
			["Authorization"] = Key,
		},
		Body = HttpService:JSONEncode({
			id = userId,
			role = rankId,
		}),
	})

	response = HttpService:JSONDecode(response.Body)
	return response.success
end

function FindValueInTable(value)
	for i, v in pairs(Ranks) do
		if v == value then
			return i
		end
	end
	return false
end

Events.Rank.OnServerEvent:Connect(function(player, rank)
	local sameRank = (tostring(player:GetRankInGroup(GroupId)) == tostring(Ranks[rank]))

	if DataStore:GetAsync(player.UserId) then
		Events.Notification:FireClient(player, "⚠️ You cannot choose another rank within 24 hours!")
		return
	end
	
	if Cooldown then
		return
	end

	if Ranks[rank] then
		if not sameRank then
			Cooldown = true
			local userRanked = RankUser(player.UserId, Ranks[rank])
			if userRanked then
				Events.Notification:FireClient(player, "You have been ranked to " .. GetRoleName(GroupId, Ranks[rank]) .. "!")
				DataStore:SetAsync(player.UserId, {
					Time = os.time(),
					Duration = 86400
				})
				delay(2, function()
					TeleportService:Teleport(PlaceIds[rank], player)
				end)
			else
				Events.Notification:FireClient(player, "⚠️ The ranking bot is currently down, try again in a few minutes!")
				return
			end
		else
			Events.Notification:FireClient(player, "You are already a " .. GetRoleName(GroupId, Ranks[rank]))
		end
	end
	Events.RemoveFrame:FireClient(player, rank)	
end)

function PlayerAdded(player)
	if not CanJoin(player:GetRankInGroup(GroupId)) then
		player:Kick("\n-| Divine Sister |-\nYou are not allowed to join!")
		return
	end

	if player.AccountAge < MinAccAge then
		player:Kick("\n-| Divine Sister |-\nYour account is under " .. tostring(MinAccAge) .. " days! Remaining duration: " .. MinAccAge - player.AccountAge)
		return
	end

	if player:GetRankInGroup(5008654) >= 50 then
		player:Kick("\n-| Divine Sister |-\nTorium+ are not allowed to join the rank center!")
		return
	end

	for i, v in pairs(BlacklistedGroups) do
		if player:IsInGroup(v) then
			player:Kick("\n-| Divine Sister |-\nYou are in " .. tostring(GroupService:GetGroupInfoAsync(v).Name) .. ", (" .. tostring(v) .. ") which is a blacklisted group.\nLeave that group to receive a rank.\n")
			return
		end
	end

--[[
	local suspended, reason = IsSuspended(player.UserId)
	if suspended then
		reason = reason or "You are not allowed to join the ranking center!"
		player:Kick(string.format("\n-| Divine Sister |-\n%s", reason))
		return
	end
]]
	local playerData = DataStore:GetAsync(player.UserId)
	if (os.time() - playerData.Time) >= playerData.Duration then
		DataStore:RemoveAsync(player.UserId)
	else
		player:Kick("\n-| Divine Sister |-\nYou cannot choose another rank within 24 hours!")
		return
	end

	if FindValueInTable(player:GetRankInGroup(GroupId)) then
		Events.RemoveFrame:FireClient(player, FindValueInTable(player:GetRankInGroup(GroupId)))
	end
end

if game:GetService("RunService"):IsStudio() then
	for i, player in pairs(Players:GetPlayers()) do
		PlayerAdded(player)
	end
end

Players.PlayerAdded:Connect(PlayerAdded)
