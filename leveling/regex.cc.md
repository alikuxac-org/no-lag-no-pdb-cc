# Regex

**Recommend Trigger**: `Regex` - **Trigger Text**: `.*`\
**Description**: This command manages messages - setting cooldowns, giving role rewards when users level up, and giving XP.\
**Usage**: None\
**Code**: below

```lua
{{ $isExclude:=false }}
{{ $settings := 0 }}{{ $cooldown := false }}{{ $roleRewards := sdict "type" "stack" }}
{{ with (dbGet 0 "xpSettings") }}{{ $settings = sdict .Value }}{{ end }}
{{ with (dbGet 0 "roleRewards") }}{{ $roleRewards = sdict .Value }}{{ end }}
{{ $exclude:= sdict ($settings.Get "exclude") }}
{{ $userex:=$settings.Get "user" }}
{{ $chanex:=$settings.Get "channel" }}
{{ $roleex:=$settings.Get "role" }}
{{ $cateex:=$settings.Get "category" }}
{{ range .Member.Roles }}{{- if inFold $roleex (str .) -}}{{ $isExclude =true -}}{{ end }}{{ end }}
{{  if inFold $userex (str .User.ID) }}{{ $isExclude =true }}{{ end }}
{{  if inFold $chanex (str .Channel.ID) }}{{ $isExclude =true }}{{ end }}
{{  if inFold $cateex (str .Channel.ParentID) }}{{ $isExclude =true }}{{ end }}

{{  if (dbGet .User.ID "xpCooldown")  }} {{  $cooldown = true  }} {{  end  }}
{{ if and (not $cooldown) $settings  }}
	{{ $giveXP := randInt $settings.min $settings.max }}
	{{ $currentXp := 0 }}{{ with (dbGet .User.ID "xp") }}{{ $currentXp = .Value }}{{ end }}
	{{ $currentLvl := roundFloor (mult 0.1 (sqrt $currentXp)) }}
	{{ $newXp := dbIncr $user.ID "xp" $giveXP }}
	{{ $newLvl := roundFloor (mult 0.1 (sqrt $newXp)) }}
	{{ $channelstg := or $settings.channel .Channel.ID }}
	{{ if not (getChannel $channelstg) }}{{ $channelstg = .Channel.ID }}{{ end }}

	{{ if ne $newLvl $currentLvl }}
		{{ $type := $roleRewards.type }}
		{{ $toAdd := or ($roleRewards.Get (json $newLvl)) 0 }}
		{{ - range $level, $reward := $roleRewards - }}
			{{ if and (ge (toInt $newLvl) (toInt $level)) (not (hasRoleID $reward)) (eq $type "stack") (ne $level "type") }}{{ addRoleID $reward }}
			{{ else if and (hasRoleID $reward) (eq $type "highest") $toAdd }}{{ removeRoleID $reward }}{{ end }}
		{{ - end - }}
		{{ if $toAdd }}{{ addRoleID $toAdd }}{{ end }}
		{{ $embed := cembed "title" "❯ Level up!"
			"thumbnail" (sdict "url" "https://webstockreview.net/images/emoji-clipart-celebration-4.png")
			"description" (printf "Congrats **%s**! You just leveling to %d " (or .User.String .Member.Nick) (toInt $newLvl))
			"color" 14232643
		 }}
		{{ if eq $settings.announce "true" }}{{ sendMessage $channelstg (complexMessage "content" .User.Mention "embed" $embed) }}{{ end }}
	{{ end }}

	{{  $cooldownSeconds := div $settings.cooldown 1000000000  }} {{ /* Convert cooldown to seconds */ }}
	{{  dbSetExpire .User.ID "xpCooldown" true $cooldownSeconds  }} {{ /* Set cooldown entry */ }}
{{ end }}
```
