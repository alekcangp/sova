<template>
  <div class="image-viewer">

    <div class="image-style-select" style="display: flex; align-items: center; justify-content: space-between;">
      <div style="display: flex; align-items: center; flex: 1 1 auto; min-width: 0;">

      <label for="art-style-combobox">{{ t.artStyle }}:&nbsp; </label>
      <div class="combobox-wrapper">
        <input
          id="art-style-combobox"
          ref="comboboxInput"
          :readonly="!isCustomSelected"
          :value="displayedValue"
          :key="currentLanguage + selectedStyle"
          @focus="openDropdown"
          @click="openDropdown"
          @input="onInput"
          @keydown.down.prevent="onArrowDown"
          @keydown.up.prevent="onArrowUp"
          @keydown.enter.prevent="onEnter"
          @keydown.esc.prevent="closeDropdown"
          @blur="onBlur"
          class="combobox-input"
          autocomplete="off"
          :placeholder="t.chooseOrEnterStyle"
        />
        <ul :class="['combobox-dropdown', filteredStyles.length > 8 ? 'too-many' : '']" v-if="dropdownOpen">
          <li
            v-for="(style, idx) in filteredStyles"
            :key="style"
            :class="{ selected: idx === highlightedIndex }"
            @mousedown.prevent="selectStyle(style)"
          >
            {{ getLocalizedStyleName(style) }}
          </li>
          <li
            v-if="!filteredStyles.includes('custom')"
            :class="{ selected: highlightedIndex === filteredStyles.length }"
            @mousedown.prevent="selectStyle('custom')"
          >
            {{ t.artStyles.custom }}
          </li>
        </ul>
      </div>
      <div class="external-links">
        <button
          @click="switchLanguage"
          class="lang-btn"
          :title="currentLanguage === 'ru' ? 'Переключить на английский' : 'Switch to Russian'"
        >
          {{ currentLanguage == 'ru' ? 'EN' : 'RU' }}
        </button>
        <a
          href="https://deepwiki.com/alekcangp/sova"
          class="docs-link"
          target="_blank"
          rel="noopener noreferrer"
          :title="currentLanguage === 'ru' ? 'Посмотреть документацию' : 'View Documentation'"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round">
            <rect x="4" y="3" width="16" height="18" rx="2"/>
            <path d="M8 6h8M8 10h8M8 14h6"/>
          </svg>
        </a>
        <a
          href="https://github.com/alekcangp/sova"
          class="github-link"
          target="_blank"
          rel="noopener noreferrer"
          :title="currentLanguage === 'ru' ? 'Посмотреть на GitHub' : 'View on GitHub'"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
            <path d="M12 2C6.477 2 2 6.484 2 12.021c0 4.428 2.865 8.184 6.839 9.504.5.09.682-.217.682-.482 0-.237-.009-.868-.014-1.703-2.782.605-3.369-1.342-3.369-1.342-.454-1.156-1.11-1.464-1.11-1.464-.908-.62.069-.608.069-.608 1.003.07 1.531 1.03 1.531 1.03.892 1.53 2.341 1.088 2.91.832.092-.647.35-1.088.636-1.338-2.22-.253-4.555-1.113-4.555-4.951 0-1.093.39-1.987 1.029-2.687-.103-.253-.446-1.272.098-2.65 0 0 .84-.27 2.75 1.025A9.564 9.564 0 0 1 12 6.844c.85.004 1.705.115 2.504.337 1.908-1.296 2.747-1.025 2.747-1.025.546 1.378.202 2.397.1 2.65.64.7 1.028 1.594 1.028 2.687 0 3.847-2.338 4.695-4.566 4.944.36.31.68.92.68 1.855 0 1.338-.012 2.419-.012 2.749 0 .268.18.577.688.48C19.138 20.2 22 16.447 22 12.021 22 6.484 17.523 2 12 2z"/>
          </svg>
        </a>
      </div>
      </div>
    </div>
    <div class="image-container">
      <div v-if="props.isGenerating" class="loading-state">
        <div class="ai-spinner">
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
          <div class="spinner-ring"></div>
        </div>
        <h4>{{ t.generatingAiIllustration }}</h4>

      </div>
      <div v-else-if="props.imageUrl && props.imageUrl.length > 0" class="image-display">
        <div class="image-hover-container">
          <img :src="props.imageUrl" alt="AI Generated Illustration" />
          <div class="image-hover-buttons">
            <button @click="downloadImage" :disabled="!props.imageUrl" class="action-btn" :title="t.downloadImage">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
                <polyline points="7,10 12,15 17,10"/>
                <line x1="12" y1="15" x2="12" y2="3"/>
              </svg>
              {{ t.downloadImage }}
            </button>
          </div>
        </div>
        <div class="image-info">
          <div class="summary-section">
            <h4></h4>
            <p></p>
          </div>
        </div>
      </div>
      <div v-else-if="props.wasCanceled" class="empty-state">
        <div class="empty-icon">
          <svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
            <rect x="3" y="3" width="18" height="18" rx="2" ry="2"/>
            <circle cx="9" cy="9" r="2"/>
            <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
          </svg>
        </div>
        <h4>{{ t.illustrationCanceled }}</h4>
        <p>{{ t.canceledPrevious }}</p>
      </div>
      <div v-else class="empty-state">
        <div class="empty-icon">
          <svg width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
            <rect x="3" y="3" width="18" height="18" rx="2" ry="2"/>
            <circle cx="9" cy="9" r="2"/>
            <path d="m21 15-3.086-3.086a2 2 0 0 0-2.828 0L6 21"/>
          </svg>
        </div>
        <h4>{{ t.noIllustrationYet }}</h4>
        <p>{{ t.turnPageOrSelect }}</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, computed, nextTick } from 'vue';
