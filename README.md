# hOn Washer & Dryer Card for Home Assistant

Dashboard cards for washing machines and tumble dryers in the **hOn** app (**Haier, Candy, Hoover**), and for **Samsung** machines through SmartThings ([see below](#samsung-smartthings)). The machine looks like the real thing: the time left on its own display, a progress ring round the door, laundry spinning in the drum, and buttons to pause it or start a programme without walking to the utility room.

**📖 [Step-by-step guide and live demo](https://mymediabuild.uk/home-assistant-washer-dryer-card/)** · **☕ [Buy me a coffee](https://buymeacoffee.com/bobgreggr)**

> **Want more?** Version 3 of this card is free on [my website](https://mymediabuild.uk/home-assistant-washer-dryer-card/): cycle details, every washing function, and the drying options when you press Start.

![Washer and dryer card demo](images/demo.gif)

## What it does

- **The machine's own display:** the time left counts down every second, with the percentage done underneath. Before a cycle it shows the programme's length and "Ready", like the machine.
- **A progress ring round the door:** blue for the washer, orange for the dryer, light blue while the dryer cools down, green when it's finished.
- **A drum that turns while it runs**, with water in the washer and a warm glow in the dryer. The current phase (Washing, Rinse, Spin, Drying, Cooldown) sits on the glass.
- **The details that matter:** programme, temperature and spin speed or dry level, and the finish time. When idle, the total number of cycles.
- **Door and Remote Control icons.**
- **Pause, Resume, Standby and Off buttons**, shown when the machine's Remote Control is on. Standby and Off cancel the cycle, so they ask first.
- **A programmes card:** your favourite programmes as buttons, plus a searchable list of the rest. Tap one, confirm, and it starts.

Both cards are in one file, with the pictures built in.

| Running | Finished / cooling down |
|---|---|
| ![Running](images/running.webp) | ![Finished](images/finished.webp) |

| Start a programme | Standby confirm |
|---|---|
| ![Programmes](images/programmes.webp) | ![Standby](images/standby.webp) |

## Requirements

- Your washer and/or dryer added to the **hOn** app on your phone.
- The hOn custom integration. **[gvigroux/hon](https://github.com/gvigroux/hon)** is the actively maintained fork. The older Andre0512 version no longer logs in. Add it in HACS as a custom repository, restart Home Assistant, then add **hOn** under **Settings → Devices & services**.
- **Remote Control** switched on at the machine if you want to start, pause or stop it from Home Assistant. Without it the card still shows everything; it just can't send commands.

## Install

### HACS (recommended)

1. In HACS, open the three-dot menu and choose **Custom repositories**.
2. Paste this repository's URL (the address of this page) and choose the type **Dashboard**.
3. Search for **hOn Washer & Dryer Card**, download it, and reload your browser when asked.

### Manual

1. Download `hon-appliance-card.js` from the [latest release](../../releases/latest).
2. Copy it to `config/www/hon-appliance-card.js`.
3. Go to **Settings → Dashboards → ⋮ → Resources**, add `/local/hon-appliance-card.js?v=2.1.1` as a **JavaScript module**, then reload your browser.

## Use

This puts each machine above its programme buttons, washer on the left and dryer on the right:

```yaml
type: grid
columns: 2
square: false
cards:
  - type: vertical-stack
    cards:
      - type: custom:hon-appliance-card
        appliance: washer
        max_width: 380px
      - type: custom:hon-programmes-card
        appliance: washer
  - type: vertical-stack
    cards:
      - type: custom:hon-appliance-card
        appliance: dryer
        max_width: 380px
      - type: custom:hon-programmes-card
        appliance: dryer
```

The cards look for entities starting `washing_machine` and `tumble_dryer`, which is what the hOn integration creates if your machines are called "Washing machine" and "Tumble dryer" in the hOn app. If yours are different, find your machine's power sensor under **Settings → Entities** (it ends in `_status`, for example `binary_sensor.hoover_washer_status`) and add the start of its name:

```yaml
type: custom:hon-appliance-card
appliance: washer
entity: hoover_washer
```

### Options

| Option | Card | What it does |
|---|---|---|
| `appliance` | both | `washer` or `dryer` (required) |
| `integration` | both | `hon` (default) or `smartthings` for Samsung machines |
| `entity` | both | Start of your entity IDs. Default `washing_machine` / `tumble_dryer` (SmartThings: `washer` / `dryer`) |
| `name` | both | Card title. Default Washer / Dryer |
| `colour` | both | Accent colour, e.g. `"#4fb3ff"` |
| `max_width` | appliance | How wide the machine can get, e.g. `380px` |
| `style` | appliance | `photo` (default, H-WASH 500 / H-DRY 500 style illustrations) or `drawn` (a simple drawing that suits any front loader) |
| `finish` | appliance | Colour of the built-in machines: `white` (default), `graphite` or `black` |
| `images` | appliance | Folder with your own pictures, e.g. `/local/myimg/` (see [Your own pictures](#your-own-pictures)) |
| `favourites` | programmes | Programme buttons to show, as hOn programme keys such as `iot_wash_cotton` |
| `programmes` | programmes | Full list for the "All programmes" search |
| `names` | both | Your own programme names (see [Your own programme names](#your-own-programme-names)) |

### Machine colour

Got a dark machine? Add `finish: graphite` or `finish: black` to the appliance card. The door ring, knob and glass stay silver and dark, like the real thing.

```yaml
type: custom:hon-appliance-card
appliance: washer
finish: graphite
```

![White, graphite and black finishes](images/finishes.webp)

### Your own pictures

You can swap the built-in machines for your own. Put the files in a folder under `config/www/`, for example `config/www/myimg/`, and point the card at it:

```yaml
type: custom:hon-appliance-card
appliance: washer
images: /local/myimg/
```

The card looks for these names in that folder:

- `washer.svg` (washer card) and `dryer.svg` (dryer card): the machine, front on
- `laundry-drum.webp`: the clothes that turn behind the glass

Include all three, because the card takes every picture from that folder once `images` is set.

To make the progress ring, display and icons line up, draw your machine the same way as the built-in ones: **600 × 850** (portrait), door centre at **298, 352**, glass about **372** across (washer) or **384** (dryer), and the display at the top right, about **374, 10** to **565, 74**. Your own pictures replace `finish`.

### Your own programme names

The hOn integration doesn't pass on the names from the hOn app, only programme keys such as `iot_wash_cotton`, so the card makes its own English names from those. To use the names you know, for example in your own language or the ones you saved in the app, add `names`. Give each programme by its key or by the name the card shows now:

```yaml
type: custom:hon-programmes-card
appliance: washer
names:
  iot_wash_cotton: Baumwolle
  Mixed: Mischwäsche
  "Rapid 30'": "Schnell 30'"
```

Your names are used on the buttons, in the search, in the "Start …?" question, and on the Programme line of the appliance card. Add the same `names` to the appliance card if you want it there too.

### Finding your programme keys

The programmes card starts a cycle with the `hon.start_program` action, using the programme's default temperature and spin. To see which keys your machine accepts, go to **Developer tools → Actions**, pick **hon.start_program**, choose your machine and enter a made-up programme such as `test`. The error lists every programme it accepts. Put the ones you use most in `favourites`.

### Samsung (SmartThings)

Samsung washers and dryers work with Home Assistant's built-in **SmartThings** integration. Add `integration: smartthings` to the appliance card, and `entity` with the start of your entity IDs if your machine isn't called "Washer" or "Dryer" in SmartThings:

```yaml
type: custom:hon-appliance-card
appliance: washer
integration: smartthings
entity: samsung_washer      # if your sensor is sensor.samsung_washer_machine_state
```

The card reads these entities (shown for `entity: washer`):

| Entity | Used for |
|---|---|
| `sensor.washer_machine_state` | Running, paused or stopped (required) |
| `sensor.washer_job_state` | The phase: Washing, Rinse, Spin, Drying, Cooldown, Finished |
| `sensor.washer_completion_time` | The countdown and the finish time |
| `sensor.washer_progress` | The progress ring |
| `binary_sensor.washer_remote_control` | The Remote Control icon, and whether the buttons show |
| `select.washer` | Start, Pause, Resume and Stop |
| `switch.washer` | Off |
| `select.washer_water_temperature`, `select.washer_spin_level` | Temperature and spin (washers) |

What's different from hOn:

- **Start** replaces the programmes card. SmartThings doesn't tell Home Assistant which programmes the machine has, so choose one on the machine, switch on Remote Control (Smart Control), and press **Start** on the card. The programmes card shows a short note instead of buttons.
- **Stop** cancels the cycle (like Standby on hOn). **Off** appears if the machine has a power switch entity.
- No programme name, dry level or cycle count, because the integration doesn't expose them.

## Thanks

Thanks to **gvigroux** for keeping the hOn integration working, and to **Andre0512** for the original.

More free Home Assistant cards and guides at [mymediabuild.uk](https://mymediabuild.uk), including a [step-by-step guide to this card](https://mymediabuild.uk/home-assistant-washer-dryer-card/) with a live demo. If you find it useful, you can [buy me a coffee](https://buymeacoffee.com/bobgreggr) ☕ (no pressure).

## Licence

MIT. Not affiliated with or endorsed by Haier, Candy, Hoover or Samsung.
