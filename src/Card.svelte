<script>
	import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    
    //Card identifier
    export let index
    //Image (pair) identifier
    export let item = ''
    //Image URL
    export let url;
    //Component State
    export let disabled, show = false, selected = false, won = false

   
    
    //Show the back of the card if it is selected or won
    $: show = selected || won;

    //Pass the click up to the game select handeler
    function selectCard() {
		if(!disabled) dispatch('selectCard', {
			id: index
		});
    }
    

</script>
<!-- Set our states on teh component for CSS styles -->
<article class:show class:selected class:won {disabled} on:click={selectCard}>
    <section class="front" >
        <div>
            {#if item}
                <img src='{url}' alt='radom image'/>
            {/if}
        </div>
    </section>
    <section class="back" />
</article>

<style>

/* Card holder */
    article {
        transform: perspective(400px) rotateY(0deg);
        transform-style: preserve-3d;
        position:relative;
        transition: all 1s cubic-bezier(0.445, 0.05, 0.55, 0.95);
        width: 240px;
        height: 265px;
        display: grid;
        place-items: center;   
    }

    article.show {
            transform: perspective(400px) rotateY(180deg);
    }
/* Faces */
    section {
        position: absolute;
        top: 0;
        width: 100%;
        height: 100%;
        border-radius: inherit;
        backface-visibility: hidden;
        background: hsl(0, 0%, 100%);
        pointer-events: none;
        border: 5px solid #666;
        transition:all 0.3s;
        border-radius:20px;
        cursor: pointer;
        cursor: pointer;
        display: grid;
    }
    
    section.back {
        background-image: url('data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" width="100" height="18" viewBox="0 0 100 18"%3E%3Cpath fill="%239C92AC" fill-opacity="0.4" d="M61.82 18c3.47-1.45 6.86-3.78 11.3-7.34C78 6.76 80.34 5.1 83.87 3.42 88.56 1.16 93.75 0 100 0v6.16C98.76 6.05 97.43 6 96 6c-9.59 0-14.23 2.23-23.13 9.34-1.28 1.03-2.39 1.9-3.4 2.66h-7.65zm-23.64 0H22.52c-1-.76-2.1-1.63-3.4-2.66C11.57 9.3 7.08 6.78 0 6.16V0c6.25 0 11.44 1.16 16.14 3.42 3.53 1.7 5.87 3.35 10.73 7.24 4.45 3.56 7.84 5.9 11.31 7.34zM61.82 0h7.66a39.57 39.57 0 0 1-7.34 4.58C57.44 6.84 52.25 8 46 8S34.56 6.84 29.86 4.58A39.57 39.57 0 0 1 22.52 0h15.66C41.65 1.44 45.21 2 50 2c4.8 0 8.35-.56 11.82-2z"%3E%3C/path%3E%3C/svg%3E');    
    }

    section.front {
         transform: rotateY(180deg);
    }

    section.front div {
        padding: 14px;
        display: grid;
        place-items: center;
    }

    section.front img {
        width:100%;
    }

/* States     */
    .selected section {
        border-color: yellow;
    }

    .won section {
        opacity:0.5;
        border-color: green;
    }

    [disabled=true]:not(.selected) {
        opacity:0.1;
    }

    .selected, .won, [disabled=true] {
        cursor: not-allowed;
    }
    

</style>