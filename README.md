<p align="center">
  <img src="https://img.shields.io/badge/Fuelink-Lavalink%20Client-FF6B35?style=for-the-badge&logo=discord&logoColor=white" alt="Fuelink">
</p>

<h1 align="center">Fuelink</h1>

<p align="center">
  <strong>A powerful, feature-complete Lavalink client for the Fuelex ecosystem</strong>
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/fuelink"><img src="https://img.shields.io/npm/v/fuelink?color=FF6B35&label=npm&style=flat-square" alt="npm version"></a>
  <a href="https://www.npmjs.com/package/fuelink"><img src="https://img.shields.io/npm/dt/fuelink?color=5865F2&style=flat-square" alt="npm downloads"></a>
  <a href="https://github.com/Fuelex-Labs/fuelink"><img src="https://img.shields.io/github/stars/Fuelex-Labs/fuelink?color=FFD43B&style=flat-square" alt="GitHub stars"></a>
  <a href="https://nodejs.org"><img src="https://img.shields.io/badge/node-%3E%3D18.0.0-339933?style=flat-square&logo=node.js" alt="Node.js"></a>
  <a href="https://github.com/Fuelex-Labs/fuelink/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="License"></a>
</p>

<p align="center">
  <a href="#features">Features</a> •
  <a href="#installation">Installation</a> •
  <a href="#quick-start">Quick Start</a> •
  <a href="#configuration">Configuration</a> •
  <a href="#documentation">Documentation</a>
</p>

---

## About

**Fuelink** is the unified Lavalink client built specifically for the **Fuelex open-source system**. It serves as the single audio control layer, centralizing all audio logic, state management, and queue handling while providing drop-in compatibility with existing Lavalink client patterns.

### Why Fuelink?

| Aspect | Description |
|--------|-------------|
| **Unified** | Single client to handle all audio operations across your Discord bot |
| **Complete** | Production-ready with all features you need out of the box |
| **Extensible** | Hot-loadable plugin system with support for custom sources |
| **Reliable** | Automatic failover, session resume, and state persistence |
| **Modern** | Built for Lavalink v4 with full JavaScript/CommonJS support |

---

## Features

### Lavalink Connectivity

- **Multi-Node Support** — Connect to multiple Lavalink nodes simultaneously
- **Load Balancing** — Intelligent penalty-based node selection (CPU, players, frames)
- **Health Monitoring** — Continuous node health checks with stats tracking
- **Auto-Reconnect** — Exponential backoff reconnection on disconnects
- **Session Resume** — Resume Lavalink sessions after reconnection
- **Node Migration** — Automatic player migration on node failure
- **Region Awareness** — Prefer nodes closest to Discord voice servers

### Player System

- **Complete Playback Control** — Play, pause, resume, stop, seek, volume
- **Gapless Playback** — Preloading for seamless track transitions
- **Crossfade Support** — Configurable crossfade between tracks
- **Inactivity Handling** — Auto-disconnect when idle, paused, or alone
- **Position Tracking** — Real-time estimated position calculation
- **State Persistence** — Save and restore player state across restarts

### Queue Management

- **Priority Queue** — Add tracks that play next before the main queue
- **Loop Modes** — Off, track repeat, queue loop
- **Advanced Operations** — Shuffle, move, swap, remove, jump, clear
- **History Tracking** — Go back to previous tracks with full history
- **Requester Tracking** — Track who requested each song
- **Auto-Play** — Automatic recommendations when queue ends

### Audio Filters & Effects

| Filter | Description |
|--------|-------------|
| **Equalizer** | 15-band EQ with presets (flat, bass, pop, rock, classical) |
| **Bass Boost** | Multiple levels (off, low, medium, high, extreme) |
| **Nightcore** | Speed up with pitch shift |
| **Vaporwave** | Slow down with pitch down |
| **8D Audio** | Rotation effect for immersive sound |
| **Timescale** | Custom speed, pitch, and rate control |
| **Karaoke** | Vocal removal effect |
| **Tremolo/Vibrato** | Oscillation effects |
| **Distortion** | Audio distortion |
| **Low Pass** | High frequency filtering |

All filters can be **stacked** and applied **in real-time** without restarting the track.

### Plugin System

- **Hot-Loadable** — Add and remove plugins at runtime
- **LavaSrc Compatible** — Built-in support for Spotify, Apple Music, Deezer, Yandex
- **Custom Sources** — Create your own source plugins
- **Middleware Hooks** — Intercept track loads and playback events
- **Directory Loading** — Auto-discover plugins from a folder

