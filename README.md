# EddieBot

![Develop workflow](https://github.com/EddieJaoudeCommunity/EddieBot/workflows/develop/badge.svg)

Discord bot for Eddie Jaoude's Discord server

## → Features of EddieBot

- Set/Get user bio with description and social links

- Timezone, listens for messages that contain 1:30pm UTC and replies with common timezones translation

- Code of Conduct

- Daily standup message consistently formatted

- Help showing a list of available commands

- Members role rewards

- Gets tips of resources on a given subject

- Server status, to display some statistics of the server (e.g total number of users and messages)

- Firebase (Firestore) integration, allowing people to easily add commands and persist data

- GitHub Actions deploys mainline branch to Azure

## → Requirements

- [optional] Docker and Docker-Compose
- discord token. To get one follow [these instructions](https://discordjs.guide/preparations/setting-up-a-bot-application.html#creating-your-bot)
- general channel ID of your discord server, read the instructions on one of these links to get yours:
  - [Get the channel ID of the Discord text channel](https://github.com/Chikachi/DiscordIntegration/wiki/How-to-get-a-token-and-channel-ID-for-Discord#get-the-channel-id-of-the-discord-text-channel)
  - [Where can I find my User/Server/Message ID?](https://support.discord.com/hc/en-us/articles/206346498-Where-can-I-find-my-User-Server-Message-ID-)
- ID of your discord server, to get your server ID follow these steps:
  - First make sure you have **Developer Mode** enabled on your Discord by visiting your Discord settings and going to Appearance
  - Right click on the server icon on the left hand sidebar then click on "Copy ID"
- firebase key for the project, check [these docs](https://firebase.google.com/docs/admin/setup) to get your key
- [optional] Github token that you can get from [here](https://github.com/settings/tokens)
- [optional] GCP account to deploy to (using GitHub Actions)

## → How to run EddieBot locally:

#### 1. Set Environment Variables

1. Copy `.env.example` to `.env`.
2. Generate the "Service Account" from Firebase Settings > Cloud Messaging.
   1. Download Service Account JSON file from this same screen.
3. Open `.env` and fill empty strings with matching credentials from the JSON file.

#### 2. To start the application
- [optional] To run with Docker ensure you have the latest version and Docker Compose installed and run
	- `docker-compose up`

- To run locally
  1. To setup: `npm run setup`
  2. To start: `npm run start:local`

---

### → Useful Commands

- **View Logs**

  `docker-compose logs --tail=all -f eddiebot-nodejs`

- **Use NodeJS instance CLI**

  `docker-compose exec eddiebot-nodejs /bin/bash`

### Standardized Code Formating

- **Before Commit**
  1. lint for errors: `npm run lint`
  2. fix any errors: `npm run lint:fix`

### Using Commitizen for commits
1. `git add <your files>`
2. `npm run commit`

### → Logging

Logging will happen to the console as well as to the Discord `bot` channel.

1. Include the logger object...

```typescript
import { log } from './logger';
```

2. Usage

```typescript
log.info('Message', 'Details');
```

or

```typescript
log.warn('Message', 'Details');
```

or

```typescript
log.error('Message', 'Details');
```

or

```typescript
log.fatal('Message', 'Details');
```

## Documentation
### Resources
- [Discord.js](https://discordjs.guide/)

---

### [Discord Gateway Intents](https://discordjs.guide/popular-topics/intents.html)

If you want the bot to receive a **new type of event**, you might need to add the required intent in `client.ts` to receive that event. Have a look at [discord.js docs](https://discord.js.org/#/docs/main/stable/class/Intents?scrollTo=s-FLAGS) for the list of intents.

#### Example adding ban capabilities to moderators:

1. Add the `GUILD_BANS` intent in `client.ts`:

```ts
export const client = new Client({
  partials: ['MESSAGE', 'CHANNEL', 'REACTION'],
  ws: {
    intents: [
      'GUILDS',
      'GUILD_MESSAGES',
      'GUILD_MESSAGE_REACTIONS',
      'GUILD_BANS',
      PrivilegedIntents.GUILD_MEMBERS,
    ],
  },
});
```

2. Add an event handler function for the events you want in `index.ts`:

```ts
client.on('guildBanAdd', (guild) => guildBanAdd(guild));
```

_Note: We are using the `GUILD_MEMBERS` **privileged intent** to receive the `guildMemberAdd` event. To know more about Privileged Intents check the [official docs](https://discord.com/developers/docs/topics/gateway#privileged-intents)_.

---

## How to add a new command to the bot
 If you are new to Discord bot development, here is a small introduction about the flow of something happening on the Discord server to the bot.
 
 ### Introduction
 When a message is sent by a user on the Discord server, the Discord app sends an event to the [Discord Gateway](https://discord.com/developers/docs/topics/gateway), which then sends that event to the clients that are connected to it with WebSockets, in our case [Discord.js](https://discord.js.org/#/).
 The start of the program is at `src/index.ts`, and here we register an event handler (the function exported on `src/commands.ts`) for the `message` event here. The client library ([Discord.js](https://discord.js.org/#/)) will call that function when it receives the `message` event from the Discord Gateway, and this is where the commands exported in the [src/commandHandlers/index.ts](https://github.com/EddieJaoudeCommunity/EddieBot/blob/develop/src/commandHandlers/index.ts) are used and where there is logic to decide which one to call.
 
 ### Steps to create a new command
 All the commands are located in the folder `src/commandHandlers` so that each command has its own file. They are then executed in `src/commands.ts` when a user types a command with the configured command prefix in [config.ts](https://github.com/EddieJaoudeCommunity/EddieBot/blob/develop/src/config.ts#L4).
 
To **create a new command**, follow these steps:

1. **Create a new file** on the `/commandHandlers` folder with the name of the new command.
   _Note: If you need to, use **camel case** for the file name, like_ `codeOfConduct.ts`
2. On this file you must **export a function** and **three variables**:

   - `command` - the function that executes the command. The **signature** of this function is the following:

   ```ts
   (arg: [string, string], embed: MessageEmbed, message: Message): Promise<MessageEmbed>
   ```

   The **arg** parameter contains the arguments given to the command in a string.
   The **arg** parameter is a tuple of two `string`s where, `arg[0]` is the command name and `arg[1]` is the string of command arguments.
   The **embed** parameter is a [MessageEmbed](https://discord.js.org/#/docs/main/stable/class/MessageEmbed) instance, and it represents the message that is returned by the bot, in response to the user. **The command should return this parameter or a new instance** with an appropriate message to the user.
   The **message** parameter is a [Message](https://discord.js.org/#/docs/main/stable/class/Message) instance that represents the message inputted by the user to execute a given command.

   - `description` - a string with a more detailed description of the command. Used for example by the [help command](https://github.com/EddieJaoudeCommunity/EddieBot/blob/develop/src/commandHandlers/help.ts)
   - `triggers` - a string array with the values that trigger this command. If the user types the configured command prefix followed by one of these values, the command **should be executed**
   - `usage` - a string explaining how the command is used (e.g. specifying the number of arguments and their separator)

3. After creating that file, you have to import it and add it to the exported list of commands on the [index.ts](https://github.com/EddieJaoudeCommunity/EddieBot/blob/develop/src/commandHandlers/index.ts) file located in this folder. Here is an example of adding the `standup` command:

```diff
import * as codeOfConduct from './codeOfConduct';
import * as help from './help';
+ import * as standup from './standup';
import * as stats from './stats';

- export default [codeOfConduct, help, stats];
+ export default [codeOfConduct, help, standup, stats];

export { fallback } from './fallback';
```

## How to add allowed word

All the allowed words are located in an array in [src/config.ts](src/config.ts) inside the object ` ALEX ` as shown below. 

```ts
ALEX: {
    profanitySureness: 2,
    noBinary: true,
    allow: [
      'just',
      'brother-sister',
      'brothers-sisters',
      'daft',
      ...

```
In order to add a new allowed word you have to do it inside of single quotes (` '' `) and for phrases (multiple words togheter) you should put a hiphon (` - `).

It must match **the following rules**: 
- [Rules for profanities](https://github.com/retextjs/retext-profanities/blob/main/rules.md)
- [Rules for equality](https://github.com/retextjs/retext-equality/blob/main/rules.md)

## Database

Using Firestore from Firebase.

[![](https://mermaid.ink/img/eyJjb2RlIjoiY2xhc3NEaWFncmFtXG5cdFVzZXIgLS0gQmlvIDogTmVzdGVkXG5cblx0VXNlcjogYmlvIChvYmplY3QpXG5cdFVzZXI6IGF2YXRhciAoc3RyaW5nKVxuXHRVc2VyOiBqb2luZWRBdCAoZGF0ZSlcblx0VXNlcjogdXBkYXRlZEF0IChkYXRlKVxuXHRVc2VyOiB1c2VybmFtZSAoc3RyaW5nKVxuXG5cdEJpbzogZGVzY3JpcHRpb25cblx0QmlvOiB0d2l0dGVyXG5cdEJpbzogZ2l0aHViXG5cdEJpbzogeW91dHViZVxuXHRCaW86IGxpbmtlZGluXG5cblx0XHRcdFx0XHQiLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoiY2xhc3NEaWFncmFtXG5cdFVzZXIgLS0gQmlvIDogTmVzdGVkXG5cblx0VXNlcjogYmlvIChvYmplY3QpXG5cdFVzZXI6IGF2YXRhciAoc3RyaW5nKVxuXHRVc2VyOiBqb2luZWRBdCAoZGF0ZSlcblx0VXNlcjogdXBkYXRlZEF0IChkYXRlKVxuXHRVc2VyOiB1c2VybmFtZSAoc3RyaW5nKVxuXG5cdEJpbzogZGVzY3JpcHRpb25cblx0QmlvOiB0d2l0dGVyXG5cdEJpbzogZ2l0aHViXG5cdEJpbzogeW91dHViZVxuXHRCaW86IGxpbmtlZGluXG5cblx0XHRcdFx0XHQiLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

Source

```
classDiagram
	User -- Bio : Nested

	User: bio (object)
	User: avatar (string)
	User: joinedAt (date)
	User: updatedAt (date)
	User: username (string)

	Bio: description
	Bio: twitter
	Bio: github
	Bio: youtube
	Bio: linkedin
```

---

If you are having trouble creating a new command, here is an [example](https://github.com/EddieJaoudeCommunity/EddieBot/blob/develop/src/commandHandlers/standup.ts).
Feel free to [create an issue](https://github.com/EddieJaoudeCommunity/EddieBot/issues) or make a PR with a new command 😃. Please see our [Contributing](CONTRIBUTING.md) file first, before making new commits or opening a PR. We appreciate it ❤️!

---

## Socials

Join our discord community [here](https://discord.gg/jZQs6Wu)
