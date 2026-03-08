<script setup lang="ts">
    const props = defineProps({
        positive: Number,
        divided: Number,
        negative: Number
    })

    const distributionData = [
        {
            name: 'Reception',
            positive: props.positive ?? 0,
            divided: props.divided ?? 0,
            negative: props.negative ?? 0
        }
    ]

    const distributionCategories = computed(() => ({
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

    const format = (i: number): string => ``
    const tooltip = (j: {}): string => `Positive: ${props.positive?.toFixed(0)}%, Divided: ${props.divided?.toFixed(0)}%, Negative: ${props.negative?.toFixed(0)}%`
</script>

<template>
    <BarChart
        :data="distributionData"
        :height="30"
        :stacked="true"
        :categories="distributionCategories"
        :y-axis="['positive', 'divided', 'negative']"
        :group-padding="0"
        :bar-padding="0.2"
        :y-formatter="format"
        :radius="20"
        :orientation="Orientation.Horizontal"
        :hide-legend="true"
        :x-formatter="format"
        :tooltip-title-formatter="tooltip"
        class="cursor-pointer"
  />
</template>