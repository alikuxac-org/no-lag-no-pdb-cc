**Recommend Trigger**: Command - **Trigger Text**: mutemyself\
**Description**: Mute your self with specific time\
**Usage**: -mutemyself <time>\
**Code**: below

```lua
{{$args := parseArgs 1 "-mutemyself <time>" (carg "duration" "")}}
{{$time := $args.Get 0}}
{{$silent := execAdmin "mute" .User.ID "Killyourself" ($time.String) }}
You will be revive after {{$time}}
```
