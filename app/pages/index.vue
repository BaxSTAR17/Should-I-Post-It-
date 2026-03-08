<template>
  <div class="h-screen bg-[#180033]" v-if="!loading">
    <Header />
    <div class="text-center">
      <h2 class="text-white text-2xl font-semibold mt-10">Welcome to Should I Post It?</h2>
      <p class="text-white mt-4 text-center box-border px-10">
        SIP is an online tool for all those hardcore social media posters. This tool is used to predict the reception of your social media post using our own built-in model trained using tens of thousands of existing social media posts. This guarantees that you understand thee value of your content next time you click post!
      </p>
    </div>
    <div class="px-10 py-5 w-screen flex flex-col items-center gap-5">
      <div class="w-full flex flex-col gap-2">
        <InputArea v-model="sharedText" :rows="19"/>
      </div>
      <div v-if="sharedText" class="flex gap-3">
          <UButton
            @click="predict(sharedText, true)"
            class="ml-4 bg-purple-600 cursor-pointer text-white hover:bg-[#370082]"
        >
          SHOULD I POST IT?
        </UButton>
        <UButton
            @click="sharedText = ''"
            class="ml-4 bg-[#200047] cursor-pointer text-white hover:bg-[#370082]"
        >
          CLEAR TEXT
        </UButton>
      </div>
    </div>
  </div>
  <div class="h-screen bg-[#180033] flex flex-col justify-center items-center" v-else>
    <img src="../assets/sip_4.png" alt="SIP_Logo" style="scale: 0.3">
    <UProgress :v-model="ref(null)" class="w-100" color="neutral"/>
  </div>
</template>

<script setup lang="ts">
  import Header from '../components/header.vue';
  import InputArea from '~/components/InputArea.vue';

  const sharedText = useState('postText', () => '');
  const sharedPrediction = useState('predictionData', () => ({} as Prediction));
  const sharedContentScore = useState('sharedContentScore', () => ([] as ContentScore[]));
  const loading = ref(false);
  const goToResult = () => {
    if (!sharedText.value) return; 

    navigateTo('/result');
  };

  interface ContentScore {
    word: String,
    positive: number;
    neutral: number;
    negative: number;
  }

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

  let loaded_model: tf.LayersModel;
  let loaded_vocab: Map<string, number> = new Map();

  const prediction_data = ref<Prediction | null>(null);

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

  const predict = async (text: string, content_mode: boolean) => {
    loading.value = true;
    const inputTensor = tokenize(text);
    if (loaded_model){
      //const input = tf.tensor([text], [1], 'string');
      const [scoreOutput, typeOutput] = loaded_model.predict(inputTensor) as tf.Tensor[];

      const scores = await scoreOutput!.data(); // [Pos, Neu, Neg]
      const types = await typeOutput!.data();   // [unknown, sarcasm, etc.]


      const result = {
        levels: { 
          positive: Number((scores[0]!*100).toFixed(2)), 
          neutral: Number((scores[1]!*100).toFixed(2)), 
          negative: Number((scores[2]!*100).toFixed(2))
        },
        types: Array.from(types).map((e, i) => ({
          label: type_labels[i]!,
          percent: Number((e*100).toFixed(2)),
        }))
      }

      if(content_mode == false) {
        return result;
      }
      prediction_data.value = result;
      // let levels = prediction_data.value.levels

      // positive.value = levels.positive+1
      // neutral.value = levels.neutral+1
      // negative.value = levels.negative+1
      
      // meter_max.value = (levels.negative+1) + (levels.neutral+1) + (levels.positive+1) 

      // let types = prediction_data.value.type
      // primary_type.value = types.primary.label
      // primary_level.value= types.primary.percent
      // secondary_type.value = types.secondary.label
      // secondary_level.value= types.secondary.percent
      sharedPrediction.value = prediction_data.value

      const words = sharedText.value.split(" ");
      if(content_mode == true && words.length > 1) {
        for(let word of words) {
          const result = await predict(word, false);
          if(result) {
            const new_content_score = {
            word: word,
              positive: result.levels.positive,
              neutral: result.levels.neutral,
              negative: result.levels.negative,
            } as ContentScore;
            sharedContentScore.value.push(new_content_score);
          }
        }
        sharedText.value = words.join(" ");
      }
      goToResult();
    }
  }

</script>