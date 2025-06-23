# yNodes Plugin

**yNodes** is a comprehensive town-and-nation Minecraft plugin (Spigot/Paper) that lets players create towns, form nations, claim territory, wage wars, engage in diplomacy, and manage economies. It is a fork of the Phonon **Nodes** plugin, updated and expanded with rich features like **territory control**, **flag war system**, **alliances/peace treaties**, **plot sub-claims**, **religion/ideology mechanics**, **chat channels**, and more. This README provides an up-to-date guide to all systems, commands, architecture, and workflows implemented in yNodes.

## Features

* **Towns & Nations:** Players can found **towns**, invite residents, and then unite towns into **nations** for mutual benefit. Towns claim land chunk-by-chunk as **territories**, have leaders/officers, and can ally or fight with other towns. Nations have a capital town and consolidate member towns under one banner (e.g. a nation’s capital town leader effectively leads the nation). Towns and nations are assigned random map colors and can be renamed or colored by leaders. Town and nation membership defines diplomacy, chat channels, and war participation.

* **Territorial Claims:** Land claiming is chunk-based. A new town starts with its founding chunk as **home territory** and can expand claims using `/town claim`. Each territory is tracked with a unique ID and may contain **resource nodes** (e.g. ore deposits, farms). The world map is divided into territory chunks stored in persistent data. Towns have a claim power system controlling how many chunks they can own (with possible bonuses or penalties). Unclaiming territory (`/town unclaim`) frees land. A **map view** (`/town map`) is available to visualize nearby territories in chat, and a toggleable sidebar **minimap** (`/town minimap`) can show an ASCII map of chunks.

* **Economy & Resource Nodes:** Every territory can yield resources which contribute to a town’s **income**. Resource nodes (categories like *crops*, *animals*, *ore*, etc.) define what a territory produces. The plugin periodically generates resources for each territory owned by a town. These accumulate in a **Town Income** inventory which town leaders/officers can access via `/town income`. The income inventory holds items (materials, crops, etc.) which can be withdrawn but not deposited (insertion is blocked to prevent exploits). Towns must manually collect the income; an **income tick** runs in the background to update these resources, and admins can force an update with `/nodesadmin runincome`. The **Ore Bonus** system allows admins to trigger double-ore events using `/orebonus <multiplier> <duration>` (e.g. `x3` drops for 60 seconds). An ore bonus affects server-wide ore yields for the duration, or can be reset with `/orebonus reset`.

* **Flag War System:** yNodes implements a **flag war** mechanic inspired by Towny’s wartime system. During a war, players can place a **“flag” block** in enemy territory to begin a **conquest timer**; if the timer completes, that chunk is captured by the attacker. A town’s **home territory (“core” chunk)** is especially important – if it’s taken, the territory becomes “occupied” by the attacker. War is an **opt-in state** toggled by admins (war can be globally enabled/disabled). When war is active, normal protections change: towns may lose land to enemies and cannot perform certain actions (configurable via rules like whether new towns can be created or players can leave towns during war). The plugin tracks a **War Score** for ongoing wars, which increases or decreases based on territory captures and possibly other factors (kills, etc.). Players can check the current war score with `/war score`.

* **Diplomacy – Alliances, Truces, Treaties:** Beyond wars, towns and nations can forge alliances or negotiate peace:

  * **Alliances:** Two towns or nations can become allies by mutual agreement. Using `/ally <town|nation>` sends an alliance request to the target. The target must also use the `/ally` command naming the requester to accept. Once allied, towns cannot declare war on each other and share an **ally chat** channel. Alliances can be dissolved with `/unally <name>` (no mutual confirmation needed). Only nation capitals or independent town leaders (and their officers) can offer or accept alliances. If a request is pending, using `/ally <name>` again by the other party will form the alliance, which is broadcast server-wide. (The system prevents alliance with enemies or duplicate requests.)
  * **Truces:** After a war, a temporary **truce period** prevents immediate re-declaration of war. The `/truce` command shows the list of current truces involving your town and how long until they expire. For example, if Town A and Town B signed a peace treaty, a truce might last a set time (configurable) during which neither can declare war. Players can also view another town’s truce status with `/truce <town>`. Truces are automatically managed by the plugin when peace treaties are finalized.
  * **Peace Treaties:** To end an active war, towns negotiate a treaty using `/peace <town|nation>`. Offering peace opens a **Treaty GUI** where terms can be set and viewed by both sides. When a treaty is offered, members of both communities get notified to use `/peace` to negotiate. During negotiation, war is paused (no further territory can be captured). The actual terms (surrender conditions, payments, etc.) would be agreed externally or via an integrated GUI if implemented. Once terms are settled, leaders/officers from each side finalize it with `/treaty finalize`. Finalizing ends the war – territories occupied may be annexed or returned depending on terms, and a truce period begins. If a treaty negotiation is initiated but not yet accepted, it remains active until either finalized or presumably cancelled by renewed conflict.

* **Religion & Ideology Systems:** yNodes adds **religion** and **ideology** as customizable concepts for towns and nations. Players can create a religion or ideology via an in-game GUI, allowing roleplay or additional mechanics (e.g. cultural influence). Using `/religion` opens a **Religion menu** for the player, where they can found a new religion or join one (the specifics of conversion are handled in the GUI). Towns and nations can have an **official religion** and ideology, and these are stored in their data. Religions/ideologies might exert “pressure” on neighboring towns (this is hinted by fields like `pressurePower` in data). Only OPs can access the admin menu `/religion menu` to manage all religions. Similarly, `/ideology` opens the Ideology menu (with an admin mode via `menu` argument for ops). These systems are largely GUI-driven; players do not directly type commands to set beliefs – instead, they use the menu to create or choose a religion/ideology, and the plugin’s **ReligionManager/IdeologyManager** handles registration and persistence. The religion and ideology features add depth (e.g. nations can have a state religion and towns can have local religions, which are saved and loaded to disk) but do not force gameplay changes unless configured (server admins can decide how to leverage these socially).

