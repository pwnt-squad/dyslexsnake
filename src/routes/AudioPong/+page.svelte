<script>
    // @ts-nocheck

    import { onMount, onDestroy } from 'svelte';

    let audioRefLeft;
    let audioRefRight;
    let canvasRef;
    
    // Variables pour les inputs de fichiers
    let fileInputRefLeft; 
    let fileInputRefRight;
    
    // Noms de fichiers par défaut
    let fileNameLeft = "Sélectionner l'audio gauche"; 
    let fileNameRight = "Sélectionner l'audio droit"; 
    
    // Sources audio d'état
    let currentAudioSrcLeft = null; 
    let currentAudioSrcRight = null;
    let isInitializedLeft = false; 
    let isInitializedRight = false; 
    
    // État du jeu
    let hasStarted = false;
    let gameEnded = false; // NOUVEAU: Pour signaler que le jeu est terminé et afficher l'écran de fin
    
    // Toggle sonore
    let isLeftPlaying = false; 
    
    // VARIABLES DE SCORE
    let scoreLeft = 0; 
    let scoreRight = 0; 

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
    const CANVAS_WIDTH = 600;
    const CANVAS_HEIGHT = 300;


    // --- Variables Balle de Rebond ---
    let ball = {
        x: CANVAS_WIDTH / 2, // Centre
        y: CANVAS_HEIGHT / 2, // Centre
        radius: 10,
        vx: 3,
        vy: 2.5,
        color: '#FFFFFF'
    };
    
    // --- CONSTANTES DE LIGNES DE BUT ---
    const GOAL_LINE_RATIO_LEFT = 1 / 5; 
    const GOAL_LINE_RATIO_RIGHT = 4 / 5; 
    const CENTER_X = CANVAS_WIDTH / 2; 
    const CENTER_Y = CANVAS_HEIGHT / 2; 
    // ---------------------------------------------
    
    // Variables pour le dessin des barres 
    const barCount = 12;
    let barHeightSpace = 0;
    let gap = 0;
    
    // Facteur d'amplitude
    const AMPLITUDE_FACTOR = 0.9; 
    
    // Facteur de réduction de la longueur physique maximale
    const REDUCTION_FACTOR = 0.5; 
    
    let PIXEL_NORMALIZER = 0; 

    // --- LOGIQUE AUDIO ---

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

    function updateAudioOutput() {
        if (!hasStarted) return; 

        if (isLeftPlaying) {
            toggleAudioOutput('left', false);
            toggleAudioOutput('right', true);
        } else {
            toggleAudioOutput('left', true);
            toggleAudioOutput('right', false);
        }
    }

    function initAudio(side) {
        const audioRef = side === 'left' ? audioRefLeft : audioRefRight;
        
        if (!audioRef) return;

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
        } catch (error) {
            console.error(`Erreur lors de l'initialisation de l'AudioContext pour le côté ${side}:`, error);
            if (side === 'left') isInitializedLeft = false;
            if (side === 'right') isInitializedRight = false;
        }
    }

    function handleFileSelect(event, side) {
        const file = event.target.files[0];
        if (file) {
            const oldSrc = side === 'left' ? currentAudioSrcLeft : currentAudioSrcRight;
            if (oldSrc) { URL.revokeObjectURL(oldSrc); }
            
            const fileName = file.name;

            const url = URL.createObjectURL(file);
            if (side === 'left') {
                currentAudioSrcLeft = url;
                fileNameLeft = fileName;
            } else {
                currentAudioSrcRight = url;
                fileNameRight = fileName;
            }
            
            if (side === 'left') isInitializedLeft = false;
            if (side === 'right') isInitializedRight = false;

            initAudio(side); 
        }
    }

    function startVisualization() {
        if (!currentAudioSrcLeft || !currentAudioSrcRight) {
            console.warn("Les deux fichiers audio ne sont pas chargés.");
            return;
        }
        
        if (!isInitializedLeft) initAudio('left');
        if (!isInitializedRight) initAudio('right');

        scoreLeft = 0; 
        scoreRight = 0;
        
        // CORRECTION: Réinitialiser l'état de fin de jeu
        gameEnded = false;
        
        // Réinitialiser la balle au centre
        ball.x = CENTER_X; 
        ball.y = CENTER_Y;
        
        if (audioRefLeft) {
            audioRefLeft.currentTime = 0; 
            audioRefLeft.play().catch(e => console.error("Échec de la lecture audio gauche:", e));
        }
        if (audioRefRight) {
            audioRefRight.currentTime = 0; 
            audioRefRight.play().catch(e => console.error("Échec de la lecture audio droite:", e));
        }
        
        hasStarted = true;
        updateAudioOutput();
        
        if (!animationFrameId) {
            draw();
        }
    }
    
    function stopVisualization() {
        // Annuler la boucle d'animation
        if (animationFrameId) {
            cancelAnimationFrame(animationFrameId);
            animationFrameId = null;
        }
        
        if (audioRefLeft) {
            audioRefLeft.pause();
        }
        if (audioRefRight) {
            audioRefRight.pause();
        }
        
        hasStarted = false;
        // CORRECTION: Marquer le jeu comme terminé
        gameEnded = true; 
        
        console.log("Jeu terminé : un fichier audio est arrivé à la fin.");
        
        // Svelte redessinera l'écran de fin via la dépendance réactive sur gameEnded
    }
    
    function handleAudioEnded() {
        stopVisualization();
    }
    
    // --- LOGIQUE VISUELLE ET COLLISION ---
    
    function checkBarCollision(WIDTH) {
        // --- Côté Gauche ---
        if (isInitializedLeft && dataArrayLeft && ball.vx < 0) { 
            const barIndex = Math.floor(ball.y / (barHeightSpace + gap));
            
            if (barIndex >= 0 && barIndex < barCount) {
                // Utiliser la valeur du tableau, qui est mis à jour dans draw()
                const frequencyValue = hasStarted ? dataArrayLeft[barIndex] : 0; 
                const currentBarLength = frequencyValue / AMPLITUDE_FACTOR * PIXEL_NORMALIZER;

                // Collision en X pour la barre GAUCHE: [0, currentBarLength]
                if (ball.x - ball.radius <= currentBarLength) {
                    // S'assurer que la collision se produit dans la zone de la barre
                    if (ball.x >= 0) { 
                        ball.vx = -ball.vx;
                        ball.x = currentBarLength + ball.radius; // Repositionnement
                    }
                }
            }
        }
        
        // --- Côté Droit ---
        if (isInitializedRight && dataArrayRight && ball.vx > 0) { 
            const barIndex = Math.floor(ball.y / (barHeightSpace + gap));
            
            if (barIndex >= 0 && barIndex < barCount) {
                const frequencyValue = hasStarted ? dataArrayRight[barIndex] : 0; 
                const currentBarLength = frequencyValue / AMPLITUDE_FACTOR * PIXEL_NORMALIZER;
                
                // Collision en X pour la barre DROITE: [WIDTH - currentBarLength, WIDTH]
                const barStart = WIDTH - currentBarLength;

                if (ball.x + ball.radius >= barStart) {
                    // S'assurer que la collision se produit dans la zone de la barre
                    if (ball.x <= WIDTH) {
                        ball.vx = -ball.vx;
                        ball.x = barStart - ball.radius; // Repositionnement
                    }
                }
            }
        }
    }

    function updateBall(WIDTH, HEIGHT) {
        if (!hasStarted) return; 

        // 1. Mettre à jour la position
        ball.x += ball.vx;
        ball.y += ball.vy;
        
        // --- LOGIQUE DE LIGNE DE BUT GAUCHE ---
        const GOAL_LINE_LEFT_X = WIDTH * GOAL_LINE_RATIO_LEFT;
        
        if (ball.x - ball.radius < GOAL_LINE_LEFT_X && ball.vx < 0) {
            scoreRight += 1; 
            ball.x = CENTER_X; 
            ball.y = CENTER_Y;
            ball.vx = -ball.vx;
        }
        
        // --- LOGIQUE DE LIGNE DE BUT DROITE ---
        const GOAL_LINE_RIGHT_X = WIDTH * GOAL_LINE_RATIO_RIGHT;
        
        if (ball.x + ball.radius > GOAL_LINE_RIGHT_X && ball.vx > 0) {
            scoreLeft += 1;
            ball.x = CENTER_X; 
            ball.y = CENTER_Y;
            ball.vx = -ball.vx; 
        }

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
        
        ctx.strokeStyle = '#FFFFFF';
        ctx.lineWidth = 2;
        ctx.setLineDash([5, 5]); 
        
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
        
        ctx.setLineDash([]); 
        ctx.lineWidth = 1;
    }


    /**
     * Boucle de dessin unique.
     */
    function draw() {
        if (!canvasRef) {
            return; 
        }
        
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
        if (hasStarted) {
            updateBall(WIDTH, HEIGHT);
        }
        // ------------------------------
        
        // 2. Fonctions de Dessin des Barres
        function drawSide(dataArray, direction, isPlaying) {
            let y = 0; 

            for(let i = 0; i < barCount; i++) {
                if (!dataArray) break;
                
                // Les barres ne sont dessinées que si le jeu est en cours
                const barValue = isPlaying ? dataArray[i] : 0; 
                const barLength = barValue / AMPLITUDE_FACTOR * PIXEL_NORMALIZER;
    
                let r, g, b, x;
                
                if (direction === 1) {
                    r = barLength + (25 * (i/barCount));
                    g = 50 * (i/barCount);
                    b = 200 - barLength;
                    x = 0; // Côté gauche
                } else { 
                    g = barLength + (25 * (i/barCount));
                    r = 50 * (i/barCount);
                    b = 200 - barLength;
                    x = WIDTH - barLength; // Côté droit
                }
                
                ctx.fillStyle = `rgb(${r},${g},${b})`;
                ctx.fillRect(x, y, barLength, barHeightSpace);

                y += barHeightSpace + gap;
            }
        }

        // 3. Lecture des données et Dessin des deux côtés
        if (isInitializedLeft && analyserLeft && dataArrayLeft) {
            if (hasStarted) {
                analyserLeft.getByteFrequencyData(dataArrayLeft);
            }
            drawSide(dataArrayLeft, 1, hasStarted); 
        }

        if (isInitializedRight && analyserRight && dataArrayRight) {
             if (hasStarted) {
                 analyserRight.getByteFrequencyData(dataArrayRight);
             }
             drawSide(dataArrayRight, -1, hasStarted); 
        }
        
        // --- Dessin de la ligne de but et de la balle ---
        // TOUJOURS dessiner le terrain et la balle si le jeu n'a pas démarré ou est en cours
        if (!gameEnded || (currentAudioSrcLeft && currentAudioSrcRight)) {
             drawGoalLine(ctx, WIDTH, HEIGHT);
             drawBall(ctx); 
        }
        // ---------------------------------
        
        // 4. Boucle d'animation Frame
        if (hasStarted) {
            animationFrameId = requestAnimationFrame(draw);
        } else if (gameEnded && currentAudioSrcLeft && currentAudioSrcRight) { // CORRECTION: Affiche l'écran de fin
            // Affichage de l'écran de fin
            ctx.fillStyle = 'rgba(0, 0, 0, 0.8)';
            ctx.fillRect(0, 0, WIDTH, HEIGHT);
            
            ctx.textAlign = 'center';
            ctx.fillStyle = '#FFFFFF';
            ctx.font = 'bold 36px Arial';
            ctx.fillText('Partie Terminée', HALF_WIDTH, HEIGHT / 2);

            // Pas de requestAnimationFrame ici, la boucle s'arrête.
        } else {
            // Si pas démarré (initialisation), on n'appelle pas requestAnimationFrame
            return;
        }
    }
    
    function cleanup() {
        if (animationFrameId) {
            cancelAnimationFrame(animationFrameId);
            animationFrameId = null;
        }
        if (audioCtx) {
            // Tenter de fermer le contexte audio si possible
            if (audioCtx.state !== 'closed') {
                audioCtx.close().catch(e => console.error("Erreur lors de la fermeture de l'AudioContext:", e));
            }
            audioCtx = null;
        }

        if (currentAudioSrcLeft) URL.revokeObjectURL(currentAudioSrcLeft);
        if (currentAudioSrcRight) URL.revokeObjectURL(currentAudioSrcRight);

        isInitializedLeft = false;
        isInitializedRight = false;
    }

    // CORRECTION: Déclenche un appel à draw() si le canvas est prêt et que le jeu est terminé (pour l'écran de fin)
    $: if (canvasRef && !hasStarted && (gameEnded || !animationFrameId)) {
        // S'assurer que la balle est au centre pour le dessin initial ou l'écran de fin
        ball.x = CENTER_X;
        ball.y = CENTER_Y;
        draw(); 
    }


    onDestroy(() => {
        cleanup();
    });
