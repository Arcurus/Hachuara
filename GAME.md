# Game mechanics

Living rules for the atlas conquest game. This file is the current design. Later notes amend it in place.

Status: design, not yet in a repository. Date: 2026-09-28.

## Five names

Five names for the game itself. These are play titles, not labels for the sheet.

1. **Crownfall.** The capital can be taken. The crown moves to the next city, and a broken realm can split.
2. **Holdfast.** You win by holding 60% of the land. Cities and rough ground are where an attack slows down.
3. **Levy.** Civilians become soldiers. A city that fields more of them pays less tax.
4. **Throne Road.** Gold, goods, and people travel the roads. You spend only what has reached the capital.
5. **Twenty Crowns.** You start against twenty other crowns, and every one of them plays by the same rules.

Working title: **Hachuara**. The five names above stay as alternates.

## What the game is

You and twenty AI empires start as small patches on a generated world. The map is the whole world on one flat sheet, drawn in code as a black-and-white atlas: seas, rivers, mountains, roads, cities. The left edge is the same place as the right edge, so the sheet is one continuous world. The game opens with that entire sheet in view. Each realm has one muted color washed over its land. Everything else stays ink.

You click land and your soldiers flow that way. You click water and your warships gather there. Both orders fade. The border advances faster through easy ground and slower where the terrain, the defenses, or another player pushes back. Cities grow people, buy goods, own factories, and send gold onward. The gold you can spend is the gold in your capital. A larger city can become the capital, and a lost capital moves to the next city instead of ending the empire.

You win by holding 60% of the land. A match is meant to finish in about 20–30 minutes. Nuclear missiles are rare, expensive, and answered.

## Match

- One human and twenty AI empires. The AI uses the same rules, the same prices, and the same map. It does not receive extra gold or soldiers.
- Each empire starts on a small random patch, far from the others, on fertile ground, with one capital.
- The world is new every match, from a seed.
- The map is one 2D world. East and west are joined: the left edge is the right edge. A ship can sail off one side and come back on the other.
- Sea tiles do not count toward the 60%. Land does, including mountains and rivers.
- An empire is out only when it has no land left. It can still watch.
- Losing the capital does not end the empire. The biggest remaining city becomes the capital. See Capital.
- Taking a tile does not downgrade the city or building on it. Destruction from the fighting stays. There is no extra level loss on conquest.
- The AI fights whichever border is under pressure. Empires fight each other. They are not aimed only at the human.
- A capital loss can split off up to five new countries. Those countries are further AI empires under the same rules.

## How it looks and how you play

The browser draws the world. The server decides what is true.

The atlas is paper and ink. A firm coastline, fine depth lines in the sea, rivers that get thinner toward the source, hachures on the slopes. Forests, plains, hills, and mountains read as different ink because they also cost different amounts of time to cross.

Owned land is a transparent wash in that realm's color, with an ink border and a name. Combat numbers are blue for your soldiers in a fight and red for the enemy. Those are the only strong colors. A ruined building is drawn broken, in proportion to its destruction. A missile trail is visible at every zoom.

Soldiers are not sprites. You see the border move, arrows along the flow, and the blue and red numbers while a fight is happening. Civilians are the same kind of flow: small marks walking to a city, thicker when a city is pulling hard. Trucks, carts, cargo boats, and warships are drawn. Defenses are ink earthworks on a tile that thicken while an enemy is near and fade after the enemy leaves. A city's supply is a ring that fills and drains. A damaged warship peels off its patrol and sails for a harbor.

The opening camera shows the whole world. Zoom and pan are smooth: wheel, drag, and pinch. Panning past the left or right edge continues onto the other side. The drawing is lines, so it stays sharp.

- Far out: continents, washes, capitals, sea lanes, missile trails, and the largest roads.
- Middle: cities, ordinary roads, fronts, arrows, bridges.
- Close: hachures, fresh tracks, trucks, ships, migrant marks, supply rings, combat numbers, defenses, ruin.

Bigger roads stay visible from farther away. A fresh track is only visible up close. A fully built road is visible with the coasts.

The bar along the bottom shows capital gold, the soldier share, and land percent toward 60%. The build tools are city, civilian factory, military factory, harbor, warship, missile silo, and SAM battery. Two fire buttons sit with them: one for an atomic missile, one for an H2 missile. The H2 button is live only while the player has an H2 missile.

