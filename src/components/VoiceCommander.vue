<script setup>
import { useTheme } from 'vuetify';
const theme = useTheme(); 
import { ref, watch } from 'vue';
import { useSpeechRecognition } from '@/composables/useSpeechRecognition';

// Importem el composable amb la variable interimTranscript
const { isListening, transcript, interimTranscript, error, start } = useSpeechRecognition();

const uiMessage = ref("Prem el botó per començar...");
const statusColor = ref("primary");

// Variables per al Snackbar (Tasca 3)
const snackbar = ref(false);
const snackbarText = ref('');
const snackbarColor = ref('error');

// Lògica de reacció a la veu
watch(transcript, (newText) => {
  const command = newText.toLowerCase().trim();
  
  // Tasca de neteja: Tanca el Snackbar si s'estava mostrant abans de processar una nova comanda.
  snackbar.value = false; 

  if (command.includes('saluda')) {
    uiMessage.value = "Hola! Benvingut a l'aplicació.";
    statusColor.value = "success";
    alert("Hola!"); 
  } 
  else if (command.includes('ajuda')) {
    uiMessage.value = "Aquesta és una prova de concepte.";
    statusColor.value = "info";
  } 
  // Tasca 1: La comanda "Esborra"
  else if (command.includes('esborra') || command.includes('borrar') || command.includes('neteja')) {
    uiMessage.value = "Prem el botó per començar..."; // Estat inicial
    statusColor.value = "primary"; // Color inicial
    transcript.value = ''; // Netejar el transcript
  } 
  // Tasca 2: Control del Tema (Dark/Light Mode)
  else if (command.includes('mode fosc')) {
    uiMessage.value = "Mode fosc activat";
    statusColor.value = "primary";
    theme.global.name.value = 'dark'; // Activa el tema fosc
  } 
  else if (command.includes('mode clar')) {
    uiMessage.value = "Mode clar activat";
    statusColor.value = "primary";
    theme.global.name.value = 'light'; // Activa el tema clar
  }
  else {
    // Tasca 3: Feedback Visual (Snackbar) - Comanda no reconeguda
    uiMessage.value = `Comanda no reconeguda: "${newText}"`;
    statusColor.value = "warning";
    
    snackbarText.value = `Comanda NO vàlida: "${newText}"`;
    snackbarColor.value = 'error';
    snackbar.value = true; // Mostrar el Snackbar
  }
});
</script>

<template>
  <v-container class="d-flex justify-center align-center fill-height">
    

    <v-card width="400" :color="statusColor" variant="tonal">
      <v-card-item>
        <v-card-title class="text-h5 text-center">Control per Veu</v-card-title>
      </v-card-item>

      <v-card-text class="text-center py-4">
        <div class="mb-4">
          <v-icon 
            :icon="isListening ? 'mdi-microphone' : 'mdi-microphone-off'" 
            size="64"
            :class="{'text-red animate-pulse': isListening}"
          ></v-icon>
        </div>
        
        <p class="text-h6 font-weight-bold">{{ isListening ? 'Escoltant...' : 'En espera' }}</p>
        
        <p v-if="interimTranscript" class="text-caption text-grey">
            Detectant: {{ interimTranscript }}
        </p>
        
        <p class="mt-2 text-body-1">{{ uiMessage }}</p>
        
        <v-alert v-if="error" type="error" class="mt-3" density="compact">{{ error }}</v-alert>
      </v-card-text>

      <v-card-actions class="justify-center pb-4">
        <v-btn 
          variant="elevated" color="primary" size="large"
          @click="start" :disabled="isListening"
        >
          Escolta
        </v-btn>
        
      </v-card-actions>
    </v-card>
  </v-container>
  
  <v-container class="d-flex justify-center mt-4">
    <v-btn
      @click="theme.global.name.value = theme.global.current.value.dark ? 'light' : 'dark'"
      text="Toggle Light / Dark Manual"
      variant="outlined"
    ></v-btn>
  </v-container>
  
  <v-snackbar
    v-model="snackbar"
    :timeout="3000"
    :color="snackbarColor"
    location="bottom"
  >
    {{ snackbarText }}
    <template v-slot:actions>
      <v-btn
        icon="mdi-close"
        variant="text"
        @click="snackbar = false"
      ></v-btn>
    </template>
  </v-snackbar>
</template>

<style scoped>
.animate-pulse { animation: pulse 1.5s infinite; }
@keyframes pulse {
  0% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(1.1); }
  100% { opacity: 1; transform: scale(1); }
}
</style>