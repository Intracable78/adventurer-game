<script setup lang="ts">
import { ref } from 'vue'

let [startXPosition, startYPosition] = [0, 0];
let directionsAdventurer = ref<string>('');
let actualXPosition =  ref<number>(0);
let actualYPosition = ref<number>(0);
let error = ref<string>();

//chargement des datas du parcours, on retourne un tableau à deux dimmensions

async function loadMapData() {
  const response: Response = await fetch('src/maps/carte.txt')
  
  if(response.status === 200) {
    const responseText : string = await response.text()
    return responseText.split('\n').map(row => row.split(''))
  }

  else
    error.value = response.statusText
}

// chargement des datas de la postition X & Y de l'abenturier ainsi que des directions à emprunter
async function loadFileMouvements() {
 const response : Response = await fetch('src/maps/mouvements.txt');
 if(response.status === 200) {
 const splitedData = (await response.text()).split('\n');
  setDataPosition(parseInt(splitedData[0].split(',')[0]), parseInt(splitedData[0].split(',')[1]),splitedData[1])
}
  else
    error.value = response.statusText
}

// on set les positions récupérées dans la fonction précédente

function setDataPosition(posX : number, posY : number, directions : string) {
  [startXPosition, startYPosition,directionsAdventurer.value] = [posX, posY, directions];
  deplacingAdventurer(startXPosition, startYPosition)
}

// on parcourt toutes les positions, afin de set X & Y sur la map

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


// fonction de lancement du programme

async function main() {
  try {
    let arrayMap = await loadMapData() ?? [];

    await loadFileMouvements();

    if(actualXPosition.value < 0 || actualYPosition.value < 0 || arrayMap[actualYPosition.value][actualXPosition.value] === '#' || actualYPosition.value > arrayMap.length || actualXPosition.value > arrayMap[actualYPosition.value].length){
      error.value = "L'aventurier se trouve sur une case invalide, veuillez modifier ses directions ou ses coordonnées de départ";
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


