# Rob

- Description: This command to rob another currency
- Usage: -rob <Mention/ID>
- Trigger type: `Command`
- Trigger text: `rob`

**Note**: Server must have currency system.
# Code
```lua
{{/*User Variable*/}}
{{ $currency := "credits" }} {{/* Name of the currency in your server */}}
{{ $dbName := "credits" }} {{/* Name of the Key of your DB that stores users currency ammount */}}
{{ $length := 1800 }} {{/* Length of the cooldown in second */}}

{{/*Code*/}}
{{ if (dbGet .User.ID "rob_cld") }}
Try again after {{humanizeDurationSeconds ((dbGet .User.ID "rob_cld").ExpiresAt.Sub currentTime) }}.
{{ else }}
{{ $target := 0 }}
{{ with .CmdArgs}} {{$target = index . 0 | userArg }} {{ end }}
{{ if eq (len .CmdArgs) 1 }}
    {{if not (eq $target.ID .User.ID) }}
    {{ $usermoney := toInt ((dbGet .User.ID $dbName).Value)}}
    {{ $targetmoney := toInt (dbGet $target.ID $dbName).Value}}
        {{ if and (gt $usermoney 0) (gt $targetmoney 0) }}
        {{ dbSetExpire .User.ID "rob_cld" "cooldown" $length}}
        {{ $networth := add $usermoney $targetmoney}}
        {{ $raterob := toInt (round (mult (fdiv $networth (add $networth $targetmoney)) 100)) }}
        {{ $random := randInt 1 3}}
        {{ $amount := toInt (round (div $targetmoney (mult $random 10)))}} 
        {{ if ge $raterob 60 }}
            {{ $rand := randInt 1 4}}
            {{ $prize := toInt (mult $rand $amount)}}
            {{ $user := dbIncr .User.ID $dbName $prize }}
            {{ $money := dbIncr $target.ID $dbName (mult -1 $amount)}}
            {{printf "Rob success! %s earned %s %s (bounty x%d)." .User.Username (humanizeThousands (toInt $prize)) $currency $rand }}
        {{ else}}
            {{ $amount = toInt (round (div $amount 5))}}
            {{ $user := dbIncr .User.ID $dbName (mult -1 $amount)}}
            {{printf "Rob fail! %s lost %s %s." .User.Username (humanizeThousands (toInt $amount)) $currency }}
        {{ end}}
        {{ else}}
            Can rob if you and target have more than 0 {{$currency}}.
        {{end}}
    {{else}}
        You can not rob yourself.
    {{end}}
{{else}}
Wrong syntax. The syntax of this command: -rob <Mention/ID>.
{{end}}
{{end}}
```