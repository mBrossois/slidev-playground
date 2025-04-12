<template>
    <div class="circle" >
      <p class="connections">{{ connnections }}</p>
    </div>
    <div class="bottom-bar">
        <span v-for="thumbsUp in thumbsUps" :key="thumbsUp" :style="left(thumbsUp)" class="thumbs-up fly-up">👍</span>
        <button class="raised-hand" :class="{show: isRaisedHand}" @click="hideRaisedHand">
          <span class="icon">✋</span>
        </button>
    </div>
</template>

<script setup lang="ts">
import PartySocket from 'partysocket';
import { computed, Ref, ref, watch } from 'vue';

const connnections = ref(0)

const partySocket = new PartySocket({
  host: 'host',
  room: 'roomies'
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
  background: rgba(0,0,0,0.8);
  backdrop-filter: saturate(180%) blur(10px);
  border-bottom-left-radius: 100%;
  padding-left: 1rem;
  padding-bottom: 1rem;
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

@keyframes up {
    from {
        bottom: -40px;
    }
    to {
        bottom: 100vh;
    }
}
</style>