import { DistributionChart } from '../../.nuxt/components';
<script setup lang="ts">
    interface ContentScore {
        word: String,
        positive: number;
        neutral: number;
        negative: number;
    }

    const sharedContentScore = useState('sharedContentScore', () => ([] as ContentScore[]));
</script>

<template>
    <h1 class="font-bold">| Content Analysis</h1>
    <main class="w-full h-full overflow-auto flex flex-col box-border p-5 gap-5 overflow-x-hidden">
        <div v-for="data in sharedContentScore" class="h-15 box-border w-full text-center flex items-center p-7 text-md bg-gray-700/30 rounded-lg justify-between">
            <span>{{ data.word }}</span>
            <DistributionChart :positive="data.positive" :divided="data.neutral" :negative="data.negative"></DistributionChart>
        </div>
    </main>
</template>