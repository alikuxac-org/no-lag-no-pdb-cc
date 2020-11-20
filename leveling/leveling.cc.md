# Leveling

- Description: This command manages the general level settings of the guild.
- Usage:
  `-leveling set <key> value` | example: -leveling set cooldown 1 minute 30 seconds
  `-leveling use-default` | use default settings
  `-leveling view` | view settings
- Trigger type: **Regex**.
- Trigger text: `\A(-|<@!?204255221017214977>)\s*(leveling|(level|lvl)-?conf|(level|lvl)-?settings)(\s+|\z)`

- **Note**: If you are using leveling system from [Joe's repository](https://github.com/Jo3-L/yagpdb-cc/tree/master/leveling), just run `-leveling set announce true`.

# Code

```lua
{{ $helpMsg := cembed
	"title" "🏆 Leveling"
	"description" (joinStr "\n\n"
		"`leveling use-default`: Use the default settings"
		"`leveling set <key> <value>`: Sets the given settings to the value provided. Valid keys are \"min\", \"max\", \"announce\" and \"cooldown\" (duration)."
		"`leveling set-channel <channel|none>`: Sets the channel where level up messages will be sent (defaults to current channel). If you want to make it the current channel, use `leveling set-channel none`."
		"`leveling view`: Views the current settings."
	)
	"color" 14232643
 }}
{{ if .CmdArgs }}
	{{ $isSaved := false }} {{/* Whether the settings are saved */}}
	{{ $currentSettings := sdict
		"min" 15
		"max" 25
		"cooldown" .TimeMinute
		"announce" "true"
		"exclude" (sdict "user" (cslice) "role" (cslice) "channel" (cslice) "category" (cslice))
	}} {{/* Defaults for level settings */}}
	{{ with (dbGet 0 "xpSettings") }}
		{{ $isSaved = true }} {{/* Settings are in DB */}}
		{{ $currentSettings = sdict .Value }} {{/* Convert value to sdict */}}
	{{ end }}

	{{ if eq (index .CmdArgs 0) "use-default" }}
		{{ $s := dbSet 0 "xpSettings" $currentSettings }} {{/* Set defaults */}}
		Done! You are now using the default settings for the leveling system.

	{{ else if and (eq (index .CmdArgs 0) "set") (ge (len .CmdArgs) 3) }}
		{{ $key := index .CmdArgs 1 }} {{/* The key of the setting being set */}}
		{{ $value := slice .CmdArgs 2 | joinStr " " }} {{/* The value of the new setting */}}
		{{ if in (cslice "min" "max" "cooldown" "announce") $key }} {{/* Check that key is valid */}}
			{{$parsed := or (and (eq $key "cooldown") (toDuration $value)) (toInt $value) }} {{/* Find the proper type of conversion needed */}}
			{{if eq $key "announce"}}{{$parsed = str $value}}{{end}}
			{{ if not $parsed }} {{/* Check whether it was parsed correctly / whether it was valid value */}}
				Please provide a valid value for the key `{{ $key }}`.
			{{ else }}
				{{ $currentSettings.Set $key $parsed }} {{/* Set key to value */}}
				{{ if ge $currentSettings.min $currentSettings.max }} {{/* Preemptively prevent user from setting larger min value than max which would cause error later */}}
					The minimum xp cannot be larger than or equal to the max xp.
				{{ else }}
					{{ $s := dbSet 0 "xpSettings" $currentSettings }} {{/* Save it */}}
					Successfully set the key `{{ $key }}` to `{{ $value }}`!
				{{ end }}
			{{ end }}
		{{ else }}
			That was not a valid key. The only valid settings are "min", "max", "announce", "cooldown" and "announce".
		{{ end }}

	{{ else if and (eq (index .CmdArgs 0) "set-channel") (ge (len .CmdArgs) 2) }}
		{{ $input := index .CmdArgs 1 }}
		{{ with reFindAllSubmatches `<#(\d+)>` $input }} {{ $input = toInt64 (index . 0 1) }} {{ end }}
		{{ $channel := getChannel $input }}
		{{ if $channel }}
			{{ $currentSettings.Set "channel" $channel.ID }}
			{{ $s := dbSet 0 "xpSettings" $currentSettings }}
			Successfully set channel to <#{{ $channel.ID }}>!
		{{ else if eq $input "none" }}
			{{ $currentSettings.Del "channel" }}
			{{ $s := dbSet 0 "xpSettings" $currentSettings }}
			Successfully set the channel for level up notifications to none.
		{{ else }}
			That was not a valid channel. Try again.
		{{ end }}

	{{ else if eq (index .CmdArgs 0) "view" }}
		{{ $channel := "*None set*" }}
		{{ with $currentSettings.channel }} {{ $channel = printf "<#%d>" . }} {{ end }}
		{{ $formatted := printf "**❯ Minimum XP:** %d\n**❯ Maximum XP:** %d\n**❯ Cooldown:** %s\n**❯ Level-up Channel:** %s\n**❯ Announce:** %s"
			$currentSettings.min
			$currentSettings.max
			(humanizeDurationSeconds ($currentSettings.cooldown | toDuration))
			$channel
			$currentSettings.announce
		 }} {{/* Construct the embed description */}}
		{{ if $isSaved }} {{/* If the settings are in DB */}}
			{{ sendMessage nil (cembed "title" "Level Settings" "description" $formatted "thumbnail" (sdict "url" "https://i.imgur.com/mJ7zu6k.png")) }}
		{{ else }}
			This server has not set up the leveling system. Run `-leveling use-default` to use the default settings or customize it using `-leveling set <key> <value>`.
		{{ end }}
	{{ else }} {{/* Send help messages */}}
		{{ sendMessage nil $helpMsg }}
	{{ end }}
{{ else }}
	{{ sendMessage nil $helpMsg }}
{{ end }}
```
