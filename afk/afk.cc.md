**Recommend Trigger**: Command - **Trigger Text**: afk\
**Description**: Set AFK Mode for yourself\
**Usage**: -afk \[time\] \[reason\] \
**Code**: below

```lua
{{$roles := .Member.Roles}}{{$yagroles := (getMember 204255221017214977).Roles}}{{$upos := 0}}{{$yagpos := 0}}
{{range $yagroles}}
    {{- if lt $yagpos ($.Guild.Role .).Position}}{{- $yagpos = ($.Guild.Role .).Position}}{{- end}}
{{end}}
{{range $roles}}
    {{- if lt $upos ($.Guild.Role .).Position}}{{- $upos = ($.Guild.Role .).Position}}{{- end}}
{{end}}
{{$name := or .User.Username .Member.Nick}}
{{$reason := "No reason specific"}}
{{$duration := ""}}
{{$StrippedMsg := .StrippedMsg}}
{{if $StrippedMsg}}
    {{if eq $StrippedMsg "help"}}
        Usages:
        -afk [-d duration] [reason] | Set AFK with reason and duration optional.
        -afk help | Show this message
        Remove your AFK with `-afk`
    {{else}}
    {{with reFindAllSubmatches `(?i)((?:-d) (\w+)(?:\s|$))` $StrippedMsg}}
        {{$StrippedMsg = reReplace `(?i)(-d \w+\s+)|(?i)(-d \w+\s+)|(-d \w+$)` $StrippedMsg ""}}
        {{$duration = (index . 0 2)}}
    {{end}}
    {{if eq $StrippedMsg ""}}{{$StrippedMsg = $reason}}{{end}}
    {{$parsedDur := 0}}
    {{with and $duration (toDuration $duration)}} {{$parsedDur = .}} {{end}}
	{{if $parsedDur}}
		{{dbSetExpire .User.ID "afk" $StrippedMsg (toInt $parsedDur.Seconds)}}
    {{else}} {{dbSet .User.ID "afk" $StrippedMsg}} {{end}}
    {{if and (lt (len $name) 27) (lt $upos $yagpos)}}{{editNickname (print "[AFK]" $name)}}{{end}}
    {{.User.Mention}}, I set your AFK to `{{$StrippedMsg}}`.
    {{end}}
{{else}}
    {{if dbGet .User.ID "afk"}}
        {{dbDel .User.ID "afk"}}
        {{if and (reFind `[AFK]` $name)  (lt $upos $yagpos)}}{{editNickname (slice $name 5)}}{{end}}
		{{.User.Mention}}, I removed your AFK.
	{{else}}
        {{dbSet .User.ID "afk" $reason}}
        {{if and (lt (len $name) 27) (lt $upos $yagpos)}}{{editNickname (print "[AFK]" $name)}}{{end}}
        {{.User.Mention}}, I set your AFK to `{{$reason}}`.
    {{end}}
{{end}}
```
