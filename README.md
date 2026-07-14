Here is your updated, fully combined multi-engine developer manual. It seamlessly merges your Godot documentation with the official Unity integration architecture, ensuring both workflows are organized side-by-side using professional markdown tooling.

---

# OmniCore AI — Multi-Engine Developer Guide (Godot & Unity)

Turn any NPC into an AI-driven character with voice, personality, and adaptive combat decisions. This plugin handles dialogue, text-to-speech (TTS), speech-to-text (STT), voice-to-voice conversation, and game-state AI vision natively across **Godot 4** and **Unity**.

---

## 💾 Installation & Setup

### Godot 4

1. Copy the `addons/omnicore_ai` folder into your project's `res://addons/` directory.
2. In the editor menu: **Project → Project Settings → Plugins** → toggle **Enable** next to OmniCore AI.
3. `OmniCoreAI` is now globally accessible everywhere in your project as an autoload singleton.

### Unity

1. Copy the `Assets/Plugins/OmniCoreAI` folder into your Unity project.


2. Open the scene you want to use OmniCore in.


3. In the top menu bar: **Tools → OmniCore AI → Add to Scene**.


4. Select the generated `OmniCoreAI` GameObject in the hierarchy and populate your `apiUrl` and `apiKey` fields within the Inspector window.


* *Alternative:* Open the centralized setup panel via **Tools → OmniCore AI → Setup**.





---

## ⚡ Quick Start

### Godot 4 (GDScript)

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

### Unity (C#)

```csharp
using OmniCore;
using UnityEngine;
using System.Collections.Generic;

public class NPC : MonoBehaviour
{
    private AudioSource _audio;

    void Start()
    {
        _audio = GetComponent<AudioSource>();
        OmniCoreAI.Instance.OnDialogueReceived += OnDialogue;
        OmniCoreAI.Instance.OnActionReceived   += OnAction;
        OmniCoreAI.Instance.OnTTSReceived      += OnTTS;
        OmniCoreAI.Instance.OnError            += OnError;
    }

    void OnDialogue(string text, string source) => Debug.Log("NPC says: " + text);
    
    void OnAction(Dictionary<string, object> actions)
    {
        string best = OmniCoreAI.Instance.GetBestAction(actions);
        Debug.Log("AI chose: " + best);
    }

    void OnTTS(string audioB64) => OmniCoreAI.Instance.PlayAudioFromBase64(audioB64, _audio);
    void OnError(string cmd, string error) => Debug.LogError("OmniCore error in " + cmd + ": " + error);
}

```

#### Inline Runtime Executions (Global Invocation)

```gdscript
# Godot 4
OmniCoreAI.talk_ai("The player just attacked me!", "loyal warrior", "dark fantasy RPG")
OmniCoreAI.listen_and_talk(mic_audio_b64, "loyal warrior", "dark fantasy RPG", "dialogue", "Respond as this character.", true, "Charon")
OmniCoreAI.game_state_parse(game_state_dict, ["attack", "defend", "retreat", "idle"])

```

```csharp
// Unity
OmniCoreAI.Instance.TalkAI("The player just attacked me!", "loyal warrior", "dark fantasy RPG");
OmniCoreAI.Instance.ListenAndTalk(micAudioB64, "loyal warrior", "dark fantasy RPG", "dialogue", "", true, "Charon");
OmniCoreAI.Instance.GameStateParse(gameStateDict, new string[] { "attack", "defend", "retreat", "idle" });

```

---

## 🛠️ API & Component Reference

### Core Global Functions

#### Dialogue & Voice

