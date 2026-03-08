<template>
  <div class="flex flex-col gap-4">
    <UCard>
      <template #header>
        Input here
      </template>
      <template #default>
        <UInput class="w-full" v-model="input" />
      </template>
      <template #footer>
        <UButton label="Submit" @click="predict()"/>
      </template>
    </Ucard>

    <UCard v-if="data">
      <template #header>
        Results
      </template>
      <template #default>
          <div>
            <span>Negative: </span>
            <UProgress color="error" v-model="negative" />
          </div>
          <div>
            <span>Neutral: </span>
            <UProgress color="neutral" v-model="neutral" />
          </div>
          <div>
            <span>Positive: </span>
            <UProgress color="primary" v-model="positive" />
          </div>
      </template>
      <template #footer>
        <div v-if="negative! > positive!">
          <span class="font-bold text-xl">Additional Context:</span>
          <div v-if="primary_type != 'unknown' && primary_level != 0"> {{  primary_type  }}: {{ primary_level }}%</div>
          <div v-if="secondary_type != 'unknown' && secondary_level != 0">{{  secondary_type  }}: {{ secondary_level }}%</div>
          
        </div>
      </template>
    </Ucard>

    
  </div>
</template>

<script setup lang="ts">
import * as tf from '@tensorflow/tfjs';

const MODEL_URL = "http://time-shop.gl.joinmc.link/predict?query="
const input = ref<string>('')
const request_input = computed(() => btoa(input.value));

interface TypePredictionData {
  label: string;
  percent: number;
}

interface TypePrediction {
  primary: TypePredictionData;
  secondary: TypePredictionData;
}

interface LevelsPrediction {
  positive: number;
  neutral: number;
  negative: number;
  
}

interface Prediction {
  levels: LevelsPrediction;
  type: TypePrediction;
}

const prediction_data = ref<Prediction | null>(null);
const meter_max = ref(0)

const levels = ref<LevelsPrediction>()
const positive = ref<number>()
const neutral = ref<number>()
const negative = ref<number>()

const primary_type = ref<string>()
const primary_level = ref<number>()
const secondary_type = ref<string>()
const secondary_level = ref<number>()


const { data , pending, error, refresh } = await useFetch<Prediction>(() => MODEL_URL + request_input.value, {
  immediate: false,
  watch: false // Prevents it from running every time you type
});

const predict = async () => {
  await refresh();
  if (data.value) {
    prediction_data.value = data.value;
    let levels = prediction_data.value.levels

    positive.value = levels.positive+1
    neutral.value = levels.neutral+1
    negative.value = levels.negative+1
    
    meter_max.value = (levels.negative+1) + (levels.neutral+1) + (levels.positive+1) 

    let types = prediction_data.value.type
    primary_type.value = types.primary.label
    primary_level.value= types.primary.percent
    secondary_type.value = types.secondary.label
    secondary_level.value= types.secondary.percent
    
    
  }
}

</script>

<style scoped>

</style>