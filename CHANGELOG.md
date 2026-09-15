# Changelog

All notable changes to MTG CardVault. Versions are the tagged releases on GitHub — each ships a self-contained Windows `.exe` and macOS `.dmg` with the OCR data bundled. From 0.12.0 the card database is downloaded on first run rather than carried in the installer.

The format loosely follows [Keep a Changelog](https://keepachangelog.com/); this project uses simple `MAJOR.MINOR.PATCH` tags.

## [0.24.0] — 2026-09-15

### Added
- **Compare two decks — or a deck against a list or a link.** On the deck builder's front page, **⇄ Compare** makes the decks tickable: tick two and press **Compare ticked**, or tick one and compare it **with a pasted list** or **with a link** (Archidekt, Moxfield or EDHREC — nothing is imported, the other side is read for the answer and forgotten). The result is three columns — *Only in A*, *Only in B*, and *In both*, the shared cards run in different numbers first — compared by card name on the main board plus commander, so a Moxfield list's printings never count as differences. Every card on the other side carries what the shelf says: **not owned**, **own it, free**, or **in other decks** (basics always count as free). Every row opens the card, large, with each side's count. Each column's heading totals its cards — price × copies — with each line priced the way the buy list prices it: a reading off the card's own Cardmarket page within the fortnight first, else a 7-day average of the history on file, else Scryfall's trend; lines priced off a Cardmarket page are marked green, and the tooltip says what each price is. **☆ Wish list the difference** sends everything B has that A doesn't onto an existing or new wish list, with quantities, leaving out copies you already own that A isn't using (tick **include owned** to send everything); names no card data matched are listed with a `?` and left out. **Swap sides** looks at it the other way round without re-fetching. The comparison is its own page in the route, so ← Back returns to the deck index.
- **The deck index as a list.** A **Tiles / List** switch beside the Decks heading (remembered between runs). The list is one line per deck — pin, name and id, format, card count, colour identity as the real mana symbols, and the stock badges — sorted *Pinned, then recent*, *A–Z*, or *Grouped by colour*: fewer colours first, then the groups A–Z by name, each headed by its symbols and the game's name for it (*Mono · Black*, *Dimir · Blue / Black*, *Grixis · Blue / Black / Red*), colourless and no-commander decks in groups of their own. A pinned deck sits at the top of its group, and of the A–Z list.

## [0.23.2] — 2026-09-14

### Changed
- **Visual Stacks fills the width.** The stacks used to be laid out by the browser's newspaper columns, which settle on the height of the tallest stack and then fill the columns one after another to it — so with one long section (36 creatures) a wide window got four columns of stacks and a black quarter-page beside them. The app packs the stacks itself now: each section goes, in order, into the column that is shortest so far, so every column the window is wide gets used. Split is unchanged, since there a long stack may break to level the columns.

### Fixed
- **The card view fits the window.** In the deck builder and the inventory browser the card art was always 540px wide, so in a short window — or with View → Zoom turned up, which the app remembers between launches — the card ran past the bottom of its panel and the rest was behind a scrollbar. The art now shrinks to whatever height the window leaves it, keeping its proportions, so the whole card is always in view.

## [0.23.1] — 2026-09-14

### Changed
- **The two biggest pages are split by feature** (nothing changes on screen). DeckBuilding.tsx (3,241 lines) is now the deck index alone (440); one open deck is `deck/DeckDetailView.tsx`, with its version history, card-swap table and feeding wish lists each in their own file beside it. WishLists.tsx (2,740) is now the wish lists page alone (856); the To buy list is its own component, `wishlists/BuyList.tsx`, and the card view, the hand-add search, the paste and shop modals and the shared helpers live beside it. The smoke run checks the buy page's add-a-card modal and price watch as well, and that the self-update bar sits above the launcher without squashing its tiles (what 0.22.0 got wrong).

## [0.23.0] — 2026-09-14

### Added
- **The tag build starts the app it just built before packaging it.** `npm run smoke` (tests/smoke/smoke.ts) drives Electron with Playwright against a scratch inventory seeded from the reference DB and walks every page — the inventory browser and a ± round-trip, the Collection count, a deck, the wish and buy lists, the set collector. It runs on the Windows runner between `npm run build` and the installer, so a bundle that will not launch (0.21.0) fails the release instead of the user.

### Changed
- **Every internal call is on the typed contract** (nothing changes on screen). 0.22.0 introduced `src/shared/ipc.ts` for eight channels; the other 144 are there now, generated from the preload's own signatures, and every page calls `window.api.invoke('deck:rename', …)` instead of a hand-written method. The preload is down from 781 lines to the events main pushes plus that one door, and `handle()` in main refuses a channel the contract does not know — so the two ends cannot drift.
- **The stock ledger is brought level with the stock, and checked at every start.** The ledger 0.22.0 seeded from the scan log inherited the scan log's gaps — sales, undos and finish moves from before it existed only ever changed the stock — so on eight stacks it disagreed with what is held. A migration writes one correcting `reconcile` entry per stack (the stock is not touched), and from now on the app checks the two agree when it opens: if they ever drift again — an older build sharing the Dropbox file would do it — a warning names the stacks, and Settings → Inventory storage shows them with a **Reconcile ledger** button. A conflicted copy's reconcile entries are its own baseline and are never replayed by the merge.
- **Pages no longer ask for their own reloads.** The 75 `loadDecks()` / `loadLists()` / `loadInventory()` calls that followed every write are gone, along with the reload callbacks threaded through the deck page and the older `inv:changed` event: every write announces what it touched and the pages refetch from that. Two writes that reached across families now announce it too — an arrival on the buy list puts copies into stock, and a Cardmarket price read off the panel changes the buy list — so the Collection page and the buy list follow them as well.
- CI runs on Node 24, the Node that Electron 43 embeds, so the tests exercise the same `node:sqlite` build the shipped app uses (they ran on Node 26).
- The deck card's flip and print buttons sit under the art, as they do in the inventory viewer.

### Fixed
- The self-update bar keeps its height above the launcher instead of squashing the tiles.

## [0.22.0] — 2026-09-14

### Added
- **A stock ledger, and a Merge button for Dropbox conflicts.** Every change to the stock is now a row in a ledger that is only ever appended to — a scan, a sale, an undo, the browser's ±, a finish move — each with its own id and the price at the time. (The scan log the Collection page's "scanned today" reads is unchanged: it still shows the adds that stand.) That is what makes running the app on two machines safe: when Dropbox leaves an "inventory (… conflicted copy …).db" beside the live one, Settings → Inventory storage → **Merge conflicted copies** replays the scans, sales and moves the other machine made since the two parted — dated when they happened — and renames the copy so it stops counting. Running it again applies nothing twice, and a copy made by an app from before this release merges the scans its log records. The start-up warning now says so instead of asking for the files to be reconciled by hand. Existing inventories get their ledger seeded from the scan log on first open.

### Changed
- **One typed contract for the app's internal calls** (nothing changes on screen). A call from a page to the data layer used to be four edits — a handler, a preload method, a hand-written signature and the call — and the two ends could drift. `src/shared/ipc.ts` now maps each channel to what it takes and answers; main's `handle()` and the renderer's `window.api.invoke()` are typed off that one map, so a new call is the contract line plus the handler. The channels added this release (the inventory browser's four, self-update's three, the app version) go through it; the rest can move as they are touched.
- **The data layer is split by domain** (nothing changes on screen). `DataStore` was one 7,400-line file; its methods now live in `src/main/store/` — decks, swaps, versions, wish lists, the buy list, prices, sets, the inventory browser — with the connections, schema, migrations and inventory writes left in `store.ts`. Every caller still sees the one class with the one API; the same 633 tests pass over the split unchanged.
- **Every page now keeps itself current.** A change made on one page — a copy added in the inventory browser, a card sent to the buy list from a set, a deck edited — used to show on the others only when they reloaded, which some did on their own and some didn't (the deck list, coming back to it; the Collection totals). Now the app itself says what each write touched, and every page showing that kind of data fetches it again at once. Coming back to a page also paints what it showed last time straight away and refreshes underneath, rather than going blank first.
- **The Inventory browser is an ordinary page of the app now.** Up to 0.21.1 it was a separate web page served by a tiny HTTP server inside the app and shown in an out-of-process frame — which is where the stuck keyboard after visiting it, the inputs that would not type until a restart, the stale totals until a reload, and the five-second polling all came from. It is a React page like every other, reading the store the same way the deck builder does, so a card scanned in or sold anywhere in the app shows on the grid the moment it happens, and the card view is built from the same pieces as the deck builder's (the flip, the magnifier, the two price lines, the pack line, the add-to-deck and wish-list pickers, Cardmarket, copy, print), with the stock rows and the "imported to collection" date that are this page's own. Everything the page did it still does: the three scopes, the set dropdown with counts, every chip and tri-state chip, the subtype and tribe boxes, the value band in pounds, the sorts, paging, ← → through the cards, F to flip, right-click for the deck picker. It also follows the app's art style, mode and contrast directly instead of being told them on a query string.
- **SQLite now comes from Node itself.** The app, the scripts and the tests all used `better-sqlite3`, a native module compiled against one runtime's ABI — Node's or Electron's, never both — which is where the rebuild step, the install-script allow-list and the occasional silent crash in a script came from. Node ≥ 22.13 has `node:sqlite` built in, and the Electron the app ships carries the same Node, so the same code now runs everywhere with nothing to compile. The store's SQL is untouched: a thin wrapper (`src/main/sqlite.ts`) provides the `pragma()` and `transaction()` calls it was written against. Nothing changes on disk — the database files, journal modes and migrations are the same — and the installer is a few megabytes lighter.

## [0.21.1] — 2026-09-14

### Fixed
- **0.21.0's installer crashed on start** with "Named export 'autoUpdater' not found": the self-updater is a CommonJS package and the app's main bundle is ESM, so Node refused the named import before a window existed. The default import carries the same object, and the app now uses that. Anyone who installed 0.21.0 needs to install this one by hand once; from here the updater does it.

## [0.21.0] — 2026-09-14

### Changed
- **Numbered database migrations.** Both databases now carry a version stamp, and the columns the app used to add on every open by asking "is this column here yet?" are one numbered list per file (`src/main/migrations/`), run in order inside transactions the first time a database that is behind is opened. An inventory from any earlier release comes up the same as before — that path is tested against the shapes that shipped — and the `CREATE TABLE`s now carry every column, so a brand-new inventory is complete without running anything. A database made by a newer build is opened as it is, with a warning, rather than touched.
- **A test runner.** The check scripts (`check-wishlists`, `check-buylist`, `check-deck-*`, `check-parse`, `check-contrast`, …) are now `tests/*.test.ts` under vitest, run by `npm test`, and CI runs them on every push and before every installer build — previously only the parser and contrast checks were gated, and the store suites needed a hand-built bundle under Electron's Node. `check:refdb` and `check:fixtures` stay as scripts because they need real data.
- **Dependencies:** `adm-zip` 0.5 → 0.6.1 (the backup archiver — two advisories closed; the 26-check backup suite still passes) and the five transitive audit findings resolved; `npm audit` is clean. `package.json` now declares Node ≥ 22.13, which ESLint 10 needs — a machine on an older Node gets warnings on `npm install` until it's updated. It also carries npm 11's `allowScripts` list: newer npm skips dependencies' install scripts unless the project approves them, and `better-sqlite3` (the native build), `esbuild` (its binary), `electron-winstaller` and `fsevents` need theirs; `tesseract.js`'s (a funding banner) is denied. Without the list a fresh `npm install` on npm 11 leaves the app unable to open its database.
- **The reference DB now keeps every printing's whole Scryfall card**, not just the columns the app happened to need — so a field wanted later (artist, rules text, keywords, whatever Scryfall adds next year) is simply read, with no schema change and no re-import; rules text, keywords, artist, mana value and the per-face details are also lifted out as columns. Stored deflated against a shared dictionary, ~600 bytes a card, so the file grows from 46 MB to about 140 MB. Existing installs get the new columns empty until the next **Refresh card data**.
- **Pointing at a stacked card no longer lifts it out of the stack.** The preview column shows the card whole; the stack stays put, with an accent edge on the strip under the pointer, and only the bottom card of a stack is seen in full — as on Moxfield. The same in the overlapping grid.

### Added
- **The Windows app updates itself.** A little after launch (and every few hours after that) it asks the releases page for a newer version, downloads one in the background, and installs it when the app next closes — or straight away from the bar that appears under the header (**Restart to update** / **Later**). Settings → About says where it is: up to date and when that was checked, downloading with a percentage, ready, or why a check failed, with a **Check now** button. Nothing is installed without a restart, so a scan session is never interrupted. macOS is not included: an unsigned `.dmg` cannot be swapped in by the updater, so a Mac still installs by hand from the releases page, and the About panel says so.
- **👁 View options on every deck and wish list — six layouts, seven groupings, five sorts, and an extra-data line.** One button on the deck page's toolbar and the wish list's opens a panel of chips: **View Style** (Text, Condensed Text, Visual Grid, Visual Stacks, Visual Stacks (Split) — a long section broken into columns of a dozen — and Visual Spoiler, every card in one grid with no headings), **Group By** (Type, SubType, Rarity, Color, Color Identity, Mana Value, Set, No Grouping), **Sort By** (Name, Mana Value, Price, Rarity, Color) and **Include Extra Data** (Mana Cost, Price, Set Symbol — a line under each card, or above it in a stack). It is one choice for the whole app: set it on a deck and the wish lists show the same way, and back. Every section heading has an arrow to fold it away, remembered per section. The commander keeps its own first column under any grouping and the deck's tokens their last; the owned / buy-list / cut / legality marks carry into every layout, and clicking a text row opens the same card view a tile does. Wish-list rows now carry mana cost and colours, and both lists carry the set symbol, so the new groupings have what they need.
- **The layouts now match Moxfield's, measured off the site.** Visual Grid is rows of small cards that overlap so only each card's name and the top of its art show; Visual Spoiler is the same cards laid out whole, with headings; Visual Stacks are 196px cards flowing down newspaper columns, so a short section sits under the one before rather than taking a column of its own; Visual Stacks (Split) lets a long stack break across columns so the columns come out level. Text rows are 30px with a hairline between them, names wrap rather than truncate, and Condensed is the same at 23px with no lines. Type sections run Creatures → Planeswalkers → Battles → Sorceries → Instants → Artifacts → Enchantments → Lands, lands last, as a deck list is read; the commander's section and the one after it always share the first column, so the commander is never a column on its own. Headings read `Creatures (22)`, with the section's value after it when the Price extra is on; the extras only ever affect the text rows — nothing is captioned under a card in the visual layouts.
- **Text rows say a card's state with the row, not with marks by the cost.** A card you own in this printing looks like any other line; one you own in a different printing has its name in amber, one you own none of in red, one another deck is holding in amber with a ⚠, and a cut card is struck through with ✂ in front. Only the buy-list state (🛒 / 📦) and an illegal card (🚫) still get a mark, and it sits by the name — the owned dot that used to sit beside the mana cost, and read as one more pip, is gone from the text layouts.
- **Text rows show each card's mana cost by default** (Include Extra Data → Mana Cost, now on unless you turn it off); Condensed Text stays names only.
- **Two-faced cards in the text rows take two lines** — the front face's name and cost on the first, the back's on the second, the `//` between them in the theme's accent — and, when the card really has a back (Scryfall's layout says so — an adventure or split card has // in its name but only one face), carry a little circle-of-arrows: press it and the preview on the left turns the card over; press it again, or point at another card, and it turns back. The preview has the same button under its art. Elsewhere a two-faced cost reads face by face with ` // ` between.
- **Real mana symbols.** Every mana cost in the app — the layouts, the card views, the sideboard, the stats — is drawn with the Mana font (the coloured, shadowed circles printed on the card) instead of text pips.
- **A card preview on the left of every deck and wish list.** The card under the pointer — in any layout, a text row as much as a tile — shows large in a column on the left with its type line, set and number, price and stock (owned this printing / other printing / not owned / in another deck; on the buy list or ordered). Before you point at anything it shows the deck's commander, or the first card on a wish list; it keeps the last card you pointed at, and clicking the art opens the card view.
- **A cut pool in the deck builder's Card Swap panel.** Drag cards from the deck into ✂ Cuts (or press ✂ Cut in the card view) to line them up to leave without saying yet what replaces them — they stay in the deck, marked ✂ on the stacks. Tap a wish-list card and press ⇄ Pair on the cut it replaces, or drag the wish card onto the cut; the pair lands in the swap table as before, with a Stock column saying whether the replacement is already in stock, ⇄ Swap cards per line, ↩ to unpair, and ⇄ Move all above the table. A cut a swap hasn't been decided for is left alone by Move all and by the exports.

## [0.20.1] — 2026-09-13

### Changed
- **The 16-bit CARDVAULT wordmark is blue-and-gold fantasy lettering** in place of the carved, weathered one that shipped in 0.20.0.
- **Sending a card to the buy list is confirmed where you can see it.** The one-line note used to sit under the wish list's toolbar — out of sight from the card's own view, which has a modal over it, and from the bottom of a long list — so a send looked like nothing had happened. It is now a floating message at the foot of the window ("Stoneforge Mystic sent to the buy list — now ×2"), above the card view, gone after a few seconds or on a click; every wish-list and buy-list confirmation uses it.

