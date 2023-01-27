<template>
    <div class="game-board">
        <Cell
            v-for="cell of allCellsInConsecutiveOrder"
            :nr="cell.cellNumber"
            :tileValue="cell.tileValue"
            :isRandomlyPlaced="cell.isRandomlyPlaced"
        />
    </div>
</template>

<script setup lang="ts">
import { onMounted, computed, reactive } from "vue";
import Cell from "./Cell.vue"

const emit = defineEmits(['won', 'lost']);

interface CellType {
    y: number, 
    x: number,
    cellNumber: number,
    tileValue: number | null,
    isRandomlyPlaced?: boolean
}

let grid: CellType[][] = reactive([
    [{ cellNumber: 0, y: 0, x: 0, tileValue: null}, {cellNumber: 1, y:0, x:1, tileValue: null}, {cellNumber: 2, y:0, x:2, tileValue: null}, {cellNumber: 3, y:0, x:3, tileValue: null}],
    [{ cellNumber: 4, y:1, x:0, tileValue: null}, {cellNumber: 5, y:1,x:1, tileValue: null}, {cellNumber: 6,y:1,x:2, tileValue: null}, {cellNumber: 7, y:1,x:3, tileValue: null}],
    [{ cellNumber: 8, y:2,x:0, tileValue: null}, {cellNumber: 9,y:2,x:1, tileValue: null}, {cellNumber: 10,y:2,x:2, tileValue: null}, {cellNumber: 11,y:2,x:3, tileValue: null}],
    [{ cellNumber: 12, y:3,x:0, tileValue: null}, {cellNumber: 13,y:3,x:1, tileValue: null}, {cellNumber: 14,y:3,x:2, tileValue: null}, {cellNumber: 15,y:3,x:3, tileValue: null}],
]);

const allCellsInConsecutiveOrder = computed(() => {
    return [...grid[0], ...grid[1], ...grid[2], ...grid[3]];
})


onMounted(() => {
    // add 2 random tiles initially
    addRandomTile(getEmptyCells())
    addRandomTile(getEmptyCells())
    addKeyDownEventListener()
})

function addKeyDownEventListener() {
    const ARROW_UP = "ArrowUp"
    const ARROW_DOWN = "ArrowDown"
    const ARROW_LEFT = "ArrowLeft"
    const ARROW_RIGHT = "ArrowRight"
    window.addEventListener("keydown", event => {
        const key = event.key
        if (key !== ARROW_UP && key !== ARROW_DOWN && key !== ARROW_LEFT && key !== ARROW_RIGHT) {
            return
        }
        if (event.key === ARROW_UP) {
            slideUp()
        }
        if (event.key === ARROW_DOWN) {
            slideDown()
        }
        if (event.key === ARROW_LEFT) {
            slideLeft()
        }
        if (event.key === ARROW_RIGHT) {
            slideRight()
        }
        nextRound()
    })
}

function nextRound() {
    const currentTiles = getCurrentTiles();
    if (currentTiles.find((cell) => cell.tileValue === 2048)) {
        emit("won")
        return
    }
    clearAllIsRandomlyPlacedMarkers() 
    addRandomTile(getEmptyCells())

    if (getEmptyCells().length === 0) {
        emit("lost")
    }
}

function clearAllIsRandomlyPlacedMarkers() {
    for (let y = 0; y < 4; y++) {
        for (let x = 0; x < 4; x++) {
            grid[y][x].isRandomlyPlaced = false
        }
    }
}

function addRandomTile(emptyCells: CellType[]) {
    const randomIndex = randomInteger(0, emptyCells.length);
    const randomEmptyCell = emptyCells[randomIndex];
    grid[randomEmptyCell.y][randomEmptyCell.x].tileValue = randomNewTileValue()
    grid[randomEmptyCell.y][randomEmptyCell.x].isRandomlyPlaced = true;
}

function getEmptyCells(): CellType[] {
    const emptyCells: CellType[] = [];
    for (let y = 0; y < 4; y++) {
        for (let x = 0; x < 4; x++) {
            const currentCell =grid[y][x]
            if (currentCell.tileValue === null) {
                emptyCells.push(grid[y][x])
            }
        }
    }
    return emptyCells;
}

function slideUp() {
    const currentTiles = getCurrentTiles()
    const sortedLowestToHighestY = currentTiles.sort((a, b) => a.y - b.y)
    sortedLowestToHighestY.forEach(tile => moveTileUp(tile))
}

function slideLeft() {
    const currentTiles = getCurrentTiles();
    const sortedLowestToHighestX = currentTiles.sort((a, b) => a.x - b.x);
    sortedLowestToHighestX.forEach(tile => moveTileLeft(tile))
}

