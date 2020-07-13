**Recommend Trigger**: Regex - **Trigger Text**: mutemyself\
**Description**: Mute your self with specific time\
**Usage**: -mutemyself <time>\
**Code**: below

```Lua
{{/*User Variable*/}}
{{$dropSuccess := "Someone dropped a lot of money, pick it fast"}}{{/*Drop success message*/}}
{{$dropFail := ""}}
{{$cmd := reFind `(?i)(pick|drop|chanbal|setbalchan)` .Cmd}}
{{$error := ""}}{{$bal := 0}}{{$chanbal := 0}}{{$perms := exec "viewperms"}}{{$chan := .Channel}}
{{with (dbGet .User.ID "money")}}{{$bal = .Value | toInt}}{{end}}
{{with (dbGet .Channel.ID "money")}}{{$chanbal = .Value | toInt}}{{end}}

{{if eq $cmd "drop"}}
{{/*Drop*/}}
{{$money := (parseArgs 1 "**Wrong Syntax**: -drop <money>" (carg "int" "money")).Get 0}}
{{if gt $money (toInt 0)}}
{{if lt $money $bal}}
{{deleteTrigger 1}}
    {{$user := dbIncr .User.ID "money" (mult -1 $money) }}
    {{$chan := dbIncr .Channel.ID "money" $money }}
    {{sendMessage nil "Ai đó vừa đi qua đây và làm rớt một cọc tiền siêu to khổng lổ kìa."}}
{{else}}
    {{$error = "Tiền drop nhiều hơn tiền đang có. Có trả nợ được hông mà drop <:nghilotuk:682120821698986014>"}}
{{end}}
{{else}}
    {{$error = "Nghĩ sao mà drop ít hơn 0 VND dị má <:nghilotuk:682120821698986014>"}}
{{end}}
{{else if eq $cmd "pick"}}
{{/*Pick*/}}
{{$money := (parseArgs 1 "**Wrong Syntax**: -pick <money>" (carg "int" "money")).Get 0}}
{{if gt $money (toInt 0)}}
{{if gt $money $chanbal}}
    {{$user := dbIncr .User.ID "money" (mult -1 $money) }}
    {{$chan := dbIncr .Channel.ID "money" $money }}
    {{sendMessage nil (print "Tham thì thâm nên `" .User "` vừa làm rớt " (humanizeThousands $money) " VND <:kekw:664991019397283840>")}}
{{else}}
    {{$user := dbIncr .Channel.ID "money" (mult -1 $money) }}
    {{$chan := dbIncr .User.ID "money" $money }}
    {{sendMessage nil (print "`" .User "` tay nhanh hơn não đã lụm được " (humanizeThousands $money) " VND <:HyperRage:526067071327535151>")}}
{{end}}
{{else}}{{$error = "Chơi dị ai chơi lại"}}
{{end}}
{{else if and (eq $cmd "chanbal") (or (hasRoleID 657240378562445333) (reFind `Administrator` $perms))}}
    {{$args := parseArgs 0 "**Syntax**: -chanbal <#channel>" (carg "channel" "")}}
    {{if $args.IsSet 0}}{{$chan = $args.Get 0}}{{end}}
    {{with (dbGet $chan.ID "money")}}{{$chanbal = .Value | toInt}}{{end}}
    {{print "Số tiền trong channel <#" $chan.ID "> đang có: " $chanbal " VND."}}
{{else if and (eq $cmd "setbalchan") (or (hasRoleID 657240378562445333) (reFind `Administrator` $perms))}}
    {{$args := parseArgs 1 "**Syntax**: -setbalchan <money> <#channel>" (carg "int" "") (carg "channel" "")}}
    {{$money := $args.Get 0}}
    {{if $args.IsSet 1}}{{$chan = $args.Get 1}}{{end}}
    {{if gt $money (toInt 0)}}
        {{dbSet $chan.ID "money" $money}}
        {{print "Đã set " (humanizeThousands $money) " VND trong channel <#" $chan.ID ">."}}
        {{sendMessage $chan.ID "Ai đó vừa đi qua đây và làm rớt một cọc tiền siêu to khổng lổ kìa."}}
    {{else}}{{$error = "Nghĩ sao mà drop ít hơn 0 VND dị má <:nghilotuk:682120821698986014>"}}
    {{end}}
{{end}}
{{with $error}}{{.}}{{end}}
```