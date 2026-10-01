# ESI Campus Map — context for coding agents

## What this is
A single-page Three.js web app: a stylized "holographic city" 3D map of the ESI campus (Algiers).
Buildings are extruded polygon footprints (any shape: rectangles, L-shapes, etc.) with per-building rotation.
Roads are flat ribbons on the ground. Users search for a building/room, the camera flies to it, and a side
panel lists its rooms. An optional satellite image underlay is available as a calibration aid.
With `?edit=1` in the URL, a level editor allows drawing building footprints and road paths, moving/reshaping
them interactively, and exporting the data as campus.json.

## Files
- index.html   -> the whole app (HTML + CSS + one inline <script type="module">). Keep it that way.
- campus.json  -> campus data (buildings + rooms). Same data is also embedded as CAMPUS_DATA in index.html.
- server.cjs   -> tiny static server. Run: `node server.cjs` then open http://localhost:3000
- assets/      -> put satellite.png here (top-down image of campus, used as an underlay)
- package.json / tsconfig.json / index.ts -> leftover Bun scaffolding, unused. Ignore.

## Rules
- No frameworks, bundlers, or npm dependencies. three.js comes from unpkg via the importmap (three@0.165.0).
- Do not rewrite the whole file. Make small edits in place, and don't touch unrelated code.
- Keep the visual style: black background, cyan wireframe + translucent fill, bloom, floating labels, dot grid.
- Do not break existing features: click-to-select, search dropdown, fly-to, side panel, labels.
- After every change, tell me exactly how to test it in the browser (what to click, what I should see).
- Ask before adding a new library or splitting the code into multiple files.

## Data schema
```json
{
  "buildings": [
    {
      "id": "string", "name": "string",
      "x": 0, "z": 0,           // footprint anchor in world space
      "rotation": 0,             // degrees, rotates footprint around anchor (Y axis)
      "levels": 1,              // number of floors / levels (integer)
      "levelHeight": 3.2,       // height per floor in meters (total height = levels * levelHeight)
      "footprint": [[0,0],[10,0],[10,6],[4,6],[4,14],[0,14]],
                                 // polygon vertices in local 2D (x,z), any simple polygon
      "rooms": [ { "id": "string", "name": "string", "type": "string", "floor": 0, "footprint": [[0,0],[5,0],[5,5],[0,5]] } ]
    }
  ],
  "roads": [
    { "id": "string", "width": 6, "path": [[x,z],[x,z],...] }
  ]
}
```
- x/z = footprint anchor position in world space. y is up.
- rotation = degrees, rotates the entire footprint around the anchor.
- levels / levelHeight = total height is levels * levelHeight.
- footprint = ordered polygon vertices in local 2D space. Can be any simple polygon (rectangle, L-shape, etc.).
- Backward compat: if a building has `width`/`depth` instead of `footprint`, it auto-converts to `[[-w/2,-d/2],[w/2,-d/2],[w/2,d/2],[-w/2,d/2]]`. Old `height` without levels/levelHeight converts to `levels: 1, levelHeight: <height>`.
- rooms[].footprint = (optional) ordered polygon vertices in building's local 2D space, representing the room's physical shape.
- roads = visual-only polylines rendered as flat ribbons on the ground.
- Room types in use: amphitheater, classroom, lab, office, library, study, gym, locker, hallway, other.

## Key names in index.html
CAMPUS_DATA, getBuildingHeight(building), createBuildingMesh(building), createRoadMesh(road), registerBuildingMesh(mesh),
buildingMeshes[], roadMeshes[], polygonCentroid(fp), ensureFootprint(b),
loadCampus(), performSearch(q), selectSearchResult(r), flyToBuilding(b), openPanel(b, room),
onClick / onMouseMove (raycasting), animate(), rebuildCampus(data).
Editor (only with ?edit=1): toggleEditMode(), enterSplitMode(b), exitSplitMode(), generateGrid(), createRoomFromSelection(), autosaveDraft(), exportJSON(), importJSON().
