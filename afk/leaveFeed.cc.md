**Recommend Trigger**: None - **Trigger Text**: None\
**Description**: Delete afk mode of user when user leave server\
**Usage**: Put code in Leave Message (Notifications & Feeds -> General -> User Leave Message)\
**Code**: below

```lua
{{dbDel .User.ID "afk"}}
```