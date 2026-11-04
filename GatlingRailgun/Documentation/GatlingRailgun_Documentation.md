# TBP Gatling Railgun

By **Totally Baked Productions** (TBP)

A spinning gatling railgun for URP with audio-synced firing. Each shot runs a light pulse down both tracks of magnetic rail teeth, flashes the firing bulkhead and fires a tracer round. Long bursts heat the barrel until it glows and shimmers with heat haze.

## Requirements

- Unity 6 (6000.0 or newer)
- Universal Render Pipeline (URP 17+)
- Works with the Input System package, the legacy Input Manager, or both

## Quick start

1. Open `Scenes/GatlingRailgunDemo`.
2. Press **Play** and hold **Space** to fire.
3. To use the gun in your own scene, drag the prefab from `Prefabs/` into it. Fire it with the keyboard, or from code (see [Scripting](#scripting)).

### Turn on Opaque Texture (needed for heat haze)

The heat haze bends the image behind it, which needs URP's Opaque Texture. Select your **URP Asset** (Project Settings > Graphics > Default Render Pipeline) and tick **Rendering > Opaque Texture**. Do this for every URP Asset your Quality levels use. Leave **Opaque Downsampling** at *None* for the sharpest haze.

If Opaque Texture is off, everything else still works and the haze just doesn't show.

### Bloom

The glows are HDR, so they need **Bloom** to look right. The demo scene's Global Volume uses `Scenes/GatlingRailgunDemo_Volume`. In your own scene, add a Volume with Bloom, or copy that profile.

## Folder contents

| Folder | Contents |
|---|---|
| `Scripts/` | `GatlingRailgunController.cs` (the gun) and `PooledProjectile.cs` (added to pooled rounds at runtime) |
| `Models/` | `GatlingRailgun.fbx` (the gun) and `GatlingRailgun_Ammo.fbx` (the projectile) |
| `Materials/` | Gun materials, plus `Tracer` and `HeatDistortion` |
| `Shaders/` | `TBP/TracerAdditive` and `TBP/HeatDistortion` |
| `Textures/` | Color ramp and the rail-teeth emission mask |
| `Audio/` | Wind-up, fire loop and cool-down clips |
| `Scenes/` | Demo scene and its volume profile |
| `Prefabs/` | The ready-to-use gun |

## How it works

- **Spin up:** holding fire spins the barrel up. Firing starts once the barrel reaches **Fire Threshold** of max speed, which with the default settings is exactly when the wind-up sound ends.
- **Shots follow the audio:** shot timing is read from the playing fire-loop clip, so shots never drift from the sound, even at low frame rates.
- **Rail teeth:** every shot steps the teeth through 7 UV states, so the light seems to travel along the rail and push the round out. There are two tracks of magnetic teeth, outer (RailTeeth.001) and inner (RailTeeth.002), and both always show the same state. The states are stored in UV channels 1–7 of the teeth mesh, and the script copies the current one into UV0. This needs **Read/Write** on in the model's import settings, which is already set on the included FBX.
- **Bulkhead flash:** the firing bulkhead flares on each shot and fades out before the next one.
- **Heat:** every shot adds heat, and heat drains over time. Heat drives the barrel glow and the haze, so a short tap stays cold while a long burst glows orange and shimmers.
- **Per-gun materials:** each gun animates its own copies of the materials it changes. Several guns can share a scene without sharing glows, and your `.mat` files are never modified.

## Inspector reference

Hover any field in the Inspector for its tooltip. These are the ones that matter most.

### References
| Field | What it does |
|---|---|
| **Weapon Root** | The gun's top object. Only renderers under it get this gun's material copies. Set this when the component isn't on the top object, which is the case on the prefab. |
| **Barrel** | The part that spins. |
| **Muzzle** | Where projectiles spawn. |
| **Projectile Prefab** | What gets fired. If it has no Rigidbody, one is added. |
| **Rail Teeth Outer / Inner Renderer** | The objects holding the outer and inner teeth. Found automatically from their materials when left empty, and they can be the same object. |

### Input
| Field | What it does |
|---|---|
| **Read Player Input** | Turn this off to fire only from code, for AI or your own player controller. |
| **Fire Action** | *(Input System)* An optional Button action. When empty, **Fire Keyboard Key** is used. |
| **Fire Keyboard Key / Fire Key** | The key held to fire. *Fire Key* is shown when only the legacy Input Manager is active. |

### Emission
| Field | What it does |
|---|---|
| **Inertial Dampener Mat / Glow Color / Intensity** | The inertial dampeners glow while a shot is cycling (UV states 1–7) and go dark between shots. |
| **Rail Teeth Outer Mat** (RailTeeth.001) | The outer track of magnetic teeth, with its own **Emission Intensity** and **Glow Color**. Leave the color white to keep the mask's colors. |
| **Rail Teeth Inner Mat** (RailTeeth.002) | The inner track, with its own intensity and color. It steps through the same UV state as the outer track. Leave it empty if your model has only one track. |
| **Bulkhead Glow Color / Bulkhead Peak Intensity** | Color and brightness of the per-shot flash. |
| **Bulkhead Falloff** | The shape of the fade from the flash back to dark. |

### Rotation and firing
| Field | What it does |
|---|---|
| **Rotation Axis / Max Rotation Speed** | The barrel's spin axis and its top speed in degrees per second. |
| **Match Spin Times To Audio** | Takes the wind-up and cool-down times from the clip lengths. Turn it off to use **Wind Up Time** and **Cool Down Time**. |
| **Shots Per Loop** | How many shots one pass of the fire-loop clip contains. Change this if you swap in your own clip. |
| **Shots Per Second** | Projectiles per second in **Bullet Hose** mode, or when there's no fire-loop clip. Otherwise the rate comes from the audio: **Shots Per Loop** divided by the clip length. |
| **Projectile Speed / Lifetime / Uses Gravity** | How the projectile flies, and how many seconds it lives before it's removed. |
| **Straight Flight Distance** | With gravity on, how many metres a round flies dead straight before bullet drop starts. |
| **Projectile Gravity Scale** | With gravity on, how strong the bullet drop is once it starts (1 = normal gravity). |
| **Align Projectile To Velocity** | Keeps each round's nose pointed along its flight path, so it tips down as it drops. On by default. |
| **Muzzle Fire Axis** | The shot direction, in the Muzzle's local space. Select the gun to see it as a red line. |
| **Projectile Nose Axis / Flip Projectile Nose** | Which way the projectile model points. If rounds fly out tail-first, tick **Flip**. |
| **Projectile Scale** | Size multiplier per axis of the projectile model (1, 1, 1 = as modeled). Scale the nose axis for longer rounds, the other two for fatter ones. |

### Bullet hose
Turn on **Bullet Hose** to fire projectiles at **Shots Per Second** instead of once per audio shot. The sound, rail-teeth sweep, bulkhead flash, heat and haze keep their normal audio-synced rhythm, so the gun still looks and sounds the same while it pours out rounds.

Rates above the frame rate fire several rounds per frame. Each round is placed where it would already be if it had left on time, so the rounds form an even stream instead of a clump at the muzzle. **Max Shots Per Frame** (default 64) caps how many rounds one frame can spawn, so a frame hitch can't cause a burst.

Every round in flight is a separate object with a Rigidbody and a trail. At very high rates, for example 2400 per second with a 2-second lifetime, that's about 4800 rounds in the air at once. Keep **Projectile Pool** on, and lower **Projectile Lifetime** to keep the count down.

### Projectile pool
With **Pool Projectiles** on (the default), rounds are reused instead of being created and destroyed for every shot. That avoids garbage-collection spikes and keeps high fire rates smooth.

- **Pool Prewarm** sets how many rounds are created when the gun starts, so the first burst doesn't stall. The pool grows on its own if it needs more.
- When a round's lifetime ends, it's switched off and goes back into the pool. Its trail is cleared before it fires again.
- Each pooled round gets a `PooledProjectile` component. To end a round early, for example on a hit, call `Release()` on it instead of `Destroy`.
- Changing **Projectile Prefab** while playing is fine. Rounds of the old prefab are destroyed instead of reused.

### Tracer trail
**Tracer Material** should use `Materials/Tracer`. **Tracer Color** times **Tracer Intensity** sets the brightness. The trail starts at the back end of the round, so it stays visible at any Projectile Scale. Trail length is **Tracer Time** times **Projectile Speed**. With **Scale Tracer With Projectile** on, the trail widens to match the round's thickness from **Projectile Scale**.

### Heat haze
| Field | What it does |
|---|---|
| **Heat Haze Material** | Should use `Materials/HeatDistortion`. |
| **Heat Per Shot / Heat Cool Per Second** | How fast the gun heats up and cools down. Heat runs from 0 to 1. |
| **Haze Strength / Haze Size** | How much the haze distorts, and how big the puffs are. Scale **Haze Size** to your gun. |
| **Haze Anchor** | The object the haze rises from (the firing bulkhead in the prefab). |
| **Haze Toward Camera** | Pulls the haze toward the camera so the gun's own surface doesn't hide it. |
| **Show Haze Debug** | Tints the puffs red so you can see where they spawn while tuning. |

### Barrel heat glow
Add materials to **Heat Glow Materials** to make them glow as the gun heats up. Don't add the Inertial Dampener, RailTeeth or FiringBulkhead materials, because those are already animated on every shot. **Heat Glow Colors** runs from first glow (left) to max heat (right). **Heat Glow Start** is the heat level where the glow begins.

### Audio
Leave **Transition Source** and **Loop Source** empty and the gun adds its own 3D AudioSources at startup. Assign your own sources to use a mixer group or custom falloff. The Loop Source must play only the fire loop, because shot timing is read from it.

## Scripting

The controller is `TBP.Weapons.GatlingRailgunController`.

```csharp
using TBP.Weapons;
using UnityEngine;

public class TurretAI : MonoBehaviour
{
    [SerializeField] GatlingRailgunController gun;
    [SerializeField] Transform target;

    void OnEnable()  => gun.ProjectileFired += OnShot;
    void OnDisable() => gun.ProjectileFired -= OnShot;

    void Update()
    {
        // Hold the trigger while the target is in range
        gun.TriggerHeld = target != null &&
                          Vector3.Distance(transform.position, target.position) < 50f;
    }

    void OnShot(GameObject projectile)
    {
        // Attach your own damage / hit logic here. Pooled rounds are launched
        // again later, so only add the component the first time.
        if (!projectile.TryGetComponent(out MyDamageOnHit _))
            projectile.AddComponent<MyDamageOnHit>();
    }
}
```

| Member | Description |
|---|---|
| `bool TriggerHeld { get; set; }` | Holds the trigger from code. This works alongside player input, so either one fires the gun. |
| `State CurrentState { get; }` | One of `Idle`, `WindingUp`, `Firing` or `CoolingDown`. |
| `float Heat { get; }` | From 0 (cold) to 1 (max heat). |
| `event Action<GameObject> ProjectileFired` | Raised every time a round is launched. With pooling, the same object is launched again later. |
| `void ReleaseProjectile(GameObject round)` | Ends a round's flight now. Pooled rounds go back to the pool, and other objects are destroyed. |
| `PooledProjectile.Release()` | The same, called on the round itself. `PooledProjectile.Owner` is the gun that fired it. |

### Hits and damage
The included projectile has a Rigidbody, which is added when missing, but no damage logic and no collider. To detect hits, either give your projectile prefab a collider and your own hit script, or attach one through `ProjectileFired` as shown above. Rounds use *Continuous Dynamic* collision detection so fast rounds don't pass through thin colliders.

When a round hits something, end its flight with the pool instead of destroying it:

```csharp
void OnCollisionEnter(Collision hit)
{
    // ...apply damage...
    if (TryGetComponent(out PooledProjectile round)) round.Release();
    else Destroy(gameObject);
}
```

## Using your own model

The controller isn't tied to this mesh. For your own gun:

1. In the model's import settings, turn on **Read/Write**.
2. Give each rail-teeth mesh 8 UV channels. UV0 is the "off" layout, and UV1–UV7 are the 7 lit states that each shot steps through.
3. Assign **Rail Teeth Outer Mat** (and **Rail Teeth Inner Mat** if you have a second track), **Firing Bulkhead Mat** and **Inertial Dampener Mat** to the materials on your mesh. Each one needs Emission turned on.
4. Set **Barrel**, **Muzzle**, **Weapon Root** and **Rotation Axis**.

## Troubleshooting

| Problem | Fix |
|---|---|
| No heat haze | Turn on **Opaque Texture** in the URP Asset, and check that **Heat Haze Material** uses `TBP/HeatDistortion`. The Console warns you if it doesn't. |
| Glows look flat | Add a Volume with **Bloom**. |
| Rail teeth don't light up | The model needs **Read/Write** on. After re-importing the model, check that both RailTeeth materials use `RailTeethEmissionMask` as their Emission Map. |
| Warning: *"Material … isn't used by any renderer under …"* | Set **Weapon Root** to the gun's top object. |
| Rounds fly backwards or sideways | Tick **Flip Projectile Nose**, or pick the **Projectile Nose Axis** explicitly. |
| Shots don't match the sound | Make sure **Loop Source** plays only the fire loop, and that **Shots Per Loop** matches your clip. |
| Space doesn't fire | Check **Read Player Input**. If you assigned a **Fire Action**, that action is used instead of the key. |

## Support

Questions or bug reports: totallybakedproductions@gmail.com

## Credits

- Skilashu - RailgunFire SFX (https://pixabay.com/sound-effects/film-special-effects-railgunfire-442775/)
- Yodguard - Machine Gun Cooldown (3) SFX (https://pixabay.com/sound-effects/film-special-effects-machine-gun-cooldown-3-402779/)
- Sovetsky_Rastov72 - Minigun Sound Effect SFX (https://pixabay.com/sound-effects/film-special-effects-minigun-sound-effect-244784/)