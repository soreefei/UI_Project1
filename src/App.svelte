<script>
  import pianoImage from "../outline-of-a-piano-with-a-chair-from-black-lines-isolated-on-a-white-background-front-view-vector-illustration-2R5HHD9.jpg";
  import profileImage from "../professionalPFP.jpg";

  const songs = [
    {
      title: "Clair de lune",
      composer: "Claude Debussy",
      level: "Intermediate",
      pages: 4,
      mood: "Nocturne",
    },
    {
      title: "Gymnopedie No. 1",
      composer: "Erik Satie",
      level: "Beginner",
      pages: 2,
      mood: "Minimal",
    },
    {
      title: "Prelude in C Major",
      composer: "J. S. Bach",
      level: "Intermediate",
      pages: 3,
      mood: "Baroque",
    },
    {
      title: "Arabesque No. 1",
      composer: "Claude Debussy",
      level: "Advanced",
      pages: 5,
      mood: "Impressionist",
    },
  ];

  const scenarios = [
    {
      name: "Evening practice",
      player: "Maya",
      detail: "Quiet room · 7:42 PM",
      tempo: 72,
      progress: 38,
      confidence: 94,
      note: "Relaxed phrasing detected. Keep the left hand light.",
    },
    {
      name: "Lesson mode",
      player: "Noah",
      detail: "Teacher connected · 4:10 PM",
      tempo: 84,
      progress: 64,
      confidence: 88,
      note: "Two measures need another pass before moving on.",
    },
    {
      name: "Performance run",
      player: "Avery",
      detail: "Concert hall · 10:15 AM",
      tempo: 96,
      progress: 82,
      confidence: 97,
      note: "Page turns are locked. Dynamics are performance-ready.",
    },
    {
      name: "Sight reading",
      player: "Jordan",
      detail: "Studio · 2:26 PM",
      tempo: 60,
      progress: 21,
      confidence: 76,
      note: "Tempo is steady. The next page is ready when you are.",
    },
  ];

  const features = [
    {
      number: 1,
      name: "Main music display",
      location: "Music desk / sheet music position",
      description:
        "A wide, glare-controlled display replaces loose sheet music. It shows the score, current page, tempo, and visual feedback while keeping the pianist's sightline centered above the keys.",
    },
    {
      number: 2,
      name: "Phone extension dock",
      location: "Right side of the music desk",
      description:
        "A smaller dock holds the pianist's phone in portrait orientation. The Smart Piano mobile app is dedicated to browsing, searching, and selecting music so the main score stays uncluttered.",
    },
    {
      number: 3,
      name: "Audio sensing system",
      location: "Under the music desk / near the soundboard",
      description:
        "A microphone or pickup listens for played notes and tempo. The system uses that input to estimate score position and decide when an automatic page turn is safe.",
    },
    {
      number: 4,
      name: "Annotation stylus",
      location: "Stored beside the main display",
      description:
        "A pressure-sensitive stylus lets the pianist circle notes, add reminders, and mark passages directly on the digital score without reaching for paper or a separate tablet.",
    },
  ];

  let selectedSong = 0;
  let selectedScenario = 0;
  let currentPage = 1;
  let isPlaying = false;
  let isAutoFlip = true;
  let showInfo = false;
  let showDocs = false;
  let annotationMode = false;
  let selectedFeature = 1;
  /** @type {Record<number, {x: number, y: number}>} */
  let featurePositions = {
    1: { x: 49.5, y: 29 },
    2: { x: 69, y: 54 },
    3: { x: 62, y: 29 },
    4: { x: 30, y: 54 },
  };
  /** @type {string[]} */
  let annotations = [];
  let lastAction = "Ready when you are";

  $: song = songs[selectedSong];
  $: scenario = scenarios[selectedScenario];
  $: progress = Math.min(
    100,
    Math.round(
      ((currentPage - 1) / song.pages) * 100 + scenario.progress / song.pages,
    ),
  );

  /** @param {number} index */
  function selectSong(index) {
    selectedSong = index;
    currentPage = 1;
    isPlaying = false;
    lastAction = `${songs[index].title} loaded on the music display`;
  }

  /** @param {number} index */
  function selectScenario(index) {
    selectedScenario = index;
    lastAction = `${scenarios[index].name} simulation loaded`;
  }

  function togglePlay() {
    isPlaying = !isPlaying;
    lastAction = isPlaying
      ? "Listening for your playing..."
      : "Practice paused";
  }

  /** @param {number} direction */
  function flipPage(direction) {
    currentPage = Math.max(1, Math.min(song.pages, currentPage + direction));
    lastAction = direction > 0 ? "Page turned forward" : "Page turned back";
  }

  function toggleAnnotation() {
    annotationMode = !annotationMode;
    lastAction = annotationMode
      ? "Stylus annotation mode on"
      : "Annotation saved";
  }

  /** @param {string} type */
  function addAnnotation(type) {
    annotations = [...annotations, type];
    lastAction = `${type} mark added to page ${currentPage}`;
  }

  /** @param {number} number */
  function selectFeature(number) {
    selectedFeature = number;
  }
