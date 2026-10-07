<template>
  <div class="home">
    <nav class="navbar">
      <div class="logo">🐾</div>
      <div class="menu">
        <a
          v-for="item in menu"
          :key="item.key"
          href="#"
          :class="{ active: page === item.key }"
          @click.prevent="goTo(item.key)"
        >
          {{ item.label }}
        </a>
      </div>
    </nav>
    <section v-if="page === 'home'" class="hero" :style="heroStyle">
      <div class="overlay"></div>
      <div class="hero-content">
        <h1>Say Hello To Your New Buddy</h1>
      </div>
    </section>
    <section v-else-if="page === 'random'" class="random-section">
      <h2>Random Dog</h2>
      <button :disabled="loading" @click="getRandomDog">
        {{ loading ? 'Loading...' : 'Get Random Dog' }}
      </button>
      <p v-if="error" class="error">Error: {{ error }}</p>
      <div v-if="randomImage" class="dog-card">
        <img :src="randomImage" alt="Random Dog" />
      </div>
    </section>
    <section v-else-if="page === 'breeds'" class="breeds-section">
      <h2>Dog Breeds</h2>
      <p v-if="breedsLoading">Loading...</p>
      <template v-else>
        <div class="search-box">
          <input
            v-model="query"
            type="text"
            class="breed-search"
            placeholder="Search breed..."
            autocomplete="off"
            @focus="open = true"
            @input="open = true"
            @blur="open = false"
            @keydown.enter.prevent="selectBreed(filteredOptions[0])"
            @keydown.esc="open = false"
          />
          <ul v-if="open && filteredOptions.length" class="suggestions">
            <li
              v-for="b in filteredOptions"
              :key="b.value"
              @mousedown.prevent="selectBreed(b)">
              {{ b.label }}
            </li>
          </ul>
          <p v-else-if="open && query" class="no-result">No breed found</p>
        </div>
        <p v-if="error" class="error">Error: {{ error }}</p>
        <p v-if="imagesLoading">Loading photos...</p>
        <div v-if="breedImages.length" class="photo-grid">
          <img
            v-for="src in breedImages"
            :key="src"
            :src="src"
            :alt="selectedLabel"
            loading="lazy"
          />
        </div>
        <button
          v-if="selectedBreed"
          class="more-btn"
          :disabled="imagesLoading"
          @click="loadBreedImages"
        >
          More photos
        </button>
      </template>
    </section>
    <section v-else class="about-section">
      <h2>About Us</h2>
      <p>
        This website shows dog photos from the free
        <a href="https://dog.ceo/dog-api/documentation" target="_blank" rel="noopener">Dog API</a>.
        Browse breeds, or get a random dog whenever you like.
      </p>
    </section>
  </div>
</template>
<script setup>
import { ref, computed, onMounted } from 'vue';
const API = 'https://dog.ceo/api';
const menu = [
  { key: 'home', label: 'Home' },
  { key: 'random', label: 'Random Dog' },
  { key: 'breeds', label: 'Breeds' },
  { key: 'about', label: 'About' },
];
const page = ref('home');
const randomImage = ref('');
const heroImage = ref('');
const breeds = ref({});
const selectedBreed = ref('');
const breedImages = ref([]);
const query = ref('');
const open = ref(false);
const randomLoading = ref(false);
const imagesLoading = ref(false);
const error = ref('');
const heroStyle = computed(() => ({
  backgroundImage: heroImage.value ? `url(${heroImage.value})` : 'none',
}));
const fetchDog = async (path) => {
  const res = await fetch(`${API}${path}`);
  const data = await res.json();
  if (data.status !== 'success') {
    throw new Error(data.message || 'Failed to fetch dog data');
  }
  return data.message;
};
const getRandomDog = async () => {
  randomLoading.value = true;
  error.value = '';
  try {
    randomImage.value = await fetchDog('/breeds/image/random');
  } catch (e) {
    error.value = e.message;
  } finally {
    randomLoading.value = false;
  }
};
const loadBreeds = async () => {
  try {
    breeds.value = await fetchDog('/breeds/list/all');
  } catch (e) {
    error.value = e.message;
  }
};
const cap = (t) => t.charAt(0).toUpperCase() + t.slice(1);
const breedOptions = computed(() => {
  const list = [];
  for (const [name, subs] of Object.entries(breeds.value)) {
    if (subs.length === 0) {
      list.push({ value: name, label: cap(name) });
    } else {
      for (const sub of subs) {
        list.push({ value: `${name}/${sub}`, label: `${cap(sub)} ${cap(name)}` });
      }
    }
  }
  return list;
});
const selectedLabel = computed(
  () => breedOptions.value.find((b) => b.value === selectedBreed.value)?.label || 'Dog'
);
const filteredOptions = computed(() => {
  const q = query.value.trim().toLowerCase();
  if (!q) return breedOptions.value;
  return breedOptions.value.filter(
    (b) => b.label.toLowerCase().includes(q) || b.value.toLowerCase().includes(q)
  );
});
const loadBreedImages = async () => {
  if (!selectedBreed.value) return;
  imagesLoading.value = true;
  error.value = '';
  try {
    const images = await fetchDog(`/breed/${selectedBreed.value}/images/random/12`);
    breedImages.value = Array.isArray(images) ? images : [];
  } catch (e) {
    error.value = e.message;
  } finally {
    imagesLoading.value = false;
  }
};
const selectBreed = (b) => {
  if (!b) return;
  selectedBreed.value = b.value;
  query.value = b.label;
  open.value = false;
  loadBreedImages();
};
const goTo = (key) => {
  page.value = key;
  error.value = '';
  if (key === 'random' && !randomImage.value) {
    getRandomDog();
  }
  if (key === 'breeds' && !Object.keys(breeds.value).length) {
    loadBreeds();
  }
};
onMounted(async () => {
  try {
    heroImage.value = await fetchDog('/breeds/image/random');
  } catch (e) {
    console.error(e);
  }
});
</script>
<style scoped>
* {
  box-sizing: border-box;
}

