<script>
	// Aucun script nécessaire pour cette page
</script>

<style>

    /* Fond avec ton image locale */
    .main-bg {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        /*background: url('/retro-gaming.jpg') center/cover no-repeat;*/
        /*background-color: red; !* pour test *!*/
        z-index: -1;
    }

    .container {
        display: flex;
        justify-content: space-evenly;
        margin: 5% auto;
        flex-flow: column;
        width: 30%;
        height: 90%;
    }


</style>

<div class="main-bg"></div>

<div class="container">
	<a href="/form" class="start-btn">Form</a>
	<a href="/quizz" class="start-btn">Quizz</a>
	<a href="/AudioPong" class="start-btn">Visualizer</a>
</div>

