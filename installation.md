
# Installation

Follow these steps to install and set up the **yNodes Plugin** on your Minecraft server.

### Requirements

- **Java 17+** (for Minecraft 1.19+ compatibility).
- A **Spigot/Paper** server running a supported Minecraft version. yNodes is developed for Minecraft 1.16.5 originally and has been updated through Minecraft **1.21.1** (NMS integration is compatible up to 1.21.1 according to the code comments).

### Installation Steps

1. **Download yNodes**
   - If you have a pre-built `yNodes.jar`, simply place it in your server’s `plugins/` directory.
   - To build from source, clone the repository and run `./gradlew build` in the `yNodes` project root – the plugin JAR will be output to `build/libs/`.

2. **Dependencies** (Optional but recommended)
   - *ProtocolLib*: Optional. If installed and `useNametags` is enabled in `config.yml`, yNodes will use ProtocolLib to update player name tags for prefixes/suffixes.
   - *Dynmap*: Optional. If present, yNodes will create a Dynmap layer showing town territories.
   - *PlaceholderAPI*: Optional. If present, yNodes registers placeholders for town/nation names, etc.
   - *CoreProtect*: Optional. If installed, yNodes integrates a `/town coinspect` feature that allows town staff to temporarily use CoreProtect’s inspector in their town.

### After Installation

After the first run, yNodes generates a `config.yml` in the `plugins/yNodes/` folder. Be sure to review and adjust these settings to suit your server needs.
