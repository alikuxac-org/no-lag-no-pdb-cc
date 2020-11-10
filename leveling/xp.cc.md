# Rank/XP

- Description: Show rank of current/specific user with Rank Card.
- Usage: `-rank [user]`
- Trigger type: **Regex**
- Trigger text: `\A(-|<@!?204255221017214977>)\s*(rank|lvl|xp)(\s+|\z)`

# Instructions

1. Sign up for Imagekit. Fill out `$IMAGEKIT_USERNAME`.
2. Add the Discord CDN as an endpoint. Follow this [Link](https://gist.github.com/alikuxac/0d4c29c523aa5df4f7a6e043482fe695)
3. Upload the Century Gothic font TTF file. Fill out `$CENTURY_GOTHIC_URL`.
4. Same with the Lucida Sans TTF file.
5. Upload a background, fill out `$BACKGROUND_URL`. Size is 934 x 282 pixels.
6. Upload avatar circle, fill out `$AVATAR_CIRCLE`.
7. Enjoy, use `-rank`

# Code

```lua
{{/*Modified version with Rank Card*/}}
{{ $IMAGEKIT_USERNAME := "Imagekit_Username" }}
{{ $LUCIDA_SANS_URL := "LucidaSansRegular.ttf" }}
{{ $CENTURY_GOTHIC_URL := "Century_Gothic.ttf" }}
{{ $BACKGROUND_URL := `3-generated.png` }}
{{ $AVATAR_CIRCLE := `3-avatar-hole.png` }}

{{/* Implementation originally written by Satty, refactored & commented by Joe. */}}
{{ define "toBase64" }}
	{{ $ENDINGS := dict 3.0 "=" 2.0 "==" }}
	{{ $octal := "" }}
	{{ range toByte .Input }}
		{{- $octal = printf "%s%08b" $octal . -}}
	{{ end }}

	{{/* Pad to nearest multiple of 6 with 0s */}}
	{{ with mod (len $octal) 6 }}
		{{ range seq 0 (sub 6 .) }}
			{{- $octal = print $octal 0 -}}
		{{ end }}
	{{ end }}

	{{/* Split into sections of 6 length each */}}
	{{ $parts := cslice }}
	{{ range seq 0 (div (len $octal) 6) }}
		{{- $start := mult . 6 }}
		{{- $parts = $parts.Append (slice $octal $start (add $start 6)) -}}
	{{ end }}

	{{ .Set "Result" "" }}
	{{ range $parts }}
		{{- $dec := 0 }}
		{{- range split . "" }}
			{{- $dec = add (mult $dec 2) (toInt .) -}}
		{{- end }}

		{{- $char := "" }}
		{{- if lt $dec 26 }} {{- $char = printf "%c" (add $dec 65) }}
		{{- else  if lt $dec 52 }} {{- $char = printf "%c" (add $dec 71) }}
		{{- else if lt $dec 62 }} {{- $char = printf "%c" (sub $dec 4) }}
		{{- else if eq $dec 63 }} {{- $char = "+" }}
		{{- else }} {{- $char = "-" }}
		{{- end }}

		{{- $.Set "Result" (print $.Result $char) -}}
	{{ end }}
	{{ with $ENDINGS.Get (mod (len $parts) 4) }} {{ $.Set "Result" (print $.Result .) }} {{ end }}
{{ end }}

{{/*
	Shortens a number.

	0 - 999 -> no change
	1,000 - 9,999 -> 1.01k (example)
	10,000 - 99,999 -> 10.1k (example)
	100,000 -> 999,999 -> 999k (example)
	1,000,000 -> 9,999,999 -> 1.01m
	...
*/}}
{{ define "shortenNumber" }}
	{{ $UNITS := cslice "" "K" "M" "B" "T" "q" "Q" "S" }}

	{{ $widths := sdict }}
	{{ if eq .Font "Century Gothic" }}
		{{ $widths = sdict
			"Units" (cslice 0 30 45 29 21 34 44 25)
			"Digit" 28
			"Dot" 13
		}}
	{{ else if eq .Font "Lucida Sans" }}
		{{ $widths = sdict
			"Units" (cslice 0 18 25 16 18 17 22 14)
			"Digit" 18
			"Dot" 8
		}}
	{{ end }}

	{{ $len := .Input | str | len }}
	{{ $unitIndex := div (sub $len 1) 3 }}
	{{ $unit := pow 10 (mult 3 $unitIndex) | toInt }}

	{{ $verb := "" }}
	{{ .Set "Width" (index $widths.Units $unitIndex) }}
	{{ $multiplier := fdiv .Input $unit }}
	{{ if gt .Input 999 }}
		{{ .Set "Width" (add .Width (mult 3 $widths.Digit)) }}
		{{ if lt $multiplier 10.0 }} {{ $verb = "%.2f" }}
		{{ else if lt $multiplier 100.0 }} {{ $verb = "%.1f" }}
		{{ end }}
	{{ else }}
		{{ .Set "Width" (add .Width (mult $len $widths.Digit)) }}
	{{ end }}

	{{ $unitName := index $UNITS $unitIndex }}
	{{ if not $verb }} {{ .Set "Result" (print (roundFloor $multiplier) $unitName) }}
	{{ else }}
		{{ .Set "Result" (printf (print $verb "%s") $multiplier $unitName) }}
		{{ .Set "Width" (add .Width $widths.Dot) }}
	{{ end }}
{{ end }}

{{ $err := "" }}{{ $user := .User}}
{{ if .CmdArgs }}
    {{ $temp := index .CmdArgs 0 | userArg }}{{ if $temp}}{{ $user = $temp }}{{ else }}{{ $err = "User not found." }}{{ end }}
{{end}}
{{ if $user }}
    {{$name := or (getMember $user.ID).Nick $user.Username}}
	{{ $data := sdict "Avatar" (print $user.ID "@@" $user.Avatar ".webp") "Name" $user.Username}}
	{{$dbuser := sdict}}{{with (dbGet $user.ID "user")}}{{$dbuser = sdict .Value}}{{end}}
    {{$color := "FF0000"}}{{with ($dbuser.Get "color")}}{{$color = printf "%06x" . | upper}}{{end}}
    {{$data.Set "Color" (str $color)}}
    {{ $xp := 0 }}{{with (dbGet $user.ID "xp")}}{{$xp = .Value}}{{end}}

    {{$level := roundFloor (mult 0.1 (sqrt $xp))}}
    {{$data.Set "Level" (toInt $level)}}

	{{$current := sub $xp (mult 100 $level $level)}}
	{{if lt (toInt $xp) 100}}{{$current = $xp}}{{end}}
	{{$data.Set "Current" (toInt $current)}}

	{{$nextLvl := add $level 1}}
	{{$total := sub (mult 100 $nextLvl $nextLvl) (mult 100 $level $level)}}
	{{$data.Set "Total" (toInt $total)}}

    {{$rank := 0}}
    {{ range $index, $entry := dbTopEntries "xp" 100 0 }}{{- if eq .UserID $user.ID}} {{- $rank = add $index 1}} {{- end}}{{end}}
    {{$data.Set "Rank" (toInt $rank)}}
	{{ if not $err  }}
		{{ $bar := "" }}
		{{/*
			101 underscores with 12px font fills up the rank bar.
			We dont want to show users a full bar, ever, so we use `roundFloor` instead of `round`.
		*/}}
		{{ range seq 0 (fdiv $data.Current $data.Total | mult 101.0 | roundFloor | toInt) }}
			{{- $bar = print $bar "_" -}}
		{{ end }}

		{{/*
			Name needs to be encoded in base64 as it can contain special characters.
			Current, total XP, level, and rank need to be encoded as well as they can contain the '.' character.
        */}}

		{{ $query := sdict "Input" $data.Name }}
		{{ template "toBase64" $query }}
		{{ $name := urlquery $query.Result }}

		{{ $query := sdict "Input" $data.Total "Font" "Lucida Sans" }}
		{{ template "shortenNumber" $query }}
		{{ $totalX := sub 831 $query.Width }} {{/* X coordinate at which to insert the total XP */}}
		{{ $query.Set "Input" (print "/ " $query.Result " XP") }}
		{{ template "toBase64" $query }}
		{{ $total := urlquery $query.Result }}

		{{ $query := sdict "Input" $data.Current "Font" "Lucida Sans" }}
		{{ template "shortenNumber" $query }}
		{{ $currentX := sub $totalX 1 $query.Width }}
		{{ $query.Set "Input" $query.Result }}
		{{ template "toBase64" $query }}
		{{ $current := urlquery $query.Result }}

		{{ $query := sdict "Input" $data.Level "Font" "Century Gothic" }}
		{{ template "shortenNumber" $query }}
		{{ $levelX := sub 860 $query.Width }}
		{{ $query.Set "Input" $query.Result }}
		{{ template "toBase64" $query }}
		{{ $level := urlquery $query.Result }}

		{{ $query := sdict "Input" $data.Rank "Font" "Century Gothic" }}
		{{ template "shortenNumber" $query }}
		{{ $rankX := sub $levelX 135 $query.Width }}
		{{ $query.Set "Input" (print "#" $query.Result) }}
		{{ template "toBase64" $query }}
		{{ $rank := urlquery $query.Result }}

		{{ $base := print "https://ik.imagekit.io/" $IMAGEKIT_USERNAME "/" }}
		{{/* Using printf instead of print here (in most cases) so its clearer what transitions do */}}
		{{ $transformations := cslice
			(printf "oi-%s,oh-160,ow-160,ox-71,oy-61,oiq-100" $data.Avatar)
			(printf "oi-%s,ox-71,oy-61" $AVATAR_CIRCLE)
			(printf "ot-____,otbg-%s,otc-%s,ox-250,oy-180,ow-40,oh-40,or-20,otp-12" $data.Color $data.Color)
			(printf "ot-%s,otbg-%s,otc-%s,ox-260,oy-180,ow-40,oh-40,or-20,otp-12" $bar $data.Color $data.Color)
			(printf "ots-38,otf-%s,ote-%s,ox-260,oy-125,otc-FFFFFF" $LUCIDA_SANS_URL $name)
			(printf "ots-24,otf-%s,otc-7F8384,ote-%s,ox-%d,oy-135" $LUCIDA_SANS_URL $total $totalX)
			(printf "ots-24,otf-%s,otc-FFFFFF,ote-%s,ox-%d,oy-135" $LUCIDA_SANS_URL $current $currentX)
			(printf "otf-%s,ote-%s,ots-50,ox-%d,oy-50,otc-%s" $CENTURY_GOTHIC_URL $level $levelX $data.Color)
			(printf "otf-%s,ot-LEVEL,ots-26,otc-%s,ox-%d,oy-75" $LUCIDA_SANS_URL $data.Color (sub $levelX 75))
			(printf "otf-%s,ote-%s,ots-50,ox-%d,oy-50,otc-FFFFFF" $CENTURY_GOTHIC_URL $rank $rankX)
			(printf "otf-%s,ot-RANK,ots-26,otc-FFFFFF,ox-%d,oy-75" $LUCIDA_SANS_URL (sub $rankX 72))
		}}

		{{ $url := print $base "tr:" (joinStr ":" $transformations.StringSlice) "/" $BACKGROUND_URL }}
		{{ sendMessage nil (cembed
			"color" 0xFF0000
			"author" (sdict "name" $user.String "icon_url" ($user.AvatarURL "1024"))
			"title" "Your rank"
			"image" (sdict "url" $url)
			"footer" (sdict "text" "The image may take a while to load.")
		) }}
	{{ end }}
    {{ end }}
{{ if $err }} {{ .User.Mention }} **::** {{ $err }} {{ end }}
```