### Added
- **☑ Select on a wish list: tick several cards and act on them together.** A **Select** button on the list's toolbar (or *Select* on a card's right-click menu) puts a tick box on every tile and a bar under the header with the count, **All** / **None**, and the three things to do with the ticked cards: **🛒 Add to buy list** (as many copies as each wants; cards already on order are left out and the message says so), **📂 Move to…** — a menu of your other wish lists plus **＋ New wish list…**, which names a new list and moves the cards into it in one go, merging copies where the target already wanted a card — and **✕ Remove**, after a confirmation. Ticked tiles are outlined in the accent; the hover cart and ✕ stay out of the way while ticking.
- **A `/buy-deck` run leaves a Cardmarket reading on every printing it priced**, the same one 💷 Get Cardmarket price would have written — the cheapest UK offer, the price trend, the 30/7/1-day averages and the available count, read off each product page with the app's own UK-sellers filter — so the card panel and the buy list stop saying *not read yet* for cards the run has just shopped for. The skill's `cmread.js` parses the page the way the app's reader does and `cmreads.py` writes `cardmarket_prices` (and a euro trend into `prices.db`) the way `saveCardmarketPrice` does; the app itself is unchanged, and the README's deck & wish list index section now documents both writes a run makes.

## [0.20.0] — 2026-09-12

### Changed
- **New CARDVAULT wordmarks for the 8-bit and 16-bit styles** — a blue-and-gold pixel one and a carved, weathered one — in place of the placeholder the two had been sharing.
- **A deck card whose buy-list copies are all ordered says 📦 Ordered**, in the accent, instead of 🛒 In buylist — on the stacked card and in its card view. While any copy is still to buy it stays In buylist, with the ordered count in the tooltip.
- **A List card's price is read from its original printing's Cardmarket page.** A card from The List has no Cardmarket product of its own on Scryfall, so 💷 Get Cardmarket price used to land on a name search and read nothing; now it opens the original printing's page — Surrak and Goreclaw (PLST) reads March of the Machine: Extras #337 — and the popup says so. And when the app does have to fall back to a name search, clicking the printing you mean on the page now reads its price, with the UK-sellers filter put on first.
- **A wish list's Send all to buy list, Copy list and Export leave out the cards already on order.** A card on its way would only be bought twice; the message after Send all says how many were left out for that reason.
- **"Hide ordered" on the buy list's title row.** A tick box that takes the "Ordered, on the way" section and its tiles off the page, so the list is what's still to order; the count in brackets says how many lines are hidden. Remembered across launches. The **📦 All ordered** button is gone — one stray click marked every line ordered, and undoing that meant unmarking them one by one. Lines are still marked ordered one at a time, and **📬 All arrived** stays.
- **The buy list's paid-against-market figure is green when you paid under the 7-day average**, red when over, in every look. In the high-contrast look the gain used to draw in that look's blue.

### Added
- **📥 Paste order, on the buy list.** Copy the order confirmation off Cardmarket's page (or another shop's) and paste it in: the app reads each line — `1x`, the name, `#225`, the condition, the price — shows them back so a bad paste is seen first, asks which shop, and marks every line ordered with what you paid per copy. A card that isn't on the list is added as that printing and marked in the same go; a printing already ordered is left alone; a name it can't find is named in the message. A euro price is turned into pounds at the day's rate.
- **A name and a collector number narrows the printings** when adding a card by hand, on the buy list and in the deck builder — `evolving wilds 153` shows that one, `evolving wilds 15` the handful whose number starts 15, the exact number first. `#153` works too. The `(FIN) 123` and `fin 123` forms are unchanged.
- **"📦 Ordered 10 Sept" under a wish-list card's name** once its buy-list line has been marked ordered — the small badge over the art was easy to miss. The shop is in the tooltip; a different printing on order says so with a dashed pill.

## [0.19.4] — 2026-09-12

### Changed
- **The "Add to a wish list" / "Add to another deck" picker sits indented under its button**, edged in the theme's accent, and the row after it is back in line with the rest.
- **The two price pills read as two sources.** The pill says **CARDMARKET** rather than "cm", in blue in every look, with "from" in plain text and the figure in the same blue; the **SCRYFALL** pill is green in every look. In the ordinary looks the two used to share the palette's green.

### Fixed
- **"From £16.70" was read as £1670.** The cheapest offer on a Cardmarket page comes as "£16.70" while the guide comes as "15,96 £", and the reader treated every dot as a thousands mark. The last separator is now the decimal point when one or two digits follow it, whichever character it is. Press 💷 Get Cardmarket price again on a card that shows a silly figure and the reading is replaced.

## [0.19.3] — 2026-09-12

### Fixed
- **Cardmarket readings are kept.** Every reading off a card's Cardmarket page had been failing to save — "could not be kept — the app's log says why" on every card — because an inventory from before the reading learned about pounds still had the table's old columns, and the app never brought it up to date. It does now, on launch, and the reading stays on the card.

### Changed
- **💷 Get Cardmarket price, on every card view.** The button that used to say "Cardmarket page" says what it does: open this printing's Cardmarket page once, inside the app, read the cheapest UK offer and the price guide off it, and keep them on the card. The moment the figures are read the page goes away and the popup shrinks to the price — **from £x**, with trend, 7-day and 30-day under it — and a Close button; Esc closes it too. When Cardmarket won't serve the page the popup says so in a sentence, without an error, and leaves the page up so a "verify you are human" tick can be answered.
- **The inventory browser's card view has the button too**, and the same two price lines as everywhere else.
- **Two price lines on every card view, both in £.** *scryfall* — Cardmarket's trend as the card data recorded it, dated, with the app's own 7-day and 30-day averages and the foil figure — and *cm* — the last reading off the card's own page: from, 7-day, 30-day, trend, copies on sale, and when it was read. Euro figures are in the tooltip. Wish list, buy list, deck, set collector and the inventory browser show them in the same order: type and set, inventory and buy-list status, the two price lines, then the quantity row and the buttons.

## [0.19.2] — 2026-09-11

### Changed
- **A deck's missing cards go to a wish list as well as the buy list.** Two buttons on the deck's title row — **🛒 Add missing cards to buy list** and **☆ Add missing cards to wish list** — replace the one that sat at the bottom of the Missing singles panel. The wish-list one asks which list, or takes a name for a new one; each card goes on once, wanting as many copies as the deck is short, and cards already on the list are left alone and counted in the message.

## [0.19.1] — 2026-09-11

### Added
- **The price watch shows the card.** A small picture of the printing heads each row, beside the name; clicking either opens the card, its figures and its six months.

## [0.19.0] — 2026-09-11

### Added
- **One way around the app.** The header is now the only place navigation lives: the drawn title on the left (click it for the launcher), the open section's face and a breadcrumb — *Wish lists › Shredder upgrades*, *To buy list › Price watch* — and on the right **← Back** and **Main menu**. Back walks a real history, whichever section the page before was in: To buy → a list's name → **← Back** lands on To buy again, which the old "← All wish lists" button could not do. Alt+← and the mouse's own back button do the same. The in-page "← All decks / sets / wish lists / To buy" buttons are gone — the breadcrumb's section name is the way up — so every page's title row carries only its actions, all at the right end. The wish lists index has a **🛒 To buy list** button in the same place (it used to only show a count and say "open it from the main menu").
- **Classic is pop art.** New portraits on every launcher tile — a lens-eyed elf, a grinning goblin, a crowned dragon, a brass automaton and the rest — a pop-art CARDVAULT wordmark drawn large over the launcher, and an owl on the Main menu button. The Spider-Verse faces are retired.
- **A framework for art styles.** A style is now a folder of pictures plus one row in a table. `npm run build:style -- <id> <folder-of-pngs>` turns any folder of full-size PNGs into the tiles, home icon and wordmark at the sizes the app wants; the registry finds every folder itself and reports at startup if one is missing a picture or a row. `src/renderer/src/assets/styles/README.md` says what each picture must be, which palette family (skin) to pick or how to add one, what every colour token is for, and the contrast floors CI checks. Classic goes through the same pipeline as every other style.
- **What Cardmarket said, on the card.** Every card view — wish list, buy list, deck, set collector — now shows the last figures read off that printing's Cardmarket page (from, 7-day, 30-day, trend, copies on sale, and how long ago), with a euro figure's £ equivalent beside it, or says plainly that the page has not been read yet. The buy list refreshes the moment a read lands. A foil line says the reading is of the non-foil page.
- **"Move to another list" names the lists in the skin's accent colour**, and so does the buy list's **From** column — which drops the word "from" the header already says. A wish list's name there is a link: click it and you are on that list.
- **"🛒 In buylist" on the card itself.** One blue flag, wherever a card is drawn, when it is on the buy list: on wish-list tiles (where it replaces the grey "To buy"; an ordered card still says **📦 Ordered**), on every card in a deck — on the right of the stacked strip so the name stays readable, shrinking to the cart icon when "In another deck" needs the room — and on the set collector, where a gap that is already on the list is lifted out of the dimming so the flag can be read. Deck cards count any printing of the name; wish lists and sets, the exact printing. A count follows when more than one copy is on the list. A wish list pins a printing, so when only a *different* printing of the card is on the list the tile says exactly that — a dashed **Other printing in buylist** / **Other printing ordered** — never the plain flag.

### Changed
- **A new app icon and a drawn title on every art style.** The installer, the `.exe`, the Dock and the running window all carry the CV emblem — a yellow monogram on an orange disc — in place of the painted Spidey. Every pixel style now draws the app's name in the header rather than typing it: sugar skulls and marigolds for Day of the Dead, iron and spikes for both Phyrexian styles, and the bone-and-blood lettering for Pulp Horror and the two fantasy sprite styles. Classic keeps its typed title. Pulp Horror's separate high-contrast cut is gone: the new lettering carries its own dark outline, so one cut reads on every ground.
- **Pulp Horror has an amber accent.** Its accent used to be the same cream as its text, so anything "in the accent" — a wish-list name, a link, the Ordered pill — was invisible as such. It is now the skin's amber; the cream stays the body ink and every hairline.
- `npm run dev` now runs `npm install` for you when `package-lock.json` has changed since the last install on that machine — a dependency added on the other computer no longer crashes the app on launch after a pull.
- **Every colour in the app now comes from the palette.** The stylesheets carried some sixty hard-coded colours — leftovers of the first dark theme — which is why light mode, the Phyrexian styles and the paper styles each needed a patch of overrides, and why anything the patch missed came out wrong: navy table rules on aged paper, a purple filter chip, a white hairline invisible on a light page, the Inventory browser's "Add to deck" button wearing the app's blue under the browser's red edge. They are all tokens now (with `color-mix` for the in-between shades), the override patches are gone, and two tokens were added so a skin can say what ink and edge its accent takes — `accent-ink` and `accent-edge` — so the primary button, the on-state toggles and the filter chips are right in every skin without a rule apiece. The contrast audit checks the new pair on every push.
- **The Inventory browser dresses by skin, not by style name.** It used to know two style names and guess the rest: under Pulp Horror it wore the Phyrexian greys, under Day of the Dead the fantasy navy. The app now hands it the palette family, the mode and the contrast — the same three things it puts on its own page — so it matches whatever is on. High contrast wins there too, as it does in the app.
- **The Set collector's All / Owned / Missing filter shows which is on.** The three buttons had no styling at all; they are pills now, the active one filled in the accent.
- **Cardmarket's panel is tidier about what it read.** Closing the panel abandons a read in flight (a hidden page can't pass the challenge, and its figures used to land under the next card's name); a name-search page says so instead of sitting silent; a reading that could not be kept says that rather than "kept"; a reading older than a fortnight steps aside for the published price series on the buy list; and stale tooltips promising a "💶 figure" that no longer existed are reworded.
- Dead weight out: a full-resolution copy of every camera frame that nothing read (drawn twice a second in auto scan), unused props and exports, orphaned comments, the unused precon `<details>` branch, three unreferenced CSS rules and the light-mode patches that only undid literals.

### Fixed
- **A Cardmarket reading's trend never reached the price series.** It was written to a table the inventory database no longer has (the series moved to `prices.db` in 0.18.0), so every reading threw, was swallowed, and the panel still said "kept". It goes to the right file now.
- **Primary buttons under the pixel styles lost their fill on hover**, and the pixel styles' accent-ink override blocks meant a new skin arrived with the wrong ink on its buttons until someone wrote it a rule. One token now.
- First-run setup promised the card database could be moved later "in Card data"; nothing there can. The sentence is gone.

## [0.18.0] — 2026-09-11

### Added
- **Your inventory is backed up every time the app starts.** A copy of `inventory.db` goes into a `backups` folder beside the inventory itself — so on the shop machine it rides Dropbox off the machine, which is the whole point: a backup that only exists on the computer that might die is not a backup. The file's **name** carries everything you need to choose one — `inventory-2026-09-07-200802-1798cards-start.db` — the time it was taken, how many cards were in it and why, so the restore list can be read without opening a single database or unpacking a single zip.

  It does not grow forever. Every five loose copies roll into one zip (five 1.7 MB copies come to 2.2 MB), the newest ten zips are kept, and the folder settles at about **29 MB** instead of climbing 1.7 MB every launch. Copies are taken with SQLite's own `VACUUM INTO`, so each one is consistent, compacted and free of the `-wal`/`-shm` sidecars, and written to a `.part` name that is only renamed into place once it is whole — a backup interrupted half-written never joins the list. An archive is only deleted from after it has been read back and found to hold every file it should.

  **Settings → Backups** takes one on demand, opens the folder, and restores. Restoring takes a fresh `pre-restore` backup **first**, so choosing the wrong one is itself undoable, then lists every backup newest-first with its card count. The file is opened and questioned before anything is overwritten — a file that is not a CardVault inventory is refused rather than swapped in — and once it is in place the app reopens it and every page refreshes on the spot.

- **Send a card somewhere else from inside a deck.** Opening a card in the deck builder now offers the same two things the Collection's card view has always offered: **＋ Add to deck** and **☆ Add to wish list**, with a pick-or-create list. Finding a card in a deck and wanting it on a wish list used to mean leaving, going to the Collection and finding it a second time. The deck you are already in is left out of the "another deck" list, and a card already on the wish list you pick is said so rather than added twice.

- **The price watch can watch one wish list at a time.** A **Wish list** dropdown on the price watch: all of them, or just one, by name. Every figure follows it — the count of cards wanted, the count under their own averages, and the days-on-file note — so the tally always describes the table underneath it rather than the whole watch. A card wanted by two lists shows under both.

### Changed
- **The price history now covers every card, and lives in its own file.** It used to be kept for the ~1,700 printings you own, want, have in a deck or on the buy list — which is no use the day you first look at something new, because the days you needed are already gone. Every card-data refresh now writes down the trend for **every card Scryfall prices — 97,929 of them**, so any card you click has a series waiting.

  That will not fit in a file that syncs, so the series moved out of `inventory.db` into a new **`prices.db`** beside the card database: local, rebuildable, and never re-uploaded by Dropbox when you scan a card. Your existing series is copied across on first launch, checked, and only then dropped from the inventory file, which reclaimed **26 MB** here (36.7 → 11.2 MB). The 7- and 30-day averages the Set collector prices with stay in the inventory, where they are small and worth backing up.

  Two things keep the size sane: only the days a price actually **moved** are stored — about half of them on any given day — and the gaps are filled back in when the series is read, so a card that sat still all week still has a seven-day average and its graph still draws a flat line rather than sloping between the two days it changed on. Stored prices cost ~110 bytes each against ~180 before, so a snapshot of everything is about 6 MB and a year of weekly refreshes lands near 300 MB, on the local disk.

- **A deck tile now wears its commander's colours.** The tiles were all the same, so finding the deck you meant was reading names. Each one is now tinted by its commander's colour identity — a Golgari deck sits in black and green, a mono-red one in red — which is how you think of a deck before you have remembered what it is called.

- **The price watch's controls have a bar of their own.** They used to ride the title row beside the heading, which wrapped as soon as the window was anything but wide, and the tick box came out the size of a text field with a tick stranded in the middle of it — the shared rule that pads a field for typing into does not suit a checkbox. The heading and the tally keep the first line; the wish list picker, the tick and **↻ Refresh** get the second, with Refresh at the far end since it re-reads the series where the other two only change what is shown of it.

- **A wish list's name, on a price-watch row, is now in the theme's own accent** rather than the same grey as the set code beside it — so you can see at a glance which list a card is wanted by. Every art style paints it in its own accent: Day of the Dead's marigold, Pulp Horror's cream, the default blue.

- **"Show Inventory" is now just "Inventory."** One name, everywhere it appears — the launcher tile, the page heading, the Collection's button and the instructions on the empty wish list and buy list pages.

- **The Deck guide writer's fields sit on the left again**, where the rest of the Settings panels put theirs.

### Fixed
- **Text fields, dropdowns and text boxes were the wrong colour in four of the six art styles.** They were filled with the 8-bit style's dark navy, which had been written into the rule every pixel style shares — so the Phyrexian styles, Pulp Horror, Day of the Dead and light pixel mode all drew their fields in a blue that belonged to none of them. A navy dropdown on Day of the Dead's aged paper is now aged paper. 8-bit itself is unchanged.

- **The primary button was close to unreadable in high contrast (dark).** That mode's accent is a pale blue, and the button's label was white on it — a contrast ratio of 1.95:1, in the one look chosen for legibility above everything else. The label is black there now, at 10.8:1. The same repaint carries the on-state tick boxes with it.