A land click with no tool selected sets the army campaign. A water click sets a warship gather point and does not move the army. Both marks fade out over the same time (start at 60 seconds). A new click of that kind replaces the old mark. Soldiers already in a fight stay in it. A click directly on a bridge sends soldiers to damage that bridge.

A click with a build tool founds a building, or upgrades one of that kind if you click near it. Costs are in Gold. A warship tool orders a ship from a harbor you control. One fire click places a target mark, if a missile of that type is ready, and the closest silo of that type launches.

## World and tiles

The map is the whole world, one 2D sheet. Width runs all the way around, and the left edge meets the right. A first look, with no armies on it yet, is `map.html` in this folder: it opens on the whole sheet, drag moves, the wheel zooms, and Another world draws a new seed. The server builds a grid of 240 by 120 tiles, about 40% land, periodic in x. The client downloads the grid once and draws it at any zoom. The sim grid is not the picture. Coasts, rivers, and roads are strokes, so the player never plays on visible squares. The sim runs at 20 ticks a second. The first frame fits the entire sheet on screen.

Each tile stores:

- Terrain: ocean, coast, plains, forest, hills, mountain, river.
- Owner. Neutral land is unowned and already full of civilians. Those civilians become yours when your border takes the tile.
- Soldiers.
- Civilians.
- Defense value.
- Road level, if a road has grown there. A road across a river is a bridge.

Fertile plains hold and grow more people. Mountains hold few. Rivers and seas block walking soldiers until a bridge exists. A bridge is a land connection for soldiers, trucks, civilians, and gold.

Capitals and other buildings sit on tiles. A building has a type, a level, a size, one owner, and a destruction value. See Buildings and Destruction.

## Soldiers and expansion

Soldiers are a density on tiles. A campaign click is an order, not a path for a single unit. The order is strongest when it is placed and fades with the mark.

Each tick, soldiers drift toward the order. They move faster on roads. At the owned edge that faces the order, they spend themselves to take the next tile. Ground that costs more time receives less of the flow, so the army thickens into plains and passes and barely climbs a ridge. The front grows fingers. It does not grow as a ring.

Claim speed falls as resistance rises:

```
resistance = enemy soldiers + defense value
claim time ~ resistance / soldiers arriving
```

On contact, both sides lose soldiers. The fight also applies destruction to a city or building on the tile. The tile changes owner when the defenders are gone and some attackers remain. Nearby fights collapse into one marker: blue count, red count. The new owner keeps the building's level. Conquest adds no further downgrade.

Water stops the walk. Soldiers gather on the near shore. A crossing starts in whichever place finishes first: building boats on that beach, or walking to a harbor and building them there. A harbor builds transport boats at five times the beach rate. The soldiers load onto a transport. That transport has a size from the number of soldiers aboard. A warship can sink it. Those soldiers are lost.

Troop movement by sea uses the same harbor scaling as ship repair. The transport picks the best harbor from size, distance, and how busy the harbor is:

```
move_mult = min(5, harbor_level / transport_level)
```

A matched harbor moves the transport at the base boat speed. A harbor five or more times the transport's level moves it at 5× that base, and no higher. A harbor half the transport's level moves it at half speed. On a sea lane the same boat is five times faster again.

With no campaign, soldiers sit in the cities and along the border, and they still fight if someone walks in.

## Defense

Every tile has a defense value. It is added to the soldiers when an attacker tries to take the tile.

The floor of that value is the terrain:

| Terrain | Defense floor |
| --- | --- |
| Plains | Low |
| Coast | Low |
| Forest | Medium |
| River | High |
| Hills | High |
| Mountain | Very high |
| City | Higher than the terrain under it |
| Capital | Highest |

A city defends better than the same terrain with no city. The capital defends better than a normal city.

When enemy soldiers are close, the tile starts raising its defense above that floor. The labor is the soldiers on the tile plus 20% of the civilians on the tile. Those civilians are busy and do not join migration while the threat lasts. A city tile builds faster because the garrison and the people are there.

When the enemy leaves, defense falls back toward the terrain floor. A quiet tile does not stay fortified.

