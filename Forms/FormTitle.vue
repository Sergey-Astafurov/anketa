<template>
    <div class="h3 mb-3 title__c">{{title}}</div>
    <hr class="mb-4" />
</template>

<script setup>

const props = defineProps({
  title: String
})


</script>

<style scoped>

.title__c{
  color: #555;
}

</style>