- **The colour audit that should have caught it now runs on every build.** The palette check measured an accent as writing *on* a background, but never as a background *under* writing, which is exactly the pairing that failed. It now checks the label colour each art style puts on an accent-filled button.

## [0.17.0] — 2026-09-10

### Added
- **A card's Cardmarket page, inside the app.** A **🛒 Cardmarket page** button on any card view — Collection, wish list, deck, buy list, Set collector — opens that exact printing's page in a panel in the middle of the window, filtered to UK sellers and English copies, so the price you are looking at is one you can actually pay. Beside it, **↗ Cardmarket in my browser** hands the same URL to your own browser, where you are signed in.

  Why a panel and not a quiet lookup: Cardmarket has no API open to apps — they state plainly that they are not accepting applications — and the site sits behind Cloudflare, which answers anything that isn't a real browser with a challenge page. So it is a real browser page, and it has to be visible: a hidden one never gets past the challenge, and now and then it asks you to tick "verify you are human". One page load, on a click, on a card you are looking at. There is deliberately no "price this whole list" button: a run that walked a list loading a page per card got this app's browsing session blocked outright, and pacing did not disguise it — hundreds of product pages from a session that does nothing else is a shape, not a speed.

- **The app now keeps its own price history.** Nobody else keeps one it will hand over: Scryfall publishes a snapshot and replaces it, so yesterday's number is gone, and the reference database is overwritten by every card-data refresh. So the app writes the numbers down as they go past — one row per printing per day per source, in a new `price_history` table. Every card-data refresh snapshots the trend for everything you have a stake in (in stock, in a deck, on a wish list, on the buy list), dated by the card data's own day rather than the day you pressed the button; the trend off every Cardmarket page you open is kept dated too, so a card you look at leaves a point behind on its own graph. It starts itself: on first launch after this update the app captures the card data already on the machine, so there is a first point on the graph rather than an empty table. For a 1,700-card stake that is about 150 KB a snapshot, so a weekly refresh costs under 8 MB a year.
- **Ninety days of back-history, fetched once.** Settings → **Price history** has a button that pulls MTGJSON's last 90 days of Cardmarket prices for every card you own, want or have in a deck. It is the only free back-history of Cardmarket prices there is, and it turns a series that starts today into one that starts three months ago — 139,000 prices across 1,647 printings here, in about 70 seconds. It is a large download (around 150 MB, plus the set files needed to match their cards to yours, which are then remembered), so it is a button you press rather than something that happens to you. Days the app recorded itself are left alone, and running it again only tops up what is missing. Cross-checked against the app's own snapshot: for 3 September both sources give Super Shredder as €25.02, to the cent.
- **A price watch on the To buy page: is now the moment?** A new **📉 Price watch** button on the To buy toolbar opens every printing your wish lists want, one row each, with the newest day on file beside that card's own **7-day, 3-month and 6-month** averages. A price under any of its own averages is green, and those cards sort to the top with the deepest discount first — *"▼ 28% under its 6-month"* — so the list opens on the cards worth buying today rather than in alphabetical order. A tick box narrows it to only those. Clicking a card gives you its art, the same four figures, and the six months drawn as a line with its high, its low and the day it was last priced; from there it goes straight onto the buy list.

  A **Range** column sits beside the averages — the cheapest and dearest that printing has been across everything on file, with the low in green when today is sitting on it. It is the honest long view while the series is young: MTGJSON publishes 90 days and no more, so the 6-month column holds whatever there is until the app's own daily snapshots carry it past three months, and the page says so with the day count under the heading rather than quietly showing the same number twice.

  Cheap means cheap *for that card*: everything is measured against its own history, not against other cards, which is the only comparison that survives the difference between a £0.03 common and a £40 land. Nothing here fetches anything — it reads the series the app already keeps, so it costs nothing and works offline. Until the series reaches six months the page says how many days it actually holds rather than letting two columns look like two answers.

- **The Cardmarket page you opened now leaves its price behind.** Clicking **🛒 Cardmarket page** on a card has always loaded that printing's page in the panel. The figures on it — cheapest offer, how many are on sale, trend, 7-day, 30-day — are now read off the page once it settles and kept, with the time they were read. They show under the panel's title straight away, and a buy-list line that has been looked at shows *cm from £8.00 · 2 hours ago* under its price; a Cardmarket 7-day average, being a real reading of the real market, is preferred over the app's own averages for that line's price.

  Nothing is fetched for this. The page is up because you clicked, and reading what is already on your screen costs Cardmarket nothing — which is exactly the line the old bulk pricer crossed, and why there is still no "price this whole list" button and never will be. If their firewall does object, the panel says so plainly and stops rather than knocking again, and offers to reset its browser session.

- **The buy list prices at the 7-day average, not the day's trend.** Every line showed the Scryfall trend under a *NOT PRICED* tag — one morning's snapshot, badged by what `/buy-deck` hadn't done rather than by what the number was. A line now shows what the card actually goes for: the **7-day average** wherever the app has a week of history for that printing (and that finish), and the trend only where it doesn't, each labelled so you can see which you are reading. The same rule the Set collector already prices by, so the two pages agree. The header total, and the paid-against-worth figure under a line, follow the same number. Where a line has no week behind it the list says so and offers the MTGJSON 90-day fetch right there, rather than sending you to Settings to find it.

