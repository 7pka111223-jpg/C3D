# MarkPoint and LinkToMarkedPoint: the naming trap and four rules

Civil 3D ships two stock subassemblies that together draw one link across an assembly: `Subassembly.MarkPoint`
names a point, `Subassembly.LinkToMarkedPoint` draws a link from where it is attached to the point with that name.
A channel section uses them to close the bottom between two slopes, or to carry one length label across the top.
`create_assembly` places both (see `skills/civil3dfactory-pkt/SKILL.md`, second pattern, and
`examples/pkt/02-two-design-lines.json`). This page is what is easy to get wrong.

## Two things share the name MarkPoint

- `SetMarkPoint` / `GetMarkPoint` **inside Subassembly Composer** are internal variables of that one PKT.
  Only the subassembly itself can read them.
- `Subassembly.MarkPoint` **in the assembly** is a separate stock part. It names a point that already exists
  in the assembly, so a `LinkToMarkedPoint` somewhere else can find it.
- Same word, two layers, no connection between them. The corridor only ever sees point, link and shape codes.

The fix in one sentence: give the point a point code inside the PKT, attach a stock `MarkPoint` to that coded
point in the assembly, and point a stock `LinkToMarkedPoint` at that name from the other side.

## Four rules

1. **Order is invisible.** An assembly is processed in item order, one subassembly after another.
   The `LinkToMarkedPoint` must come after the `MarkPoint` it names, so attach it to the side that is processed
   later. On the wrong side the link is empty and nothing reports an error: the corridor builds, the receipt
   says ok, the section has no closing link. In `create_assembly` the `items` order is the processing order.
2. **A link is not a label.** The link only gets a label when its code has an entry in the code set style with a
   label style. The demo code set `@C3DF-Dredge` labels the code `bottom` with `@C3DF-Length-Bottom`; that is why
   the closing link in the example carries `SurfaceCodes: "Top,Datum,Bottom,bottom"`. A code with no entry draws
   the link and prints nothing.
3. **One point, one code.** Every code on a marked point that has a label in the code set prints its own label.
   Two codes on the same point print the elevation twice. Keep the marked point on one code.
4. **In the API the stock parameters have no names, only resource ids.** `PointName` is 4205,
   `MarkedPointName` is 3905, `SurfaceCodes` is 3907. `create_assembly` accepts the catalog names and maps them
   from the tool catalog files (`Tool Catalogs\Road Catalog\*.atc`); the result echoes the values under their
   resource ids so you can check what was written.

## Where it is used

- `examples/pkt/02-two-design-lines.json`: right slope + `MarkPoint` on `toe-right` first, then left slope +
  `LinkToMarkedPoint` on `toe-left` pointing at it. The closing link runs left to right like the slopes.
- The same pair across the top of a channel (`daylight-left` to `daylight-right`) gives one top-width label per
  section: mark one daylight point, hook the link on the other side, put the link on a code the code set labels.
