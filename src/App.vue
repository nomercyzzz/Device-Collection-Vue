<template>
    <Transition appear name="page-fade">
        <main class="min-h-screen bg-black text-white">

            <div class="mx-auto flex min-h-screen w-full max-w-7xl flex-col gap-8 px-4 py-8 sm:px-6 lg:px-8">

                <header class="rounded-3xl bg-white/5 p-6 ">

                    <div class="flex flex-col gap-4 lg:flex-row lg:items-end lg:justify-between">

                        <div class="max-w-3xl">
                            <h1 class="text-3xl font-semibold tracking-tight sm:text-4xl ">Device Collection</h1>
                            <p class="mt-3 max-w-2xl text-sm leading-6 text-white/70 sm:text-base">
                                Добавляй карточки устройств, фильтруй по категориям, сортируй список и находи нужные девайсы через поиск.
                            </p>
                        </div>

                        <div class="grid grid-cols-2 gap-3 sm:grid-cols-3">

                        <div class="rounded-2xl border border-white/10 bg-black/60 px-4 py-3">
                            <p class="text-xs uppercase tracking-[0.24em] text-white/45">Карточек</p>
                            <p class="mt-2 text-xl font-semibold color-blue-600">{{ items.length }}</p>
                        </div>

                        <div class="rounded-2xl border border-white/10  bg-black/60 px-4 py-3">
                            <p class="text-xs uppercase tracking-[0.24em] text-white/45">Найдено</p>
                            <p class="mt-2 text-xl font-semibold">{{ filteredItems.length }}</p>
                        </div>

                        <div class="col-span-2 rounded-2xl border border-white/10 bg-black/60 px-4 py-3 sm:col-span-1">
                            <p class="text-xs uppercase tracking-[0.24em] text-white/45">Фильтр</p>
                            <p class="mt-2 font-medium">{{ selectedCategory }}</p>
                        </div>

                        </div>
                    </div>
                </header>

                <section class="grid gap-6 lg:grid-cols-[390px_minmax(0,1fr)]">

                    <aside class="rounded-3xl bg-white/5 p-5 backdrop-blur">

                        <div class="mb-5">
                            <h2 class="text-xl font-semibold">Добавить устройство</h2>
                            <p class="mt-2 text-sm leading-6 text-white/65">Укажи название, категорию, описание и загрузи фото. После добавления карточка появится с плавной анимацией.</p>
                        </div>

                        <form class="space-y-4" @submit.prevent="addItem">

                            <label class="block space-y-2">
                                <span class="text-sm text-white/75">Название</span>
                                <input 
                                    v-model.trim="form.title" 
                                    type="text"
                                    placeholder="Например, Keychron K8 Pro" 
                                    class="w-full rounded-2xl border border-white/10 bg-black/60 px-4 py-3 text-white outline-none transition focus:border-blue-600/70" 
                                    />
                            </label>

                            <label class="block space-y-2">
                                <span class="text-sm text-white/75">Категория</span>

                                <select 
                                    v-model="form.category" 
                                    class="w-full rounded-2xl bg-black/60 px-4 py-3 text-white outline-none transition focus:border-blue-600/70"
                                    >

                                <option v-for="category in categories" :key="category" :value="category">{{ category }}</option>
                                </select>
                            </label>

                            <label class="block space-y-2">
                                <span class="text-sm text-white/75">Описание</span>

                                <textarea 
                                    v-model.trim="form.description" 
                                    rows="4" 
                                    placeholder="Коротко опиши устройство" 
                                    class="w-full resize-none rounded-2xl border border-white/10 bg-black/60 px-4 py-3 text-white outline-none transition focus:border-blue-600/70" />
                            </label>

                            <label class="block space-y-2">
                                <span class="text-sm text-white/75">Фото</span>

                                <input 
                                    accept="image/png,image/jpeg,image/webp,image/jpg" 
                                    ref="fileInput" 
                                    type="file" 
                                    class="block w-full rounded-2xl border border-dashed border-blue-700/35 bg-black/60 px-4 py-3 text-sm text-white/75 file:mr-4 file:rounded-xl file:border-0 file:bg-blue-700 file:px-4 file:py-2 file:font-medium file:text-white hover:file:bg-blue-600" 
                                    @change="handleImageUpload"
                                />
                            </label>

                            <div v-if="imagePreview" class="overflow-hidden rounded-3xl border border-blue-700/20 bg-black/60">
                                
                                <div class="flex items-center justify-between gap-3 border-b border-white/10 px-4 py-3">
                                    <div>
                                        <p class="text-sm font-medium text-white">Предпросмотр</p>
                                        <p class="text-xs text-white/45">{{ selectedImageName || "Выбранное изображение" }}</p>
                                    </div>

                                <button 
                                    type="button" 
                                    class="rounded-2xl border border-white/10 px-3 py-2 text-xs text-white transition hover:border-blue-600/50 hover:text-blue-300" 
                                    @click="clearImage">Убрать фото
                                </button>

                                </div>
                                <img :src="imagePreview" alt="Предпросмотр" class="h-44 w-full object-cover" />
                            </div>

                            <p v-if="errorMessage" class="rounded-2xl border border-red-500/20 bg-red-500/10 px-4 py-3 text-sm text-red-300">{{ errorMessage }}</p>
                            <button type="submit" class="w-full rounded-2xl bg-blue-700 px-4 py-3 font-medium text-white shadow-[0_12px_30px_rgba(29,78,216,0.35)] transition hover:bg-blue-600">Добавить карточку</button>
                        </form>
                    </aside>

                    <section class="space-y-5">
                        <div class="grid gap-4 rounded-3xl bg-white/5 p-5 md:grid-cols-[1fr_max-content]">

                            <label class="block space-y-2 md:col-span-3 lg:col-span-1">
                                <span class="text-sm text-white/75">Поиск</span>

                                <input 
                                    v-model.trim="search" 
                                    type="text" 
                                    placeholder="Искать по названию, описанию или категории" 
                                    class="w-full rounded-2xl border border-white/10 bg-black/60 px-4 py-3 text-white outline-none transition focus:border-blue-600/70"
                                />
                            </label>

                            <label class="block space-y-2">

                                <span class="text-sm text-white/75">Сортировка</span>

                                <select v-model="sortMode" class="w-full rounded-2xl border border-white/10 bg-black/60 px-4 py-3 text-white outline-none transition focus:border-blue-600/70">
                                    <option value="newest">Сначала новые</option>
                                    <option value="oldest">Сначала старые</option>
                                    <option value="title-asc">По названию А-Я</option>
                                    <option value="title-desc">По названию Я-А</option>
                                    <option value="category">По категории</option>
                                </select>


                            </label>

                        </div>

                        <div class="flex flex-wrap gap-3">

                            <button 
                                type="button" 
                                class="rounded-2xl border px-4 py-2 text-sm transition" 
                                :class="selectedCategory === 'Все категории' ? 'border-blue-600/50 bg-blue-700/20 text-blue-300' : 'border-white/10 bg-white/5 text-white/75 hover:border-blue-600/35 hover:text-blue-300'" 
                                @click="setQuickCategory('Все категории')">Все
                            </button>

                            <button 
                                v-for="category in categories" 
                                :key="category" 
                                type="button"
                                class="rounded-2xl border px-4 py-2 text-sm transition" 
                                :class="selectedCategory === category ? 'border-blue-600/50 bg-blue-700' : 'border-white/10 bg-white/5 hover:bg-blue-700/10 hover:border-blue-600/50'" 
                                @click="setQuickCategory(category)">{{ category }}
                            </button>

                        </div>

                        <TransitionGroup name="card-list" tag="div" class="grid gap-4 sm:grid-cols-2 xl:grid-cols-3">

                            <article v-for="item in filteredItems" :key="item.id" class="card-item overflow-hidden rounded-3xl border border-white/10 bg-white/5 shadow-[0_0px_0px_rgba(0,0,0,0.35)] hover:border-blue-700/20">

                                <div class="relative aspect-[16/10] overflow-hidden bg-white/5">
                                    <img :src="item.image" :alt="item.title" class="h-full w-full object-cover" />
                                    <span class="absolute left-4 top-4 rounded-full bg-blue-600 px-3 py-1 text-xs">{{ item.category }}</span>
                                </div>

                                <div class="flex h-[220px] flex-col p-5">
                                    <div class="mb-4">
                                        <h3 class="text-xl font-semibold leading-tight">{{ item.title }}</h3>
                                        <p class="mt-3 line-clamp-4 text-sm leading-6 text-white/70">{{ item.description }}</p>
                                    </div>
                            
                                    <div class="mt-auto flex items-center justify-between gap-3">
                                        <span class="text-xs uppercase tracking-[0.22em] text-white/35">{{ formatDate(item.createdAt) }}</span>
                                        <button type="button" class="rounded-2xl border border-white/10 px-4 py-2 text-sm text-white transition hover:border-blue-600/50 hover:text-blue-300" @click="removeItem(item.id)">Удалить</button>
                                    </div>

                                </div>
                            </article>
                        </TransitionGroup>

                        <div v-if="!filteredItems.length" class="rounded-3xl border border-dashed border-blue-700/20 bg-white/5 px-6 py-12 text-center text-white/55">Ничего не найдено. Попробуй изменить фильтр, поиск или добавить новую карточку.</div>
                    </section>
                </section>
            </div>
        </main>
    </Transition>