* **Land Plots & Chest Protection:** Within a town’s territory, residents can carve out personal **plots** of land. A plot is a sub-zone defined by two corner positions (like WorldEdit selections) and is owned by a resident. The `/plot` command allows town members to manage these plots:

  * `/plot select` – Enable selection mode to choose two corners with left/right-click.
  * `/plot create <name>` – Claim the selected area as your plot (must be within your town’s territory). The selection must lie entirely in one territory chunk and cannot overlap existing plots. Upon creation, the plot is recorded in town data and saved persistently.
  * `/plot trust <plot> <player>` – Give another player access to your plot. Trusted players can build or open chests there. `/plot untrust <plot> <player>` to revoke access.
  * `/plot transfer <plot> <player>` – Transfer ownership of a plot to another resident.
  * `/plot delete <name>` – Remove a plot (must be owner or town leader). This frees the land for general town use.
  * `/plot show` – Temporarily display the borders of your plots (e.g. by particle effects or highlights).
  * *(There is also `/plot save`, used to manually force saving plots to disk, mainly for debugging.)*

  Town leaders can manage **trusted** status globally via `/town trust <name>` (mark a resident as trusted in town) and `/town untrust <name>`. Trusted status, and **town permission settings**, control who can interact in the town. Towns have configurable **permissions** for various actions (build, destroy, use items, open chests, etc.) for different groups (town members, allies, nation members, outsiders) via `/town permissions [type] [group] [allow|deny]`. For example, a town could `/town permissions build ally deny` to prevent allies from building in its territory. These settings are stored per town and enforced by event listeners.

* **Custom Chat Channels:** The plugin provides multiple in-game chat channels for more immersive communication:

  * **Global Chat:** All players server-wide. Typing normally goes to global by default, but players can toggle to other channels. `/globalchat` or `/gc` toggles your chat mode to **Global**. With no arguments, it switches your active channel to global; with subcommands `/gc mute` or `leave` you can temporarily silence global chat (meaning you stop receiving it) and `/gc unmute` or `join` to enable it again. *(Global is enabled by default; these commands mainly allow opting out if desired.)*
  * **Local Chat:** Range-limited chat (e.g. only heard by players nearby in the world). `/localchat` or `/lc` toggles Local channel. Similar subcommands `/lc mute` and `/lc unmute` let you leave or rejoin local chat listening.
  * **Town Chat:** Private chat for members of your town. `/townchat` or `/tc` sets your chat mode to Town. When active, messages you send will only be seen by fellow town members. There is a `/tc leave` option to leave the town channel (though by default players are always in their town chat unless they switch; the `leave` subcommand is a placeholder for future use).
  * **Nation Chat:** Similar to town chat, but includes all players in your nation (across all member towns). `/nationchat` or `/nc` toggles Nation channel. Only members of the same nation see these messages.
  * **Ally Chat:** Communication channel shared between your town and any allied towns/nations. `/allychat` or `/ac` toggles Ally channel. This is useful for coordinating with allies during war or joint events. (Allied towns and nations share one combined ally chat.)

  *Mechanics:* Players can switch channels on the fly; the plugin prepends messages with appropriate tags. Using a channel command with no args toggles your **active speaking channel** (e.g. after `/tc`, your messages go to Town chat until you switch). The plugin maintains each player’s `ChatMode` (GLOBAL, LOCAL, TOWN, NATION, ALLY). Chat channel membership is automatic based on town/nation; however, global and local can be muted/unmuted explicitly. The **tab-completion** for channel commands offers “mute/unmute/join/leave” options where applicable. By default, upon login players have Global and Local chat enabled and can read both; switching channel only affects where **their** messages go. The plugin also supports **local radius** (configured range for local chat in blocks) and **channel prefixes** (e.g. “\[Town]”, “\[Nation]”) through its `Chat` module.

* **Name Tag Customization:** Towns can set prefixes and suffixes that appear before/after player names in chat or above their head. `/town prefix [text]` and `/town suffix [text]` let a player set their own title (e.g. town rank or nickname). Town leaders can also set or remove another member’s prefix/suffix (`/town prefix <player> <text>`). These tags integrate with PlaceholderAPI for use in scoreboard plugins or above-head name displays. yNodes also integrates with **ProtocolLib** (if present) to allow modifying the actual name tags above players: the plugin can spawn invisible marker armor stands to display town/nation tags or apply scoreboard teams to color names by town relationship (friendly, enemy, etc.). For example, towns have pre-generated nametag formats for how their name appears to neutrals, allies, enemies, etc. (see code: `[TownName]` in green for same-town, red for enemies, etc.). If ProtocolLib is installed and `useNametags` is enabled in config, yNodes will use NMS to show custom nameplates. (The plugin’s NMS module creates packets to set ArmorStand name visibility, marker status, etc..)

* **Dynmap Integration:** If the **Dynmap** plugin is installed, yNodes will automatically output town and world data to JSON files for Dynmap to display territory overlays. On startup, yNodes detects Dynmap and logs its usage. The plugin writes `nodes/world.json` and `nodes/towns.json` in Dynmap’s web directory, containing information about territories and town locations. These JSON files are updated asynchronously whenever the world data saves, allowing Dynmap to render an up-to-date map of town claims (often with each town’s color and name on their lands). In addition, an asynchronous task continuously writes out updates (using `NodesDynmapJsonWriter`) so that changes (new claims, wars, etc.) reflect on the live map. Dynmap support is a simple flag overlay (yNodes doesn’t draw fancy icons, but provides raw data that can be styled via Dynmap’s custom layer settings). If Dynmap is not present, these JSON outputs are skipped. There is also a config option `dynmapCopyTowns` that when enabled will write the towns JSON even if Dynmap plugin isn’t detected (useful for external map tools).

* **PlaceholderAPI Support:** yNodes includes a `NodesPlaceholderAPIExtension` that registers placeholders for use in other plugins (like chat formatters or scoreboards). Once the plugin is running, you can use placeholders such as `%nodes_town%`, `%nodes_nation%`, `%nodes_prefix%` etc., to display a player’s town name, nation name, or town prefix, respectively. This integration is automatically enabled if PlaceholderAPI is present, and it updates in real-time as town membership or prefixes change.