- **A card can go straight onto the buy list, foil or not.** The To buy page has an **➕ Add a card** button: search by name and every printing of it comes back with its art, set and price, or go straight to one with a set and a collector number — `(FIN) 123`, or just `fin 123`, brackets optional. Pick the finish (a printing never issued in foil won't offer it, and a foil-only one starts on foil), pick the copies, and it's on the list; the search stays open for the next card. Lines added this way read *from Added by hand*, and the From filter picks them out like any wish list.

  Foil is now a real thing on a buy-list line rather than an assumption. The line carries the finish: it is priced at the foil trend rather than the non-foil one, it exports as `1 Sol Ring (LTC) 288 *F*` so the shop and `/buy-deck` both know, and when the parcel lands its copies go into stock **as foil** instead of arriving non-foil and needing flipping by hand. Every line already on your list keeps the finish it has — the new ✨ beside a card name flips it, and a send never rewrites what an earlier one asked for.

- **Mark ordered asks what you paid.** The picker that asks where you ordered from now also takes the price per copy, without postage. It is stored on the line, follows it into the History when the cards arrive, and sits under the line beside the Cardmarket 7-day average with the difference between the two — what you paid against what it goes for. Leaving it blank still marks the line ordered.

- **New launcher art for both pixel styles.** Ten new tiles each for 8-bit and 16-bit.
- **A fifth art style: Pulp Horror.** Ten painted 1970s horror-paperback covers, each with its own title screamed across the top. The rest of the app is the Phyrexian look with high contrast on — black ground, bone hairlines, the same square-cornered chrome — and it stays that way while the style is on: Pulp Horror is dark only, like the Phyrexian pair, and always high contrast, because the softer chrome washes out against covers painted in black ink. Your own light/dark and high-contrast settings are remembered untouched and come back with the next style.

### Changed
- **A new app icon — the spider emblem, everywhere.** The installer, the `.exe` and its desktop and Start Menu shortcuts, the macOS `.app` in Finder and the Dock, and the window and taskbar icon of the running app all now carry the same artwork, at every size Windows and macOS ask for (16px in a taskbar up to 1024px on a Retina Dock). One source image produces all three formats the packagers want — `npm run build:icons` — so changing it again is one command rather than three rounds with an image editor. The running app also finally gets its icon when packaged: it looked for one in the install folder that was never shipped there, and fell back to whatever Windows had embedded in the `.exe`.
- **The old prices now say what they are.** A deck, a wish list and the buy list each showed a total with no source on it — and it is not a live price: it is Cardmarket's **trend** price for every card as Scryfall recorded it, which is only as fresh as your last card-data refresh. Those totals now read *"£188.73 Scryfall trend · 3 Sept, 6 days old"*, with the date of the data and how old it is, and hovering explains the difference from the live 💶 figure beside them. Nothing about the number has changed; it just stops being mistaken for today's price.
- **Every page uses the whole window.** Pages were laid out in a fixed 1,100px column with the rest of the monitor left empty — on a 1080p screen that was a third of the width, and on anything bigger it was most of it. There is no page width in pixels any more: the content runs the window, and what it does with the extra space follows the window rather than a number in the stylesheet. The wish list index and a list's cards, the Set collector and the deck index all fit as many tiles per row as there is room for (a 100-card wish list is 9 across on a 1900px window rather than 5); the buy list's tables and the Collection's stock list stretch, and the buy table's price, source and *Ordered* columns grow with them, so a line that used to wrap onto three rows now sits on one; Settings lays its cards out side by side, one column on a laptop and two or three on a big monitor, packed so a tall card leaves no hole beside a short one; and the scan page's *Just scanned* card grows with the window instead of holding a fixed 340px beside a widening gap. Help text keeps a readable measure — it is capped in `ch`, so it follows the theme's own type rather than a pixel count — because the point is to give the space to the cards, tables and tiles that gain from it, not to run a sentence across two feet of glass.
- **A wish list is now grouped by card type.** An 88-card list was one wall of art you had to read name by name. Its cards now sit under labelled rules — Lands, Creatures, Planeswalkers, Battles, Instants, Sorceries, Artifacts, Enchantments, in that order — with the count of lines beside each heading. Only the types actually on the list get a heading, and nothing is reordered inside a section.

### Fixed
- **A wish list's header counts copies, not lines.** It read "88 cards · £104.21" off the number of rows and one copy of each price, so putting a card up to ×3 moved neither number. Both now count the copies you actually want, matching the copies-per-list figure the wish list index already shows.

## [0.16.0] — 2026-09-07

### Added
- **Mark ordered now asks where from.** Marking a buy-list line (or the whole list) ordered opens a small picker: the usual shops as one-click chips — Cardmarket, MageFinder, Magic Madhouse, Manaleak, Chaos Cards, eBay — plus a free-text box for anywhere else, a person, a car boot. Whatever you type is remembered and offered as a chip next time, and leaving it blank still marks the line ordered. The shop shows on the 📦 pill in the Ordered table, on the tile, in the card's detail view and in the History's Ordered column once the cards have arrived; a 🏬 button beside the pill (and in the detail view) changes it. Tools outside the app see it as `ordered_from` on `buylist_contents`.
- **Click a card on the Set collector to open it.** Every card in a set's grid now opens the same full-card view the inventory browser gives it — art that flips (F) and zooms, set and collector number, which packs hold it, Cardmarket and USD prices, how many you hold and whether it is already on the buy list, a *Copy card details* button, and the Scryfall and Cardmarket links. ← and → step through the grid as it is filtered, so with **Missing** on you walk the missing cards alone. Deliberately absent: *add to deck* and *add to wish list* — this page is about one set, and the missing half of it already has 🛒 **Missing → buy list**.

### Changed
- **The deck page's "Copy buy list" is now "🛒 Missing → buy list".** It used to put the shortfall on the clipboard and say so in the status line, which read as nothing happening when what you wanted was the list itself. It now sends every copy the deck cannot field — the cards you own none of, plus the ones another deck is holding — straight to the buy list, one copy per shortfall, and each line there says which deck sent it, the way wish-list lines name their list: a 🃏 chip in the line's detail, and the deck in the buy list's **From** filter, so Copy list and Export can be narrowed to one deck's order. A deck renamed or deleted afterwards keeps its name on the lines it sent. The clipboard copy is gone from the deck page; the buy list's own Copy list does that job now.

### Fixed
- **The deck sideboard's rows are readable again, and open the card.** A rule meant for the panel's *Add to sideboard* button was matching every button inside the panel, so each row's ✕ stretched across the whole line and left the card name no width at all — a row was a thumbnail and a large ✕ and nothing else. The ✕ is back to its own size, and the rest of the row is now a card: name, copies, mana cost and type line beside the art, and clicking it opens the same full-card view the deck stacks give it. The thumbnail stays the size it was.
- **"Different printing owned" now says which deck has the printing you pinned, and lets you move it.** When two built decks pin the same printing and you own one copy, the app gives it to the older deck and the newer one was told only that it had "a different printing" — which reads as wrong when that very card is sitting in the box. The line now names the deck holding it (*Command Tower MSC #235 · in Fayes Sinister Six*) with the same **⇄ Move to this deck** the "In another deck" list has; the deck that had it then reports the shortfall, and no decklist changes. The card's detail view names the deck too. A pinned printing you own no copy of still shows as before, with nobody to blame.
- **The "Copies wanted" stepper in a wish list's card view no longer overlaps the note beside it.** It borrowed the buy list table's stepper styling for the button sizes, and with that came a rule meant for a table column — hug the buttons, take no width — which as a flex item shrank the stepper to nothing and let its buttons spill over "already covered by stock". It now lays out as its own little row.
- **The buy list no longer scrolls sideways.** Every cell in its two tables was set to keep to one line, and in the pixel skins' monospace face the seven columns came to ~1,280px against a 1,100px page — so the tables grew a horizontal scrollbar and the card names, the one column allowed to shrink, folded onto two lines. The column widths are now pinned on the header row, the card name takes whatever is left, and the price, source and stock cells wrap rather than hold their width. The sideways scroll remains only as the last resort for a window narrower than about 1,040px, where the columns would otherwise crush the card name to nothing.
- **A card-data refresh that is quit during "Finalising…" no longer vanishes without a trace.** That stage is where the set list is fetched and the new database replaces the old one, and closing the app there discarded the whole 78 MB download silently: the next launch opened the old data with no error, leaving a complete-looking `reference.db.tmp` (every card, no sets) and the bulk file sitting beside the real one, and with them the Set collector's symbols missing, since those only arrive with a *finished* refresh. The progress line now says so — *keep the app open — quitting now throws the download away* — and the app clears any leftover `.tmp` and bulk download at launch, so a half-finished refresh costs a re-run and nothing else.

## [0.15.1] — 2026-09-04

### Changed
- **The framing box now moves over the whole card, and it resizes.** In 0.15.0 it sat over Scryfall's art crop, which is nearly the tile's own shape — so on an ordinary card the box had almost nowhere to go, and on a borderless printing it clipped whatever Scryfall's cropper decided was art, title lettering included. Now you see the entire card and the box can sit on the illustration, the text, a signature, wherever.
  Drag the grip in its corner to make it smaller and the tile zooms in on that part — enough to frame a face rather than a whole picture. The box keeps the tile's shape as it shrinks, so what is inside it is always exactly what the tile will show. A slider and the **+** / **−** keys do the same. Up to 4×; past that a card image is being blown up beyond what a tile can resolve.
  A deck you have not framed still shows Scryfall's art crop, exactly as before. **Use art crop** puts a framed one back. Anything framed in 0.15.0 keeps its place at 1×.
- **Settings is regrouped** the way most applications settle on: **Appearance** (art style, light or dark, accessibility), **Scanning**, **Deck guides**, **Card data**, **Storage**, **About** — with jump links along the top and the version number at the end.

### Fixed
- **Deck tile art was never cropping to fit under the 8-bit or 16-bit styles**, and the new Phyrexian ones inherited the fault. The art painted at its natural size from the top-left corner, which is why framing appeared to do nothing there: a stylesheet rule for pixel-style buttons was quietly resetting the tile's background sizing, and a deck tile is a button. Classic was never affected. This has been true since tile art shipped; the framing box just gave you a control that made it visible.
- **Adjust framing from the card view opened the editor behind the card.** It sits on top now, and one press of Escape closes the editor alone rather than the card behind it as well.
- **Checkbox labels in Settings stacked the box above centred text.** Every checkbox in the app inherited a column layout meant for captions over fields, and was given a text field's width. Box then words, on one line, flush left.
- The native About panel still described the app as MIT licensed. It is not, since 0.12.0.

## [0.15.0] — 2026-09-04

### Added
- **Two new art styles: Phyrexian 8-bit and Phyrexian 16-bit.** Bone-white biomechanical horror on near-black — the same ten launcher subjects as the fantasy sets, drawn in monochrome. Pick them in **Settings → Art style**, same as the others.
  Both are dark only. The artwork is pure greyscale, and on a light page it stops reading as artwork at all. **Light** still appears in **Light or dark**, greyed out with a line saying why, and your choice is remembered — switch to another style and it comes straight back. Nothing is overwritten behind your back.
  Deck and wish-list tiles show their card art in greyscale under these two styles, so a wall of full-colour Scryfall crops does not sit inside a black-and-white theme. The badges on those tiles keep their colours, because that is what tells **Commander** from **To buy** at a glance. So do the owned/missing/contended status colours everywhere else — a grey "you own none of this" dot would be a legibility bug dressed up as a design decision.
- **Choose which part of a deck's art the tile shows.** A tile crops its art to fit, and it used to keep whatever sat in the middle, which is often not the face. Now there is a box to drag over the picture, with the real tile underneath showing the result as you move it. Arrow keys nudge it; hold shift for bigger steps.
  Reachable two ways: the **⛶** handle on the tile itself, under the pin, and next to **Set as deck image** in the card view — where you are when you pick the art in the first place. The tile handle is the only way to reframe art that came from the commander rather than being chosen by hand.
  Choosing different art clears the framing. A spot picked on one illustration means nothing on another. **Centre** puts it back.

### Fixed
- **The subtype and tribe suggestion lists stay on the screen.** Choosing **Creature** in the inventory browser and opening the subtype box drew a list of forty-odd creature types that ran off the bottom of the window, with no way to reach the rest of it. The list is now capped to the room actually below the box and scrolls inside that — in a short window it stops short of the bottom edge rather than running past it.
  It also picked up the things the old one could not do: the count of each type on the right, arrow keys and enter, and it follows the box when the filter bar rewraps onto another row.

## [0.14.6] — 2026-09-04

### Fixed
- **A deck guide no longer throws away a reply you have paid for.** The request allowed 16,000 tokens for the whole job, and on Claude that budget covers the model's thinking as well as the guide itself — so a six-section guide ran out mid-sentence, the JSON would not parse, and the entire answer was discarded with nothing to show for the money. The limit is now 64,000, which costs nothing extra: you are billed for the tokens generated, never for the ceiling.
  If a reply is cut off anyway, the finished sections are now kept instead of the lot being binned. A section that was still being written when the reply stopped is dropped rather than tidied up and shown — half a paragraph presented as a finished one would be worse than not having it.
  The raw reply is also written to `ai-last-response.txt` in the app data folder every time, so a failure leaves something to look at rather than a bill and a shrug, and the error says where it is. A reply that ran out of room now says so, rather than blaming the JSON.

## [0.14.5] — 2026-09-04

### Fixed
- **A provider's refusal now reads as a sentence rather than a JSON dump.** Asking for a deck guide without API credit reported itself as `400 {"type":"error","error":{"type":"invalid_request_error","message":"Your credit balance is too low…"},"request_id":"req_011…"}`. Both SDKs bury the useful sentence inside a body and set the error's own text to the status plus the whole blob; the app digs the sentence out and shows that instead.
  Where there is an obvious next step it adds one — a rejected key points at Settings, an unknown model at the model list, and running out of credit says outright that **API credit is separate from a Claude.ai or ChatGPT subscription**, with the console to top it up at. That last one catches people first and is the easiest to misread: a Pro or Max plan does not pay for API calls.

## [0.14.4] — 2026-09-04

### Changed
- **The Set collector's section headings actually collapse now.** The ▶ on **Started** looked like something you could click and wasn't. Both it and **All sets** are real buttons: click either to fold that grid away, and the arrow turns down when it is open. With a hundred started sets the full list sits a long way down the page, so which one you leave open is remembered between visits.

## [0.14.3] — 2026-09-04

### Added
- **A "▶ Started" box at the top of the Set collector** — every set you own at least one card from, most complete first, above the full list of everything. The sets you are actually collecting were otherwise scattered through nine hundred you are not, and the near-finished ones were the hardest to find. It says how many you have started and how many are complete, and a started set still appears in **All sets** below.

### Fixed
- **The set progress bar is legible in the light themes.** The track was drawn in the border colour, which on the light pixel theme is a dark brown, under a fill in the accent colour, which is a dark orange — two dark colours against each other, so the whole bar read as one line and you could not see how far along a set was. The track is now a pale neutral wash that works under both palettes, the bar is a little taller, and a completed set fills in the theme's own "ok" colour rather than a hardcoded green.
  A set you own nothing from draws no fill at all — the bar keeps a 3px floor so that one card in four hundred still shows, and without that guard every untouched set wore a sliver of progress it had not earned.

## [0.14.2] — 2026-09-04

### Fixed
- **The mouse wheel now works anywhere on a page, not only over the cards.** Every section scrolls inside a box that was capped at 1100px and centred, so the empty margins either side belonged to nothing scrollable — point at them and the wheel did nothing, and you had to keep the cursor over the grid. The scrolling box now spans the window and the content is centred inside it, which looks identical and means the wheel works wherever the pointer happens to be.

## [0.14.1] — 2026-09-04

### Fixed
- **The surge foil badge no longer appears on cards that are not surge foils.** Scryfall records the treatment on the *printing*, and 693 surge printings across Magic also had an ordinary non-foil run — for those, only the **foil** copy is a surge foil. The badge was reading the printing and claiming it for every copy, so a non-foil Final Fantasy Commander card sat there labelled surge foil.
  It now asks which copy you actually hold: a foil (or etched) one, or a printing that only ever existed in foil — 1,748 are like that, and every copy of those really is one. The **Surge foil** filter follows the same rule, so the chip and the badges can no longer disagree. Browsing all cards, where there is no copy to ask about, a dual-finish printing is labelled **surge foil (in foil)**.

## [0.14.0] — 2026-09-04

### Added
- **Set collector.** A new page on the main menu for working through a set. It opens on every paper set as a grid of Scryfall's own set symbols with a completion bar under each — how many of its printings you hold against how many there are — and a search box that takes a set name or code and narrows the grid as you type.
  Open a set and you get every card in it, in collector-number order like a binder page. The ones you own are in full colour with a green tick; everything else is dimmed, because the gaps are the point of the page. Filter to **All**, **Owned** or **Missing**, and the header says how far along you are and what the rest would cost at Cardmarket prices.
  Three things to do with the missing half: **Copy missing** puts them on the clipboard as `1 Name (SET) 123` lines, **Export missing** saves the same to a .txt, and **Missing → buy list** sends every one of them to the To buy list in a single go.
  "Owned" counts distinct printings, not copies — a set is finished when you have one of each, and four Llanowar Elves is still one slot filled. Tokens and art cards are left out for the same reason.
  Set symbols come from Scryfall and are cached to disk the first time each one is drawn, so the page works with no connection after its first visit. They need one **Refresh card data** to appear.

### Changed
- **Card data has moved into Settings.** It was a launcher tile of its own for one button; it now sits with the other things you set once and forget, freeing its place on the menu for the Set collector.

## [0.13.3] — 2026-09-04

### Changed
- **New art for the To buy list tile** in both pixel sets: a medieval messenger fleeing a castle drawbridge with an armful of sealed letters and a dragon roaring out of the gatehouse behind him. The first attempt was a modern postman outrunning a rottweiler, which was the only present-day scene in a set of wizards, goblins, dwarves and elves. This one belongs.

## [0.13.2] — 2026-09-04

### Changed
- **The To buy list has its own place on the main menu.** It was a button tucked into the Wish lists header; it is now a tile of its own — Mysterio in the painted set, a postman outrunning a rottweiler in both pixel sets. The list itself is unchanged: the same two tables, the same steppers, ordered and arrived flags, history, filters and exports, fed from the wish lists exactly as before.
  The wish list page keeps a quiet **🛒 N on the buy list** count in its toolbar, so sending cards over still shows you it worked — it just is not the way in any more.

### Removed
- The unused Green Goblin face left over from the old vector set.

## [0.13.1] — 2026-09-04

### Added
- **Surge foil and extended art now show on the card, in Show Inventory.** A blue **≋ surge foil** badge and a green **◧ extended art** badge sit with the existing full art and borderless ones, and both have a filter chip beside Extended art so you can pull every one you own.
  Surge foil was invisible to the app until now, and worse than merely missing. Scryfall models only three finishes — nonfoil, foil and etched — so a surge foil **is** an ordinary foil in every field the app stored: same border, same frame effects, same art, same layout. *Doctor Doom, King of Latveria* in Marvel Super Heroes Commander is the case that shows why it matters — #6 and #880 are identical on every one of those, and are **€0.88 and €5.65**. The only thing that separates them is Scryfall's `promo_types`, which the reference data did not keep. It does now.
  Nothing was priced wrongly before: the app pins each card by its Scryfall id, so the two Dooms were always separate rows with separate prices. What it could not do was *tell* you which one you had, or let you check.
  **This needs a card-data refresh.** The new column starts empty and cannot be back-filled from anything already stored, so no badge appears until **Card data → Refresh card data** has run once.

## [0.13.0] — 2026-09-03

### Added
- **Deck guides written by Claude or ChatGPT.** The app already works out the keep rule and the opening-hand odds for every deck, offline and for free. The other half of a guide — what the cards actually *do* — is now available on a deck page: **Write it** fills in the deck by role, the play pattern turn by turn, win conditions, rules traps and warnings, a table of budget swaps with what each one loses, and an ordered upgrade path.
  It needs **your own API key**, in **Settings → 🤖 Deck guide writer**, and each guide costs money on your own account. Pick **Claude (Anthropic)** or **ChatGPT (OpenAI)**; both are asked the same question, so the difference between two guides is the difference between the models. The model menu is fetched from the provider using your key, so it only ever offers models your account can actually run.
  The key is stored on that machine alone, in the app data folder and **never in the inventory database** — so it cannot follow a shared inventory into Dropbox — and is encrypted by the OS where the OS offers it. The Settings page says outright when it cannot be. Nothing here is required: with no key the app behaves exactly as before, and that is what anyone you hand it to gets.
  The model is explicitly told **not** to write a mulligan guide or a keep rule. Those are computed from the list, and a guessed keep rule sitting under a measured one would be worse than nothing.

### Removed
- **Obsidian, entirely.** The vault export of scanned cards (**💎 → _Collection**) and the write-back that mirrored deck edits into a linked vault note are both gone, along with the deck import removed in 0.12.4. None of it worked without a synced vault, which nobody but the author has, and the app is being handed to people who have none. Decks keep every card, name and version; only the link to a note is gone.

## [0.12.4] — 2026-09-03

### Added
- **A keep rule on every deck, worked out from the deck itself.** The deck page has a new **✋ Keep rule & opening hands** panel: the land range to keep on seven and on six, the odds behind it (your keep floor, three lands, an early play, no lands, a flood), the three hand shapes to ship, a reading of the curve, when the commander is castable and which cards you can cast on turns one and two. The numbers move with the list — a 38-land deck and a 32-land deck do not keep the same hands, and the panel says which yours is and why in one line.
  It needs no internet, no key and no Obsidian vault, so it works for anyone the app is given to. Two things it will not tell you, both said out loud rather than guessed: there are no colour-screw odds when the deck runs a non-basic land, because the card data here does not record what a Command Tower taps for and an invented probability is worse than none; and a deck short of its format's size is flagged, because a half-built Commander deck reads as wildly land-heavy and its keep rule with it.

### Removed
- **The mulligan guide read from an Obsidian note.** It only ever appeared on a machine with the vault mounted, so for everyone else the panel simply was not there. The computed keep rule above replaces it and works everywhere.
- **Importing a deck from the Obsidian vault** — the deck page's **💎 Obsidian** button and the note picker behind it. **📋 Clipboard**, **📄 File**, **🔗 URL** and **✚ Blank** are unchanged, and they are every route that works without a vault. A deck already linked to a note still writes changes back to it; nothing about the collection export to Obsidian changes.

## [0.12.3] — 2026-09-03

### Added
- **Search the inventory by tribe.** Show Inventory has a **tribe** box beside the card-type menu: type `Elf`, or `Elf, Goblin` for several, and the grid narrows to those creatures. It suggests the tribes actually in front of you, commonest first, so the list is what you own rather than every creature type Magic has printed. It works in all three scopes — your inventory, all cards, and not owned — and needs no card type chosen first, which is what the older subtype box required.

### Fixed
- **Subtype and tribe searches no longer match parts of other words.** The subtype filter asked whether the type line *contained* the text, so searching `Rat` returned every Pirate, `Ox` every Fox, `Ape` every Shapeshifter and `Bat` the Wombats — on the page and, in the all-cards scope, in the SQL behind it. Both now match whole words. Tribe goes further and only reads creature faces, so `Aura` is a subtype but never a tribe, while a Kindred card — "Kindred Sorcery — Elf" — counts with its Elves, which is the point of one.

## [0.12.2] — 2026-09-03

### Added
- **Wish lists remember how many copies you want.** A wish list was a set of printings with the count implied as one, so wanting four Sol Rings meant one row and a note to yourself. Open a card on a wish list and there is now a **Copies wanted** stepper beside the inventory line, and the tile carries a **×N** badge once it is more than one. Stepping down to zero clamps at one rather than deleting the row — the row holds the printing you pinned and the provenance the buy list reads back, and losing that to one stray click on a repeat-clickable button is worse than making **Remove from list** an explicit action. It caps at 99.
  The count reaches everywhere the implicit 1 used to be assumed: **Copy list** and **Export** write `4 Sol Ring (LTC) 280` rather than `1 …`, the card counts on the wish list page are copies rather than rows (matching the buy list, so a list of four Sol Rings and a Mox reads "5 cards"), and **🛒 Send all to buy list** orders what the list actually asks for — it was sending a single copy of a card you wanted four of.
  Existing lists are untouched: the new column defaults to 1, so every row keeps exactly the meaning it had, and `inventory.db` migrates itself on first launch.

## [0.12.1] — 2026-09-03

### Fixed
- **The launcher's faces are the right size again.** 0.12.0 shipped them stuck at their 96px minimum — all nine crammed into a single row with the window empty above and below, and in the pixel styles the blurb sliced through the middle at the tile's bottom edge. The tile maths measured its box through a `useRef`, but the grid is mounted conditionally now that first-run setup can hold the same slot: on the first paint neither screen exists, so the measuring ran against nothing, gave up, and never attached the observer that would have re-run it when the grid appeared a moment later. The size then stayed at its starting value for the life of the window. It measures through a callback ref instead, which React hands the element whenever it mounts. On a 1315×820 window that is 96px tiles in one row against 235px tiles in five columns.
- **Captions no longer clip.** The fixed 66px caption fitted one line of label over two of blurb — but the pixel styles uppercase the label and space its letters, so "Scan cards in" wraps onto a second line and pushed the blurb out of the box. The height is now a `--face-cap` custom property that the tile maths reads rather than keeps its own copy of, so the two cannot drift apart when a theme changes the caption's type, and label and blurb each clamp to two lines so neither can shove the other out.

## [0.12.0] — 2026-09-03

### Changed
- **The installer no longer carries the card database; the app asks where to put it.** `reference.db` used to ride along in the installer, ~40MB of Scryfall data built on whatever day the release was cut. First launch now asks **"Where should the card database live?"**, offers the local app-data folder or a folder of your choosing, and downloads the data from Scryfall — 78MB compressed — with the same byte-counting progress readout the Card data page uses. Three things fall out of it: nothing Scryfall-derived ships inside the installer, prices are current on the day you install rather than as of the build, and a shop with a small C: drive gets a say in which drive holds 40MB of rebuildable data. The installer itself is only about 9MB smaller (135.4MB against 0.11.8's 144.2MB) — a SQLite file compresses well inside NSIS, so the saving was never the point.
  The question is asked once, and only when there is genuinely nothing to work with. **An existing install is not disturbed**: its `reference.db` stays where it is, the setup screen never appears, and the inventory keeps pointing wherever it pointed — a synced Dropbox folder included. Choosing a folder later, from Card data, moves the file rather than re-downloading it, across drives as well as within one.
  It happens on first run rather than in the installer on purpose. The importer needs `better-sqlite3` built against Electron's ABI, so the installer would have to download the file and then launch the app to import it anyway — and a multi-minute import inside an installer, with no resume, turns dropped shop wifi into a broken install instead of an app that opens and offers a retry button.
  A build that *does* want the database bundled — an offline or kiosk install — still can: put `reference.db` back in `extraResources` and the app adopts it exactly as before.

### Fixed
- **The whole launcher fits the window, and grows with it.** The home grid sized its tiles from the window's width alone, so a small window gave three enormous columns that ran off the bottom of the screen — and widening the window added a column, which divided the width further and *shrank* the faces, the opposite of what more room should do. Tiles are now sized from both axes: every column count is tried and the largest square that still fits the leftover height wins. All nine sections are on screen when the app opens at any size, and stretching the window grows the art up to 340px. Captions sit in a fixed-height box, so a blurb that wraps to two lines can no longer push the bottom row off — which also lines the labels up across a row.

### Added
- **A licence.** `LICENSE` covers the source and `EULA.md` covers the installed app, with the plain-text copy shown by the Windows installer before the first install. The parts that matter to a shop: card prices are third-party market estimates and explicitly **not** a valuation or an appraisal, OCR is fallible and scanned results want checking against the physical cards, and your statutory rights as a UK consumer are untouched. Releases up to and including 0.11.8 went out under MIT and stay that way for anyone who has them.

### Added
- **A "To buy" list beside the wish lists.** A wish list is *cards I'd like*; the new list is *cards I'm ordering now*, and there is exactly one of it — **🛒 To buy list** on the wish list page, with a live count of copies. It is fed only from wish lists: **right-click** a card on a wish list and pick **Add to buy list**, hover the tile for the cart button, open the card and press **🛒 Add to buy list**, or **🛒 Send all to buy list** from an open list's toolbar. The wish list itself is never changed by any of these.
  The list is a text list first — `Name (SET) 123`, one line per printing, in alphabetical order, with a **− / +** copies stepper, the price, the wish list(s) the card was sent from, an in-stock badge and a remove ✕ — then the same cards as art tiles underneath. Clicking a line or a tile opens the card, which says where it came from and can remove it. **Copy list** and **Export** write the same `2 Name (SET) 123` lines the wish lists and `/buy-deck` already speak, and **✓ Clear in stock** takes off every line the inventory already covers — explicit rather than automatic, since a card can arrive for another reason.
  Sending a card twice — from two wish lists, or twice from one — makes one line with two copies, and each line remembers every wish list it came from, name snapshotted, so it still reads "from Shredder upgrades" after that list is renamed or deleted. In the database it is the `buy_list` and `buy_list_sources` tables plus a `buylist_contents` view with provenance — and, for anything that prices wish lists, it simply *is* a wish list: a row named **To buy** with the reserved id 0 in `wishlist_index` and `list_index`, its still-to-order lines under `wishlist_id = 0` in `wishlist_contents` with real quantities. `npm run index -- buy` prints it priced, and `/buy-deck To buy` shops for it like any other list.
- **Ordered and arrived, with a history.** Once the basket is paid for, **📦 Mark ordered** on a line — or **📦 All ordered** in the toolbar — stamps it with the moment and puts an **Ordered** flag over the card; the flag is a button, so a mistake is one click to clear. When the parcel turns up, **📬 Arrived** on a line, **📬 All arrived** for every ordered line, or the same from the card's own view: after a confirmation the copies go into stock (as non-foil, since a list pins a printing rather than a finish), the line comes off the buy list, and it moves to a **History** table at the bottom showing the card, copies, which wish lists it was for, and when it was ordered and when it arrived, both in this PC's local time. The stamps are stored as UTC and rendered locally, so the Mac and the Windows box agree on them. `/buy-deck buy` skips ordered lines, and the buy list's `cards` in `list_index` counts only what is still to order.
  **Copy list** and **Export** leave ordered lines out too — they are the next basket, not the whole list — and **Copy all** / **Export all** beside them include everything.
  **What `/buy-deck` found, on the line.** When the skill prices the To-buy list it writes each card's per-copy price, vendor and a note back into the buy list — the one write anything outside the app makes to the database, bounded to four columns of one table. The buy list shows "£3.38 @ Cardmarket · BecPoke92" on the line in place of the reference trend (the trend stays in the tooltip), the header adds a "found £X" total beside the trend total, the card's own view says when it was priced, and the History table gains a **Paid** column carried over when the card arrives. A strip under the toolbar says outright whether `/buy-deck` has priced the list and when, because a page of grey trend prices looks the same whether it ran or not; unpriced lines are tagged **not priced** rather than left as a bare number.
  Back on the wish lists, each card now says where it stands: an **📦 Ordered** flag over the art once its buy-list line is flagged ordered, a quieter **🛒 To buy** when it is on the buy list but not yet ordered, and nothing when it isn't there; the card's own view says the same, with the order time.
  A **From wish list** filter in each buy table's title bar narrows the rows and tiles to lines sent from one wish list (deleted lists included, by name), and Copy list / Export follow the filter.
  The buy list is two tables, each its own box with a title bar and column headings: **To order** (what Copy list and Export give you) and **Ordered, on the way**, outlined in the accent colour so the two cannot be read as one. Every cell in a row used to refuse to wrap, so once an in-stock pill appeared the pixel themes' wider type pushed the row past the panel — a sideways scrollbar, and card names broken onto a word per line. The ordered controls now wrap when they must, the name column has a floor, the row's date drops the year, and the tables sit in their own scroller so the page itself can never scroll sideways.

## [0.11.8] — 2026-09-02

### Added
- **The scanner follows the card, wherever the camera put it.** The corner and title crops were cut from a fixed guide rectangle: the card had to fill 92% of the frame, dead centre. A camera that reframes on its own — macOS **Center Stage** tracking a face in the artwork, or the card held short of the outline — moved the corner text out of the crop and the OCR read the table; one machine's log had 181 of 182 corner reads coming back empty for exactly that reason. A small detector now finds the card first (column and row edge profiles give the candidate sides; boxes of card aspect whose four sides are real *step* edges — table on one side, plain border on the other, which is what tells the card's bottom from a wood-grain line — are kept, and the largest wins; a few milliseconds on a 480px copy of the frame). The crops follow the found rectangle, padded so a black-bordered card on a dark mat, where only the border's inner edge shows, still keeps its collector number in the strip. A **green outline** on the stage shows where the scanner is looking.
  When the card is found somewhere other than the guide, the guide's own crops are appended as fallbacks — the OCR tries variants in order and stops at the first usable read — so a detector mistake reads no worse than the old fixed crop did. Measured on synthetic frames of eight real printings over six table surfaces, zoomed and off-centre: card found 97%, corner text inside the crop 85%, no false rectangles on empty tables.
- **A Center Stage note on Macs.** When the camera reports no controls (the iPhone Continuity Camera and Studio Display case) a note under the stage says the reframing is macOS's, not the app's, and that **Control Center → Video Effects** is where it turns off while the camera is on. The app cannot switch it off itself — Chromium exposes nothing for it — and a feed zoomed past the card's edges cannot be read by anything.

## [0.11.7] — 2026-09-02

### Added
- **A quantity on each "This session" row.** Every scan's row now carries the copies it put into stock — a manual add with a quantity records that quantity rather than a bare row — with a **− / +** stepper that moves one copy out of or into stock, so "actually I have three of these" is two clicks instead of two more scans. Stepping the last copy down is the same as **Remove**. **Foil** and **Remove** act on all of a row's copies, and the Obsidian export writes one line per copy, as repeated scans already did.

### Changed
- **"Shiny" is now "Foil"** on the session rows — the word the rest of the app, and the hobby, uses.

### Fixed
- **The session list no longer grows a horizontal scrollbar.** The base `label` stacks its children in a column and the base `input` is 10rem wide — right for the form fields, wrong for a tick box, which put the box on its own line above the word and, in the pixel styles' wider type, pushed the row past the panel. The tick is explicitly a row with an auto-width box, the controls wrap when they must, and the list clips sideways.

## [0.11.6] — 2026-09-01

### Added
- **A "Not owned" scope in the inventory browser.** The page had two states in one tick — your inventory, or every printing in the reference data — and no way to ask the question you actually ask while shopping: *what does this set have that I hold none of?* You could only answer it by eye, hunting for tiles **without** a green border across a few hundred cards. The tick becomes a three-way scope, because the states are mutually exclusive and a checkbox cannot hold three: **In my inventory · Not owned · All cards**. It sits where the tick sat, beside the set dropdown — the other control that says what you are looking at rather than filtering it — and **✕ Clear filters** leaves it alone for the same reason.
  "Not owned" narrows the query rather than the rows it returns: that mode is paged and the count under the grid is a `COUNT(*)`, so dropping owned cards afterwards would page over the wrong set, report a total that included them, and leave the set dropdown's per-set counts disagreeing with the grid. Since `reference.db` and `inventory.db` are separate connections that cannot see each other, the owned ids are mirrored into the reference connection as a temp table, rebuilt only when the inventory changes — so adding a copy from the page drops that card out of the list at once.

## [0.11.5] — 2026-09-01

### Added
- **Delete a single version from a deck's history.** The Version history panel could only clear the lot — keeping whatever was pinned — so one entry you did not want to keep meant taking the ones around it too. Each row now has **🗑 Delete** beside **⟲ Revert**: the same button, because they are the same kind of action on the same row, with a red label and a confirmation that names the version, says the deck's own card list is not touched, and calls a pinned version pinned. Deleting a pinned version is allowed deliberately — the pin means "Clear history keeps this", and removing one by hand is a different intention.
  The red needed a token of its own. `--danger` is tuned as a **fill** (the ownership pip) and reads at only 3.34:1 as *text* on the dark palette's button face, under the 4.5 floor the contrast check holds everything else to. A dark-mode-only override would be the same fault this project has shipped four times now — a value painted in every look and correct in one — so the palette gains **`--danger-ink`**, set per palette and audited against both the button face and the panel. Only the two dark palettes needed a different value; the other four already passed and keep the red they had.

## [0.11.4] — 2026-08-31

### Added
- **The deck builder's side panels retract.** The buy list, legality, owned meter and sideboard are reference — you read them between passes, not while sorting cards. A slim full-height tab between the stacks and the panels collapses them and hands the width back: measured **830px → 1194px** of stacks on a 1280-wide window, enough to pull a third column onto the first row instead of wrapping. The tab stays put when the panels are away, so the way back is where the way out was, and its chevrons repeat down the full height — a deck is taller than the screen, and a single arrow at the top is one you have already scrolled past. The choice is remembered between sessions.
- **A red pip for "you own none of this."** The ownership dot had two states — green for this exact printing, amber for a different one — and signalled *own none* by having no dot at all. Absence is a weak thing to spot across a hundred cards, and it is exactly the state you are hunting when shopping, so it now gets a mark of its own. A card the reference data couldn't resolve still gets nothing: unknown is not unowned.
  The red is a new palette token, audited by the contrast check like every other colour. It carries a pale outline where the other two have only the dark one — deliberately, because simulating deuteranopia puts amber and red barely 1.3 units apart in the light palettes, so hue alone would not separate the pair that matters most.

## [0.11.3] — 2026-08-31

Nothing drawn around a card that the card doesn't draw itself.

### Changed
- **No box around the cards, in the inventory browser or the wish lists.** Each tile painted a panel and a border behind its card; a card already carries its own hard edge, so that frame was pure chrome — and in high contrast, where the border colour is deliberately near-white on black and near-black on white, it drew a hard rectangle around every single card. The tile now shows a **bracket** instead: the bottom rule plus half-height verticals, which still ties a caption to its card without framing the art. The border keeps the meanings it had — owned, and hover — rather than being permanent decoration.
- **No drop shadow behind a card either.** The 8-bit and 16-bit styles cast a hard `4px 4px` shadow on the inventory's tiles, and the full-screen card carried a soft black glow sized for the dark overlay. Both were being drawn in every mode and only *seen* in the light ones — the pixel shade is near-black in dark and a warm grey in light, so the same shadow was invisible half the time and a grey smear the other half. Same shape of fault as the white card corners in 0.11.2: something painted unconditionally that only one palette reveals. The pixel styles keep their shadows on chrome — panels, buttons, launcher tiles — where the hard shadow is the retro look.
- **Wish-list tidying.** The card grid now clears the *Feeding deck* row rather than sitting flush beneath it, and the per-card copy button is gone from the tiles — copying a card's details stays on the full-card view with the other per-card actions.

## [0.11.2] — 2026-08-31

### Fixed
- **Cards are card-shaped again, in every view and every art style.** The 8-bit and 16-bit styles square the whole UI on purpose — rounded corners being the loudest tell that a interface is modern — but the sweep caught the card images too, and a Magic card is the wrong shape square. Worse, it exposed something the rounding had been hiding: Scryfall's `normal`/`large` images are **JPEG**, which has no transparency, so the rounded corner is filled with **solid white** (measured: pure white running 15px in from each corner of the 488px image, ~3.1% of the width). Against a dark panel that showed as a pale nub on all four corners — 67 pure-white pixels per corner on the full-screen card.
  The corner is now clipped the way **scryfall.com clips it on their own site**: `border-radius: 4.75% / 3.5%` on the opaque image, no transparent PNG involved. Two percentages rather than one because percentage radii resolve per-axis — horizontal against width, vertical against height — so a single figure gives an egg on a card-shaped box; the pair keeps the arc circular at any size, which is why it needs no `calc()` and is correct at 22px in a swap chip and at 540px full screen alike. It also comfortably swallows the white fill, so one number gives both the right shape and a clean corner.
- **Every card view now shares one corner.** Auditing each `<img>` that draws a card face turned up **eight** different radii for the same physical object — 9px on the deck stacks, 8px on the scan preview, 10px on "Just scanned", 4px on the swap table, 3px on session thumbnails, the sideboard, the legality rows and the wish-list chips — several of which no view had ever been checked against. All of them, plus the inventory browser's grid tiles, big view and magnifier, now draw from a single `--card-radius` rule that also holds the corner against the pixel themes' squaring. Adding a card view is one line in one place instead of a radius invented per surface.
  Left deliberately square: the OCR corner crop (a photograph, not a card face) and the launcher/deck tiles (cropped illustration used as UI art).

## [0.11.1] — 2026-08-31

An engineering release: high contrast for accessibility, and the ten structural improvements from the code review — one palette, real files for the viewer page, a CI gate, split monoliths, and a scan-fixture pipeline.

### Added
- **♿ High contrast mode.** A checkbox under Settings → Light or dark, composing with either mode and any art style (the pixel styles keep their shapes and lose only their colours). Pure black-or-white surfaces, thickened edges, an unmissable focus ring — and the ok/warn status pair moved **off the red–green axis** (ok is blue, warn amber), so state colours survive the common deutan/protan colour blindness. Verified numerically: high-contrast text pairs meet WCAG AAA (≥7:1), status colours ≥4.5:1.
- **Prices carry their age.** The Collection header and the inventory browser's totals now say what day the price data was built ("· prices 30/08"), turning amber with a ⚠ once it's over a week old — every £ in the app is only as fresh as the last *Refresh card data*, and a stale number looks exactly like a current one.
- **A sync-conflict warning at launch.** Dropbox "conflicted copy" files beside `inventory.db` mean two machines edited before sync settled; the app now says so in a dialog at startup rather than only in a Settings banner nobody opens.
- **A scan-fixture pipeline.** Run with `MTG_CARDVAULT_SAVE_SCANS=<dir>` and every corner scan saves its exact input images and outcome; curate the misreads into `fixtures/corners/` and `npm run check:fixtures` replays them through the real OCR pipeline — every live mistake becomes a permanent regression test.
- **A CI gate on every push.** Strict typecheck (unused symbols fail), eslint over the viewer page, the corner-parser suite, and a WCAG contrast audit of every palette — the installer build only ever ran on tags, which is how a stale README shipped for a month.

### Changed
- **One palette, defined once.** Every colour for both surfaces — the app and the Show Inventory page — now lives in `src/shared/theme.ts`; the app applies it as custom properties at runtime and the viewer serves it as a generated `tokens.css`. The two stylesheets carrying separate copies is how the same light-mode bug shipped twice in one day.
- **The viewer page is real files.** Its HTML, CSS and JS lived in a 73 kB template literal inside `viewer.ts`, invisible to every tool; they're now `viewerPage/page.{html,css,js}`, bundled at build and served on their own routes — and eslint immediately found a shadowed global and a dozen dead bindings in the freed JS.
- **The monoliths are split.** `DeckBuilding.tsx` (3,725 lines) sheds its modals, card views, analysis panels and shared helpers into `components/deck/`; `App.tsx` (2,209) sheds the reference panel, scan presentation, precon import, storage panel and price helpers. No behaviour change; the strict unused-symbol pass is clean.
- **Every full-card view behaves the same.** Flip (F), the magnifier and Esc-to-close now come from one `cardView.ts` — previously three hand-copied variants that had already drifted (Esc closed some views and not others). **Esc now closes the deck builder's and wish lists' card views**, matching the browser.
- **One TTL cache.** The FX rate, Cardmarket links and booster-sheet caches shared no code and three subtly different failure modes; they now share `TtlCache` — one shared in-flight fetch per key, stale value kept while offline, failures never cached as answers.

### Fixed
- **The inventory browser's tile counters were unreadable in light mode.** The same fault 0.10.2 fixed in the deck builder, in the stylesheet that never got swept — the browser is a self-contained loopback page with its own CSS. The `×N` badge now carries its own light ink on its dark chip, and a single copy dims the *ink* rather than the whole chip (58% opacity on the chip left a `×1` invisible over pale artwork). The own/foil/full-art/borderless badges keep their bright colours in both modes for the same reason: they're read against the chip, not the page.

### Removed
- **The separate-window Show Inventory path.** The browser has been embedded in-app since the `viewer` section existed; the old open-a-window IPC (`viewer:open`), its window plumbing and the two preload methods riding on it went unused and are gone.
- **The viewer's inlined Gwen/Spidey art module** (`viewerArt.ts`, 376 lines, plus its generator) — orphaned when the page header was redesigned; its source SVG no longer even existed.
- **Nine dead stylesheet rules** from retired designs (the two-column scan layout, the old per-card hover controls, the card-modal header arrows replaced in 0.9.1), a leftover keyboard diagnostic `console.log` in the deck-name modal, and three unused exports. Nothing user-visible changes; the checks were "class appears in no component" and the TypeScript unused-symbol pass, then a full walk of every page in light and dark, classic and 16-bit.

### Documentation
- **README caught up from v0.6.2 to v0.11.1**: status line, release count, the looks row (art styles × light/dark), the where-to-buy row (Cardmarket link + booster pull rates), the browser's walk arrows/buy button/pack line, and `viewer.ts`/`boosters.ts` in the project layout. The Obsidian mirror is refreshed to match.

## [0.10.2] — 2026-08-31

### Added
- **The pack odds, split by finish, on hover.** The 📦 line now opens a small table: *Play Booster — 1 in 102 non-foil, 1 in 1,360 foil · Collector Booster — 1 in 136 foil*. A pack's slots are finish-specific, so one blended number hid something worth knowing: a Collector Booster's ordinary rare/mythic slot is **foil only**, which means a non-foil mythic can only come out of a Play Booster. A foil-only printing simply has no non-foil column to fill, which says the same thing without a word of explanation. The line itself carries a dotted underline now, since a hover with no affordance is a hover nobody finds.
  The figures are checked, not asserted: summing every card's expected copies comes to exactly 14.0000 cards for a Play Booster and 15.0000 for a Collector Booster — the real contents of those packs — and the Play Booster's rare/mythic slot works out to a mythic 1 in 7.9 packs against the ~1 in 7 the maker quotes. One caveat: the number is per rarity-and-treatment, not per card, because ordinary rares share one equal-weighted sheet.

### Fixed
- **Quantity chips and stock pills were unreadable in light mode.** A badge that sits on card artwork keeps a dark background in both modes, but these ones never declared a text colour — so they inherited it, which was light in the dark theme by luck and dark-on-dark the moment light mode arrived. The deck's `2×` chips, the deck-tile id, the *owned / move / buy* pills and the illegal-card marker now carry their own ink. Nothing changes in dark mode: it's exactly what they were resolving to before.

## [0.10.1] — 2026-08-31

Light mode, where to buy a card, and which pack it comes in.

### Added
- **☀️ Light mode.** Settings gains **Light or dark**, sitting above the art style picker and independent of it — either mode works with any of the three styles, including the two pixel ones, which get a parchment-and-ink daylight palette that keeps their square corners and hard drop shadows. The choice is remembered between launches and applied before the first paint, so nobody sees a frame of a dark app on the way in, and it carries `color-scheme` with it so the scrollbars, date pickers and focus rings the OS draws come out the right way round too.
  **Every page was walked in it**: the launcher, scanning, sell, the collection table, the deck builder and its card view, wish lists and their card view, precon import and its preview table, card data, settings, and the Show Inventory browser — which is a loopback page sharing no stylesheet with the app, so the mode rides in on its query string beside the art style. What is deliberately *not* repainted is anything sitting on card artwork: the stock badges, deck-tile overlays and the camera stage stay dark, because a card's art is the same picture in either mode and a pale badge on it would be unreadable.
- **📦 Which packs a printing comes in.** The card view in both the inventory browser and the deck builder now says **Play Booster · Collector Booster**, **Collector Booster only** or **Not in any booster**, with the pull rate on hover — *"about 1 in 128"*. Scryfall can't answer this and its `booster` flag actively misleads: every Booster Fun printing is `booster: false`, including showcases that are plainly in Play Boosters, because the flag means "in the base numbered set". The answer comes from MTGJSON's actual print sheets, one ~0.5 MB file per set fetched on the click that first needs it and distilled to a small cache in the app data dir — so it costs one download per set, ever, and nothing at all for a set you never open. A set MTGJSON has no pack data for shows nothing rather than guessing.
- **🛒 Buy on Cardmarket.** The inventory browser's card view gains a button that opens the Cardmarket page for *that exact printing*. Cardmarket keys its products by its own numeric id, which nothing in the card data knows, so the link is read from Scryfall on the click and remembered for the rest of the run; offline, or for a printing Cardmarket doesn't list, it falls back to a name search — which is also the link the button carries from the moment it appears, so it's never dead.
- **‹ › Step through the collection from inside a card.** The inventory browser's full-card view gains the walk arrows the deck builder got in 0.9.1, plus the **←/→** keys. They follow the grid behind them in whatever order the filters and sort have left it, and stop at the ends rather than wrapping.

### Changed
- **The deck builder's *Set as deck image* is a real on/off switch.** It was a ✓ tacked onto a label, which left "is this on?" to interpretation. Now an empty box, the panel's own colours and **OFF** when it isn't set; a ticked box, the accent fill and **ON** when it is.

## [0.9.1] — 2026-08-31

Read a deck without closing the card.

### Added
- **Step through a deck from inside the card view.** The card modal gains **‹** and **›** on the dimmed area either side of the card, and the **←/→** keys do the same — open one card and walk the whole deck without closing and reopening. They follow the deck exactly as it's laid out on screen, categories in order and each sorted by mana value then name, rather than the order rows happen to sit in the database. The arrow is *absent* rather than greyed at each end, so whether there's a **‹** tells you at a glance that you're on the first card.

### Changed
- **The unpinned deck pin is visible now.** It sat at 65% opacity over deck art that can be any colour or brightness, so on a pale crop it all but vanished — a control you can't see is a control you don't know you have. The chip now carries its own contrast rather than borrowing it from the artwork: an opaque well, a lifted glyph and a shadow to cut it off the art behind. It still reads as an offer rather than a state, staying colourless and upright where a pinned deck turns gold and tilts.

## [0.9.0] — 2026-08-30

Pick how the app looks.

### Added
- **🎮 Two pixel art styles.** Settings gains an **Art style** picker, and so does the native menu bar under **View → Art style**: **Classic** (the painted portraits), **8-bit** (bold, high contrast) and **16-bit** (softer and more detailed). Either swaps every image in the menus — the nine launcher tiles, the face beside the section name, the icon on *Return to main menu* — and repaints the whole app to match: square corners, two-pixel frames, hard drop shadows with no blur, uppercase headings and a monospace face. Classic is still the default and is untouched by any of it; the choice is remembered between launches and applied before the first paint, so nobody sees a frame of the wrong style on the way in.
  **Show Inventory follows too.** That browser is a loopback web page served by the main process, so it shares neither the app's stylesheet nor its `<html data-theme>` and stayed resolutely modern — rounded pills and a system sans — while everything around it went retro. The style now rides in on the page's query string: monospace throughout, uppercase labels and headings, square corners, two-pixel borders and hard shadows on the chips, cards and pager. Switching style while the browser is open reloads it into the new one.
  The artwork is a fantasy set drawn for the job — a wizard with a lens for scanning, a goblin shopkeeper for selling, a dragon on its hoard for the collection, a scrying orb for the browser, dwarves raising a fort for deck building, a halfling asleep under a tree dreaming of bacon for wish lists, robed figures at a lit altar for precons, a sphinx in an archive for card data, and a workbench of cogs for settings. Shipped as 512px WebP, downscaled from the 1254px originals in halving steps so nothing aliases: 737 kB for both sets, where the same images as PNG would be 7.5 MB and the nine painted portraits they sit beside are 17 MB.

### Fixed
- **The launcher is a grid again.** The home screen asks for as many columns of tiles as the window will take, but `.home` sets an auto left/right margin inside a column flexbox, and an auto cross-axis margin cancels the default stretch — so the box shrink-wrapped its contents, `auto-fit` resolved to one column, and the nine tiles ran down the middle of the screen in a single file. An explicit `width: 100%` restores the stretch; at the usual window size that's five across. Affects both art styles.

## [0.8.1] — 2026-08-29

### Changed
- **No price is not a price of nothing.** A card showing nothing in the value column isn't free — it's a printing nobody has put a price on, and a card you can't cost is a card you can't shop for. **Find cheaper cards** now moves those lines to the cheapest printing that *has* a price, whatever it costs, rather than skipping them for failing to beat a number that was never there. It also won't pick a zero-priced printing as the cheapest, for the same reason.
  Across the decks here that's 30 lines — Secret Lair, promos, Double Feature and the like — every one of which has a priced ordinary printing to move to. Expect deck totals to rise as a result: that's the point, since those cards were being costed at nothing. The result line reports repriced cards separately from genuine savings, because they aren't the same news.

## [0.8.0] — 2026-08-29

Pin the decks you care about; buy the rest cheaply.

### Added
- **📌 Pin a deck to the front of the list.** A pin sits on every deck tile; click it and the deck goes first, pin a second and it goes second, and so on. Pins hold the order you made them in, and unpinning one never disturbs the rest.
- **💰 Find cheaper cards.** Repins every card a deck can't currently field — ones you own no copy of, and ones another deck is holding — to the cheapest printing on Cardmarket, from the card data already on disk. No network, no waiting. Cards already sitting in the box are left alone: repinning those would move them off the printing you own and have the app report your own cards as missing.
  Two things it won't do. It won't send you shopping in sets that aren't cards to play with — World Championship decks and Collectors' Edition are gold- or square-bordered and often the cheapest thing going, and the legality data can't catch them, because Scryfall's legalities are per card: a 30th Anniversary Smoke carries the same legality string as the Revised one. The set's type is what tells them apart. And it won't chase a price that isn't real — Summer Magic's Library of Leng lists at €0.02 against a €5.40 median across its printings, which is a data artifact, not a bargain. Anything under a fifth of a card's median price is passed over.

## [0.7.3] — 2026-08-29

### Added
- **Every deck tile says whether you can build it.** Opposite the format flag, a deck now carries its stock: **✅ Complete** when you own every card and no other deck is holding any of them, **⇄ N in other decks** when the cards exist but are sleeved elsewhere — move them over and the deck is done — and **🛒 N to buy** for copies you own no version of in any printing. The middle two show together when both are true, because they are different jobs: one is a trip to the boxes, the other is a trip to the shop.
  Worked out from one share-out for the whole index rather than a deck read apiece, and the index now follows inventory changes live like the deck page does.

## [0.7.2] — 2026-08-29

### Fixed
- **"Use the printings I own" works again.** v0.7.0 made the 🎨 panel allocation-aware — it lists a line whose pinned printing you own but *another deck is holding* — while the button still checked raw inventory and skipped every line whose printing you owned at all. Panel said "fix this", button said "nothing to do". The button now asks the share-out which art this deck can actually have, so it repins to the copy going spare.
  The second half of the same bug: the button repinned to the *most-owned* printing, which with one copy each of two arts could land on the very printing another deck has. The share-out's second pass now draws printing by printing instead of from a bare name count, so every deck holds a known art and the repin has a real answer to aim at.

## [0.7.1] — 2026-08-29

Move the card, don't just be told where it is.

### Added
- **✅ Show owned, in the art picker.** *Change printing / art* gains a filter that narrows the grid to printings already in your inventory, with the count on the button and a running "N of M printings" beside it. Off by default — the picker's other job is choosing an art to go and buy.
- **⇄ Move card to this deck.** Clicking a card that's owned but sleeved into another deck now offers to move it. The deck that had it keeps the card on its list and starts reporting the shortfall instead — you moved a card between boxes, you didn't edit either decklist. The **⚠ In another deck** panel gains the same button on every row, plus **⇄ Move all to this deck** at the top for a sitting at the table with the boxes open.
  Moves are stored as exceptions only: an empty table means the collection is shared out oldest-deck-first as always, and the most recent move wins, so moving a card back is just claiming it from the other side.

### Changed
- **The deck holding *your* printing is named first.** When a card is short, the deck holding the exact printing this line pins leads the list — that's the card you're actually missing — and decks holding a different art follow, labelled as such. If no deck holds this printing at all, the card window says so outright, so "move one here" doesn't quietly bring a different art.
- **The label on the card says less; the card window says more.** A contended card in the stacks now reads plainly **⚠ In another deck** — the amber outline and nothing to squint at — and which deck has it appears when you click the card. The side panel lists the cards without repeating deck names.

## [0.7.0] — 2026-08-29

A card sleeved into one deck is no longer a card the next deck can have.

### Added
- **Decks know what the other decks are holding.** A card you own but have already sleeved into another deck no longer counts as one you've got. The whole collection is shared out across your built decks — oldest deck first, arbitrarily, because no deck has priority — and any deck that misses out marks the card in its stacks with an amber **⚠ in <deck>** label naming who has it. A new **⚠ In another deck** list beside the deck spells it out, and the **Cards owned** meter gains a striped middle band for copies you own but can't put in this box.
  Nothing about this is stored. The share-out is recomputed on every read, so buying a fourth Sol Ring silently hands it to whichever deck was short — the label clears itself, with no allocation record to update and none to go stale. Basic lands are exempt.
- **The share-out is printing-specific: Sol Ring A is not Sol Ring B.** Deck lines are matched against copies of the exact printing they pin before anything draws on other arts, so the LTC Sol Ring sleeved into one deck is that physical card and no other deck can have it — and a deck pinned to a printing nobody else wants gets its own copy instead of losing a queue it was never really in. Only demand left over after that draws on a different art, which keeps "you own one, in the wrong art" the softer thing it always was: it lands in the 🎨 panel, not the ⚠ one. That panel is now allocation-aware too.
- **Clicking a contended card says who has it.** The card window gains an **⚠ In use in …** block naming each deck holding copies, whether it's this exact printing or a different art, how many you own, and how many your built decks want between them.
- **📦 Built / 📝 List only, per deck.** A deck marked built is sleeved up in a box, so it holds its copies against the other decks. A list you haven't bought yet holds nothing, sees the whole collection, and can't make your real decks look short. Existing decks all start as built.

### Changed
- **The buy list counts copies you can't actually reach.** "Missing singles" and its exported list now work from what a deck was allocated rather than what exists somewhere in the collection, so a card three other decks have already claimed lands on the buy list instead of being quietly counted as owned.

## [0.6.2] — 2026-08-28

### Changed
- **Deck and wish list tiles show the card's art, not the whole card.** Tiles now use Scryfall's art crop — the illustration alone, no title bar or rules text — at 4:3, which shows the crop almost exactly as painted.
- **Tiles stay tile-sized.** v0.6.1 stretched them to fill the window in both directions, which on a 1440p monitor turned eight decks into wall posters. A wide window now gets more columns instead of bigger tiles, the space where further decks would go stays empty, and resizing scales the columns smoothly from a 240px floor.

## [0.6.1] — 2026-08-28

### Fixed
- **Text boxes and dropdowns no longer die until the app is restarted.** The culprit was `window.confirm` / `window.alert`: Electron on Windows has a long-standing bug where, after one of those renderer-blocking dialogs closes, the window's inputs keep their caret but stop receiving keystrokes — and dropdowns stop responding — until the window is refocused at OS level, which is why a restart "fixed" it. The app had two such dialogs for months; v0.5.0–v0.6.0 added six more on the most-used paths (revert, clear history, swap-in-all, unassign a wish list, remove banned cards, delete), which is why it started happening constantly. Every one of them is now a native message box run by the main process — same look, proper Cancel/confirm buttons, no renderer blocking, and an explicit focus hand-back when it closes. (The embedded inventory browser's focus-stealing, the other historical cause, keeps its existing mitigations.)

### Changed
- **The deck list fills the window properly.** The decks panel now stretches to the full height of the window, and the tile grid takes all of it: with a handful of decks the tiles grow — in both directions — to use the whole area, shrink as the window shrinks, and only start scrolling once each row would drop below a readable minimum. The wish list index behaves the same.

## [0.6.0] — 2026-08-28

Decks know their format, and their format knows its banned list.

### Added
- **Format flags on every deck.** The deck's format shows as a pill flag on its tile on the deck list — icon and name, in the same chip language as the viewer's foil / full art / borderless filters — and the format badge on the deck page is now a dropdown, so a deck's format can be changed at any time (recorded in the version history). The format list grows from 8 to 14, covering every constructed format on [magic.wizards.com/en/formats](https://magic.wizards.com/en/formats): Commander, Commander 1v1, Oathbreaker, Brawl, Standard, Pioneer, Modern, Legacy, Vintage, Pauper, Historic, Timeless, Alchemy, and Casual (house rules — no banned list).
- **⚖️ Banned & restricted checking, per deck, against its own format.** A new legality panel beside the deck lists every card that breaks the format's banned & restricted list — **banned**, **restricted** (Vintage/Timeless, flagged only above the 1 allowed copy), or **not legal** in the format — and the offending cards are highlighted in the deck stacks with a 🚫 mark (red for banned, amber otherwise). Each row has a **Remove** (or **Trim to 1** for restricted) button, plus **Remove all banned**; every removal lands in the panel's "Removed this session" log and the version history *with the reason* — "removed Dockside Extortionist — banned in Commander" — and the card's line is removed from the Obsidian note as any removal is. Sideboard cards are checked too; the maybeboard is not.
- **The banned list updates itself with your card data.** Wizards' Monday banned-and-restricted announcements ([magic.wizards.com/en/banned-restricted-list](https://magic.wizards.com/en/banned-restricted-list)) land in Scryfall's bulk data, which is exactly what **Refresh card data** rebuilds — so every refresh re-checks every deck against the current list, offline, with no scraping. Installers bundle a freshly built reference DB, so a new release carries the current list out of the box; a reference DB from before this feature reports "couldn't be checked — refresh card data" rather than guessing.

### Changed
- **Deck tiles fill the window and scale with it.** The deck index (and the wish list index, which shares the grid) used `auto-fill` columns with a fixed tile height — spare width on a big window became invisible empty columns, so the tiles sat at minimum size no matter how much room there was, and resizing the window only reflowed them. The grid now collapses unused columns so tiles stretch to fill the row, and each tile's height follows its width (16:9, capped so a near-empty page doesn't become one enormous banner) — drag the window and everything grows and shrinks together, art included.
  (This was briefly tagged v0.5.3, which was withdrawn before anyone installed it — it ships here.)

