# haven-group-invites: design

Status: design only, no code yet. Mockups: `docs/mockups/index.html` (open it in a browser).

## Goal

MMO-style groups for the Haven server (NeoForge 1.21.1). Players invite each other into a group from the Social
Interactions screen or with a command. Group members can use each other's claims with the permissions that each party
owner sets. Players no longer need to remember FTB Teams and FTB Chunks commands.

## How it maps to FTB (FTB Teams 2101.1.11, FTB Chunks 2101.1.21)

- The group is the mod's own list, kept on the server. FTB has no group.
- For each group member who has an FTB party, every other group member becomes an FTB ally (rank `ALLY`) of that party.
- A member with no party shares nothing: FTB allies need a party.
- In a party's claims, the group follows that party's FTB Chunks settings. Only the party owner can change them
  (`/ftbteams party settings` needs rank `OWNER`; `allies add` and `allies remove` need `OFFICER`).
- A group formed by a plain party member still follows the owner's settings, even when the owner is not in that group.
- When a member leaves or is removed, the mod removes only the ally links it added. Allies added by hand stay.
- When a group member creates a party later, the mod adds the other group members as allies of it.
- Claims in a party belong to the party, so a party member cannot share only the chunks they claimed. Out of scope.

## Permissions panel (one per party, owner edits, others read-only)

A checked box sets the setting to `allies`, an unchecked box to `private`.

| Label | FTB Chunks setting |
|---|---|
| Build in our claims | `block_edit_mode` |
| Use blocks | `block_interact_mode` |
| Use villagers and animals | `entity_interact_mode` |
| Break decorations and vehicles (item frames, armor stands, paintings, minecarts, boats) | `nonliving_entity_attack_mode` |
| See our claims on the map | `claim_visibility` |
| See where we are on the map | `location_mode` |

`block_edit_and_interact_mode` is Fabric only and is left out.

## Screens

- Social Interactions gets a fourth tab, **Group**.
- All tab: a green + button with a tooltip ("Invite X to your group") on each player who is not in your group.
  Rows show "In your group" or "Invited".
- Group tab:
  - **My Party Name** box: the FTB party name (`ftbteams:display_name`). It saves about 1 second after typing stops.
    Typing a name with no party creates the party.
  - The help text names the + button and `/invite <player>`.
  - Member list with heads, each member's party, and "No party, shares nothing" where it applies.
  - The leader has a red - button with a tooltip on each member. Other members have **Leave group**.
  - Open invites with a timer and a cancel button.
  - **Group Permissions for <party>'s claims**: the panel above.
- Invites are chat only: "X invites you to the group <party>. [Accept] [Decline] (60 s)", with a tooltip on [Accept].

## Commands

- `/invite <player>`: invite a player. Tab completes online players who are not in your group. Any member can invite.
- `/uninvite <player>`: remove a member. Leader only. Tab completes members. (`/kick` is vanilla's.)
- `/group <message>` and `/g <message>`: message the group.
- `/group` or `/g` with no message: switch group chat mode on or off.
- `/group` is chat only, so a message that starts with a word never runs a subcommand.

## Group frames on the HUD

- Each member's head, name and health bar, with the colors and rules of haven-mob-health-bars (blue `#4A76B8`
  players, magenta `#A8409A` a player who hit a player in the last 30 s, a white trail for recent damage).
- Anchored top-left, growing down. You are not in the list.
- Near members use the game's own health data. Far members and other dimensions use server data, sent on change at
  most 4 times a second. Full state on join.
- Row states: near, far (distance), other dimension (dimmed), offline (grey head), dead.
- Full rows up to 6 members, small rows (no names) from 7.

## Staying in step with FTB

The mod builds on FTB and does not patch it. It writes the same data as the FTB commands and FTB's own team screen
(inventory sidebar, **My Team**: members, Manage Allies, invites, settings).

- If an owner removes a group member as an ally in FTB, the mod accepts it and does not add them back (FTB ally events).
- An ally link that existed before the group is not removed when the member leaves the group. The mod records the links
  it added itself.
- Party changes (join another party, leave, party ends, ownership moves) move the member's links to match.
- Party settings apply to every ally of the party, manual allies too. The panel should say so.
- `ally_mode` in `ftbchunks-world.snbt` must stay `default`.
- Removing the mod leaves the ally links in FTB. Owners can remove them there.
- To check first: whether FTB Teams' public API can add an ally without a command.

## Decided

- Groups, not per-ally permissions. One permissions panel per party, set by its owner.
- Everyone can invite. Only the leader can remove.
- No switch for "members may share our claims": the owner's settings already decide what every group gets.
- Commands as listed above.

## Open

1. One group per player? (The mockups assume yes.)
2. What happens when the leader leaves: the next member becomes leader, or the group ends?
3. How the panel shows a setting that is already `public` (for example a shop).
4. Group size limit (suggested 8).
5. HUD: move the anchor to about 18% of the screen height and measure the chat at run time. Open chat at GUI scale 3
   reaches about 140 of 360 units, so frames at 30% overlap it. Fade the frames while the chat is open. Check whether
   haven-create-ui-ux moves the chat.
6. Group chat on the web map: keep it out of the public web chat; decide whether group members on the web see it.
7. Check that no other mod uses `/g`, `/group`, `/invite` or `/uninvite`. Check what havenlink does with chat.
8. How the mod shares the bar drawing code with haven-mob-health-bars.