.home {
  min-height: 100vh;
  font-family: Arial, sans-serif;
  color: #333;
  background: #fffdf8;
}
.navbar {
  position: sticky;
  top: 0;
  z-index: 20;
  display: flex;
  align-items: center;
  padding: 18px 70px;
  background: #46411e;
  box-shadow: 0 3px 15px rgba(0, 0, 0, 0.15);
}

.logo {
  font-size: 32px;
  margin-right: 45px;
  cursor: pointer;
  transition: 0.3s;
}
.logo:hover {
  transform: scale(1.1);
}
.menu {
  display: flex;
  align-items: center;
  gap: 32px;
}

.menu a {
  position: relative;
  color: white;
  text-decoration: none;
  font-size: 14px;
  font-weight: 500;
  padding: 8px 0;
  transition: 0.3s;
}
.menu a::after {
  content: "";
  position: absolute;
  left: 0;
  bottom: 0;
  width: 0;
  height: 2px;
  background: #ffe2a8;
  transition: 0.3s;
}

.menu a:hover,
.menu a.active {
  color: #ffe2a8;
}

.menu a:hover::after,
.menu a.active::after {
  width: 100%;
}
.hero {
  position: relative;
  min-height: 620px;
  background-color: #6b6130;
  background-size: cover;
  background-position: center;
  display: flex;
  align-items: center;
  overflow: hidden;
}
.overlay {
  position: absolute;
  inset: 0;
  background:
    linear-gradient(
      90deg,
      rgba(45, 40, 15, 0.82),
      rgba(70, 65, 30, 0.48),
      rgba(70, 65, 30, 0.18)
    );
}
.hero-content {
  position: relative;
  z-index: 2;
  width: 500px;
  margin-left: 70px;
  color: white;
}
.hero-content h1 {
  margin: 0 0 18px;
  font-size: 48px;
  line-height: 1.15;
  letter-spacing: -1px;
}

.hero-content p {
  max-width: 470px;
  margin: 0 0 30px;
  color: #f8f4e9;
  font-size: 16px;
  line-height: 1.8;
}
.hero-content button {
  padding: 14px 32px;
  border: none;
  border-radius: 30px;
  background: #ffe3a8;
  color: #6b552f;
  font-size: 15px;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.18);
  transition: 0.3s;
}

.hero-content button:hover {
  background: #fff2d0;
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.22);
}
h2 {
  margin: 0 0 25px;
  color: #51491f;
  font-size: 32px;
}
.random-section {
  min-height: 75vh;
  padding: 90px 20px;
  text-align: center;
  background: #fffdf8;
}

.random-section button {
  padding: 13px 28px;
  border: none;
  border-radius: 25px;
  background: #c9a66b;
  color: white;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}

.random-section button:hover {
  background: #ae8950;
  transform: translateY(-2px);
}

