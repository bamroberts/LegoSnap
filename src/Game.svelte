<script>
import Card from './Card.svelte'
import {scale, fade} from 'svelte/transition'
import {images} from './images.js'
import {onMount} from 'svelte'

export let mode = 'hard'

let modes = [
    {
        name:'easy',
        pairs:10,
        sizes:[2, 5, 10]
    },{
        name:'medium',
        pairs:20,
        sizes:[2, 4, 5, 10, 20]
    },{
        name:'hard',
        pairs:40,
        sizes:[2, 4, 5, 8, 10, 16, 20, 40]
    },{
        name:'insane',
        pairs:80,
        sizes:[2, 4, 5, 8, 10, 16, 20, 32, 40, 80]
    }
]

let pairs
let blank
let modeConfig;

$: modeConfig = modes.find(x=>x.name==mode)
$: pairs = modeConfig.pairs
$: blank = Array(pairs).fill(0);


let grid;
let portrait = false;
let holder;
let ratio = 1;
let reverseRatio
let cols;
$: if(holder) holder.offsetHeight , holder.offsetWidth
$: if(holder) ratio = holder.offsetHeight / holder.offsetWidth
$: if(holder) portrait = (ratio > 1.2)
$: if(holder) reverseRatio = holder.offsetWidth / holder.offsetHeight
//$: if(holder) console.log(cols = modeConfig.sizes.find((x)=> x > ratio * pairs * 2)
$: if(holder) cols = modeConfig.sizes[modeConfig.sizes.findIndex((x)=> x > portrait ? reverseRatio : ratio * pairs) + (portrait ? 1 : -1) ]

    
//Init game varables
let gameItems //Storage for selected game images 
let tries //Number of goes
let time //Time taken
let started //Has the current game started - To start timer
let disabled //Lets us disable input for all cards
let selectedA, selectedB //Holders for checking our selected pairs


//We are using Svelte rective properties to reset the game when 'options' is changed. So we start with populating with all options available in the game
let options =  images;

let promises = pause(3000);
let promiseCount = 0

//Each time options changes we reset the game vars - this only changes when the game starts or we restart the game (as easy as options = options)
$: if(options) {time = 0; tries = 0, selectedA = null, selectedB = null, started = null, promiseCount = 0}
//Generate out list of play cards - We duplicate our options array, randomise it and take the number of elements we need for the game - in this case 10 items - This is triggered to recalc when options variable changes.
$: gameItems = [...options].sort(randomSort).slice(0, pairs)

//Promise to preload all images before rendering the game.
$: promises = Promise.all(gameItems.filter(x => x).map(x => load_image(x)))

// We then duplicate the array twice (so we will have two of each card), randomise the list and use map to initiate an object for each time to store its state - This is triggered when the above code runs and gameItems changes
$: cards = [...gameItems, ...gameItems].sort(randomSort).map( (x,i) => {
    return {
        item: gameItems.findIndex((y) => x == y) +1,
        url: x
    };
});

//Makes us an array of all won cards - Score helper
$: wonCards = cards.filter(c=>c.won)
//Is the game won - the length of won cards is the length of our cards array, this doesn't run if the game interface is disabled (used to stop flash of Winning message as we reset the game and init arrays)
$: wonGame = !disabled && wonCards.length == cards.length 
//So we can start the timer we see if the game has already registered as started, if we have had a full or partial go yet.
$: if(!started) started = (tries || selectedA)
//Cheat way of running the timer. Because Svelte waches all varable defined in a $: block, time changes also causes this to rerun (whilst started and !won are true). Probably no an acurate way to do this, but fine for the game and lets us redraw the clock each second.
$: if(started && !wonGame) setTimeout(()=>time++, 1000);


//Ux trigger to handle card being clicked on 
const selectCard = (event) => {
    //ID passed from card and event
    let item = cards[event.detail.id]
    //Can't select a card it is already won, selected or until match is checked if second card is already picked.
    if(item.won || selectedA == item || selectedB) {
        return
    } 
    
    //Mark the card as slected
    item.selected = true;

    if(!selectedA) {
        //If it is the first card pick, just set it as selected
        selectedA = item;
    }
    else {
        //If it is the second card then select it, disable the board and check the pair
        selectedB = item;
        disabled = true;
        
        checkMatch();
    }
    //We push the array of cards back into its holder varaible so Svelte redraws the changed card states
    cards = [...cards];
}

