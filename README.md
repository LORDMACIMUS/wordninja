<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Morpheme Ninja: Word Slicer</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@700;900&family=JetBrains+Mono:wght@700;800&family=Nunito:wght@700;800;900&display=swap" rel="stylesheet">
  <style>
    :root {
      --dojo-bg: #160F0B;
      --wood-dark: #241710;
      --wood-panel: #2F1F16;
      --wood-border: #5E3D28;
      --parchment: #FAF1DF;
      --parchment-shadow: #DECAA6;
      --ink: #241911;
      --katana-cyan: #38F2FF;
      --jade-green: #14D98B;
      --gold-amber: #FFB830;
      --crimson-bomb: #FF3B5C;
      --violet-root: #B36BFF;
      --font-display: 'Cinzel', Georgia, serif;
      --font-mono: 'JetBrains Mono', monospace;
      --font-ui: 'Nunito', system-ui, sans-serif;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      -webkit-tap-highlight-color: transparent;
      user-select: none;
    }

    html, body {
      width: 100%;
      height: 100%;
      overflow: hidden;
      background: var(--dojo-bg);
      color: var(--parchment);
      font-family: var(--font-ui);
    }

    .dojo-shell {
      display: flex;
      flex-direction: column;
      width: 100%;
      height: 100%;
      max-width: 1220px;
      margin: 0 auto;
      padding: 8px 10px 10px;
      gap: 7px;
      position: relative;
    }

    /* TOP DOJO HUD BAR */
    .dojo-hud {
      background: rgba(36, 23, 16, 0.96);
      border: 2px solid var(--wood-border);
      border-radius: 14px;
      padding: 6px 12px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 8px;
      flex-wrap: wrap;
      z-index: 15;
      box-shadow: 0 4px 16px rgba(0, 0, 0, 0.55);
    }

    .hud-group {
      display: flex;
      align-items: center;
      gap: 8px;
      flex-wrap: wrap;
    }

    .dojo-select {
      min-height: 38px;
      background: #1A100B;
      color: var(--gold-amber);
      border: 2px solid var(--gold-amber);
      border-radius: 10px;
      padding: 0 10px;
      font-family: var(--font-mono);
      font-size: 0.82rem;
      font-weight: 800;
      cursor: pointer;
    }

    .hud-pill {
      background: #1A100B;
      border: 1.5px solid var(--wood-border);
      border-radius: 10px;
      padding: 5px 10px;
      font-family: var(--font-mono);
      font-size: 0.84rem;
      font-weight: 800;
      display: inline-flex;
      align-items: center;
      gap: 5px;
    }
    .pill-score { border-color: rgba(255, 184, 48, 0.5); color: var(--gold-amber); }
    .pill-combo { border-color: rgba(56, 242, 255, 0.5); color: var(--katana-cyan); }
    .pill-lives { border-color: rgba(255, 59, 92, 0.5); color: #FF7A90; }

    .btn-dojo {
      background: var(--wood-panel);
      color: var(--parchment);
      border: 2px solid var(--wood-border);
      border-radius: 10px;
      min-height: 38px;
      padding: 6px 12px;
      font-family: var(--font-mono);
      font-size: 0.8rem;
      font-weight: 800;
      cursor: pointer;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: 5px;
      transition: all 0.12s ease;
    }
    .btn-dojo:hover {
      border-color: var(--gold-amber);
      color: #FFFFFF;
    }
    .btn-dojo:active { transform: translateY(2px); }
    .btn-jade {
      background: rgba(20, 217, 139, 0.16);
      border-color: var(--jade-green);
      color: var(--jade-green);
    }
    .btn-gold {
      background: rgba(255, 184, 48, 0.18);
      border-color: var(--gold-amber);
      color: var(--gold-amber);
    }

    /* SENSEI INSTRUCTION & STAGE INDICATOR */
    .sensei-strip {
      background: linear-gradient(90deg, rgba(36, 23, 16, 0.96), rgba(52, 34, 23, 0.96), rgba(36, 23, 16, 0.96));
      border: 1.5px solid rgba(255, 184, 48, 0.35);
      border-radius: 12px;
      padding: 6px 14px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 10px;
      font-size: 0.88rem;
      font-weight: 800;
      z-index: 15;
    }
    .sensei-prompt {
      color: var(--parchment);
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
    .phase-pills {
      display: flex;
      align-items: center;
      gap: 6px;
      flex-shrink: 0;
      font-family: var(--font-mono);
      font-size: 0.72rem;
    }
    .phase-step {
      padding: 2px 8px;
      border-radius: 999px;
      background: #1A100B;
      border: 1px solid var(--wood-border);
      color: #A89480;
    }
    .phase-step.active {
      border-color: var(--katana-cyan);
      color: var(--katana-cyan);
      box-shadow: 0 0 8px rgba(56, 242, 255, 0.25);
    }
    .phase-step.done {
      border-color: var(--jade-green);
      color: var(--jade-green);
    }

    /* MAIN DOJO CANVAS ARENA */
    .dojo-arena {
      position: relative;
      flex: 1;
      width: 100%;
      min-height: 280px;
      border-radius: 18px;
      overflow: hidden;
      border: 3px solid var(--wood-border);
      background: radial-gradient(circle at 50% 45%, #342116 0%, #1B110B 72%, #110A07 100%);
      box-shadow: inset 0 0 80px rgba(0, 0, 0, 0.85);
    }

    canvas {
      display: block;
      width: 100%;
      height: 100%;
      touch-action: none;
      cursor: crosshair;
    }

    /* ========================================================================
       BULLET-TIME DEFINITION DOJO OVERLAY (STEP 2: PARTS -> STEP 3: FULL WORD)
       ======================================================================== */
    .definition-overlay {
      position: absolute;
      inset: 0;
      background: rgba(14, 9, 6, 0.84);
      backdrop-filter: blur(4px);
      display: none;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 14px;
      z-index: 25;
      animation: fadeInFast 0.16s ease;
    }
    @keyframes fadeInFast {
      from { opacity: 0; transform: scale(0.97); }
      to { opacity: 1; transform: scale(1); }
    }

    .def-card {
      background: #241710;
      border: 2.5px solid var(--gold-amber);
      border-radius: 20px;
      max-width: 660px;
      width: 100%;
      padding: 18px 18px 20px;
      box-shadow: 0 18px 50px rgba(0, 0, 0, 0.8), 0 0 30px rgba(255, 184, 48, 0.18);
      display: flex;
      flex-direction: column;
      gap: 14px;
      text-align: center;
    }

    /* SPLIT MORPHEME TILES DISPLAY */
    .sliced-parts-row {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 10px;
      flex-wrap: wrap;
    }

    .morpheme-slab {
      background: var(--parchment);
      color: var(--ink);
      border: 3px solid #9E7E56;
      border-radius: 14px;
      padding: 8px 16px;
      min-width: 112px;
      box-shadow: 0 5px 0 var(--parchment-shadow);
      transition: all 0.2s ease;
    }
    .morpheme-slab.active-part {
      border-color: var(--katana-cyan);
      box-shadow: 0 0 18px rgba(56, 242, 255, 0.55), 0 5px 0 var(--parchment-shadow);
      transform: translateY(-3px) scale(1.04);
    }
    .morpheme-slab.solved-part {
      border-color: var(--jade-green);
      background: #E6FFF4;
    }
    .slab-role {
      font-family: var(--font-mono);
      font-size: 0.68rem;
      font-weight: 800;
      text-transform: uppercase;
      color: #7A5C3D;
    }
    .slab-text {
      font-family: var(--font-mono);
      font-size: 1.35rem;
      font-weight: 800;
      color: var(--ink);
    }
    .slab-meaning {
      font-size: 0.78rem;
      font-weight: 900;
      color: #0B7A4E;
      margin-top: 2px;
      min-height: 16px;
    }

    .plus-sign {
      font-family: var(--font-mono);
      font-size: 1.4rem;
      font-weight: 900;
      color: var(--gold-amber);
    }

    .def-question-banner {
      background: #170E09;
      border: 1.5px solid var(--wood-border);
      border-radius: 12px;
      padding: 9px 14px;
      font-size: 1.02rem;
      font-weight: 900;
      color: var(--parchment);
    }
    .def-question-banner span {
      color: var(--katana-cyan);
      font-family: var(--font-mono);
    }

    .def-choices-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 9px;
    }
    @media (max-width: 540px) {
      .def-choices-grid { grid-template-columns: 1fr; }
    }

    .choice-btn {
      background: #322117;
      color: var(--parchment);
      border: 2px solid var(--wood-border);
      border-radius: 14px;
      min-height: 52px;
      padding: 10px 14px;
      font-family: var(--font-ui);
      font-size: 0.96rem;
      font-weight: 900;
      cursor: pointer;
      transition: transform 0.08s ease, border-color 0.14s ease, background 0.14s ease;
    }
    .choice-btn:hover:not(:disabled) {
      border-color: var(--katana-cyan);
      background: #3E291D;
    }
    .choice-btn:active:not(:disabled) {
      transform: translateY(2px);
    }
    .choice-btn.correct {
      background: rgba(20, 217, 139, 0.24);
      border-color: var(--jade-green);
      color: #9EFFD6;
    }
    .choice-btn.wrong {
      background: rgba(255, 59, 92, 0.24);
      border-color: var(--crimson-bomb);
      color: #FFB3C0;
      animation: shakeBtn 0.28s ease;
    }
    @keyframes shakeBtn {
      0%, 100% { transform: translateX(0); }
      25% { transform: translateX(-6px); }
      75% { transform: translateX(6px); }
    }

    /* MODALS (INTRO, SCROLL LEXICON, GAME OVER) */
    .dojo-modal {
      position: fixed;
      inset: 0;
      background: rgba(12, 7, 5, 0.92);
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 16px;
      z-index: 60;
    }
    .modal-card {
      background: #241710;
      border: 2.5px solid var(--gold-amber);
      border-radius: 22px;
      max-width: 650px;
      width: 100%;
      max-height: 92vh;
      overflow-y: auto;
      padding: 22px 20px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.85);
      display: flex;
      flex-direction: column;
      gap: 14px;
      text-align: center;
    }
    .modal-title {
      font-family: var(--font-display);
      font-size: clamp(1.5rem, 3.8vw, 2.15rem);
      font-weight: 900;
      color: var(--parchment);
      letter-spacing: 1px;
    }
    .modal-sub {
      font-family: var(--font-mono);
      font-size: 0.82rem;
      color: var(--gold-amber);
      text-transform: uppercase;
      letter-spacing: 1.5px;
    }

    .steps-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 8px;
      text-align: left;
    }
    @media (max-width: 600px) {
      .steps-grid { grid-template-columns: 1fr; }
    }
    .step-box {
      background: #180F0A;
      border: 1.5px solid var(--wood-border);
      border-radius: 14px;
      padding: 10px 12px;
      font-size: 0.84rem;
      line-height: 1.4;
    }

    .scroll-bank {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      max-height: 230px;
      overflow-y: auto;
      padding: 12px;
      background: #140C08;
      border: 1.5px solid var(--wood-border);
      border-radius: 14px;
    }
    .scroll-chip {
      background: var(--parchment);
      color: var(--ink);
      border: 2px solid var(--gold-amber);
      border-radius: 10px;
      padding: 6px 11px;
      font-family: var(--font-mono);
      font-size: 0.8rem;
      font-weight: 800;
      text-align: left;
    }
  </style>