## [0.5.2] — 2026-08-28

### Fixed
- **The version history panel is styled again.** v0.5.1 shipped with every one of its CSS rules accidentally deleted, so the panel rendered as a bare bulleted list with a plain button for a heading. Functionally it worked; it just looked like nothing else in the app.

### Added
- **＋ Save this state** — stamp the deck exactly as it stands, under a name you choose. Until now a version was only written when something *changed*, so a deck you had just imported and not yet touched had no history at all and nothing to go back to. This is what lets you mark a precon **before** the first edit rather than after it. Saving mid-session folds the changes so far into the version you named, so leaving the deck doesn't then write them again under a name you didn't pick.
- **📌 Pin a version.** Pinned versions survive **🗑 Clear history**, which otherwise empties the deck's history. Click a version's name to rename it, so "Before the first tracked change" can become "Precon — as bought".
- Clear history says what it will delete and what it will keep, and refuses to pretend: if nothing is pinned it tells you the whole history is going, including the state the deck started from.

## [0.5.1] — 2026-08-28

### Changed
- **A wish list feeding a deck really is the same thing wherever you meet it.** v0.5.0 gave the deck's assigned lists their own wider chip with a "what does it replace?" dropdown inside it, which still didn't match the compact card chips you get when you pick a wish list from inside the swap panel. They are now literally the same chip, in the same grid, doing the same thing: **tap a card to set it as the replacement**, and the swap panel scrolls into view with the cursor already in the deck-search box, ready for you to say what it's replacing. One layout, one gesture, whether the list got there by being assigned from the wish list page or by being picked in the deck.
- **A card already standing in a swap stays on show but stops being offered** — it keeps the ⇄ mark naming what it replaces, since that is the panel's job, and can't be picked a second time.

