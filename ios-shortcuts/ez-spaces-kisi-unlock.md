# EZ Spaces – Auto Unlock (Kisi) iOS Shortcut

Unlocks the EZ Spaces office door with Kisi when you arrive at the entrance.

## How it works

The phone has no single setting that fires an automation within 5 ft. iOS location
automations use geofences, and the smallest radius is about 100 m / 330 ft. Indoor GPS
error is often 15–50 ft. So the shortcut works in two stages:

| Stage | What runs | Precision |
|---|---|---|
| 1. Wake up | Personal Automation, **Arrive** at a geofence around the door (smallest radius) | ~100 m |
| 2. Close in | The shortcut polls **Get Current Location** (Precise Location on) and measures the distance to the door coordinates you capture | as good as GPS allows (best effort) |
| 3. Unlock | Kisi checks the 5 ft rule itself, using its Bluetooth reader or geofence, when the unlock fires | Kisi enforces it |

Shortcuts **cannot tap UI elements inside other apps**. "Tap the EZ Spaces card" is done
with one of these, in order of preference:

Kisi does **not** offer an Unlock action in Shortcuts (checked on 2026-10-01). That
leaves two options:

- **Default: Open Kisi + notification.** The shortcut opens Kisi as you reach the door and
  shows "At the door – tap EZ Spaces". You make one tap.
- **Upgrade: Kisi API** (fully hands-free). Use *Get Contents of URL* →
  `POST https://api.kisi.io/locks/{LOCK_ID}/unlock`. This needs an API key and the lock ID
  from your Kisi admin. Your org's 5 ft / geofence rules may still apply to API unlocks,
  so confirm with the admin.

---

## Captured door location

| Field | Value |
|---|---|
| Latitude | `33.3081908` |
| Longitude | `-111.7570556` |
| Captured | 2026-10-01, standing at the EZ Spaces entrance |
| Paste-ready | `33.3081908, -111.7570556` |

You can paste these straight into Shortcut 2 instead of reading `ez_door.txt`. To do
that, replace the first five actions with one **Location** action set to
`33.3081908, -111.7570556`, then **Set Variable** `DoorLoc`. Use the same coordinates
for the Arrive automation's pin.

---

## Phone settings (do these first)

1. **Settings → Privacy & Security → Location Services → Shortcuts** → *While Using the App*
   (or *Always*) and **Precise Location ON**.
2. Same screen → **Kisi** → *Always* and **Precise Location ON**.
3. **Settings → Privacy & Security → Location Services → System Services** → *Significant
   Locations* ON. This makes Arrive triggers more reliable.
4. Kisi app: sign in and confirm the **EZ Spaces** card is visible. Turn Bluetooth ON.

---

## Shortcut 1: "Capture EZ Door Location" (run once, standing at the door)

1. **Get Current Location**
2. **Get Details of Locations** → *Latitude*  → **Set Variable** `DoorLat`
3. **Get Details of Locations** (from Current Location) → *Longitude* → **Set Variable** `DoorLon`
4. **Text**: `EZ_DOOR|[DoorLat]|[DoorLon]`
5. **Save File** → path `Shortcuts/ez_door.txt`, *Ask Where to Save* OFF, *Overwrite* ON
6. **Show Result**: `Door saved: [DoorLat], [DoorLon]`

> Tip: run it 3 times, standing still against the door each time, and keep the last
> reading. GPS settles after about 10–20 s.

---

## Shortcut 2: "EZ Spaces Unlock"

```
Get File  Shortcuts/ez_door.txt            (Error If Not Found: ON)
Split Text  by Custom "|"
Get Item from List  index 2  → Set Variable DoorLat
Get Item from List  index 3  → Set Variable DoorLon
Text  "[DoorLat], [DoorLon]"  → Get Addresses/Location from text → Set Variable DoorLoc
     (alternatively: add a Location action and paste the coordinates in directly)

Set Variable  Unlocked = No
Repeat 40 times                              (~2 min budget)
    Get Current Location
    Get Distance  from Current Location to DoorLoc   (Direct, in Feet)
    If Distance  is less than  25            (tune after testing — see below)
        ── Upgrade ───  Get Contents of URL
                          POST https://api.kisi.io/locks/LOCK_ID/unlock
                          Header  Authorization: KISI-LOGIN <API_KEY>
                          Header  Content-Type: application/json
        ── Default ───  Open App  Kisi
                        Show Notification  "At the door – tap EZ Spaces"
        Set Variable  Unlocked = Yes
        Stop and Output                        (exits the loop)
    End If
    Wait 3 seconds
End Repeat
If Unlocked is No
    Open App  Kisi
    Show Notification  "Couldn’t confirm you’re at the door – Kisi is open"
End If
```

Why the threshold is 25 ft and not 5 ft: GPS can't reliably resolve 5 ft indoors, so a
5 ft check would often never fire. The shortcut sends the unlock once you're in the
area, and Kisi applies its own 5 ft rule. After testing, lower the threshold to the
smallest value that still fires every time.

---

## Automation: run on arrival

Shortcuts → **Automation** → **+** → **Arrive**

- Location: **EZ Spaces door** (search the address, then drag the pin onto the entrance and
  shrink the radius to the minimum)
- Time: **Any Time** (or a work-hours range)
- **Run Immediately** (not "Run After Confirmation"), and **Notify When Run** OFF
- Action: **Run Shortcut → EZ Spaces Unlock**

---

## Test checklist

- [ ] Capture shortcut saves coordinates while standing at the door
- [ ] Run **EZ Spaces Unlock** by hand at the door → door unlocks
- [ ] Run it by hand about 50 ft away → it waits, then unlocks as you walk up
- [ ] Leave the area (more than 200 m away), walk back → automation fires on its own
- [ ] Adjust the distance threshold (25 ft) and the loop length based on results