function slideRight() {
    const currentTiles = getCurrentTiles();
    const sortedHighestToLowestX = currentTiles.sort((a, b) => b.x - a.x)
    sortedHighestToLowestX.forEach(tile => moveTileRight(tile))
}

function slideDown() {
    const currentTiles = getCurrentTiles()
    const highestToLowestY = currentTiles.sort((a, b) => b.y - a.y)
    highestToLowestY.forEach(tile => moveTileDown(tile))
}

function getCurrentTiles(): CellType[] {
    let tiles: CellType[] = [];
    for (let y = 0; y < 4; y++) {
        for (let x = 0; x < 4; x++) {
            if (grid[y][x].tileValue !== undefined) {
                tiles.push(grid[y][x])
            }
        }
    } 
    return tiles
}

function moveTileUp(tile: CellType) {
    let currentY = tile.y;
    while(currentY > 0) {
        if (cellsCanMerge(tile, grid[currentY-1][tile.x])) {
            grid[currentY-1][tile.x].tileValue += tile.tileValue
            grid[tile.y][tile.x].tileValue = null
        }
        if (!isCellEmpty(grid[currentY-1][tile.x])) {
            break
        }
        currentY--;
    }
    const tileValueToBeMoved = grid[tile.y][tile.x].tileValue
    grid[tile.y][tile.x].tileValue = null
    grid[currentY][tile.x].tileValue = tileValueToBeMoved
}

function moveTileLeft(tile: CellType) {
    let currentX = tile.x;
    while (currentX > 0) {
        if (cellsCanMerge(tile, grid[tile.y][currentX-1])) {
            grid[tile.y][currentX-1].tileValue += tile.tileValue
            grid[tile.y][tile.x].tileValue = null
        }
        if (!isCellEmpty(grid[tile.y][currentX-1])) {
            break
        }
        currentX--;
    }
    const tileValueToBeMoved = tile.tileValue
    grid[tile.y][tile.x].tileValue = null
    grid[tile.y][currentX].tileValue = tileValueToBeMoved
}

function moveTileRight(tile: CellType) {
    let currentX = tile.x
    while(currentX < 4) {
        if (cellsCanMerge(tile, grid[tile.y][currentX+1])) {
            grid[tile.y][currentX+1].tileValue += tile.tileValue
            grid[tile.y][tile.x].tileValue = null
        }
        if (!isCellEmpty(grid[tile.y][currentX+1])) {
            break
        }
        currentX++
    }
    const tileValueToBeMovedRight = tile.tileValue
    grid[tile.y][tile.x].tileValue = null
    grid[tile.y][currentX].tileValue = tileValueToBeMovedRight
}

function moveTileDown(tile: CellType) {
    let currentY = tile.y;
    while (currentY < 4) {
        if (grid[currentY+1] === undefined) {
            break
        }
        if (cellsCanMerge(tile, grid[currentY+1][tile.x])) {
            grid[currentY+1][tile.x].tileValue += tile.tileValue
            grid[tile.y][tile.x].tileValue = null
        }
        if (!isCellEmpty(grid[currentY+1][tile.x])) {
            break
        }
        currentY++
    }
    const tileValueToBeMovedDown = tile.tileValue
    grid[tile.y][tile.x].tileValue = null
    grid[currentY][tile.x].tileValue = tileValueToBeMovedDown
}

function isCellEmpty(cell?: CellType): boolean {
    if (cell === undefined) {
        return false
    }
    if (cell.y < 0 || cell.y > 3 || cell.x < 0 || cell.x > 3) {
        return false;
    }
    return cell.tileValue === null;
}

function cellsCanMerge(currentCell: CellType, mergeCandidate?: CellType): boolean {
    if (mergeCandidate === undefined) {
        return false
    }
    if (currentCell.tileValue === null) {
        return false
    }
    //console.log(`tile ${currentCell.cellNumber} and ${mergeCandidate.cellNumber} can merge`)
    return currentCell.tileValue === mergeCandidate.tileValue
}

function randomInteger(min: number, max: number): number {
    return Math.floor(Math.random() * (max - min) + min);
}

function randomNewTileValue(): number {
    return Math.random() < 0.1 ? 4 : 2;
}

</script>

<style scoped>
.game-board {
    position: relative;
    background-color: slategray;
    display: grid;
    grid-template-rows: repeat(4, 20vmin);
    grid-template-columns: repeat(4, 20vmin);
    gap: 1vmin;
}
</style>