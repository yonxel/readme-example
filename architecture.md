
# System Architecture

Internally, yNodes is structured into various components and data managers. The core is the `Nodes` object, which holds global collections and flags: lists of all towns, nations, residents, maps of territories, resource nodes, etc. It also holds references to key managers like WarManager, ReligionManager, etc.

## Key Components:
- **`Nodes` Object:** Centralized management of town, nation, and resident data.
- **Data Classes:** 
  - `Resident`: Represents a player and stores town membership, chat mode, and personal data.
  - `Town`: Represents a town, including details like leadership, residents, territories, and relations.
  - `Nation`: Represents a nation, containing member towns and nation-level settings.
  - `Territory`: Represents chunks of land claimed by towns or nations.

These components interact through commands, events, and data storage, allowing dynamic management of towns, wars, and resources.
