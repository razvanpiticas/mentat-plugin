# Drawing a picture

Load this when a step draws a picture, before its first write.

Every picture a step draws goes the same way, whatever it shows: the persona's picture, a napkin
sketch, a use case's visual, a product's diagram, a wireframe of one frame of the drawing, the sketch
of a concept set aside. The step's instructions say what to draw from and in which style; this says
where it goes and how it gets there. Each picture has a **slot**, and the slot sits on one **owner**:

| Slot | Owner, sent as `ownerId` | What it is |
| --- | --- | --- |
| `PersonaPicture` | the end user profile whose persona it is | the persona's picture |
| `IdeaSketch` | the idea | the sketch of an idea |
| `ConceptNapkinSketch` | our value proposition | the napkin sketch of the concept chosen |
| `ConceptSetAsideNapkinSketch` | our value proposition, with the concept set aside's `id` as `drawingFrameId` | the napkin sketch of a concept set aside |
| `UseCaseVisual` | our value proposition | a diagram or flowchart of the use case's full life cycle |
| `ProductDiagram` | our value proposition | the diagram of a hardware product |
| `DrawingFrameWireframe` | our value proposition, with the frame as `drawingFrameId` | the wireframe of one frame, part or step of the product's drawing |
| `PromiseNapkinSketch` | our value proposition | the final napkin sketch of the promise |

Only the wireframe names its frame, and the sketch of a concept set aside its concept, each as `drawingFrameId`; every other slot is sent without one. An owner refuses a slot it
does not hold.

**1. The prompt first, always.** Build the one prompt the step's text says, and save it with the
owner's own write before any picture: `write_persona` (`picturePrompt`) for the persona, `add_idea` or
`update_idea` (`picturePrompt`) for an idea, and for our value proposition the write of the part the
picture belongs to, as the step names it: `write_concept` (the sketch's `picturePrompt`, and each concept set aside's, its `id` then read from the concept it answers),
`write_use_case` (`visualPrompt`), `draw_product` (`diagramPrompt`, and each frame's
`wireframePrompt`, the frame's id then read from the drawing it answers) or `write_promise` (the final
napkin sketch's `picturePrompt`). The prompt is what a person or another agent draws from
when the picture is missing or wrong, and without a picture the step's own words carry what it shows.
A real end user gets a picture only when what the person agreed to be kept includes one. Upload after
the owner's last write in the step: a rewrite of the persona takes its picture off, and a rewrite of
an idea or a part with another prompt drops the picture drawn from the old one.

**2. The picture**, which depends on what this agent can do:

OpenCode has no image tool here: write the picture yourself as an SVG, as below.

**With no image tool, an SVG written from the prompt.** The same prompt, drawn in code. The file
opens with `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1024 1024">` — the root is `svg` in
the SVG namespace, or it is not read as an image — with no DOCTYPE, flat shapes and gradients only,
no text, no script, no event handler, no external link or font. Write it to a file named for what it
shows, such as `persona.svg` or `promise-napkin-sketch.svg`.

**3. Send the file to an upload link, never as text.** The picture's bytes never pass through the
conversation: one wrong character copied by hand and the file is not an image, and a photo is
millions of characters.

1. Ask for the link: `request_picture_upload(projectId, canvasVersion, slot, ownerId,
   drawingFrameId, fileName, runId)` — the slot and its owner from the table, `drawingFrameId` only
   for `DrawingFrameWireframe` and `ConceptSetAsideNapkinSketch`, `fileName` the name it is kept under, such as `dana-whitfield.png`;
   the `canvasVersion` the owner's write answered. The answer is `uploadUrl`, `expiresAtUtc` (fifteen
   minutes on) and `maxBytes`. Asking does not move the canvas version.
2. Send the file as the whole body of one PUT, with no token and nothing else:
   `curl -sS -X PUT --data-binary @persona.png "<uploadUrl>"` — the same on Windows, macOS and Linux;
   quote the address. The answer is `{"canvasVersion": N}`: the next write sends `N`.
3. Report where the picture now shows: the Persona section, the idea, or the part of the value
   proposition the slot belongs to.

The link takes one picture, recorded as yours inside this run. A refused PUT leaves it usable until it
expires: fix the file and send it to the same link. A picture sent to a slot that already holds one
replaces it. `remove_picture` with the same slot, owner and frame takes a picture off and keeps its
prompt; take one off only when the step or the person says so.

A refusal says what to change: "The picture is 30.0 MB; the limit is 25 MB.", "A picture is a PNG,
JPEG, WebP or SVG image.", "An SVG picture holds no script, event handler or external link." The
second, on an `.svg`, means the file is not an SVG drawing: its root is not `svg`, or not in the
`http://www.w3.org/2000/svg` namespace, or the XML is malformed or declares a DOCTYPE. A refusal
naming the slot means the owner does not hold it, or the frame is missing or not wanted: send the
owner and the frame the table says. A `CONFLICT` naming the canvas version means the canvas moved
after the link was issued: ask for a new link at the version the refusal names and send the file
again. A link that expired, or whose run ended, is refused the same way: ask for a new one, inside an
open run. The prompt is already saved.
