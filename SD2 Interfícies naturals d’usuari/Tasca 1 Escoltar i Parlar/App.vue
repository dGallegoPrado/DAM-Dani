<template>
    <button @click="startLlegir">ESCOLTAR</button>
    <p >{{ text }}</p>
    <button @click="startSpeak(text)" >Llegir</button>

</template>


<script setup>
import { ref, onMounted } from 'vue'

const text = ref('')
const error = ref('')
let recognition = null



onMounted(() => {
  speechSynthesis.getVoices() 

  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
  if (!SpeechRecognition) {
    error.value = 'Aquest navegador no suporta reconeixement de veu.'
    return
  }


  recognition = new SpeechRecognition()
  recognition.lang = 'ca-ES'
  recognition.interimResults = false
  recognition.maxAlternatives = 1


   recognition.onresult = (event) => {
    text.value = event.results[0][0].transcript
    error.value = ''
  }



  recognition.onerror = (event) => {
    error.value = event.error
  }
})


function startLlegir() {
  if (recognition) {
    recognition.start()
    error.value = ''
  }
}



function startSpeak(text){
    if ('speechSynthesis' in window) {
        let veusDisponibles = speechSynthesis.getVoices();
        let veuAnglesa;

        console.log ("Buscant veu angles");
        for (let i = 0; i < veusDisponibles.length; i++) {
            console.log (veusDisponibles[i].lang);
            if (veusDisponibles[i].lang.replace('_', '-').startsWith('en')) {
                veuAnglesa = veusDisponibles[i];
                break;
            }
        }

        if (veuAnglesa) {
            let textToSpeak = text;
            let utterance = new SpeechSynthesisUtterance(textToSpeak);
            utterance.voice = veuAnglesa;

            speechSynthesis.speak(utterance);
        } else {
              alert ("Aquest navegador no disposa de veu en anglès");
               }
    } else {
        alert ('API Web Speech no és suportada pel navegador');
    }

}

</script>


<style scoped>
.contenidor {
  text-align: center;
  margin-top: 40px;
}


.moviment {
  margin-top: 30px;
  height: 100px;
  overflow: hidden;
}
</style>