| Godot Methods (GDScript) | Unity Methods (C#)

 | Purpose |
| --- | --- | --- |
| `talk_ai(text, personality, context, style, instructions, voice_name)` | `TalkAI(text, personality, gameContext, style, instructions)` | Text dialogue calculations only.

 |
| `talk_and_speak(text, personality, game_context, style)` | `TalkAndSpeak(text, personality, gameContext, style)` | Dialogue string + spoken response (base64 WAV).

 |
| `speak_ai(text, voice_name)` | `SpeakAI(text, voiceName)` | Text-to-speech runtime invocation.

 |
| `listen_ai(audio_b64)` | `ListenAI(audioB64)` | Speech-to-text decoding array translation.

 |
| `listen_and_talk(audio_b64, personality, ...)` | `ListenAndTalk(audioB64, personality, ...)` | **Voice-to-Voice Loop:** Input voice bytes, return parsed voice data.

 |
| `play_audio_from_base64(audio_b64, player)` | `PlayAudioFromBase64(audioB64, audioSource)` | Ingestion engine component. Parses WAV structure dynamically.

 |

#### AI Vision & Decisions

| Godot Methods (GDScript) | Unity Methods (C#)

 | Purpose |
| --- | --- | --- |
| `run_ai(input_vector: Array)` | `RunAI(inputVector)` | Send raw 64-float matrix vectors directly to core.

 |
| `game_state_parse(game_state, labels)` | `GameStateParse(gameState, outputLabels)` | Transmit environmental variables → dynamic weight arrays.

 |
| `game_state_summary(game_state)` | `GameStateSummary(gameState)` | Context metrics processing → natural language text summary.

 |

#### Unity Utilities & Platform Helpers



* `GetBestAction(Dictionary<string, object> actions)`: Analyzes weight probability tables and automatically selects the highest-scoring action string.


* `AudioClipToBase64(AudioClip clip)`: Helper matrix utility that safely reformats standard raw Unity `AudioClip` tracks into proper base64 formatted WAV buffers for transmission.



---

## 🎛️ Component Architectures

### Unity Target Behaviours



* **OmniCoreAI (MonoBehaviour):** Primary system controller manager. Added once per scene, structural runtime properties handle continuous `DontDestroyOnLoad` scene switches automatically.


* `apiUrl`: Remote host endpoint configuration.


* `apiKey`: Remote host security authentication token.


* `requestTimeout`: Network thread lifecycle duration control (Default: 120 seconds).




* **OmniCoreMic (MonoBehaviour):** High-level component utility for simple recording pipelines.


```csharp
OmniCoreMic mic = GetComponent<OmniCoreMic>();
mic.OnRecordingFinished += (audioB64) => {
    OmniCoreAI.Instance.ListenAndTalk(audioB64, "guard", "medieval RPG", "dialogue", "", true, "Fenrir");
};
mic.RecordForDuration(3f);

```



---

## 🧠 AI Vision — Context Engineering Architecture

Developers must construct localized context state arrays programmatically and interface them with the server pipeline for inference weight distributions.

### Complete Cross-Engine Context Implementations

#### Godot 4 (Context Builder Routine)

```gdscript
func _request_ai_decision():
    var game_state := {
        "player_health":    health,
        "player_pos_x":     global_position.x / 100.0,
        "player_pos_y":     global_position.y / 100.0,
        "player_pos_z":     global_position.z / 100.0,
        "player_vel_x":     velocity.x / 10.0,
        "player_weapon":    "melee",
        "zone_danger":      0.8 if _enemy_nearby() else 0.1,
        "entities":         _gather_entities(),
    }
    var output_labels := ["attack_light", "block", "retreat", "idle"]
    OmniCoreAI.game_state_parse(game_state, output_labels)

func _on_omni_action(actions: Dictionary):
    var top_action := ""
    var top_value := 0.0
    for action in actions:
        if actions[action] > top_value:
            top_value = actions[action]
            top_action = action
            
    match top_action:
        "attack_light", "attack_heavy": _attack_target()
        "retreat": _retreat()
        _: _idle()

```

#### Unity 3D (Builder Pattern Pipeline)



```csharp
void RequestDecision()
{
    GameObject player = GameObject.FindWithTag("Player");
    if (player == null) return;

    // Fluent matrix structure generation via Builder component
    var state = OmniCoreGameState.Create()
        .SetPlayerHealth(health)
        .SetPlayerPosition(player.transform.position)
        .SetPlayerVelocity(_rb.linearVelocity)
        .SetPlayerWeapon("melee")
        .SetZoneDanger(0.7f)
        .AddEntity("enemy", transform.position, health, "melee", true, isAttacking)
        .Build();

    string[] labels = { "attack_light", "block", "retreat", "idle" };
    OmniCoreAI.Instance.GameStateParse(state, labels);
}

void OnAction(Dictionary<string, object> actions)
{
    string top = OmniCoreAI.Instance.GetBestAction(actions);
    switch (top)
    {
        case "attack_light": AttackPlayer(); break;
        case "retreat":      Retreat();      break;
        default:             Idle();         break;
    }
}

```

---

## 🎤 Comprehensive Voice-to-Voice Loop Configuration

Ensure underlying hardware configuration tracks match standard execution layers:

### 1. Hardware Permissions Setup

* **Godot 4:** Navigate to **Project Settings → Audio → Driver** and set **Enable Input** to **ON**.
* *Windows Override:* Switch driver parameters to `driver="XAudio2"` if tracking `WASAPI: GetBufferSize` loop faults.


* **Unity Platform Compilation Profiles:** Ensure microphone permissions blocks are initialized globally:


* **Android Devices:** Append `<uses-permission android:name="android.permission.RECORD_AUDIO" />` to the application manifest configuration layout.


* **iOS Devices:** Assign appropriate descriptions to `NSMicrophoneUsageDescription` strings inside target `Info.plist` packages.





### 2. Stream Playback Management

The localized runtime functions (`play_audio_from_base64()` / `PlayAudioFromBase64()`) parse the complete base64 data wrapper, strip metadata descriptors, detect internal byte limits (`RIFF...WAVE`), and immediately structure native asset objects directly into memory runtime components without physical disc serialization overhead.

> ⚠️ **Important:** Do not manually modify or strip byte signatures from arrays prior to feeding them into processing modules. Systems look for the initial 44-byte initialization signature.

---

## 🔍 System Logs & Troubleshooting

| Observable Fault Mode | Underlying Root Cause | Resolution Strategy Protocol |
| --- | --- | --- |
| **System captures zero voice profiles** | Core Hardware Routing | Check underlying engine device options. Ensure OS-level microphone context security rules explicitly allow engine software data transmission.

 |
| **Random dialogue responses generated** | Silent Data Capture | Whisper engines interpret completely empty silence intervals as noise configurations, filling state records with default context loops. Optimize local hardware gateway cut-off parameters. |
| **Console Errors: HTTP 401** | Verification Failure | API keys missing or formatted improperly. Re-verify component attributes.

 |
| **Console Errors: HTTP 429** | Performance Overrun | Rate limitation rules reached (Standard execution tier permits 30 calls/min; Premium scaling structures allow up to 120 calls/min).

 |
| **Console Errors: HTTP 500** | Server Crash | Backend node execution exception. Review the standard debugging logs on your hosted Hugging Face instance.

 |

---

## 📋 System Metrics & Compatibility

* **Voice Target Matrix Profile Selection Blocks:** `Charon`, `Fenrir`, `Orus`, `Algenib`, `Puck` (Deep/Male variations) or `Aoede`, `Kore`, `Leda`, `Freya` (Light/Female variations).


* **Godot Engine Targets:** v4.0 through modern stable frameworks.
* **Unity Engine Targets:** v2022.3 LTS through modern iteration builds.


* **OmniCore Project Iteration Framework:** Compatible with Core Server Deployment Matrix v4.0.
