# OmniCore AI — Godot Plugin

Turn any NPC into an AI-driven character with voice, personality, and adaptive
combat decisions. One plugin handles dialogue, text-to-speech, speech-to-text,
voice-to-voice conversation, and game-state AI vision.

---

## Install

1. Copy the `addons/omnicore_ai` folder into your project's `res://addons/` folder.
2. In Godot: **Project → Project Settings → Plugins** → enable **OmniCore AI**.
3. `OmniCoreAI` is now available everywhere in your project as a global singleton.

## Quick Start

```gdscript
func _ready():
    OmniCoreAI.ai_dialogue_received.connect(_on_dialogue)
    OmniCoreAI.ai_action_received.connect(_on_actions)
    OmniCoreAI.ai_tts_received.connect(_on_tts)
    OmniCoreAI.ai_error.connect(_on_error)

func _on_dialogue(text: String, source: String):
    print("NPC says: ", text)

func _on_actions(actions: Dictionary):
    print("AI chose: ", actions)

func _on_tts(audio_b64: String):
    OmniCoreAI.play_audio_from_base64(audio_b64, $AudioStreamPlayer3D)

func _on_error(cmd: String, error: String):
    print("Error in ", cmd, ": ", error)
```

Now call any function from anywhere:

```gdscript
# Text dialogue
OmniCoreAI.talk_ai("The player just attacked me!", "loyal warrior", "dark fantasy RPG")

# Voice-to-voice (player mic → AI voice response)
OmniCoreAI.listen_and_talk(mic_audio_b64, "loyal warrior", "dark fantasy RPG", "dialogue", "Respond as this character.", true, "Charon")

# AI vision — NPC decides what to do
OmniCoreAI.game_state_parse(game_state_dict, ["attack", "defend", "retreat", "idle"])
```

---

## Functions

### Dialogue & Voice

| Function | Purpose |
|---|---|
| `talk_ai(text, personality, game_context, style="dialogue", instructions="", voice_name="")` | Get a line of dialogue (text only). |
| `talk_and_speak(text, personality, game_context, style="dialogue")` | Dialogue + spoken audio (base64 WAV). |
| `speak_ai(text, voice_name="")` | Text-to-speech only. |
| `listen_ai(audio_b64)` | Speech-to-text only. |
| `listen_and_talk(audio_b64, personality, game_context, style="dialogue", instructions="", wants_voice=false, voice_name="")` | **Voice-to-voice**: send player mic audio, get AI dialogue + spoken audio back. |
| `play_audio_from_base64(audio_b64, audio_player: AudioStreamPlayer3D)` | Plays returned audio locally (no network call). Auto-strips WAV header. |

### AI Vision & Decisions

| Function | Purpose |
|---|---|
| `run_ai(input_vector: Array)` | Send a raw input vector, get back an action dictionary. |
| `game_state_parse(game_state, output_labels=[])` | Send a game-state snapshot, get back chosen action(s). |
| `game_state_summary(game_state)` | Send a game-state snapshot, get back a text summary. |

---

## Signals

| Signal | Fires when |
|---|---|
| `ai_response_received(cmd, data)` | Any request finishes successfully. |
| `ai_action_received(actions)` | `run_ai` / `game_state_parse` returns actions. |
| `ai_dialogue_received(text, source)` | Dialogue text comes back. |
| `ai_tts_received(audio_b64)` | Audio comes back. |
| `ai_error(cmd, error)` | A request fails. |

---

## Voice Setup

To use voice input/output, you need three things:

### 1. Enable audio input in Godot
**Project → Project Settings → Audio → Driver → Enable Input = ON**
Or add this to `project.godot`:
```ini
[audio]
driver/enable_input=true
```

If you get a `WASAPI: GetBufferSize error` on Windows, try switching the driver:
```ini
[audio]
driver="XAudio2"
driver/enable_input=true
```

### 2. Add an AudioStreamPlayer3D to your NPC
```gdscript
func _ready():
    _audio = AudioStreamPlayer3D.new()
    _audio.max_distance = 20.0
    add_child(_audio)

func _on_tts(audio_b64: String):
    OmniCoreAI.play_audio_from_base64(audio_b64, _audio)
```