.random-section button:disabled {
  opacity: 0.6;
  cursor: wait;
}
.dog-card {
  width: fit-content;
  margin: 35px auto 0;
  padding: 12px;
  background: white;
  border-radius: 20px;
  box-shadow: 0 10px 35px rgba(70, 60, 30, 0.15);
  transition: 0.3s;
}
.dog-card:hover {
  transform: translateY(-5px);

  box-shadow: 0 15px 40px rgba(70, 60, 30, 0.22);
}
.dog-card img {
  display: block;
  width: 320px;
  height: 320px;
  object-fit: cover;
  border-radius: 14px;
}
.error {
  margin-top: 20px;
  color: #c0392b;
  font-size: 14px;
}
.breeds-section {
  min-height: 75vh;
  padding: 90px 70px;
  text-align: center;
  background: #faf7f0;
}
.search-box {
  position: relative;
  max-width: 400px;
  margin: 0 auto;
  text-align: left;
}
.breed-search {
  width: 100%;
  padding: 14px 18px;
  border: 1px solid #dfcda8;
  border-radius: 30px;
  background: white;
  color: #333;
  font-size: 15px;
  outline: none;
  box-shadow: 0 4px 15px rgba(80, 60, 30, 0.06);
  transition: 0.3s;
}
.breed-search:focus {
  border-color: #c9a66b;
  box-shadow:
    0 0 0 4px rgba(201, 166, 107, 0.15);
}
.breed-search::placeholder {
  color: #aaa;
}
.suggestions {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  right: 0;
  z-index: 10;
  margin: 0;
  padding: 6px 0;
  list-style: none;
  max-height: 260px;
  overflow-y: auto;
  background: white;
  border: 1px solid #eadfc9;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.12);
}
.suggestions li {
  padding: 11px 16px;
  color: #555;
  cursor: pointer;
  transition: 0.2s;
}
.suggestions li:hover {
  background: #faf1dc;
  color: #8d7044;
}
.no-result {
  position: absolute;
  top: calc(100% + 10px);
  left: 5px;
  margin: 0;
  color: #8d7044;
  font-size: 14px;
}
.photo-grid {
  max-width: 1050px;
  margin: 40px auto 0;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 18px;
}
.photo-grid img {
  width: 100%;
  aspect-ratio: 1;
  object-fit: cover;
  border-radius: 14px;
  background: #eee;
  box-shadow: 0 5px 18px rgba(60, 50, 30, 0.1);
  transition: 0.3s;
}
.photo-grid img:hover {
  transform: scale(1.04);
  box-shadow: 0 10px 25px rgba(60, 50, 30, 0.18);
}
.more-btn {
  margin-top: 30px;
  padding: 13px 28px;
  border: none;
  border-radius: 25px;
  background: #c9a66b;
  color: white;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}
.more-btn:hover {
  background: #ae8950;
  transform: translateY(-2px);
}
.more-btn:disabled {
  opacity: 0.6;
  cursor: wait;
}
.about-section {
  min-height: 75vh;
  padding: 90px 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  background: #fffdf8;
}
.about-section p {
  max-width: 600px;
  margin: 0;
  color: #666;
  font-size: 15px;
  line-height: 1.8;
}
.about-section a {
  color: #8d7044;
  font-weight: bold;
  text-decoration: none;
}
.about-section a:hover {
  text-decoration: underline;
}
@media (max-width: 900px) {
  .navbar {
    padding: 18px 35px;
  }
  .hero-content {
    margin-left: 45px;
  }
  .breeds-section {
    padding: 70px 35px;
  }
  .photo-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
@media (max-width: 768px) {
  .navbar {
    padding: 15px 20px;
    flex-wrap: wrap;
  }
  .logo {
    margin-right: 20px;
  }
  .menu {
    gap: 15px;
    flex-wrap: wrap;
  }
  .menu a {
    font-size: 12px;
  }
  .hero {
    min-height: 550px;
  }
  .hero-content {
    width: auto;
    margin: 0 25px;
  }
  .hero-content h1 {
    font-size: 36px;
  }
  .hero-content p {
    font-size: 14px;
  }
  .random-section,
  .about-section {
    padding: 60px 20px;
  }
  h2 {
    font-size: 28px;
  }
  .dog-card img {
    width: 280px;
    height: 280px;
  }
  .breeds-section {
    padding: 60px 20px;
  }
  .photo-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 12px;
  }
}
@media (max-width: 480px) {
  .navbar {
    justify-content: center;
  }
  .logo {
    width: 100%;
    margin: 0 0 8px;
    text-align: center;
  }
  .menu {
    justify-content: center;
    gap: 12px;
  }
  .menu a {
    font-size: 11px;
  }
  .hero-content h1 {
    font-size: 30px;
  }
  .hero-content button {
    padding: 12px 25px;
  }
  .dog-card img {
    width: 250px;
    height: 250px;
  }
}
</style>
