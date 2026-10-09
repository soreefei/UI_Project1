# Smart Piano

Smart Piano is a Svelte and JavaScript prototype for a smart piano. It begins with a front-view physical feature map, then moves into the interactive interface for the digital music desk, the button hub, and the phone extension.

## Physical feature map

The opening section sits beside a front-view line drawing of the piano. Its numbered marks point to the four additions:

1. **Main music display:** Replaces loose sheet music on the music desk and shows the score, current page, and sync feedback.
2. **Phone extension dock:** Holds the pianist's phone to the right of the main display. The mobile app is dedicated to browsing and selecting music so the score stays uncluttered.
3. **Button hub:** A physical control strip for power, the pull-out coffee table, and the phone connection.
4. **Pull-out coffee table:** A concealed surface that slides out from beneath the keyboard.

## Interactive prototype

- **Main display:** Renders the real sheet music PDFs with manual page controls, audio-sync state, an auto-flip toggle, sync confidence, and stylus annotation with eraser, undo, and redo.
- **Button hub:** Turns the system on and off (which opens and closes the fallboard), pulls out the coffee table, and connects or disconnects the phone.
- **Phone extension:** Search and select a piece to load it onto the main display, control playback and replay, record a take, and add circle, star, and line marks to the score.

The sensing features are feasible assumptions: a microphone or pickup detects notes and tempo, and the interface estimates score position. The current app simulates those behaviors.

## Run locally

```bash
npm install
npm run dev
```

Create a production build with `npm run build`.

## AI documentation

AI was used to help scaffold the Svelte interaction model, refine the visual hierarchy, and check the implementation against the project requirements. The interface concept, smart-object choice, interaction goals, and final design decisions belong to the project author.