### 3. Use a voice name (optional)
Pass any of these to functions that accept `voice_name`:
- **Charon**, **Fenrir**, **Orus**, **Algenib**, **Puck** (male/deep)
- **Aoede**, **Kore**, **Leda**, **Freya** (female/light)

---

## AI Vision — Game State

The plugin sends game state to the server for AI decision-making. The plugin
**does not gather this data automatically** — you must build the dictionary
from your game's nodes and pass it to `game_state_parse()`.

### Minimal example

```gdscript
func _request_ai_decision():
	var game_state := {
		"player_health":    health,
		"player_pos_x":     global_position.x / 100.0,
		"player_pos_y":     global_position.y / 100.0,
		"player_pos_z":     global_position.z / 100.0,
		"player_vel_x":     velocity.x / 10.0,
		"player_vel_y":     velocity.y / 10.0,
		"player_vel_z":     velocity.z / 10.0,
		"player_weapon":    "melee",
		"zone_danger":      0.8 if _enemy_nearby() else 0.1,
		"time_remaining":   1.0,
		"visibility_range": 1.0,
		"score":            0.0,
		"collision_flags":  0,
		"entities":         _gather_entities(),
		"event_labels":     _current_events(),
	}
	var output_labels := [
		"attack_light", "attack_heavy", "block", "dodge",
		"use_ability", "retreat", "take_cover", "idle",
		"use_item", "call_ally", "taunt", "observe",
		"flank", "rush", "counter", "wait",
	]
	OmniCoreAI.game_state_parse(game_state, output_labels)

func _gather_entities() -> Array:
	var entities := []
	for enemy in get_tree().get_nodes_in_group("enemy"):
		if not is_instance_valid(enemy) or enemy.is_dead:
			continue
		entities.append({
			"type":         "enemy",
			"pos_x":        enemy.global_position.x / 100.0,
			"pos_y":        enemy.global_position.y / 100.0,
			"pos_z":        enemy.global_position.z / 100.0,
			"health":       enemy.health,
			"weapon":       "melee",
			"visible":      true,
			"attacking":    enemy.is_attacking,
			"team_id":      1,
			"label":        enemy.npc_name,
			"danger_score": _calculate_danger(enemy),
		})
	return entities

func _on_omni_action(actions: Dictionary):
	# Pick the action with the highest probability
	var top_action := ""
	var top_value := 0.0
	for action in actions:
		if actions[action] > top_value:
			top_value = actions[action]
			top_action = action
	# Map to game behavior
	match top_action:
		"attack_light", "attack_heavy", "rush", "flank":
			_attack_target()
		"retreat", "take_cover":
			_retreat()
		"observe", "wait", "idle":
			_idle()
```

### How it works

1. Your game code gathers state (health, positions, enemies, events)
2. `OmniCoreAI.game_state_parse()` sends it to the server
3. Server's `game_vision.py` normalizes values and builds a 64-dim input vector
4. Server runs the `StudentGameBrain` neural network (61M params)
5. Server returns action probabilities
6. `ai_action_received` fires with the action dictionary
7. Your game maps the highest-probability action to actual behavior

---

## Voice-to-Voice Example

Full example: player speaks into mic → AI transcribes → AI generates in-character
voice response → plays through NPC's AudioStreamPlayer3D.

```gdscript
# In your NPC script

var _audio: AudioStreamPlayer3D
var _mic_input: Node   # your mic capture node

func _ready():
	_audio = AudioStreamPlayer3D.new()
	_audio.max_distance = 20.0
	add_child(_audio)
	
	OmniCoreAI.ai_dialogue_received.connect(_on_dialogue)
	OmniCoreAI.ai_tts_received.connect(_on_tts)
	OmniCoreAI.ai_error.connect(_on_error)

func talk_to_npc_via_mic(mic_audio_b64: String):
	# Send player's voice, get AI's voice back
	OmniCoreAI.listen_and_talk(
		mic_audio_b64,
		"You are a friendly loyal warrior companion.",
		"dark fantasy RPG",
		"dialogue",
		"Respond as this character in 1-2 sentences.",
		true,           # wants_voice = get audio back
		"Charon"        # voice name
	)

func _on_dialogue(text: String, _source: String):
	# Show subtitles
	print("NPC says: ", text)
	_show_subtitle(text)

func _on_tts(audio_b64: String):
	# Play the AI's spoken response
    OmniCoreAI.play_audio_from_base64(audio_b64, _audio)

func _on_error(cmd: String, error: String):
    print("OmniCore error in ", cmd, ": ", error)
```

