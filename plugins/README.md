# Hytale Plugin Documentation

This folder contains documentation for developing Hytale server plugins.

## Contents

| File | Description |
|------|-------------|
| [HytaleServer-Plugin-API.md](./HytaleServer-Plugin-API.md) | Complete API reference for plugin development |
| [HytaleServer-Plugin-Examples.md](./HytaleServer-Plugin-Examples.md) | Practical examples plus external example index (27 projects) |
| [Structures-Guide.md](./Structures-Guide.md) | Guide for saving and loading structures (prefabs) |
| [Plugin-Starter.md](./Plugin-Starter.md) | Buildable starter template (Java 21 + manifest.json) |

## Quick Links

### API Reference Topics
- Project Setup (Maven/Gradle)
- Plugin Structure & Manifest
- Lifecycle Methods (setup, start, shutdown)
- Event System
- Command System
- Packet Adapters
- Inventory and ItemStack
- Custom UI
- Server Transfer (ClientReferral)
- Configuration
- Registries
- Dependencies

### Example Plugins
1. Custom Commands
2. Event Listeners
3. Scheduled Tasks
4. Configuration Files
5. Player Data Tracking
6. Chat Features
7. Block Interactions
8. Admin Tools
9. Economy System
10. Teleportation System
11. Cooldown System
12. Kits and Loadouts
13. AFK Detection
14. Announcements
15. Voting/Polls
16. Combat Logging Prevention
17. Home System
18. Private Messaging
19. Spawn System
20. Report System
21. Daily Rewards
22. Custom Items and Weapons
23. Crafting Recipes
24. Weather and Particles
25. Portals and Warps
26. Quests and Reputation
27. Permissions and Moderation

External example projects are referenced in `plugins/HytaleServer-Plugin-Examples.md`.

### Onboarding
- Buildable starter template: `plugins/Plugin-Starter.md`
- Verified use-case ideas: `plugins/HytaleServer-Plugin-Examples.md` (Use-Case Ideas section)
- Packet, inventory, UI, and transfer examples: `plugins/HytaleServer-Plugin-Examples.md`
- Command transfer example: `commandexamples.md` (ClientReferral)

## See Also

- [../HytaleServer.md](../HytaleServer.md) - Server documentation and CLI options