</template>

<script setup>
import { computed, ref, watch } from "vue";

const maxImageSizeInMb = 2;

const categories = ["Клавиатура", "Мышь", "Монитор", "Наушники", "Ноутбук", "Смартфон", "Планшет", "Аксессуар"];

const demoItems = [
    { 
        id: 1,
        title: "Keychron K8 Pro", 
        category: "Клавиатура", 
        description: "Беспроводная механическая клавиатура с хорошим ходом клавиш для повседневной работы.", 
        image: "https://images.unsplash.com/photo-1511467687858-23d96c32e4ae?auto=format&fit=crop&w=1200&q=80", 
        createdAt: new Date("2026-03-14T10:00:00").toISOString() 
    },
    { 
        id: 2, 
        title: "Logitech MX Master 3S", 
        category: "Мышь", 
        description: "Удобная беспроводная мышь для работы, монтажа и длительных сессий за компьютером.", 
        image: "https://images.unsplash.com/photo-1527814050087-3793815479db?auto=format&fit=crop&w=1200&q=80", 
        createdAt: new Date("2026-03-15T12:30:00").toISOString() 
    },
    { 
        id: 3, 
        title: "Dell UltraSharp 27", 
        category: "Монитор", 
        description: "Тонкие рамки, аккуратный дизайн и комфортный формат для учебы, кода и дизайна.", 
        image: "https://images.unsplash.com/photo-1527443224154-c4a3942d3acf?auto=format&fit=crop&w=1200&q=80", 
        createdAt: new Date("2026-03-16T16:15:00").toISOString() 
    },
    { 
        id: 4, 
        title: "Sony WH-1000XM5", 
        category: "Наушники", 
        description: "Полноразмерные наушники с шумоподавлением для работы, поездок и концентрации.", 
        image: "https://images.unsplash.com/photo-1505740420928-5e560c06d30e?auto=format&fit=crop&w=1200&q=80", 
        createdAt: new Date("2026-03-17T09:20:00").toISOString() 
    },
    { 
        id: 6, 
        title: "iPad Air", 
        category: "Планшет", 
        description: "Лёгкий планшет для конспектов, просмотра материалов, заметок и повседневных задач.", 
        image: "https://images.unsplash.com/photo-1544244015-0df4b3ffc6b0?auto=format&fit=crop&w=1200&q=80", 
        createdAt: new Date("2026-03-19T11:10:00").toISOString() 
    },
    { 
        id: 7, 
        title: "Nothing Phone", 
        category: "Смартфон", 
        description: "Смартфон с необычным дизайном корпуса и чистым визуальным стилем интерфейса.", 
        image: "https://images.unsplash.com/photo-1511707171634-5f897ff02aa9?auto=format&fit=crop&w=1200&q=80", 
        createdAt: new Date("2026-03-20T15:05:00").toISOString() 
    }
];

