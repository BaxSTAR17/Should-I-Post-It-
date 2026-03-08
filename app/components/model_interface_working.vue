<template>
  <div>
    <UCard>
      <template #header>
        Input here
      </template>
      <template #default>
        <UInput v-model="model_input" />
      </template>
      <template #footer>
        <UButton label="Submit" @click="predict(model_input)"/>
      </template>
    </Ucard>

    <UCard v-if="prediction_data">
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
          <template v-for="score in prediction_data.types">
            <div v-if="score.label != 'unknown' && Math.round(score.percent) != 0"> {{  score.label  }}: {{ score.percent }}%</div>
          </template>
        </div>
      </template>
    </Ucard>
  </div>
</template>

<script setup lang="ts">
import * as tf from '@tensorflow/tfjs';

interface TypePredictionData {
  label: string;
  percent: number;
}

const type_labels = ['unknown', 'sarcasm', 'harassment', 'hate_speech']

interface LevelsPrediction {
  positive: number;
  neutral: number;
  negative: number;
  
}

interface Prediction {
  levels: LevelsPrediction;
  types: Array<TypePredictionData>;
}

// const MODEL_URL = "localhost:3000/models/tfjs_revised_model/model.json"
// const VOCAB_URL = "localhost:3000/models/tfjs_revised_model/vocab.json"
const MAX_LEN = 200
const model_input = ref("")

let loaded_model: tf.LayersModel;
let loaded_vocab: Map<string, number> = new Map();

const prediction_data = ref<Prediction | null>(null);
const meter_max = ref(0)

const positive = ref<number>()
const neutral = ref<number>()
const negative = ref<number>()

onMounted(async () => {
    // These will now work because they only execute in the browser
    const model = await tf.loadLayersModel('/models/tfjs_downgrade_model/model.json');
    const vocabResponse = await fetch('/models/tfjs_downgrade_model/vocab.json');
    const vocab: string[] = await vocabResponse.json();

    vocab.forEach((word, index) => {
      loaded_vocab.set(word, index);
    });
    // Assign to your refs
    loaded_model = model;
});

const tokenize = (text: string) => {
  // Basic cleanup to match your Python vectorizer
  const sequence = text.toLowerCase()
    .replace(/[^\w\s]/g, '')
    .split(/\s+/)
    .map(word => {
      const id = loaded_vocab.get(word) || 1;
      return id >= 19969 ? 1 : id;
    }) // 1 is usually the [UNK] token
    .slice(0, MAX_LEN);

  // Padding (Pre-padding or Post-padding based on your training)
  while (sequence.length < MAX_LEN) {
    sequence.push(0); // 0 is usually the [PAD] token
  }
  return tf.tensor2d([sequence], [1, MAX_LEN], 'int32');
}


const predict = async (text: string) => {

  const inputTensor = tokenize(text);
  console.log('test')
  if (loaded_model){
    const input = tf.tensor([text], [1], 'string');
    const [scoreOutput, typeOutput] = loaded_model.predict(inputTensor) as tf.Tensor[];

    const scores = await scoreOutput!.data(); // [Pos, Neu, Neg]
    const types = await typeOutput!.data();   // [unknown, sarcasm, etc.]


    const result = {
      levels: { positive: scores[0]!*100, neutral: scores[1]!*100, negative: scores[2]!*100 },
      types: Array.from(types).map((e, i) => ({
        label: type_labels[i]!,
        percent: Number((e*100).toFixed(2)),
      }))
    }

    prediction_data.value = result;
    let levels = prediction_data.value.levels

    positive.value = levels.positive
    neutral.value = levels.neutral
    negative.value = levels.negative
    
    meter_max.value = (positive.value) + (neutral.value) + (negative.value) 
    
  }
  else {
    alert("Model has not loaded!");
  }
}

</script>

<style scoped>

</style>