### Persistence & Recovery

- **Multiple Backends** — Memory, file, Redis, MongoDB
- **Auto-Save** — Periodic automatic state saving
- **Crash Recovery** — Restore players after bot restarts
- **TTL Management** — Automatic cleanup of expired data

### Discord Integration

- **Multi-Library Support** — Works with Discord.js and Eris
- **Auto-Detection** — Automatically detects your Discord library
- **Raw Events** — Direct voice state/server update handling
- **DisTube Compatible** — Use DisTube plugins for search and resolve

---

## Installation

```bash
# npm
npm install fuelink

# yarn
yarn add fuelink

# pnpm
pnpm add fuelink
```

### Requirements

| Requirement | Version |
|-------------|---------|
| Node.js | >= 18.0.0 |
| Lavalink | >= 4.0.0 |
| Discord.js | >= 14.x (optional) |
| Eris | >= 0.17.x (optional) |

---

## Quick Start

### Basic Setup

```javascript
const { Client, GatewayIntentBits } = require('discord.js');
const { Fuelink } = require('fuelink');

// Create Discord client
const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildVoiceStates
  ]
});

// Create Fuelink client
const fuelink = new Fuelink({
  nodes: [
    {
      name: 'main',
      host: 'localhost',
      port: 2333,
      password: 'youshallnotpass'
    }
  ]
});

// Initialize when Discord is ready
client.once('ready', async () => {
  await fuelink.init(client);
  console.log('Fuelink is ready!');
});

// Handle events
fuelink.on('trackStart', ({ player, track }) => {
  console.log(`Now playing: ${track.title}`);
});

fuelink.on('queueEnd', ({ player }) => {
  console.log('Queue finished!');
});

client.login('YOUR_TOKEN');
```

### Playing Music

```javascript
async function play(guildId, voiceChannelId, query) {
  // Get or create player
  let player = fuelink.getPlayer(guildId);
  if (!player) {
    player = await fuelink.createPlayer({
      guildId,
      voiceChannel: voiceChannelId
    });
  }

  // Search for tracks
  const tracks = await fuelink.search(query, {
    requester: { id: 'user-id', username: 'User' }
  });

  if (tracks.length === 0) {
    return console.log('No results found');
  }

  // Add to queue and play
  player.queue.add(tracks[0]);
  
  if (!player.playing) {
    await player.play();
  }
}
```

---

## Configuration

### Full Configuration Example

```javascript
const fuelink = new Fuelink({
  // Lavalink nodes
  nodes: [
    {
      name: 'main',
      host: 'lavalink.example.com',
      port: 2333,
      password: 'secure_password',
      secure: true,              // Use HTTPS/WSS
      retryAmount: 5,            // Reconnection attempts
      retryDelay: 5000,          // Delay between retries (ms)
      resumeKey: 'fuelink-main', // Session resume key
      resumeTimeout: 60,         // Resume timeout (seconds)
      priority: 1,               // Lower = preferred
      regions: ['us-east']       // Preferred regions
    },
    {
      name: 'backup',
      host: 'lavalink2.example.com',
      port: 2333,
      password: 'secure_password',
      priority: 2
    }
  ],

  // Auto-connect to nodes on init
  autoConnect: true,

  // Auto-restore players on startup
  autoResume: true,

  // Persistence configuration
  persistence: {
    enabled: true,
    backend: 'redis',  // 'memory' | 'file' | 'redis' | 'mongodb'
    autoSave: true,
    saveInterval: 30000,
    options: {
      // Backend-specific options
      client: redisClient  // For Redis
    }
  },

  // Plugin configuration
  plugins: {
    directory: './plugins',  // Auto-load plugins from directory
    lavaSrc: {
      spotify: true,
      applemusic: true,
      deezer: true
    }
  },

  // Logger configuration
  logger: {
    level: 1  // 0=DEBUG, 1=INFO, 2=WARN, 3=ERROR
  }
});
```

---

## Documentation

### Player Methods

| Method | Description |
|--------|-------------|
| `player.play(track?, options?)` | Play a track or next in queue |
| `player.pause()` | Pause playback |
| `player.resume()` | Resume playback |
| `player.stop(clearQueue?)` | Stop playback |
| `player.skip()` | Skip to next track |
| `player.back()` | Go to previous track |
| `player.seek(position)` | Seek to position (ms) |
| `player.setVolume(0-100)` | Set volume |
| `player.destroy()` | Destroy the player |