const storageKey = "device-collection-items";
const savedItems = safelyParseLocalStorage(storageKey);
const items = ref(savedItems?.length ? savedItems : demoItems);

const form = ref({ title: "", category: categories[0], description: "", image: "" });

const search = ref("");

const selectedCategory = ref("Все категории");

const sortMode = ref("newest");

const errorMessage = ref("");

const imagePreview = ref("");


const selectedImageName = ref("");

const fileInput = ref(null);

const filteredItems = computed(() => {
    const normalizedSearch = search.value.toLowerCase().trim();

    const filtered = items.value.filter((item) => {
        const matchByCategory = selectedCategory.value === "Все категории" || item.category === selectedCategory.value;

        if (!matchByCategory) return false;
        if (!normalizedSearch) return true;

        return [item.title, item.description, item.category].some((field) => field.toLowerCase().includes(normalizedSearch));
    });

    const sorted = [...filtered];
    sorted.sort((a, b) => {
        switch (sortMode.value) {

        case "oldest": return new Date(a.createdAt) - new Date(b.createdAt);

        case "title-asc": return a.title.localeCompare(b.title, "ru");

        case "title-desc": return b.title.localeCompare(a.title, "ru");

        case "category": return a.category.localeCompare(b.category, "ru");

        case "newest":

        default: return new Date(b.createdAt) - new Date(a.createdAt);
        }
    });

    return sorted;
});

