# Smart Piano

Smart Piano is a Svelte and JavaScript prototype for a smart grand piano. It begins with a front-view physical feature map, then moves into the interactive interface for the digital music desk and phone extension.

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Svelte](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode).

## Need an official Svelte framework?

Check out [SvelteKit](https://github.com/sveltejs/kit#readme), which is also powered by Vite. Deploy anywhere with its serverless-first approach and adapt to various platforms, with out of the box support for TypeScript, SCSS, and Less, and easily-added support for mdsvex, GraphQL, PostCSS, Tailwind CSS, and more.

## Technical considerations

**Why use this over SvelteKit?**

- It brings its own routing solution which might not be preferable for some users.
- It is first and foremost a framework that just happens to use Vite under the hood, not a Vite app.

This template contains as little as possible to get started with Vite + Svelte, while taking into account the developer experience with regards to HMR and intellisense. It demonstrates capabilities on par with the other `create-vite` templates and is a good starting point for beginners dipping their toes into a Vite + Svelte project.

Should you later need the extended capabilities and extensibility provided by SvelteKit, the template has been structured similarly to SvelteKit so that it is easy to migrate.

**Why include `.vscode/extensions.json`?**

Other templates indirectly recommend extensions via the README, but this file allows VS Code to prompt the user to install the recommended extension upon opening the project.

**Why enable `checkJs` in the JS template?**

It is likely that most cases of changing variable types in runtime are likely to be accidental, rather than deliberate. This provides advanced typechecking out of the box. Should you like to take advantage of the dynamically-typed nature of JavaScript, it is trivial to change the configuration.

**Why is HMR not preserving my local component state?**

HMR state preservation comes with a number of gotchas! It has been disabled by default in both `svelte-hmr` and `@sveltejs/vite-plugin-svelte` due to its often surprising behavior. You can read the details [here](https://github.com/sveltejs/svelte-hmr/tree/master/packages/svelte-hmr#preservation-of-local-state).

If you have state that's important to retain within a component, consider creating an external store which would not be replaced by HMR.

````js
# Resonance: Smart Grand Piano

Resonance is a Svelte and JavaScript prototype for a smart grand piano. It treats the piano as a large, fixed physical object with two interface zones: a main digital music desk in place of paper sheet music, and a smaller side display for browsing songs.

## Interface

- **Main display:** Shows the score, page count, sync confidence, and manual page controls.
- **Audio-sync simulation:** The playback control represents note sensing through a microphone or pickup. The prototype exposes the listening state and score progress without requiring audio hardware.
- **Auto-flip:** Represents the system choosing when a page turn is safe based on the played notes.
- **Stylus annotation:** Stylus, circle, star, and line controls add visible marks to the score.
- **Companion song picker:** Four pieces can be selected on the smaller display and loaded into the main music desk.
- **User scenarios:** Evening practice, lesson mode, performance run, and sight reading show different tempos, progress, confidence, and coaching feedback.
- **Project information:** The info button explains the controls. The documentation button contains the object assumptions, user needs, smart features, implemented options, and future work.

## Design requirements

The design helps pianists stay focused on playing, remove paper from the music desk, annotate without interrupting practice, and recover quickly from a missed passage. The physical integration is intentionally split across the music desk and the side of the piano rather than presenting one flat generic screen.

The envisioned sensing features are feasible assumptions: a microphone or pickup detects notes and tempo, the interface estimates score position, and practice analytics summarize confidence and progress. The current app simulates those behaviors.

## Run locally

```bash
## Physical feature map

The opening section is designed to sit beside a front-view photograph or hybrid sketch of the piano. Its numbered points describe the features to color-code in the final image:

1. **Main music display:** Replaces loose sheet music on the music desk and shows score, page, tempo, and feedback.
2. **Phone extension dock:** Holds the pianist's phone to the right of the main display. The mobile app is dedicated to browsing and selecting songs so the score stays uncluttered.
3. **Audio sensing system:** A microphone or pickup under the music desk or near the soundboard detects notes and tempo for score synchronization.

## Interactive prototype

- **Main display:** Manual page controls, audio-sync state, auto-flip toggle, sync confidence, and digital score.
- **Phone extension:** Four songs can be selected on the smaller portrait-oriented interface and loaded into the main display.
- **Stylus annotation:** Stylus, circle, star, and line controls add visible marks to the score.
- **User scenarios:** Evening practice, lesson mode, performance run, and sight reading show different tempos, progress, confidence, and coaching feedback.
- **Project information:** The info button explains the controls. The documentation button covers assumptions, user needs, smart features, implemented options, and future work.

## Run locally

```bash
npm install
npm run dev
```

Create a production build with `npm run build`.

## AI documentation

AI was used to help scaffold the Svelte interaction model, refine the visual hierarchy, and check the implementation against the project requirements. The interface concept, smart-object choice, interaction goals, and final design decisions belong to the project author.
````
