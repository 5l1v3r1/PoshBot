
# Triggers

A trigger tells PoshBot what to look for in an incoming chat message.
Upon receiving a message, PoshBot will search through all loaded plugins and looking for a command trigger that matches the message.
The most common trigger will be `Command` which tells PoshBot to look at the text of the message and determine if it is a command string.
Once a match is found, that command will be executed.

## Defining a trigger

PoshBot provides a **custom attribute** called `[PoshBot.BotCommand()]` that you can decorate your functions with.
This custom attribute is how PoshBot knows what type of trigger to use for the command.
The custom attribute is applied to the function similarly to the `[CmdletBinding()]` attribute.

#### [PoshBot.BotCommand()] properties

| Property       | Type     | Description |
| :--------------|:---------|:------------|
| CommandName    | string   | The name of the bot command. Default is the function name
| Aliases        | string[] | Alternate name(s) for the command
| TriggerType    | string   | The type of trigger. Values: `Command`, `Regex`, `Event`
| HideFromHelp   | bool     | Whether to hide the command when the !help command is used. Default is `$false`
| Regex          | string   | A regex string to match the command against. Only valid when TriggerType is `Regex`
| MessageType    | string   | Type of message this command is triggered against. Only valid when TriggerType is `Event`
| MessageSubtype | string   | Subtype of message this command is triggered against. Only valid when TriggerType is `Event`
| Permissions    | string[] | String array of permissions to apply to the command. Only users with the given permissions are allow to execute command

#### Example

```powershell
function Get-Foo {
    [PoshBot.BotCommand()]
    [CmdletBinding()]
    param()

    Write-Output 'Bar'
}
```

By default, PoshBot will set each command in a plugin to the `Command` trigger.
You do not need to apply the `[PoshBot.BotCommand()]` attribute in that case unless you also want to override the command name like below:

#### Example

Override the bot commands name to `Foo` instead of the function name `Get-Foo`.

```powershell
function Get-Foo {
    [PoshBot.BotCommand(CommandName = 'Foo')]
    [CmdletBinding()]
    param()

    Write-Output 'Bar'
}

```

## Trigger Types

### Command

This is the most common type of trigger.
It tells PoshBot to look at the text of the incoming message and match it to the name (or aliases) of a registered command.
Commands will only be matched if the message includes the `command prefix` as the first character, proceeded by the command name.
If `alternate command prefixes` have been defined in the bot configuration, those will be matched as well.

#### Example 1

The default configuration sets the character `!` as the command prefix.
The syntax for executing the command would be:

```
!mycommand --param 'some value'
```

#### Example 2

If an `alternate command prefix` like `bender` has been defined, the syntax for executing the command would be:

```
bender, mycommand --param 'some value'
```

#### Example 3

If both an `alternate command prefix` called `bender` and `alternate command prefix seperator` of `,` have been defined, the syntax would be:

```
hal, open-doors --type 'pod'
```

### Event

This type of trigger listens for certain types of event messages that the chat network returns.
Events like a user entering or exiting a room, channel topic change, a user's presence status is changed, etc.
When a message is received by PoshBot, it will evaluate the message type and look for commands that have registered a trigger that matches this message type.
When one is found, the command will be executed.

### RegEx

This type of trigger matches a regex expression against the chat message text.
If a match is found, the command will be executed.
This type of trigger is useful if you want a command to be casually listening in the chat room and matching each message against it's regex trigger.

#### Passing parameters to regex commands

When PoshBot evaluates the regex expression, it collects the regex capture groups and passes them to the `Arguments` parameter of the function.

> You **must** define an `Arguments` parameter of type `[object[]]` in order for PoshBot to send the parameters captured from the regex expression to the function.

#### Example 1

This command will match every incoming chat message against the regex `'cookies'`.
If a user types something like **"I bought some Girl Scout cookies yesterday"** PoshBot will execute this command and return a message saying **"Did someone mention cookies? I love cookies! Nom Nom Nom!"**

```powershell
function Cookies {
    <#
    .SYNOPSIS
        Respond to cookies
    #>
    [PoshBot.BotCommand(
        Command = $false,
        CommandName = 'cookies',
        TriggerType = 'regex',
        Regex = 'cookies'
    )]
    [cmdletbinding()]
    param(
        [parameter(ValueFromRemainingArguments)]
        $Dummy
    )

    Write-Output 'Did someone mention cookies? I love cookies! Nom Nom Nom!'
}
```

#### Example 2

This command will match every incoming chat message against the regex `'^grafana\s(cpu|disk)\s(.*)'`.
This regex expects the word `grafana` to start at the beginning of the line and the next word to be `cpu` or `disk`.
The next work can be anything.

```powershell
function Get-GrafanaGraph {
    <#
    .SYNOPSIS
        Displays a Grafana graph for a given type for a server.
    .EXAMPLE
        grafana cpu server01
    .EXAMPLE
        grafana disk myotherserver02
    #>
    [PoshBot.BotCommand(
        CommandName = 'grafana',
        TriggerType = 'regex',
        Regex = '^grafana\s(cpu|disk)\s(.*)'
    )]
    [cmdletbinding()]
    param(
        [parameter(ValueFromRemainingArguments = $true)]
        [object[]]$Arguments
    )

    $graphType = $Arguments[1]
    $computer = $Arguments[2]

    # Code to get the graph
    ...
}
```

> Note, with the `regex` trigger, you **MUST** use the `[PoshBot.BotCommand]` custom attribute and set the `Command` property to `$false` and the `TriggerType` to `regex`.