//Function to compare the cards, mark winners and reset the UI for the next go
const checkMatch = async () => {
    //Update the number of tries
    tries++;
    //Pause the game - We want the selected pair to hang around for a second for user feedback to wait for animation to complete etc
    await pause(1500)

    //Are the cards the same library item
    if(selectedA.item == selectedB.item) {
        //Mark them true
        selectedA.won = true;
        selectedB.won = true;
    } else {
        //Otherwise unselect them
        selectedA.selected = false
        selectedB.selected = false
    }
    //Again we push the array of cards back into its holder varaible so Svelte redraws the changed card states
    cards = [...cards];

    //Wipe out holders 
    selectedA = null;
    selectedB = null;

    //Open up the board again for next go
    disabled = false
}

//Function to reset the board and restart the game.
const restart = async (m) => {
    //We could do this as simply as options = options, but the new card images appear before the cards have finished flipping back around
    //To counter that we actually reset the board twice, once with emty elements, to trigger the reset and card flip and then again to push the new set of game options in and shuffle everything
    //We also disable the board so you can't pick any cards before the reset has finished.

    if(m && m != mode) {
        mode = m
    }

    disabled = true; 
    options = blank; 
    
    await pause(1000); 

    options = images; 
    disabled = false;
}

//Helper function that lets us randomise our array
const randomSort = () => 0.5 - Math.random()

//Helper function that lets us sleep the application in async function (ux state changes would happen too quick otherwise )
function pause(ms) {
	return new Promise(fulfil => {
		setTimeout(fulfil, ms);
	});
} 

//Helper function to pad our timer numbers and kill fractions
const niceTime = (t) => (''+Math.floor(t)).padStart(2, '0'); 

function load_image(src) {
	return new Promise((fulfil, reject) => {
		const img = new Image();
		img.onload = () => fulfil();
		img.onerror = reject;
		img.src = src;
	}).then(
        () => promiseCount++
    );
}