</script>

<svelte:head>
    <title>Audio Pong</title>
</svelte:head>

<div class="visualizer-page-wrapper">
    <div class="visualizer-container">
        
        <div class="file-inputs">
            <label for="audio-file-input-left" class="file-label">
                {fileNameLeft}
            </label>
            <input 
                id="audio-file-input-left"
                type="file" 
                accept="audio/*" 
                bind:this={fileInputRefLeft}
                on:change={(e) => handleFileSelect(e, 'left')}
            />

            <label for="audio-file-input-right" class="file-label">
                {fileNameRight}
            </label>
            <input 
                id="audio-file-input-right"
                type="file" 
                accept="audio/*" 
                bind:this={fileInputRefRight}
                on:change={(e) => handleFileSelect(e, 'right')}
            />
        </div>

    {#if currentAudioSrcLeft && currentAudioSrcRight && !hasStarted && !gameEnded} <button class="start-button" on:click={startVisualization}>
            Commencer
            </button>
        {/if}
        
    {#if currentAudioSrcLeft && currentAudioSrcRight && !hasStarted && gameEnded} <button class="start-button" on:click={startVisualization}>
            Rejouer
            </button>
        {/if}

    <div class="audio-players" class:hidden={!hasStarted}>
            {#if currentAudioSrcLeft}
                <audio 
                    src={currentAudioSrcLeft} 
                    bind:this={audioRefLeft} 
                    controls={hasStarted}
                    on:play={() => audioCtx?.resume()} 
                    on:ended={handleAudioEnded}
                ></audio>
            {/if}

            {#if currentAudioSrcRight}
                <audio 
                    src={currentAudioSrcRight} 
                    bind:this={audioRefRight} 
                    controls={hasStarted}
                    on:play={() => audioCtx?.resume()} 
                    on:ended={handleAudioEnded}
                ></audio>
            {/if}
        </div>

        {#if hasStarted}
            <div class="controls-container">
                <div class="audio-toggle">
                    <label class="switch-label">
                        <input 
                            type="checkbox" 
                            bind:checked={isLeftPlaying}
                            on:change={updateAudioOutput}
                        >
                        <span class="slider round"></span>
                    </label>
                </div>
            </div>
        {/if}
        
        <canvas 
            bind:this={canvasRef} 
            width="{CANVAS_WIDTH}" 
            height="{CANVAS_HEIGHT}"
        ></canvas>
        
    {#if currentAudioSrcLeft && currentAudioSrcRight} <div class="score-display">
                <span class="score-values">
                    <span class="score-left">{scoreLeft}</span>
                    <span class="score-separator">-</span>
                    <span class="score-right">{scoreRight}</span>
                </span>
            </div>
        {/if}
    </div>
</div>

<style>
    /* Styles inchangés */
    :global(body), :global(html) {
        height: 100%; 
        width: 100%;
        background-color: #000;
        margin: 0;
        padding: 0;
    }

    .visualizer-page-wrapper {
				font-family: SansSerif, sans-serif;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100%;
        width: 100%;
    }

    .visualizer-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 15px;
        background: #222;
        padding: 20px;
        border-radius: 8px;
        box-shadow: 0 0 20px rgba(0, 0, 0, 0.5); 
    }

    .file-inputs {
        display: flex;
        justify-content: space-around;
        width: 100%;
        gap: 10px;
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
        background-color: #DDD;
        color: black;
        padding: 10px 20px;
        border-radius: 5px;
        cursor: pointer;
        transition: background-color 0.2s;
        font-weight: bold;
        width: 280px; 
        overflow: hidden; 
        text-overflow: ellipsis;
        text-align: center;
        white-space: nowrap;
    }

    .file-label:hover {
        background-color: #AAA;
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
    
    .start-button {
        padding: 12px 30px;
        background-color: #DDD;
        color: #000;
        border: none;
        border-radius: 5px;
        font-size: 1.2em;
        cursor: pointer;
        transition: background-color 0.3s, transform 0.1s;
        font-weight: bold;
        margin-top: 15px;
    }

    .start-button:hover {
        background-color: #AAA;
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

    .score-display {
        display: flex;
        justify-content: center;
        align-items: center;
        max-width: 100%;
        padding: 10px;
    }
    
    .score-values {
        font-size: 2.5em;
        font-weight: 900;
        color: white;
    }
    
    .score-left {
        color: #9C0D33; 
    }
    
    .score-right {
        color: #0D8F3F; 
    }

    .score-separator {
        margin: 0 15px;
    }


    .audio-toggle {
        display: flex;
        align-items: center;
    }

    .switch-label {
        position: relative;
        display: inline-block;
        width: 90px; 
        height: 40px;
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
        background-color: #9C0D33;
        transition: .4s;
        border-radius: 34px;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }

    .slider:before {
        position: absolute;
        content: "";
        height: 32px;
        width: 32px;
        left: 4px;
        bottom: 4px;
        background-color: white;
        transition: .4s;
        border-radius: 50%;
    }

    input:checked + .slider {
        background-color: #0D8F3F;
    }

    input:checked + .slider:before {
        transform: translateX(50px); 
    }
</style>