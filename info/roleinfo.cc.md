# Roleinfo

- Description: Show information of specific role.
- Usage:
  `-roleinfo RoleName` | Check role infomation by input Name.
  `-roleinfo RoleID` | Check role infomation by input ID.
  `-roleinfo RoleMention` | Check role infomation by mention role.
- Trigger type: **Command**
- Trigger text: `roleinfo`

```lua
{{$exactRole := 0}}{{$maybeRole := 0}}{{$perm := ""}}{{$hoist := "False"}}{{$managed := "False"}}{{$mention := "False"}}
{{$permissions := cslice "Create Instant Invite" "Kick Members" "Ban Members" "Administrator" "Manage Channels" "Manage Guild" "Add Reactions" "View Audit Log" "Priority Speaker" "Stream" "View Channel" "Send Message" "Send TTS Message" "Manage Messages" "Embed Links" "Attach Files" "Read Message History" "Mention Everyone" "Use External Eemoji" "" "Connect" "Speak" "Mute Members" "Deafen Members" "Move Member" "Use VAD" "Change Nickname" "Manage Nicknames" "Manage Roles" "Manage Webhooks" "Manage Emojis"}}
{{if .CmdArgs}}
    {{$input := index .CmdArgs 0}}
    {{/* I got this from Joe & Satty*/}}
    {{with reFindAllSubmatches `^<@&(\d{17,19})>|(\d{17,19})$` $input}}
			{{$id := toInt (or (index . 0 1) (index . 0 2))}}
			{{range $.Guild.Roles}}
				{{- if eq .ID $id}} {{$exactRole = .}} {{end -}}
			{{end}}
	{{else}}
		{{range .Guild.Roles}}
			{{- if eq (lower .Name) (lower $input)}} {{$exactRole = .}}
			{{- else if inFold (lower .Name) (lower $input)}} {{$maybeRole = .}}
			{{- end -}}
		{{end}}
    {{end}}
    {{$role := or $exactRole $maybeRole}}
    {{if $role}}
        {{$createdAt := div $role.ID 4194304 | add 1420070400000 | mult 1000000 | toDuration | (newDate 1970 1 1 0 0 0).Add}}
        {{if $role.Managed}}{{$managed = "True"}}{{end}}
        {{if $role.Hoist}}{{$hoist = "True"}}{{end}}
        {{if $role.Mentionable}}{{$mention = "True"}}{{end}}
        {{$pbit := $role.Permissions}}
        {{range seq 0 (len $permissions) }}
        {{- if (mod $pbit 2) }}{{$perm = joinStr "" $perm " " (index $permissions . ) -}}{{- end -}}
        {{- $pbit = div $pbit 2 -}}
        {{end}}
        {{$embed := sdict
        "title" (printf "❯ Info for %s" $role.Name)
		"fields" (cslice
			(sdict "name" "❯ ID" "value" (str $role.ID))
			(sdict "name" "❯ Managed" "value" $managed)
			(sdict "name" "❯ Mentionable" "value" $mention)
			(sdict "name" "❯ Hoist" "value" $hoist)
			(sdict "name" "❯ Position" "value" (str (add $role.Position 1)))
			(sdict "name" (print "❯ Permissions [" $role.Permissions "]") "value" $perm)
		)
        "color" ($role.Color) "footer" (sdict "text" "Created at") "timestamp" $createdAt
        }}
        {{sendMessage nil (cembed $embed)}}
    {{else}}Role not found.
    {{end}}
{{else}}
    Usage: {{.Cmd}} <RoleName/RoleMention/RoleID>
{{end}}
```