</script>
    
    {#await promises}
    <div class="loading">
        <h3>Loading.... </h3>
        Image {promiseCount} of {pairs} 
        <progress id="file" value="{(promiseCount/pairs)*100}" max="100"> {(promiseCount/pairs)*100}% </progress>
    </div>
    {:then results}
        <h2>
            <span>
                <!-- half the won cards (as there are two wins for each pair), against the total number of pairs -->
                Score: {wonCards.length / 2}/{pairs} 
            </span>
            <span>
                Tries: {tries}
            </span>
            <span>
                <!-- Split our seconds into hh:mm:ss formart -->
                Time: {niceTime(time/60/60)}:{niceTime((time/60)%60)}:{niceTime(time%60)}
            </span>
        </h2>
        <!-- Messaging on game completion -->
        {#if wonGame}
            <div in:scale class="won message">
                <h3>Yay you won the game!!!</h3>
                <button type=button on:click={restart}>Go Again</button>
            </div>   
        {/if} 
        <main bind:this={holder}>
            <ul class="{mode}" class:portrait  style="--pairs:{pairs}; --col-guess:{cols}; --row-guess:{(pairs * 2) / cols}" bind:this={grid} >
            <!-- Loop through and draw each card -->    
            {#each cards as item, i}
                <li>
                    <!-- We want to pass everything in the cards state as props to our component. Disabled is a global game state so we pass that seperatly, we also pass the index to the component so we can find it in the state array when the use interacts later  -->
                    <Card {...item} index={i} {disabled} on:selectCard={selectCard} />
                </li>
            {/each}
            </ul>
        </main>
        <div class='reset'>
        RESTART:
        {#each modes as {name}}
            <button type=button class:selected={mode==name} on:click={()=>restart(name)}>{name}</button>
        {/each}
        </div>
    {:catch error}
        <h3>Could not load all images :( </h3>
        Thre might be a network connection issue
        <button type=button on:click={restart}>try loading again?</button>
    {/await}
   
   
 <footer>
    <div>Icons made by <a href="https://www.flaticon.com/authors/smashicons" title="Smashicons">Smashicons</a> from <a href="https://www.flaticon.com/" title="Flaticon">www.flaticon.com</a></div>
    
    <div class="links">
        Game 
        <a href="githiub.com/bamroberts/LegoSnap" title="Link to source code" target="_blank" rel="noopener noreferrer">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/></svg>
            code
        </a> 
         made by 
        <a href="https://twitter.com/bamroberts" title="Ben Roberts twitter account" target="_blank" rel="noopener noreferrer" >
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path d="M12 0c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12-5.373-12-12-12zm6.066 9.645c.183 4.04-2.83 8.544-8.164 8.544-1.622 0-3.131-.476-4.402-1.291 1.524.18 3.045-.244 4.252-1.189-1.256-.023-2.317-.854-2.684-1.995.451.086.895.061 1.298-.049-1.381-.278-2.335-1.522-2.304-2.853.388.215.83.344 1.301.359-1.279-.855-1.641-2.544-.889-3.835 1.416 1.738 3.533 2.881 5.92 3.001-.419-1.796.944-3.527 2.799-3.527.825 0 1.572.349 2.096.907.654-.128 1.27-.368 1.824-.697-.215.671-.67 1.233-1.263 1.589.581-.07 1.135-.224 1.649-.453-.384.578-.87 1.084-1.433 1.489z"/></svg>
            bamroberts
        </a>
         with 
        <a href="https://svelte.dev" target="_blank" rel="noopener noreferrer" title="Svelte website">
            <img src="/favicon.png" alt="Svelte logo" width="24" height="24" />svelte
        </a>
    </div>

    </footer>
    

<style>
    ul {
        
        --col-count: 5;
        --row-count: 4;

        /* --target-width:1200px;
        --target-height:1320px; */
        display:grid;
        grid-template-columns: repeat(var(--col-count), 1fr);
        gap:min(calc(max(250px, 60vmin) / var(--pairs)), 20px);    
        list-style: none;
        margin:0;
        padding:0;
        
        width:100%;

        align-self: start;
        max-width:1200px;
    }

    [style*="--aspect-ratio"] > :first-child {
  width: 100%;
}

li :global(article) {position: relative;}
li :global(article::before) {
    content: "";
    display: block;
    padding-bottom: calc(100% / .85);
  }  
  li :global(article > :first-child) {
    position: absolute;

    height: 100%;

  }



    ul.medium {
         --col-count: 8;
         --row-count:5;
    }

    ul.hard {
         --col-count: 10;
         --row-count:8;
    }

    ul.insane {
         --col-count: 16;
         --row-count: 10;
    }


     ul.portrait {
        --col-count: var(--row-count);
        gap:1vh;
    }

    main {
        display: grid;
        position: relative; 
        place-items:center;  
    }


    main div.message {
        padding: 8em;
        
        text-align: center;

    }

    main div.won {
        position: absolute;
        background-color: rgba(200,255,200,0.1);
        box-shadow: 2px 2px 5px 4px rgba(0,0,0,0.25);
        z-index: 1;
        backdrop-filter: blur(3px);

    }

   
    h2, footer {
                        grid-template-columns: 1fr max-content 1fr;

        font-size: clamp(12px,4vmin, 2em);
        display:grid;
        justify-content: space-between;
        width:100%;
        text-align:center;
        align-items: baseline;

    }   

    footer {
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        font-size: clamp(10px, 2vmin, 1em);
        gap: 10px 40px;
        padding: 5px 0 2px;

    }
    footer a {white-space: nowrap;}
    footer svg, footer img {
        transform:scale(clamp(12px,4vw, 2em));
    }
  

    h3 {
        font-size:3em;
    }

    button {
        background-color: #eee;
        color: #333;
        outline: none;
        border: 2px solid #666;
        border-radius: 10px;
        padding: 0.5em 1em;
        text-transform: uppercase;
        margin-top:20px;
        cursor:pointer;
        transition:all 0.3s;
    }

    div button {
        font-size: 1.5em;
        padding: 1em 2em;
        border-color: rgba(100,255,100,1);
        box-shadow: 0px 0px 9px 2px rgba(0,0,0,0.2);
    }

    button:hover {
        border: 2px solid #ff3e00;
    }

    .links svg {
        opacity:0.15;
    }

    .links svg, .links img {
        position:absolute;
        left:0;
        top:-4px;
    }
    .links a {
        position:relative;
    }
    .links a:before {
        content:'';
        display:inline-block;
        width:22px;
    }
    a {
        text-decoration: none;
    }
    

    main {
        transition: all 0.5s;
        opacity:1;
        min-height:50vh;
    }
    div.reset {
        font-size:50%;
        text-align: center;
    }

    div.reset button {
        margin-left:10px;
        padding:3px;
        box-shadow: 2px 2px 4px rgba(0,0,0,0.125)
    }


    div.reset button:not(:hover):not(.selected)  {
        border-color: #333;
    }
    div.reset button.selected {font-weight:bold;}

.loading { align-self:center; text-align:center;}

h2, .reset button, .reset {margin: 1vh;}
</style>