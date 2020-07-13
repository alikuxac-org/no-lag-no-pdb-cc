**Recommend Trigger**: Regex - **Trigger Text**: \A-(drop|pick|setbalchhan|chanbal)\
**Description**: Pick&Drop System.\
**Usage**: -pick <money> | -drop <money> | -setbalchan <money> <#channel> | -chanbal <#channel>\
**Code**: below

```Lua
{{/*User Variable*/}}
{{$dropSuccess := "Someone dropped a lot of money, pick it now."}}{{/*Drop success message*/}}
{{$dropFail := "You can't drop more than money you have."}}
{{$dropbelow0 := "You can't drop lower than 0. "}}
{{$NoMoney := "Check you wallet before play this game."}}
{{$staff := 657240378562445333}}

{{$cmd := reFind `(?i)(pick|drop|chanbal|setbalchan)` .Cmd}}
{{$error := ""}}{{$bal := 0}}{{$chanbal := 0}}{{$perms := exec "viewperms"}}{{$chan := .Channel}}
{{with (dbGet .User.ID "money")}}{{$bal = .Value | toInt}}{{end}}
{{with (dbGet .Channel.ID "cmoney")}}{{$chanbal = .Value | toInt}}{{end}}

{{if eq $cmd "drop"}}
{{/*Drop*/}}
{{$money := (parseArgs 1 "**Wrong Syntax**: -drop <money>" (carg "int" "money")).Get 0}}
{{if gt $bal 0 }}
{{if (gt $money (toInt 0))}}
{{if lt $money $bal}}
{{deleteTrigger 1}}
    {{$user := dbIncr .User.ID "money" (mult -1 $money) }}
    {{$chan := dbIncr .Channel.ID "cmoney" $money }}
    {{$dropSuccess}}
{{else}}
    {{$error = $dropFail}}
{{end}}
{{else}}
    {{$error = $dropbelow0}}
{{end}}
{{else}}
{{$error = $NoMoney}}
{{end}}
{{else if eq $cmd "pick"}}
{{/*Pick*/}}
{{$money := (parseArgs 1 "**Wrong Syntax**: -pick <money>" (carg "int" "money")).Get 0}}
{{if gt $bal 0 }}
{{if and (gt $money (toInt 0)) (or (lt $money $bal) (lt $money $chanbal))}}
{{if lt $money $chanbal}}
    {{$user := dbIncr .Channel.ID "cmoney" (mult -1 $money) }}
    {{$chan := dbIncr .User.ID "money" $money }}
    {{sendMessage nil (print "`" .User "` pick successful and get " $money " .")}}
{{else}}
    {{$user := dbIncr .User.ID "money" (mult -1 $money) }}
    {{$chan := dbIncr .Channel.ID "cmoney" $money }}
    {{sendMessage nil (print "`" .User "`pick fail and drop " $money" .")}}
{{end}}
{{else}}{{$error = "Don't pick a lot of money."}}{{end}}
{{else}}{{$error = $NoMoney}}{{end}}

{{else if and (eq $cmd "chanbal") (or (hasRoleID $staff) (reFind `Administrator` $perms))}}
    {{$args := parseArgs 0 "**Syntax**: -chanbal <#channel>" (carg "channel" "")}}
    {{if $args.IsSet 0}}{{$chan = $args.Get 0}}{{end}}
    {{with (dbGet $chan.ID "cmoney")}}{{$chanbal = .Value | toInt}}{{end}}
    {{print "Channel <#" $chan.ID "> have " $chanbal " ."}}

{{else if and (eq $cmd "setbalchan") (or (hasRoleID $staff) (reFind `Administrator` $perms))}}
    {{$args := parseArgs 1 "**Syntax**: -setbalchan <money> <#channel>" (carg "int" "") (carg "channel" "")}}
    {{$money := $args.Get 0}}
    {{if $args.IsSet 1}}{{$chan = $args.Get 1}}{{end}}
    {{if gt $money (toInt 0)}}
        {{dbSet $chan.ID "cmoney" $money}}
        {{print "Set " $money " successful in channel <#" $chan.ID ">."}}
        {{sendMessage $chan.ID $dropSuccess}}
    {{else}}{{$error = $dropbelow0}}
    {{end}}
{{end}}

{{with $error}}{{.}}{{end}}
```