Defense is visible: earthworks in ink, stronger as the value climbs, fading as it decays.

## Destruction and repair

Cities, buildings, and bridges have a destruction value from intact (0) to ruined (1).

Production scales with what is left. Ten percent destruction is ten percent less output. The same intact share scales city gold, soldier recruitment, factory output, harbor build and repair, silo loading, and the speed of a bridge.

```
output *= (1 - destruction)
```

Further damage lands harder on an intact target. Incoming damage is multiplied by the intact share, then added:

```
destruction += incoming * (1 - destruction)
destruction = min(1, destruction)
```

A blow that would add 0.30 to a fresh building adds 0.15 when the building is already half ruined. Ruining the last intact part takes much more fighting than the first cracks. A missile's incoming value is large enough that a hit can still push a damaged city close to ruined.

Civilians repair destruction over time. The rate follows how many civilians are at that city or building. Repair lowers destruction. It also shrinks the city or building. Restoring a fully ruined 100-size city all the way to intact leaves a city of 80. Any smaller repair takes the same fifth, in proportion to how much destruction was removed:

```
size *= 1 - (1/5) * (destruction_before - destruction_after)
```

A 100-size city repaired from 40% destroyed to intact becomes size 92. The level follows the size. Warships are repaired at harbors and do not lose size this way.

Combat writes destruction. Conquest does not add another loss of level on top.

## Civilians

People grow on land and in cities. Fertile ground and a well-supplied city grow faster. A damaged city grows and produces with its intact share. Growth slows as a tile fills.

Three movements, all of them walked, never teleported:

1. **Growth drift.** Half of the new people stay where they were born. Half join the drift toward cities.
2. **Standing drift.** Civilians already on the land, and civilians already in a city, slide toward cities that pull harder. A city can pull people out of another city.
3. **Player magnet.** When the player founds or upgrades a city, that city pulls nearby civilians the way a campaign pulls soldiers. They walk in over time. A founding aims to gather about a third of the people within six tiles. An upgrade gathers a smaller share. The pull lasts until that share has arrived.

Pull of a city, highest score wins, and distance reduces it:

```
pull = strength
     * (1 + gold / gold_scale)
     * (1 + civilian_supply)
     * (1 + 0.5 * military_supply)
     / travel_time
```

Strength is the city's civilians. A richer city pulls harder. A better-supplied city pulls harder. Roads and sea lanes shorten travel time, so a connected city pulls from farther away. Size still matters, because strength is in the score.

Busy civilians (the 20% building defenses under threat) do not enter this drift. Civilians who are repairing stay in the city and do that work as they live there.

Arriving people increase the city. The city on the map grows as the number grows.

## Soldiers of a city

A city recruits from its own civilians. Military goods are a supply stock, filled by trucks the same way civilian goods are. They are not a separate soldier packet.

At military supply `S` from 0 to 1:

```
speed     = 1 + 4 * S          // 1× at empty, 5× at full
cost      = 1 / speed          // civilians spent per soldier
cap       = 0.10 + 0.10 * S    // 10% at empty, 20% at full
```

A fully supplied city makes soldiers five times faster and spends one fifth of the civilians per soldier. The civilians leave the city at about the same rate as an empty city. The army fills five times faster, and the cap rises from 10% to 20% of that city's population. Population is the city's civilians plus the city's soldiers.

A half-supplied city is in between: 3× speed, one third of the civilian cost, cap 15%.

Soldiers already above the current cap stay. The city stops making more until the share falls or supply rises. Destruction slows recruitment by the intact share.

The empire bar shows the overall soldier share.

The soldier share also cuts the city's tax. See Gold.

## Gold

The player spends the gold in the capital. If the capital changes, the purse that can be spent changes with it. Carts already on the road retarget to the new capital.

### Prices

Founding a city, factory, harbor, or atomic silo costs 100 gold. That initial cost is a setting.

Upgrading costs a percentage of the initial cost. The percentage is a setting. It starts at 80, so an upgrade costs 80 gold.

```
upgrade_cost = initial_cost * upgrade_cost_percent / 100
```

A SAM battery costs five times the matching silo. At the starting numbers that is 500 to found, and 400 to upgrade while the upgrade percent is 80.

Turning an atomic silo into an H2 silo is a separate purchase, not the normal upgrade. It starts at 1000 gold and is its own setting.

