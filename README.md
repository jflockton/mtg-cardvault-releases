# MTG CardVault

**A webcam card scanner and local-first inventory for Magic: The Gathering singles**, built for a game shop counter. Hold a card up to the camera, it locks, beeps and writes itself to stock. Everything else — the collection browser, the deck builder, wish lists, the buy list and the price watch — works off that same database, on your own machine.

This repository holds the **installers and release notes**. The application is proprietary; its source is not published.

## Download

The latest installers are on the [Releases page](../../releases/latest):

| Platform | File |
| --- | --- |
| Windows 10 / 11 | `MTG.CardVault.Setup.<version>.exe` |
| macOS (Apple Silicon) | `MTG.CardVault-<version>-arm64.dmg` |

Both are self-contained. Nothing else needs installing.

### First run

1. Run the installer. On Windows, SmartScreen may warn about an unknown publisher — the installer is unsigned; choose *More info → Run anyway*.
2. On first launch the app asks where to keep its card database (about 80 MB, downloaded from Scryfall). Keep it on a local drive; it is rebuildable.
3. Your inventory is a separate, small file. **Settings → Storage** can put it in a Dropbox or OneDrive folder so it is backed up and shared between machines. Backups are also taken automatically on every launch.
4. Plug in a webcam, open **Scan cards in**, and hold a card so its bottom-left corner sits in the guide. Beep = counted; no beep = not counted.

## What it does

- **Scan in / scan out.** Corner scanning reads the set code and collector number; name mode handles old cards with no corner text. Auto-lock, audio confirmation, foil flip and undo from the keyboard. Sell mode runs the same loop in reverse.
- **Collection.** Every card in stock with its finish, rarity, Cardmarket value and when it was scanned, with date filters, search, and CSV / deck-list exports.
- **Inventory browser.** The collection as card images — searchable by name, set, type, tribe, rarity, colour, mana value, price and treatment — or any of the ~107,000 printings Scryfall knows, with a Cardmarket link per printing and which boosters it comes in.
- **Deck builder.** Build, import (text, file, Archidekt / Moxfield / EDHREC link) and analyse Commander decks against live stock: what you own, what is in another deck, curve, odds, legality, a sideboard, swap-outs, version history and an optional AI-written guide.
- **Wish lists and the buy list.** Named lists of cards to get; one **To buy** list fed from them, with ordered / arrived tracking that puts arrivals straight into stock.
- **Price watch.** Every wanted printing against its own 7-day, 3-month and 6-month averages, so you buy when a card is cheap *for that card*.
- **Set collector.** Track a set — owned, missing, and what finishing it would cost.
- **Cardmarket.** A card's Cardmarket page opens inside the app, filtered to UK sellers and English copies; what its price guide says is read off and kept on the card.
- **Looks.** Seven art styles, light or dark, and a high-contrast mode with colour-blind-safe status colours.

## Requirements

- Windows 10/11 or macOS 13+ on Apple Silicon.
- A webcam (1080p recommended) for scanning; every other feature works without one.
- Internet for the first-run card database, price refreshes and card images. Scanning and the inventory itself work offline.

## Licence

MTG CardVault is proprietary software licensed, not sold, under the [End User Licence Agreement](EULA.md) shown by the installer. Card data and images come from [Scryfall](https://scryfall.com); prices are indicative, not a valuation. MTG CardVault is unofficial Fan Content and is not affiliated with Wizards of the Coast.

## Release notes

Every version is described in [CHANGELOG.md](CHANGELOG.md).
