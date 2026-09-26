# Galveston 2050 island model: handoff for a new chat

**Start here.** Attach **galveston-2050-source.zip** to the new chat, paste this file, and ask Claude to continue from the source and republish to the same artifact.

* **Live artifact:** https://claude.ai/artifact/16mvQA9QDfR2SajfLtCYq1 (Version 93, Sept 25 2026). To keep the link, always publish with that URL as `url`.
* **Build:** unzip, then run `python3 build.py`. It writes `dist/build-strand-2050-pub.html`; publish that file.
* **Test page:** `python3 tools/mktest.py` writes `/tmp/claude-0/game_t.html`, which exposes `window.__t` (cam, REG, EE, LANES, MOVFS, PARKF, RIDES, RIDEUP, TPT, SEA, setEra, setZone, setExplore, blocked, groundY, summonCar and more).
  * Headless: playwright-core with Chromium at `/opt/pw-browsers/chromium` and the args `--use-gl=swiftshader --enable-unsafe-swiftshader --allow-file-access-from-files`.
  * Block Google Fonts with `page.route`. The page renders at about 1 to 2 fps; give the first screenshot a long timeout.
  * To install the tooling: `npm i playwright-core three@0.128.0` under /tmp/claude-0.
  * A handy view shooter used in Part 4: a JSON list of views {n, t:[x,y,z], d, yaw, p, eval} set on `__t.cam`, one screenshot each.