A player-ordered warship costs 40 gold and is built at 5× the harbor's own production speed. Buying a level-1 building from the player costs 80, which is still below the cost of founding one.

### Income of a city

A city's income is gold it earns: its own production, land tax that arrives, the return from a factory it owns, a skim of someone else's shipment, and the price of a building it sold. Production is multiplied by the city's intact share and by its civilian supply. A starved or ruined city earns little.

Land tax from a tile is paid to the closest city and becomes that city's income.

### What a city keeps

Of its income, a city keeps 50% and treats the other 50% as its tax. The soldier share then reduces that tax. The reduction is 2.5% of tax for each 1% of the city's people who are soldiers, and it stops at a 50% reduction.

```
soldier_share = soldiers / (soldiers + civilians)
reduction     = min(0.50, soldier_share * 2.5)
tax           = 0.50 * income * (1 - reduction)
```

| Soldiers | Tax reduction | Tax from 100 income |
| --- | --- | --- |
| 0% | 0 | 50 |
| 10% | 25% | 37.5 |
| 20% or more | 50% | 25 |

The city keeps the rest.

Up to half of that tax can be diverted to the closest missile silo that still needs gold for military goods. The diverted gold walks to the silo. Whatever tax remains walks to the next bigger city, or to the capital when no city is bigger.

If another city lies on the road to that addressee, the in-between city keeps 20% of the shipment and forwards the rest. The 20% stays there. It is not sent on again.

The city the shipment was addressed to counts what arrives as its own income, then keeps and taxes that income the same way.

Worked numbers, no soldiers, no silo:

- A village earns 100. It keeps 50 and sends 50.
- A hamlet sits on that road. The hamlet keeps 10 and forwards 40.
- The next city receives 40 as income.

Same village, 10% soldiers, and a silo that needs funding:

- Tax before the soldier cut is 50. The cut is 25%, so the tax is 37.5. The village keeps 62.5.
- The silo may take half of 37.5, which is 18.75.
- 18.75 walks toward the next city and can be skimmed on the way.

The player sees the carts. A distant city pays the capital late.

### Sale of a player-owned building

When a city buys a building from the player, the price is shipped like other gold, with one special entry: it is delivered first to the biggest city near the building, then that city treats it as income and the normal route carries it on to the capital. The player receives it when the cart arrives, not when the sale happens.

## Goods and supply

Each city has two stocks, civilian supply and military supply, each from empty to full. Max stock equals the city's civilian count. A city of 100 needs 100 goods of a kind to be full.

```
supply = stock / max_stock
```

A factory's level sets its size, 100 per level, and size sets how fast goods are produced. A level-1 factory produces 100 goods a minute, scaled down by its own destruction. Once a truck has left, the factory's level no longer matters. Only the goods on that truck matter.

The truck, or a cargo boat on a sea-lane stretch, picks the city with the best pay for the time spent:

```
score = (1 - supply_of_that_good) / travel_time
```

A far, empty city can beat a near, full one. Sea lanes count as fast time.

On arrival the city buys only what it still needs:

```
accepted = min(goods_on_truck, max_stock * (1 - supply))
stock   += accepted
gold     = gold_per_good * accepted
```

`gold_per_good` starts at 1.5. Goods the city cannot take are not sold. A factory therefore never sells more than the city's max need.

A truck of 10 goods into an empty city of 100 raises supply from 0 to 10% and the city pays 15. A second truck of 10 raises it from 10% to 20% and pays 15 again, because the room is still there. A truck of 100 into a city that is already 90% full sells 10 goods and is paid for 10.

The city pays from its purse. If it cannot pay for all the goods that would fit, the sale shrinks to the gold on hand.

That payment is the factory's income:

- 50% goes back to the city that owns the factory.
- 50% stays on the factory and pays for its upgrades.

If the owner is the buyer, the city pays the bill and receives half back. The factory still banks the other half. A factory feeding only full cities sells little, so it stops climbing.

Decay does not wait on trucks and does not reset when one arrives. The city eats in proportion to its supply. A full city eats a full ration. A city at 10% supply eats 10% of that ration in the same time. With a three-minute half-life, any stock falls by half in three minutes: a full city eats 50 goods out of 100, and a 10% city eats 5, which is a tenth of what the full city ate.

