<script>
    // @ts-nocheck

    import { onMount, onDestroy } from 'svelte';

    // --- Variables Svelte ---
    let audioRefLeft;
    let audioRefRight;
    let canvasRef;
    
    // Variables pour les inputs de fichiers
    let fileInputRefLeft; 
    let fileInputRefRight;
    
    // Sources audio d'état
    let currentAudioSrcLeft = null; 
    let currentAudioSrcRight = null;
    let isInitializedLeft = false; 
    let isInitializedRight = false; 
    
    // NOUVELLE VARIABLE: Indique si l'utilisateur a pressé le bouton Start
    let hasStarted = false;

    // Toggle sonore
    let isLeftPlaying = true; 
    
    // VARIABLES DE SCORE
    let scoreLeft = 0; // Score pour le joueur de gauche (but marqué à droite)
    let scoreRight = 0; // Score pour le joueur de droite (but marqué à gauche)

    // --- Variables Audio/Analyse GLOBALES ---
    let audioCtx; 
    let analyserLeft;
    let sourceLeft;
    let analyserRight;
    let sourceRight;

    // Tableaux de données pour l'analyse
    let dataArrayLeft;
    let bufferLengthLeft;
    let dataArrayRight;
    let bufferLengthRight;
    
    let animationFrameId;

    const FFT_SIZE = 512;

    // --- Variables Balle de Rebond ---
    let ball = {
        x: 300,
        y: 100,
        radius: 10,
        vx: 3,
        vy: 2.5,
        color: 'red'
    };
    
    // --- CONSTANTES DE LIGNES DE BUT MODIFIÉES ---
    const GOAL_LINE_RATIO_LEFT = 1 / 5; 
    const GOAL_LINE_RATIO_RIGHT = 4 / 5; 
    const CENTER_X = 300; 
    const CENTER_Y = 100; 
    // ---------------------------------------------
    
    // Variables pour le dessin des barres (nécessaires pour le calcul de collision)
    const barCount = 10;
    let barHeightSpace = 0;
    let gap = 0;
    
    // Facteur d'amplitude (sensibilité)
    const AMPLITUDE_FACTOR = 0.5; 
    
    // Facteur de réduction de la longueur physique maximale
    const REDUCTION_FACTOR = 0.5; 
    
    let PIXEL_NORMALIZER = 0; 

    // --- LOGIQUE AUDIO ---

    /**
     * Connecte ou déconnecte un nœud source du AudioContext.
     */
    function toggleAudioOutput(side, connect) {
        if (!audioCtx) return;

        const source = side === 'left' ? sourceLeft : sourceRight;
        const analyser = side === 'left' ? analyserLeft : analyserRight;

        if (source && analyser) {
            source.disconnect();
            analyser.disconnect();
            
            source.connect(analyser); 
            
            if (connect) {
                analyser.connect(audioCtx.destination); 
            }
        }
    }

    /**
     * Met à jour la sortie audio en fonction de l'état du toggle `isLeftPlaying`.
     */
    function updateAudioOutput() {
        if (!hasStarted) return; 

        if (isLeftPlaying) {
            toggleAudioOutput('left', true);
            toggleAudioOutput('right', false);
        } else {
            toggleAudioOutput('left', false);
            toggleAudioOutput('right', true);
        }
    }

    /**
     * Initialise l'API Web Audio et connecte les nœuds pour un côté donné.
     */
    function initAudio(side) {
        const audioRef = side === 'left' ? audioRefLeft : audioRefRight;
        
        if (!audioRef) {
            console.error(`L'élément <audio> pour le côté ${side} n'est pas prêt.`);
            return;
        }

        if (!audioCtx) {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        }

        if (audioCtx.state === 'suspended') {
            audioCtx.resume();
        }

        try {
            if (side === 'left' && sourceLeft) { sourceLeft.disconnect(); analyserLeft?.disconnect(); }
            if (side === 'right' && sourceRight) { sourceRight.disconnect(); analyserRight?.disconnect(); }

            const analyser = audioCtx.createAnalyser();
            analyser.fftSize = FFT_SIZE;
            const bufferLength = analyser.frequencyBinCount;
            const dataArray = new Uint8Array(bufferLength);
            const source = audioCtx.createMediaElementSource(audioRef);

            if (side === 'left') {
                analyserLeft = analyser;
                bufferLengthLeft = bufferLength;
                dataArrayLeft = dataArray;
                sourceLeft = source;
                isInitializedLeft = true;
            } else {
                analyserRight = analyser;
                bufferLengthRight = bufferLength;
                dataArrayRight = dataArray;
                sourceRight = source;
                isInitializedRight = true;
            }
            
            updateAudioOutput();

            if (!animationFrameId) {
                draw();
            }

        } catch (error) {
            console.error(`Erreur lors de l'initialisation de l'AudioContext pour le côté ${side}:`, error);
            if (side === 'left') isInitializedLeft = false;
            if (side === 'right') isInitializedRight = false;
        }
    }

    /**
     * Gère la sélection du fichier par l'utilisateur.
     */
    function handleFileSelect(event, side) {
        const file = event.target.files[0];
        if (file) {
            const oldSrc = side === 'left' ? currentAudioSrcLeft : currentAudioSrcRight;
            if (oldSrc) { URL.revokeObjectURL(oldSrc); }
            
            const url = URL.createObjectURL(file);
            if (side === 'left') {
                currentAudioSrcLeft = url;
            } else {
                currentAudioSrcRight = url;
            }
            
            if (side === 'left') isInitializedLeft = false;
            if (side === 'right') isInitializedRight = false;

            initAudio(side);
        }
    }

    /**
     * DÉMARRE la lecture et la visualisation une fois que les deux sont prêts.
     */
    function startVisualization() {
        if (!currentAudioSrcLeft || !currentAudioSrcRight) {
            console.warn("Les deux fichiers audio ne sont pas chargés.");
            return;
        }
        
        if (!isInitializedLeft) initAudio('left');
        if (!isInitializedRight) initAudio('right');

        // Réinitialiser le score au début d'un nouveau jeu
        scoreLeft = 0; 
        scoreRight = 0;

        if (audioRefLeft) {
            audioRefLeft.play().catch(e => console.error("Échec de la lecture audio gauche:", e));
        }
        if (audioRefRight) {
            audioRefRight.play().catch(e => console.error("Échec de la lecture audio droite:", e));
        }
        
        hasStarted = true;
        updateAudioOutput();
        
        if (!animationFrameId) {
            draw();
        }
    }
    
    // --- LOGIQUE VISUELLE ET COLLISION ---
    
    /**
     * Vérifie la collision de la balle avec la longueur actuelle de l'une des barres.
     */
    function checkBarCollision(WIDTH) {
        // --- Côté Gauche ---
        if (isInitializedLeft && dataArrayLeft && ball.vx < 0) { // Balle va vers la gauche
            const barIndex = Math.floor(ball.y / (barHeightSpace + gap));
            
            if (barIndex >= 0 && barIndex < barCount) {
                const currentBarLength = dataArrayLeft[barIndex] / AMPLITUDE_FACTOR * PIXEL_NORMALIZER;

                // Collision en X pour la barre GAUCHE: [0, currentBarLength]
                if (ball.x - ball.radius <= currentBarLength) {
                    if (ball.x >= 0) {
                        ball.vx = -ball.vx;
                        ball.x = currentBarLength + ball.radius; // Repositionnement
                    }
                }
            }
        }
        
        // --- Côté Droit ---
        if (isInitializedRight && dataArrayRight && ball.vx > 0) { // Balle va vers la droite
            const barIndex = Math.floor(ball.y / (barHeightSpace + gap));
            
            if (barIndex >= 0 && barIndex < barCount) {
                const currentBarLength = dataArrayRight[barIndex] / AMPLITUDE_FACTOR * PIXEL_NORMALIZER;
                
                // Collision en X pour la barre DROITE: [WIDTH - currentBarLength, WIDTH]
                const barStart = WIDTH - currentBarLength;

                if (ball.x + ball.radius >= barStart) {
                    if (ball.x <= WIDTH) {
                        ball.vx = -ball.vx;
                        ball.x = barStart - ball.radius; // Repositionnement
                    }
                }
            }
        }
    }

    /**
     * Met à jour la position de la balle et vérifie les collisions.
     */
    function updateBall(WIDTH, HEIGHT) {
        if (!hasStarted) return; 

        // 1. Mettre à jour la position
        ball.x += ball.vx;
        ball.y += ball.vy;
        
        // --- LOGIQUE DE LIGNE DE BUT GAUCHE ---
        const GOAL_LINE_LEFT_X = WIDTH * GOAL_LINE_RATIO_LEFT;
        
        // Si la balle franchit la ligne de but GAUCHE vers la gauche
        if (ball.x - ball.radius < GOAL_LINE_LEFT_X && ball.vx < 0) {
            // INCLEMENTATION DU SCORE pour le côté DROIT
            scoreRight += 1; 
            
            // Repositionner la balle au centre
            ball.x = CENTER_X; 
            ball.y = CENTER_Y;
            // Inverser la direction pour relancer le jeu
            ball.vx = -ball.vx; 
            console.log("But marqué à GAUCHE ! Score Droit:", scoreRight);
        }
        // --------------------------------------
        
        // --- LOGIQUE DE LIGNE DE BUT DROITE ---
        const GOAL_LINE_RIGHT_X = WIDTH * GOAL_LINE_RATIO_RIGHT;
        
        // Si la balle franchit la ligne de but DROITE vers la droite
        if (ball.x + ball.radius > GOAL_LINE_RIGHT_X && ball.vx > 0) {
            // INCLEMENTATION DU SCORE pour le côté GAUCHE
            scoreLeft += 1;
            
            // Repositionner la balle au centre
            ball.x = CENTER_X; 
            ball.y = CENTER_Y;
            // Inverser la direction pour relancer le jeu
            ball.vx = -ball.vx; 
            console.log("But marqué à DROITE ! Score Gauche:", scoreLeft);
        }
        // --------------------------------------

        // 2. Comportement Horizontal (Collision des barres)
        checkBarCollision(WIDTH);
        
        // 3. Comportement Vertical (Haut/Bas) : Rebond
        if (ball.y + ball.radius > HEIGHT || ball.y - ball.radius < 0) {
            ball.vy = -ball.vy;
            if (ball.y + ball.radius > HEIGHT) ball.y = HEIGHT - ball.radius;
            if (ball.y - ball.radius < 0) ball.y = ball.radius;
        }
    }

    function drawBall(ctx) {
        ctx.fillStyle = ball.color;
        ctx.beginPath();
        ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
        ctx.fill();
    }
    
    function drawGoalLine(ctx, WIDTH, HEIGHT) {
        const GOAL_LINE_LEFT_X = WIDTH * GOAL_LINE_RATIO_LEFT;
        const GOAL_LINE_RIGHT_X = WIDTH * GOAL_LINE_RATIO_RIGHT;
        
        ctx.strokeStyle = '#FF5733'; // Couleur orange/rouge
        ctx.lineWidth = 2;
        ctx.setLineDash([5, 5]); // Ligne pointillée
        
        // Ligne GAUCHE
        ctx.beginPath();
        ctx.moveTo(GOAL_LINE_LEFT_X, 0);
        ctx.lineTo(GOAL_LINE_LEFT_X, HEIGHT);
        ctx.stroke();
        
        // Ligne DROITE 
        ctx.beginPath();
        ctx.moveTo(GOAL_LINE_RIGHT_X, 0);
        ctx.lineTo(GOAL_LINE_RIGHT_X, HEIGHT);
        ctx.stroke();
        
        ctx.setLineDash([]); // Réinitialiser pour ne pas affecter les autres dessins
        ctx.lineWidth = 1;
    }


    /**
     * Boucle de dessin unique.
     */
    function draw() {
        if (!canvasRef) {
             animationFrameId = requestAnimationFrame(draw);
             return;
        }

        animationFrameId = requestAnimationFrame(draw);

        const canvas = canvasRef;
        const ctx = canvas.getContext('2d');
        const WIDTH = canvas.width;
        const HEIGHT = canvas.height;
        const HALF_WIDTH = WIDTH / 2;
        
        barHeightSpace = (HEIGHT / barCount) * 0.9;
        gap = (HEIGHT / barCount) * 0.1;
        PIXEL_NORMALIZER = (HALF_WIDTH / 256) * REDUCTION_FACTOR; 

        // 1. Nettoyage du Canvas 
        ctx.fillStyle = 'rgb(0, 0, 0)';
        ctx.fillRect(0, 0, WIDTH, HEIGHT);

        // --- Mise à jour de la balle ---
        updateBall(WIDTH, HEIGHT);
        // ------------------------------
        
        // 2. Fonctions de Dessin des Barres
        function drawSide(dataArray, direction) {
            let y = 0; 

            for(let i = 0; i < barCount; i++) {
                if (!dataArray) break;
                
                const barLength = dataArray[i] / AMPLITUDE_FACTOR * PIXEL_NORMALIZER;
                
                const r = barLength + (25 * (i/barCount));
                const g = 50 * (i/barCount);
                const b = 200 - barLength;

                ctx.fillStyle = `rgb(${r},${g},${b})`;
                
                let x;
                
                if (direction === 1) {
                    x = 0; // Côté gauche
                } else { // direction === -1
                    x = WIDTH - barLength; // Côté droit (collé à droite)
                }
                
                ctx.fillRect(x, y, barLength, barHeightSpace);

                y += barHeightSpace + gap;
            }
        }

        // 3. Dessin des deux côtés
        if (isInitializedLeft && analyserLeft && dataArrayLeft) {
            analyserLeft.getByteFrequencyData(dataArrayLeft);
            drawSide(dataArrayLeft, 1); 
        } else {
            ctx.fillStyle = '#333';
            ctx.fillText('Charger audio GAUCHE', HALF_WIDTH / 4 - 50, HEIGHT / 2);
        }

        if (isInitializedRight && analyserRight && dataArrayRight) {
            analyserRight.getByteFrequencyData(dataArrayRight);
            drawSide(dataArrayRight, -1); 
        } else {
            ctx.fillStyle = '#333';
            ctx.fillText('Charger audio DROIT', WIDTH - HALF_WIDTH / 4 - 50, HEIGHT / 2);
        }
        
        // --- Dessin de la ligne de but ---
        drawGoalLine(ctx, WIDTH, HEIGHT);
        // ---------------------------------

        // --- Dessin de la balle ---
        drawBall(ctx);
        // -------------------------
    }
    
    /**
     * Nettoyage des ressources.
     */
    function cleanup() {
        if (animationFrameId) {
            cancelAnimationFrame(animationFrameId);
            animationFrameId = null;
        }
        if (audioCtx) {
            audioCtx.close();
            audioCtx = null;
        }

        if (currentAudioSrcLeft) URL.revokeObjectURL(currentAudioSrcLeft);
        if (currentAudioSrcRight) URL.revokeObjectURL(currentAudioSrcRight);

        isInitializedLeft = false;
        isInitializedRight = false;
    }

    onDestroy(() => {
        cleanup();
    });