import { useI18n } from '../composables/useI18n';

interface Props {
  imageUrl?: string;
  summary?: string;
  isGenerating?: boolean;
  selectedText?: string;
  bookTitle?: string;
  selectedArtStyle?: string;
  wasCanceled?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  imageUrl: '',
  summary: '',
  isGenerating: false,
  selectedText: '',
  bookTitle: 'Unknown Book',
  selectedArtStyle: 'futuristic',
  wasCanceled: false
});

const emit = defineEmits<{
  regenerate: [style: string];
  download: [url: string];
}>();

const { t, currentLanguage, switchLanguage, getEnglishStyleName } = useI18n();

const ArtStyles = [
  'futuristic',
  'fantasy',
  'steampunk',
  'cyberpunk',
  'watercolor',
  'oil_painting',
  'anime',
  'sketch',
  'minimalism',
  'realism',
  'impressionism',
  'surrealism',
  'cartoon',
  'pixel_art',
  'noir',
  'gothic',
  'baroque',
  'pop_art',
  'abstract',
  'photorealistic',
  'render_3d',
  'ink',
  'classic',
  'modern',
  
];

const comboboxInput = ref<HTMLInputElement | null>(null);
const dropdownOpen = ref(false);
const highlightedIndex = ref(-1);
const filterText = ref('');
const isCustomSelected = computed(() => selectedStyle.value === 'custom');

const selectedStyle = ref(
  ArtStyles.includes(props.selectedArtStyle as any)
    ? props.selectedArtStyle
    : 'custom'
);
const customStyle = ref(
  ArtStyles.includes(props.selectedArtStyle as any)
    ? ''
    : props.selectedArtStyle || ''
);

const displayedValue = computed(() => {
 // console.log(selectedStyle.value)
  if (isCustomSelected.value) {
    return customStyle.value;
  }
  return getLocalizedStyleName(selectedStyle.value);
});

const getLocalizedStyleName = (styleKey: string) => {
  return t.value.artStyles[styleKey as keyof typeof t.value.artStyles] || styleKey;
};

