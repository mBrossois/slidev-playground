<template>
  <div v-if="$nav.currentPage > 1 && $nav.currentPage < 5">
    <div class="circle" >
      <p class="connections">{{ connnections }}</p>
    </div>
    <div class="bottom-bar">
        <span v-for="thumbsUp in thumbsUps" :key="thumbsUp" :style="left(thumbsUp)" class="thumbs-up fly-up">👍</span>
        <button class="raised-hand" :class="{show: isRaisedHand}" @click="hideRaisedHand">
          <span class="icon">✋</span>
        </button>
    </div>
  </div>
  <div v-else-if="$nav.currentPage !== 1 " class="topics">
    <div class="topics-list">
      <div v-for="topic of votesPerTopic" :key="topic.name" class="topic-circle" >
        <p class="name">{{ topic.name }}</p>
        <p class="connections">{{ topic.votes }}</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import PartySocket from 'partysocket';
import { Ref, ref } from 'vue';
import { useNav } from '@slidev/client'
import { getCookie, setCookie } from './utils/cookie'

const connnections = ref(0)

const { currentPage } = useNav()

const pageToTopic = {
  1: 'home',
  2: 'secondTopic',
  3: 'secondTopic',
  4: 'thirdTopic'
}

const cookieVotes = getCookie('votes')

const votesPerTopic = cookieVotes ? JSON.parse(cookieVotes) : {
  home:  {name: 'home', votes: 0},
  secondTopic: {name: 'second topic', votes: 0},
  thirdTopic: {name: 'third topic', votes: 0},
}
  
const partySocket = new PartySocket({
  host: import.meta.env.VITE_PARTYKIT_URL ?? '',
  room: 'vuejsamsterdam'
});

const left = (value: number) => {
  return {'--left': `${value%100}%`};
}

const isRaisedHand = ref(false)
function hideRaisedHand() {
  isRaisedHand.value = false
}

const thumbsUps: Ref<Array<number>> = ref([])
let currthumbsUp = 0
function addthumbsUp() {
    const id = currthumbsUp
    currthumbsUp += 1
    const topic = pageToTopic[currentPage.value]
    votesPerTopic[topic].votes += 1;
    setCookie('votes', JSON.stringify(votesPerTopic), 1);
    thumbsUps.value.push(id)
    setTimeout(() => {
      thumbsUps.value = thumbsUps.value.filter(value => value !== id)
    }, 5000)
}

// print each incoming message from the server to console
partySocket.addEventListener("message", (e) => {
  const message = e.data
  if(message === 'add-connection') {
    connnections.value += 1
    return;
  }
  if(message === 'lose-connection') {
    connnections.value -= 1
    return;
  }
  if(message === 'raised-hand') {
    isRaisedHand.value = true
  }

  if(message === 'thumbs-up') {
    addthumbsUp()
  }
});
</script>

<style scoped>
.circle {
  position: absolute;
  top: 0rem;
  right: 0rem;
  height: 100px;
  width: 100Px;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(0,0,0,0.2);
  backdrop-filter: saturate(20%) blur(5px);
  border-bottom-left-radius: 100%;
  padding-left: 1rem;
  padding-bottom: 1rem;
  z-index: 1000;
}

.circle .connections {
  font-family: sans-serif;
  font-size: 20px;
}

.raised-hand {
  position: absolute;
  right: 3rem;
  bottom: -2rem;

  height: 2rem;
  width: 2rem;
  transition: bottom ease-in-out 350ms;
}

.raised-hand.show {
  z-index: 1000;
  bottom: 4rem;
}

.raised-hand .icon {
  font-size: 3rem;
}

.thumbs-up {
  left: var(--left);
}

.fly-up {
    position: absolute;
    bottom: -16px;

    animation: up 2500ms ease-in forwards;
}

.topics {
  position: absolute;
  bottom: 2rem;
  left: 2rem;
  top: 2rem;
  right: 2rem;
  z-index: 100;
}

.topics-list {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.topic-circle {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;

  flex-shrink: 0;

  background: rgba(95, 0, 249);
  color: white;
  border-radius: 100%;
  padding: 1rem;
  aspect-ratio: 1;
  min-width: 130px;
}

.topic-circle .name {
  font-size: 18px;
}

@keyframes up {
    from {
        bottom: -40px;
    }
    to {
        bottom: 100vh;
    }
}
</style>