Civilian goods and military goods travel, sell, and decay separately. Civilian supply raises the city's gold production. Military supply raises soldier recruitment, as in Soldiers of a city.

## Buildings

A building has one owner. A factory is owned by one city, or it is owned by the player and by no city. It is never shared.

Founding near an existing building of the same kind upgrades that building. This is true for the player and for a city. The upgrade price is the configured percent of that building's initial cost.

| Building | Role |
| --- | --- |
| City | Holds people, earns gold, recruits soldiers, buys goods, and can own factories, harbors, and SAMs. Has a size and a destruction value. |
| Civilian factory | Produces civilian goods by its size. A truck sells only the goods it is carrying, and only up to the city's need. |
| Military factory | Produces military goods the same way. The buying city turns that supply into faster recruitment. |
| Harbor | Has a level. Builds transport boats at 5× a beach, grows a sea lane, builds warships slowly, and repairs ships that fit its yards. |
| Warship | Has a level, normally the level of the harbor that built it. Patrols, fights, and returns to a harbor when damaged. |
| Missile silo | Builds atomic missiles from its size and its military supply. Can be converted into an H2 silo. |
| SAM battery | Shoots at missiles in its radius. Costs 5× the silo of the same level. |

Player-founded buildings start owned by the player. Cities may buy them.

On conquest the building keeps its level and its destruction. It changes realm with the tile. A factory's city-owner is cleared, and a city of the new realm may buy it.

## What cities do on their own

Cities spend gold. They do not spend the player's capital gold.

A city is short of civilian goods when civilian supply stays low. It is short of military goods when military supply stays low. It is short of a harbor when no usable harbor is close and it needs the sea.

When it is short, and it has gold, it acts in this order:

1. **Buy a building no city owns**, if one of the right kind exists and the city can pay. Player-owned buildings are in this group. This is preferred to founding a new one.
2. **Buy a building from another city**, if that city will sell.
3. **Upgrade** a building of that kind it already owns.
4. **Found** a new one nearby.

A civilian shortage leads to a civilian factory. A military shortage leads to a military factory. A missing harbor leads to a harbor. The city founds on a valid tile it owns: a harbor on coast, factories on land.

A city with gold above its own needs also puts a share into SAMs, by the same four steps: buy one, buy one from another city, upgrade one it supports, or found one nearby. The share starts at 10% of the gold it keeps and is a setting. Cities under nuclear threat spend this before they spend it on a new factory.

### Buying from another city

Another city may buy out a factory, harbor, or SAM. The sale depends on the two purses and on where the building sits:

```
price = level_price
      * (seller.gold / buyer.gold)
      * (distance to buyer / distance to seller)
```

`level_price` starts at that building's found cost. The price has a floor so a very close building is not free.

The seller agrees when the buyer can pay that price. A richer buyer pays less. A building close to the buyer and far from the seller is cheap. A building next to a rich owner is expensive and tends to stay.

If the buyer can afford the agreed price, it buys and does not found a duplicate. Ownership moves immediately. The next income of a factory returns 50% to the new owner.

The seller's purse receives the price. That gold is income of the selling city and follows the normal tax route.

### Buying from the player

No city has to agree. Level 1 sells for 80, under the found cost of 100, so a city that can pay buys instead of founding. Higher levels cost more.

The price walks to the biggest city near the building, then onward to the capital.

### Harbor

The same four steps. If nothing close can be bought or upgraded, the city founds a harbor on a nearby coast it owns, or upgrades a harbor it already owns.

## Roads, trucks, bridges, ships

Buildings grow roads toward each other as soon as they exist. The path searches across slopes and rivers and wanders a little, so it bends. Across a river the work becomes a bridge. A finished bridge is a normal land connection for soldiers and carts.

The moment a road connects, movement on it is twice open ground. Further work raises that toward five times open ground:

```
speed = open_ground * (2 + 3 * completion)
```

`completion` is 0 on a new road and 1 when it is fully built. Top speed is 5×, not higher. This covers soldiers, civilians, trucks, and gold carts.

A bridge uses the same destruction rules. Crossing speed is multiplied by the bridge's intact share. If the player aims a campaign directly at a bridge, soldiers damage it instead of only crossing. Civilians repair it over time, and a full repair from ruin also costs a fifth of the bridge's size.