</script>

<svelte:head>
  <title>Smart Piano | Interactive Grand Piano Prototype</title>
  <meta
    name="description"
    content="A smart grand piano interface prototype with synchronized sheet music and companion song picker."
  />
</svelte:head>

<main class="app-shell">
  <header class="topbar">
    <h1 class="topbar-title">Smart Piano</h1>
    <button class="text-button" onclick={() => (showInfo = true)}
      >How it works <span>i</span></button
    >
  </header>

  <section class="portfolio-intro" aria-labelledby="portfolio-title">
    <div class="portfolio-copy">
      <p class="eyebrow">PORTFOLIO / PROJECT 01</p>
      <h2 id="portfolio-title">Evan Soreefan</h2>
      <p>
        Hi, I'm Evan Soreefan, a Computer Science student from the University of
        Cincinnati.
      </p>
    </div>
    <img
      class="profile-image"
      src={profileImage}
      alt="Portrait of Evan Soreefan"
    />
  </section>

  <section class="object-overview" aria-labelledby="overview-title">
    <div class="overview-copy">
      <p class="eyebrow">PHYSICAL OBJECT / FEATURE MAP</p>
      <h1 id="overview-title">Smart Piano</h1>
      <p class="overview-lede">
        A grand piano that keeps the pianist in the music. The physical
        instrument stays familiar while digital tools quietly extend the music
        desk, the soundboard, and the pianist's workflow.
      </p>
      <div class="overview-rule"></div>
      <p class="overview-note">
        Drag each numbered point to the feature's location on the piano. Select
        a feature in the list to read its design description.
      </p>
      <div class="feature-list">
        {#each features as feature}
          <button
            class:selected={selectedFeature === feature.number}
            class="feature-card"
            onclick={() => selectFeature(feature.number)}
          >
            <span class="feature-number">0{feature.number}</span>
            <span class="feature-card-copy"
              ><strong>{feature.name}</strong><small>{feature.location}</small
              ></span
            >
            <span class="feature-card-arrow"
              >{selectedFeature === feature.number ? "—" : "→"}</span
            >
          </button>
        {/each}
      </div>
    </div>
    <div
      class="piano-map"
      aria-label="Front-view grand piano image with feature markers"
    >
      <img
        class="piano-image"
        src={pianoImage}
        alt="Front view of a grand piano"
      />
      {#each features as feature}
        <button
          class:active={selectedFeature === feature.number}
          class="map-point"
          style={`left: ${featurePositions[feature.number].x}%; top: ${featurePositions[feature.number].y}%;`}
          aria-label={`Select ${feature.name}`}
          onclick={() => selectFeature(feature.number)}>{feature.number}</button
        >
      {/each}
      <div class="map-selected">
        <span class="map-selected-number">0{selectedFeature}</span>
        <div>
          <strong>{features[selectedFeature - 1].name}</strong><small
            >{features[selectedFeature - 1].description}</small
          >
        </div>
      </div>
    </div>
  </section>

  <div class="prototype-divider">
    <span>INTERACTIVE PROTOTYPE</span><span
      >Scroll to explore the interface</span
    >
  </div>

  <div class="workspace">
    <section class="device-stage" aria-label="Smart grand piano device">
      <div class="stage-heading">
        <div>
          <p class="eyebrow">DEVICE UI / MAIN DISPLAY</p>
          <h1>Play in the moment.</h1>
        </div>
        <div class="stage-status">
          <span class:active={isPlaying} class="status-ring"></span><span
            >{isPlaying ? "LISTENING" : "STANDBY"}</span
          >
        </div>
      </div>

      <div class="piano-display">
        <div class="display-rail">
          <span class="display-label">{song.title}</span><span
            class="display-page">{currentPage} / {song.pages}</span
          >
        </div>
        <div class="music-toolbar">
          <div class="toolbar-group">
            <button
              class="icon-button"
              aria-label="Previous page"
              onclick={() => flipPage(-1)}
              disabled={currentPage === 1}>‹</button
            ><button
              class="icon-button"
              aria-label="Next page"
              onclick={() => flipPage(1)}
              disabled={currentPage === song.pages}>›</button
            >
          </div>
          <div class="toolbar-center">
            <span class="tempo-pulse" class:playing={isPlaying}
            ></span>{isPlaying ? "Audio sync active" : "Manual page turn"}<span
              class="toolbar-separator"
            ></span><button
              class:enabled={isAutoFlip}
              class="toggle-text"
              onclick={() => (isAutoFlip = !isAutoFlip)}
              >Auto-flip {isAutoFlip ? "on" : "off"}</button
            >
          </div>
          <div class="toolbar-group">
            <button
              class:enabled={annotationMode}
              class="tool-button"
              onclick={toggleAnnotation}
              ><span class="pen-icon">✎</span> Annotate</button
            ><button
              class="tool-button"
              onclick={() => (lastAction = "Display brightness adjusted")}
              >☼</button
            >
          </div>
        </div>
        <div class="score-paper" class:annotation-mode={annotationMode}>
          <div class="paper-header">
            <span>{song.composer.toUpperCase()}</span><span class="paper-title"
              >{song.title}</span
            ><span>p. {currentPage}</span>
          </div>
          <div class="score-subtitle">
            {song.mood} · {song.level} arrangement
          </div>
          <div class="staff-block">
            <div class="clef">𝄞</div>
            {#each [0, 1, 2, 3, 4] as line}
              <div class="staff" style={`top: ${40 + line * 14}px`}></div>
            {/each}
            <div class="notes note-a">♪ ♫ ♩</div>
            <div class="notes note-b">♩ ♩ ♪ ♫</div>
            <div class="notes note-c">♫ ♩ ♪</div>
            <div class="measure measure-one"></div>
            <div class="measure measure-two"></div>
            <div class="measure measure-three"></div>
          </div>
          <div class="lyric-line">
            The score follows your hands. Your attention stays with the music.
          </div>
          {#if annotations.length > 0}<div class="annotation-pins">
              {#each annotations as mark}<span
                  >{mark === "Circle" ? "◯" : mark === "Star" ? "★" : "—"}</span
                >{/each}
            </div>{/if}
          {#if annotationMode}<div class="annotation-tip">
              Stylus ready · tap a mark below to place it
            </div>{/if}
          <div class="paper-footer">
            <span>SMART PIANO DIGITAL EDITION</span><span>4 / 4</span>
          </div>
        </div>
        <div class="display-footer">
          <div>
            <span class="footer-label">SYNC CONFIDENCE</span><strong
              >{scenario.confidence}%</strong
            >
          </div>
          <div class="progress-track">
            <span style={`width: ${progress}%`}></span>
          </div>
          <div class="page-gesture">Swipe or use arrows to turn</div>
        </div>
      </div>
    </section>

    <aside
      class="control-deck phone-extension"
      aria-label="Testing UI and companion song picker"
    >
      <div class="phone-screen">
        <div class="deck-header">
          <div>
            <p class="eyebrow">PHONE EXTENSION / TESTING UI</p>
            <h2>Smart Piano app</h2>
          </div>
          <span class="device-chip">CONNECTED</span>
        </div>
        <p class="deck-description">
          Place your phone in the dock and choose a piece here. Your selection
          loads onto the main music display instantly.
        </p>
        <div class="song-list">
          {#each songs as item, index}<button
              class:selected={selectedSong === index}
              class="song-item"
              onclick={() => selectSong(index)}
              ><span class="song-number">0{index + 1}</span><span
                class="song-copy"
                ><strong>{item.title}</strong><small
                  >{item.composer} · {item.level}</small
                ></span
              ><span class="song-arrow">→</span></button
            >{/each}
        </div>
        <div class="deck-section">
          <div class="section-label">
            <span>PLAYBACK SIMULATOR</span><span class="mini-status"
              >{lastAction}</span
            >
          </div>
          <button
            class:playing={isPlaying}
            class="play-control"
            onclick={togglePlay}
            ><span class="play-symbol">{isPlaying ? "Ⅱ" : "▶"}</span><span
              ><strong
                >{isPlaying ? "Pause listening" : "Simulate playing"}</strong
              ><small
                >{isPlaying
                  ? "Audio notes are moving the score"
                  : "Test audio-synchronized page turns"}</small
              ></span
            ><span class="control-chevron">{isPlaying ? "■" : "01"}</span
            ></button
          >
        </div>
        <div class="deck-section">
          <div class="section-label">
            <span>ANNOTATION TOOLS</span><span>{annotations.length} marks</span>
          </div>
          <div class="annotation-tools">
            <button class:active={annotationMode} onclick={toggleAnnotation}
              >✎ <span>Stylus</span></button
            ><button onclick={() => addAnnotation("Circle")}
              >◯ <span>Circle</span></button
            ><button onclick={() => addAnnotation("Star")}
              >★ <span>Star</span></button
            ><button onclick={() => addAnnotation("Line")}
              >— <span>Line</span></button
            >
          </div>
        </div>
        <div class="deck-section scenarios">
          <div class="section-label">
            <span>USER SCENARIOS</span><span>OPTION 3 / DATA</span>
          </div>
          <div class="scenario-grid">
            {#each scenarios as item, index}<button
                class:selected={selectedScenario === index}
                onclick={() => selectScenario(index)}
                ><span class="scenario-avatar">{item.player[0]}</span><span
                  ><strong>{item.name}</strong><small
                    >{item.player} · {item.tempo} BPM</small
                  ></span
                ></button
              >{/each}
          </div>
        </div>
        <div class="scenario-readout">
          <div class="readout-top">
            <span>NOW PLAYING AS</span><strong>{scenario.player}</strong><span
              class="bpm">{scenario.tempo} <small>BPM</small></span
            >
          </div>
          <p>{scenario.note}</p>
        </div>
        <button class="documentation-button" onclick={() => (showDocs = true)}
          ><span>↗</span> View project documentation
          <small>Design notes, requirements & process</small></button
        >
      </div>
    </aside>
  </div>

  <footer class="app-footer">
    <span>PROJECT 01 · INTERFACE TO A SMART OBJECT</span><span
      >EVAN SOREEFAN · 2026</span
    ><span>SMART PIANO / v0.4</span>
  </footer>
</main>

{#if showInfo}<div
    class="modal-backdrop"
    role="presentation"
    onclick={(event) =>
      event.target === event.currentTarget && (showInfo = false)}
  >
    <div
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-labelledby="info-title"
    >
      <button
        class="modal-close"
        aria-label="Close"
        onclick={() => (showInfo = false)}>×</button
      >
      <p class="eyebrow">INTERACTION GUIDE</p>
      <h2 id="info-title">A piano that listens back.</h2>
      <p>
        The main display sits where sheet music normally lives. Choose a song on
        the companion display, then simulate a performance to test
        audio-synchronized page turns.
      </p>
      <div class="guide-row">
        <span>01</span><strong>Pick a piece</strong><small
          >Use the companion display on the right.</small
        >
      </div>
      <div class="guide-row">
        <span>02</span><strong>Start listening</strong><small
          >Smart Piano follows the pianist's notes and advances the score.</small
        >
      </div>
      <div class="guide-row">
        <span>03</span><strong>Mark the score</strong><small
          >Turn on Stylus mode and add circles, stars, or lines.</small
        >
      </div>
    </div>
  </div>{/if}
{#if showDocs}<div
    class="modal-backdrop"
    role="presentation"
    onclick={(event) =>
      event.target === event.currentTarget && (showDocs = false)}
  >
    <div
      class="modal docs-modal"
      role="dialog"
      aria-modal="true"
      aria-labelledby="docs-title"
    >
      <button
        class="modal-close"
        aria-label="Close"
        onclick={() => (showDocs = false)}>×</button
      >
      <p class="eyebrow">PROJECT DOCUMENTATION</p>
      <h2 id="docs-title">Smart Piano / Design notes</h2>
      <p class="docs-lede">
        A smart grand piano designed to make digital sheet music feel as
        immediate as the instrument itself.
      </p>
      <div class="docs-grid">
        <div>
          <span>OBJECT</span><strong>Grand piano</strong>
          <p>
            Large, fixed, acoustic instrument with separate interaction zones:
            music desk and side-mounted picker.
          </p>
        </div>
        <div>
          <span>SMART FEATURES</span><strong>Audio sync + sensing</strong>
          <p>
            Microphones detect played notes; the system estimates position,
            tempo, and practice confidence.
          </p>
        </div>
        <div>
          <span>USER NEEDS</span><strong>Focus and flow</strong>
          <p>
            Keep both hands on the instrument, annotate without paper, and
            recover quickly when lost.
          </p>
        </div>
        <div>
          <span>IMPLEMENTED OPTIONS</span><strong>Complex input + data</strong>
          <p>
            Song selection, stylus marks, playback simulation, four user
            scenarios, and performance readouts.
          </p>
        </div>
      </div>
      <div class="docs-next">
        <span>NEXT STUDY</span>
        <p>
          Interview three pianists, test the page-turn threshold, and connect a
          real audio pitch tracker.
        </p>
      </div>
      <div class="docs-links">
        <a href="https://github.com/" target="_blank" rel="noreferrer"
          >Source code ↗</a
        ><a href="https://example.com/" target="_blank" rel="noreferrer"
          >Portfolio write-up ↗</a
        >
      </div>
    </div>
  </div>{/if}