### Added
- **A wish list can be taken off a deck.** Each list in the deck's ⭐ panel has an ✕ on its heading that stops it feeding this deck. The list, its cards, and any swaps already made from it are untouched — only the link goes. Previously a list could be assigned to a deck from the wish list page but only unassigned from there, never from the deck looking at it.
- Each list's heading shows its card count and how many of them are already assigned to a swap.

## [0.5.0] — 2026-08-28

Decks remember what you did to them, and the swap workflow stops asking you to click.

### Added
- **Every deck keeps a version history you can revert.** A new 🕘 panel at the top of each deck lists the states it has been in, with a ⟲ Revert on each. A version is one *sitting*, not one card change: opening a deck starts recording, and leaving it saves everything you did as a single entry — so a wish list brought in over twenty swaps is one line to undo, not twenty. Each entry says what happened in plain words (`imported a list of 100 cards · swapped Sol Ring → Mana Crypt · changed Cultivate's print to LTC #288`) alongside the card and line counts at that point. The first change to a deck also saves the state it started from, so there is always something to go back to, and a revert is itself recorded — an unwanted revert can be reverted in turn. Reverting rewrites the deck's Obsidian note to match, the same way every other deck edit does. History rides in `inventory.db` and dies with the deck.
- **⇄ Swap in all**, beside ＋ Add swap. After planning a stack of wish list swaps, this performs every one of them at once — the deck list and the Obsidian note both follow, and any row that can't be applied is named rather than quietly skipped. The button counts what it will do, so it says exactly how many cards are about to move.

