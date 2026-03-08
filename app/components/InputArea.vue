<template>
    <div class="relative w-full h-full"> 
        <UTextarea
            v-model="textContent"
            placeholder="Enter your post here..."
            type="text"
            class="w-full h-full resize-none"
            :ui="{base: 'bg-[#200047]'}" 
            :rows="rows"
            :disabled="disabled" 
            autoresize
        />

        <div class="absolute bottom-3 right-3 flex gap-2">
            <UButton @click="speakText" :disabled="isSpeaking" size="sm" class="cursor-pointer">
                <UIcon name="tabler:volume" class="text-xl text-gray-900"/>
            </UButton>
            <UButton @click="stopSpeech" :disabled="!isSpeaking" size="sm" class="cursor-pointer">
                <UIcon name="tabler:player-pause-filled" class="text-xl text-gray-900"/>
            </UButton>
        </div>
    </div>
</template>

<script setup lang="ts">
    import { ref } from 'vue';

    withDefaults(defineProps<{
        rows?: number,
        disabled?: boolean 
    }>(), {
        rows: 19,
        disabled: false 
    });

    const textContent = defineModel<string>({ default: '' }); 
    const isSpeaking = ref(false);

    const speakText = () => {
        if (!textContent.value) return; 

        window.speechSynthesis.cancel();
        const utterance = new SpeechSynthesisUtterance(textContent.value);
        
        utterance.onstart = () => isSpeaking.value = true;
        utterance.onend = () => isSpeaking.value = false;
        
        window.speechSynthesis.speak(utterance);
    };

    const pauseSpeech = () => {
        window.speechSynthesis.pause();
        isSpeaking.value = false;
    };

    const stopSpeech = () => {
        window.speechSynthesis.cancel();
        isSpeaking.value = false;
    };
</script>