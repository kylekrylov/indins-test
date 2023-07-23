<template>
  <div class="container">
    <div
      v-for="message in messages"
      :key="message.date"
      class="post"
    >
      <div class="post__header">
          <span class="post__date">
            {{ message.date }}
          </span>
        <span class="post__author">
            {{ message.authorName }}
          </span>
        <a
          :href="message.authorUrl"
          target="_blank"
          class="post__link"
        >
          link
        </a>
      </div>
      <div
        class="post__content"
        v-html='colorizeText(message.content, message.contentPostTones)'
      />
    </div>
    <Button
      @click="loadMoreMessages"
    >
      <span v-if="isLoading">
        Loading...
      </span>
      <span v-else-if="error">
        Error occurred while loading messages.
      </span>
      <span v-else>
        Load More 10
      </span>
    </Button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Button from "@/components/AtomButton.vue";

const messages = ref([]);
const isLoading = ref(true);
const error = ref(null);
const currentPage = ref(1);

const loadMessages = async () => {
  try {
    
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    const response = await fetch('http://localhost:3000/messages');
    const data = await response.json();
    
    messages.value = data.slice(0, currentPage.value * 10);
    isLoading.value = false;
    
  } catch (err) {
    error.value = err.message;
    isLoading.value = false;
  }
};

onMounted(() => {
  loadMessages();
  // document.documentElement.setAttribute('data-theme', 'light');
});

const colorizeText = (content, contentPostTones) => {
  
  // const text = content.replace(/\n/g, ' ').replace(/ {2,}/g, ' ')
  const text = content
  
  // Градиентные цвета
  const red = [255, 0, 0];
  const yellow = [255, 255, 0];
  const green = [0, 255, 0];
  
  // Функция для получения цвета в зависимости от тональности (от -1 до 1)
  const getColor = tone => {
    
    const interpolatedColor = tone < 0
      ? red.map((r, index) => Math.round(r + (yellow[index] - r) * (1 + tone)))
      : yellow.map((y, index) => Math.round(y + (green[index] - y) * tone));
    
    return `rgb(${interpolatedColor.join(', ')})`;
  }
  
  // Применяем раскрашивание для каждого элемента в массиве 'contentPostTones'
  let result = '';
  let currentPosition = 0;
  
  contentPostTones.forEach(({position, tone, length}) => {
    
    
    if (tone > 1 || tone < -1) console.log("не верно значение тональности")
    const color = getColor(tone);
    
    // Добавляем неокрашенный текст до текущей позиции
    result += text.slice(currentPosition, position);
    
    // Добавляем окрашенный текст внутри <span> тегов
    const coloredSubstring = text.slice(position, position + length);
    result += `<span style="background-color:${color};">${coloredSubstring}</span>`;
    
    // Обновляем текущую позицию
    currentPosition = position + length;
  });
  
  // Добавляем оставшийся неокрашенный текст после последней окрашенной подстроки
  result += text.slice(currentPosition);
  
  return result;
}

const loadMoreMessages = () => {
  isLoading.value = true;
  currentPage.value += 1;
  loadMessages();
};
</script>

<style lang="scss">
.post {
  margin-bottom: 20px;
  border-bottom: 8px solid #ccc;
  
  &__header {
    display: inline-flex;
    flex-wrap: wrap;
  }
  
  &__date {
    padding-right: 8px;
    font-size: 14px;
    white-space: nowrap;
  }
  
  &__author {
    padding-right: 8px;
    font-weight: 700;
  }
  
  &__content {
    margin-top: 8px;
    padding-bottom: 8px;
  }
}

</style>