watch(items, (value) => { localStorage.setItem(storageKey, JSON.stringify(value)); }, { deep: true });

function addItem() {
    errorMessage.value = "";
    if (!form.value.title || !form.value.description || !form.value.image) {
        errorMessage.value = "Заполни все поля и загрузи фото, чтобы карточка добавилась корректно.";
        return;
    }

    items.value.unshift({ id: Date.now(), title: form.value.title, category: form.value.category, description: form.value.description, image: form.value.image, createdAt: new Date().toISOString() });

    form.value = { title: "", category: categories[0], description: "", image: "" };
    clearImage();
}

function removeItem(id) {
    items.value = items.value.filter((item) => item.id !== id);
}

function setQuickCategory(category) {
    selectedCategory.value = category;
}

function handleImageUpload(event) {
    errorMessage.value = "";
    const file = event.target.files?.[0];

    if (!file) {
        clearImage(event.target);
        return;
    }

    const isValidType = ["image/png", "image/jpeg", "image/jpg", "image/webp"].includes(file.type);
  const isValidSize = file.size <= maxImageSizeInMb * 1024 * 1024;

    if (!isValidType) {
        errorMessage.value = "Загрузи изображение в формате PNG, JPG или WEBP.";

        clearImage(event.target);
        return;
    }

    if (!isValidSize) {

        errorMessage.value = `Размер изображения не должен превышать ${maxImageSizeInMb} МБ.`;
        clearImage(event.target);
        return;
    }

    const reader = new FileReader();
    reader.onload = () => {
        form.value.image = String(reader.result);

        imagePreview.value = String(reader.result);

        selectedImageName.value = file.name;
    };

    reader.readAsDataURL(file);
}

function clearImage(inputElement = null) {
    form.value.image = "";
    imagePreview.value = "";
    selectedImageName.value = "";

    if (inputElement) inputElement.value = "";
    if (fileInput.value) fileInput.value.value = "";
}

function formatDate(date) {
    return new Intl.DateTimeFormat("ru-RU", { day: "2-digit", month: "2-digit", year: "numeric" }).format(new Date(date));
}

function safelyParseLocalStorage(key) {
    try {
        const value = localStorage.getItem(key);
        return value ? JSON.parse(value) : null;
    } catch {
        return null;
    }
}
</script>

<style scoped>
.page-fade-enter-active, .page-fade-leave-active {
    transition: opacity 0.4s ease-in-out, transform 0.4s ease-in-out;
}

.page-fade-enter-from, .page-fade-leave-to {
    opacity: 0;
    transform: translateY(14px);
}

.card-list-enter-active, .card-list-leave-active, .card-list-move {
    transition: opacity 0.4s ease-in-out, transform 0.4s ease-in-out;
}

.card-list-enter-from, .card-list-leave-to {
    opacity: 0;
    transform: translateY(18px) scale(0.98);
}

.card-item {
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card-item:hover {
    transform: translateY(-4px);
    box-shadow: 0 4px 8px rgba(29, 78, 216, 0.2);
}
</style>