How far away a road is still drawn follows its completion. Large roads read at the continental zoom. New tracks read up close.

Sea lanes connect a realm's harbors. A new harbor etches a lane to the nearest harbor of the same realm. The lane is a curve across water. A boat on a lane moves five times faster than the same boat off the lane.

### Harbors and warships

Harbors slowly build warships of their own level. The player can pay a harbor to build one too. A paid warship is produced at 5× the harbor's current build speed.

Build and repair use the harbor's military supply. Full supply is 5× the base speed. Empty supply is 1×. Anything between scales on a straight line:

```
supply_speed = 1 + 4 * military_supply
```

| Military supply | Speed |
| --- | --- |
| 0 | 1× |
| 0.25 | 2× |
| 0.5 | 3× |
| 0.75 | 4× |
| 1 | 5× |

```
build  = base * supply_speed * intact
paid   = build * 5
repair = min(5, harbor_level / ship_level) * supply_speed
```

`build` is the harbor building a warship on its own. `paid` is a warship the player orders. `repair` is a damaged ship in the yard. The `min(5, harbor_level / ship_level)` term is only the fit of harbor to ship. A matched pair is 1× from size. A level-1 harbor on a level-2 ship is 0.5×. A level-2 harbor on a level-1 ship is 2×. A level-100 harbor on a level-1 ship stays at 5× from size. Supply multiplies that yard speed, so a matched harbor at full supply repairs at 5×, and a capped huge harbor at full supply repairs at 25× the matched empty yard.

The harbor uses the military supply of its owner city. A player-owned harbor uses the nearest city's military supply. Destruction slows build and repair by the intact share. That share is already inside `build`. Apply it to repair the same way.

A damaged warship leaves its patrol and goes to a harbor with free repair space. Among harbors with space, it picks the best repair rate for the time and the crowding:

```
score = repair / (travel_time * (1 + occupation))
```

Occupation is ships currently in the yard divided by capacity. Capacity starts equal to the harbor's level. If every yard is full, the ship sails to the closest harbor and waits.

Troop transports use this same choice and the same 5× cap when they pick a harbor to move from.

Warships fight enemy warships and loaded transports. The fight shows blue and red numbers. They do not count as soldiers. A water click pulls them toward that point, and the pull fades on the same timer as a land campaign.

## Missiles

Silos build missiles on their own. A larger silo builds faster. Military supply speeds that work with the same formula as a harbor: `1 + 4 * military_supply`, so full supply is 5× and half supply is 3×. The gold for those military goods comes from cities. Each city may send up to half of its tax to the closest silo that still needs it.

An atomic silo builds atomic missiles. An H2 silo builds H2 missiles. Converting a silo to H2 costs the H2 setting (start 1000). H2 missiles are worth more megatons. A level-1 atomic missile starts at 1 megaton. Each further silo level adds 1. An H2 missile of the same level is worth 10 megatons times that, and the 10 is a setting.

The fire button places a target if the player has a missile of that type. The closest silo that holds one fires. H2 has its own button.

### Impact

A missile that gets through uses the same destruction rule as battle damage, with an incoming value set by its megatons. It also kills a large share of the civilians and soldiers in the impact area. The kill share and the radius are highest at the center and scale with megatons. H2 hits harder because its megaton count is higher.

### SAM

A SAM battery fires at missiles inside its radius. The intended match is even: a level-1 SAM, on average, shoots down the missiles of a level-1 silo, and it costs five times as much. Extra SAM levels in range make a shoot-down more likely. A higher missile level makes it less likely.

```
hit_chance = 1 - 0.5 ^ (sam_levels_in_range / missile_level)
```

One equal SAM stops the missile half the time. Two equal SAMs stop it three times out of four. The shot and the intercept are both drawn.

Cities spend a part of their gold on SAMs and on SAM upgrades, by the same buy-then-upgrade-then-found order they use for factories.

### Retaliation

A deliberate nuclear launch is a player fire order or an AI fire order. When one lands on a realm that still has nuclear missiles, that realm fires back once.

The reply is five times the incoming megatons. H2 counts at its higher megaton value. The realm fires from the silos it has until it reaches that total, or until its missiles are gone.

