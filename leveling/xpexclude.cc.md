# XP Exclude
- Description: Exclude a channel, role, user, category from the xp system.
- Usage:
    `-xpex user @user` | Exclude specific user.
    `-xpex channel @channel` | Exclude specific channel.
    `-xpex role @role` | Exclude specific role.
    `-xpex cate/category categoryID` | Exclude specific category.
    `-xpex list` | Show exclude list
- Trigger type: **Regex**
- Trigger text: `\A(-|<@!?204255221017214977>)\s*(xpexclude|xpex)(\s+|\z)`

# Code
```lua
{{ define "splice" }}
    {{ $data := .Data }}{{ $index := .Index }}
    {{ $last := sub (len $data) 1 }}
    {{ if gt $index $last }} {{ .Set "Res" 0 }}
    {{ else if eq $index $last }} {{ .Set "Res" (slice $data 0 $last) }}
    {{ else if $index }} {{ .Set "Res" ((slice $data 0 $index).AppendSlice (slice $data (add $index 1))) }}
    {{ else }} {{ .Set "Res" (slice $data 1) }} {{ end }}
{{ end }}

{{ $icon:="" }}
{{ if .Guild.Icon }}{{ $icon = printf "https://cdn.discordapp.com/icons/%d/%s.webp" .Guild.ID .Guild.Icon }}{{ end }}
{{ $isExclude := false }}{{ $index := 0 }}
{{ $userdisplay := "" }}{{ $chandisplay := "" }}{{ $roledisplay := "" }}{{ $catedisplay := "" }}
{{ $settings := sdict }}{{ with (dbGet 0 "xpSettings") }}{{ $settings =sdict .Value }}{{ end }}
{{ $exclude := sdict }}{{ if ($settings.Get "exclude") }}{{ $exclude = sdict ($settings.Get "exclude") }}{{ end }}
{{ $userex := cslice }}{{ if $exclude.Get "user" }}{{ $userex = (cslice).AppendSlice ($exclude.Get "user") }}{{ end }}
{{ $chanex := cslice }}{{ if $exclude.Get "channel" }}{{ $chanex = (cslice).AppendSlice ($exclude.Get "channel") }}{{ end }}
{{ $roleex := cslice }}{{ if $exclude.Get "role" }}{{ $roleex = (cslice).AppendSlice ($exclude.Get "role") }}{{ end }}
{{ $cateex := cslice }}{{ if ($exclude.Get "category") }}{{ $cateex = (cslice).AppendSlice ($exclude.Get "category") }}{{ end }}
{{ $user := "" }}{{ $channel := "" }}{{ $id := 0 }}
{{ if .CmdArgs }}
{{ $type := index .CmdArgs 0 | str }}
{{ if eq $type "user" }}
    {{ $input := index .CmdArgs 1 }}
    {{ with reFindAllSubmatches `^<@!?(\d{17,19})>|(\d{17,19})$` $input }}{{ with (toInt64 (or (index . 0 1) (index . 0 2))) }}{{ $user = getMember . }}{{ end }}{{ end }}
    {{ if $user }}
        {{ range $key, $value := $userex }}
            {{- if eq $value (str $user.User.ID) -}}{{- $isExclude = true -}}{{- $index = $key -}}{{- end -}}
        {{ end }}
        {{ if $isExclude }}
        {{ if eq (len $userex) 1 }}{{ $userex = cslice }}{{ else }}{{ $data := sdict "Data" $userex "Index" $index }}{{ template "splice" $data }}{{ $userex = $data.Res }}{{ end }}
        `{{ $user.User.String }}` is no longer excluded from the XP system on this server.
        {{ else }}
            {{ $userex = $userex.Append (str $user.User.ID) }}
            `{{ $user.User.String }}` has been excluded from the XP system on this server.
        {{ end }}
        {{ $exclude.Set "user" $userex }}
        {{ $settings.Set "exclude" $exclude }}
        {{ dbSet 0 "xpSettings" $settings }}
    {{ else }}
        Member not found.
    {{ end }}
{{ else if eq $type "channel" }}
    {{ $input := index .CmdArgs 1 }}
    {{ with reFindAllSubmatches `^<#(\d{17,19})>|(\d{17,19})$` $input }}{{ with (toInt64 (or (index . 0 1) (index . 0 2))) }}{{ $channel = getChannel . }}{{ end }}{{ end }}
    {{ if $channel }}
        {{ range $key, $value := $chanex }}
            {{- if eq $value (str $channel.ID) -}}{{- $isExclude = true -}}{{- $index = $key -}}{{- end -}}
        {{ end }}
        {{ if $isExclude }}
        {{ if eq (len $chanex) 1 }}{{ $chanex = cslice }}{{ else }}{{ $data := sdict "Data" $chanex "Index" $index }}{{ template "splice" $data }}{{ $chanex = $data.Res }}{{ end }}
        `{{ $channel.Name }}` is no longer excluded from the XP system on this server.
        {{ else }}
            {{ $chanex = $chanex.Append (str $channel.ID) }}
            `{{ $channel.Name }}` has been excluded from the XP system on this server.
        {{ end }}
        {{ $exclude.Set "channel" $chanex }}
        {{ $settings.Set "exclude" $exclude }}
        {{ dbSet 0 "xpSettings" $settings }}
    {{ else }}
        Channel not found.
    {{ end }}
{{ else if eq $type "role" }}
    {{ $roleex := cslice }}{{ if ($exclude.Get "role") }}{{ $roleex = (cslice).AppendSlice ($exclude.Get "role") }}{{ end }}
    {{ $exactRole:="" }}{{ $maybeRole:="" }}
    {{ $input := index .CmdArgs 1 }}
    {{ with reFindAllSubmatches `^<@&(\d{17,19})>|(\d{17,19})$` $input }}
		{{ $id := toInt (or (index . 0 1) (index . 0 2)) }}
		{{ range $.Guild.Roles }}
			{{- if eq .ID $id }} {{ $exactRole = . }} {{ end -}}
		{{ end }}
	{{ else }}
		{{ range .Guild.Roles }}
			{{- if eq (lower .Name) (lower $input) }} {{ $exactRole = . }}
			{{- else if inFold (lower .Name) (lower $input) }} {{ $maybeRole = . }}
			{{- end -}}
		{{ end }}
    {{ end }}
        {{ $role := or $exactRole $maybeRole }}
    {{ if $role }}
        {{ range $key, $value := $roleex }}
            {{- if eq $value (str $role.ID) -}}{{- $isExclude = true -}}{{- $index = $key -}}{{- end -}}
        {{ end }}
        {{ if $isExclude }}
        {{ if eq (len $roleex) 1 }}{{ $roleex = cslice }}{{ else }}{{ $data := sdict "Data" $roleex "Index" $index }}{{ template "splice" $data }}{{ $roleex = $data.Res }}{{ end }}
        `{{ $role.Name }}` is no longer excluded from the XP system on this server.
        {{ else }}
        {{ $roleex = $roleex.Append (str $role.ID) }}
        `{{ $role.Name }}` has been excluded from the XP system on this server.
        {{ end }}
        {{ $exclude.Set "role" $roleex }}
        {{ $settings.Set "exclude" $exclude }}
        {{ dbSet 0 "xpSettings" $settings }}
    {{ else }}
        Role not found.
    {{ end }}
{{ else if eq $type "cate" "category" }}
    {{ $cateex := cslice }}{{ if ($exclude.Get "category") }}{{ $cateex = (cslice).AppendSlice ($exclude.Get "category") }}{{ end }}
    {{ $input := index .CmdArgs 1 }}
    {{ $channel := getChannel $input }}
    {{ if $channel }}
        {{ range $key, $value := $cateex }}
            {{- if eq $value (str $channel.ID) -}}{{- $isExclude = true -}}{{- $index = $key -}}{{- end -}}
        {{ end }}
        {{ if $isExclude }}
        {{ if eq (len $cateex) 1 }}{{ $cateex = cslice }}{{ else }}{{ $data := sdict "Data" $cateex "Index" $index }}{{ template "splice" $data }}{{ $cateex = $data.Res }}{{ end }}
        `{{ $channel.Name }}` is no longer excluded from the XP system on this server.
        {{ else }}
            {{ $cateex = $cateex.Append (str $channel.ID) }}
            `{{ $channel.Name }}` has been excluded from the XP system on this server.
        {{ end }}
        {{ $exclude.Set "category" $cateex }}
        {{ $settings.Set "exclude" $exclude }}
        {{ dbSet 0 "xpSettings" $settings }}
    {{ else }}
        Category not found.
    {{ end }}
{{ else if eq $type "list" }}
    {{ if eq (len $userex) 0 }}{{ $userdisplay = "None" }}{{ else }}{{ range $userex }}{{- $userdisplay = print $userdisplay " <@" . ">" -}}{{ end }}{{ end }}
    {{ if eq (len $chanex) 0 }}{{ $chandisplay = "None" }}{{ else }}{{ range $chanex }}{{- $chandisplay = print $chandisplay " <#" . ">" -}}{{ end }}{{ end }}
    {{ if eq (len $roleex) 0 }}{{ $roledisplay = "None" }}{{ else }}{{ range $roleex }}{{- $roledisplay = print $roledisplay " <@&" . ">" -}}{{ end }}{{ end }}
    {{ if eq (len $cateex) 0 }}{{ $catedisplay = "None" }}{{ else }}{{ range $cateex }}{{- $catedisplay = print $catedisplay " <#" . ">" -}}{{ end }}{{ end }}
    {{ $embed := sdict 
        "title" "XP Excluded List"
        "description" "This is a list about XP Excluded in this server."
        "fields" (cslice
            (sdict "name" "❯ User" "value" $userdisplay)
            (sdict "name" "❯ Channel" "value" $chandisplay)
            (sdict "name" "❯ Role" "value" $roledisplay)
            (sdict "name" "❯ Category" "value" $catedisplay))
        "footer" (sdict "text" .Server.Name "icon_url" $icon)
        "timestamp" currentTime
 }}
    {{ sendMessage nil (cembed $embed) }}
{{ else }}
    {{ sendMessage nil (cembed "title" "XP Exclusion Help" "description" (print "`" .Cmd " user @user` to exclude specific user.\n`" .Cmd " channel @channel` to exclude specific channel.\n`" .Cmd " role @role` to exclude specific role.\n`" .Cmd " cate/category categoryID` to exclude specific category.\n`" .Cmd " list to show exclude list.\n\n**Currently support ID**")) }}
{{ end }}

{{ else }}Usages: -xpexclude/xpex (role/channel/user/cate/category/list) [#channel/@user/@role/categoryID]
{{ end }}
```