---

## Audio Playback Note

`play_audio_from_base64()` expects a **full WAV file** (44-byte header + PCM data)
returned by the server. The function automatically:

1. Detects the WAV header (`RIFF...WAVE`)
2. Parses the sample rate and channel count from the header
3. Locates the `data` chunk and extracts raw PCM
4. Creates an `AudioStreamWAV` with the correct format

**Do NOT pre-strip the header** — pass the full WAV base64 string directly.
The server returns `audio_b64` in exactly the format this function expects.

---

## Mic Capture (optional helper)

The plugin does not include mic capture — you handle that in your game. Here's
a minimal Godot 4 example:

```gdscript
extends Node

signal recording_finished(audio_b64: String)

var _record_effect: AudioEffectRecord
var _record_bus_idx: int

func _ready():
	# Create a "Record" bus if it doesn't exist
    var bus_name := "Record"
    if AudioServer.get_bus_index(bus_name) == -1:
        AudioServer.add_bus()
        AudioServer.set_bus_name(AudioServer.bus_count - 1, bus_name)
    _record_bus_idx = AudioServer.get_bus_index(bus_name)
    AudioServer.set_bus_send(_record_bus_idx, "Master")
    
    # Add record effect
    var effect := AudioEffectRecord.new()
    AudioServer.add_bus_effect(_record_bus_idx, effect)
    _record_effect = AudioServer.get_bus_effect(_record_bus_idx, 0)
    
    # Connect a mic stream to the Record bus
    var capture := AudioStreamMicrophone.new()
    var player := AudioStreamPlayer.new()
    player.stream = capture
    player.bus = bus_name
    add_child(player)
    player.play()

func start_recording(duration_sec: float = 3.0):
    _record_effect.set_recording_active(false)
    await get_tree().create_timer(0.05).timeout
    _record_effect.set_recording_active(true)
    await get_tree().create_timer(duration_sec).timeout
    _record_effect.set_recording_active(false)
    await get_tree().create_timer(0.05).timeout
    
    var recording: AudioStreamWAV = _record_effect.get_recording()
    if recording == null:
        recording_finished.emit("")
        return
    
    var wav_bytes := _audiostream_to_wav_bytes(recording)
    recording_finished.emit(Marshalls.raw_to_base64(wav_bytes))

func _audiostream_to_wav_bytes(stream: AudioStreamWAV) -> PackedByteArray:
    # Convert AudioStreamWAV to 16kHz mono PCM WAV
    # (Your existing conversion code here — see mic_input.gd for reference)
    pass
```

---

## Troubleshooting

### Mic records silence (rms=0.0000)
- **Enable audio input**: Project Settings → Audio → Driver → Enable Input = ON
- **Check OS mic permissions**: Windows Settings → Privacy → Microphone → allow Godot
- **WASAPI errors on Windows**: Switch driver to `XAudio2` in `project.godot`
- **Restart audio service**: Run `net stop AudioEndpointBuilder && net start AudioEndpointBuilder` as admin

### AI responds with random dialogue
- This means the mic captured silence and Whisper returned empty text
- The AI responds to `"[Silent background noise]"` placeholder
- Fix the mic (see above), not the server

### No audio plays back
- Ensure `play_audio_from_base64()` uses the version that strips the WAV header
- Check that `AudioStreamPlayer3D` is positioned and not muted
- Verify `mix_rate` matches (Gemini outputs 24kHz — the function auto-detects this)

### HTTP errors (401, 429, 500)
- **401**: API key missing or invalid
- **429**: Rate limit exceeded (Standard tier = 30/min, Premium = 120/min)
- **500**: Server error — check the server logs at your HF Space

---

## Version

Plugin v4.0 — compatible with OmniCore Game AI server v4.0+
Requires Godot 4.0+ (tested on 4.5)