The reply aims at the attacker. It prefers high-impact targets, mainly cities, and scores them by economic value, closeness, and thin SAM cover. A city with loaded silos nearby scores worse, because those silos will fire before they are destroyed. The volley would rather hit a rich city that does not still have a loaded silo beside it.

This reply does not cause another reply.

A silo inside a blast fires the missiles it still holds, and only then takes the destruction. Those last shots are real attacks. They do not start a five-times reply. Deliberate fire is the only launch that does.

Order of one strike:

1. SAMs along the path roll their intercepts.
2. Silos that would be ruined by a missile that got through fire their loads first.
3. Destruction and deaths are applied.
4. If the launch was deliberate, the victim fires one reply of five times the megatons.
5. The reply runs through steps 1–3 and then stops.

## Capital

The capital is the city whose gold the player can spend.

If any city reaches five times the civilian count of the current capital, that city becomes the capital.

If the capital tile is taken:

1. The city keeps its level and its destruction, and belongs to the conqueror.
2. The loser's biggest remaining city becomes the capital.
3. The loser's remaining land is split into pieces separated by sea, or by a river that has no standing bridge. The piece that holds the new capital stays with the loser.
4. The largest other pieces become new countries, at most five. Each takes the cities, people, buildings, and city gold on its piece. Its biggest city is its capital. It plays by these rules as an AI empire.
5. Any further separated pieces stay with the loser, so a sixth country is not created.
6. If the loser has no land left after that, the loser is out and can only watch.

East-west wrap counts as connected. A bridge keeps two banks in the same country.

If the capital is lost and no city remains, the loser keeps any land they still hold and has nothing to spend until they found a city. That city becomes the capital. They are out only when the land itself is gone.

## The AI

Every second, each AI empire may set one campaign, one warship gather, and one build if the capital holds the gold. It campaigns toward the border under the most pressure. It founds a civilian factory when gold is thin, a military factory when soldier caps are stuck low, a harbor when the sea is the block, a SAM when missiles can reach it, and a silo when it is at war and can afford one.

Nuclear reply, city purchase, migration, trucks, repair, and defense all happen inside the shared sim. The AI does not get a private economy. A deliberate AI launch is answered by the same five-times rule as a player's launch.

## End of the match

Check about once a second. If the human's land tiles are at least 60% of all land tiles, the human wins and the sim pauses. If the human has no land left, the human is out and the view remains as a watch.

AI empires leave when they have no land left. A lost capital makes a new capital, and can create up to five new AI countries from the separated pieces.

## Program

- **Server.** C#. One process, one match, one simulation thread at 20 ticks a second. The human and the AI submit the same commands: set land campaign, set warship gather, found, upgrade, order warship, fire atomic, fire H2.
- **Client.** The browser. It draws and sends clicks. It does not decide combat, pay, movement, repair, or retaliation.
- **Link.** SignalR. At the start the client receives the grid. Several times a second it receives a snapshot and moves trucks, boats, and carts between snapshots. Zoom and pan stay on the client.
- **Map size.** 240 × 120 tiles, east-west wrap, about 40% land.

The server is the authority for ownership, people, gold, supply, defense, destruction, and combat.

Configurable settings, starting values in the table below:

- Initial build cost.
- Upgrade cost as a percent of that initial cost.
- SAM cost multiplier over a silo.
- H2 conversion cost.
- H2 megatons per atomic megaton of the same level.
- Retaliation multiple.
- Maximum new countries when a capital falls.
- Size multiple that moves the capital.

## Order to build

1. Generate the world and draw the ink map, with wrap, zoom, and pan.
2. Place the twenty-one starts, paint the washes, and run click-to-expand through empty land. Orders fade.
3. Add combat, defense values, destruction, repair, the red and blue numbers, and the 60% check.
4. Add civilians, the three pulls, tax carts, the soldier tax cut, and capital succession.
5. Add both factories, truck loads, the need cap, supply decay, ownership, and self-funded upgrades.
6. Let cities buy, upgrade, and found factories, harbors, and SAMs on their own.
7. Grow roads, bridges, and trucks. Draw large roads from far away.
8. Add harbors, crossings, sea lanes, ship size, repair yards, and warship orders.
9. Add silo production, both fire buttons, SAM intercepts, and the single nuclear reply.
10. Split separated land into at most five countries when a capital falls.
11. Add the twenty AI commanders.
12. Play a full match and retune claim speed, prices, pull strength, and supply half-life until a good game lands near 25 minutes.

