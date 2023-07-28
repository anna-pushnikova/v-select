<script setup lang="ts">
enum DogPedegree {
  Labrador = "labrador",
  Shepherd = "shepherd",
  Chihua = "Chihua",
}

const data = ref({
  params: {
    type: DogPedegree.Labrador,
    name: "",
  },
})

interface DogOption {
  id: DogPedegree
  text: string
}

const typeOptions = [
  { id: DogPedegree.Labrador, text: "Labrador" },
  { id: DogPedegree.Shepherd, text: "Shepherd" },
  { id: DogPedegree.Chihua, text: "Chihua" },
]

// Простые селекты
const simpleTypeObj = ref<DogOption>(
  typeOptions.find((option) => option.id == data.value.params.type) ??
    typeOptions[0]
)
</script>

<template>
  <div class="playground">
    <div class="sm">
      <div class="col">
        <h2>Simple selects</h2>

        <VSelect
          v-model="simpleTypeObj"
          label="Select a dog"
          :dv="(option) => option?.text"
          :error="'ОШибка'"
          id="select"
          by="id"
          placeholder="select a dog"
          size="lg"
        >
          <VSelectOption
            v-for="option in typeOptions"
            :key="option.id"
            :value="option"
            size="sm"
            >{{ option.text }}</VSelectOption
          >
        </VSelect>
      </div>
    </div>
  </div>
</template>

<style scoped>
.playground {
  min-height: calc(100vh - 64px);
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-gap: 16px;
  padding: 24px;
}
</style>