* **Data Storage & Backup:** All persistent data (towns, nations, residents, territory) is saved to disk in JSON format. The main files are `world.json` (containing territory/resource definitions) and `towns.json` (containing the dynamic data: towns, nations, residents) in the plugin’s folder. On server shutdown (or plugin disable), yNodes performs a final save of all data and also saves religion and ideology data and war score to their own files. The plugin supports periodic **auto-backups**: it keeps track of `lastBackupTime` and can be configured to dump backups on intervals (notably, `Nodes.lastBackupTime` is loaded on startup, though backup scheduling might require enabling in config). Additionally, **Plot Auto-Save** is run for player plot data to ensure plot changes persist regularly. The data loading on startup first clears any in-memory data structures and then deserializes `world.json` and `towns.json` back into objects. If anything goes wrong with loading (file missing or corrupted), by default the plugin will disable world interactions to prevent further damage. Admins can use `/nodesadmin save` and `/nodesadmin load` to manually trigger saving or reloading the world data from disk (useful for backing up before a risky operation or reloading changes made offline).

&#x20;*Figure 1: Core data model – relationships between Residents (players), Towns, Nations, and Plots. A town consists of many residents and may belong to a nation; a nation contains multiple towns. Plots are subdivisions of a town’s territory owned by individual residents. Arrows indicate reference and multiplicity.*