### Changed
- **The card to take out can be chosen from the keyboard.** ↑ / ↓ walk the "card in this deck" suggestions and Enter takes the one highlighted, then focus jumps straight to the replacement field, which behaves the same way. Enter with both sides chosen adds the swap. Esc clears the box. The mouse still works exactly as before.
- **A card already lined up to be swapped out disappears from the list.** It was still offered, so the same card could be queued to come out twice. It now drops out of the deck-search suggestions and out of the "what does it replace?" picker on the deck's wish lists, in both cases as soon as the swap is added.
- **A wish list feeding a deck looks the same wherever you meet it.** Assigning a list to a deck from the wish list page gave a plain stacked list, while picking the same list from inside the deck's swap panel gave a grid of art chips. The deck's assigned lists now draw the same chip grid — art, printing and price — so the two views are recognisably one thing.

## [0.4.7] — 2026-08-28

### Fixed
- **The card type, subtype and rarity filters work while browsing all cards.** Picking Land, then Swamp, still listed all 108,258 cards — the first page of everything, in name order. Browsing the whole database is paged 300 cards at a time, so those filters have to be applied by the database rather than on the page, and they never were: they were only ever wired into the set dropdown, which is why the dropdown correctly narrowed to sets holding Swamps while the grid below it ignored the whole thing. The same hole swallowed the rarity chips. Colour, mana value, card value and the treatment chips were unaffected. Land + Swamp now returns 1,161 printings, duals like Badlands included.

## [0.4.6] — 2026-08-28

### Fixed
- **An imported deck keeps the printings the list actually names.** Importing a precon could fill a deck with cards from entirely different sets — the Duskmourn Commander list came back with its Arcane Signet, Sol Ring and Cultivate on Secret Lair printings, its lands on sets the shop has never stocked, and both Forest lines merged into one pile. Only 42 of the 100 cards were the printing the list asked for. Deck links imported from Moxfield and EDHREC threw the printing away entirely and matched on card name alone, which lands on whatever printing came out most recently. Moxfield's and Archidekt's set and collector numbers are now read and honoured, and a set code that resolves with a bad collector number finds the card within that set rather than giving up and going global.
- **A card list that doesn't name a printing lands on the one you own.** "1 Sol Ring" used to resolve to the newest Sol Ring in existence. In an inventory app that's the wrong guess — it means the copy in the drawer. The same precon, imported by name only, now comes back 100 owned out of 100 instead of 42.
- **Substituted printings are reported instead of being silent.** When a list names a printing that can't be honoured, the import says how many cards it had to place elsewhere. Previously the deck simply looked complete while holding cards that aren't on the shelf, with nothing to suggest anything had happened.
- **Split and double-faced cards written with one slash resolve.** `Experimental Lab/Staff Room` matched nothing and sat in the deck as an unresolved row; only Scryfall's `Experimental Lab // Staff Room` worked. The front face is now tried as a fallback, so either spelling finds the card.
- **The wish list's 📋 Paste list box takes typing again.** The name field looked focused but swallowed every keystroke, and clicking out to escape it just closed the window. The embedded collection viewer runs in its own frame and can hold the keyboard; the deck builder already worked around this, and the wish list now uses the same fix.

## [0.4.5] — 2026-08-27

### Added
- **🖨️ Print A4 on every card view** — the collection viewer, the deck builder's card and token views, and the wish list preview. Sends just the card, filling an A4 sheet, at Scryfall's high-resolution art. Handy for proxies, playtesting, or putting a card in front of someone across the counter.

## [0.4.4] — 2026-08-27

### Added
- **Paste a card list straight into a new wish list.** 📋 Paste list opens with whatever's on your clipboard already filled in — name it and it's imported, exact printings kept. Still editable first, in case the clipboard held something else.

## [0.4.3] — 2026-08-27

### Fixed
- **Every card in your collection shows how many you have, including the ones you own a single copy of.** The count was hidden at exactly one, so a single looked the same as a card the app hadn't counted — an unhelpful doubt while stock-taking. Singles are dimmed so stacks still stand out at a glance.

## [0.4.2] — 2026-08-27

### Added
- **Deck exports include the deck's tokens**, listed after the cards with each face on its own line and its own set and collector number — so a two-sided token card gives you both things to look for.

### Changed
- **A paired token is one row, one count.** Two tokens on the same physical card were listed twice, which double-counted a single piece of cardboard. The pair now shows as one card carrying a ⇄ mark, flips between its faces in the card view, and has a single quantity. Unpairing puts both back.
- **The precon preview lays out properly.** The deck name, the whole card table and the buttons were sharing one row, squeezing the table into a narrow strip with its own scrollbar. Name sits left, the warning right, the buttons above, and the table fills the width and scrolls with the page.

### Fixed
- **The precon preview shows the printing that will actually be saved** rather than the one named in the source file. They agree in practice, but the preview should never be able to disagree with the import.

## [0.4.1] — 2026-08-27

### Added
- **A deck shows the wish lists feeding it.** Assign a wish list to a deck and its cards appear at the bottom of that deck, each waiting for you to say what it replaces. Choosing turns it into a normal swap — art, price, and the swap-in button that performs the exchange for real once you've bought the card. Cards already standing in a swap show what they replace rather than asking again.

## [0.4.0] — 2026-08-27

Precon imports show you the cards first, tokens know what you own, and wish lists can be built from a file.

### Added
- **The precon importer shows the whole card list before importing anything.** Picking a precon used to give you a name and a count — the only way to see which printings it would add was to add them. It now lists every card with its set, collector number, count, finish, and how many you already hold, with Cancel sitting beside Import. This matters more than it sounds: a set's ordinary and Collector's Edition decks are separate lists whose cards carry **different collector numbers**, so choosing the wrong one gives genuinely wrong printings rather than just wrong foils.
- **Wish lists can be built from a text file.** Point it at a decklist-shaped `.txt`, give the list a name, done — exact printings kept. It reads the same formats deck import does, and any line it can't match is named rather than silently dropped.
- **A wish list can be assigned to decks**, and to more than one, so a single buy list can supply several decks.
- **Token front and back can be paired up, per deck.** Token cards are printed two to a card and there's no record anywhere of which pairs with which — the same front turns up with different backs. So you set it by hand, from the token's card view, and only for the deck you're setting it on. The card then flips between its two faces.

### Fixed
- **Tokens you own are marked as owned.** A token was only counted if you happened to own the exact printing the deck's card pointed at — which for the Marvel sets is the scarce surge-foil version rather than the everyday one. Five of six owned tokens in a deck read as missing. Ownership now counts any printing of the token, as it always has for cards.
- **Tokens show an ordinary printing rather than a foil one.** The printing displayed is one you own if you have one, otherwise the everyday non-foil version from the right set — not the foil-only variant that happened to ship in the product.

## [0.3.9] — 2026-08-27

The Tokens column now behaves like the rest of the deck.

### Changed
- **Tokens sit on the far right and work like the deck's other cards.** The little `＋`/`−` stepper on each token is gone. A token now shows its count the way a basic land does, carries the same green dot when you own one — which matters, since you need the tokens in hand for the deck to work — and clicking it opens the card view, where the count is set alongside everything else about it.
- **A two-sided token appears once, showing its front.** Both faces listed separately read as two different tokens; the back is a flip away in the card view, or the `F` key.

## [0.3.8] — 2026-08-27

A Tokens column in the deck builder that works out for itself which tokens a deck needs, plus a filter bar that stops rearranging itself.

### Added
- **The deck builder has a Tokens column.** It sits alongside the card columns and moves and scales with them, showing every token the deck's cards can make — with the token's own art, a tick if you already own one, and a `− n ＋` stepper for how many to bring. Which tokens are needed is worked out from the card data rather than read off the rules text, so a Saga that makes different tokens on different chapters is picked up correctly, and the list follows the deck: swap a card and the tokens change with it. Only the counts are remembered, so nothing goes stale.
  - Where several sets have the same token, the one shown is a printing you own if you have one, otherwise the newest — so the art matches what you'd actually put on the table.
  - It knows *which* tokens, not how many. "Create two 2/1 Villains" reads as one kind of token, which is why the count is yours to set.

### Changed
- **The viewer's filter bar has proper rows.** It was one long line that wrapped wherever the window ran out, so a group of buttons could sit together at one window size and split across two lines at another — the Foil button in particular drifted away from the printing filters it belongs with. There are now three deliberate rows: search and card filters, then colours, then mana cost, value and sorting.

## [0.3.7] — 2026-08-27

A fix for v0.3.6's new filters arriving empty on an existing install, plus a deck-builder toggle for the cards you don't own.

### Added
- **A `Not owned` button in the deck toolbar.** Filters the card columns and the sideboard down to the cards you hold no copy of, in any printing — the things you'd actually have to buy. It carries a live count, greys itself out when nothing is missing, and the column totals follow the filter so they describe what's on screen. Cards you own in a different art than the one pinned stay visible: you own those, and the **🎨 Different printing owned** panel already lists them.

### Fixed
- **v0.3.6's Extended art and Inverted filters matched nothing on an existing install.** The app only replaced its bundled card database when a sentinel column was missing, and that sentinel wasn't updated when v0.3.6 added new columns — so upgrades kept the older database, the new columns were added empty, and the new filters found no cards while their "not" states matched everything. The installer's database is now taken whenever it was built more recently than the installed one, which also brings fresher prices on every upgrade. A database you built yourself that is newer than the bundled one is left alone.
  - If you hit this on v0.3.6, installing v0.3.7 fixes it — no need to delete anything by hand.
- **The reference-database check now verifies the treatment and printed-name columns actually contain data**, not merely that they exist, so an empty column fails the build instead of shipping.

## [0.3.6] — 2026-08-27

Extended-art and inverted printings become filterable, cards are findable by the name printed on them, and three fixes where parts of the viewer disagreed with each other.

### Added
- **Extended art and Inverted are now filter chips**, alongside Borderless and Full art, all four cycling only → not → off. These are four independent flags, not one choice: a card can be several at once — `SPM` Miles Morales #200 is borderless *and* full art *and* inverted. They were deliberately not merged into the existing chips: Full art looks broken in `MSC` only because that set has **zero** full-art cards while `SPM` has 51, and borderless is not the same as inverted (identical 26 in `MSC`, but 46 vs 45 in `SPM`).
- **`reference.db` now stores Scryfall's `frame_effects`**, which is what made extended art invisible — 210 of `MSC`'s 866 printings were indistinguishable from plain ones except by collector number. A reference DB carried over from an older version reports every card as having no treatment until it is rebuilt; installers bundle a freshly built one.
- **The finish chip moved into its own section, left of Mana cost.**

### Fixed
- **Cards can be found by the name printed on them.** Reskinned cards carry two names — `MSC` #279 is `Spark Double` to Scryfall but reads **Loki's Double** on the card — and only the first was stored, so searching what you were holding found nothing. Both names are now searched, in the collection and when browsing. **26 printings in `MSC`** are affected, **649 across Magic** (including the Godzilla cards in `IKO`).
- **The set dropdown no longer says "no matches" beside a card it is showing.** Searching a bare collector number was taught to the grid but not to the set list, which kept looking for a card *named* "279". Both now share one piece of search parsing, so they cannot drift apart again.
- **The − button works on a card added while browsing all cards.** That view sent no per-finish breakdown, so the card panel showed `nonfoil ×0 / foil ×0` for a card in stock and left both − buttons disabled — a copy added by mistake could not be taken off again without switching to the collection view.

## [0.3.5] — 2026-08-26

Owned printings are marked in the art pickers, and a fix for the Non-foil filter hiding almost everything.

### Added
- **The printing pickers show which printings you already own.** Choosing art for a deck card or a wish list card listed every printing identically, so the one already on your shelf looked like all the others. Owned printings now carry the same green border the inventory grid uses, plus an **own ×N** count in the corner and in the tooltip. Ownership is matched on the printing's Scryfall id rather than its name, so flavour-named cards and two-sided `A // B` spellings line up correctly.

### Fixed
- **`Non-foil` no longer hides every card that comes in both finishes.** It was built as a strict negation — *never issued in foil* — so a card issued in both matched neither `Foil` nor `Non-foil`, and **53,245 printings are issued in both**. Searching `kang dy` in `MSC` with `Non-foil` set returned "No cards match" and the set dropdown read "MSC (no matches)", for a card sitting in the collection in non-foil. The two chips now mean *available in this finish* rather than opposites, so a card issued in both appears under both — matching how they already behaved for owned copies in the inventory.

## [0.3.4] — 2026-08-26

The card panel only offers finishes the printing was actually issued in, and the Foil filter stops guessing from price.

### Added
- **A card's ＋ buttons now respect what that printing was issued in.** Scryfall records the finishes a printing exists in, and roughly half of all printings are one finish only — 40,529 non-foil-only, 12,363 foil-only. The panel used to offer a non-foil and a foil row for every card regardless, so a foil-only commander could be recorded as non-foil and vice versa. The unavailable row is now greyed out with its ＋ disabled and a tooltip naming the set and number. A finish you already hold copies in keeps its − even when the printing has no such version, so a copy recorded by a bad scan can still be removed rather than becoming unreachable. Etched appears only when the printing has it or you own one, and a card the reference DB has no row for still offers both rather than blocking entry on missing data.

### Fixed
- **The Foil filter no longer misses cards that have never been priced in foil.** It inferred "comes in foil" from the presence of a foil price, but **5,128 printings are issued in foil with no foil price on record** — nobody is currently selling one — and every one of them was filed under Non-foil. Dragonskull Summit `(MSC) 238` was one. Both the grid and the set counts now read Scryfall's finishes list instead. No card is affected in the other direction: none has a foil price without being issued in foil.

## [0.3.3] — 2026-08-26

Find a card by its collector number in the viewer, and filter for the *absence* of foil / full art / borderless.

### Added
- **Search a set by collector number alone.** With a set chosen, a digit-led query is now read as a collector number instead of a name substring — pick `msc`, type `59`, get Puppet Master, String Puller. Works in both the inventory grid and the browse-all-cards grid. Leading zeros are ignored (`007` finds `7`). Without a set selected it stays a name search, since the same number exists in nearly every set.
- **Foil, Full art and Borderless are tri-state.** Clicking a chip cycles **only these → only those without → off**, so you can ask for non-foil, not-full-art or not-borderless printings. The chip relabels itself (`Foil` ⇄ `Non-foil`) and takes a distinct colour rather than the row growing a second chip for each. In the inventory a card owned in *both* finishes shows under Foil **and** Non-foil — the question there is "do I own a non-foil copy", not "is a foil absent". Browsing all cards there are only printings, so `Non-foil` means never issued in foil.

### Fixed
- **A pasted card id no longer comes back empty when the set dropdown is on a different set.** `Name (SET) 123` searches were ANDed with the dropdown, so `(MSC) 59` with the dropdown on `FIC` asked for `set_code='msc' AND set_code='fic'` and silently returned nothing. The set typed into the query now wins, and the dropdown term is skipped.

## [0.3.2] — 2026-08-26

A queryable index of every deck and wish list (plus `npm run index`), clickable buy-list rows, and two import/layout fixes.