</script>

---

<div class="visualizer-container">
    
    <div class="file-inputs">
        <label for="audio-file-input-left" class="file-label">
            🎧 Audio GAUCHE {currentAudioSrcLeft ? '✅' : ''}
        </label>
        <input 
            id="audio-file-input-left"
            type="file" 
            accept="audio/*" 
            bind:this={fileInputRefLeft}
            on:change={(e) => handleFileSelect(e, 'left')}
        />

        <label for="audio-file-input-right" class="file-label">
            🎶 Audio DROIT {currentAudioSrcRight ? '✅' : ''}
        </label>
        <input 
            id="audio-file-input-right"
            type="file" 
            accept="audio/*" 
            bind:this={fileInputRefRight}
            on:change={(e) => handleFileSelect(e, 'right')}
        />
    </div>

    {#if currentAudioSrcLeft && currentAudioSrcRight && !hasStarted}
        <button class="start-button" on:click={startVisualization}>
            ▶️ Démarrer la Visualisation
        </button>
    {/if}

    {#if hasStarted}
        <div class="controls-container">
            <div class="audio-toggle">
                <label class="switch-label">
                    <span class="audio-text left-text" class:active={isLeftPlaying}>GAUCHE</span>
                    <input 
                        type="checkbox" 
                        bind:checked={isLeftPlaying}
                        on:change={updateAudioOutput}
                    >
                    <span class="slider round"></span>
                    <span class="audio-text right-text" class:active={!isLeftPlaying}>DROIT</span>
                </label>
            </div>
        </div>
    {/if}

    <div class="audio-players" class:hidden={!hasStarted}>
        {#if currentAudioSrcLeft}
            <audio 
                src={currentAudioSrcLeft} 
                bind:this={audioRefLeft} 
                loop
                controls={hasStarted}
                on:play={() => audioCtx?.resume()} 
            ></audio>
        {/if}

        {#if currentAudioSrcRight}
            <audio 
                src={currentAudioSrcRight} 
                bind:this={audioRefRight} 
                loop
                controls={hasStarted}
                on:play={() => audioCtx?.resume()} 
            ></audio>
        {/if}
    </div>
    
    <canvas 
        bind:this={canvasRef} 
        width="600" 
        height="200"
    ></canvas>
    
    {#if hasStarted}
        <div class="score-display">
            <span class="score-values">
                <span class="score-left">{scoreLeft}</span>
                <span class="score-separator">-</span>
                <span class="score-right">{scoreRight}</span>
            </span>
        </div>
    {/if}

    
    {#if !currentAudioSrcLeft || !currentAudioSrcRight || !hasStarted}
        <div class="placeholder">
            {#if !currentAudioSrcLeft || !currentAudioSrcRight}
                **Charger les deux fichiers audio** ci-dessus pour activer le bouton de démarrage.
            {:else}
                **Appuyez sur "Démarrer"** pour commencer le jeu de barres musicales ! (Score initialisé à 0)
            {/if}
        </div>
    {/if}
</div>

<style>
    /* ----------------------------------- */
    /* --- Styles Généraux & Audio --- */
    /* ----------------------------------- */
    .visualizer-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 15px;
        background: #1e1e1e;
        padding: 20px;
        border-radius: 8px;
    }

    .file-inputs {
        display: flex;
        gap: 20px;
    }

    .audio-players {
        display: flex;
        gap: 10px;
        width: 100%;
        max-width: 600px;
        justify-content: space-between;
        height: auto;
        opacity: 1;
        transition: opacity 0.3s;
    }

    .audio-players.hidden {
        height: 0;
        opacity: 0;
        overflow: hidden;
        margin: 0;
    }

    .file-label {
        background-color: #007bff;
        color: white;
        padding: 10px 20px;
        border-radius: 5px;
        cursor: pointer;
        transition: background-color 0.2s;
        font-weight: bold;
    }

    .file-label:hover {
        background-color: #0056b3;
    }

    input[type="file"] {
        display: none;
    }

    canvas {
        background: #000;
        border: 2px solid #555;
        max-width: 100%;
        height: auto;
    }

    audio {
        flex-grow: 1; 
        max-width: calc(50% - 5px);
    }
    
    .placeholder {
        color: #999;
        border: 2px dashed #444;
        padding: 50px;
        width: 600px;
        text-align: center;
        max-width: 100%;
        margin-top: 10px;
    }

    /* ----------------------------------- */
    /* --- Styles Bouton Start & Score --- */
    /* ----------------------------------- */
    .start-button {
        padding: 12px 30px;
        background-color: #28a745;
        color: white;
        border: none;
        border-radius: 5px;
        font-size: 1.2em;
        cursor: pointer;
        transition: background-color 0.3s, transform 0.1s;
        font-weight: bold;
    }

    .start-button:hover {
        background-color: #218838;
    }

    .start-button:active {
        transform: scale(0.98);
    }
    
    .controls-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 10px;
    }

    /* MODIFICATION ICI pour centrer et simplifier le score */
    .score-display {
        display: flex;
        justify-content: center; /* CENTRAGE HORIZONTAL */
        align-items: center;
        /* width: 600px; <- Retiré pour que le conteneur s'adapte au contenu */
        max-width: 100%;
        margin-top: 15px;
        padding: 10px;
        border: 1px solid #333;
        border-radius: 5px;
        background-color: #222;
    }
    
    .score-values {
        font-size: 2.5em;
        font-weight: 900;
        color: white;
        text-shadow: 0 0 5px rgba(255, 255, 255, 0.5);
    }
    
    .score-left {
        color: #ff5733; 
    }
    
    .score-right {
        color: #4CAF50; 
    }

    .score-separator {
        margin: 0 15px;
        opacity: 0.7;
    }


    /* ----------------------------------- */
    /* --- Styles du Toggle (Switch) --- */
    /* ----------------------------------- */
    .audio-toggle {
        display: flex;
        align-items: center;
    }

    .switch-label {
        position: relative;
        display: inline-block;
        width: 150px; 
        height: 34px;
    }

    .switch-label input { 
        opacity: 0;
        width: 0;
        height: 0;
    }

    .slider {
        position: absolute;
        cursor: pointer;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background-color: #ccc;
        transition: .4s;
        border-radius: 34px;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }

    .slider:before {
        position: absolute;
        content: "";
        height: 26px;
        width: 26px;
        left: 4px;
        bottom: 4px;
        background-color: white;
        transition: .4s;
        border-radius: 50%;
    }

    input:checked + .slider {
        background-color: #2196F3;
    }

    input:checked + .slider:before {
        transform: translateX(116px); 
    }

    .audio-text {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        font-weight: bold;
        color: #555;
        transition: color 0.4s;
        font-size: 14px;
    }

    .left-text {
        left: 10px;
    }

    .right-text {
        right: 10px;
    }
    
    .audio-text.active {
        color: white; 
    }
</style>