</head>
<body>

  <div class="dojo-shell">
    <!-- TOP HUD BAR -->
    <div class="dojo-hud">
      <div class="hud-group">
        <select id="curriculumSetSelect" class="dojo-select" onchange="changeSet()" title="Select Curriculum Set">
          <option value="set1">SET 1 • Grades ≤ 3</option>
          <option value="set2">SET 2 • Grades 3–4</option>
          <option value="set3">SET 3 • Grades 4–5+</option>
          <option value="all">ALL SETS MIX (59 Units)</option>
        </select>

        <div class="hud-pill pill-score" id="hudScore">⭐ 0</div>
        <div class="hud-pill pill-combo" id="hudCombo">⚔️ 0 Mastered</div>
        <div class="hud-pill pill-lives" id="hudLives">❤️❤️❤️</div>
      </div>

      <div class="hud-group">
        <button class="btn-dojo btn-gold" onclick="openScrollModal()">📜 Scroll Book (<span id="masteredCount">0</span>)</button>
        <button class="btn-dojo" id="audioBtn" onclick="toggleAudio()">🔊 Dojo Audio: ON</button>
        <button class="btn-dojo" onclick="openIntroModal()">?</button>
      </div>
    </div>

    <!-- SENSEI TIMING & 3-STEP FLOW STRIP -->
    <div class="sensei-strip">
      <div class="sensei-prompt" id="senseiBanner">
        Swipe across the glowing cut-line of flying Word Scrolls—avoid Trickster Bombs!
      </div>
      <div class="phase-pills">
        <span class="phase-step active" id="step1Pill">1. SLICE</span>
        <span class="phase-step" id="step2Pill">2. DEFINE PARTS</span>
        <span class="phase-step" id="step3Pill">3. FULL WORD</span>
      </div>
    </div>

    <!-- MAIN FRUIT-NINJA STYLE CANVAS ARENA -->
    <div class="dojo-arena" id="dojoArena">
      <canvas id="gameCanvas"></canvas>

      <!-- BULLET-TIME DEFINITION CHALLENGE OVERLAY -->
      <div class="definition-overlay" id="definitionOverlay">
        <div class="def-card">
          <div style="display:flex; align-items:center; justify-content:space-between; gap:8px;">
            <span style="font-family:var(--font-mono); font-size:0.78rem; color:var(--gold-amber); font-weight:800;" id="defStepTitle">
              STEP 2 OF 3 • DEFINE EACH MORPHEME PART
            </span>
            <span style="font-family:var(--font-mono); font-size:0.78rem; color:var(--katana-cyan); font-weight:800;" id="sliceAccuracyBadge">
              CLEAN CUT!
            </span>
          </div>

          <!-- SEPARATED MORPHEME SLABS -->
          <div class="sliced-parts-row" id="slicedPartsRow"></div>

          <!-- ACTIVE QUESTION PROMPT -->
          <div class="def-question-banner" id="defQuestionPrompt">
            What does the prefix <span>un-</span> mean?
          </div>

          <!-- 4 MULTIPLE CHOICE BUTTONS -->
          <div class="def-choices-grid" id="defChoicesGrid"></div>
        </div>
      </div>
    </div>
  </div>

  <!-- ========================================================================
       INTRO / DOJO RULES MODAL
       ======================================================================== -->
  <div class="dojo-modal" id="introModal">
    <div class="modal-card">
      <div>
        <div class="modal-sub">Morphology Katana Arcade</div>
        <div class="modal-title">MORPHEME NINJA</div>
      </div>

      <!-- KATANA & BAMBOO SCROLL EMBLEM -->
      <div style="margin: 0 auto; width: 92px; height: 74px;">
        <svg viewBox="0 0 120 90" width="100%" height="100%">
          <rect x="18" y="24" width="84" height="42" rx="8" fill="#FAF1DF" stroke="#FFB830" stroke-width="3"/>
          <line x1="52" y1="18" x2="52" y2="72" stroke="#38F2FF" stroke-width="3" stroke-dasharray="5,4"/>
          <line x1="12" y1="78" x2="108" y2="12" stroke="#E8F8FF" stroke-width="4" stroke-linecap="round"/>
          <circle cx="52" cy="45" r="5" fill="#38F2FF"/>
        </svg>
      </div>

      <div class="steps-grid">
        <div class="step-box">
          <strong style="color:var(--katana-cyan);">1. Timed Morpheme Slice</strong><br>
          Word Scrolls launch into the air! Swipe your blade across the <b>dotted seam</b> between prefix, root, and suffix—and dodge red <b>Trickster Bombs</b>!
        </div>
        <div class="step-box">
          <strong style="color:var(--gold-amber);">2. Define Each Small Part</strong><br>
          When a word splits in mid-air, time slows down! Define each separated morpheme piece (<code>un-</code> = <em>not</em>, <code>happy</code> = <em>feeling glad</em>).
        </div>
        <div class="step-box">
          <strong style="color:var(--jade-green);">3. Define the Full Word</strong><br>
          Combine the meanings of the pieces to unlock the full word's definition and earn bonus Dojo Points!
        </div>
      </div>

      <button class="btn-dojo btn-jade" style="min-height:50px; font-size:1rem; width:100%;" onclick="startDojoSession()">
        ⚔️ Draw Katana & Begin!
      </button>
    </div>
  </div>

  <!-- ========================================================================
       SCROLL BOOK (MASTERED WORDS) MODAL
       ======================================================================== -->
  <div class="dojo-modal" id="scrollModal" style="display:none;">
    <div class="modal-card">
      <div class="modal-sub">Mastered Word Breakdowns</div>
      <div class="modal-title" style="font-size:1.5rem;">NINJA SCROLL BOOK</div>
      <div class="scroll-bank" id="scrollBankGrid"></div>
      <button class="btn-dojo btn-gold" style="width:100%;" onclick="document.getElementById('scrollModal').style.display='none'">
        Back to Dojo
      </button>
    </div>
  </div>

  <!-- ========================================================================
       GAME OVER / SESSION SUMMARY MODAL
       ======================================================================== -->
  <div class="dojo-modal" id="gameOverModal" style="display:none;">
    <div class="modal-card" style="max-width:520px;">
      <div class="modal-sub">Training Complete</div>
      <div class="modal-title" id="gameOverTitle">DOJO SESSION ENDED</div>
      <p style="font-size:0.96rem; line-height:1.5; color:var(--parchment);" id="gameOverStats"></p>
      <div style="display:flex; gap:10px; flex-wrap:wrap;">
        <button class="btn-dojo btn-jade" style="flex:1; min-height:48px;" onclick="restartDojo()">
          ⚔️ Play Again
        </button>
        <button class="btn-dojo btn-gold" style="flex:1; min-height:48px;" onclick="openScrollModal()">
          📜 View Scroll Book
        </button>
      </div>
    </div>
  </div>

  <script>
    /* ========================================================================
       1. CURATED MORPHEME WORD DATABASE (WITH PART & FULL WORD DEFINITIONS)
       Organized across Set 1 (Grades <=3), Set 2 (Grades 3-4), Set 3 (Grades 4-5+)
       ======================================================================== */
    const WORD_SCROLLS = {
      set1: [
        {
          word: "unhappy",
          parts: [
            { text: "un-", role: "Prefix", meaning: "not" },
            { text: "happy", role: "Base Word", meaning: "feeling glad or cheerful" }
          ],
          fullMeaning: "not feeling glad or cheerful",
          distractorFull: ["full of cheerful joy", "feeling glad before", "someone who is glad"],
          trickster: "uncle"
        },
        {
          word: "unlock",
          parts: [
            { text: "un-", role: "Prefix", meaning: "not / reverse" },
            { text: "lock", role: "Base Word", meaning: "fasten shut with a key" }
          ],
          fullMeaning: "open something that was fastened shut",
          distractorFull: ["fasten shut before", "locked many times", "without any key"],
          trickster: "under"
        },
        {
          word: "repaint",
          parts: [
            { text: "re-", role: "Prefix", meaning: "again, back" },
            { text: "paint", role: "Base Word", meaning: "cover with color" }
          ],
          fullMeaning: "cover with color again",
          distractorFull: ["not covered with color", "without any paint", "paint before anyone else"],
          trickster: "ready"
        },
        {
          word: "replay",
          parts: [
            { text: "re-", role: "Prefix", meaning: "again, back" },
            { text: "play", role: "Base Word", meaning: "run a game or video" }
          ],
          fullMeaning: "run a game or video again",
          distractorFull: ["stop playing forever", "play in the middle", "someone who plays"],
          trickster: "reach"
        },
        {
          word: "preheat",
          parts: [
            { text: "pre-", role: "Prefix", meaning: "before" },
            { text: "heat", role: "Base Word", meaning: "make warm or hot" }
          ],
          fullMeaning: "warm up an oven before cooking",
          distractorFull: ["cool down completely", "too hot to touch", "without any heat"],
          trickster: "pretty"
        },
        {
          word: "nonstop",
          parts: [
            { text: "non-", role: "Prefix", meaning: "not, absence" },
            { text: "stop", role: "Base Word", meaning: "come to a halt" }
          ],
          fullMeaning: "continuing without coming to a halt",
          distractorFull: ["halting before noon", "stopping again", "under a stop sign"],
          trickster: "noon"
        },
        {
          word: "midway",
          parts: [
            { text: "mid-", role: "Prefix", meaning: "in the middle, part way" },
            { text: "way", role: "Base Word", meaning: "a path or distance" }
          ],
          fullMeaning: "in the middle of a path or distance",
          distractorFull: ["at the very end", "before starting", "without a path"],
          trickster: "middle"
        },
        {
          word: "teacher",
          parts: [
            { text: "teach", role: "Base Word", meaning: "help others learn" },
            { text: "-er", role: "Suffix", meaning: "someone who" }
          ],
          fullMeaning: "someone who helps others learn",
          distractorFull: ["not helping others learn", "learning in the past", "full of lessons"],
          trickster: "water"
        },
        {
          word: "harmless",
          parts: [
            { text: "harm", role: "Base Word", meaning: "hurt or damage" },
            { text: "-less", role: "Suffix", meaning: "without" }
          ],
          fullMeaning: "unable to cause hurt or damage",
          distractorFull: ["full of danger", "causing hurt again", "before any damage"],
          trickster: "bless"
        },
        {
          word: "joyful",
          parts: [
            { text: "joy", role: "Base Word", meaning: "great happiness" },
            { text: "-ful", role: "Suffix", meaning: "full of" }
          ],
          fullMeaning: "full of great happiness",
          distractorFull: ["without any happiness", "not happy at all", "happening in the past"],
          trickster: "awful"
        },
        {
          word: "jumped",
          parts: [
            { text: "jump", role: "Base Word", meaning: "leap into the air" },
            { text: "-ed", role: "Suffix", meaning: "past tense verb" }
          ],
          fullMeaning: "leaped into the air in the past",
          distractorFull: ["someone who leaps", "without leaping", "leaping before noon"],
          trickster: "shed"
        }
      ],
      set2: [
        {
          word: "disagree",
          parts: [
            { text: "dis-", role: "Prefix", meaning: "not, apart, opposite" },
            { text: "agree", role: "Base Word", meaning: "have the same opinion" }
          ],
          fullMeaning: "have a different or opposite opinion",
          distractorFull: ["have the exact same opinion", "agree again later", "capable of agreeing"],
          trickster: "distance"
        },
        {
          word: "misread",
          parts: [
            { text: "mis-", role: "Prefix", meaning: "wrongly, badly" },
            { text: "read", role: "Base Word", meaning: "look at and understand words" }
          ],
          fullMeaning: "read or understand words wrongly",
          distractorFull: ["read before class", "unable to be read", "someone who reads"],
          trickster: "mist"
        },
        {
          word: "submarine",
          parts: [
            { text: "sub-", role: "Prefix", meaning: "under, below" },
            { text: "marine", role: "Base Word", meaning: "relating to the sea" }
          ],
          fullMeaning: "a vessel that travels under the sea",
          distractorFull: ["flying high above the clouds", "tiny drop of water", "across the desert"],
          trickster: "subject"
        },
        {
          word: "microscope",
          parts: [
            { text: "micro-", role: "Prefix", meaning: "tiny, one millionth" },
            { text: "scope", role: "Root", meaning: "instrument for viewing" }
          ],
          fullMeaning: "an instrument for viewing tiny objects",
          distractorFull: ["viewing distant stars", "viewing underwater", "broken viewing glass"],
          trickster: "microbe"
        },
        {
          word: "invisible",
          parts: [
            { text: "in-", role: "Prefix", meaning: "not" },
            { text: "vis", role: "Latin Root", meaning: "to see" },
            { text: "-ible", role: "Suffix", meaning: "capable of being" }
          ],
          fullMeaning: "not capable of being seen",
          distractorFull: ["easy to see from far away", "seen wrongly", "seeing many things"],
          trickster: "insect"
        },
        {
          word: "washable",
          parts: [
            { text: "wash", role: "Base Word", meaning: "clean with water" },
            { text: "-able", role: "Suffix", meaning: "capable of being" }
          ],
          fullMeaning: "capable of being cleaned with water",
          distractorFull: ["without any water", "cleaned wrongly", "condition of being dirty"],
          trickster: "table"
        },
        {
          word: "kindness",
          parts: [
            { text: "kind", role: "Base Word", meaning: "friendly and caring" },
            { text: "-ness", role: "Suffix", meaning: "condition, state of" }
          ],
          fullMeaning: "the quality or state of being friendly and caring",
          distractorFull: ["not friendly at all", "someone who is caring", "before being friendly"],
          trickster: "witness"
        },
        {
          word: "semicircle",
          parts: [
            { text: "semi-", role: "Prefix", meaning: "half, partly" },
            { text: "circle", role: "Base Word", meaning: "a round shape" }
          ],
          fullMeaning: "half of a round shape",
          distractorFull: ["many round shapes", "against circles", "under a circle"],
          trickster: "seminar"
        }
      ],
      set3: [
        {
          word: "transport",
          parts: [
            { text: "trans-", role: "Prefix", meaning: "across, beyond" },
            { text: "port", role: "Latin Root", meaning: "to carry" }
          ],
          fullMeaning: "to carry goods or people across from one place to another",
          distractorFull: ["to break something apart", "to write down a message", "to hear from afar"],
          trickster: "portion"
        },
        {
          word: "telescope",
          parts: [
            { text: "tele", role: "Greek Root", meaning: "distant, far" },
            { text: "scope", role: "Root", meaning: "instrument for viewing" }
          ],
          fullMeaning: "an instrument for viewing distant objects in space",
          distractorFull: ["an instrument for hearing whispers", "writing a long story", "building a tall tower"],
          trickster: "telephone"
        },
        {
          word: "construct",
          parts: [
            { text: "con-", role: "Prefix", meaning: "together, with" },
            { text: "struct", role: "Latin Root", meaning: "to build" }
          ],
          fullMeaning: "to build or put parts together",
          distractorFull: ["to tear down and break", "to drag behind", "to speak loudly"],
          trickster: "strict"
        },
        {
          word: "predict",
          parts: [
            { text: "pre-", role: "Prefix", meaning: "before" },
            { text: "dict", role: "Latin Root", meaning: "say" }
          ],
          fullMeaning: "to say what will happen before it occurs",
          distractorFull: ["to write after an event", "to carry back", "to break suddenly"],
          trickster: "dictum"
        },
        {
          word: "disrupt",
          parts: [
            { text: "dis-", role: "Prefix", meaning: "apart, opposite" },
            { text: "rupt", role: "Latin Root", meaning: "to break, burst" }
          ],
          fullMeaning: "to break apart or interrupt normal order",
          distractorFull: ["to build together carefully", "to see clearly", "to carry across"],
          trickster: "rupture"
        },
        {
          word: "auditorium",
          parts: [
            { text: "aud", role: "Latin Root", meaning: "hear" },
            { text: "-orium", role: "Suffix", meaning: "a place for" }
          ],
          fullMeaning: "a large hall built for an audience to hear performances",
          distractorFull: ["a place to store tractors", "a book of drawings", "a clock that tracks time"],
          trickster: "audit"
        },
        {
          word: "chronology",
          parts: [
            { text: "chron", role: "Greek Root", meaning: "time" },
            { text: "-ology", role: "Suffix", meaning: "study or order of" }
          ],
          fullMeaning: "the arrangement of events in order of time",
          distractorFull: ["the study of distant stars", "the act of writing", "pulling heavy loads"],
          trickster: "chrome"
        },
        {
          word: "extract",
          parts: [
            { text: "ex-", role: "Prefix", meaning: "out, away" },
            { text: "tract", role: "Latin Root", meaning: "to drag or pull" }
          ],
          fullMeaning: "to pull or draw something out",
          distractorFull: ["to push together inside", "to speak softly", "to look closely"],
          trickster: "track"
        }
      ]
    };

    const EXTRA_MEANING_DISTRACTORS = [
      "in the middle of",
      "full of energy",
      "someone who",
      "before",
      "again, back",
      "without",
      "under, below",
      "wrongly, badly",
      "to carry",
      "to build",
      "to break, burst",
      "to hear",
      "distant, far",
      "time",
      "past tense verb"
    ];

    /* ========================================================================
       2. KATANA SWOOSH & DOJO TAUNT SYNTHESIZER (WEB AUDIO)
       ======================================================================== */
    let audioCtx = null;
    let audioEnabled = true;

    function initAudio() {
      if (!audioCtx) {
        const Ctx = window.AudioContext || window.webkitAudioContext;
        if (Ctx) audioCtx = new Ctx();
      }
      if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
    }

    function toggleAudio() {
      initAudio();
      audioEnabled = !audioEnabled;
      document.getElementById('audioBtn').textContent = audioEnabled ? '🔊 Dojo Audio: ON' : '🔇 Dojo Audio: OFF';
    }

    function playTone(freq1, freq2, wave, dur, vol, delay = 0) {
      if (!audioEnabled || !audioCtx) return;
      const t = audioCtx.currentTime + delay;
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.type = wave;
      osc.frequency.setValueAtTime(freq1, t);
      if (freq2 !== freq1) osc.frequency.exponentialRampToValueAtTime(Math.max(20, freq2), t + dur);
      gain.gain.setValueAtTime(vol, t);
      gain.gain.exponentialRampToValueAtTime(0.001, t + dur);
      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.start(t);
      osc.stop(t + dur + 0.02);
    }

    const sfx = {
      swoosh: () => playTone(680, 190, 'triangle', 0.11, 0.07),
      slice: () => {
        playTone(980, 320, 'sawtooth', 0.14, 0.11, 0);
        playTone(587.33, 880, 'sine', 0.18, 0.1, 0.04);
      },
      partCorrect: () => {
        playTone(523.25, 659.25, 'triangle', 0.14, 0.11, 0);
      },
      wordMastered: () => {
        [523.25, 659.25, 783.99, 1046.5].forEach((n, i) => playTone(n, n, 'triangle', 0.2, 0.11, i * 0.06));
      },
      wrong: () => {
        playTone(170, 110, 'sawtooth', 0.22, 0.12, 0);
      },
      bomb: () => {
        playTone(130, 48, 'sawtooth', 0.36, 0.16, 0);
        playTone(95, 40, 'square', 0.36, 0.14, 0.05);
      }
    };

    /* ========================================================================
       3. GAME STATE & SCROLL SPAWNER
       ======================================================================== */
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    const game = {
      running: false,
      inDefinitionPhase: false,
      setKey: 'set1',
      score: 0,
      lives: 3,
      masteredList: [],
      wordQueue: [],
      activeScrolls: [],
      splitHalves: [],
      bladeTrail: [],
      particles: [],
      popups: [],
      spawnCooldown: 0.7,
      lastSwooshTime: 0,

      // Current Active Definition Challenge State
      challenge: null
    };

    function getPoolForSet(setKey) {
      if (setKey === 'all') {
        return [...WORD_SCROLLS.set1, ...WORD_SCROLLS.set2, ...WORD_SCROLLS.set3];
      }
      return WORD_SCROLLS[setKey] || WORD_SCROLLS.set1;
    }

    function updatePhasePills(activeStep) {
      const p1 = document.getElementById('step1Pill');
      const p2 = document.getElementById('step2Pill');
      const p3 = document.getElementById('step3Pill');
      p1.className = `phase-step ${activeStep === 1 ? 'active' : 'done'}`;
      p2.className = `phase-step ${activeStep === 2 ? 'active' : (activeStep > 2 ? 'done' : '')}`;
      p3.className = `phase-step ${activeStep === 3 ? 'active' : ''}`;
    }

    function updateHud() {
      document.getElementById('hudScore').textContent = `⭐ ${game.score}`;
      document.getElementById('hudCombo').textContent = `⚔️ ${game.masteredList.length} Mastered`;
      document.getElementById('hudLives').textContent = '❤️'.repeat(Math.max(0, game.lives)) || '💔';
      document.getElementById('masteredCount').textContent = game.masteredList.length;
    }

    function startDojoSession() {
      initAudio();
      document.getElementById('introModal').style.display = 'none';
      restartDojo();
    }

    function openIntroModal() {
      document.getElementById('introModal').style.display = 'flex';
    }

    function openScrollModal() {
      const grid = document.getElementById('scrollBankGrid');
      grid.innerHTML = '';
      if (!game.masteredList.length) {
        grid.innerHTML = `<p style="color:#A89480; width:100%; text-align:center;">Slice a Word Scroll and define its morphemes to record it here!</p>`;
      } else {
        game.masteredList.forEach(item => {
          const chip = document.createElement('div');
          chip.className = 'scroll-chip';
          const breakdown = item.parts.map(p => `${p.text} (${p.meaning})`).join(' + ');
          chip.innerHTML = `
            <div style="color:#8F2D14; font-size:0.9rem;">✦ ${item.word}</div>
            <div style="font-size:0.72rem; color:#4A3525;">${breakdown}</div>
            <div style="font-size:0.76rem; color:#0B7A4E; margin-top:2px;">= "${item.fullMeaning}"</div>
          `;
          grid.appendChild(chip);
        });
      }
      document.getElementById('scrollModal').style.display = 'flex';
    }

    function changeSet() {
      game.setKey = document.getElementById('curriculumSetSelect').value;
      restartDojo();
    }

    function restartDojo() {
      game.setKey = document.getElementById('curriculumSetSelect').value;
      game.score = 0;
      game.lives = 3;
      game.inDefinitionPhase = false;
      game.activeScrolls = [];
      game.splitHalves = [];
      game.bladeTrail = [];
      game.particles = [];
      game.popups = [];
      game.wordQueue = [...getPoolForSet(game.setKey)].sort(() => Math.random() - 0.5);
      game.spawnCooldown = 0.45;
      game.running = true;

      document.getElementById('definitionOverlay').style.display = 'none';
      document.getElementById('gameOverModal').style.display = 'none';
      updatePhasePills(1);
      updateHud();
      document.getElementById('senseiBanner').innerHTML =
        `Timing Challenge: Swipe across the <strong>cyan morpheme seam</strong> on flying Word Scrolls—avoid <strong>Trickster Bombs</strong>!`;
    }

    function launchNextScrollWave() {
      if (game.wordQueue.length === 0) {
        game.wordQueue = [...getPoolForSet(game.setKey)].sort(() => Math.random() - 0.5);
      }
      const entry = game.wordQueue.shift();
      const w = canvas.width;
      const h = canvas.height;

      // Launch target Word Scroll from bottom upward in a classic Fruit Ninja parabolic arc
      const startX = w * (0.24 + Math.random() * 0.52);
      const targetPeakY = h * (0.20 + Math.random() * 0.16);
      const gravity = 520;
      const riseDist = Math.max(160, (h + 30) - targetPeakY);
      const vy = -Math.sqrt(2 * gravity * riseDist);
      const vx = (w * 0.5 - startX) * (0.55 + Math.random() * 0.35);

      const scrollW = Math.max(156, entry.word.length * 19 + 44);
      const scrollH = 66;

      // Calculate exact morpheme seam X offset inside the scroll
      const firstPartLen = entry.parts[0].text.replace(/-/g, '').length;
      const seamRatio = Math.max(0.25, Math.min(0.75, firstPartLen / entry.word.length));

      game.activeScrolls.push({
        kind: 'word',
        entry,
        x: startX,
        y: h + 35,
        w: scrollW,
        h: scrollH,
        vx,
        vy,
        gravity,
        angle: (Math.random() - 0.5) * 0.25,
        vAngle: (Math.random() - 0.5) * 0.9,
        seamRatio
      });

      // 65% chance to also toss a red Trickster Bomb decoy alongside the scroll!
      if (Math.random() < 0.65) {
        const bombX = startX > w * 0.5 ? w * (0.18 + Math.random() * 0.24) : w * (0.58 + Math.random() * 0.24);
        const bombVy = -Math.sqrt(2 * gravity * (riseDist * (0.82 + Math.random() * 0.15)));
        const bombVx = (w * 0.5 - bombX) * 0.5;
        game.activeScrolls.push({
          kind: 'bomb',
          label: entry.trickster || 'uncle',
          x: bombX,
          y: h + 45,
          r: 36,
          vx: bombVx,
          vy: bombVy,
          gravity,
          angle: 0,
          vAngle: 1.5
        });
      }
    }

    /* ========================================================================
       4. KATANA SWIPE DETECTION & TIMED SEAM ACCURACY
       ======================================================================== */
    let isPointerDown = false;
    let lastBladePt = null;

    function getCanvasPoint(e) {
      const rect = canvas.getBoundingClientRect();
      return {
        x: ((e.clientX - rect.left) / rect.width) * canvas.width,
        y: ((e.clientY - rect.top) / rect.height) * canvas.height
      };
    }

    canvas.addEventListener('pointerdown', (e) => {
      initAudio();
      if (!game.running || game.inDefinitionPhase) return;
      isPointerDown = true;
      const pt = getCanvasPoint(e);
      lastBladePt = pt;
      game.bladeTrail.push({ x: pt.x, y: pt.y, life: 0.22 });
    });

    canvas.addEventListener('pointermove', (e) => {
      if (!isPointerDown || !game.running || game.inDefinitionPhase) return;
      const pt = getCanvasPoint(e);
      game.bladeTrail.push({ x: pt.x, y: pt.y, life: 0.22 });

      if (lastBladePt) {
        const dist = Math.hypot(pt.x - lastBladePt.x, pt.y - lastBladePt.y);
        if (dist > 14) {
          const now = performance.now();
          if (now - game.lastSwooshTime > 170) {
            sfx.swoosh();
            game.lastSwooshTime = now;
          }
          checkBladeIntersection(lastBladePt, pt);
          lastBladePt = pt;
        }
      }
    });

    window.addEventListener('pointerup', () => {
      isPointerDown = false;
      lastBladePt = null;
    });

    function segmentIntersectsRect(p1, p2, rx, ry, rw, rh) {
      // Sample 8 points along the blade segment for responsive touch & mouse slicing
      for (let i = 0; i <= 8; i++) {
        const t = i / 8;
        const sx = p1.x + (p2.x - p1.x) * t;
        const sy = p1.y + (p2.y - p1.y) * t;
        if (sx >= rx && sx <= rx + rw && sy >= ry && sy <= ry + rh) {
          return { x: sx, y: sy };
        }
      }
      return null;
    }

    function checkBladeIntersection(p1, p2) {
      for (let i = game.activeScrolls.length - 1; i >= 0; i--) {
        const obj = game.activeScrolls[i];
        if (obj.kind === 'bomb') {
          const hit = segmentIntersectsRect(p1, p2, obj.x - obj.r, obj.y - obj.r, obj.r * 2, obj.r * 2);
          if (hit) {
            game.activeScrolls.splice(i, 1);
            sfx.bomb();
            spawnExplosion(obj.x, obj.y, ['#FF3B5C', '#FFB830', '#FFFFFF'], 28);
            game.popups.push({ x: obj.x, y: obj.y, text: `TRICKSTER BOMB "${obj.label}"! -1 ❤️`, color: '#FF3B5C', life: 1.3 });
            loseLife(`Sliced a Trickster Bomb ("${obj.label}")! Only slice true morphological scrolls!`);
            return;
          }
        } else if (obj.kind === 'word') {
          const rx = obj.x - obj.w / 2;
          const ry = obj.y - obj.h / 2;
          const hit = segmentIntersectsRect(p1, p2, rx, ry, obj.w, obj.h);
          if (hit) {
            // Evaluate timing / seam accuracy!
            const seamX = rx + obj.w * obj.seamRatio;
            const seamDist = Math.abs(hit.x - seamX);
            const cleanCut = seamDist <= 34;

            game.activeScrolls.splice(i, 1);
            sfx.slice();
            spawnExplosion(hit.x, hit.y, ['#38F2FF', '#FAF1DF', '#FFB830'], 24);

            // Create physical flying scroll halves for visual juice
            game.splitHalves.push(
              { text: obj.entry.parts[0].text, x: obj.x - 36, y: obj.y, vx: -130, vy: -120, angle: -0.15, life: 0.7 },
              { text: obj.entry.parts.slice(1).map(p => p.text).join(''), x: obj.x + 36, y: obj.y, vx: 130, vy: -120, angle: 0.15, life: 0.7 }
            );

            // Clear remaining airborne scrolls and enter Bullet-Time Definition Challenge!
            game.activeScrolls = [];
            beginDefinitionChallenge(obj.entry, cleanCut);
            return;
          }
        }
      }
    }

    function loseLife(reason) {
      game.lives = Math.max(0, game.lives - 1);
      updateHud();
      document.getElementById('senseiBanner').innerHTML = `<span style="color:#FF7A90;">${reason}</span>`;
      if (game.lives <= 0) {
        game.running = false;
        document.getElementById('gameOverTitle').textContent = 'DOJO TRAINING ENDED';
        document.getElementById('gameOverStats').innerHTML =
          `You earned <strong>${game.score} Dojo Points</strong> and mastered <strong>${game.masteredList.length} words</strong> by slicing and defining their morphological parts!`;
        document.getElementById('gameOverModal').style.display = 'flex';
      }
    }

    /* ========================================================================
       5. STEP 2 & STEP 3: DEFINE EACH PART -> DEFINE THE FULL WORD
       ======================================================================== */
    function beginDefinitionChallenge(entry, cleanCut) {
      game.inDefinitionPhase = true;
      const sliceBonus = cleanCut ? 50 : 20;
      game.score += sliceBonus;
      updateHud();

      game.challenge = {
        entry,
        cleanCut,
        partIndex: 0,
        solvedMeanings: Array(entry.parts.length).fill(null),
        stage: 'parts' // 'parts' -> 'full'
      };

      const badge = document.getElementById('sliceAccuracyBadge');
      badge.textContent = cleanCut ? `⚡ PERFECT SEAM CUT (+${sliceBonus})` : `⚔️ WORD SPLIT (+${sliceBonus})`;
      badge.style.color = cleanCut ? 'var(--jade-green)' : 'var(--gold-amber)';

      document.getElementById('definitionOverlay').style.display = 'flex';
      renderDefinitionStep();
    }

    function renderDefinitionStep() {
      const ch = game.challenge;
      const entry = ch.entry;

      // Render separated Morpheme Slabs at top of card
      const partsRow = document.getElementById('slicedPartsRow');
      partsRow.innerHTML = '';
      entry.parts.forEach((p, idx) => {
        if (idx > 0) {
          const plus = document.createElement('span');
          plus.className = 'plus-sign';
          plus.textContent = '+';
          partsRow.appendChild(plus);
        }
        const isCurrent = ch.stage === 'parts' && ch.partIndex === idx;
        const isSolved = ch.solvedMeanings[idx] !== null;

        const slab = document.createElement('div');
        slab.className = `morpheme-slab ${isCurrent ? 'active-part' : ''} ${isSolved ? 'solved-part' : ''}`;
        slab.innerHTML = `
          <div class="slab-role">${p.role}</div>
          <div class="slab-text">${p.text}</div>
          <div class="slab-meaning">${isSolved ? `"${ch.solvedMeanings[idx]}"` : '?'}</div>
        `;
        partsRow.appendChild(slab);
      });

      const titleEl = document.getElementById('defStepTitle');
      const promptEl = document.getElementById('defQuestionPrompt');
      const gridEl = document.getElementById('defChoicesGrid');
      gridEl.innerHTML = '';

      if (ch.stage === 'parts') {
        updatePhasePills(2);
        const currentPart = entry.parts[ch.partIndex];
        titleEl.textContent = `STEP 2 OF 3 • DEFINE PART ${ch.partIndex + 1} OF ${entry.parts.length}`;
        promptEl.innerHTML = `What does the ${currentPart.role.toLowerCase()} <span>${currentPart.text}</span> mean?`;
        document.getElementById('senseiBanner').innerHTML =
          `Step 2: Define the sliced ${currentPart.role.toLowerCase()} <strong>${currentPart.text}</strong>!`;

        // Build 4 distinct choices for this morpheme part
        const options = buildPartOptions(currentPart.meaning, entry);
        options.forEach(optText => {
          const btn = document.createElement('button');
          btn.className = 'choice-btn';
          btn.textContent = optText;
          btn.onclick = () => handlePartAnswer(btn, optText === currentPart.meaning);
          gridEl.appendChild(btn);
        });
      } else {
        // STEP 3: DEFINE THE FULL COMBINED WORD
        updatePhasePills(3);
        titleEl.textContent = `STEP 3 OF 3 • DEFINE THE FULL WORD`;
        promptEl.innerHTML = `Combine the parts! What does the full word <span>${entry.word}</span> mean?`;
        document.getElementById('senseiBanner').innerHTML =
          `Step 3: Combine the part meanings to define <strong>${entry.word}</strong>!`;

        const fullOptions = [entry.fullMeaning, ...entry.distractorFull].sort(() => Math.random() - 0.5);
        fullOptions.forEach(optText => {
          const btn = document.createElement('button');
          btn.className = 'choice-btn';
          btn.textContent = optText;
          btn.onclick = () => handleFullWordAnswer(btn, optText === entry.fullMeaning);
          gridEl.appendChild(btn);
        });
      }
    }

    function buildPartOptions(correctMeaning, entry) {
      const pool = new Set([correctMeaning]);
      // Pull meanings from other parts in the set as plausible distractors
      const allEntries = getPoolForSet(game.setKey);
      const candidateMeanings = [];
      allEntries.forEach(e => e.parts.forEach(p => {
        if (p.meaning !== correctMeaning) candidateMeanings.push(p.meaning);
      }));
      EXTRA_MEANING_DISTRACTORS.forEach(m => {
        if (m !== correctMeaning) candidateMeanings.push(m);
      });

      const shuffled = candidateMeanings.sort(() => Math.random() - 0.5);
      for (const m of shuffled) {
        if (pool.size >= 4) break;
        pool.add(m);
      }
      return Array.from(pool).sort(() => Math.random() - 0.5);
    }

    function handlePartAnswer(btnEl, isCorrect) {
      if (!isCorrect) {
        btnEl.classList.add('wrong');
        btnEl.disabled = true;
        sfx.wrong();
        game.score = Math.max(0, game.score - 10);
        updateHud();
        return;
      }

      btnEl.classList.add('correct');
      sfx.partCorrect();
      const ch = game.challenge;
      ch.solvedMeanings[ch.partIndex] = ch.entry.parts[ch.partIndex].meaning;
      game.score += 30;
      updateHud();

      setTimeout(() => {
        if (ch.partIndex + 1 < ch.entry.parts.length) {
          ch.partIndex++;
          renderDefinitionStep();
        } else {
          ch.stage = 'full';
          renderDefinitionStep();
        }
      }, 280);
    }

    function handleFullWordAnswer(btnEl, isCorrect) {
      if (!isCorrect) {
        btnEl.classList.add('wrong');
        btnEl.disabled = true;
        sfx.wrong();
        game.score = Math.max(0, game.score - 15);
        updateHud();
        return;
      }

      btnEl.classList.add('correct');
      sfx.wordMastered();
      const entry = game.challenge.entry;
      game.score += 75;

      if (!game.masteredList.some(m => m.word === entry.word)) {
        game.masteredList.push(entry);
      }
      updateHud();

      setTimeout(() => {
        document.getElementById('definitionOverlay').style.display = 'none';
        game.inDefinitionPhase = false;
        game.challenge = null;
        game.spawnCooldown = 0.55;
        updatePhasePills(1);
        document.getElementById('senseiBanner').innerHTML =
          `Mastered <strong style="color:var(--jade-green);">${entry.word}</strong> ("${entry.fullMeaning}")! Ready for the next Word Scroll!`;
      }, 420);
    }

    /* ========================================================================
       6. PHYSICS & DOJO RENDERER
       ======================================================================== */
    function spawnExplosion(x, y, colors, count = 18) {
      for (let i = 0; i < count; i++) {
        const a = Math.random() * Math.PI * 2;
        const spd = 60 + Math.random() * 220;
        game.particles.push({
          x, y,
          vx: Math.cos(a) * spd,
          vy: Math.sin(a) * spd,
          r: 3 + Math.random() * 4,
          color: colors[i % colors.length],
          life: 0.45 + Math.random() * 0.3
        });
      }
    }

    function updateSimulation(dt) {
      // Fade blade trail
      for (let i = game.bladeTrail.length - 1; i >= 0; i--) {
        game.bladeTrail[i].life -= dt;
        if (game.bladeTrail[i].life <= 0) game.bladeTrail.splice(i, 1);
      }

      // Particles & floating callouts
      for (let i = game.particles.length - 1; i >= 0; i--) {
        const p = game.particles[i];
        p.x += p.vx * dt;
        p.y += p.vy * dt;
        p.vy += 360 * dt;
        p.life -= dt;
        if (p.life <= 0) game.particles.splice(i, 1);
      }
      for (let i = game.popups.length - 1; i >= 0; i--) {
        game.popups[i].y -= 35 * dt;
        game.popups[i].life -= dt;
        if (game.popups[i].life <= 0) game.popups.splice(i, 1);
      }

      // Sliced halves falling away
      for (let i = game.splitHalves.length - 1; i >= 0; i--) {
        const h = game.splitHalves[i];
        h.x += h.vx * dt;
        h.y += h.vy * dt;
        h.vy += 450 * dt;
        h.life -= dt;
        if (h.life <= 0) game.splitHalves.splice(i, 1);
      }

      if (!game.running || game.inDefinitionPhase) return;

      // Spawn airborne scrolls when arena is empty
      if (game.activeScrolls.length === 0) {
        game.spawnCooldown -= dt;
        if (game.spawnCooldown <= 0) {
          launchNextScrollWave();
          game.spawnCooldown = 1.1;
        }
      }

      // Update airborne scrolls & bombs
      for (let i = game.activeScrolls.length - 1; i >= 0; i--) {
        const obj = game.activeScrolls[i];
        obj.x += obj.vx * dt;
        obj.y += obj.vy * dt;
        obj.vy += obj.gravity * dt;
        obj.angle += obj.vAngle * dt;

        // If a target Word Scroll falls off the bottom unsliced, lose 1 life
        if (obj.y > canvas.height + 85 && obj.vy > 0) {
          game.activeScrolls.splice(i, 1);
          if (obj.kind === 'word') {
            loseLife(`Missed the scroll "${obj.entry.word}"! Swipe before it drops!`);
          }
        }
      }
    }

    function roundRect(x, y, w, h, r) {
      ctx.beginPath();
      ctx.moveTo(x + r, y);
      ctx.arcTo(x + w, y, x + w, y + h, r);
      ctx.arcTo(x + w, y + h, x, y + h, r);
      ctx.arcTo(x, y + h, x, y, r);
      ctx.arcTo(x, y, x + w, y, r);
      ctx.closePath();
    }

    function renderFrame() {
      const vp = document.getElementById('dojoArena').getBoundingClientRect();
      if (canvas.width !== Math.floor(vp.width) || canvas.height !== Math.floor(vp.height)) {
        canvas.width = Math.max(320, Math.floor(vp.width));
        canvas.height = Math.max(240, Math.floor(vp.height));
      }

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Subtle Dojo Bamboo & Tatami Wood Planks Background
      ctx.strokeStyle = 'rgba(255, 184, 48, 0.05)';
      ctx.lineWidth = 2;
      for (let x = 80; x < canvas.width; x += 140) {
        ctx.beginPath();
        ctx.moveTo(x, 0);
        ctx.lineTo(x, canvas.height);
        ctx.stroke();
      }

      // Faint Center Dojo Crest
      ctx.save();
      ctx.strokeStyle = 'rgba(250, 241, 223, 0.04)';
      ctx.lineWidth = 6;
      ctx.beginPath();
      ctx.arc(canvas.width / 2, canvas.height / 2, Math.min(canvas.width, canvas.height) * 0.28, 0, Math.PI * 2);
      ctx.stroke();
      ctx.restore();

      // Render Active Flying Word Scrolls & Trickster Bombs
      for (const obj of game.activeScrolls) {
        ctx.save();
        ctx.translate(obj.x, obj.y);
        ctx.rotate(obj.angle);

        if (obj.kind === 'word') {
          // Bamboo Roll Ends
          ctx.fillStyle = '#8C5832';
          roundRect(-obj.w / 2 - 8, -obj.h / 2 - 4, 12, obj.h + 8, 5);
          ctx.fill();
          roundRect(obj.w / 2 - 4, -obj.h / 2 - 4, 12, obj.h + 8, 5);
          ctx.fill();

          // Parchment Scroll Body
          ctx.fillStyle = '#FAF1DF';
          ctx.strokeStyle = '#FFB830';
          ctx.lineWidth = 3;
          roundRect(-obj.w / 2, -obj.h / 2, obj.w, obj.h, 8);
          ctx.fill();
          ctx.stroke();

          // Glowing Morpheme Cut-Line Target Seam
          const seamX = -obj.w / 2 + obj.w * obj.seamRatio;
          ctx.strokeStyle = '#0891B2';
          ctx.lineWidth = 2.5;
          ctx.setLineDash([5, 4]);
          ctx.beginPath();
          ctx.moveTo(seamX, -obj.h / 2 - 8);
          ctx.lineTo(seamX, obj.h / 2 + 8);
          ctx.stroke();
          ctx.setLineDash([]);

          // Word Typography
          ctx.fillStyle = '#241911';
          ctx.font = '800 24px "JetBrains Mono", monospace';
          ctx.textAlign = 'center';
          ctx.textBaseline = 'middle';
          ctx.fillText(obj.entry.word, 0, 1);
        } else {
          // Crimson Trickster Bomb
          ctx.fillStyle = '#1C0D12';
          ctx.strokeStyle = '#FF3B5C';
          ctx.lineWidth = 3.5;
          ctx.beginPath();
          ctx.arc(0, 0, obj.r, 0, Math.PI * 2);
          ctx.fill();
          ctx.stroke();

          // Fuse Spark
          ctx.strokeStyle = '#FFB830';
          ctx.lineWidth = 3;
          ctx.beginPath();
          ctx.moveTo(0, -obj.r);
          ctx.quadraticCurveTo(10, -obj.r - 12, 16, -obj.r - 8);
          ctx.stroke();

          ctx.fillStyle = '#FFB3C0';
          ctx.font = '800 13px "JetBrains Mono", monospace';
          ctx.textAlign = 'center';
          ctx.textBaseline = 'middle';
          ctx.fillText(obj.label, 0, 1);
        }
        ctx.restore();
      }

      // Render Sliced Scroll Halves
      for (const h of game.splitHalves) {
        ctx.save();
        ctx.globalAlpha = Math.max(0, h.life / 0.7);
        ctx.translate(h.x, h.y);
        ctx.rotate(h.angle);
        ctx.fillStyle = '#FAF1DF';
        ctx.strokeStyle = '#38F2FF';
        ctx.lineWidth = 2.5;
        roundRect(-48, -24, 96, 48, 8);
        ctx.fill();
        ctx.stroke();
        ctx.fillStyle = '#241911';
        ctx.font = '800 18px "JetBrains Mono", monospace';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText(h.text, 0, 0);
        ctx.restore();
      }

      // Particles
      for (const p of game.particles) {
        ctx.fillStyle = p.color;
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
        ctx.fill();
      }

      // Popups
      for (const pop of game.popups) {
        ctx.save();
        ctx.font = '900 16px "Nunito", sans-serif';
        ctx.textAlign = 'center';
        ctx.strokeStyle = '#160F0B';
        ctx.lineWidth = 4;
        ctx.strokeText(pop.text, pop.x, pop.y);
        ctx.fillStyle = pop.color;
        ctx.fillText(pop.text, pop.x, pop.y);
        ctx.restore();
      }

      // Katana Blade Slash Trail
      if (game.bladeTrail.length > 1) {
        ctx.save();
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';
        for (let i = 1; i < game.bladeTrail.length; i++) {
          const prev = game.bladeTrail[i - 1];
          const curr = game.bladeTrail[i];
          const alpha = Math.max(0, curr.life / 0.22);
          ctx.strokeStyle = `rgba(56, 242, 255, ${alpha})`;
          ctx.shadowColor = '#38F2FF';
          ctx.shadowBlur = 14;
          ctx.lineWidth = 10 * alpha;
          ctx.beginPath();
          ctx.moveTo(prev.x, prev.y);
          ctx.lineTo(curr.x, curr.y);
          ctx.stroke();
        }
        ctx.restore();
      }
    }

    /* ========================================================================
       7. MAIN LOOP BOOTSTRAP
       ======================================================================== */
    let lastTs = 0;
    function gameLoop(ts) {
      const dt = Math.min(0.05, (ts - lastTs) / 1000 || 0.016);
      lastTs = ts;
      updateSimulation(dt);
      renderFrame();
      requestAnimationFrame(gameLoop);
    }

    window.onload = function () {
      updateHud();
      requestAnimationFrame(gameLoop);
    };
  </script>
</body>
</html>
