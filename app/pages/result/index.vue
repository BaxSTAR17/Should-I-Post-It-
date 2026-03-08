<template>
  <div class="h-screen bg-[#180033]">
    <Header />
    <div class="px-10 py-5 flex flex-col items-center gap-5">
      <div class="w-full flex flex-row gap-10">
        <div class="w-1/2">
            <InputArea v-model="sharedText" :rows="26" disabled/>
        </div>

        <div class="grow flex flex-col">
            <div class="m-2 p-2 grow text-white h-120">
                <ReceptionAnalysis v-if="resultMode == 0" :positive="sharedPrediction.levels.positive" :divided="sharedPrediction.levels.neutral" :negative="sharedPrediction.levels.negative"></ReceptionAnalysis>
                <CommunityFeedback v-if="resultMode == 1" :types="sharedPrediction.types"></CommunityFeedback>
                <ContentAnalysis v-if="resultMode == 2"></ContentAnalysis>
            </div>
            <div class="m-2 p-2 flex flex-row justify-between">
                <div class="bg-[#200047] p-1 border-2 border-[#200047] rounded-xl hover:border-white hover:bg-[#370082] cursor-pointer" @click="resultMode = (resultMode > 0) ? (resultMode - 1) % max : resultMode">
                    <UIcon name="tabler:arrow-big-left-filled" class="text-white" size="50" v-show="resultMode > 0"/>
                    <UIcon name="tabler:arrow-big-left-filled" class="text-gray opacity-30" size="50" v-show="resultMode == 0"/>
                </div>
                <div class="bg-[#200047] p-1 border-2 border-[#200047] rounded-xl hover:border-white hover:bg-[#370082] cursor-pointer" @click="resultMode = (resultMode < max - 1) ? (resultMode + 1) % max : resultMode">
                    <UIcon name="tabler:arrow-big-right-filled" class="text-white" size="50" v-show="resultMode < max - 1"/>
                    <UIcon name="tabler:arrow-big-right-filled" class="text-gray opacity-30" size="50" v-show="resultMode == max - 1"/>
                </div>
            </div>
        </div>
      </div>
      <UButton
          @click="goHome"
          class="ml-4 bg-[#200047] text-white hover:bg-[#370082]"
      >
        TRY ANOTHER POST
      </UButton>
    </div>
  </div>
</template>

<script setup lang="ts">
  import Header from '../../components/header.vue';
  import InputArea from '~/components/InputArea.vue';

  interface TypePredictionData {
    label: string;
    percent: number;
  }

  //const type_labels = ['unknown', 'sarcasm', 'harassment', 'hate_speech']

  interface LevelsPrediction {
    positive: number;
    neutral: number;
    negative: number;
    
  }

  interface Prediction {
    levels: LevelsPrediction;
    types: Array<TypePredictionData>;
  }

    interface ContentScore {
      word: String,
      positive: number;
      neutral: number;
      negative: number;
    }

    const sharedText = useState('postText', () => '');
    const sharedPrediction = useState('predictionData', () => ({} as Prediction));
    const sharedContentScore = useState('sharedContentScore', () => ([] as ContentScore[]));
    // const positive_score = useState('positive', () => '');
    // const divided_score = useState('divided', () => '');
    // const negative_score = useState('negative', () => '');
    // const positive = ref(Number(positive_score.value))
    // const divided = ref(Number(divided_score.value))
    // const negative = ref(Number(negative_score.value))
    const resultMode = ref(0)
    const max = ref(3);
    if(sharedText.value.split(' ').length <= 1) {
      max.value = 2;
    }

    const goHome = () => {
      sharedText.value = '';
      sharedContentScore.value = [];
      navigateTo('/');
    };
</script>