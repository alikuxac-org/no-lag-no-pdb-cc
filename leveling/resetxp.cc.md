# ResetXP

- Description: This command reset specific User XP.
- Usage: `-resetxp <@Mention|ID>`
- Trigger type: **Command**
- Trigger text: `resetxp`

# Code
```lua
{{ if reFind `ManageServer` (exec "viewperms") }}
{{ $user := 0 }} {{ /* Target user */ }}
{{ with .CmdArgs }} {{ $user = index . 0 | userArg }} {{ end }} {{ /* We try to resolve user from arguments given */ }}
{{ if eq (len .CmdArgs) 1 }}
	{{ dbDel $user.ID "xp" }}
	{{ printf "Successfully reset **%s**'s XP !" $user.String }}
{{ else }}
	The syntax for this command is `-resetxp <user>`.
{{ end }}
{{ else }}
	You dont have `Manage Server` Permission to run this command.
{{ end }}
```
