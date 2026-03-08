<script setup lang="ts">
    interface TypePredictionData {
        label: string;
        percent: number;
    }

    const props = defineProps({
        types: Array<TypePredictionData>
    })

    type Classification = 'unknown' | 'sarcasm' | 'harrassment' | 'hate speech'

    type Flag = 'green' | 'yellow' | 'red'

    const current_flag = ref(('yellow' as Flag));
    const types = props.types;
    const primary = ref('');
    const secondary = ref('');
    let primary_percentage = 0;
    let secondary_percentage = 0;
    for(let type of types!) {
        if(primary_percentage <= type.percent) {
            secondary_percentage = primary_percentage
            secondary.value = primary.value
            primary_percentage = type.percent;
            primary.value = type.label;
        }
    }
    if(primary.value == 'unknown') {
        current_flag.value = 'green';
    } else if(primary.value == 'sarcasm' || secondary.value == 'unknown') {
        current_flag.value = 'yellow';
    } else {
        current_flag.value = 'red';
    }
</script>

<template>
    <h1 class="font-bold">| Community Feedback</h1>
    <main class="w-full h-full flex flex-row box-border p-5 gap-5">
         <section class="h-full w-full rounded-xl position-relative border-solid border-3 bg-green-900/30 border-green-300" v-if="current_flag == 'green'">
            <div class="position-absolute w-full h-full flex flex-col justify-center items-center gap-5">
                <p class="text-center text-green-300"><b class="text-green-200 text-3xl">GREEN FLAG</b></p>
                <UIcon name="tabler:flag-check" class="text-green-300 text-9xl"/>
                <p class="text-center text-green-300">There's no reason to report or flag your post</p>
            </div>
         </section>
         <section class="h-full w-full rounded-xl position-relative border-solid border-3 bg-yellow-500/30 border-yellow-300" v-if="current_flag == 'yellow'">
            <div class="position-absolute w-full h-full flex flex-col justify-center items-center gap-5">
                <p class="text-center text-yellow-300"><b class="text-yellow-200 text-3xl">YELLOW FLAG</b></p>
                <UIcon name="tabler:flag-question" class="text-yellow-300 text-9xl"/>
                <p class="text-center text-yellow-300">WARNING: Your post may get reported.</p>
            </div>
         </section>
         <section class="h-full w-full rounded-xl position-relative border-solid border-3 bg-red-900/30 border-red-500 box-border" v-if="current_flag == 'red'">
            <div class="position-absolute w-full h-full flex flex-col justify-center items-center gap-5">
                <p class="text-center text-red-300"><b class="text-red-300 text-3xl">RED FLAG</b></p>
                <UIcon name="tabler:flag-x" class="text-red-400 text-9xl"/>
                <p class="text-center text-red-300 m-0 p-3">IMPORTANT: Your post has a <b>HIGH</b> chance of being reported.</p>
            </div>
         </section>
         <section class="h-full w-full flex flex-col gap-5" v-if="current_flag == 'yellow' || current_flag == 'red'">
            <p class="text-red-200 text-center">Your post can be classified as:</p>
            <div class="h-full w-full bg-red-900/30 rounded-xl flex flex-col justify-center items-center" v-if="primary != 'unknown'">
                <b class="text-red-300 text-3xl text-center underline">{{ primary?.replace('_', ' ').toUpperCase() }}</b>
                <b class="text-red-300 text-4xl text-center">{{ primary_percentage }}%</b>
            </div>
            <div class="h-full w-full bg-red-900/30 rounded-xl flex flex-col justify-center items-center" v-if="secondary != 'unknown'">
                <b class="text-red-300 text-3xl text-center underline">{{ secondary?.replace('_', ' ').toUpperCase() }}</b>
                <b class="text-red-300 text-4xl text-center">{{ secondary_percentage }}%</b>
            </div>
         </section>
    </main>
</template>