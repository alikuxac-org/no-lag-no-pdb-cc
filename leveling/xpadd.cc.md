# Add XP

- Description: This command add xp of specific user.
- Usage: `-xpadd <user> <number of XP>`
- Trigger type: **Command**
- Trigger text: `xpadd`

# Code
```lua
{{ $user := 0  }} 
{{ $newLvl := 0  }} 
{{ $oldLvl := 0  }}  
{{ with .CmdArgs  }} {{ $user = index . 0 | userArg  }} {{ end  }} 
{{ if (eq (len .CmdArgs) 2)  }}
    {{ $xp := index .CmdArgs 1 | toInt  }} 
    {{ if and $xp $user  }}
        {{ $oldXp := 0  }} {{/* Old XP */}}
        {{ with (dbGet $user.ID "xp")  }} {{ $oldXp = .Value  }} {{ end  }}
        {{ $oldLvl := roundFloor (mult 0.1 (sqrt $oldXp))  }} 
        {{ $newXp := add $oldXp $xp }}
        {{ $s := dbSet $user.ID "xp" $newXp  }} 
        {{ $updatedLvl := roundFloor (mult 0.1 (sqrt $newXp))  }} 
        {{ if ne $oldLvl $updatedLvl  }} 
            {{ $newLvl = $updatedLvl  }} 
        {{ end  }} 
        {{ printf "Successfully add %d to **%s**'s XP!" $xp $user.String  }}
    {{ else  }}
        The syntax for this command is `-xpadd <user> <xp>`. 
    {{ end  }}
{{ end  }}
{{ if $newLvl  }}
    {{ $roleRewards := sdict "type" "stack"  }}
    {{ with dbGet 0 "roleRewards"  }} {{ $roleRewards = sdict .Value  }} {{ end  }}
    {{ $type := $roleRewards.type  }}
    {{ $toAdd := 0  }}
    {{ $dist := -1  }}
	{{ range $level, $reward := $roleRewards  }}
		{{ if and (le (toInt $level) (toInt $newLvl)) (or (gt $dist (sub (toInt $newLvl) (toInt $level))) (eq $dist -1)) (eq $type "highest")  }}
			{{ $dist = sub (toInt $newLvl) (toInt $level)  }} {{ $toAdd = $reward  }}
		{{ end  }}
		{{ if and (ge (toInt $newLvl) (toInt $level)) (not (targetHasRoleID $user.ID $reward)) (eq $type "stack") (ne $level "type")  }} {{ giveRoleID $user.ID $reward  }}
		{{ else if and (targetHasRoleID $user.ID $reward) (eq $type "highest")  }} {{ takeRoleID $user.ID $reward  }} {{ end  }}
	{{ end  }}
	{{ if $toAdd  }} {{ giveRoleID $user.ID $toAdd  }} {{ end  }} 
{{ end  }}
```
