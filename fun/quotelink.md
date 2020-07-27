**Recommend Trigger Type**: Regex - **Trigger Text**: `<?https://(ptb.|canary.)?(discordapp\.com|discord\.com)/channels\/\d+\/(\d+)\/(\d+)>?`\
**Description**: Quote the message\
**Usage**: Get message link and post in any channel you want\
**Code**:

```lua
{{/* v3 */}}
{{if (reFind `(Administrator)` (exec "viewperms"))}}
{{ $matches := reFindAllSubmatches `https://(ptb.|canary.)?(discordapp\.com|discord\.com)/channels\/\d+\/(\d+)\/(\d+)` .Message.Content }} 

{{$msg := getMessage (index (index $matches 0) 3) (index (index $matches 0) 4) }}

{{if $msg}}
{{if (reFind `(?i)(discord\.gg|discordapp\.com\/invite)(?:\/#)?\/([a-zA-Z0-9-])+|discord\.me\/.+|invite\.gg\/.+|discord\.io\/.+|discord\.li\/.+|disboard\.org\/server\/join\/.+|discordy\.com\/server\.php` (toString $msg.Content))}}
{{ $avatar := (joinStr "" "https://cdn.discordapp.com/avatars/" (toString $msg.Author.ID) "/" $msg.Author ".png") }}Message has an invite!{{else}}

{{$embedRaw := sdict 
"description" (joinStr "" "**[Message Link](" (index (index $matches 0) 0) ")  to <#" $msg.ChannelID ">**\n" $msg.Content)
"color" 4645612 
"author" (sdict "name"  $msg.Author.Username "icon_url" ($msg.Author.AvatarURL "64")) 
"footer" (sdict "text" (joinStr "" "Req. by "  .Message.Author.Username ". Quote from ")) 
"timestamp" $msg.Timestamp  }}

{{if $msg.Attachments}}
{{$embedRaw.Set "image" (sdict "url" (index $msg.Attachments 0).URL) }}
{{end}}

{{ sendMessage nil (cembed $embedRaw) }}

{{/* delete the trigger if it only contained a link and nothing more */}}
{{if eq (len (index (index $matches 0) 0)) (len .Message.Content) }} {{deleteTrigger 0}} {{end}} 
{{end}}{{end}}{{end}}
```