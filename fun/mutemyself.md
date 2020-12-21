**Recommend Trigger**: Command - **Trigger Text**: mutemyself\
**Description**: Mute your self with specific time\
**Usage**: -mutemyself <time>\
**Code**: below

```lua
{{$time := toDuration "30m"}}
{{$args := parseArgs 0 "-mutemyself <time>" (carg "duration" "")}}
{{if $args.IsSet 0}}
{{$time = $args.Get 0}}
{{end}}
{{$silent := execAdmin "mute" .User.ID "Thank you for kill your self" ($time.String) }}
You will be revive after {{$time.String}}
```
