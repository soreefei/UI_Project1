<script>
  import { afterUpdate } from "svelte";
  import * as pdfjsLib from "pdfjs-dist";
  import pdfWorker from "pdfjs-dist/build/pdf.worker.min.mjs?url";
  import pianoImage from "../outline-of-a-piano-with-a-chair-from-black-lines-isolated-on-a-white-background-front-view-vector-illustration-2R5HHD9.jpg";
  import profileImage from "../professionalPFP.jpg";
  import clairDeLunePdf from "../clair-de-lune-claude-debussy.pdf";
  import arabesquePdf from "../arabesque-l-66-no-1-in-e-major.pdf";
  import gymnopediePdf from "../gymnopedie-no1-erik-satie-eric-satie-gymnopedie-nr1.pdf";
  import preludePdf from "../prelude-i-in-c-major-bwv-846-well-tempered-clavier-first-book.pdf";

  pdfjsLib.GlobalWorkerOptions.workerSrc = pdfWorker;

  const songs = [
    {
      title: "Clair de lune",
      composer: "Claude Debussy",
      level: "Intermediate",
      pages: 4,
      mood: "Nocturne",
      pdf: clairDeLunePdf,
    },
    {
      title: "Gymnopedie No. 1",
      composer: "Erik Satie",
      level: "Beginner",
      pages: 2,
      mood: "Minimal",
      pdf: gymnopediePdf,
    },
    {
      title: "Prelude in C Major",
      composer: "J. S. Bach",
      level: "Intermediate",
      pages: 3,
      mood: "Baroque",
      pdf: preludePdf,
    },
    {
      title: "Arabesque No. 1",
      composer: "Claude Debussy",
      level: "Advanced",
      pages: 5,
      mood: "Impressionist",
      pdf: arabesquePdf,
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
      name: "Button hub",
      location: "Control strip below the main display",
      description:
        "A compact physical control hub gives the pianist quick access to page turns, playback, annotation mode, and other frequently used actions without reaching into the digital score.",
    },
    {
      number: 4,
      name: "Pull-out coffee table",
      location: "Lower front panel beneath the keyboard",
      description:
        "A concealed table slides out from beneath the keyboard to hold a coffee, water, or small personal item, giving the pianist a convenient surface without adding permanent bulk to the piano.",
    },
  ];

  let selectedSong = 0;
  let selectedScenario = 0;
  let currentPage = 1;
  let isPlaying = false;
  let isAutoFlip = true;
  let showDocs = false;
  let showFeatureInfo = false;
  let annotationMode = false;
  let isEraser = false;
  let selectedFeature = 1;
  /** @type {HTMLCanvasElement | undefined} */
  let drawingCanvas;
  /** @type {HTMLCanvasElement | undefined} */
  let pdfCanvas;
  /** @type {HTMLDivElement | undefined} */
  let pageLayer;
  let lastRenderedPdf = "";
  let isDrawing = false;
  /** @type {{x: number, y: number}[][]} */
  let strokes = [];
  /** @type {{strokes: {x: number, y: number}[][], annotations: string[]}[]} */
  let undoStack = [];
  /** @type {{strokes: {x: number, y: number}[][], annotations: string[]}[]} */
  let redoStack = [];
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
  $: annotationCount = annotations.length + strokes.length;
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
    strokes = [];
    annotations = [];
    undoStack = [];
    redoStack = [];
    redrawStrokes();
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
    strokes = [];
    undoStack = [];
    redoStack = [];
    redrawStrokes();
    lastAction = direction > 0 ? "Page turned forward" : "Page turned back";
  }

  function toggleAnnotation() {
    annotationMode = !annotationMode;
    if (!annotationMode) isEraser = false;
    lastAction = annotationMode
      ? "Stylus annotation mode on"
      : "Annotation saved";
  }

  /** @param {string} type */
  function addAnnotation(type) {
    saveAnnotationHistory();
    annotations = [...annotations, type];
    lastAction = `${type} mark added to page ${currentPage}`;
  }

  function saveAnnotationHistory() {
    undoStack = [
      ...undoStack,
      {
        strokes: strokes.map((stroke) => stroke.map((point) => ({ ...point }))),
        annotations: [...annotations],
      },
    ];
    redoStack = [];
  }

  /** @param {{strokes: {x: number, y: number}[][], annotations: string[]}} state */
  function restoreAnnotationState(state) {
    strokes = state.strokes.map((stroke) =>
      stroke.map((point) => ({ ...point })),
    );
    annotations = [...state.annotations];
    redrawStrokes();
  }

  function undoAnnotation() {
    if (undoStack.length === 0) return;
    const currentState = {
      strokes: strokes.map((stroke) => stroke.map((point) => ({ ...point }))),
      annotations: [...annotations],
    };
    redoStack = [...redoStack, currentState];
    const previousState = undoStack[undoStack.length - 1];
    undoStack = undoStack.slice(0, -1);
    restoreAnnotationState(previousState);
    lastAction = "Annotation undone";
  }

  function redoAnnotation() {
    if (redoStack.length === 0) return;
    const currentState = {
      strokes: strokes.map((stroke) => stroke.map((point) => ({ ...point }))),
      annotations: [...annotations],
    };
    undoStack = [...undoStack, currentState];
    const nextState = redoStack[redoStack.length - 1];
    redoStack = redoStack.slice(0, -1);
    restoreAnnotationState(nextState);
    lastAction = "Annotation redone";
  }

  function toggleEraser() {
    annotationMode = true;
    isEraser = !isEraser;
    lastAction = isEraser ? "Eraser mode on" : "Drawing mode on";
  }

  function resizeDrawingCanvas() {
    if (!drawingCanvas) return;
    const bounds = drawingCanvas.getBoundingClientRect();
    const ratio = window.devicePixelRatio || 1;
    const width = Math.round(bounds.width * ratio);
    const height = Math.round(bounds.height * ratio);
    if (drawingCanvas.width === width && drawingCanvas.height === height)
      return;
    drawingCanvas.width = width;
    drawingCanvas.height = height;
    redrawStrokes();
  }

  /** @param {PointerEvent} event */
  function getDrawingPoint(event) {
    const canvas = drawingCanvas;
    if (!canvas) return { x: 0, y: 0 };
    const bounds = canvas.getBoundingClientRect();
    return { x: event.clientX - bounds.left, y: event.clientY - bounds.top };
  }

  function redrawStrokes() {
    if (!drawingCanvas) return;
    const context = drawingCanvas.getContext("2d");
    if (!context) return;
    const bounds = drawingCanvas.getBoundingClientRect();
    const ratio = window.devicePixelRatio || 1;
    context.setTransform(ratio, 0, 0, ratio, 0, 0);
    context.clearRect(0, 0, bounds.width, bounds.height);
    context.strokeStyle = "#bd4f43";
    context.lineWidth = 3;
    context.lineCap = "round";
    context.lineJoin = "round";
    for (const stroke of strokes) {
      if (stroke.length < 2) continue;
      context.beginPath();
      context.moveTo(stroke[0].x, stroke[0].y);
      for (const point of stroke.slice(1)) context.lineTo(point.x, point.y);
      context.stroke();
    }
  }

  /** @param {PointerEvent} event */
  function startDrawing(event) {
    if (!annotationMode || !drawingCanvas) return;
    resizeDrawingCanvas();
    saveAnnotationHistory();
    isDrawing = true;
    drawingCanvas.setPointerCapture(event.pointerId);
    if (isEraser) {
      eraseAt(getDrawingPoint(event));
    } else {
      strokes = [...strokes, [getDrawingPoint(event)]];
    }
    event.preventDefault();
  }

  /** @param {PointerEvent} event */
  function draw(event) {
    if (!isDrawing) return;
    if (isEraser) {
      eraseAt(getDrawingPoint(event));
      event.preventDefault();
      return;
    }
    const currentStroke = strokes[strokes.length - 1];
    strokes = [
      ...strokes.slice(0, -1),
      [...currentStroke, getDrawingPoint(event)],
    ];
    redrawStrokes();
    event.preventDefault();
  }

  /** @param {{x: number, y: number}} point */
  function eraseAt(point) {
    const radius = 18;
    strokes = strokes.filter(
      (stroke) =>
        !stroke.some(
          (strokePoint) =>
            Math.hypot(strokePoint.x - point.x, strokePoint.y - point.y) <=
            radius,
        ),
    );
    redrawStrokes();
  }

  function stopDrawing() {
    isDrawing = false;
  }

  /** @param {string} pdfUrl @param {number} pageNumber */
  async function renderPdfPage(pdfUrl, pageNumber) {
    if (!pdfCanvas || !pageLayer) return;
    const pdf = await pdfjsLib.getDocument({ url: pdfUrl }).promise;
    const page = await pdf.getPage(pageNumber);
    const baseViewport = page.getViewport({ scale: 1 });
    const scale = pageLayer.clientWidth / baseViewport.width;
    const viewport = page.getViewport({ scale });
    const ratio = window.devicePixelRatio || 1;

    pageLayer.style.height = `${viewport.height}px`;
    pdfCanvas.width = Math.round(viewport.width * ratio);
    pdfCanvas.height = Math.round(viewport.height * ratio);
    pdfCanvas.style.width = `${viewport.width}px`;
    pdfCanvas.style.height = `${viewport.height}px`;

    const context = pdfCanvas.getContext("2d");
    if (!context) return;
    await page.render({
      canvas: pdfCanvas,
      canvasContext: context,
      viewport,
      transform: ratio === 1 ? undefined : [ratio, 0, 0, ratio, 0, 0],
    }).promise;
    resizeDrawingCanvas();
  }

  afterUpdate(() => {
    const pdfKey = `${song.pdf}-${currentPage}`;
    if (pdfCanvas && pageLayer && pdfKey !== lastRenderedPdf) {
      lastRenderedPdf = pdfKey;
      renderPdfPage(song.pdf, currentPage);
    }
  });

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
  </header>

  <section class="portfolio-intro" aria-labelledby="portfolio-title">
    <div class="portfolio-copy">
      <p class="eyebrow">PORTFOLIO / PROJECT 01</p>
      <h2 id="portfolio-title">Evan Soreefan</h2>
      <p>
        Hi, I'm Evan Soreefan, a fourth-year Computer Science student from the
        University of Cincinnati.
      </p>
      <p class="portfolio-detail">
        Most of my knowledge and expertise involve programming in C/C++ and
        front-end development. I have been a co-op at Bilstein of America and
        Siemens Digital Industries Software (two-time intern).
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
        A piano that keeps the pianist in the music. The physical instrument
        stays familiar while digital tools quietly extend the music desk, the
        soundboard, and the pianist's workflow.
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
      <button
        class="map-selected"
        aria-label={`Open details for ${features[selectedFeature - 1].name}`}
        onclick={() => (showFeatureInfo = true)}
      >
        <span class="map-selected-number">0{selectedFeature}</span>
        <div>
          <strong>{features[selectedFeature - 1].name}</strong><small
            >{features[selectedFeature - 1].description}</small
          >
        </div>
      </button>
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
              class:enabled={isEraser}
              class="tool-button"
              aria-label="Toggle eraser"
              title="Toggle eraser"
              onclick={toggleEraser}>⌫</button
            ><button
              class="tool-button"
              aria-label="Undo annotation"
              title="Undo annotation"
              disabled={undoStack.length === 0}
              onclick={undoAnnotation}>↶</button
            ><button
              class="tool-button"
              aria-label="Redo annotation"
              title="Redo annotation"
              disabled={redoStack.length === 0}
              onclick={redoAnnotation}>↷</button
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
          <div class="sheet-layer">
            <div class="pdf-page-layer" bind:this={pageLayer}>
              <canvas
                bind:this={pdfCanvas}
                class="pdf-render"
                aria-label={`${song.title} sheet music, page ${currentPage}`}
              ></canvas>
              <canvas
                bind:this={drawingCanvas}
                class:active={annotationMode}
                class="drawing-canvas"
                aria-label="Draw annotations over the sheet music"
                onpointerdown={startDrawing}
                onpointermove={draw}
                onpointerup={stopDrawing}
                onpointercancel={stopDrawing}
              ></canvas>
            </div>
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
            <span>ANNOTATION TOOLS</span><span>{annotationCount} marks</span>
          </div>
          <div class="annotation-tools">
            <button class:active={annotationMode} onclick={toggleAnnotation}
              >✎ <span>Stylus</span></button
            ><button class:active={isEraser} onclick={toggleEraser}
              >⌫ <span>Eraser</span></button
            ><button onclick={() => addAnnotation("Circle")}
              >◯ <span>Circle</span></button
            ><button onclick={() => addAnnotation("Star")}
              >★ <span>Star</span></button
            ><button onclick={() => addAnnotation("Line")}
              >— <span>Line</span></button
            >
          </div>
          <div class="annotation-history">
            <button disabled={undoStack.length === 0} onclick={undoAnnotation}
              >↶ Undo</button
            >
            <button disabled={redoStack.length === 0} onclick={redoAnnotation}
              >Redo ↷</button
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

{#if showFeatureInfo}<div
    class="modal-backdrop"
    role="presentation"
    onclick={(event) =>
      event.target === event.currentTarget && (showFeatureInfo = false)}
  >
    <div
      class="modal feature-modal"
      role="dialog"
      aria-modal="true"
      aria-labelledby="feature-title"
    >
      <button
        class="modal-close"
        aria-label="Close feature details"
        onclick={() => (showFeatureInfo = false)}>×</button
      >
      <p class="eyebrow">FEATURE 0{selectedFeature}</p>
      <h2 id="feature-title">{features[selectedFeature - 1].name}</h2>
      <p class="feature-location">{features[selectedFeature - 1].location}</p>
      <p class="feature-description">
        {features[selectedFeature - 1].description}
      </p>
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
