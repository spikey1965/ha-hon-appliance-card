# hOn & GE Washer & Dryer Card for Home Assistant

Dashboard cards for washing machines and tumble dryers in the **hOn** app (**Haier, Candy, Hoover**) and the **GE Appliances SmartHQ** app (**GE, GE Profile, Café**). The machine looks like the real thing: the time left on its own display, a progress ring round the door, laundry spinning in the drum, and buttons to pause it or start a programme without walking to the utility room.

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

For **hOn** machines (Haier, Candy, Hoover):

- Your washer and/or dryer added to the **hOn** app on your phone.
- The hOn custom integration. **[gvigroux/hon](https://github.com/gvigroux/hon)** is the actively maintained fork. The older Andre0512 version no longer logs in. Add it in HACS as a custom repository, restart Home Assistant, then add **hOn** under **Settings → Devices & services**.
- **Remote Control** switched on at the machine if you want to start, pause or stop it from Home Assistant. Without it the card still shows everything; it just can't send commands.

For **GE Appliances** machines (GE, GE Profile, Café):

- Your washer and/or dryer added to the **SmartHQ** app (formerly GE Appliances / Comfort) on your phone.
- The **GE Home (SmartHQ)** custom integration, **[simbaja/ha_gehome](https://github.com/simbaja/ha_gehome)**, version 2025.12 or newer. Install it from HACS, restart Home Assistant, then add **GE Home (SmartHQ)** under **Settings → Devices & services**.
- **Remote Enable** pressed on the machine if you want to start it from Home Assistant. See [GE Appliances](#ge-appliances-ge-home--smarthq) for what the card can and can't do with GE.

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

### GE Appliances (GE Home / SmartHQ)

The appliance card works with GE washers and dryers through the [GE Home (SmartHQ)](https://github.com/simbaja/ha_gehome) integration. You don't need any extra options: the card finds your GE washer or dryer by itself. Put the two machines side by side like this:

```yaml
type: grid
columns: 2
square: false
cards:
  - type: custom:hon-appliance-card
    appliance: washer
    max_width: 380px
  - type: custom:hon-appliance-card
    appliance: dryer
    max_width: 380px
```

GE entity IDs start with the machine's serial number, for example `sensor.zl123456g_laundry_machine_state`, or with its MAC address, for example `sensor.aa_bb_cc_dd_ee_ff_laundry_machine_state`, depending on the **Device identifier** chosen in GE Home's **Configure** options. If you have more than one GE washer or dryer, or the card picks the wrong one, find the machine-state sensor under **Settings → Entities** (search for `laundry_machine_state`) and set `entity` to the part before `_laundry_machine_state`:

```yaml
type: custom:hon-appliance-card
appliance: dryer
entity: gd987654
```

What's different with GE:

- **Start, but no Pause or Stop.** The GE Home integration can't pause, resume, stop or switch off a machine, or choose a programme. Choose the cycle on the machine and press **Remote Enable**. A **Start** button then appears on the card, and it asks before starting.
- **No programmes card.** For a GE machine, `hon-programmes-card` shows a short note instead of programme buttons, so leave it off your dashboard.
- **What it shows:** the cycle, the phase (Filling, Washing, Rinse, Spin, Drying, Cooldown, ...), the time left and the finish time. It also shows wash temperature and spin for the washer, or heat and dryness for the dryer, plus the door, the washer's door lock and Remote Enable. A delayed start shows as Scheduled, counting down to when the laundry will be done. GE has no cycle counter, so there's no "Cycles" line.
- **The progress ring** uses Home Assistant's history to find when the cycle started, because GE only reports the time left. That keeps the ring right after a pause. History is on unless you've excluded the machine-state sensor from the recorder.
- **Pictures:** the built-in machines are Haier designs. `style: drawn` is a plainer front loader, or use [your own pictures](#your-own-pictures).

### Options

| Option | Card | What it does |
|---|---|---|
| `appliance` | both | `washer` or `dryer` (required) |
| `entity` | both | Start of your entity IDs. Default `washing_machine` / `tumble_dryer`. For GE, the serial number or MAC address, found by itself if left out |
| `integration` | both | `hon` or `ge`. Normally left out, as the card works it out from your entities |
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

For a GE machine, add `names` to the appliance card and give each cycle as GE Home reports it, such as `Normal Or Mixed Load`, or as the card shows it, such as `Normal / Mixed Load`.

### Finding your programme keys (hOn)

The programmes card starts a cycle with the `hon.start_program` action, using the programme's default temperature and spin. To see which keys your machine accepts, go to **Developer tools → Actions**, pick **hon.start_program**, choose your machine and enter a made-up programme such as `test`. The error lists every programme it accepts. Put the ones you use most in `favourites`.

## Thanks

Thanks to **gvigroux** for keeping the hOn integration working, to **Andre0512** for the original, and to **simbaja** for the GE Home integration.

More free Home Assistant cards and guides at [mymediabuild.uk](https://mymediabuild.uk), including a [step-by-step guide to this card](https://mymediabuild.uk/home-assistant-washer-dryer-card/) with a live demo. If you find it useful, you can [buy me a coffee](https://buymeacoffee.com/bobgreggr) ☕ (no pressure).

## Licence

MIT. Not affiliated with or endorsed by Haier, Candy, Hoover or GE Appliances.
