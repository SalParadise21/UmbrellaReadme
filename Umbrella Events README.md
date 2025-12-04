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
- `create_discord_event`: Whether to create a Discord scheduled event in the configured voice channel (optional, default: False)

**Example:**
```
/create_event name:"Operation Thunder" description:"Large scale offensive" start_time:"2024-01-15 8:00 PM" end_time:"2024-01-15 11:00 PM" timezone:"America/New_York" required_attendees:20
```

The bot will automatically convert your local time to UTC for storage, and Discord will display it in each user's local timezone. The embed will show signup progress as "Signups (x/y)" where x is the current number of signups and y is the required number of attendees.

**Note:** If `create_discord_event` is set to `True`, the bot will create a Discord scheduled event in the configured voice channel. The bot must have "Manage Events" permission and access to the voice channel.

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
- If a Discord scheduled event exists, its status is verified

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

**Example:**
```
/event_stats
```

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
│   ├── list_events.py          # List active events command
│   ├── list_past_events.py      # List past events command
│   ├── start_event.py          # Start event command
│   ├── utils.py                # Utility functions for commands
│   └── view_event.py           # View event details command
├── helpers/
│   ├── config.py              # Configuration and constants
│   ├── database.py             # Database operations
│   ├── embeds.py               # Embed creation functions
│   ├── models.py               # Data models
│   ├── reactions.py            # Reaction handling logic (raw reaction events)
│   └── startup.py              # Startup functions (reaction restoration)
├── requirements.txt            # Python dependencies
└── README.md                   # This file
```

## Discord Scheduled Events

When creating an event with `create_discord_event: True`, the bot will:
- Create a Discord scheduled event in the configured voice channel (channel ID: 1435806546771841085)
- Set the privacy level to "Guild Only" (visible only to server members)
- Link the scheduled event to your bot event for easy tracking

**Requirements for Discord Scheduled Events:**
- Bot must have "Manage Events" permission
- Bot must have access to the configured voice channel
- Event start time must be in the future
- Event name must be 100 characters or less
- Event description must be 1000 characters or less

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

- **Message Filtering**: The bot ignores messages that start with `!` to avoid conflicts with other bots using prefix commands.

- **Reaction Persistence**: Reactions and signups persist across bot restarts. The bot automatically restores reactions to all active event messages when it starts up.

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

