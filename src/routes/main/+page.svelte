<script>
    // Aucun script nécessaire pour cette page
</script>

<style>
    :global(html, body) {
        margin: 0;
        padding: 0;
        height: 100%;
        font-family: 'Press Start 2P', monospace;
        overflow: hidden;
        background: #000;
    }

    /* Fond avec ton image locale */
    .main-bg {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: url('/retro-gaming.jpg') center/cover no-repeat;
        background-color: red; /* pour test */
	z-index: -1;
    }

    .container {
        display: flex;
        justify-content: center;
        align-items: flex-start;
        height: 100vh;
        position: relative;
    }

    /* Bouton Start placé sur l'écran de la console */
    .start-btn {
        position: absolute;
        top: 38%;       /* Ajustable selon ton image pour centrer le bouton sur l'écran */
        left: 50%;
        transform: translateX(-50%);
        font-size: 1.5rem;
        padding: 1rem 2rem;
        background: #111;
        color: #0ff;
        border: 4px solid #0ff;
        border-radius: 8px;
        cursor: pointer;
        text-transform: uppercase;
        text-decoration: none;
        text-align: center;
        transition: 0.15s ease-in-out;
    }

    .start-btn:hover {
        transform: translateX(-50%) scale(1.08);
        box-shadow: 0 0 15px #0ff;
    }

    @media (max-width: 600px) {
        .start-btn {
            font-size: 1.2rem;
            padding: 0.8rem 1.5rem;
        }
    }
</style>

<div class="main-bg"></div>

<div class="container">
    <a href="/form" class="start-btn">Start</a>
</div>