### Added
- **A queryable index of every deck and wish list.** `inventory.db` now carries read-only views — `list_index`, `deck_index`, `wishlist_index`, `deck_contents`, `wishlist_contents` — so tools outside the app (the pricing skills) can find a deck or list by id and read its cards without going through the UI. Views, not tables, so they can't go stale. Printings and prices live in `reference.db` and SQLite won't let a view reach into an attached database, so the views stop at `scryfall_id` and the reader joins the two files (one documented `ATTACH` + `LEFT JOIN`).
- **`npm run index`** — prints that index already joined and priced: every deck and wish list with its id, card count, commander and total value, or one list's cards line by line (`npm run index -- deck 40`), with `--json` for machines. Resolves both database paths the way the app does (Dropbox inventory included) and converts EUR→GBP at the same ECB rate the app's £ figures use, so its numbers match the deck view exactly. Cards with no Cardmarket price are reported as unpriced rather than counted as £0.
- **Deck and wish list tiles show their id** (`#40`), so you can name one to a skill without looking it up in the database.
- **Click a buy-list line to open that card.** Rows in **🛒 Missing singles** (and in **🎨 Different printing owned**) now open the same card modal that clicking the card in the deck does — printing picker, quantity, remove, copy card details — and scroll that card into view behind the modal. Keyboard-reachable: tab to a row, press Enter or Space.

### Fixed
- **Long buy-list lines truncate with an ellipsis instead of scrolling sideways.** The panel no longer grows a horizontal scrollbar; a card name too wide for the column is cut with `…` and the full name stays in the row's tooltip.
- **Pasted Moxfield lists no longer import their own header as cards.** Moxfield's Export text opens with an `About` block — `About`, `Name <deck name>`, sometimes `Description ...` — which the list importer read as three unresolved cards. They sat in the deck and rode out at the top of every export as `1 About` / `1 Name Jump Scare!`. The block is now skipped (it ends at the first blank line or the next section header), and decks imported before the fix have those junk rows removed once, on next launch.

## [0.3.1] — 2026-08-25

Perform a tracked swap for real, plus wish list card flipping, a wish list art picker, an About panel, and ⌘Q on macOS.

### Added
- **Perform a swap for real.** The Swapouts table tracked proposals but couldn't act on one — now each row has a **⇄ Swap in** button for when you've actually bought the replacement: one copy of the original leaves the deck, one of the replacement joins it, the swap row is consumed, and the deck's Obsidian note is rewritten on both sides. Refused (rather than half-applied) if the replacement has no pinned printing or the original has already left the deck.
- **Bigger card art in the Swapouts table** — the thumbnails were postage stamps; a swap plan is reviewed by looking at the cards, so they're now large enough to recognise at a glance.
- **Flip reversible cards on a wish list** — the card preview gains the inventory viewer's flip button (and the `F` shortcut) for two-sided printings. (#3)
- **Change a wish list card's printing / art** — the deck builder's 🎴 picker now lives in the wish list preview too, so you can change your mind about which art you're chasing without removing and re-adding the card. Swapping onto a printing already on the list is refused rather than silently merging. (#4)
- **About** — version, author and licence, on the macOS app menu and under Help on Windows. (#5)

### Fixed
- **⌘Q quits on macOS.** The app built a custom menu with no app-menu role, and that's the menu macOS puts the Quit item (and its ⌘Q accelerator) on — so ⌘Q did nothing. The same omission is why there was no About entry. (#6, #5)

## [0.3.0] — 2026-08-24

Deck Swapouts + Sideboard, deck import from Moxfield and EDHREC links, and two wish list fixes (export quantity, click-to-preview).

### Added
- **Swapouts** — an open deck now has its own "Card Swap" table (renamed by clicking the title) for tracking "what if I swapped this" ideas without touching the real 100. Search a card already in the deck as the original, search-or-paste a card code (`Name (SET) 123`) for the replacement, and it lands as a row showing both cards' art and the replacement's price. Export the replacement cards straight to a wish list (existing or new), a `.txt` file, or the clipboard — same buyable-line shape as everywhere else in the app.
- **Sideboard** — a small panel next to the deck's buy list for tracking extra swap-option cards to bring to an event. Cards are added via the same search picker as the main deck, kept out of the deck's 100/analysis entirely, with a friendly (non-blocking) note past 10 cards.
- **Import decks from Moxfield and EDHREC links**, alongside Archidekt. EDHREC decks are read from the page's own embedded data, no browser needed. Moxfield sits behind bot protection that blocks plain requests and headless browsers alike, so the app briefly opens (and closes) a small Chromium window to read the deck — the same trick a real browser gets away with, no fingerprint spoofing.

### Fixed
- **Wish list exports now include the quantity** — `1 Necropotence (SLC) 1995`, matching every other export/import format in the app (deck lists, `/buy-deck`, Obsidian). Previously the leading `1 ` was missing.
- **Clicking a wish list card now opens a full preview** — art, type line, set/rarity, price (with foil), owned status and a Scryfall link — the same information the inventory and deck builder show for a card, instead of doing nothing.

## [0.2.5] — 2026-08-24

Card-id search in the viewer, cross-platform deck ⇄ Obsidian links, and a mulligan panel that says why it can't load.

### Added
- **Search by card id** — the viewer's search box now understands the `Name (SET) 448` line the copy buttons and exports produce, in both inventory and any-card mode. Paste `Edea, Possessed Sorceress (FIC) 448` (or just `(FIC) 448`) and it finds that exact printing; a trailing `*F*` foil marker is ignored. Plain name search is unchanged.

### Fixed
- **Deck ⇄ Obsidian links survive crossing platforms.** A deck imported from the vault on Windows stored its note path with backslashes in the synced database, so the Mac couldn't resolve it (and vice versa) — the mulligan guide and note write-back silently failed for those decks. Paths are now normalized on read.
- **The mulligan panel says why it can't load.** When a deck has a linked note that can't be read (no vault synced on this machine, note moved or deleted), the panel now shows the reason instead of silently not appearing. A deck whose note simply has no `### ✋ Mulligan guide` section still shows nothing — write one with `/deck-guide`.
- The set filter's multi-word matching actually splits on spaces now — the page script's `\s` regex was losing its backslash inside the template literal, so word splitting silently ran on the letter "s" and only worked by coincidence.

## [0.2.4] — 2026-08-23

Two small correctness fixes for wish lists and the Obsidian deck picker.

### Changed
- **The wishlist *in stock ×N* badge now counts only the exact printing.** A list pins a particular art, so owning the same name in another printing no longer lights a wished-for card up as in stock. (The deck builder's owned logic is unchanged — there, any printing still counts, with the amber different-printing tier on top.)
- **Analysis briefs no longer appear in the 💎 Obsidian deck picker.** Notes under `_analysis-briefs` duplicate the decklists of the real deck notes they analyse, so every deck showed twice.

## [0.2.3] — 2026-08-23

Wish lists, double-faced card flipping, deck ⇄ Obsidian two-way sync, the mulligan guide panel, printing-aware owned logic, copy-card buttons, the _Collection.md bridge, and a batch of viewer filters, sorting, and self-refresh fixes.

### Added
- **Deck ⇄ Obsidian note write-back** — decks imported from a vault note remember their source (`decks.source_note`), and edits in the app mirror back into the note's decklist block: changing a card's printing rewrites just that line's `(SET) num` pin, quantity edits and add/remove update or delete the line (foil markers and `[Category]` tags preserved, new cards appended above any Sideboard section). Toasts confirm "Obsidian note updated too". Only main-board and commander lines mirror; decks imported before this release aren't linked.
- **＋ Add card** — a name-search modal in the deck view (reference DB, stays open across adds) for building a deck without leaving the app.
- **✋ Mulligan guide panel** — decks imported from a vault note show the note's `### ✋ Mulligan guide` section (written by `/deck-guide`) as a collapsible panel, read live from the note on every deck open and again on window focus — edit the guide in Obsidian and the app follows without a re-import.
- **Printing-aware owned logic** — the deck view now distinguishes owning a card's *exact pinned printing* from owning it in any printing: stack dots go green (exact) or amber (name only), the card modal says "(different printing)", and a collapsed **🎨 Different printing owned** panel lists the affected lines. The buy list deliberately stays name-based, so nothing new lands on it. A **🔄 Use the printings I own** button bulk re-pins those lines to the most-owned printing of each name (basics and truly-missing lines skipped) and mirrors every change into the source note. Note it also resets deliberate art pins — re-pick those afterwards.
- **📋 Copy card details** — every card surface (deck card modal, scanner preview, "Just scanned" line, collection table rows, wishlist tiles, and the browser viewer's full-card view) can copy a buyable `Name (SET) num` line to the clipboard.
- **_Collection.md bridge** — two-way sync with the vault's master collection note: a **💎 → _Collection** button in the scanner appends the session's scans under a dated heading (same-day re-exports merge, per-printing quantities summed), and `scripts/import-collection.ts` diff-imports the note into the inventory (adds only the shortfall, never removes; dry-run by default, `--apply` to write).
- **Wish lists** — a new 🦸 *Wish lists* section on the home screen holds named lists of cards to buy, the way the deck builder holds decks. Add cards from **Show Inventory**: open any card and hit **☆ Add to wish list**, then pick an existing list or name a new one on the spot. A list is deliberately plain — no formats, boards, filters or curves, just the cards with their art, what each costs, and an *in stock ×N* badge when the shop turns out to own one already. The same printing can't go on a list twice: a repeat add says *"Already on <list>"* rather than stacking a second copy. Each list exports as buyable lines — `Kaldra Compleat (CMM) 958` — to the clipboard or a `.txt` named after the list, ready for a shop search or `/buy-deck`. Lists live in `inventory.db` beside the decks, so they ride along with the Dropbox backup.
- **Flip double-faced cards** — open a two-sided printing (transform, modal DFC, double-faced token, reversible or art-series card — e.g. *Norman Osborn // Green Goblin*, Marvel's Spider-Man #220) in the inventory viewer and a button directly under the card turns it over with a 3D rotation, landing on the other face. The button is named after the face it will show ("⟳ Green Goblin", then "⟳ Norman Osborn"), `F` does the same from the keyboard, and the magnifier still zooms and pans whichever side is up. The reference DB gained `layout` and `back_image_uri` columns, populated on the next **Refresh card data**; until then the viewer resolves a card's faces from Scryfall the first time it's opened (one small call) and caches the answer, so flipping works without waiting for a rebuild.
- **Import decks from Obsidian** — a 💎 Obsidian button in the deck builder lists every vault note with a `## 📜 Deck List` section (searchable, newest first, with commander shown), parses the decklist code block under that heading, and imports it through the usual review modal. The note's commander (frontmatter `commander:`, or the body line the analysis briefs use) is lifted into a Commander section so the commander comes through set. Vault discovery is automatic (the `obsidianVault` folder in Dropbox), and only files inside the vault are readable through this channel.
- **Card value filter** in the inventory viewer / any-card browser — a min/max band in **£** (€ when offline), matched against the Cardmarket price shown on the tile. Unpriced cards fall outside every band rather than counting as £0.
- **Sort dropdown** — value high→low / low→high, mana cost either way, or name A–Z, on top of any combination of filters. In any-card mode the sort runs in SQL so it orders the whole result set, not just the current page.
- **Full-art filter + badge** — a "Full art" chip restricts the grid to full-art printings (Scryfall's `full_art` flag: full-art basic lands, textless promos, etc.), and full-art tiles carry a ◈ badge in both inventory and any-card mode. The reference DB gained a `full_art` column, populated on the next **Refresh card data** (existing DBs migrate the column in and default it to 0 until refreshed).
- **Borderless filter + badge** — a "Borderless" chip and a ▢ badge, from Scryfall's `border_color`. Separate from full art: a card can be either, both, or neither. Same `borderless` reference-DB column and refresh rule as above.

### Changed
- **Searching the inventory only filters now.** A name search used to fling the full-card view open the moment the results narrowed to a single card name — which fights you while you are still typing, and buries the grid behind a modal you did not ask for. Search filters, clicking a tile opens.
- The `Cost:` filter is now labelled **`Mana cost:`** — it filters by mana value, and the new `Value £:` boxes are the money one.
- **Set filtering ignores punctuation and word order.** It was a literal substring match, so `spiderman` and `spider man` both missed "Marvel's Spider-Man" over a hyphen. Now the query and the set name are compared on letters and digits only, with each word matched independently — `spiderman`, `spider man`, `marvel spider` and the set code `spm` all find it, and `assassins creed` finds "Assassin's Creed".

### Fixed
- **The 🛒 Missing singles panel now tracks inventory live** — every inventory mutation (scan-in, sale, viewer ± adjustment, precon add) broadcasts to all windows, so an open deck re-fetches and ticks cards off the buy list in real time.
- **Decklists with Archidekt category tags now resolve.** Lines like `1 Ashnod's Altar (CMM) 368 *F* [Ramp]` — the format Archidekt exports and the vault deck notes use — left the `[Tag]` glued to the card name, so the set/collector matcher never fired and *every* card imported unresolved (a deck of 100 flagged singles priced at £0.00). The tag is now stripped, and the board-level ones (`[Commander]`, `[Sideboard]`, `[Maybeboard{noDeck}]`) set the card's board. Affects clipboard, file, and Obsidian imports alike.
- An open inventory viewer no longer shows stale totals: the page caches the collection for fast filtering but now polls a cheap change token (`/api/stamp`) and redraws within ~5s of any inventory write — a scan-in, a sale, a ± adjustment in another window, or a Dropbox-synced change from the shop's other machine. It also re-checks the moment the page regains focus.
- Overlapping refreshes can no longer paint out of order: a slow any-card query returning after a newer one used to leave the wrong result set (and the wrong status line) on screen.
- A **"Refresh card data"** now also refreshes the viewer's cached collection. The change token was built from inventory writes alone, so a reference-DB rebuild left the cached cards carrying stale reference-derived fields — most visibly, owned full-art cards vanished under the Full art filter until the page was reloaded by hand.

## [0.2.2] — 2026-08-08

Scan-in workspace overhaul, camera controls, and auto-scan de-duplication. *(Entry backfilled — this release shipped without one; see the v0.2.2 tag for the change.)*

## [0.2.1] — 2026-08-06

Viewer filters, card magnifier & deck cost polish.

### Added
- **Colour + mana-cost filters** in the inventory viewer / any-card browser.
- **Card magnifier** — enlarge any card on hover, in both the deck builder and the inventory viewer.
- **Per-category cost** in each Stacks column header in the deck builder (e.g. `Lands  11 · £17.55`).
- Zoom-In keyboard shortcut bound to `Ctrl+=` and numpad `+`/`-` alongside `Ctrl+Plus`.

### Changed
- Missing-singles list in decks is now printing-aware.
- Colour chips default to "Only these", with strict colourless handling; colour filters never match lands.
- Dropped a redundant viewer header now that the viewer is embedded.

### Fixed
- Focus recovery for dead name inputs in deck modals (three-layer fix).
- Window blur/focus workaround to dislodge a stuck out-of-process-iframe keyboard frame.

### Packaging note
- The v0.2.1 CI run failed on a transient GitHub outage (*"Failed to resolve action download info: Service Unavailable"*), not a code fault. The release installers were built and uploaded manually; the reference DB was built on macOS and copied into `resources/` for the Windows package (SQLite files are cross-platform).

## [0.2.0] — 2026-08-02

Deck builder: build, import, analyse, and export decks against live shop inventory.

### Added
- **Deck builder** — a full Commander deck workspace:
  - Create/import decks: New (blank), Clipboard, File (`.txt`), or Archidekt URL (auto-sets commander + exact printings).
  - Archidekt-style **Stacks view** with hover-fan and a click-a-card modal (big art, set/unset commander, change printing/art, quantity, remove); commander pinned first with eligibility + max-two rules enforced.
  - **Analysis**: colour breakdown, mana curve, and opening-hand draw-odds by card type (exact hypergeometric).
  - **Owned-vs-missing** panel + **Copy buy list**.
  - Printing-specific export to clipboard or `.txt`; excludes art-series and Secret-Lair/promo printings, handles MDFC front faces.
  - Embedded inventory browser with right-click / full-card "Add to deck" hooks feeding decks from stock.
- New launcher art (high-res face tiles).

## [0.1.1] — 2026-08-02

Cloud (Dropbox) inventory storage.

### Added
- The shop's `inventory.db` can live in a Dropbox folder (`<Dropbox>/mtgCardVault`, auto-detected) to back it up and share it between machines, while the large rebuildable `reference.db` stays local. **Settings → Inventory storage** to choose local vs Dropbox and move between them.

### Fixed
- Synced inventory DBs use `journal_mode = DELETE` (never WAL) so Dropbox can't corrupt a SQLite file by syncing `-wal`/`-shm` sidecars out of step. Dropbox "conflicted copy" files are detected and surfaced.

## [0.1.0] — 2026-08-02

First Windows/macOS installer build.

### Added
- First end-to-end CI installer build on `windows-latest` + `macos-latest`: self-contained NSIS `.exe` and macOS `.dmg` with bundled `reference.db` + tessdata, needing zero dev tools on the target machine.
- Steps 1–6 shipped: webcam scan-in with auto-lock loop, corner-only OCR with old-frame resolution, sell/remove mode, the Show-Inventory browser, precon bulk add, and Cardmarket £ pricing throughout.

### Fixed
- CI: granted `permissions: contents: write` (release-attach was 403ing) and set `fail-fast: false` so one OS failing doesn't cancel the other.

[0.2.1]: https://github.com/jflockton/mtg-cardvault/releases/tag/v0.2.1
[0.2.0]: https://github.com/jflockton/mtg-cardvault/releases/tag/v0.2.0
[0.1.1]: https://github.com/jflockton/mtg-cardvault/releases/tag/v0.1.1
[0.1.0]: https://github.com/jflockton/mtg-cardvault/releases/tag/v0.1.0
