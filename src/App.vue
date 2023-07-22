<script setup lang="ts">
import { ref } from 'vue'

let [startXPosition, startYPosition] = [0, 0];
let directionsAdventurer = ref<string>('');
let actualXPosition =  ref<number>(0);
let actualYPosition = ref<number>(0);
let error = ref<string>();

async function loadMapData() {
  const response: Response = await fetch('src/maps/carte.txt')
  const responseText : string = await response.text()

  if(response.status === 200)
    return responseText.split('\n').map(row => row.split(''))
  else
    error.value = response.statusText
}


async function loadFileMouvements() {
 const response : Response = await fetch('src/maps/mouvements.txt');
 const splitedData = (await response.text()).split('\n');

 if(response.status === 200) 
  setDataPosition(parseInt(splitedData[0].split(',')[0]), parseInt(splitedData[0].split(',')[1]),splitedData[1])
  else
    error.value = response.statusText
}

function setDataPosition(posX : number, posY : number, directions : string) {
  [startXPosition, startYPosition,directionsAdventurer.value] = [posX, posY, directions];
  deplacingAdventurer(startXPosition, startYPosition)
}

function deplacingAdventurer(startXPosition : number, startYPosition: number ) {

 actualXPosition.value = startXPosition;
 actualYPosition.value = startYPosition;

for(const direction of directionsAdventurer.value) {
  switch(direction) {
    case 'S':
    actualYPosition.value++;
      break;
    case 'N':
    actualYPosition.value--;
      break;
    case 'E':
    actualXPosition.value++;
      break;
    case 'O':
    actualXPosition.value--;
      break;
  }
}


}

async function main() {
  try {
    let arrayMap = await loadMapData() ?? [];

    await loadFileMouvements();

    


    if(actualXPosition.value < 0 || actualYPosition.value < 0 || arrayMap[actualYPosition.value][actualXPosition.value] === '#'){
      error.value = "L'aventurier se trouve sur une case invalide, veuillez modifier ses directions";
     
      return;
    }
      
   
      arrayMap[actualYPosition.value][actualXPosition.value] = '🧑'; 
      const mapDisplay : HTMLElement | null  = document.getElementById('map');
      
      if(mapDisplay ) {
        mapDisplay.innerHTML = arrayMap.map(row => row.join('')).join('\n');
      }
        
  } catch (error) {
    console.error('Une erreur s\'est produite :', error);
  } 
}

main();

</script>

<template>
  <header>
   
  </header>

  <main>
   <h2>Adventurer game</h2>
   <pre id="map"></pre><br>
   <div >
    L'aventurier se trouve actuellement en X : {{actualXPosition}} et en Y : {{actualYPosition}}
   </div>
   <div v-if="error">{{ error }}</div>
  </main>
</template>


