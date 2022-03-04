<script>
    const icons = {
        closed: '',
        flag: '🚩',
        bomb: '💣'
    }
	// For score timer
	let startDate
	let timer = 0
	let setIntervalRef

	let isGameOver = false
	let win = false

	// Form creation form
	let inputHeight=10
	let inputWidth=10
	let inputMinesQuantity=10

	// For playing grid
	let height
	let width
	let gridSize
	let minesQuantity
	// Keep track of the cells
	let gridCells = []
	
	// Count number of undiscoverd mines
  	$: remainingMines = minesQuantity - gridCells.reduce((a,c) => a + (c === icons.flag), 0)

	// List of the mines location
	let minesLocation
	
	// Generate mines position
	const createMines = () => {
		// Max mines = (X-1)*(Y-1)
		minesQuantity = inputMinesQuantity > (width-1) * (height - 1) ?
			(width-1) * (height - 1)
			: inputMinesQuantity

		const arr = new Set()
		while(arr.size < minesQuantity) {
			arr.add(parseInt(Math.random()*gridSize))
		}
		minesLocation = Array.from(arr)
	}
	
	const createGrid = () => {
		height = inputHeight
		width = inputWidth
		gridSize = height * width

		createMines()
		
		const arr = []
		for(let i=0; i<gridSize; i++) {
			arr.push(icons.closed)
		}
		gridCells = arr

		isGameOver=false
		win=false

		startDate = Date.now()

		// Start score timer
		clearInterval(setIntervalRef) // if still running
		setIntervalRef = setInterval(() => {
			timer = parseInt((Date.now() - startDate) / 1000)
		}, 1000)
	}
	
	const gameover = () => {
		clearInterval(setIntervalRef)
		
		isGameOver=true
		minesLocation.forEach(m => {
			gridCells[m] = icons.bomb
		})
	}

    const getNeighboursList = id => {
        if(id < 0 || id > gridSize) return []
        const neighboursList = []
        
        if(id % width !== 0){ // !left
            neighboursList.push(id-1) // add left
            if((id+1) % width !== 0){ // !right
                neighboursList.push(id+1) // add right
                if(id-width >=0) // not on top
                    neighboursList.push(id-width, id-width-1, id-width+1)
                if(id+width < gridSize)
                    neighboursList.push(id+width, id+width-1, id+width+1)
            } else { // right
                if(id-width >=0) // !top
                    neighboursList.push(id-width, id-width-1)
                if(id+width < gridSize) // !bottom
                    neighboursList.push(id+width, id+width-1)
            }
        } else { // left
            neighboursList.push(id+1) // add right
            if(id-width >=0) // !top
                neighboursList.push(id-width, id-width+1)
            if(id+width < gridSize) // !bottom
                neighboursList.push(id+width, id+width+1)
        }
        return neighboursList
    }
	
	
    const countNeighboursMines = id => {
		if(id < 0 || id > gridSize) return -1
		
		let neightbors = 0
        const neighboursList = getNeighboursList(id)
		neighboursList.forEach(n => {
			if(minesLocation.includes(n))
				neightbors++
		})
		return neightbors
	}

	const openNeighbours = id => {
        if(id<0 || id > gridSize) return
        const neighboursList = getNeighboursList(id)
		neighboursList.forEach(n => {
			if(gridCells[n] === icons.closed){
				const neighbours = countNeighboursMines(n)
				gridCells[n] = neighbours
				if(neighbours === 0)
					openNeighbours(n)
			}
		})
	}

	// Win condition

	/*
		- Toutes les cases ouvertes sauf les bombes
		- Tous les drapeaux sur les bombes + cases ouvertes
	*/
    const checkWin = () => {
        if(remainingMines < 0) return
		// count empty or flagged cells if they are not a mine
		const emptyCellsNumber = gridCells.reduce((a,c,i) => a + (!minesLocation.includes(i) && (c === icons.closed || c === icons.flag)), 0)

		// no more empty cells : WIN
		if(emptyCellsNumber === 0){
			clearInterval(setIntervalRef)

			isGameOver = true
			win = true
			// replace bombs with flags
			minesLocation.forEach(m => {
				gridCells[m] = icons.flag
			})
		}
    }

    const rightClick = e => {
        e.preventDefault()
        const id = parseInt(e.target.id)
        // Cell not open
        if(gridCells[id] === icons.closed)
            gridCells[id] = icons.flag
        else if(gridCells[id] === icons.flag)
        gridCells[id] = icons.closed
		checkWin()
    }
	
	const openCell = e => {
		if(isGameOver) return
		const id = parseInt(e.target.id)
		// Found a mine: game is over
        if(gridCells[id] !== icons.closed) return
		if(minesLocation.includes(id)) {
			gameover(id)
		} else {
			const neighbours = countNeighboursMines(id)
			gridCells[id] = neighbours
			if(neighbours === 0) {
				openNeighbours(id)
			}
            
			checkWin()
		}
	}
</script>

<h1>Demineur</h1>

<div id='form'>
	<label for='height'>Hauteur : </label> <input id='height' type='number' bind:value={inputHeight} />
	<label for='width'>Largeur : </label> <input id='width' type='number' bind:value={inputWidth} />
	<label for='mines'>Mines : </label> <input id='mines' type='number' bind:value={inputMinesQuantity} />
	<button id='create' on:click={createGrid}>
		Nouvelle grille
	</button>
</div>

{#if gridSize}
<p id='time'>
	<span>Timer: {timer}s</span> 
	<span>Mines: {remainingMines}</span> 
	{#if isGameOver}<span class={win? 'win':'loose'}>{win ? 'VICTOIRE' : 'GAME OVER'}</span>{/if}
</p>
<div id='grid' class:gameover="{isGameOver}" style="grid-template-columns: repeat({width}, 2em)">
	{#each gridCells as cell, i}
	<button id={i}
		on:click={openCell}
		on:contextmenu={rightClick}
		class={cell === '' ? 'closedCell' : cell !== icons.bomb ? 'openedCell' : 'bombCell'}>
			{cell ? cell : ''}
		</button>
	{/each}
</div>
{/if}



<style>
	#form input {
		width: 6em;
		display:inline-block;
	}
	#form label {
		display:inline-block;
	}
	#grid {
		display: grid;
	}
	.gameover {
		opacity: 0.6;
	}
	#grid button {
		height:2em;
		width: 2em;
		margin:0;
		vertical-align:top;
	}
	.closedCell {background-color: #eaeaea;}
	.openedCell {background-color: #fafafa;}
	.bombCell{background-color: red;}
	.win{color: green;}
	.loose{color:red;}
</style>