const filteredStyles = computed(() => {
  if (!dropdownOpen.value || !filterText.value) return [...ArtStyles];
  return ArtStyles.filter(styleKey =>
    getLocalizedStyleName(styleKey).toLowerCase().includes(filterText.value.toLowerCase()) ||
    getEnglishStyleName(styleKey).toLowerCase().includes(filterText.value.toLowerCase())
  );
});

function openDropdown() {
  dropdownOpen.value = true;
  filterText.value = '';
  highlightedIndex.value = -1;
}

function closeDropdown() {
  dropdownOpen.value = false;
  filterText.value = '';
  highlightedIndex.value = -1;
}

async function selectStyle(style: string) {
  
  if (style === 'custom') {
    selectedStyle.value = 'custom';
    const saved = localStorage.getItem('epub-custom-art-style');
    customStyle.value = saved || '';
    nextTick(() => {
      if (comboboxInput.value) comboboxInput.value.focus();
    });
    if ((saved && saved.trim()) || (customStyle.value && customStyle.value.trim())) {
      emit('regenerate', customStyle.value);
      emit('update:selectedArtStyle', customStyle.value);
    }
  } else {

    selectedStyle.value = style;
   // console.log(style)
    customStyle.value = '';
    emit('regenerate', style);
    emit('update:selectedArtStyle', style);
  }
  closeDropdown();
}

function onInput(e: Event) {
  const val = (e.target as HTMLInputElement).value;
  if (isCustomSelected.value) {
    customStyle.value = val;
    emit('update:selectedArtStyle', val);
    // Close dropdown when typing custom style
    closeDropdown();
  } else {
    // Only show dropdown if there's text to filter with
    if (val.trim()) {
      if (!dropdownOpen.value) {
        openDropdown();
      }
      // Find the matching style key from the localized or English name
      const matchedStyle = ArtStyles.find(style => 
        getLocalizedStyleName(style).toLowerCase().includes(val.toLowerCase()) ||
        getEnglishStyleName(style).toLowerCase().includes(val.toLowerCase())
      );
      
      if (matchedStyle) {
        selectedStyle.value = matchedStyle;
        emit('update:selectedArtStyle', matchedStyle);
      }
    } else {
      closeDropdown();
    }
  }
  filterText.value = val;
  highlightedIndex.value = -1;
}

function onArrowDown() {
  if (!dropdownOpen.value) openDropdown();
  if (highlightedIndex.value < filteredStyles.value.length - 1) {
    highlightedIndex.value++;
  } else if (isCustomSelected.value && highlightedIndex.value < filteredStyles.value.length) {
    highlightedIndex.value++;
  }
}

function onArrowUp() {
  if (highlightedIndex.value > 0) {
    highlightedIndex.value--;
  }
}

function onEnter() {
  if (dropdownOpen.value) {
    if (highlightedIndex.value >= 0 && highlightedIndex.value < filteredStyles.value.length) {
      selectStyle(filteredStyles.value[highlightedIndex.value]);
    } else if (highlightedIndex.value === filteredStyles.value.length) {
      selectStyle('custom');
    } else if (isCustomSelected.value && customStyle.value) {
      emit('regenerate', customStyle.value);
      emit('update:selectedArtStyle', customStyle.value);
      localStorage.setItem('epub-custom-art-style', customStyle.value);
      closeDropdown();
    }
  } else if (isCustomSelected.value && customStyle.value) {
    emit('regenerate',  customStyle.value);
    emit('update:selectedArtStyle', customStyle.value);
    localStorage.setItem('epub-custom-art-style', customStyle.value);
  }
}

watch([isCustomSelected, customStyle], ([isCustom, val]) => {
  if (isCustom && val) {
    localStorage.setItem('epub-custom-art-style', val);
  }
});

function onBlur() {
  setTimeout(() => closeDropdown(), 100);
}

watch(() => props.selectedArtStyle, (val) => {
  if (val && ArtStyles.includes(val as any)) {
    selectedStyle.value = val;
    customStyle.value = '';
  } else if (val && val !== 'custom') {
    selectedStyle.value = 'custom';
    customStyle.value = val;
  }
});