### Queue Methods

| Method | Description |
|--------|-------------|
| `queue.add(track, options?)` | Add track(s) to queue |
| `queue.addPriority(track)` | Add to priority queue (plays next) |
| `queue.remove(index)` | Remove track by index |
| `queue.shuffle()` | Shuffle the queue |
| `queue.move(from, to)` | Move track position |
| `queue.swap(i1, i2)` | Swap two tracks |
| `queue.jump(index)` | Skip to specific position |
| `queue.clear()` | Clear the queue |
| `queue.setLoop(mode)` | Set loop mode ('off', 'track', 'queue') |

### Filter Methods

| Method | Description |
|--------|-------------|
| `filters.setEqualizer(bands)` | Set EQ bands |
| `filters.setEQPreset(preset)` | Apply EQ preset |
| `filters.setBassBoost(level)` | Set bass boost level |
| `filters.setNightcore(enabled)` | Toggle nightcore |
| `filters.setVaporwave(enabled)` | Toggle vaporwave |
| `filters.set8D(enabled)` | Toggle 8D audio |
| `filters.setSpeed(speed)` | Set playback speed |
| `filters.setPitch(pitch)` | Set pitch |
| `filters.reset()` | Reset all filters |

### Events

```javascript
// Player events
fuelink.on('playerCreate', ({ player }) => {});
fuelink.on('playerDestroy', ({ player }) => {});
fuelink.on('playerMove', ({ player, oldChannel, newChannel }) => {});
fuelink.on('playerDisconnect', ({ player }) => {});
fuelink.on('playerUpdate', ({ player, state }) => {});

// Track events
fuelink.on('trackStart', ({ player, track }) => {});
fuelink.on('trackEnd', ({ player, track, reason }) => {});
fuelink.on('trackStuck', ({ player, track, threshold }) => {});
fuelink.on('trackError', ({ player, track, error }) => {});

// Queue events
fuelink.on('queueAdd', ({ player, tracks }) => {});
fuelink.on('queueRemove', ({ player, track }) => {});
fuelink.on('queueShuffle', ({ player }) => {});
fuelink.on('queueEnd', ({ player }) => {});

// Node events
fuelink.on('nodeConnect', ({ node }) => {});
fuelink.on('nodeDisconnect', ({ node, reason }) => {});
fuelink.on('nodeError', ({ node, error }) => {});
fuelink.on('nodeReady', ({ node, resumed }) => {});
```

---

## Creating Plugins

### Custom Source Plugin

```javascript
const { BaseSource } = require('fuelink');

class MyMusicSource extends BaseSource {
  name = 'mymusic';

  canResolve(query) {
    return query.includes('mymusic.com');
  }

  async resolve(query, options) {
    // Fetch and return track data
    const data = await fetchFromMyAPI(query);
    return [{
      encoded: data.encodedTrack,
      info: {
        title: data.title,
        author: data.artist,
        duration: data.duration,
        uri: data.url
      }
    }];
  }

  async search(query, options) {
    // Search and return results
    const results = await searchMyAPI(query);
    return results.map(track => ({
      encoded: track.encodedTrack,
      info: { /* ... */ }
    }));
  }
}

// Register the plugin
fuelink.use(new MyMusicSource());
```

---

## Music Platforms Support

| Platform | Support | Notes |
|----------|---------|-------|
| YouTube | Built-in | Native Lavalink support |
| YouTube Music | Built-in | Native Lavalink support |
| SoundCloud | Built-in | Native Lavalink support |
| Spotify | LavaSrc | Requires LavaSrc plugin on Lavalink |
| Apple Music | LavaSrc | Requires LavaSrc plugin on Lavalink |
| Deezer | LavaSrc | Requires LavaSrc plugin on Lavalink |
| Yandex Music | LavaSrc | Requires LavaSrc plugin on Lavalink |
| Bandcamp | Custom | Via custom plugin |
| HTTP Streams | Built-in | Direct URL playback |

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<p align="center">
  Made with care by <a href="https://github.com/Fuelex-Labs">Fuelex Labs</a>
</p>

<p align="center">
  <sub>Part of the Fuelex Open-Source Ecosystem</sub>
</p>
