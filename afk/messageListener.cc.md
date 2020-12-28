**Recommend Trigger**: `Regex` - **Trigger Text**: `.*`\
**Description**: Delete AFK Mode when user send a message or attachment\
**Usage**: Send any message\
**Note**: If you already have custom command with regex `.*`, put code inside that command
**Code**: below

```lua
{{$prefix := index (reFindAllSubmatches `Prefix of \x60\d+\x60: \x60(.+)\x60` (exec "prefix")) 0 1}}
{{if (not (reFind (print `\A` $prefix `afk`) .Cmd))}}
    {{if dbGet .User.ID "afk"}}
        {{print "Welcome back. " .User.Mention "\nRemoved your AFK's status"}}
        {{dbDel .User.ID "afk"}}
        {{if and .Member.Nick (reFind `\[AFK\]` .Member.Nick)}}
            {{editNickname (slice .Member.Nick 5)}}
        {{end}}
    {{end}}
{{end}}
```