// Watch for language changes to ensure the displayed value updates
watch(currentLanguage, () => {
  // The displayedValue computed property will automatically update
  // when the language changes, causing Vue to re-render the input
});

function downloadImage() {
  if (props.imageUrl) {
    emit('download', props.imageUrl);
  }
}

onMounted(() => {
  if (props.selectedArtStyle && !ArtStyles.includes(props.selectedArtStyle as any)) {
    selectedStyle.value = 'custom';
    customStyle.value = props.selectedArtStyle;
  } else if (selectedStyle.value === 'custom') {
    const saved = localStorage.getItem('epub-custom-art-style');
    if (saved) customStyle.value = saved;
  }
});
</script>

<style scoped>
.image-viewer {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: var(--color-card);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  overflow: hidden;
  width: 100%;
  max-width: 100vw;
  margin: 0;
  padding: 28px 32px 24px 32px;
  box-sizing: border-box;
  overflow-x: auto;
}

.image-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 0 8px;
  width: 100%;
  margin: 0;
  box-sizing: border-box;
  overflow-x: auto;
}

.loading-state {
  text-align: center;
  padding: 48px 24px;
}

.ai-spinner {
  position: relative;
  width: 80px;
  height: 80px;
  margin: 0 auto 24px;
}

.spinner-ring {
  position: absolute;
  width: 100%;
  height: 100%;
  border: 3px solid transparent;
  border-radius: 50%;
  animation: spin 2s linear infinite;
}

.spinner-ring:nth-child(1) {
  border-top-color: var(--color-primary);
  animation-delay: 0s;
}

.spinner-ring:nth-child(2) {
  border-right-color: var(--color-secondary);
  animation-delay: 0.3s;
}

.spinner-ring:nth-child(3) {
  border-bottom-color: var(--color-accent);
  animation-delay: 0.6s;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-state h4 {
  font-size: 18px;
  font-weight: 600;
  margin: 0 0 8px 0;
  color: var(--color-text);
}

.loading-state p {
  font-size: 14px;
  color: var(--color-muted);
  margin: 0 0 20px 0;
}

.progress-bar {
  width: 100%;
  height: 4px;
  background: var(--color-border);
  border-radius: 2px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, var(--color-primary), var(--color-secondary));
  border-radius: 2px;
  transition: width 0.3s ease;
}

.image-display {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 320px;
  width: 100%;
  max-width: 100%;
}

.image-hover-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  max-width: 100%;
}

.image-hover-container::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.55);
  border-radius: 12px;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s;
  z-index: 1;
}

.image-hover-container:hover::before,
.image-hover-container:focus-within::before {
  opacity: 0.22;
  background: rgba(0,0,0,0.72);
}

.image-hover-buttons {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  gap: 14px;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s, transform 0.18s;
  width: auto;
  background: none;
  padding: 0;
  border-radius: 0;
  box-shadow: none;
  align-items: center;
  z-index: 10;
}

.image-hover-container:hover .image-hover-buttons,
.image-hover-container:focus-within .image-hover-buttons {
  opacity: 1;
  pointer-events: auto;
  transform: translate(-50%, -50%) scale(1.04);
}

.image-hover-container img {
  display: block;
  width: 100%;
  max-width: 100%;
  border-radius: 12px;
  box-shadow: 0 2px 16px rgba(0,0,0,0.12);
  object-fit: contain;
  z-index: 0;
}

.image-hover-buttons .action-btn {
  font-size: 1rem;
  padding: 12px 0;
  border-radius: 8px;
  background: #fff;
  color: #007AFF;
  border: none;
  box-shadow: 0 2px 8px rgba(0,0,0,0.13);
  transition: background 0.18s, color 0.18s, box-shadow 0.18s, transform 0.12s, opacity 0.18s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-weight: 500;
  cursor: pointer;
  width: 160px;
  min-width: 160px;
  min-height: 48px;
  opacity: 0.7;
}

