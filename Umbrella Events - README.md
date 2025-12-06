# Umbrella Events - Foxhole Event Management Discord Bot

A comprehensive Discord bot for managing Foxhole events with sign-up functionality, Discord scheduled events integration, and real-time updates.

## Features

- ✅ Create events with custom names, descriptions, and times
- ✅ 12-hour time format with timezone selection (no need to convert to UTC!)
- ✅ Automatic Discord scheduled event creation in configured voice channel
- ✅ Simple Yes/No attendance system with reaction-based sign-ups
- ✅ Required attendees tracking with signup count display
- ✅ Real-time embed updates showing sign-ups with Discord mentions
- ✅ Local time display (automatically shows in each user's timezone)
- ✅ Event management (start, end, delete, view) with autocomplete selection
- ✅ Status tracking (Upcoming, In Progress, Ended)
- ✅ Event start notifications with user mentions and embed
- ✅ SQLite database for persistent event storage
- ✅ **Reaction persistence across bot restarts** - signups continue to work after bot restarts
- ✅ **Automatic reaction restoration** - reactions are restored to all active events on startup
- ✅ Modular code structure for easy maintenance
- ✅ Command synchronization without bot restart (`/sync_commands`)
- ✅ Event statistics command (`/event_stats`)
- ✅ Message filtering (ignores messages starting with `!`)
- ✅ **Configurable event output channel** - Events can be posted to a specific channel (admin-configurable)
- ✅ **Discord scheduled event sync** - Users marking "Interested" in Discord events automatically sync to bot signups
- ✅ **Automatic Discord event management** - Starting/ending bot events also starts/ends linked Discord scheduled events
- ✅ **Database export** - Export all event data to CSV files for backup or analysis (`/export_database`)
- ✅ **War tracking** - Track events by war number, automatically assign events to active wars, and view statistics per war

## Setup Instructions

### 1. Prerequisites

- Python 3.8 or higher
- A Discord bot token
- Discord bot with the following permissions:
  - Send Messages
  - Embed Links
  - Add Reactions
  - Manage Events
  - Read Message History

### 2. Installation

1. Clone or download this repository

2. Install required packages:
```bash
pip install -r requirements.txt
```

3. Create a `.env` file in the project root:
```
DISCORD_BOT_TOKEN=your_bot_token_here
```

4. Get your Discord bot token:
   - Go to https://discord.com/developers/applications
   - Create a new application or select an existing one
   - Go to the "Bot" section
   - Click "Reset Token" or copy your existing token
   - Paste it in your `.env` file

5. Invite your bot to your server with these permissions:
   - Send Messages
   - Embed Links
   - Add Reactions
   - Manage Events
   - Read Message History
   - Use Slash Commands

### 3. Running the Bot

```bash
python bot.py
```

You should see a message confirming the bot has logged in and synced commands.

## Commands

All commands are slash commands (type `/` in Discord to see them):

### `/create_event`
Creates a new event with the following **required** parameters:
- `name`: The name of the event (required)
- `description`: Description of the event (required)
- `start_time`: Start time in format `YYYY-MM-DD HH:MM AM/PM` (12-hour format in your local timezone) (required)
- `end_time`: End time in format `YYYY-MM-DD HH:MM AM/PM` (12-hour format in your local timezone) (required)
- `timezone`: Your local timezone for time parsing (required). Options include ET, CT, MT, PT, UTC, GMT, CET, JST, AEST, and more
- `required_attendees`: Number of people required for this event (required)
- `create_discord_event`: Whether to create a Discord scheduled event in the configured voice channel (optional, default: True)

**Example:**
```
/create_event name:"Operation Thunder" description:"Large scale offensive" start_time:"2024-01-15 8:00 PM" end_time:"2024-01-15 11:00 PM" timezone:"America/New_York" required_attendees:20
```

The bot will automatically convert your local time to UTC for storage, and Discord will display it in each user's local timezone. The embed will show signup progress as "Signups (x/y)" where x is the current number of signups and y is the required number of attendees.

**Note:** By default, the bot will create a Discord scheduled event in the configured voice channel. You can set `create_discord_event` to `False` to skip creating a Discord scheduled event. The bot must have "Manage Events" permission and access to the voice channel.

### `/list_events`
Lists all active events in the server (upcoming and in-progress) with their:
- Event name and status (⏳ UPCOMING or 🟢 IN PROGRESS)
- Event ID (format: YYYY-MM-DD_XXX)
- Start time (displayed in your local timezone)
- Number of signups
- Event creator (as Discord mention)

**Example:**
```
/list_events
```

### `/list_past_events`
Lists all past/ended events in the server with their:
- Event name and status (🔴 ENDED)
- Event ID (format: YYYY-MM-DD_XXX)
- Start time (displayed in your local timezone)
- Number of signups
- Event creator (as Discord mention)

**Example:**
```
/list_past_events
```

### `/view_event`
View detailed information about a specific event, including:
- Full event description
- Start and end times (with relative time)
- Current signup status and list of attendees
- Event creator
- Event status

**Parameters:**
- `event`: Select from a dropdown list of all events (upcoming, in progress, and ended)

The command uses autocomplete - simply start typing the event name to filter the list.

**Example:**
```
/view_event event:"Operation Thunder"
```

### `/start_event`
Marks an event as started (changes status to "In Progress").

**Parameters:**
- `event`: Select from a dropdown list of upcoming events (not yet started)

**What happens when an event is started:**
- The event status changes to "In Progress" in the embed
- All users who responded (Yes or No) are mentioned in a notification message
- An embed with full event details is posted in the event channel
- If a Discord scheduled event exists, it is automatically started (status set to active)

**Permissions:** Only the event creator or users with "Manage Events" permission can start events.

The command uses autocomplete - simply start typing the event name to filter the list.

**Example:**
```
/start_event event:"Operation Thunder"
```

### `/end_event`
Marks an event as ended (changes status to "Ended").

**Parameters:**
- `event`: Select from a dropdown list of in-progress events

**What happens when an event is ended:**
- The event status changes to "Ended" in the embed
- The event will appear in `/list_past_events` instead of `/list_events`
- If a Discord scheduled event exists, it is automatically ended (status set to completed)

**Permissions:** Only the event creator or users with "Manage Events" permission can end events.

The command uses autocomplete - simply start typing the event name to filter the list.

**Example:**
```
/end_event event:"Operation Thunder"
```

### `/delete_event`
Permanently deletes an event and its associated Discord scheduled event (if one exists).

**Parameters:**
- `event`: Select from a dropdown list of all events (upcoming, in progress, and ended)

**What happens when an event is deleted:**
- The event embed message is removed from the channel
- The Discord scheduled event is deleted (if it exists)
- All signup data is removed from the database
- The event is permanently removed

**Permissions:** Only the event creator or users with "Manage Events" permission can delete events.

The command uses autocomplete - simply start typing the event name to filter the list.

**Example:**
```
/delete_event event:"Operation Thunder"
```

**⚠️ Warning:** This action cannot be undone!

### `/sync_commands` (Admin Only)
Re-registers all bot commands without restarting the bot. Useful for developers when adding or updating commands.
- Requires Administrator permissions
- Provides terminal confirmation of successful command registration

**Example:**
```
/sync_commands
```

### `/event_stats`
Displays statistics about events and attendance in the server, including:
- Count of upcoming, in-progress, and ended events
- Total required attendees across all events
- Total signups across all events
- Overall signup rate percentage

**Parameters:**
- `war_filter` (optional): Filter statistics by war number (e.g., `100`) or use `lifetime` for all wars combined. If omitted, shows lifetime statistics.

**Examples:**
```
/event_stats
/event_stats war_filter:100
/event_stats war_filter:lifetime
```

### `/settings` (Admin Only)
Configure bot settings for your server. Requires Administrator permissions.

**Actions:**
- **View current settings** - Display current configuration
- **Enable event output to channel** - Enable posting new events to a specific channel
- **Disable event output to channel** - Disable channel output (events post where command is used)
- **Set event output channel** - Configure which channel events should be posted to

**What it does:**
- When enabled, all new events created with `/create_event` will be posted to the configured channel instead of the channel where the command was used
- Useful for centralizing all events in a dedicated channel
- The bot will verify it has permissions in the specified channel before allowing the setting

**Example:**
```
/settings action:"Set event output channel" channel:#events-channel
/settings action:"Enable event output to channel"
/settings action:"View current settings"
```

### `/export_database` (Admin Only)
Exports the database to a single CSV file for backup or analysis. Requires Administrator permissions.

**What it exports:**
- **Single CSV file** containing all data for your server:
  - All events (event_id, name, description, start_time, end_time, creator_id, etc.)
  - All signups (user_id, position, emoji) linked to their events
  - Guild settings (event output channel configuration)
  - One row per event-signup combination (events with no signups appear once with empty signup fields)

**Features:**
- Exports only data for your server (guild-specific)
- Single comprehensive CSV file with all data combined
- File is timestamped for easy organization
- **Sent via DM** - The CSV file is delivered to your DMs (not posted in the channel)
- Automatically cleans up temporary files after sending

**Note:** If you have DMs disabled, the bot cannot send you the file. Please enable DMs from server members to use this command.

**Example:**
```
/export_database
```

The bot will send you a single CSV file via DM containing all your server's event data.

## War Tracking

The bot can track events by war number, allowing you to organize and analyze events from specific wars.

### `/war_setup` (Admin Only)
Sets up a new war with a war number. The war is created but not yet active.

**Parameters:**
- `war_number`: The war number (e.g., 100, 101, 102)

**What it does:**
- Creates a new war entry in the database
- War is not active until you use `/war_start`
- Prevents duplicate war numbers for the same server

**Example:**
```
/war_setup war_number:100
```

### `/war_start` (Admin Only)
Marks a war as active. All events created while a war is active will be automatically assigned to that war.

**Parameters:**
- `war_number`: The war number to start (for verification)

**What it does:**
- Activates the specified war
- Deactivates any previously active war
- Events created after this will be assigned to the active war
- Requires verification with the war number

**Example:**
```
/war_start war_number:100
```

### `/war_end` (Admin Only)
Ends the currently active war. New events will no longer be assigned to this war.

**Parameters:**
- `war_number`: The war number to end (for verification)

**What it does:**
- Deactivates the specified war
- Events created after this will not be assigned to any war (until a new war is started)
- Requires verification with the war number to prevent accidental endings

**Example:**
```
/war_end war_number:100
```

**Workflow:**
1. Use `/war_setup` to create a new war (e.g., War 100)
2. Use `/war_start` to activate the war when it begins
3. All events created while the war is active will be automatically assigned to that war
4. Use `/war_end` to end the war when it concludes
5. Use `/event_stats war_filter:100` to view statistics for that specific war

**Note:** Events created before a war is started or after a war is ended will not be assigned to any war (war_id will be NULL). You can still view lifetime statistics using `/event_stats war_filter:lifetime`.

## Attendance System

Users can indicate their attendance for events by reacting to the event embed message:

- ✅ **Yes** - User will be attending the event
- ❌ **No** - User will not be attending the event

The embed automatically updates to show who has responded, organized into two sections:
- **✅ Attending:** List of users who selected Yes
- **❌ Not Attending:** List of users who selected No

Signed-up users are displayed as Discord mentions (clickable user tags) that automatically update if users change their display names. Users can change their response by reacting with the other emoji, or remove their response by removing their reaction.

**Switching between Yes and No:**
- Users can switch between ✅ (Yes) and ❌ (No) by clicking the other emoji
- The bot automatically handles the switch and updates the signup accordingly
- Users will not be dropped from the signup list when switching reactions

The embed also displays signup progress as **"Signups (x/y)"** where:
- `x` = Current number of people who responded (Yes or No)
- `y` = Required number of attendees for the event

**Persistence Across Restarts:**
- Reactions and signups persist across bot restarts
- The bot automatically restores reactions to all active event messages on startup
- Users can continue signing up for events even after the bot has been restarted

**Discord Scheduled Event Sync:**
- If an event has a linked Discord scheduled event, users who mark themselves as "Interested" in the Discord event will automatically be added as "Yes" in the bot's signup system
- When users remove their "Interested" status from a Discord scheduled event, they are removed from bot signups (if they were marked as "Yes")
- This provides one-way sync from Discord scheduled events to bot signups
- **Note:** Bots cannot programmatically add users to Discord scheduled events - users must manually mark themselves as interested in Discord

## Time Display

Event times are displayed using Discord's timestamp feature, which automatically shows times in each user's local timezone. The embed shows:
- Full date and time in the user's local timezone
- Relative time (e.g., "in 2 hours", "3 days ago")

## Permissions

- **Create Events**: Anyone can create events
- **Start/End/Delete Events**: Only the event creator or users with "Manage Events" permission can modify events

## Project Structure

The bot uses a modular file structure:

```
├── bot.py                      # Main bot file and event handlers
├── commands/
│   ├── admin.py               # Administrative commands
│   ├── create_event.py         # Create event command
│   ├── delete_event.py         # Delete event command
│   ├── end_event.py            # End event command
│   ├── event_stats.py          # Event statistics command
│   ├── export_database.py      # Export database to CSV command (admin)
│   ├── list_events.py          # List active events command
│   ├── list_past_events.py      # List past events command
│   ├── settings.py             # Bot settings command (admin)
│   ├── start_event.py          # Start event command
│   ├── utils.py                # Utility functions for commands
│   ├── view_event.py           # View event details command
│   ├── war_setup.py            # War setup command (admin)
│   ├── war_start.py            # War start command (admin)
│   └── war_end.py              # War end command (admin)
├── helpers/
│   ├── config.py              # Configuration and constants
│   ├── database.py             # Database operations
│   ├── discord_event_sync.py   # Discord scheduled event sync handlers
│   ├── embeds.py               # Embed creation functions
│   ├── models.py               # Data models
│   ├── reactions.py            # Reaction handling logic (raw reaction events)
│   └── startup.py              # Startup functions (reaction restoration)
├── requirements.txt            # Python dependencies
└── README.md                   # This file
```

## Discord Scheduled Events

When creating an event (Discord scheduled events are created by default), the bot will:
- Create a Discord scheduled event in the configured voice channel (channel ID: 1435806546771841085)
- Set the privacy level to "Guild Only" (visible only to server members)
- Link the scheduled event to your bot event for easy tracking

**Requirements for Discord Scheduled Events:**
- Bot must have "Manage Events" permission
- Bot must have access to the configured voice channel
- Event start time must be in the future
- Event name must be 100 characters or less
- Event description must be 1000 characters or less

**Automatic Event Management:**
- When you use `/start_event`, the bot automatically starts the linked Discord scheduled event (sets status to active)
- When you use `/end_event`, the bot automatically ends the linked Discord scheduled event (sets status to completed)
- When you use `/delete_event`, the bot automatically deletes the linked Discord scheduled event

**Discord Event Sync:**
- Users who mark themselves as "Interested" in a Discord scheduled event are automatically added as "Yes" in the bot's signup system
- Users who remove their "Interested" status are removed from bot signups (if they were "Yes")
- This provides seamless integration between Discord's native event system and the bot's signup tracking
- **Note:** Bots cannot add users to Discord scheduled events programmatically - this is a Discord API limitation

## Notes

- **Event IDs**: Events use a date-based ID format `YYYY-MM-DD_XXX` where:
  - `YYYY-MM-DD` is the date the event was created (in UTC)
  - `XXX` is a zero-padded 3-digit count (000, 001, 002, etc.) of events created that day
  - Example: `2024-01-15_000` is the first event created on January 15, 2024
  - The count resets each day and is per-guild

- **Event Storage**: Events are stored in a SQLite database (`events.db`). The database is automatically created when the bot starts.

- **Time Handling**: Times are stored in UTC internally, but you can enter times in your local timezone when creating events. Discord automatically displays times in each user's local timezone.

- **Intents**: The bot requires the following intents:
  - `message_content` - to read reactions properly
  - `reactions` - to handle reaction events
  - `members` - to display user information

- **Database File**: The database file (`events.db`) will be created in the same directory as the bot script.

- **Bot Status**: The bot's status displays "Staying Dry and Creating Events" when online.

- **Event Selection**: All event selection commands use autocomplete for easier navigation - no need to remember event IDs!

- **Creator Display**:
  - Event embeds show the creator's display name in the footer
  - `/list_events` and `/list_past_events` show creators as Discord mentions (clickable)

- **Discord Scheduled Events**: The voice channel for Discord scheduled events is hardcoded in the bot configuration (channel ID: `1435806546771841085`).

- **Event Output Channel**: Admins can configure a specific channel where all new events will be posted using `/settings`. When enabled, events are posted to the configured channel instead of where the command was used.

- **Message Filtering**: The bot ignores messages that start with `!` to avoid conflicts with other bots using prefix commands.

- **Reaction Persistence**: Reactions and signups persist across bot restarts. The bot automatically restores reactions to all active event messages when it starts up.

- **Discord Event Sync**: The bot automatically syncs users who mark themselves as "Interested" in Discord scheduled events to the bot's signup system. This provides one-way sync from Discord events to bot signups.

## Troubleshooting

**Bot doesn't respond to commands:**
- Make sure the bot has been invited with the correct permissions
- Check that slash commands have been synced (you should see a message when the bot starts)
- Try restarting the bot

**Reactions don't work:**
- Ensure the bot has "Add Reactions" and "Read Message History" permissions
- Check that the `message_content` and `reactions` intents are enabled in your Discord application settings
- After a bot restart, reactions should be automatically restored - wait a few seconds after the bot starts
- If reactions still don't work after restart, try removing and re-adding your reaction

**Times show incorrectly:**
- Make sure you've selected the correct timezone when creating events
- Times are stored in UTC but displayed in each user's local timezone automatically
- The autocomplete dropdown shows times in UTC for consistency

## License

This project is open source and available for personal use.

