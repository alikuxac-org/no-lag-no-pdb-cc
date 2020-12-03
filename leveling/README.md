# V2 Leveling System
These commands are not standalone. Add all the commands if you wish to use it. Note that you will need configuration for each custom command which is noted by a comment.

As usual, there is a leading comment showing the trigger and description of the CC.

# Functionality
- Role rewards (one per level, up to 200 from level 1 to 200)
- Two types of role rewards (stack, and highest, explained more)
- Source is easy to read (indented, commented with leading comment in each "command" to explain that command)
- Paginated leaderboard
- Utilizes DB system
- Allows you to set xp and level easily
- Get started with 1 command, no editing of source needed
- Reset xp per user.
- XP exclusion.

# Getting Started
Copy the commands given below, add them with the instructions given in the comment at the very TOP of the file, and run `-leveling use-default` to get started with the default settings!

# Command List
1. **Leveling**
    - Description: This command manages the general level settings of the guild.
    - Usage: 
    `-leveling set <key> value` | example: -leveling set cooldown 1 minute 30 seconds 
	`-leveling use-default` | use default settings 
	`-leveling view` | view settings
    - Trigger type: **Regex**.
    - Trigger text: `\A(-|<@!?204255221017214977>)\s*(leveling|(level|lvl)-?conf|(level|lvl)-?settings)(\s+|\z)`
2. **Regex**
    - Description: This command manages messages - setting cooldowns, giving role rewards when users level up, and giving XP.
    - Usage: Any message sent.
    - Trigger type: **Regex**
    - Trigger text: `.*`
3. **Reset XP** (New)
    - Description: This command reset specific User XP.
    - Usage: `-resetxp <@Mention|ID>`
    - Trigger type: **Command**
    - Trigger text: `resetxp`
4. **Role Reward**
    - Description: This command manages the role rewards of the server.
    - Usage:
        `-rr add <level> <role name>` | Adds a role reward to given level in range 1-100. Max 1 per level.
	    `-rr remove <level>` | Removes role reward from level.
	    `-rr set-type <stack|highest>` | Sets type of role giving.
	    `-rr view` | Views current setup
	    `-rr reset` | Resets the settings
    - Trigger type: **Regex**
    - Trigger text: `\A(-|<@!?204255221017214977>)\s*(role-?rewards|rr)(\s+|\z)`
5. **Set XP/Level**
    - Description: This command set xp/level of specific user.
    - Usage: `-setxp/setlv <user> <number of xp|level>`
    - Trigger type: **Regex**
    - Trigger text: `\A(-|<@!?204255221017214977>)\s*(set-?(?:xp|level))(\s+|\z)`
6. **XP/Rank** (Modified)
    - Description: Show rank of current/specific user with Rank Card.
    - Usage: `-rank [user]`
    - Trigger type: **Regex**
    - Trigger text: `\A(-|<@!?204255221017214977>)\s*(rank|lvl|xp)(\s+|\z)`
7.  **Add XP**
    - Description: This command add xp of specific user.
    - Usage: `-xpadd <user> <number of XP>`
    - Trigger type: **Command**
    - Trigger text: `xpadd`
8. **XP Exclude** (New)
    - Description: Exclude a channel, role, user, category from the xp system.
    - Usage:
    `-xpex user @user` | Exclude specific user.
`-xpex channel @channel` | Exclude specific channel.
`-xpex role @role` | Exclude specific role.
`-xpex cate/category categoryID` | Exclude specific category.
    - Trigger type: **Regex**
    - Trigger text: `\A(-|<@!?204255221017214977>)\s*(xpexclude|xpex)(\s+|\z)`
9. **Leaderboard**
- Description: This command show XP leaderboard in a server.
- Usage:
  `-leaderboard [page]` where page is optional
- Trigger type: **Regex**.
- Trigger text: `\A(-|<@!?204255221017214977>)\s*(leaderboard|lb|top)(\s+|\z)`
10. **ReactionListener**
- Description: This command manages the pagination of the leaderboard command.
- Usage: React valid emoji in leaderboard
- Trigger type: **Reaction**.
- Trigger: `Added Only`
# Credit
Author: [Jo3-L](https://github.com/Jo3-L/)
From reposity: [yagpdb-cc](https://github.com/yagpdb-cc/yagpdb-cc)
Modified by **Alikuxac#4177**.