![Figure 1](https://i.imgur.com/4qUIpbR.png)

## Installation

**Requirements:** Java 17+ (for Minecraft 1.19+ compatibility), and a Spigot/Paper server running a supported Minecraft version. yNodes is developed for Minecraft 1.16.5 originally and has been updated through Minecraft **1.21.1** (NMS integration is compatible up to 1.21.1 according to the code comments).

**Download:** If you have a pre-built `yNodes.jar`, simply place it in your server’s `plugins/` directory. Otherwise, to build from source you will need **Gradle**. Clone the repository and run `./gradlew build` in the `yNodes` project root – the plugin JAR will be output to `build/libs/`. (No additional compilation steps are needed since yNodes uses the Gradle build to handle shading Kotlin and dependencies.)

**Dependencies:** yNodes can run standalone, but it **soft-depends** on certain plugins for full functionality:

* *ProtocolLib* – Optional. If installed and `useNametags` is enabled in `config.yml`, yNodes will use ProtocolLib to update player name tags for prefixes/suffixes.
* *Dynmap* – Optional. If present, yNodes will create a Dynmap layer showing town territories.
* *PlaceholderAPI* – Optional. If present, yNodes registers placeholders for town/nation names, etc. (e.g. `%nodes_town%`).
* *CoreProtect* – Optional. If installed, yNodes integrates a `/town coinspect` feature that allows town staff to temporarily use CoreProtect’s inspector in their town (essentially a shortcut to `/co inspect`). This requires CoreProtect and appropriate permission.
* *Kotlin* – Not needed explicitly. yNodes is written in Kotlin but the plugin JAR by default bundles Kotlin runtime. It will work out-of-the-box. (If running multiple Kotlin plugins, you can set up a shared Kotlin runtime plugin and build yNodes with `-P no-kotlin`, but this is advanced use.)

**Configuration:** After first run, yNodes generates a `config.yml` in the `plugins/yNodes/` folder. Key settings include claim limits, war rules (whether war is enabled by default, war timer durations, truce duration, etc.), chat settings (local chat radius), and Dynmap options (e.g. `dynmapCopyTowns`). Review and adjust these settings to suit your server. Most changes require a server restart or plugin reload (`/nodesadmin reload config`) to take effect.

**Upgrading from Phonon Nodes:** If you have data from the original Nodes plugin, you can migrate the JSON files. Ensure the old `world.json` and `towns.json` are placed in `plugins/yNodes/` directory (rename if needed). yNodes will attempt to deserialize them on startup. Always backup your data beforehand. Due to being a fork, yNodes should recognize the data structure, but test in a staging server first. New fields (like religion/ideology) will be initialized with defaults if absent.

## Usage & Commands

yNodes provides a robust set of commands. Almost every feature is accessible via commands, which are listed below by category. In-game, players can use `/town help`, `/nation help`, etc., for brief guidance. Here we document each command and its purpose. **Note:** In the commands below, **\[square brackets]** denote arguments, and vertical bars **( | )** denote options.

### Town Commands (Town Management)

The `/town` command (alias `/t`) is the core for managing your town. It has numerous subcommands for different actions:

* **`/town create <name>`** – Create a new town with the given name at your current location. You become the town **Leader**, and the chunk you are standing in becomes the town’s first claimed territory (home). Costs or requirements (if any) are configurable. *Usage example:* `/town create Springfield`.

* **`/town delete`** (alias `/town disband`) – Disband your town, deleting it permanently. Only the town leader can do this. All claims are unclaimed and residents are evicted. *No undo!* Use with caution.

* **`/town invite <player>`** – Invite a player to join your town. Only leader/officers can invite. The invited player can use `/town accept` or `/town deny` (or `/reject`) to respond.

* **`/town apply <town>`** (alias `/town join <town>`) – Request to join a town. If a town is open (no invite needed) or if an officer reviews applications, this puts your name in that town’s pending list. Town officers can then `/town accept <player>` or `/town deny <player>` an application.

* **`/town accept <player>` / `/town deny <player>`** – Accept or reject a specific player’s join request. Also used by invited players to accept/decline an invitation (in that context no `<player>` argument is needed, they just type `/town accept` to accept the invite they received).

* **`/town leave`** – Leave your current town. If you are the leader, typically you must transfer leadership or disband instead. If a regular resident leaves, their personal plots (if any) revert to town ownership.

* **`/town kick <player>`** – Remove a resident from your town. Leader/officers only.

* **`/town promote <player>` / `/town demote <player>`** – Give or revoke **Officer** status to a resident. Officers can invite/kick and perform some management tasks.

* **`/town officer <player>`** – (Same as promote; an older alias).

* **`/town leader <player>`** – Transfer leadership to another member. Makes that resident the new town leader. Only the current leader can do this.

* **`/town list`** – List all towns on the server. Shows the count of towns and may list their names (and possibly number of residents).

* **`/town info`** – Show detailed info about *your* town: town name, leader, officers, number of residents, list of territories, nation (if any), allies, enemies, etc. Use `/town info <town>` to view info about another town (even if you’re not a member).

* **`/town online`** – List online players in your town. `/town online <town>` shows online members of another town.

* **`/town spawn [outpost]`** – Teleport to your town’s spawn point. Each town has a primary spawn at its home base set by `setspawn`. If you have outposts (secondary bases), you can do `/town spawn <outpostName>` to go to that outpost’s spawn. This may cost money/items if configured. *(Requires a permissions plugin or economy plugin integration if costs are enabled.)*

* **`/town setspawn`** – Set the town’s spawn location to your current position. Typically must be within the town’s home chunk. Leader only.

* **`/town home`** – *Alias for `/town spawn` on some servers.* (Not explicitly listed in code, but commonly configured as an alias.)

* **`/town claim`** – Claim the chunk you are currently standing in for your town. This increases your town’s territory. Claiming might require you to be adjacent to existing town land and have enough **claim power** available. Only leader/officers (or anyone with `town.claim` permission) can use.

* **`/town unclaim`** – Unclaim the current chunk, relinquishing it from your town. (Often used if you claimed wrong land or to shrink town.)

* **`/town map`** – Show an ASCII map of the chunks around you, indicating which are owned and by whom. The map legend uses colored symbols for **Town land**, **Allied land**, **Enemy land**, **Unclaimed**, etc., and labels your position and directions. This is printed in chat.

* **`/town minimap [size]`** – Toggle a continuous minimap on your sidebar (using the boss bar or scoreboard) showing territory around you. The optional size can be 3, 4, or 5 (defaults to 5 chunks radius). This is useful for navigation as you move.

* **`/town color <r> <g> <b>`** – Set your town’s color on the Dynmap map (and in text displays). The color is in RGB values 0-255. Leader only.

* **`/town rename <newName>`** – Change your town’s name. Leader only. The new name must not conflict with an existing town’s name.

* **`/town announce <title|actionbar> <message>`** – Send a formatted announcement to all online town members. You can choose to send it as a Title or as an ActionBar message on their screen. For example: `/town announce title Welcome to the town!` shows “Welcome to the town!” as a big title screen for members. (Officer or leader only by default.)

* **`/town alerts`** – Toggle on/off receiving alert messages on your action bar for hostile actions. When enabled, if an enemy starts breaking blocks or a flag is placed in your town, you’ll see an alert (if war/raid is happening). When off, you won’t see these alerts.

* **`/town prefix [prefix]`** – Set a personal chat prefix for yourself, visible next to your name (e.g. titles like “\[Sheriff]”). `/town prefix remove` removes your prefix. Leaders/officers can set another player’s prefix: `/town prefix <player> <prefix>` (or remove it with `<player> remove`).

* **`/town suffix [suffix]`** – Similar to prefix, but sets a suffix after your name. Usage mirrors `/town prefix` (including removal and setting for others).

* **`/town permissions <type> <group> <allow|deny>`** – Adjust town permission flags. **Type** can be `build`, `destroy`, `interact`, `chests`, `items`, or `income` (for collecting income). **Group** can be `town` (town members), `nation` (nation-mates), `ally` (allied outsiders), `outsider` (strangers), or `trusted` (trusted outsiders). For example, `/town permissions build ally deny` would prevent allied players from building in your town. These settings determine who can do what in your town territory and are enforced by the plugin’s event listeners. By default, only town members can build/destroy, etc., but you might open interaction to allies or nation members via these flags.

* **`/town protect [show]`** – Toggle protection mode to (un)protect containers in your town. While active, you can click chests, furnaces, etc., to mark them as protected (only accessible by town members) or unprotected. Using `/town protect show` highlights currently protected blocks. (Protected blocks are stored in `town.protectedBlocks` and prevent outsiders from opening them.) This is an alternative to using a separate lock plugin, integrated into yNodes.

* **`/town trust <player>` / `/town untrust <player>`** – Mark a town member as **trusted** (or remove trusted status). Trusted players can bypass certain restrictions in town (depending on config, they might access protected chests or build in areas where outsiders can’t). It’s a way to differentiate full members from newer members, for example.

* **`/town capital`** – Move the town’s **home block** to your current location. This changes which territory chunk is considered the “capital” or home chunk of the town (useful if you want to designate a new main base). It also updates the town spawn to that chunk. Usually has a cooldown (to prevent abuse of moving capital frequently).

* **`/town annex`** – Annex an occupied territory into your town. This is used after a war when your town has occupied enemy chunks. Running `/town annex` in an occupied chunk that your town controls will convert it into a proper claimed territory of your town (consuming claim power). Essentially, it transfers permanent ownership to your town.

* **`/town outpost`** – Manage **outposts** (secondary bases) of your town. Subcommands include:

  * `/town outpost list` – List all outposts and their coordinates.
  * `/town outpost setspawn` – Set the spawn point of the outpost you are currently at.

  Outposts allow towns to have multiple “spawn” locations (e.g., a distant colony). They count as additional claims and typically require special permission or cost. (Note: to create an outpost, a common method is claiming land disconnected from your main town, which automatically becomes an outpost. The plugin’s outpost system is still basic, mainly providing separate spawn teleports.)

* **`/town fly`** – Toggle flight within your town territory. When enabled, town members can fly (as if in creative mode) but only inside their town’s owned chunks. If they leave town land, flight is usually disabled (and they fall). This is often reserved for donators or specific roles. Note: This requires the server to have flight allowed or handled via the plugin; yNodes hooks into this if configured. On toggle, it either allows you to fly in-region or not.

This list is long, but covers the **full town command functionality**. Typical daily use commands are `/town create/delete`, `/town invite/accept/leave`, `/town claim/unclaim`, and `/town chat` commands. Management commands like `/town permissions` and `/town protect` are used as needed by town leadership.

### Nation Commands (Nation Management)

Nation commands (`/nation` or alias `/n`) let town leaders band together into nations and manage nation-level settings:

* **`/nation create <name>`** – Form a new nation with your town as the capital. Only a town that is *not already in a nation* can do this, and usually only the town leader can execute it. This consumes your town (and any member towns, if you had allied towns that want to join simultaneously – though typically nation creation starts with one town). After creation, your town becomes the capital and you (the town leader) are effectively the nation leader. The nation now exists and you can invite other towns.
* **`/nation delete`** – Disband your nation. Only the nation’s leader (capital town’s leader) can do this. It will remove the nation tag from all member towns (they become independent). Use carefully.
* **`/nation invite <town>`** – Invite another town to join your nation. Only the capital town’s leader or officers can invite. The target town’s leader then can use `/nation accept` or `/nation deny` to respond (similar to town invites). Note: The invited town must not already be in a nation.
* **`/nation leave`** – Leave your current nation. This is used by a **town leader** to remove their town from the nation. It can only be used by non-capital towns (the capital cannot leave its own nation except by deleting the nation). If a town leaves, its residents no longer have nation benefits or nation chat with that group.
* **`/nation capital <town>`** – Change the capital of the nation to another member town. Only current nation leader can do. This effectively transfers nation leadership to that town’s leader. Useful if the capital town’s leader is stepping down – they can designate another town as the new capital.
* **`/nation list`** – List all nations on the server and their member towns.
* **`/nation info [nation]`** – Show info about your nation or a specified nation. Includes the nation’s name, capital, leader, list of towns, total residents, allies, enemies, and official religion/ideology if any.
* **`/nation online [nation]`** – List online players in your nation. If a nation name is provided, lists online players of that nation.
* **`/nation color <r> <g> <b>`** – Set the nation’s map color (applied to all member town areas on Dynmap). Capital leader only. Similar to `/town color` but for nation. Helps distinguish nations on the map.
* **`/nation rename <name>`** – Change the nation’s name. Nation names must be unique.
* **`/nation announce <title|actionbar> <message>`** – Send an announcement to all online nation members. Only the nation leader (capital’s leader) or capital officers can use. Behaves like town announce but on nation scale, with a cooldown (to prevent spam): default cooldown of 5 minutes for titles, 2 minutes for actionbars.
* **`/nation spawn <town>`** – Teleport to a town in your nation. For larger nations, the capital may wish to visit member towns. This command allows nation members to warp to any town within their nation, possibly at a resource or item cost (configurable). e.g. `/nation spawn Gotham` takes you to Gotham’s town spawn if it’s in your nation. If costs are enabled, it might consume certain items (like a portion of the distance in gold or a special token).

Nations fundamentally serve to formalize alliances between towns. By creating a nation, allied towns share a nation chat channel and have a structured hierarchy. Nations can also engage in diplomacy: entire nations can ally with or declare war on other nations. In war, all member towns of one nation become enemies of the other nation’s towns automatically. The **allies/enemies** at the nation level are tracked similarly to towns (nations have `allies` and `enemies` sets). If two nations are allied, all their towns are effectively allied. If nations are enemies (from a war or set via admin command), all towns under them are enemies.

### War & Diplomacy Commands

These commands govern declaring war, making peace, and managing allies/enemies outside of the town/nation creation context:

* **`/war [town|nation]`** – Declare war on another town or nation. This command opens an interactive GUI for the player to select a valid target and a **Casus Belli** (cause for war). Only the leader (or officers) of a nation’s capital town can initiate war, and only if war is globally enabled by admins. If a town is independent (not in a nation), its leader can declare war on another independent town. Usage: `/war <targetName>` directly attempts war if valid, but typically just `/war` will prompt the GUI. The plugin prevents declaring war on yourself, on allies, or if a truce is active. When war is declared, both sides are notified by a broadcast message including the cause. During war, players can capture each other’s territory using the flag war mechanics described earlier. The `/war score` subcommand shows your town’s current war score if any war is ongoing involving you.
* **`/peace [town|nation]`** – Offer peace to end a war. This initiates a peace treaty negotiation as explained in *Diplomacy* above. If the target is currently at war with you (enemy town or nation), a GUI opens allowing treaty terms selection. Essentially, one side starts with `/peace EnemyName` and both sides then use the treaty GUI and `/treaty finalize` to conclude. If you simply want to check your current treaties or negotiations, use `/truce` instead. (If used outside of war context, `/peace` does nothing.)
* **`/truce [town]`** – View active truces (post-war peace cooldowns). With no argument, shows your town’s current truces and time remaining. With a town name, shows that town’s truce agreements. Helpful to determine how long until you can declare war again on a recent enemy.
* **`/treaty [finalize]`** – Manage an active peace treaty negotiation. With no args, opens the **War Treaty GUI** if a treaty negotiation involving your town is in progress (for example, to adjust terms). Only the town leader/officers can use the GUI. With the `finalize` argument, it attempts to finalize the treaty immediately. This will officially end the war **only if** both sides have agreed (the War Score Manager checks that all parties accepted). Basically, each side needs to run `/treaty finalize` once they’re satisfied, at which point the war ends. If you try to finalize and no treaty is active or you’re not authorized, you get an error.
* **`/ally <town|nation>`** – Propose an alliance with another town or nation. If your town is in a nation, only your nation’s capital (or that nation’s leader) should use this to ally another nation. If independent, it allies town-to-town. The command acts as a toggle/request: if the target isn’t currently allied, it sends a request; if your town/nation already has a pending request **from** that target, using `/ally <target>` will accept it and form the alliance. Must be leader or officer to use. After sending, both towns will see messages (“offering alliance” and instructing the other to use `/ally <YourName>` to accept). There is no GUI; it’s done via these commands and chat feedback. When accepted, a broadcast announces the new alliance.
* **`/unally <town|nation>`** – Break an existing alliance. This immediately removes the alliance status; typically only leaders can do this. Both parties are informed that the alliance is broken. There is no confirmation required from the other side to unally.

Keep in mind that alliances affect war: you cannot declare war on an allied town/nation (the plugin checks and blocks it). Allies also share the ally chat channel and cannot hurt each other in friendly fire (if enabled). Truces similarly block war until expired. The war/peace system is one of the most complex parts of yNodes, involving interactive steps:

&#x20;*Figure 2: High-level flow of a war from declaration to treaty. Town leaders declare war via `/war`, engage in flag capturing battles, then negotiate peace with `/peace` and finalize the treaty with `/treaty finalize`. After war ends, a truce period prevents immediate re-hostility.* 

![Figure 2](https://i.imgur.com/z6is5Lw.png)

### Plot and Territory Commands

* **`/plot`** – This command (no arguments) shows a quick help list of plot subcommands, as described in the *Features* section. It is only usable by players (console cannot own plots). See earlier **Plot system** details for `/plot select`, `/plot create`, etc. Plot commands help players manage personal land within towns.

* **`/territory [id]`** – View territory information. This is essentially an alias of `/nodes territory` and provides info on town land claims:

  * With no argument (in-game), it will display info about the territory chunk you are currently standing in. It tells you whether the chunk is unclaimed or, if claimed, which town owns it, its coordinates/ID, and possibly resource node info.
  * With an ID argument (or if used from console), it looks up that specific territory by ID and prints details.

  This is useful for admins or for debugging map data. Players can use it to identify a territory’s ID to annex or for support tickets. The output includes ownership, coordinates, and any special status (occupied, core, etc.).

* **`/player [name]`** – Shows a summary of a player’s status in yNodes. With no name, if you run it yourself it prints *your* info (town, nation, rank). If a name is given, it looks up that player’s resident record and prints their town and nation, and possibly their titles or stats. For example, `/player Alice` might output “Player Alice – Town: Springfield, Nation: None, Town Role: Officer, Joined: <date>, Prestige: X”. This command is handy to check if someone is part of a town/nation and their role. It only shows information, no editing.

* **`/nodes [subcommand]`** – General **information commands** about the world and high-level data:

  * With no subcommand, `/nodes` displays the plugin name and version and a quick summary of how many resource nodes, territories, residents, towns, and nations are currently loaded. For example: “**Nodes v0.0.10** World info: - Resource Nodes: 5 - Territories: 120 - Residents: 34 - Towns: 4 - Nations: 2”. It then prompts you to use `/nodes help` for subcommands.
  * **`/nodes resource [name]`** – List resource node types or show details of one. If used as `/nodes resource` with no name, it will list all known resource node keys (like “Forest, Mine, Farm, Oil” etc., depending on server configuration). If a specific resource node name is given (e.g. `/nodes resource IronMine`), it prints that node’s properties – such as income rate, what crops or ores it yields, spawn chances for NPCs, etc. (These details are defined in config or data files; yNodes simply retrieves them and formats info via `resource.printInfo()`.) This command is mostly for admins or curious players to understand resource outputs.
  * **`/nodes territory [id]`** – Same as `/territory` command described above, to get territory info by ID or current location. The `/territory` alias is provided for convenience; both do the same thing.
  * **`/nodes town <townName>`** – Get an overview of a specified town. This prints similar info to `/town info <town>` (name, leader, residents count, etc.), but can be used by anyone without needing the `/town` context. It’s like a quick lookup. If no town is specified, the command might default to your town.
  * **`/nodes nation <nationName>`** – Similarly, shows an overview of a nation (like `/nation info`). If no name given, defaults to your nation.
  * **`/nodes player <playerName>`** – Shows info about a player (like `/player` command). Provided as an alternative way to check someone’s stats.
  * **`/nodes war`** – Displays whether the global war mode is enabled or disabled. `/nodes war` by itself simply prints “Nodes war status: enabled/disabled” along with some settings. It reminds you that toggling war must be done via the admin command `/nodesadmin war enable/disable`. (If war is enabled, it also lists the settings like whether annexation is allowed, only border attacks, destruction enabled, etc., in a detailed view.)

Overall, `/nodes` is an informational command for server-wide data, whereas `/town` and `/nation` are for personal town/nation management. Regular players will primarily use the town and nation commands, and maybe `/ally`, `/war`, etc., while `/nodes ...` and `/territory` are of more interest to admins or for debugging.

### Admin Commands (`/nodesadmin`)

All admin commands are under `/nodesadmin` (alias `/na`). These allow server staff to override or fix things and are not for players (requires appropriate permissions, typically op). **Use caution** as these can modify data directly. The subcommands include:

* **`/nodesadmin reload [config|managers|resources|territory]`** – Reload various parts of the plugin:

  * `reload config` – Reload the `config.yml` from disk without restarting.
  * `reload managers` – Reload internal managers (religion, ideology, etc.) if needed.
  * `reload resources` – Reload resource node definitions from the data file.
  * `reload territory` – Reload world territory data (likely from `world.json`). Generally, `reload` without arguments may reload everything. This is safer done by full server restart; use with discretion.

* **`/nodesadmin save`** – Force save the world data to disk immediately. This writes the `towns.json` file (and related data like religions, ideologies, war scores) to disk at that moment. Useful before shutting down or if you suspect an imminent crash.

* **`/nodesadmin load`** – Force a reload of world data from disk. This will discard current in-memory data (towns, etc.) and attempt to load from `world.json`/`towns.json` again. **Warning:** any unsaved changes will be lost. This is essentially a full data reset to last saved state.

* **`/nodesadmin war <enable|disable|skirmish|whitelist|blacklist|cleanup>`** – Toggle or configure global war mode:

  * `war enable` – Turn on war globally (allows towns to declare war).
  * `war disable` – Turn off war globally (immediately ends any ongoing wars and prevents new ones).
  * `war skirmish` – Enable **skirmish mode**, which is a limited war where territory annexation is disabled (only temporary occupations). This sets `canAnnexTerritories` false internally and might be used for events.
  * `war whitelist` / `war blacklist` – Possibly toggles using a whitelist of towns that *can* participate in war. (The config has `warUseWhitelist`; this command likely flips it or configures which towns are on the list. Not heavily documented, use carefully.)
  * `war cleanup` – Force remove all war-related markers (flags, occupied chunks) on the map. Use if something glitched (for example, if war was aborted but flags didn’t clear). This tries to restore occupied land to original owners and clear ongoing attack data.

* **`/nodesadmin resident <towncooldown>`** – Manage resident data. Currently the only subcommand is `towncooldown` which likely resets or checks the cooldown preventing a player from rejoining a town after leaving. (If a player leaves a town, they might have to wait X hours to join another – this command could override that for a specific resident, though usage syntax isn’t fully clear from code.)

* **`/nodesadmin town <...>`** – A suite of commands to directly manipulate town data. **Use extreme caution** as these do not perform normal game checks:

  * `town create <name>` – Create a town at your (admin’s) location without a player. (Might create a dummy leader or none; primarily for testing.)
  * `town delete <name>` – Delete the specified town. Instantly disbands it.
  * `town addplayer <town> <player>` – Force-add a player to a town (useful if a player glitched out of a town or to assign offline players).
  * `town removeplayer <town> <player>` – Force-remove a player from a town.
  * `town addterritory <town> <x> <z>` – Claim a specific chunk (by coordinates) for the town. Allows admins to give territory without the player being there.
  * `town removeterritory <town> <x> <z>` – Unclaim a specific chunk from the town.
  * `town captureterritory <attackerTown> <x> <z>` – Mark a chunk as occupied by attackerTown (as if captured in war). Possibly used to manually trigger war outcomes.
  * `town releaseterritory <town> <x> <z>` – Remove an occupied status, returning the chunk to original town (or unclaim if none).
  * `town addofficer <town> <player>` / `removeofficer` – Promote or demote a town officer by command.
  * `town leader <town> <player>` – Set a new leader for the town.
  * `town removeleader <town>` – Remove the leader (making it null? Or maybe it’s meant to be used after transferring leadership).
  * `town claimsbonus <town> <amount>` – Modify a town’s claim bonus (extra claim power).
  * `town claimsannex <town> <amount>` – Modify a town’s annexed-claims count (used to calculate penalties).
  * `town claimspenalty <town> <amount>` – Set a town’s current claims penalty (from over-claiming, etc.).
  * `town open <town> <true|false>` – Set whether a town is open to public joining (no-invite-needed). If open, players can `/town join <town>` freely.
  * `town income <town> [amount]` – Possibly to adjust town income or collect it by admin (not clearly documented; could remove or add items to income storage).
  * `town setspawn <town>` – Set town’s spawn to admin’s current location.
  * `town spawn <town>` – Teleport to that town’s spawn (admin bypass).
  * `town sethome <town> <x> <z>` – Change a town’s home chunk to specified coords (updates the core/home territory).
  * `town sethomecooldown <town> <minutes>` – Adjust a town’s move-home cooldown timer.
  * `town addoutpost <town> <name>` – Create a new outpost for the town at admin’s location and give it a name.
  * `town removeoutpost <town> <name>` – Delete the named outpost.
  * `town merge <sourceTown> <targetTown>` – Merge two towns together. Likely moves all residents and land from source into target, then deletes source. Use carefully as this can be complex (untested thoroughly).

* **`/nodesadmin nation <...>`** – Similar direct controls for nations:

  * `nation create <name> <town>` – Create a nation with the specified town as capital.
  * `nation delete <name>` – Delete a nation (remove all member towns from it).
  * `nation addtown <nation> <town>` – Add a town to a nation (without invite).
  * `nation removetown <nation> <town>` – Remove a town from a nation.
  * `nation capital <nation> <town>` – Set a different capital town for the nation.

* **`/nodesadmin ally <townA> <townB>`** – Force-set two towns (or nations, if using nation names) as allies. This bypasses the request system and instantly marks them allied. There’s also `allyremove` to break an alliance forcefully.

* **`/nodesadmin enemy <townA> <townB>`** – Force-set two towns/nations as enemies. This is usually used to manually toggle war stances or to set up scenarios. Equivalent to declaring war administratively (but does not start flag war, it just sets relations).

* **`/nodesadmin peace <townA> <townB>`** – Force towns to peace (remove enemy status). Essentially ends a war or hostility state immediately.

* **`/nodesadmin truce <townA> <townB>`** – Manually create a truce between two towns (so they cannot war for the truce period). `truceremove` to cancel a truce. Useful to enforce a peace cooldown or remove one.

* **`/nodesadmin treaty <...>`** – Manage global treaties. This might open a treaty GUI or set terms via command, but details are scarce. Possibly not fully implemented for command-line and meant to be via GUI only.

* **`/nodesadmin resource <add|remove|swap|create|edit|delete>`** – Edit **resource nodes** in the world:

  * `resource add <territoryID> <resourceName>` – Assign a resource node type to a territory. For example, `resource add 57 IronMine` would mark territory #57 as an Iron Mine node.
  * `resource remove <territoryID> <resourceName>` – Remove a resource assignment from a territory.
  * `resource swap <territoryID1> <territoryID2>` – Swap the resource nodes between two territories.
  * `resource create <name>` – Create a new resource node type (likely then requires editing its properties in config or via subsequent commands).
  * `resource edit <name>` – Edit properties of a resource type (possibly interactive or via further args).
  * `resource delete <name>` – Remove a resource node type entirely.
    These commands allow dynamic changing of the resource distribution without server restart. Only for admins – misuse can unbalance economy.

* **`/nodesadmin debug <resource|chunk|territory|resident|town|nation>`** – Output debug information to console about the specified subsystem. For instance, `debug chunk` might print data about the chunk you’re in (coords, territory ID, who owns it). `debug town` could print the internal object state of your town (all fields/values). These are mainly for developers or troubleshooting issues in the data.

As seen, `/nodesadmin` is quite powerful and should be limited to trusted admins. Many of its functions modify the same data that the plugin manages automatically, so use them to correct mistakes or implement events (like a story-driven war or merging towns by admin decision). Always backup data before using direct admin modifications.

## System Architecture

Internally, yNodes is structured into various components and data managers:

* **Singleton Container:** The core is the `Nodes` object which holds global collections and flags: lists of all towns, nations, residents, maps of territories, resource nodes, etc. It also holds references to key managers (War manager, ReligionManager, etc.). This acts as an in-memory database while the server is running.

* **Data Classes:** Key game concepts are represented by Kotlin classes:

  * `Resident` – represents a player (by UUID) and stores their town membership, chat mode, prefix/suffix, and runtime data like whether they’re inspecting with CoreProtect.
  * `Town` – represents a town. Contains fields for leader, officers, residents list, territories owned, and relations (allies, enemies, truce status). It also includes town-specific structures: e.g., an `IncomeInventory` for resources, `permissions` map for protection settings, `protectedBlocks` list for chest protections, and religion/ideology info (stored in `TownReligion` and `TownIdeology` objects).
  * `Nation` – represents a nation. Holds a set of member towns, a set of all residents (for quick access), allied nations, enemy nations, color, and religion/ideology info (in `NationReligion/NationIdeology`). The nation’s leader is determined by its capital town’s leader.
  * `Territory` and `TerritoryChunk` – represent claimed chunks. A Territory has an ID (likely a wrapper around coordinates or an index) and may have pointers to the owning town and possibly a resource node type. The plugin often uses a `Coord` value class for chunk coordinates and maps them to Territory objects.
  * `ResourceNode` – describes a resource-producing node (with name, type, yields). There are derived types or attributes like DefaultResourceAttributeLoader, and the system can load custom resource definitions from JSON on startup.
  * War-related classes: `FlagWar` (object managing war state), `Attack` (represents an ongoing flag attack on a chunk), `WarScoreManager` (tracks scores and treaties), and GUI classes for war (`WarGui`, `WarTargetGui`, `CasusBelliGui` which gather the war cause). War status like enabled/disabled and rules (canAnnexTerritories, etc.) are stored in `FlagWar` as flags and also in config.
  * `Religion` and `Ideology` classes – these hold info about each religion or ideology created (name, perhaps tenets). Each town or nation has a `TownReligion`/`NationReligion` which includes what religion is official and a “pressure” value indicating influence spread. Religion and Ideology managers handle saving/loading in JSON via `ReligionStorage` and `IdeologyStorage`.

* **Event Listeners:** The plugin registers numerous listeners on enable to enforce game rules:

  * Block events (grow, break) – e.g. cancel crop growth in unowned chunks or other custom effects.
  * Chat events – to route chat to channels (NodesChatListener).
  * Chest protection events – `NodesChestProtectionListener` and `NodesChestProtectionDestroyListener` prevent unauthorized opening or breaking of protected chests.
  * Alliance/Truce expiration – e.g. `NodesDiplomacyTruceExpiredListener` likely handles when a truce timer ends (to notify or allow war again).
  * GUI interaction – `NodesGuiListener` manages clicks in custom inventory menus (to handle when players choose an option in war/treaty/religion GUIs).
  * Movement and world events – `NodesPlayerMoveListener` might track entering/exiting territories for flight toggle or display messages like “Entering X town”. `NodesWorldListener` could handle preventing actions in disabled worlds or unconfigured worlds. `NodesPlayerJoinQuitListener` ensures a player’s Resident entry is created on join and removed or saved on quit (e.g., add player to town’s online set and update nametags). There’s also an AFK kick listener (NodesPlayerAFKKickListener) which could automatically kick idle players from the server or town if configured.
  * Economy events – `NodesIncomeInventoryListener` (described earlier) prevents item insertion into the income GUI and triggers a save on close. `BuildAttemptListener` might cancel building in enemy territory during war. `NodesSheepShearListener` and `NodesEntityBreedListener` might tie into resource production (e.g. limiting breeding or wool production to certain nodes).

* **Command Structure:** Each command class (e.g., TownCommand, NationCommand, etc.) implements Bukkit’s CommandExecutor (and TabCompleter). They parse arguments and call appropriate functions in the `Nodes` API or data objects. Many commands simply fetch the relevant object and call a method, for example:

  * `/town claim` -> TownCommand calls `claimTerritory(player)` which in turn calls `Nodes.addTerritory(town, chunk)` in the Nodes API, then maybe triggers a save and Dynmap update.
  * `/town invite` -> calls TownCommand `invite(player, args)` which adds an application entry for the target and maybe sends them a Message.

  The classes use `Message.print(sender, text)` for user feedback and `Message.error(sender, text)` for errors, centralizing message formatting (with color codes). Tab completers use utility filters to suggest names (e.g., `filterTown`, `filterResident` to autocomplete town or player names).

* **Persistence:** Data is saved in JSON through **serializer** classes. For example, `Serializer.worldToJson()` compiles all residents, towns, and nations into a single JSON string, and `Deserializer.worldFromJson()` reads it back. Town, Nation, Resident each have an inner `SaveState` class (immutable snapshot) that defines how to serialize to JSON. The plugin uses these snapshots to avoid concurrency issues: it updates objects on the main thread, then uses async tasks to write JSON to disk. This design prevents lag by offloading file I/O to another thread while ensuring data consistency via the snapshots.

* **Config & Tuning:** The `Config` object (and config.yml) contains many toggles:

  * War settings: durations for flag capture, war enabled flag, destruction rules, leave town during war allowed or not, etc.
  * Costs for actions: cost to create town, cost per claim, etc.
  * Chat radius for local, cooldowns for announce commands, whether to use dynmap, whether to use ProtocolLib for nametags, etc.
  * These are accessed via `Config.someSetting` throughout the code (likely loaded on startup from YamlConfiguration).

In summary, yNodes is a **comprehensive town/nation plugin** with a modular architecture: data stored in memory with periodic saves, command-driven interactions, and listener-enforced rules. The design emphasizes **persistent world alteration** (land claiming, war) and **player cooperation/competition** mechanics. By combining Towny-like land management with Factions-like war and additional RP elements (religions, ideologies), yNodes creates a unique gameplay experience.

## Contributing

Contributions to yNodes are welcome. If you are a developer, you can contact yonxel in discord to apply as a contributor in the project. Given that this is a fork of another project, please ensure your contributions align with the project’s goals and maintain code quality. Before coding a new feature or major change, it’s a good idea to open a discussion (issue) to verify it fits the scope of yNodes.

When contributing:

* Follow the existing code style (Kotlin conventions, use of `Message.print` for output, etc.).
* Test your changes thoroughly, especially since this plugin handles critical data (town ownership, etc.). Ideally, set up a test server and simulate scenarios (founding towns, declaring wars, etc.) to ensure no regressions.
* Document any new commands or config options in the README and in code comments.
* Keep performance in mind. yNodes operates across potentially hundreds of chunks and players, so avoid heavy computations on the main thread where possible (the plugin offloads JSON saves and uses Bukkit scheduler for repeated tasks).
* By contributing, you agree that your code will be licensed under the same license (GPLv3).

For any questions or to discuss development, you can reach out via the GitHub issues or any official Discord/Forum if provided.

*Acknowledgements:* This plugin is based on Phonon’s original Nodes plugin. Credit to the original authors and contributors of that project for the foundational systems (towns, flag war concept) which yNodes builds upon. Special thanks to all testers and community members who provided feedback to improve yNodes.
