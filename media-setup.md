# Aura Cuts — media setup

The site runs with **zero images present**. Every image slot shows a labelled placeholder
with its own filename printed on it, so the layout and copy can be signed off before the
photography happens. Drop a file into `media/` with the exact name below and it appears.
No code changes, ever.

```
aura-cuts/
├── index.html          ← the whole site
├── media-setup.md      ← this file
└── media/
    ├── hero-shop.webp
    ├── haircut.webp
    ├── cut-beard.webp
    ├── shave.webp
    ├── massage.webp
    ├── facial.webp
    ├── regular.webp
    ├── sufyan.webp
    ├── phillamoon.webp
    ├── usman.webp
    └── og-image.jpg
```

---

## 1. Shot list

| Filename | Size (px) | What it is | Notes |
|---|---|---|---|
| `hero-shop.webp` | 2400 × 1350 | The shop floor, lights on, mirrors visible, chairs empty | This is the single most important image on the page. Shoot after closing so nobody's face is in it. Keep the left third quiet — the headline glow sits over it on desktop. |
| `haircut.webp` | 900 × 720 | Mid-cut, clippers on the side of a head | Hands and hair only — no full faces unless you have permission. |
| `cut-beard.webp` | 900 × 720 | Finished cut + shaped beard, three-quarter angle | The "after" shot. Needs a real client and a signed OK to use it. |
| `shave.webp` | 900 × 720 | Hot towel on the face, or lather being brushed on | Steam is the whole point. Shoot it while the towel is genuinely hot. |
| `massage.webp` | 900 × 720 | Hands on scalp, overhead or side | Keep it calm, no motion blur. |
| `facial.webp` | 900 × 720 | Mask applied, or the product row on the counter | The counter shot is safer if no client will agree to be photographed. |
| `regular.webp` | 900 × 720 | Wide shot of one chair mid-service | Represents "The Regular" package. |
| `sufyan.webp` | 800 × 800 | Mr. Sufyan, square, waist up | Get all three barber portraits in the same 10-minute window, same spot, same light. Consistency matters more than perfection. |
| `phillamoon.webp` | 800 × 800 | Phillamoon, same framing | |
| `usman.webp` | 800 × 800 | Usman, same framing | |
| `og-image.jpg` | 1200 × 630 | Whatever the strongest shop shot is, with the AURA CUTS wordmark placed on it | This is what shows when the link is pasted into WhatsApp. JPG, not WebP — some previewers still choke on WebP. |

### Shooting it on a phone

A recent phone is fine. Three rules that matter more than the camera:

1. **Turn the shop lights on and shoot at night.** Daylight through the door fights the interior lights and everything goes green on one side, orange on the other.
2. **Wipe the mirrors first.** Every mark shows.
3. **Shoot wider than you need.** The cards crop to 5:4 and the portraits to 1:1 — leave room.

---

## 2. Compressing

Install once:

```bash
# macOS
brew install webp imagemagick
# Ubuntu / WSL
sudo apt install webp imagemagick
```

Then, from the folder holding the originals:

```bash
# Hero — resize to 2400 wide, quality 78
magick hero-raw.jpg -resize 2400x -strip hero-tmp.png
cwebp -q 78 hero-tmp.png -o media/hero-shop.webp && rm hero-tmp.png

# Service cards — 900 × 720, centre crop
magick service-raw.jpg -resize 900x720^ -gravity center -extent 900x720 -strip tmp.png
cwebp -q 80 tmp.png -o media/haircut.webp && rm tmp.png

# Barber portraits — 800 × 800 square
magick barber-raw.jpg -resize 800x800^ -gravity north -extent 800x800 -strip tmp.png
cwebp -q 80 tmp.png -o media/sufyan.webp && rm tmp.png

# Share image — JPG, 1200 × 630
magick hero-raw.jpg -resize 1200x630^ -gravity center -extent 1200x630 \
  -strip -quality 82 media/og-image.jpg
```

Batch all the service shots at once:

```bash
for f in cut1 cut2 shave massage facial regular; do
  magick "raw/$f.jpg" -resize 900x720^ -gravity center -extent 900x720 -strip "/tmp/$f.png"
  cwebp -q 80 "/tmp/$f.png" -o "media/$f.webp"
done
```

### Target weights

| File | Target | Hard ceiling |
|---|---|---|
| `hero-shop.webp` | 110–160 KB | 220 KB |
| each service card | 45–70 KB | 90 KB |
| each portrait | 40–60 KB | 80 KB |
| `og-image.jpg` | 80–110 KB | 150 KB |

**Whole-page budget: under 1.2 MB, Largest Contentful Paint under 2.5s on 4G.**
Check it before launch:

```bash
du -ch media/* index.html | tail -1
```

If it's over, drop hero quality to `-q 70` before you drop any images — the hero is the
only one big enough to matter.

---

## 3. Before it goes live

- [ ] **Confirm every price with Mr. Sufyan in writing.** The ones in the file are realistic
      placeholders for a Malir shop, not real. A wrong price on a website is the fastest way
      to lose a 4.9 rating.
- [ ] **Confirm the barbers' names, spellings and roles.** "Phillamoon" and "Usman" are taken
      from how reviewers spelled them, which is not necessarily how they spell them.
- [ ] Confirm the guard-number chart matches the clippers actually in use — most Wahl and
      Andis sets match these lengths, some cheaper sets don't.
- [ ] Confirm payment methods (the schema block currently claims Cash, JazzCash, Easypaisa).
- [ ] Add the Instagram / Facebook / TikTok links in the footer, or delete those `<li>` lines.
- [ ] **Delete the orange draft banner** — the very first `<div class="draft">` in the body,
      plus its `.draft` CSS rule.
- [ ] Register the domain and put it in the Google listing's website field. That field is
      currently empty, which is the whole reason this site exists.
- [ ] Deploy: drag the folder onto **Cloudflare Pages** or **Netlify** — free tier, no build
      step, HTTPS included.

---

## 4. What the site does without a backend

- **Booking** is a WhatsApp deep link on `wa.me/923160010226`. Tapped services and the chosen
  guard number are written into the message body, so the shop receives a readable list with an
  approximate total. No server, no database, no payment gateway, nothing to maintain or renew.
- **Open / closed** in the hero is computed live against Pakistan Standard Time (UTC+5) from
  the `hours` values in the data block. Change the hours there and the badge follows.
- **Structured data** is a `HairSalon` JSON-LD block with the address, coordinates, hours and
  the 4.9 / 54 rating. This is what makes Google show the rich result and the hours panel.
- **Prices** live in one array. A non-developer can change any price in a text editor and
  re-upload the single file.
