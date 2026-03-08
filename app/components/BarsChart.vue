<script setup lang="ts">
    const props = defineProps({
        positive: Number,
        divided: Number,
        negative: Number
    })

    const ReceptionData = [
        { 
            name: 'Positive', 
            percentage: props.positive ?? 0
        },{ 
            name: 'Divided', 
            percentage: props.divided ?? 0
        },{ 
            name: 'Negative', 
            percentage: props.negative ?? 0
        },
    ]

    const ReceptionCategories = computed(() => ({
        positive: {
            name: 'positive',
            color: '#22c55e'
        },
        divided: {
            name: 'divided',
            color: '#ffff00'
        },
        negative: {
            name: 'negative',
            color: '#ff0000'
        }
    }))
</script>

<template>
    <DonutChart
      :data="ReceptionData.map((i) => i.percentage)"
      :height="250"
      :categories="ReceptionCategories"
      :radius="0"
      :hide-legend="false"
      :arc-width="40"
      :legend-position="LegendPosition.bottomCenter"
      :legend-style="{ fontSize: '1.2rem' }"
      class="text-white cursor-pointer"
    />
</template>

<style>
    :root {
        --vis-legend-spacing: 20px;
    }
</style>