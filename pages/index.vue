<template>
    <main class="main-wrapper">
        <transition-group name="slide-fade" >
        <section v-if="choreToDo" key="doChore">
            <h1>{{choreToDo.name}}</h1>
            <button @click="completeChore(choreToDo)" :disabled="loading" :aria-disabled="loading" >Done!</button>
            <div class="underline" style="padding-top: 24px; cursor: pointer; color: rgba(0,0,0,0.8)" @click="getChore(choreToDo)" >Give me a different chore</div>
        </section>
        <section v-else-if="choreCompleted">
 <h1>Great job!</h1>
  <button @click="getChore" :disabled="loading" :aria-disabled="loading" >Give me another chore</button>
        </section>
        <section v-else-if="allChoresCompleted" key="choresDone">
            <h1>All your chores are done!</h1>
        </section>
        <section v-else key="getChore">
        <h1>So you want a chore?</h1>
        <button @click="getChore" :disabled="loading" :aria-disabled="loading">Give me a chore</button>
        </section>
        </transition-group>
        <Confetti v-if="confetti"/>
    </main>
</template>
<style lang="scss">
body {
    margin: 0
}
    .main-wrapper {
        height: 100vh;
        width: 100vw;
        display: flex;
        justify-content: center;
        align-items: center;
        section {
           display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center; 
        h1 {
            white-space: nowrap;
        }
        }
    }
    .slide-fade-enter-active {
  transition: all 0.2s ease-out;
  transition-delay: 0.3s;
}

.slide-fade-leave-active {
  transition: all 0.2s ease-out;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  opacity: 0;
}
.slide-fade-leave-to {
     transform: translateX(300px);
}
.slide-fade-enter-from {
     transform: translateX(-300px);
}
</style>
<script setup>

    const choreToDo = ref(null)
    const client = useSupabaseClient();
    const chores = ref([])
    const allChoresCompleted = ref(false)
    const loading = ref(false)
    const choreCompleted = ref(false)

    const confetti = computed(() => {
        return allChoresCompleted.value || choreCompleted.value
    })
    

     const differenceDays = (day) => {
        const now = new Date();
        const timeDif = now.getTime() - day.getTime();
        return timeDif / (1000 * 3600 * 24);
    }
    const randomIntFromInterval = (min, max) => { // min and max included 
        return Math.floor(Math.random() * (max - min + 1) + min)
      }
   
   const getChores = async () => {
const {data} =  await client.from('chores').select()
return data;
   }
    const getChore = async (notThisChore) => {
        loading.value = true;
        choreCompleted.value = false;
        if (!chores.value.length) {
  chores.value = await getChores();
        }
       
        const choresToDo = chores.value.filter((chore) => {
            if (chore.id == notThisChore?.id) return false
            return differenceDays(new Date(chore.last_completed)) > chore.frequency   
        })
        if (!choresToDo.length) {
            allChoresCompleted.value = true;
            return;
        }
        const randomIndex = randomIntFromInterval(0, choresToDo.length - 1);
        
        choreToDo.value = choresToDo[randomIndex]
        loading.value = false;
    }

    const completeChore = async ({id}) => {
        loading.value = true;
        const {data, error} = await client.from('chores').update({last_completed: new Date()}).eq('id', id).select()
        if (!error) {
            const index = chores.value.findIndex((c) => c.id === data[0].id)
            chores.value.splice(index, 1, data[0])
            choreToDo.value = null;
            choreCompleted.value = true;
        }
        loading.value = false;
    }
    
</script>