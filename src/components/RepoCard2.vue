<script setup lang="ts">
import StarLarge from '@/icons/star-icons/StarLarge.vue';
import StarMiddle from '@/icons/star-icons/StarMiddle.vue';
import UserIcon1 from '@/icons/user-icons/UserIcon1.vue';
import { IonCard, IonCardContent, IonCardSubtitle, IonCardHeader, IonCardTitle } from '@ionic/vue'
import { Octokit } from '@octokit/rest'
import { inject, ref } from 'vue'

const oc = inject('oc') as Octokit

const props = defineProps<{
  name: string
  owner: string
}>()

const stared = ref((await oc.rest.activity.checkRepoIsStarredByAuthenticatedUser({
  owner: props.owner,
  repo: props.name,
})).data)

function findKeyWithMaxValue(obj: any) {
  let maxKey = null;
  let maxValue = -Infinity;

  for (const key in obj) {
    if (obj[key] > maxValue) {
      maxValue = obj[key];
      maxKey = key;
    }
  }

  return maxKey;
}

async function starClicked() {
  if (!stared.value)
    await oc.rest.activity.starRepoForAuthenticatedUser({
      owner: props.owner,
      repo: props.name
    })
  else
    await oc.rest.activity.unstarRepoForAuthenticatedUser({
      owner: props.owner,
      repo: props.name
    })
  stared.value = !stared.value as never
}

const { data } = await oc.rest.repos.get({
  owner: props.owner,
  repo: props.name
})

const contributors = (await oc.rest.repos.listContributors({
  owner: props.owner,
  repo: props.name
})).data

console.log((await oc.rest.repos.listLanguages({
  owner: props.owner,
  repo: props.name
})).data)

const response = await fetch('https://raw.githubusercontent.com/ozh/github-colors/master/colors.json');
const colors = await response.json();
const languageName = findKeyWithMaxValue((await oc.rest.repos.listLanguages({
  owner: props.owner,
  repo: props.name
})).data) as string
const color = colors[languageName].color
</script>

<template>
  <ion-card>
    <ion-card-header>
      <div class="flex flex-row items-center">
        <img :src="data.owner.avatar_url" class="w-[16px] h-[16px] rounded-[4px] mr-2">
        <span class="text-[15px] font-black">{{ data.owner.login }}</span>
        <span class="text-[15px] font-black text-[rgba(60,60,67,0.60)]">/</span>
        <span class="text-[15px] font-black">{{ data.name }}</span>
      </div>
    </ion-card-header>
    <ion-card-content>
      <div class="mb-[12px]">{{ data.description }}</div>
      <div class="flex flex-row items-center">
        <div class="flex flex-col float-left w-full">
          <div class="flex flex-row text-[rgba(60,60,67,0.60)] items-center">
            <span class="mr-1 w-[16px] h-[16px] rounded-full" :style="{
              backgroundColor: color
            }"></span>
            {{ languageName }}
          </div>
          <div class="flex flex-row text-[rgba(60,60,67,0.60)] items-center">
            <UserIcon1 class="mr-1"/>
            {{ contributors.length }}
          </div>
        </div>
        <div class="float-right">
          <StarLarge :clicked="stared" @click="starClicked()"/>
        </div>
      </div>
    </ion-card-content>
  </ion-card>
</template>