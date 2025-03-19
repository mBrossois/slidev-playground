<template>
    <div class="circle" >
      <p>{{ connnections }}</p>
      {{ updatePage }}
    </div>
    <div class="bottom-bar">
        <span v-for="upvote in upvotes" :key="upvote" class="fly-up">👍</span>
        <span v-for="downvote in downvotes" :key="downvote" class="fly-up">👎</span>
    </div>
</template>

<script setup lang="ts">
import PartySocket from 'partysocket';
import { computed, Ref, ref, watch } from 'vue';

const connnections = ref(0)

const partySocket = new PartySocket({
  host: 'host-adres',
  room: 'roomies'
});


const upvotes: Ref<Array<number>> = ref([])
let currUpvote = 0
function addUpvote() {
    const id = currUpvote
    currUpvote += 1
    upvotes.value.push(id)
    setTimeout(() => {
        upvotes.value = upvotes.value.filter(value => value !== id)
    }, 5000)
}

const downvotes: Ref<Array<number>> = ref([])
let currDownvote = 0
function addDownvote() {
    const id = currDownvote
    currDownvote += 1
    downvotes.value.push(id)
    setTimeout(() => {
        downvotes.value = downvotes.value.filter(value => value !== id)
    }, 5000)
}

watch($page, () => {
  partySocket.send(JSON.stringify({messageType: 'results', value: ''}))
})

// print each incoming message from the server to console
partySocket.addEventListener("message", (e) => {
  const {messageType, value} = JSON.parse(e.data)
  if(messageType === 'connections') {
    connnections.value = value - 1
    return;
  }
  if(messageType === 'results') {
    console.log(value)
    return;
  }
  if(messageType === 'vote') {
    if(value === 'upvote') {
        addUpvote()
    }
    if(value === 'downvote') {
        addDownvote()
    }
    return;
  }
});
</script>

<style scoped>
.circle {
  position: absolute;
  bottom: 1rem;
  right: 1rem;
  height: 75px;
  width: 75px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: blue;
  border-radius: 100%;
}
.fly-up {
    position: absolute;
    bottom: -16px;

    animation: up 2500ms ease-in;
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