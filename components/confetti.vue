<template>
    <div/>
</template>
<script setup>
    const interval = ref(null)
    onMounted(() => {
        interval.value = setInterval(() => {
confetti({
        particleCount: 100,
        spread: 100,
        ticks: 500,
origin: {
    x: 0.5,
    // since they fall down, start a bit higher than random
    y: 0
  }
    });
    confetti({
        particleCount: 100,
        spread: 100,
     

origin: {
    x: Math.random(),
    // since they fall down, start a bit higher than random
    y: Math.random() - 0.2
  }
    });
        }, 500)
        
    })
    onUnmounted(() => {
        clearInterval(interval.value)
    })
</script>