* **Street View method** (claude-in-chrome, Sky's "Browser 1", deviceId 77c7837c-bf88-4a8f-b1b8-f29d328168d7; each agent in its own tab):
  * `python3 work/geo.py X Z HEADING` gives a pano URL for a model point; navigating straight to it works.
  * Block surveys are in `docs/survey/*.md` (Part 4 added ee-mech-market, ee-market-post, ee-post-church and ee-landmarks).

## Standing rules (from Sky)

* The title is **Galveston 2050**; the eyebrow reads "Island model".
* No dash or hyphen characters in on screen copy or drafted prose.
* Never name the corridor apartment developer.
* Trolley and bus stops are never labeled in 3D. Only Strand Station is labeled.
* No faces on any figure: the explorer, the drag figure, NPCs, and the carriage horse.
* Desktop first, with mobile touch controls.
* Streets south of Church St are visualization only, except the built out row nearest the Seawall from 29th to 19th and the 25th St walks.
* Confirm with Sky before starting each task, and republish the artifact between tasks.

## Part 5 (Versions 77 to 93)

* **Seawall real curve** (`SW_NODES`, `swDz`, `swRealZ`, `swAng` in 01-grid.js; survey docs/survey/seawall-curve.md): traced from Google Maps and the transit map. 20.0° 37th to 25th, kink at the Pleasure Pier, 25.2° to 13th, then bending to about 42° at 6th. The local frame stays straight; `swW`/`swL`/`seaZ` include a vertical shear, and **02b-seawall-bend.js** (`SWB`) shears everything under the Seawall roots on the GPU (vertex shader patch incl. shadows), splitting big triangles at the rounded corners. Register new Seawall roots with `SWB.add(root)`.
* **Nothing west of 37th St** (Sky): no boulevard tail, no wall or city west of 37th, a sand bank instead; 39th removed from `SW_K`. East of 19th a plain boulevard (`buildBoulevardTail(XB, 10th, true)`) runs to 10th.
* **Street grid**: 06b-south-grid.js now runs 27th to 10th, Church to the Seawall; 26th St added to STREETS (Mechanic to Church); 27th and 28th and the avenues west to 28th drawn at street level (West Market grid, buildings later). Maps reach the Seawall along the curve; map land runs to 37th.
* **Districts**: Broadway 27th to 10th; UTMB runs south to Broadway from 10th; Seawall 37th to 6th. **Districts and Streets menus start off** (`state.zone=null`).
* **Port**: Wharf Rd runs east to 14th St and on into **Royal Caribbean Way** (`RCWAY`, `RCCIR` in 06-world.js, one traffic lane pair `WHARF+RCWAY`); traced from Sky's Google Earth image onto the model's own shoreline. Terminal 10 rebuilt from the screenshot (angular hall, blue rotunda, plain white roof, no brand mark), Terminal 10 lot and car pier lot via `polyLot`; overflow lot removed; cruise lots 70% full. Port yard split by Wharf Rd; 18th St clear to Wharf Rd.
* **Vehicle ramp**: the Build Your Seawall access ramp (level pad, eased slope) replaces the old one, with matching walk heights.
* **Alleys**: parked cars in and across the mouths of the East End and Strand alleys are dropped (08-instanced-detail.js).
* **2050 25th St bike lane** (12c-bike-25th.js): 6 ft two way on the east curb, Strand to Seawall, planter curb protection, bends 1 ft outward at crossings.
* **2050 extras**: Wilson Park on the Immigration Station grounds (lots and curb cars gone, two fountain plazas, `IMMLOT`); brick sidewalks on all four sides of every block facing Harborside.
* **Signature buildings** (hand modeled from Sky's photos; `sig:true`, listed in `SIGNATURE`; the Street View facade pass must leave them alone). Toolkit in **06f-signature.js**: `sigFaces` (face coordinates u, y, d), `sigText`, `sigReg`, `XSOL` (extra solids; SOLID adds them), `mapLabel:true` names them on the full map.
  * 06f: Hutchings, Sealy and Co.; Galveston Immigration Station (faces south, centered in its block, walkable stair and terrace via `EWALK`); The Grand 1894 Opera House (walk in entrance passage, `nosol` + XSOL); Santa Fe Building.
  * 06g-post-2100.js: MOD Coffeehouse (pergola), the buff brick building and walled patio with turrets, the Maritime Building (Vargas, 2100 Postoffice at 21st, corner tower, L gallery), Old Galveston Square (sign, conservatory), The Tremont House (US and Texas flags, `sgFlag`), Luna Home & Gifts, 2220 Postoffice (from the survey only, no photo yet), 1914 Postoffice townhouses (`SIG1914`). Helpers `sgWin`, `sgMansard`, `sgFlag`.
  * Keep glass slightly proud of the wall face (d > 0) to avoid z fighting.
* **2050 Seawall facade**: paused by Sky; Seawall frontage is the real 2026 row in both eras.

## Part 4 (Versions 65 to 76)

* **Murdoch's hitbox:** one boulevard frame box over the sand; its axis aligned world box is skipped (`nosol` in REG, honored by `SOLID()`), so it no longer blocks Seawall traffic.
* **V summons your ride** (`summonCar`, 25-character-lab.js): your chosen ride appears at the nearest lane and you are at the wheel; V again gets out. The car button beside the minimap does the same.
* **East End Historic District** (`06e-east-end.js`, `EE`): every house is one of a set of instanced kits (InstancedMesh per archetype per material slot, instance colors):
  * Archetypes: shotgun, Gulf Coast cottage, Queen Anne, Eastlake, Italianate (Ashton Villa), Gothic Revival, Richardsonian Romanesque, Greek Revival, Folk Victorian L plan, Craftsman bungalow, small apartments, alley garage. Weights follow the Street View surveys (Queen Anne and Folk Victorian most common).
  * Landmark kits and addresses (list `LM`): Cameron Historic House (1126 Church, 1100 block added), Davidson Penland House (1207 Postoffice), Landes, Reymershoffer, Kruger, Heffron, Lovenberg, Grover, Wilbur Cherry, 1804 Church and others. House numbers are placed as 00 to 30 across a block from the east street (approximate).
  * Darragh Park (15th and Church, gazebo, labeled park), The Cottage (1501 Postoffice), Sunflower Bakery and Cafe (512 14th St), apartments at 1322 and 1324 Postoffice, the 1300 block between Mechanic and Market as parking with the garden house at Market and 13th.
  * Colors weighted to whites and pastels; purple and deep red rare. Alleys are street level cuts through the raised blocks (`ALLEYS` in 06-world.js, honored by `groundY`).
  * `parkway(p,gap)`: curbside grass strip, 5 ft walk, grass to the property line, on every face. Also used on the Strand UTMB corridor blocks.
* **2050 brick walks** along both sides of the Strand from 19th St east through UTMB (`GARB.plan`).
* **One way streets** (`ONEWAY`, 01-grid.js): Postoffice eastbound from 21st, Church westbound, 22nd (Kempner) northbound Broadway to the Strand, Avenue O westbound, Avenue P eastbound. White dashed lane lines and arrows replace the double yellow; traffic lanes match.
* **Transit map v3/v4 data** (`TRS`, `TRS_STOPNAMES` in 27-transit.js): the Heritage Line returns west on Church St and up 20th (tracks in 11-trolleys.js and 27-transit.js); RHC is the Church leg on the maps.
* **Strand Station:** restrooms and the bike parking sign removed; bike racks fill that corner.
* **Rides** (`04b-rides.js`): the 1908 Model T and the horse carriage, blocky like the other vehicles.
  * Rarest vehicles (`RIDE_P`, `rideKind`): about 5 moving and 3 parked Model Ts downtown in each era, a few on the Seawall; carriages today only on Strand district lanes and curbs (about 6), in 2050 anywhere at about 1.5 times that, and on the Seawall.
  * Traffic horses walk: the Fleet draws horse legs as their own hinged instanced parts (`CGLEGS`, `Fleet.set(i,m,gait)`); Seawall carriages keep live legs (`carriageLive`, `legStride`).
  * Pickups: the Model T at Hendley Green, the carriage at Darragh Park (`RIDEUP`, saved per browser as `s2050-rides`). Once found, the Character Lab unlocks their colors (Model T body, top, wheels; carriage body, seats, horse coat) and the "V calls" chooser.
* **Bike jump:** Space hops about 4 ft (`bikeJump`).
* **Trolley stops** (27b-bus-stops.js, `trolleyShelterG` in 17-seawall-trolley.js): a half canopy of the green Strand Station platform at every rail stop inside the model, per era, from the map's stop lists; shelters must sit fully on raised walk. Bus stops fast travel only to bus stops, trolley stops only to trolley stops (`net` on each TPT entry, `BSP.list`).
* **Districts:** new West Market, Lost Bayou and Silk Stocking (map outlines, labels and panels only, not built). East End now Mechanic to Broadway, 19th to 10th; UTMB to 6th St; Seawall zone 61st to 6th. The Seawall moved from the Streets menu to Districts.
* **Seawall west end:** the boulevard runs on past 37th to about 45th as a visual tail with simple frontage (`buildBoulevardTail` in 13-seawall-module.js).

## Open list (Sky's order may change; confirm before each)

1. Task 6: 2050 Seawall setback errors.
2. Task 8: building sides on the numbered streets.
3. Task 9: street lights, lamps, street signs, traffic lights (in 2050 some street signs on building sides, London style).
4. 2050 Seawall facade (paused, needs Sky's direction).
5. West Market district buildings (later; streets are in, 28th to 25th).
6. 2220 Postoffice: rebuild when Sky sends a photo. Two REG entries are named Maritime Building (the Vargas corner and the Market St one); confirm names with Sky.
7. Earlier open items: 2150 Postoffice as a garage north of Santa Fe; confirm the taupe building at Harborside and 21st; no bus shelters south of Church; buildings.csv out of date; name the garden house on the 1300 block of Market; resolve the 1500 block between Market and Postoffice; carriage speed cap; touch button for the bike jump.

## History (older notes; details may be outdated, trust the code)

Paste this file into a new chat and attach **build-strand-2050-source.zip** (the source project: src, data, docs, tools, build.py). If only **build-strand-2050.html** is at hand, it is the full built page and can be split again by its section banners. Ask Claude to keep working from the source and republish to the same artifact.

## What it is

* A playable 3D arcade style model of downtown Galveston for the **Galveston 2050** master plan proposal (Chapters 4 and 5), a companion to **Build Your Seawall** in the same look.
* Live artifact: **claude.ai/artifact/16mvQA9QDfR2SajfLtCYq1** (Version 55). To update it from a new chat, publish with that URL as `url` so the link stays the same.
* Area: 25th St east past 10th St, Galveston Channel south to Church St, plus the **Seawall district** (Seawall Blvd, 37th to 19th St since Version 46) southwest of downtown. The ordinary island grid continues between Church St and the Seawall on the maps.
* Tech: one HTML page, three.js r128 from cdnjs, fonts Rokkitt, Public Sans and IBM Plex Mono (Google Fonts). No other libraries.
* Companion: **Strand 2050 Character Lab**, claude.ai/artifact/45meygPHYSY8aXKP6jorNW (standalone version of the in game Character Lab screen).

## Standing rules (from Sky)

* Trolley stops are never labeled or listed, and district descriptions do not mention trolley stops. Only Strand Station is labeled. (The idle Seawall trolleys are not labeled either; only the HUD prompt names them.)
* No terminating vista label.
* Title is "Build Strand 2050". The corridor is called "Strand UTMB corridor".

## Districts (tabs)

* **The Strand** (historic district): 25th to 19th St, from the alley north of the Strand south to the alley between Mechanic and Market.
* **Postoffice St** (arts and entertainment district): from that Mechanic to Market alley south to Church St, 25th to 19th St. Borders the Strand at the alley.
* **Strand UTMB corridor**: 19th to 12th St, Harborside Dr to Mechanic.
* **East End Historic District** (tab "East End"): 19th to 12th St, Mechanic St to Church St.
* **UTMB** (renamed from UTMB Medical District; tab, panel and full map read just "UTMB"; the 3D label reads "UTMB" with the subtitle "medical district", centered in the district via `ZONES.ut.lab`; the old UTMB landmark label was removed as a duplicate): east of 12th St to the model edge (just past 10th; Sky says it runs to 9th and beyond), south of Harborside Dr (following the curve) to Church St.
* **Harborside Drive**: 25th St to the UTMB curve (no title card in explore mode).
* **Harborside district**: everything north of Harborside Dr from 25th St to Cruise Terminal 10, following the coastline and piers, plus the Harborside strip of the Strand blocks. Pelican Island is excluded.
* **The Seawall** (tab "Seawall"): Seawall Blvd from 37th to 33rd St, from the frontage lots to the sand. Title card "The Seawall" when you arrive.

Clicking a tab frames the district, draws its border in the 3D view as a bold line in the district color with a white edge (drawn over buildings, like the minimap outline), raises a gently pulsing fence along it, tints its ground, dims the other districts' outlines (`ZONES[k].hi`, `hiM`, `lm`), and zooms the minimap onto it; clicking the same tab again closes the panel. The scroll wheel over the minimap zooms it too.

## Today vs Build 2050

* **Today**: Rail Trolley loop (25th, Mechanic, 23rd, Postoffice, 20th, back west on the Strand) plus the 25th St run south. Strand Station is the curbside bus hub on 25th St, with an Island Trolley bus waiting at the curb. Seawall: Build Your Seawall's Today preset (4 lanes and a center turn lane, parallel parking both curbs, 11 ft walks, existing stairwells every fourth slot, low shops and motels behind front lots).
* **Build 2050**: Heritage Line from Strand Station (Strand, 20th, Postoffice east). **Strand Station is inside the ground floor of the Strand parking garage** (25th St face): lit glass front, green canopy with benches, "STRAND STATION" signs on the 25th St face and the face toward the Strand, and two Island Trolley buses parked in painted bays in the curb lane of 25th St. The Heritage Line platform stays on the 25th St median. Houston Line trains run from the rail yard behind the Santa Fe Building.
* **Seawall 2050**: Build Your Seawall's 2050 proposal lanes (trolley in the outer lanes, 20 ft cafe promenade, two way bike lane between planters) with every edge type spread along the whole wall except the existing stairwell and rain garden: grand stair, overlook, umbrellas, ADA ramp, plaza, skatepark, terrace, beach station, food trucks, beach access ramp (36 slots, each type used several times). Galveston Victorian frontage.
* **Signal at 35th St** (Seawall): crosswalks run curb to curb and over the bike lane to the seaside walk; parking and planters stop at the lights; a raised walkway sits between the crosswalks.
* Trolleys run frequently on every line. Rails are bright silver.
* Traffic also runs Postoffice St, Church St and 23rd, 19th, 17th, 15th and 13th St, so the Postoffice district and the East End have cars.
* The top left eyebrow reads "Galveston 2050" (no chapter numbers).
* **Golf carts and open Jeeps**: about 1 in 20 vehicles today, about 1 in 6 in Build 2050, on the Strand streets and on the Seawall (moving and parked). Traffic carts and Jeeps carry two seated riders.
* **Galveston Island Trolley buses**: a modeled rubber tire trolley bus (red lower body, green upper with arched windows, gold pinstripes, white roof with a green clerestory, "GALVESTON ISLAND TROLLEY" lettering, two pane windshield, grille, round headlights, door on the curb side). Eight run in downtown traffic (Harborside, the Strand loop, Market, 25th St, Postoffice both ways) and one runs in each plain Seawall travel lane. You can take one with E.

## Explore mode

* W A S D move, arrows or drag look, Shift runs, scroll zooms, Esc leaves. W A S D also pans the viewer map.
* Both Today and 2050 can be explored: the Today / Build 2050 buttons stay in the bottom left corner while exploring (the layer checkboxes hide). The explore HUD shows the mode, the key guide and the context prompt; the key guide disappears after a minute of exploring. The Character button lives only in the tools bar.
* **E**: get in your car, take a moving car or bus from traffic (the old car stays where you left it), or board a trolley. On a trolley, hold W or S to take the controls; it stays on its rails. E steps off.
* **Houston Line** (Build 2050): at the rail yard platform behind the Santa Fe Building, E rides the train to Houston; a long fade and you come back out on the sidewalk in front of the Santa Fe Building on 25th St (`nearHouston`, `houstonTrip`).
* Your figure at the wheel of a cart or Jeep shows only while you drive (`VEH.driver`); cars you leave behind are empty.
* **Y**: bicycle (the Build Your Seawall bike). W pedal, S brake, A D steer, Shift sprint, Y off.
* **C** or the **Character** button (tools bar, next to Explore and Hide UI, and in the HUD): opens the **Character Lab**, a full screen with its own 3D sidewalk preview, On foot / Bike / Car modes, Idle / Walk, Randomize, Reset, Done, a "Player 1 ready" summary card, and every choice in a side panel. Choices: top (tee, tank top, hoodie, jacket), bottoms (shorts, board shorts, chinos, joggers), high tops color, cap, bike helmet, bike frame, and their own car (sedan, van, pickup, golf cart, Jeep) and color. Curated palettes only. Saved per browser as `s2050-explorer`. Esc or Done closes it.
* **M**: full map popup. The minimap becomes a square local map with street names and the current district at the top. At the Seawall it shows the leaning grid with 37th, 35th and 33rd St.
* **To the Seawall** (Island Transit bus stops, 17-seawall-trolley.js): a bus stop on the south walk of Church St just west of 25th (E: to the 25th St stop at the Seawall). Every other Seawall cross street has a shelter on the north walk with an idle white Island Transit bus at the curb (E: to Strand Station); the 25th St Seawall stop has no bus (E: to 25th and Church). A short fade; your own car comes along; on a bike you keep the bike; no trips on the jetpack.
* In the Seawall district you can walk the boulevard and **walk down every stair, ramp and terrace to the sand and back up**; decks and plazas are walkable, and you can walk the sand under them. You can drive the boulevard, bump into buildings and the Seawall traffic.
* Vehicles pass through each other and never queue; they only yield to you. Anything you hit slides and spins loose (buses too); only trolleys stop you. Parked cars have small hitboxes. District title card when entering a district.
* Labels hide while exploring; a building's name shows only when you are right beside it.

## Modeled detail worth knowing

* Sidewalks and blocks stand 1.5 ft above the street. Parallel parking on both curbs of the numbered streets and on the Strand and Mechanic (cleared where the bus bays are).
* 25th St esplanade median with trees, broken at intersections. Brick sidewalks and alleys in the Strand district; four brick crossings on the Strand (24th, 23rd, Kempner, Moody).
* The Strand ends at 25th St on the Santa Fe Building (123 Rosenberg Ave); rail yard behind it has four tracks.
* Labeled places include Santa Fe Building, Cruise Terminals 25, 16 and 10, Texas Seaport Museum, Elissa, Pier 21, Ocean Star rig, UTMB, One Moody Plaza (23 stories), Water tower, U.S. National Bank Building, Grand 1894 Opera House, MOD Coffeehouse, **The Tremont House** and **Saengerfest Park**. Seawall district: "Seawall Blvd", "37th St", "35th St", "33rd St", district label "The Seawall", water label "Gulf of Mexico".
* **Seawall grid**: the island grid behind the boulevard is turned 18° against it (streets lean east going north), traced from Sky's aerial of 39th to 33rd St, Ave S, Ave S½ and Ave T. Even streets (39th, 37th, 35th, 33rd, 31st) reach the boulevard; the odd ones stop at the frontage alley. The streets run square to the boulevard through the 138 ft frontage band (so the rectangular frontage lots fit cleanly) and take the lean behind it; host helper `swSt(k,t,o)` gives points along them. The signal street has a single crosswalk, on the north walk. Its placement southwest of downtown (SWX = X(35), SWZ = 6500) is approximate and only matters for the maps.
* The Tremont House is drawn as one four story building on Mechanic just west of 23rd (placement from memory, not yet checked against imagery).
* Public parks and plazas (green, counted together): Saengerfest Park (23rd and the Strand), Hendley Green (2028 The Strand), Pier 21 park. Every REG entry of type park is labeled in 3D (green landmark label) and named on the full map automatically.
* MOD Coffeehouse row, north side of Postoffice between Kempner and Moody, under a green metal canopy over the sidewalk. Storefront awnings follow the Seawall style.
* Parking garages count as parking in the stats. Medical Arts Building (302 Moody) is 13 stories per Sky.
* Streets named on maps: 25th St · Rosenberg, 23rd St · Tremont, 22nd St · Kempner, 21st St · Moody.

## Source layout (since Version 45)

* `src/head.html` (styles and HTML shell), `src/js/NN-name.js` (the game in load order: 00 start, 01 grid, 02 renderer, 03 batcher, 04 vehicles, 05 zones, 05a building data (generated), 05b building data hook, 06 world, 07 ships, 08 instanced detail, 09 zone fences, 10 traffic, 11 trolleys, 12 Strand Station, 13 Seawall module, 17 Seawall trolley, 18 camera, 19 zone copy, 20 labels, 21 minimap, 22 UI wiring, 23 explore, 24 explorer, 25 Character Lab, 26 loop), `src/tail.html`.
* `python3 build.py` writes `dist/build-strand-2050.html` and `dist/build-strand-2050-pub.html` (publish this one). The old patch pipeline (patch.py on the v33 file) is retired; edit src directly.
* `data/buildings.csv`: one row per building (335), stable ids from footprint centers (`bidOf`), filled `set_*` columns override the model through `BDATA`/`applyBD` in addB. `tools/export_inventory.js` refreshes the descriptive columns and keeps set_ values. See data/README.md.
* `tools/mktest.py` makes the headless test page (/tmp/claude-0/game_t.html) with `window.__t`.
* `docs/`: build-2050-change-notes.md (plan to build gaps, also in the project), performance-and-engine.md, decisions.md.
* Performance (Sept 2026): about 1,200 draw calls and 2 million triangles per frame in the drone view; fine on desktop. Decision: stay on three.js, upgrade from r128 at the start of phase 3, facade parts instanced, distance detail.
* Standing decisions: desktop first; traffic passes through itself and will stop only at red lights in phase 4 (Seawall traffic to match); phase 3 starts with the most visible areas (Strand, then Postoffice and Mechanic); the Save image button was removed.

## Code map (inside the HTML)

* Coordinates in feet: x east, z south, Harborside centerline z = 0. `X(n)` gives numbered street n's x; `AV` holds avenue centerlines (Harborside, Strand, Mechanic, Market, Postoffice, Church). `harbZ(x)` is the Harborside curve east of 2890. `SWH` is sidewalk height.
* `SWX`, `SWZ`, `SW_ANG`: the Seawall district origin (35th St at the boulevard, top of the seawall) and the grid lean. Anything with z > ZMAX + 200 belongs to the district: `onLand`, `groundY`, `blocked` and `bodies` hand off to `SEA`.
* `SEA` (section "THE SEAWALL DISTRICT"): Build Your Seawall's code carried over inside its own scope, with `SEA.ensure(era)`, `SEA.setEra`, `SEA.step`, `SEA.land`, `SEA.ground`, `SEA.solids`, `SEA.bodies`, `SEA.info(era)` (lots, streets, pieces). Walking heights come from `edgeY` per edge type; `SEA.fy` is set each frame from `EX.gy` so decks over the sand work.
* `TPT` (one entry per bus stop: hot box, `to`, prompt, arrival `out`/`outH`, own car spot `car`; `TPT.station` is the arrival at Strand Station), `TPSOL` (per era solids for shelters and buses), `transitBusG()`, `shelterG()`, `teleport()`, `nearTP()`: the Island Transit bus stops. `SW_BUSK`/`SW_BUSLX` (01-grid.js) place the Seawall buses; the Seawall module keeps parked cars off those curbs. The 2050 Seawall rail (11-trolleys.js) runs from Strand Station down 25th St, around the monument, and turns both ways onto the boulevard trolley lanes.
* Vehicles: `cartG(body,solo)`, `jeepG(body,solo)` (riders unless solo), `busG()` with canvas textures from `busTex`, `VKIND.cart/jeep/bus`; `vKindOf(i,era,parked)` sets the share and the bus slots (`BUSI`). Parked and moving fleets exist once per version (`PARKG`, `PARKF`, `MOVFS`). Parked station buses are in `STBUS`.
* Strand Station: section "2050: STRAND STATION"; it finds the garage in `REG` by name ('Strand parking garage'); `STATION_AT` places its label; `BAYS` clears parked cars.
* Explorer: `LOOK`, `buildHero(p,head)`, `makeBike(col)`, `poseBikeRig`, `makeCar(kind,col)`, `dressHero`, `dressBike`, `dressCar` (seats a driver in carts and Jeeps via `VSEAT`).
* Character Lab: `LAB`, `labInit` (its own renderer and scene on `#labC`), `labDress`, `LAB.frame`, `renderChar`, `setLook`, `openLab`. The main loop skips its own render while the lab is open.
* `ZONES` holds district polygons; `harbWater()` clips the waterfront land for the Harborside district.
* `addB()` builds buildings and fits every footprint inside its block. `ground()` makes lots, lawns, parks and plazas. `REG` is the clickable registry.
* Trolley paths: `TODAY_LOOP`, `TODAY_25`, `HERITAGE`, `HERITAGE_BACK`; cars on them live in `TROLS`.
* Explore code: `EX`, `stepExplore`, `toggleCar`, `toggleBike`, `drawLocal`, `drawBig`, `seaMap`, `seaText`.

## Reference methods

* Google Earth works in Claude's built in browser: `https://earth.google.com/web/@LAT,LON,5a,380d,35y,343.3h,0t,0r` (heading 343.3 lines the grid up with the screen; allow about 20 seconds to load).

## Plan for the next steps (from Sky)

* Step 3: detail and aesthetics of the Strand buildings from Sky's reference photos, Google Earth, Apple Maps and Street View, so buildings look familiar and realistic.
* Step 4: traffic lights (the Seawall already has Build Your Seawall's signal at 35th St).

## Open ideas for next time

* The East End blocks south of Market and east of 19th are still plain placeholder blocks; step 3 should fill them with homes.
* Seawall traffic still follows and stops at its signal (its own Build Your Seawall logic).

* Check the Tremont House footprint and the MOD Coffeehouse building against imagery.
* Retrace Mechanic to Market east of 19th and the port side from Google Earth.
* More Build 2050 tools; Terminal 16 garage once located.
* Seawall traffic does not yet yield to the player; closed cars do not show the driver.

## Version 46 changes

* **Seawall extended to 19th St.** Local range `SW_XA=-1100` to `SW_XB=6700` (01-grid.js); cross streets reaching the boulevard are `SW_K` (every odd street, 39th to 19th), named by `SW_NAME(k)`. Slots, lots, city grid, jetties, sand, water, traffic, crowds, labels, zone polygon and maps all follow the new range; the existing frontage and edge pieces repeat along it. Density scales with `DEN`.
* **Seawall cyclists have no hitbox** (`SEA.bodies` skips bikes).
* **Parking garages** (`garageB` in 06-world.js): open decks with spandrels, columns, a dark core, cars on every level, a stair tower, a blue P sign and a gate arm, in both eras. **Build 2050** adds a brick facade screen with open arches on the upper levels, a cornice ring and a PARKING sign, so they still read as garages. Era pieces go to `GARB.today` / `GARB.plan`, built into `eraG` in 11-trolleys.js. Never put a solid box over a garage footprint (it hides the deck cars).
* **Skybridge piers** stand on the south sidewalk, in the strip between the road and the rail, north of the rail, and in the lot; none in traffic or on the rail.
* **Harborside Dr today**: no buses or golf carts (`kindAt` in 10-traffic.js); Build 2050 allows them.
* **Explore mode tools bar** shows only Exit explore, Character and Hide UI.
* Fleet instanced meshes have `frustumCulled=false`.

## Step 3 (Strand styling) status

* Street View works by reading pano ids from Google Maps and loading `streetviewpixels-pa.clients6.google.com/v1/thumbnail?...&panoid=ID&yaw=Y&pitch=P&thumbfov=F` (negative pitch looks up). Model to lat/lon: anchor Strand and 20th = (0,331) at 29.3080634,-94.7909382, avenues bear 73.3 degrees.
* Sky chose: start on Strand St 25th to 19th, data plus a facade kit (material, window shape, cornice, gallery type, bay width), republish after each block. 2400 block surveyed, not yet written to buildings.csv.

## Version 47 changes

* **Seawall geometry**: the island grid runs straight; the wall angles 18 degrees (checked against Google Maps). The district root is rotated `SW_ANG`; `swW(lx,lz)` / `swL(x,z)` convert local and world; `seaZ(x)` is the wall line. Frontage buildings collide in the local frame (`lsol`), grid houses in the world (`sol`); `SEA.hit` and `SEA.rayNear` do both.
* **25th St** runs from Church St to the Seawall (`Z25S`, `in25`, `AV25` avenues every 328 ft), with its trolley run extended. No houses within about two blocks of the Seawall end.
* **Pleasure Pier** (`buildPier`, `buildPierRides`, `pierWrap`) runs straight on from 25th St, set `PIER_OFF` seaward with a landing deck. Visual only; the base is solid.
* **Seawall signals** at 35th plus `SIGX` (31st, 27th, 25th grand, 23rd, 19th); traffic stops at all of them. Near 25th (and everywhere in 2050) frontage builds to the walk with parking behind and no driveways.
* **Trolley pads** at every Seawall intersection (`TPT.s<k>`); downtown pad goes to 25th.
* **Drop in** (`25b-dropper.js`): drag the figure beside the era buttons onto the map; `diveTo` sweeps the camera down into explore mode.
* **Harborside**: Pier 21 and 22 rebuilt from Sky's Google Earth images, Wharf Rd extended west to 22nd with traffic both ways, the marina walk is walkable (`DECKS`, `deckAt`), Elissa rebuilt and berthed on the Pier 22 finger. The Harborside walk now stops at cross streets with zebra crosswalks.
* Renames: tabs "Art district", "Historic district", "Harborside"; era button "2050 proposed". No curbside parking on the Strand or 25th; knocked loose cars are towed back after a few seconds; trees are kept off streets.
* Still to do: Strand 2400 block survey (step 3), then Saengerfest Park.

## Version 48 changes

* **Menus**: the top bar has two collapsible menus, Districts (The Strand, Art District, Strand UTMB Corridor, Historic District, UTMB, Harborside) and Streets (Harborside Drive, Seawall Blvd, 25th Street, Broadway). `MENUS`, `openMenu` in 22-ui-wiring.js; street zones `r25` and `bw` (street:true) with panels from the plan in 19-zone-copy.js.
* **South street grid** (06b-south-grid.js): 27th to 12th St and the avenues (Winnie through Ave R, table `SAVE`) from Church St to the Seawall, visualization only, no buildings. Broadway (`BROADWAY_Z`) has two roadways and a palm median. Avenue names label on the maps.
* **Seawall**: every intersection copies the 35th St style; palms and signs sit on the property line; 2050 setback 15 ft; the Pleasure Pier is 20% more compact (`PS`), has an approach stair and ramps, and you can walk under it on the sand. Seawall label centered at 23rd.
* **Harborside**: Elissa is rideable (E to board, ship mode); the 21st St lot flicker is fixed.
* **Strand 2400 block** rebuilt from the Street View survey (Hearsay, Shake Shack, the Gothic and stucco fronts on the south side).
* Explorer figure is cuter; the drop in ghost no longer sticks.
* Known leftovers: the Seawall module still draws its grey slab and leaning avenues in the band behind the boulevard where the new grid overlaps; Broadway polygon runs slightly west of XMIN.
* Queued next: jeep grille fix, car spawn button beside the minimap, minimap title Downtown / Seawall, district density charts (multifamily vs single family residents and residents per acre), then Saengerfest Park.

## Versions 49 to 54 changes

* **Part 3 block by block styling is complete** for 25th to 19th St, covering every face on the Strand, Mechanic, Market and Postoffice, plus the north face of Church St. Each block has a Street View survey in docs/survey/<street>-<block>.md with pano ids and uncertainties.
  * Found and fixed along the way:
    * The Galveston Arts Center moved to 2127 Strand at 22nd.
    * The MOD Coffeehouse moved to the Kempner corner of Postoffice.
    * The Grand's Postoffice frontage (`OPERA`) is narrowed to about 120 ft.
    * The Tremont House is one continuous front with a mansard.
    * The corner of the Strand south side at 22nd is now a pocket plaza and lot.
  * New facade options on 'vic' buildings: `mansard` (slate roof with dormers) and `upper` (the kind of upper facade: 'rear' gives square windows, and 'house' and 'deco' are also available).
* **2026 Rail Trolley** follows Sky's transit map: the downtown loop (25th, Mechanic, 23rd, Postoffice, 20th, the Strand), then 25th south to Ave O½, east to 21st, south to the Seawall, west along the boulevard, and back north on 25th (`TODAY_25` in 11-trolleys.js, with `swLn` for the boulevard lane). The 2050 lines are unchanged.
* **New labels:** UTMB hospital tower, Harbor House Hotel, Galveston Arts Center, Hotel Galvez (a new building in the Seawall frame at 21st to 20th), US Customs House (502 20th St) and Katie's Seafood House. The terminating vista ring (`vistaG`) is removed.
* The Seawall module's old slab, leaning avenues and houses are clipped east of 27th (`CITY_UB`).
* **Other additions:**
  * District panels show a density chart (`DENSITY`, `densityHTML` in 19-zone-copy.js), with estimates and sources in comments.
  * The car spawn button left of the minimap is `spawnOwnCar`.
  * The minimap title reads Downtown or Seawall.
  * The jeep has a Wrangler style grille.
* **Mobile:** on touch devices the explore mode joystick sits bottom left, and bike and enter/exit buttons sit above the minimap. The key guide and touch key row are hidden.
* **Faces removed:** the explorer and the drag figure have no face, and this is a standing rule from Sky: no faces on figures.
* Next: Saengerfest Park.

## Version 55 changes

* One Moody Plaza (the American National Insurance tower) now stands on its ground floor colonnade: tall white tapered columns around a recessed glass lobby under a transfer band. The tower box starts above it.
* Removed a fenced lot and its palms that an agent placed inside the 19th St right of way beside One Moody Plaza; its parked cars sat in the street. A PARKED scan of all downtown streets finds no other cars in travel lanes, except where the cruise lots meet the street stubs north of Harborside.
