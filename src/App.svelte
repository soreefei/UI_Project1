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

  const searchableScores = [
    {
      title: "Clair de lune",
      composer: "Claude Debussy",
      source: "Your library",
      songIndex: 0,
    },
    {
      title: "Gymnopedie No. 1",
      composer: "Erik Satie",
      source: "Your library",
      songIndex: 1,
    },
    {
      title: "Prelude in C Major",
      composer: "J. S. Bach",
      source: "Your library",
      songIndex: 2,
    },
    {
      title: "Arabesque No. 1",
      composer: "Claude Debussy",
      source: "Your library",
      songIndex: 3,
    },
    {
      title: "Piano Concerto No. 2",
      composer: "Sergei Rachmaninov",
      source: "Online catalog",
    },
    {
      title: "Prelude in E Minor, Op. 28 No. 4",
      composer: "Frederic Chopin",
      source: "Online catalog",
    },
    {
      title: "Liebestraum No. 3",
      composer: "Franz Liszt",
      source: "Online catalog",
    },
    {
      title: "La Campanella",
      composer: "Franz Liszt",
      source: "Online catalog",
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

  // 24x24 stroke icons, drawn as a single path each.
  const icons = {
    prev: "M15 18l-6-6 6-6",
    next: "M9 18l6-6-6-6",
    pen: "M12 20h9M16.5 3.5a2.12 2.12 0 0 1 3 3L7 19l-4 1 1-4z",
    eraser:
      "M7 21l-4.3-4.3a2.4 2.4 0 0 1 0-3.4l9.6-9.6a2.4 2.4 0 0 1 3.4 0l5.6 5.6a2.4 2.4 0 0 1 0 3.4L13 21M22 21H7M5 11l9 9",
    undo: "M9 14L4 9l5-5M20 20v-7a4 4 0 0 0-4-4H4",
    redo: "M15 14l5-5-5-5M4 20v-7a4 4 0 0 1 4-4h12",
    sun: "M12 8a4 4 0 1 0 0 8 4 4 0 0 0 0-8zM12 2v2M12 20v2M4.9 4.9l1.4 1.4M17.7 17.7l1.4 1.4M2 12h2M20 12h2M4.9 19.1l1.4-1.4M17.7 6.3l1.4-1.4",
    power: "M18.36 6.64a9 9 0 1 1-12.73 0M12 2v10",
    cup: "M18 8h1a4 4 0 0 1 0 8h-1M2 8h16v9a4 4 0 0 1-4 4H6a4 4 0 0 1-4-4zM6 1v3M10 1v3M14 1v3",
    bluetooth: "M6.5 6.5l11 11L12 23V1l5.5 5.5-11 11",
    replay: "M1 4v6h6M3.51 15a9 9 0 1 0 2.13-9.36L1 10",
    circle: "M12 4a8 8 0 1 0 0 16 8 8 0 0 0 0-16z",
    star: "M12 3l2.7 5.6 6.1.9-4.4 4.3 1 6.1L12 17l-5.4 2.9 1-6.1L3.2 9.5l6.1-.9z",
    line: "M4 12h16",
    close: "M6 6l12 12M18 6L6 18",
  };

  let selectedSong = 0;
  let currentPage = 1;
  let isPlaying = false;
  let isAutoFlip = true;
  let systemOn = true;
  let fallboardOpen = true;
  let fallboardTransition = "";
  let fallboardTimer;
  let coffeeTableOpen = false;
  let bluetoothConnected = true;
  let recording = false;
  let recordingProgress = 68;
  let replayOffset = 0;
  let searchQuery = "";
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
  // Percentages of the piano drawing itself, so markers stay put at any width.
  /** @type {Record<number, {x: number, y: number}>} */
  let featurePositions = {
    1: { x: 49.5, y: 29 },
    2: { x: 76, y: 29 },
    3: { x: 79, y: 51 },
    4: { x: 21, y: 54.5 },
  };
  /** @type {string[]} */
  let annotations = [];
  let lastAction = "Ready when you are";
  let playbackStatus = "Ready when you are";

  $: song = songs[selectedSong];
  $: filteredScores = searchableScores.filter((score) => {
    const query = searchQuery.trim().toLowerCase();
    return (
      !query || `${score.title} ${score.composer}`.toLowerCase().includes(query)
    );
  });
  $: annotationCount = annotations.length + strokes.length;
  $: progress = Math.min(
    100,
    Math.round(((currentPage - 1) / song.pages) * 100 + 38 / song.pages),
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
    playbackStatus = `${songs[index].title} loaded`;
  }

  /** @param {{title: string, composer: string, source: string, songIndex?: number}} score */
  function selectSearchResult(score) {
    if (score.songIndex !== undefined) {
      selectSong(score.songIndex);
      searchQuery = "";
      return;
    }
    lastAction = `${score.title} found in the online catalog`;
  }

  function togglePlay() {
    isPlaying = !isPlaying;
    lastAction = isPlaying
      ? "Listening for your playing..."
      : "Practice paused";
    playbackStatus = lastAction;
  }

  function toggleSystem() {
    if (fallboardTransition) return;
    const nextSystemOn = !systemOn;
    systemOn = nextSystemOn;
    fallboardTransition = nextSystemOn ? "Opening…" : "Closing…";
    if (!nextSystemOn) {
      fallboardOpen = false;
      bluetoothConnected = false;
      isPlaying = false;
      recording = false;
      annotationMode = false;
      isEraser = false;
    }
    lastAction = nextSystemOn
      ? "Smart Piano system on"
      : "Smart Piano system off";
    fallboardTimer = setTimeout(() => {
      fallboardOpen = nextSystemOn;
      fallboardTransition = "";
    }, 3000);
  }

  function toggleCoffeeTable() {
    coffeeTableOpen = !coffeeTableOpen;
    lastAction = coffeeTableOpen
      ? "Coffee table pulled out"
      : "Coffee table closed";
  }

  function toggleBluetooth() {
    if (!systemOn) return;
    bluetoothConnected = !bluetoothConnected;
    lastAction = bluetoothConnected
      ? "Phone extension connected"
      : "Phone extension disconnected";
  }

  /** @param {number} seconds */
  function replay(seconds) {
    replayOffset = seconds;
    lastAction = `Replaying the last ${seconds} seconds`;
    playbackStatus = lastAction;
  }

  function toggleRecording() {
    if (!systemOn) return;
    recording = !recording;
    lastAction = recording ? "Recording playing" : "Recording saved";
    playbackStatus = lastAction;
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
    context.strokeStyle = "#a81f2d";
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
  <title>Smart Piano, a prototype by Evan Soreefan</title>
  <meta
    name="description"
    content="A smart grand piano interface prototype with synchronized sheet music and companion song picker."
  />
</svelte:head>

<svelte:window
  onkeydown={(event) => event.key === "Escape" && (showFeatureInfo = false)}
/>

<main class="page">
  <header class="title-page">
    <h1>Smart Piano</h1>
    <p class="title-sub">A piano that keeps the pianist in the music.</p>
    <div class="title-credits">
      <span>Project 1, interactive prototype</span>
      <span>Evan Soreefan</span>
    </div>
    <div class="staff" aria-hidden="true"></div>
  </header>

  <section class="about" aria-labelledby="about-title">
    <h2 id="about-title" class="visually-hidden">About Evan Soreefan</h2>
    <img class="portrait" src={profileImage} alt="Portrait of Evan Soreefan" />
    <div class="about-copy">
      <p class="about-lead">
        Hi, I'm Evan Soreefan, a fourth-year Computer Science student from the
        University of Cincinnati.
      </p>
      <p>
        Most of my knowledge and expertise involve programming in C/C++ and
        front-end development. I have been a co-op at Bilstein of America and
        Siemens Digital Industries Software (two-time intern).
      </p>
    </div>
  </section>

  <section class="instrument" aria-labelledby="instrument-title">
    <div class="instrument-copy">
      <h2 id="instrument-title">What's built into the piano</h2>
      <p class="lede">
        The physical instrument stays familiar while digital tools quietly
        extend the music desk, the soundboard, and the pianist's workflow.
      </p>
      <ul class="feature-list">
        {#each features as feature}
          <li>
            <button
              class:selected={selectedFeature === feature.number}
              class="feature-row"
              onclick={() => selectFeature(feature.number)}
            >
              <span class="mark">{feature.number}</span>
              <span class="feature-row-copy"
                ><strong>{feature.name}</strong><small>{feature.location}</small
                ></span
              >
            </button>
          </li>
        {/each}
      </ul>
    </div>
    <figure class="plate">
      <div
        class="piano-map"
        role="group"
        aria-label="Front-view piano drawing with feature markers"
      >
        <img
          class="piano-image"
          src={pianoImage}
          alt="Line drawing of a piano and bench, seen from the front"
        />
        {#each features as feature}
          <button
            class:active={selectedFeature === feature.number}
            class="mark map-point"
            style={`left: ${featurePositions[feature.number].x}%; top: ${featurePositions[feature.number].y}%;`}
            aria-label={`Select ${feature.name}`}
            onclick={() => selectFeature(feature.number)}
            >{feature.number}</button
          >
        {/each}
      </div>
      <figcaption>
        <button
          class="plate-caption"
          aria-label={`Open details for ${features[selectedFeature - 1].name}`}
          onclick={() => (showFeatureInfo = true)}
        >
          <span class="mark is-current">{selectedFeature}</span>
          <span>
            <strong>{features[selectedFeature - 1].name}</strong>
            <span class="caption-text"
              >{features[selectedFeature - 1].description}</span
            >
            <span class="caption-action">Open details</span>
          </span>
        </button>
      </figcaption>
    </figure>
  </section>

  <section class="prototype" aria-labelledby="prototype-title">
    <div class="prototype-intro">
      <h2 id="prototype-title">Try the prototype</h2>
      <p>
        Everything below is live. Choose a piece on the phone, turn pages and
        annotate on the display, and press the hub buttons to see the piano
        respond.
      </p>
    </div>

    <div class="instrument-front">
      <section class="device-stage" aria-labelledby="display-title">
        <div class="device-heading">
          <h3 id="display-title">Main music display</h3>
          <p class:active={isPlaying} class="listen-state">
            <span class="lamp"></span>{isPlaying ? "Listening" : "Standing by"}
          </p>
        </div>

        <div class:system-off={!systemOn} class="screen">
          <div class="screen-rail">
            <span class="now-title"
              >{song.title}<span
                >{song.composer}, {song.mood.toLowerCase()}</span
              ></span
            >
            <span class="page-count">Page {currentPage} of {song.pages}</span>
          </div>
          {#if !systemOn || !bluetoothConnected}
            <div class="connection-warning">Phone extension disconnected</div>
          {/if}
          <div class="screen-toolbar">
            <div class="toolbar-group">
              <button
                class="tool-button"
                aria-label="Previous page"
                title="Previous page"
                onclick={() => flipPage(-1)}
                disabled={currentPage === 1}
                ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                  ><path d={icons.prev} /></svg
                ></button
              ><button
                class="tool-button"
                aria-label="Next page"
                title="Next page"
                onclick={() => flipPage(1)}
                disabled={currentPage === song.pages}
                ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                  ><path d={icons.next} /></svg
                ></button
              >
            </div>
            <div class="toolbar-center">
              <span class="tempo-pulse" class:playing={isPlaying}></span>
              <span>{isPlaying ? "Audio sync active" : "Manual page turn"}</span
              >
              <button
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
                ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                  ><path d={icons.pen} /></svg
                >Annotate</button
              ><button
                class:enabled={isEraser}
                class="tool-button"
                aria-label="Toggle eraser"
                title="Toggle eraser"
                onclick={toggleEraser}
                ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                  ><path d={icons.eraser} /></svg
                ></button
              ><button
                class="tool-button"
                aria-label="Undo annotation"
                title="Undo annotation"
                disabled={undoStack.length === 0}
                onclick={undoAnnotation}
                ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                  ><path d={icons.undo} /></svg
                ></button
              ><button
                class="tool-button"
                aria-label="Redo annotation"
                title="Redo annotation"
                disabled={redoStack.length === 0}
                onclick={redoAnnotation}
                ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                  ><path d={icons.redo} /></svg
                ></button
              ><button
                class="tool-button"
                aria-label="Adjust brightness"
                title="Adjust brightness"
                onclick={() => (lastAction = "Display brightness adjusted")}
                ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                  ><path d={icons.sun} /></svg
                ></button
              >
            </div>
          </div>
          <div class="score-tray" class:annotation-mode={annotationMode}>
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
                {#each annotations as mark}<svg
                    class="icon"
                    viewBox="0 0 24 24"
                    role="img"
                    aria-label={`${mark} mark`}
                    ><path
                      d={mark === "Circle"
                        ? icons.circle
                        : mark === "Star"
                          ? icons.star
                          : icons.line}
                    /></svg
                  >{/each}
              </div>{/if}
            {#if annotationMode}<div class="annotation-tip">
                Stylus ready. Draw on the page, or add a mark from the phone.
              </div>{/if}
          </div>
          <div class="screen-footer">
            <div>Sync confidence <strong>94%</strong></div>
            <div class="progress-track">
              <span style={`width: ${progress}%`}></span>
            </div>
            <div>Swipe or use the arrows to turn</div>
          </div>
        </div>
      </section>

      <section class="hub" aria-labelledby="hub-title">
        <div class="device-heading">
          <h3 id="hub-title">Button hub</h3>
        </div>
        <div class="hub-keys">
          <button
            class:active={systemOn}
            class="hub-key"
            onclick={toggleSystem}
          >
            <svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
              ><path d={icons.power} /></svg
            >
            <span
              ><strong>{systemOn ? "Turn system off" : "Turn system on"}</strong
              ><small>Power display, fallboard, and phone connection</small
              ></span
            >
            <span class="hub-lamp" aria-hidden="true"></span>
          </button>
          <button
            class:active={coffeeTableOpen}
            class="hub-key"
            onclick={toggleCoffeeTable}
          >
            <svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
              ><path d={icons.cup} /></svg
            >
            <span
              ><strong
                >{coffeeTableOpen
                  ? "Close coffee table"
                  : "Pull out coffee table"}</strong
              ><small>Slide-out surface beneath the keyboard</small></span
            >
            <span class="hub-lamp" aria-hidden="true"></span>
          </button>
          <button
            class:active={bluetoothConnected}
            class="hub-key"
            onclick={toggleBluetooth}
          >
            <svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
              ><path d={icons.bluetooth} /></svg
            >
            <span
              ><strong
                >{bluetoothConnected
                  ? "Disconnect phone"
                  : "Connect phone"}</strong
              ><small>Bluetooth link for the phone extension</small></span
            >
            <span class="hub-lamp" aria-hidden="true"></span>
          </button>
        </div>
        <dl class="hub-status" aria-label="Smart Piano system status">
          <div>
            <dt>Coffee table</dt>
            <dd class:status-off={!coffeeTableOpen}>
              {coffeeTableOpen ? "Pulled out" : "Closed"}
            </dd>
          </div>
          <div>
            <dt>Fallboard</dt>
            <dd class:status-off={!fallboardTransition && !fallboardOpen}>
              {fallboardTransition || (fallboardOpen ? "Open" : "Closed")}
            </dd>
          </div>
          <div>
            <dt>Phone</dt>
            <dd class:status-off={!(bluetoothConnected && systemOn)}>
              {bluetoothConnected && systemOn ? "Connected" : "Disconnected"}
            </dd>
          </div>
          <div>
            <dt>System</dt>
            <dd class:status-off={!systemOn}>{systemOn ? "On" : "Off"}</dd>
          </div>
        </dl>
      </section>

      <section class="dock" aria-labelledby="dock-title">
        <div class="device-heading">
          <h3 id="dock-title">Phone extension dock</h3>
        </div>
        <p class="device-note">
          Place your phone in the dock and choose a piece here. Your selection
          loads onto the main music display instantly.
        </p>
        <aside class="phone" aria-label="Companion app on the docked phone">
          <div class="phone-screen">
            {#if !systemOn || !bluetoothConnected}
              <div class="disconnected-state">
                <span class="disconnected-icon"
                  ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                    ><path d={icons.close} /></svg
                  ></span
                >
                <strong>Music display disconnected</strong>
                <p>
                  Turn on the system and reconnect Bluetooth to restore the
                  phone extension.
                </p>
              </div>
            {:else}
              <p class="app-title">Smart Piano Companion</p>
              <p class="companion-status">
                <span></span>Connected via Bluetooth
              </p>
              <div class="phone-search">
                <label for="sheet-search">Search sheet music</label>
                <input
                  id="sheet-search"
                  type="search"
                  placeholder="Title or composer"
                  bind:value={searchQuery}
                />
                {#if searchQuery.trim()}
                  <div class="search-results">
                    {#if filteredScores.length === 0}
                      <p class="search-empty">
                        Nothing matches. Try a composer's last name.
                      </p>
                    {:else}
                      {#each filteredScores as result}
                        <button
                          class="search-result"
                          onclick={() => selectSearchResult(result)}
                        >
                          <span
                            ><strong>{result.title}</strong><small
                              >{result.composer}</small
                            ></span
                          >
                          <em>{result.source}</em>
                        </button>
                      {/each}
                    {/if}
                  </div>
                {/if}
              </div>
              <div class="song-list">
                {#each songs as item, index}<button
                    class:selected={selectedSong === index}
                    class="song-item"
                    onclick={() => selectSong(index)}
                    ><span class="song-copy"
                      ><strong>{item.title}</strong><small
                        >{item.composer}, {item.level.toLowerCase()}</small
                      ></span
                    >{#if selectedSong === index}<span class="song-state"
                        >On the display</span
                      >{/if}</button
                  >{/each}
              </div>
              <div class="app-section">
                <div class="app-section-head">
                  <h4>Playback</h4>
                  <span>{playbackStatus}</span>
                </div>
                <div class="playback-buttons">
                  <button
                    class:playing={isPlaying}
                    class="app-button play-control"
                    onclick={togglePlay}
                  >
                    {isPlaying ? "Pause" : "Play"}
                  </button>
                  {#each [30, 20, 10] as seconds}
                    <button
                      class:active={replayOffset === seconds}
                      class="app-button"
                      aria-label={`Replay the last ${seconds} seconds`}
                      onclick={() => replay(seconds)}
                      ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                        ><path d={icons.replay} /></svg
                      >{seconds}s</button
                    >
                  {/each}
                </div>
              </div>
              <div class="app-section">
                <div class="app-section-head">
                  <h4>Record your playing</h4>
                  <span>{recording ? "Recording" : "Latest take"}</span>
                </div>
                <button
                  class:recording
                  class="record-button"
                  onclick={toggleRecording}
                >
                  <span class="record-dot"></span><strong
                    >{recording ? "Stop recording" : "Record playing"}</strong
                  ><small
                    >{recording
                      ? "Capturing your performance"
                      : "Ready for a new take"}</small
                  >
                </button>
                <div class="recording-scrubber">
                  <div class="scrubber-label">
                    <span>Latest recording</span><span
                      >{recordingProgress}%</span
                    >
                  </div>
                  <input
                    aria-label="Scrub latest recording"
                    type="range"
                    min="0"
                    max="100"
                    bind:value={recordingProgress}
                  />
                </div>
              </div>
              <div class="app-section">
                <div class="app-section-head">
                  <h4>Annotation tools</h4>
                  <span>{annotationCount} marks</span>
                </div>
                <div class="annotation-tools">
                  <button
                    class:active={annotationMode}
                    class="app-button"
                    onclick={toggleAnnotation}
                    ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                      ><path d={icons.pen} /></svg
                    >Stylus</button
                  ><button
                    class:active={isEraser}
                    class="app-button"
                    onclick={toggleEraser}
                    ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                      ><path d={icons.eraser} /></svg
                    >Eraser</button
                  ><button
                    class="app-button"
                    onclick={() => addAnnotation("Circle")}
                    ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                      ><path d={icons.circle} /></svg
                    >Circle</button
                  ><button
                    class="app-button"
                    onclick={() => addAnnotation("Star")}
                    ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                      ><path d={icons.star} /></svg
                    >Star</button
                  ><button
                    class="app-button"
                    onclick={() => addAnnotation("Line")}
                    ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                      ><path d={icons.line} /></svg
                    >Line</button
                  >
                </div>
                <div class="annotation-history">
                  <button
                    class="app-button"
                    disabled={undoStack.length === 0}
                    onclick={undoAnnotation}
                    ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                      ><path d={icons.undo} /></svg
                    >Undo</button
                  >
                  <button
                    class="app-button"
                    disabled={redoStack.length === 0}
                    onclick={redoAnnotation}
                    ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
                      ><path d={icons.redo} /></svg
                    >Redo</button
                  >
                </div>
              </div>
            {/if}
          </div>
        </aside>
      </section>
    </div>
  </section>

  <footer class="colophon">
    <p>
      Smart Piano is a Project 1 prototype by Evan Soreefan, University of
      Cincinnati.
    </p>
  </footer>
</main>

{#if showFeatureInfo}<div
    class="modal-backdrop"
    role="presentation"
    onclick={(event) =>
      event.target === event.currentTarget && (showFeatureInfo = false)}
  >
    <div
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-labelledby="feature-title"
    >
      <button
        class="modal-close"
        aria-label="Close feature details"
        onclick={() => (showFeatureInfo = false)}
        ><svg class="icon" viewBox="0 0 24 24" aria-hidden="true"
          ><path d={icons.close} /></svg
        ></button
      >
      <span class="mark is-current">{selectedFeature}</span>
      <h2 id="feature-title">{features[selectedFeature - 1].name}</h2>
      <p class="feature-location">{features[selectedFeature - 1].location}</p>
      <p class="feature-description">
        {features[selectedFeature - 1].description}
      </p>
    </div>
  </div>{/if}