## Numbers to tune

These are the starting values. A playtest is allowed to change them. The rules above are not.

| Constant | Start |
| --- | --- |
| Grid | 240 × 120, wrap X |
| Land share | about 40% |
| Tick | 20 Hz |
| Match length | 20–30 minutes to 60% if the player plays well |
| Start gold at the capital | 100 |
| Initial build cost | 100 |
| Upgrade cost | 80% of that building's initial cost |
| H2 silo conversion | 1000 |
| SAM cost | 5× the silo of the same level |
| Warship order | 40, and 5× the harbor's own build |
| Buy a level-1 building from the player | 80 |
| Factory size | 100 per level |
| Factory output | 100 goods a minute per 100 size, times intact share |
| Gold per good accepted | 1.5 |
| Harbor transport-boat build | 5× a beach |
| Harbor versus ship speed | harbor level / ship level, capped at 5× |
| Harbor supply speed | `1 + 4 * supply`: 1× empty, 3× half, 5× full |
| Paid warship | 5× the harbor's current build speed |
| Repair | size match, capped at 5×, then times supply speed |
| Sea lane | 5× the same boat off the lane |
| Road speed | 2× when it connects, up to 5× when fully built |
| Soldier cap | 10% at no military supply, 20% at full |
| Soldier speed from military supply | 1× up to 5× |
| Civilians per soldier | full cost down to 1/5 at full military supply |
| Tax | 50% of income, then cut by the soldier share |
| Soldier tax cut | 25% off the tax at 10% soldiers, 50% off at 20% |
| Silo funding | up to 50% of the tax, to the closest silo that needs it |
| Income skimmed by a city on the way | 20% of that shipment |
| Factory income to the owner city | 50% |
| Factory income kept for upgrades | 50% |
| Supply decay | half the stock every 3 minutes, at every supply level |
| City SAM share | 10% of kept gold when the city can afford it |
| Civilians gathered by founding a city | about a third within 6 tiles, walked |
| Civilians put to work on defenses | 20% of the tile while enemies are close |
| Full repair from ruin | costs 1/5 of that city or building's size |
| Campaign fade | 60 seconds, land and sea |
| Atomic yield | 1 megaton at silo level 1, plus 1 per extra level |
| H2 yield | 10× the atomic yield of that level |
| Nuclear reply | 5× incoming megatons, once, no chain |
| New capital on growth | a city at 5× the capital's civilians |
| New countries on a lost capital | at most 5 |
| Win | 60% of land tiles |
| Loss | no land left; watch only |

## Worked checks

**Supply.** City of 100, empty. Truck of 10 goods. Supply becomes 10%. The city pays 15. Decay over the next three minutes eats half of whatever stock is there, whether or not another truck is on the road. A city holding 10 goods eats 5. A city holding 100 eats 50. The smaller city eats a tenth of the goods.

**Military supply.** The same truck of military goods raises military supply by the same 10%. Recruitment speed becomes 1.4×, civilian cost becomes about 0.71, and the cap becomes 11%. At full military supply those figures are 5×, 1/5, and 20%.

**Repair.** A size-100 city at 100% destruction produces nothing. Civilians repair it. When destruction reaches 0, the size is 80.

**Tax.** A city earns 100 with 20% of its people soldiers. The tax is 25, not 50. A silo that needs funding can take 12.5 of that. The other 12.5 starts toward the next bigger city.

**Harbor.** `supply_speed = 1 + 4 * supply`. Empty is 1×, half is 3×, full is 5×, for both building ships and repairing them. A level-1 harbor repairs a level-1 ship at that supply speed, and a level-2 ship at half of it. Size match never exceeds 5× before supply is applied. A paid warship is produced at 5× the harbor's current build speed.

**Reply.** One level-1 atomic missile is 1 megaton. The victim fires 5 megatons back, once. One level-1 H2 missile is 10 megatons. The victim fires 50 megatons back, once. Silos in either blast empty themselves first. Those shots do not start a further reply.
