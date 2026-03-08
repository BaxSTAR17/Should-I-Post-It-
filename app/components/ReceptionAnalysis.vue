import { DistributionChart, UIcon } from '../../.nuxt/components';
<script setup lang="ts">
    const props = defineProps({
        positive: Number,
        divided: Number,
        negative: Number
    })

    type ReceptionLevel = 'Positive' | 'Mixed' | 'Negative'

    const current_reception = ref(('Mixed' as ReceptionLevel));
    if((props.positive ?? 0) >= 50) current_reception.value = 'Positive';
    else if((props.negative ?? 0) >= 50) current_reception.value = 'Negative';
    else current_reception.value = 'Mixed';
</script>

<template>
    <h1 class="font-bold">| Reception Analysis</h1>
    <main class="w-full h-full flex flex-row box-border p-5 gap-5">
         <section :class="`h-full min-w-1/2 w-full rounded-xl position-relative border-solid border-3 bg-green-900/30 border-green-300`" v-if="current_reception == 'Positive'">
            <div class="position-absolute w-full h-full flex flex-col justify-center items-center">
                <p class="text-center text-green-300">Your post's reception is: <br> <b class="text-green-200 text-3xl">POSITIVE</b></p>
                <UIcon name="tabler:mood-check" class="text-green-300 text-9xl"/>
            </div>
         </section>
         <section :class="`h-full min-w-1/2 w-full rounded-xl position-relative border-solid border-3 bg-yellow-500/25 border-yellow-300`" v-if="current_reception == 'Mixed'">
            <div class="position-absolute w-full h-full flex flex-col justify-center items-center">
                <p class="text-center text-yellow-300">Your post's reception is: <br> <b class="text-yellow-200 text-3xl">MIXED</b></p>
                <UIcon name="tabler:mood-puzzled" class="text-yellow-300 text-9xl"/>
            </div>
         </section>
         <section :class="`h-full min-w-1/2 w-full rounded-xl position-relative border-solid border-3 bg-red-900/30 border-red-500`" v-if="current_reception == 'Negative'">
            <div class="position-absolute w-full h-full flex flex-col justify-center items-center">
                <p class="text-center text-red-300">Your post's reception is: <br> <b class="text-red-300 text-3xl">NEGATIVE</b></p>
                <UIcon name="tabler:mood-cry" class="text-red-400 text-9xl"/>
            </div>
         </section>
         <section class="h-full w-full flex flex-col justify-between">
            <BarsChart :positive="props.positive" :divided="props.divided" :negative="props.negative"></BarsChart>
            <DistributionChart :positive="props.positive" :divided="props.divided" :negative="props.negative"></DistributionChart>
            <div class="flex flex-row justify-around text-sm font-bold"><span class="text-green-200">Positive: {{ props.positive }}%</span><span class="text-yellow-200">Divided: {{ props.divided }}%</span><span class="text-red-300">Negative: {{ props.negative }}%</span></div>
         </section>
    </main>
</template>