.image-hover-buttons .action-btn:hover,
.image-hover-buttons .action-btn:focus {
  background: #e6eeff;
  color: #0056CC;
  box-shadow: 0 4px 16px rgba(0,122,255,0.10);
  transform: translateY(-2px) scale(1.05);
  opacity: 1;
}

.image-hover-buttons .action-btn:active {
  background: #dbeafe;
  color: #003e99;
  transform: scale(0.97);
}

.image-hover-buttons .action-btn:disabled {
  color: #C7C7CC;
  cursor: not-allowed;
  background: #f0f0f0;
  box-shadow: none;
}

.image-info {
  margin-top: 20px;
}

.summary-section h4 {
  font-size: 16px;
  font-weight: 600;
  margin: 0 0 8px 0;
  color: var(--color-text);
}

.summary-section p {
  font-size: 14px;
  color: var(--color-muted);
  line-height: 1.5;
  margin: 0;
}

.empty-state {
  text-align: center;
  padding: 48px 24px;
}

.empty-icon {
  color: #C7C7CC;
  margin-bottom: 16px;
}

.empty-state h4 {
  font-size: 18px;
  font-weight: 600;
  margin: 0 0 8px 0;
  color: #86868B;
}

.empty-state p {
  font-size: 14px;
  color: #C7C7CC;
  margin: 0;
  line-height: 1.4;
}

.combobox-wrapper {
  position: relative;
  display: inline-block;
  min-width: 240px;
  margin-top: 8px
}

.combobox-input {
  font-size: 1rem;
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid #ccc;
  outline: none;
  transition: border 0.2s;
  width: 220px;
  background: rgb(239,226,204);
  color: #1D1D1F;
}
.combobox-input:focus {
  border: 1.5px solid #007AFF;
}
.combobox-dropdown {
  position: absolute;
  left: 0;
  right: 0;
  z-index: 10;
  background: rgb(239,226,204);
  border: 1px solid #ccc;
  border-radius: 0 0 8px 8px;
  box-shadow: 0 4px 16px rgba(0,0,0,0.08);
  margin-top: 2px;
  list-style: none;
  padding: 0;
  height: auto;
  overflow-y: visible;
}
/* If there are more than 8 items, limit height and enable scroll */
.combobox-dropdown.too-many {
  max-height: 320px;
  overflow-y: auto;
}
.combobox-dropdown li {
  padding: 10px 14px;
  cursor: pointer;
  font-size: 1rem;
  color: #1D1D1F;
  background: rgb(239,226,204);
  transition: background 0.15s;
}
.combobox-dropdown li.selected,
.combobox-dropdown li:hover {
  background: #e6eeff;
  color: #0056CC;
}
.external-links {
  display: flex;
  align-items: right;
  gap: 8px;
  margin-left: auto;
}
.docs-link, .github-link {
  position: static;
  bottom: unset;
  right: unset;
  z-index: 1;
  opacity: 0.82;
  transition: opacity 0.18s, transform 0.18s;
  border-radius: 50%;
  background: rgba(255,255,255,0.3);
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  padding: 4px;
  display: flex;
  align-items: center;
}

.lang-btn {
  font-size: 12px;
  font-weight: 600;
  padding: 4px 6px;
  border-radius: 50%;
  border:0;
  position: static;
  bottom: unset;
  right: unset;
  z-index: 1;
  opacity: 0.82;
  transition: opacity 0.18s, transform 0.18s;
  border-radius: 50%;
  background: rgba(255,255,255,0.3);
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  display: flex;
  align-items: center;
  cursor: pointer;
}

  
.docs-link {
  right: 62px;
}
.github-link {
  right: 24px;
}

 
.lang-btn:hover,
.docs-link:hover, .github-link:hover {
  opacity: 1;
  background:  rgba(255,255,255,0.5);
  transform: scale(1.02) translateY(-1px);
  box-shadow: 0 4px 16px rgba(0,0,0,0.15);
  border:0
}
.image-viewer {
  position: relative;
}

</style>