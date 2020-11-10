# Set XP/Level

- Description: This command set xp/level of specific user.
- Usage: `-setxp/setlv <user> <number of xp|level>`
- Trigger type: **Regex**
- Trigger text: `\A(-|<@!?204255221017214977>)\s*(set-?(?:xp|level))(\s+|\z)`

# Code

```lua
{{ $cmd := reFind `(?i)xp|level` .Cmd }}
{{ $user := 0 }}
{{ $newLvl := 0 }}
{{ $oldLvl := 0}}
{{ with .CmdArgs }} {{ $user = index . 0 | userArg }} {{ end }} {{/* We try to resolve user from arguments given */}}
{{ if and (eq $cmd "level") (eq (len .CmdArgs) 2) }}
	{{ $level := index .CmdArgs 1 | toInt }} {{/* The new level */}}
    {{ if and $level $user }}
        {{ $oldXp := 0 }} {{/* Old XP */}}
        {{ with (dbGet $user.ID "xp") }} {{ $oldXp = .Value }} {{ end }} {{/* Update old xp with db entry */}}
        {{ $oldLvl := roundFloor (mult 0.1 (sqrt $oldXp)) }}
		{{ $calculated := mult 100 (mult $level $level) }} {{/* Calculate XP for this level */}}
		{{ $s := dbSet $user.ID "xp" $calculated }} {{/* Update db */}}
		{{ if ne (toInt $oldLvl) (toInt $level) }}{{ $newLvl = $level }} {{ end }}
		{{ printf "Successfully set **%s**'s level to %d!" $user.String $level }}
	{{ else }}
		The syntax for this command is `-setlevel <user> <level>`.
	{{ end }}
{{ else if eq (len .CmdArgs) 2 }}
	{{ $xp := index .CmdArgs 1 | toInt }} {{/* The new XP */}}
    {{ if and $xp $user }}
        {{ $oldXp := 0 }} {{/* Old XP */}}
        {{ with (dbGet $user.ID "xp") }} {{ $oldXp = .Value }} {{ end }} {{/* Update old xp with db entry */}}
        {{ $oldLvl := roundFloor (mult 0.1 (sqrt $oldXp)) }}
		{{ $s := dbSet $user.ID  "xp" $xp }} {{/* Silence the dbSet */}}
		{{ $updatedLvl := roundFloor (mult 0.1 (sqrt $xp)) }} {{/* Calculate updated level */}}
		{{ if ne $oldLvl $updatedLvl }} {{ $newLvl = $updatedLvl }} {{ end }} {{/* If level was updated, update newLvl */}}
		{{ printf "Successfully set **%s**'s XP to %d!" $user.String $xp }}
	{{ else }}
		The syntax for this command is `-setxp <user> <xp>`.
	{{ end }}
{{ else }}
	The syntax for this command is `-setxp/setlevel <user> <level|xp>`.
{{ end }}

{{/* Handle leveling up | Basically the same as whats in the message listener, so if you are curious look at that */}}
{{ if $newLvl }}
    {{ execCC 40 nil 0 (sdict "leveling" 1 "user" $user.ID) }}
    {{ $roleRewards := sdict "type" "stack" }}
    {{ with dbGet 0 "roleRewards" }} {{ $roleRewards = sdict .Value }} {{ end }}
    {{ $type := $roleRewards.type }}
    {{ $toAdd := 0 }}
    {{ $dist := -1 }}
	{{ range $level, $reward := $roleRewards }}
		{{ if and (le (toInt $level) (toInt $newLvl)) (or (gt $dist (sub (toInt $newLvl) (toInt $level))) (eq $dist -1)) (eq $type "highest") }}
			{{ $dist = sub (toInt $newLvl) (toInt $level) }} {{ $toAdd = $reward }}
		{{ end }}
		{{ if and (ge (toInt $newLvl) (toInt $level)) (not (targetHasRoleID $user.ID $reward)) (eq $type "stack") (ne $level "type") }} {{ giveRoleID $user.ID $reward }}
		{{ else if and (targetHasRoleID $user.ID $reward) (eq $type "highest") }} {{ takeRoleID $user.ID $reward }} {{ end }}
	{{ end }}
	{{ if $toAdd }} {{ giveRoleID $user.ID $toAdd }} {{ end }}
{{ end }}
```
