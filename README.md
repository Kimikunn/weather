## 流程
### 一、新建项目
```terminal
npm init vue@latest
```
将项目中不需要的内容都清除:
1. 删除`assets`和`components`中的所有文件
2. `router`中`index.js`只需要根路径
3. 只需要`HomeView.vue`
4. 在`App.vue`中使用vue3模板片段重构
```js
<vbase-3-setup />

<template>
  <div>
    <RouterView />
  </div>
</template>

<script setup>
import { RouterView } from 'vue-router';
</script>

<style lang="scss" scoped>

</style>

```
### 二、配置`tailwind`
1. 安装依赖
```terminal
npm install -D tailwindcss postcss autoprefixer
```
2. 初始化`tailwind`
```
npx tailwindcss init -p
```
运行后会生成配置文件，在配置文件中做如下配置，使这些目录下的文件能够使用`tailwind`模板：
```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./index.html","./src/**/*.{vue,js,ts,jsx,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
}
```
之后，我们需要一个`tailwind css`的入口文件，在`assets`目录下，新建`tailwind.css`。
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
最后，在`main.js`中引入，即可使用此模板。

3. 使用`tailwind`
在`tailwind`中，我们可以自定义样式，例如为这个app构建一些自定义的全局颜色。
```js
  theme: {
    extend: {
      colors: {
        'weather-primary':'#00668A',
        'weather-secondary':'#004E71'
      }
    },
    fontFamily:{
      Roboto:["Roboto","sans-serif"]
    },
    container:{
        padding: "2rem",
        center: true
    },
    screens:{
      sm: '640px',
      md: '768px'
    }
  },
```
如果想要使用谷歌自定义字体，则需要在网站上`https://fonts.google.com/`获取嵌入代码片段，例如`Roboto`字体：
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&display=swap" rel="stylesheet">
```
然后，在`index.html`的`<head>`标签下插入这个片段。

由于在`tailwind`中，我们可以取代默认的css样式，因此我们可以把每个页面都需要的样式（例如每个页面固定添加padding）设置一个容器默认值，这样每个页面的容器都共享一个默认的样式，不需要再手动添加。

`screens`用于定义响应式设计的断点（breakpoints）。这些断点用于指定在不同屏幕宽度下应用不同的样式，从而实现响应式设计。通过配置`screens`，你可以自定义项目中使用的断点，以适应不同设备和屏幕尺寸。

这里定义了两个断点：

sm: '640px'：表示当屏幕宽度达到或超过 640 像素时，应用指定的样式。

md: '768px'：表示当屏幕宽度达到或超过 768 像素时，应用指定的样式。

### 三、导航栏
>需要用到： `<RouterLink>`，如何引用`tailwind css`的样式，组件式开发。

首先，我们需要一个图标库，通过这个网站，得到图标库的链接：`https://cdnjs.com/libraries/font-awesome`，然后用引入字体一样的方式，添加到`index.html`中。

接着我们就开始构建导航栏组件。
1. 在`components`下新建`SiteNavigation.vue`
2. 将页面在`App.vue`中引入
3. 在`App.vue`中为主程序构建`tailwind css`样式
4. 在`SiteNavigation.vue`中新增块并构建`tailwind css`样式

`App.vue`中
```html
<template>
  <div class="flex flex-col min-h-screen font-Roboto bg-weather-primary">
    <SiteNavigation />
    <RouterView />
  </div>
</template>

<script setup>
import { RouterView } from 'vue-router';
import SiteNavigation from './components/SiteNavigation.vue';
</script>

<style lang="scss" scoped></style>
```

在 App.vue 中这样引用导航栏组件 `<SiteNavigation />`，那么导航栏会被放置在 RouterView 组件的上方，并且会在所有视图之间共享，也就是说它会出现在每个视图（view）的顶部。

`SiteNavigation.vue`中
```html
<template>
    <header class="sticky top-0 bg-weather-primary shadow-lg">
        <nav class="container flex flex-col sm:flex-row 
        items-center gap-4 text-white py-6">
            <RouterLink :to="{name: 'home'}">
                <div class="flex items-center gap-3">
                    <i class="fa-solid fa-sun text-2xl"></i>
                    <p class="text-2xl">The Local Weather</p>
                </div>
            </RouterLink>
            <div class="flex gap-3 flex-1 justify-end">
                <i class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
                <i class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
            </div>
        </nav>
    </header>
</template>

<script setup>
import { RouterLink } from 'vue-router';
</script>
```
对组件的详细拆解在后文。

### 四、信息弹窗
>需要用到： `<slot>`，`props`,`emits`,`ref`，`<Transition>`，`<Teleport>`，变量与函数
#### 流程

1. 将弹窗部分也作为一个单独的组件`BaseModal.vue`
2. 在`SiteNavigation.vue`中引入
3. 构建`BaseModal`页面，并使用`props`、`emits`传递参数。

`BaseModal.vue`中
```html
<template>
  <Teleport to="body">
    <Transition name="modal-outer">
      <div
        v-show="modalActive"
        class="absolute w-full bg-black bg-opacity-30 h-screen top-0 left-0 flex justify-center px-8"
      >
        <Transition name="modal-inner">
          <div
            v-if="modalActive"
            class="p-4 bg-white self-start mt-32 max-w-screen-md"
          >
            <slot />
            <button
              class="text-white mt-8 bg-weather-primary py-2 px-6"
              @click="$emit('close-modal')"
            >
              Close
            </button>
          </div>
        </Transition>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
defineEmits(["close-modal"]);
defineProps({
  modalActive: {
    type: Boolean,
    default: false,
  },
});
</script>

<style scoped>
.modal-outer-enter-active,
.modal-outer-leave-active {
  transition: opacity 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02);
}

.modal-outer-enter-from,
.modal-outer-leave-to {
  opacity: 0;
}

.modal-inner-enter-active {
  transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02) 0.15s;
}

.modal-inner-leave-active {
  transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02);
}

.modal-inner-enter-from {
  opacity: 0;
  transform: scale(0.8);
}

.modal-inner-leave-to {
  transform: scale(0.8);
}
</style>
```
`SiteNavigation.vue`中
```html
<template>
  <header class="sticky top-0 bg-weather-primary shadow-lg">
    <nav
      class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6"
    >
      <RouterLink :to="{ name: 'home' }">
        <div class="flex items-center gap-3">
          <i class="fa-solid fa-sun text-2xl"></i>
          <p class="text-2xl">The Local Weather</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <i
          class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="toggleModal"
        ></i>
        <i
          class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"
        ></i>
      </div>

      <BaseModal
        :modalActive="modalActive"
        @close-modal="toggleModal"
      >
        <div class="text-black">
          <h1 class="text-2xl mb-1">About:</h1>
          <p class="mb-4">
            The Local Weather allows you to track the current and
            future weather of cities of your choosing.
          </p>
          <h2 class="text-2xl">How it works:</h2>
          <ol class="list-decimal list-inside mb-4">
            <li>
              Search for your city by entering the name into the
              search bar.
            </li>
            <li>
              Select a city within the results, this will take
              you to the current weather for your selection.
            </li>
            <li>
              Track the city by clicking on the "+" icon in the
              top right. This will save the city to view at a
              later time on the home page.
            </li>
          </ol>

          <h2 class="text-2xl">Removing a city</h2>
          <p>
            If you no longer wish to track a city, simply select
            the city within the home page. At the bottom of the
            page, there will be am option to delete the city.
          </p>
        </div>
      </BaseModal>
    </nav>
  </header>
</template>

<script setup>
import { RouterLink } from "vue-router";
import { ref } from "vue";
import BaseModal from "./BaseModal.vue";

const modalActive = ref(null);
const toggleModal = () => {
  modalActive.value = !modalActive.value;
};
</script>
```

### 五、城市搜索
>需要用到：`axios`，懒加载，`async`,`await`，`try..catch`，`<template>`

#### 流程
1. 在`HomeView.vue`主视图下构建
2. 安装`axios`依赖
3. 获取城市地理位置的api
4. 通过`axios`对api进行操作

`HomeView.vue`中
```html
<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResults"
      >
        <li
          v-for="searchResult in mapboxSearchResults"
          :key="searchResult.id"
          class="py-2 cursor-pointer"
        >
          {{ searchResult.place_name }}
        </li>
      </ul>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

const mapboxAPIKey =
  "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      const result = await axios.get(
        `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
      );
      mapboxSearchResults.value = result.data.features;

      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};
</script>

<style lang="scss" scoped></style>
```
但基于以上代码还不能够做到完美。例如，万一这个api失效了，那我们就得不到任何信息，这一点应该作为一个错误告知用户；同样的，如果我们搜索的这个城市是api中不存在的，也应该告知用户，而现在对于以上两种情况，页面的显示就和没有输入任何内容一样，显然还不够友好。

因此，我们需要使用`try...catch`对以上两种情况进行处理。如下：

```html
<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResults"
      >
        <p class="py-2" v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>
        <p
          class="py-2"
          v-if="!searchError && mapboxSearchResults.length === 0"
        >
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResults"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

const mapboxAPIKey =
  "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }

      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};
</script>

<style lang="scss" scoped></style>
```

### 六、搜索路由跳转
>需要用到：路由，`js解构`
#### 流程
1. 新建视图`CityView.vue`
2. 在路由`index.js`中新增路由
3. 在主路由下写跳转到新路由的代码

`index.js`中
```js
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'
import CityView from '../views/CityView.vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path: '/weather/:state/:city',
      name: 'cityView',
      component: CityView
    }
  ]
})

export default router
```
新路由接收两个参数

`HomeView.vue`中
```html
<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResults"
      >
        <p class="py-2" v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>
        <p
          class="py-2"
          v-if="!searchError && mapboxSearchResults.length === 0"
        >
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResults"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click=previewCity(searchResult)
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

const router = useRouter()
const previewCity = (searchResult) =>{
  console.log(searchResult);
  const [city,state] = searchResult.place_name.split(',')
  router.push({
    name: 'cityView',
    params:{state:state.replaceAll(" ",""),city:city},
    query:{
      lng:searchResult.geometry.coordinates[0],
      lat:searchResult.geometry.coordinates[1],
      preview:true
    }
  })
}

const mapboxAPIKey =
  "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }

      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};
</script>

<style lang="scss" scoped></style>
```

### 七、城市天气获取
>时间函数，`<Suspense>`

1. 首先，新建一个新的组件`AsyncCityView.vue`
2. 在网站上得到天气api，`https://openweathermap.org/`
3. 在`AsyncCityView.vue`组件中异步地获取城市天气信息
4. 在`CityView.vue`中使用`<Suspense>`在异步获取信息的过程中，渲染页面

`AsyncCityView.vue`中
```html
<template>
    <div></div>
</template>

<script setup>
import axios from "axios";
import { useRoute } from "vue-router";

const route = useRoute();
const getWeatherData = async () => {
    try {
        const weatherData = await axios.get(
            `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=metric`
        );

        // cal current date & time
        const localOffset = new Date().getTimezoneOffset() * 60000;
        const utc = weatherData.data.current.dt * 1000 + localOffset;
        weatherData.data.currentTime =
            utc + 1000 * weatherData.data.timezone_offset;

        // cal hourly weather offset
        weatherData.data.hourly.forEach((hour) => {
            const utc = hour.dt * 1000 + localOffset;
            hour.currentTime = utc + 1000 * weatherData.data.timezone_offset;
        });

        return weatherData.data;
    } catch (err) {
        console.log(err);
    }
};
const weatherData = await getWeatherData();
</script>
```
`CityView.vue`中
```html
<template>
    <div>
        <Suspense>
            <AsyncCityView />
            <template #fallback>
                <p>Loading...</p>
            </template>
        </Suspense>
    </div>
</template>

<script setup>
import AsyncCityView from "../components/AsyncCityView.vue";
</script>
```
之后，在`AsyncCityView.vue`中绘制天气页面。

`AsyncCityView.vue`中
```html
<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        You are currently previewing this city, click the "+"
        icon to start tracking this city.
      </p>
    </div>
    <!-- Weather Overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString(
            "en-us",
            {
              weekday: "short",
              day: "2-digit",
              month: "long",
            }
          )
        }}
        {{
          new Date(weatherData.currentTime).toLocaleTimeString(
            "en-us",
            {
              timeStyle: "short",
            }
          )
        }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>
      <p>
        Feels like
        {{ Math.round(weatherData.current.feels_like) }} &deg;
      </p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="
          `http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`
        "
        alt=""
      />
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Hourly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(
                  hourData.currentTime
                ).toLocaleTimeString("en-us", {
                  hour: "numeric",
                })
              }}
            </p>
            <img
              class="w-auto h-[50px] object-cover"
              :src="
                `http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`
              "
              alt=""
            />
            <p class="text-xl">
              {{ Math.round(hourData.temp) }}&deg;
            </p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Weekly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div
          v-for="day in weatherData.daily"
          :key="day.dt"
          class="flex items-center"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString(
                "en-us",
                {
                  weekday: "long",
                }
              )
            }}
          </p>
          <img
            class="w-[50px] h-[50px] object-cover"
            :src="
              `http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`
            "
            alt=""
          />
          <div class="flex gap-2 flex-1 justify-end">
            <p>H: {{ Math.round(day.temp.max) }}</p>
            <p>L: {{ Math.round(day.temp.min) }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute } from "vue-router";

const route = useRoute();
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
    );

    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime =
        utc + 1000 * weatherData.data.timezone_offset;
    });

    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};
const weatherData = await getWeatherData();
</script>
```

### 八、添加城市功能
>需要用到：`LocalStorage`，uid，JSON
1. 由于涉及到将内容存储到`LocalStorage`，因此需要uid，首先安装uid的依赖
2. 在`SiteNavigation`中为‘+’增加函数

`SiteNavigation`中
```html
<template>
  <header class="sticky top-0 bg-weather-primary shadow-lg">
    <nav
      class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6"
    >
      <RouterLink :to="{ name: 'home' }">
        <div class="flex items-center gap-3">
          <i class="fa-solid fa-sun text-2xl"></i>
          <p class="text-2xl">The Local Weather</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <i
          class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="toggleModal"
        ></i>
        <i
          class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          @click="addCity"
          v-if="route.query"
        ></i>
      </div>

      <BaseModal
        :modalActive="modalActive"
        @close-modal="toggleModal"
      >
        <div class="text-black">
          <h1 class="text-2xl mb-1">About:</h1>
          <p class="mb-4">
            The Local Weather allows you to track the current and
            future weather of cities of your choosing.
          </p>
          <h2 class="text-2xl">How it works:</h2>
          <ol class="list-decimal list-inside mb-4">
            <li>
              Search for your city by entering the name into the
              search bar.
            </li>
            <li>
              Select a city within the results, this will take
              you to the current weather for your selection.
            </li>
            <li>
              Track the city by clicking on the "+" icon in the
              top right. This will save the city to view at a
              later time on the home page.
            </li>
          </ol>

          <h2 class="text-2xl">Removing a city</h2>
          <p>
            If you no longer wish to track a city, simply select
            the city within the home page. At the bottom of the
            page, there will be am option to delete the city.
          </p>
        </div>
      </BaseModal>
    </nav>
  </header>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from "vue-router";
import { uid } from "uid";
import { ref } from "vue";
import BaseModal from "./BaseModal.vue";

const savedCities = ref([]);
const route = useRoute();
const router = useRouter();
const addCity = () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(
      localStorage.getItem("savedCities")
    );
  }

  const locationObj = {
    id: uid(),
    state: route.params.state,
    city: route.params.city,
    coords: {
      lat: route.query.lat,
      lng: route.query.lng,
    },
  };

  savedCities.value.push(locationObj);
  localStorage.setItem(
    "savedCities",
    JSON.stringify(savedCities.value)
  );

  let query = Object.assign({}, route.query);
  delete query.preview;
  router.replace({ query });
};

const modalActive = ref(null);
const toggleModal = () => {
  modalActive.value = !modalActive.value;
};
</script>
```

### 九、展示城市列表
>需要用到：`promise`，`forEach`
1. 新增`CityList.vue`和`CityCard.vue`
2. 完成城市列表页面逻辑
3. 在`HomeView.vue`中引入

`CityList.vue`中
```html
<template>
    <div v-for="city in savedCities" :key="city.id">
      <CityCard :city="city" @click="goToCityView(city)" />
    </div>
  
    <p v-if="savedCities.length === 0">
      No locations added. To start tracking a location, search in
      the field above.
    </p>
  </template>
  
  <script setup>
  import axios from "axios";
  import { ref } from "vue";
  import { useRouter } from "vue-router";
  import CityCard from "./CityCard.vue";
  
  const savedCities = ref([]);
  const getCities = async () => {
    if (localStorage.getItem("savedCities")) {
      savedCities.value = JSON.parse(
        localStorage.getItem("savedCities")
      );
  
      const requests = [];
      savedCities.value.forEach((city) => {
        requests.push(
          axios.get(
            `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=metric`
          )
        );
      });
  
      const weatherData = await Promise.all(requests);
  
      weatherData.forEach((value, index) => {
        savedCities.value[index].weather = value.data;
      });
    }
  };
  await getCities();
  
  const router = useRouter();
  const goToCityView = (city) => {
    router.push({
      name: "cityView",
      params: { state: city.state, city: city.city },
      query: {
        lat: city.coords.lat,
        lng: city.coords.lng,
      },
    });
  };
  </script>
```
`CityCard.vue`中
```html
<template>
    <div
      class="flex py-6 px-3 bg-weather-secondary rounded-md shadow-md cursor-pointer"
    >
      <div class="flex flex-col flex-1">
        <h2 class="text-3xl">{{ city.city }}</h2>
        <h3>{{ city.state }}</h3>
      </div>
  
      <div class="flex flex-col gap-2">
        <p class="text-3xl self-end">
          {{ Math.round(city.weather.main.temp) }}&deg;
        </p>
        <div class="flex gap-2">
          <span class="text-xs">
            H:
            {{ Math.round(city.weather.main.temp_max) }}&deg;
          </span>
          <span class="text-xs">
            L:
            {{ Math.round(city.weather.main.temp_min) }}&deg;
          </span>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  defineProps({
    city: {
      type: Object,
      default: () => {},
    },
  });
  </script>
```
`HomeView.vue`中
```html
<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResults"
      >
        <p class="py-2" v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>
        <p
          class="py-2"
          v-if="!searchError && mapboxSearchResults.length === 0"
        >
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResults"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <p>Loading...</p>
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";

const router = useRouter();
const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(",");
  router.push({
    name: "cityView",
    params: { state: state.replaceAll(" ", ""), city: city },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    },
  });
};

const mapboxAPIKey =
  "pk.eyJ1Ijoiam9obmtvbWFybmlja2kiLCJhIjoiY2t5NjFzODZvMHJkaDJ1bWx6OGVieGxreSJ9.IpojdT3U3NENknF6_WhR2Q";
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        );
        mapboxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }

      return;
    }
    mapboxSearchResults.value = null;
  }, 300);
};
</script>

<style lang="scss" scoped></style>
```

### 十、删除城市
>需要用到：数组的`filter`方法

我们需要根据id来删除对应的内容，因此需要在原先的页面中传递id。

`CityList.vue`中
```js
const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: { state: city.state, city: city.city },
    query: {
      id: city.id,
      lat: city.coords.lat,
      lng: city.coords.lng,
    },
  });
};
```

`SiteNavigation.vue`中
```js
const addCity = () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));
  }

  const locationObj = {
    id: uid(),
    state: route.params.state,
    city: route.params.city,
    coords: {
      lat: route.query.lat,
      lng: route.query.lng,
    },
  };

  savedCities.value.push(locationObj);
  localStorage.setItem("savedCities", JSON.stringify(savedCities.value));

  let query = Object.assign({}, route.query);
  delete query.preview;
  query.id = locationObj.id;
  router.replace({ query });
};
```

之后再删除时，对应id相同则从存储中去除。

`AsyncCityView`中
```html
<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        You are currently previewing this city, click the "+"
        icon to start tracking this city.
      </p>
    </div>
    <!-- Weather Overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString(
            "en-us",
            {
              weekday: "short",
              day: "2-digit",
              month: "long",
            }
          )
        }}
        {{
          new Date(weatherData.currentTime).toLocaleTimeString(
            "en-us",
            {
              timeStyle: "short",
            }
          )
        }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>
      <p>
        Feels like
        {{ Math.round(weatherData.current.feels_like) }} &deg;
      </p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="
          `http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`
        "
        alt=""
      />
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Hourly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(
                  hourData.currentTime
                ).toLocaleTimeString("en-us", {
                  hour: "numeric",
                })
              }}
            </p>
            <img
              class="w-auto h-[50px] object-cover"
              :src="
                `http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`
              "
              alt=""
            />
            <p class="text-xl">
              {{ Math.round(hourData.temp) }}&deg;
            </p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Weekly Weather -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div
          v-for="day in weatherData.daily"
          :key="day.dt"
          class="flex items-center"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString(
                "en-us",
                {
                  weekday: "long",
                }
              )
            }}
          </p>
          <img
            class="w-[50px] h-[50px] object-cover"
            :src="
              `http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`
            "
            alt=""
          />
          <div class="flex gap-2 flex-1 justify-end">
            <p>H: {{ Math.round(day.temp.max) }}</p>
            <p>L: {{ Math.round(day.temp.min) }}</p>
          </div>
        </div>
      </div>
    </div>

    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
    >
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=7efa332cf48aeb9d2d391a51027f1a71&units=imperial`
    );

    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime =
        utc + 1000 * weatherData.data.timezone_offset;
    });

    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};
const weatherData = await getWeatherData();

const router = useRouter();
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem("savedCities"));
  const updatedCities = cities.filter(
    (city) => city.id !== route.query.id
  );
  localStorage.setItem(
    "savedCities",
    JSON.stringify(updatedCities)
  );
  router.push({
    name: "home",
  });
};
</script>
```

### 十一、载入动画
>需要用到：自定义延时，`v-slot`,`<component>`
1. 新建组件`AnimatedPlaceholder.vue`，在其中设置动画效果，是一个脉冲的灰色动画，用于提示加载过程的占位
2. 新建组件`CityCardSkeleton.vue`，在对应位置引入动画占位组件并在`HomeView.vue`视图中引入这个动画占位（在异步任务还未获取到请求内容时）
3. 新建组件`CityViewSkeleton.vue`，参考以上同样的方式
4. 可以在请求数据的位置之前，加入延时，以平滑动画效果
5. 对于路由平滑动画，我们需要在`App.vue`中添加`v-slot`的方式实现

`AnimatedPlaceholder.vue`
```html
<template>
  <div class="animate-pulse bg-gradient-to-r from-gray-100 ">
    &nbsp;
  </div>
</template>
```
`CityCardSkeleton.vue`
```html
<template>
  <div
    class="flex py-6 px-3  bg-primary-green rounded-md shadow-md"
  >
    <div class="flex flex-col flex-1 gap-2">
      <AnimatedPlaceholder class="max-w-[50%]" />
      <AnimatedPlaceholder class="max-w-[40%]" />
    </div>

    <div class="flex flex-col items-end flex-1 gap-2">
      <AnimatedPlaceholder class="max-w-[50px] w-full" />
      <AnimatedPlaceholder class="max-w-[75px] w-full" />
    </div>
  </div>
</template>

<script setup>
import AnimatedPlaceholder from "./AnimatedPlaceholder.vue";
</script>
```
`CityViewSkeleton.vue`
```html
<template>
  <div class="flex flex-col flex-1">
    <!-- Overview -->
    <div class="flex flex-col py-12 items-center">
      <AnimatedPlaceholder class="max-w-[300px] w-full mb-2" />
      <AnimatedPlaceholder class="max-w-[300px] w-full mb-12" />
      <AnimatedPlaceholder
        class="max-w-[300px] h-[100px] w-full mb-12"
      />
      <AnimatedPlaceholder class="max-w-[300px] w-full mb-8" />
      <AnimatedPlaceholder
        class="max-w-[300px] h-[75px] w-full"
      />
    </div>
    <!-- Hourly -->
    <div class="flex flex-col py-12 px-8 items-center">
      <AnimatedPlaceholder
        class="max-w-screen-md h-[100px] w-full mb-12 "
      />
    </div>
    <!-- Weekly -->
    <div class="flex flex-col py-12 px-8 items-center">
      <AnimatedPlaceholder
        class="max-w-screen-md h-[100px] w-full mb-12"
      />
    </div>
  </div>
</template>

<script setup>
import AnimatedPlaceholder from "./AnimatedPlaceholder.vue";
</script>
```
`HomeView.vue`
```html
<Suspense>
  <CityList />
  <template #fallback>
    <CityCardSkeleton />
  </template>
</Suspense>
```
`CityView.vue`
```html
<template>
  <div>
    <Suspense>
      <template #default>
        <AsyncCityView />
      </template>
      <template #fallback>
        <CityViewSkeleton />
      </template>
    </Suspense>
  </div>
</template>

<script setup>
import AsyncCityView from "../components/AsyncCityView.vue";
import CityViewSkeleton from "../components/CityViewSkeleton.vue";
</script>
```

`App.vue`
```html
<template>
  <div
    class="flex flex-col min-h-screen font-Roboto bg-weather-primary"
  >
    <SiteNavigation />
    <RouterView class="flex-1" v-slot="{ Component }">
      <Transition name="page">
        <component :is="Component" />
      </Transition>
    </RouterView>
  </div>
</template>

<script setup>
import { RouterView } from "vue-router";
import SiteNavigation from "./components/SiteNavigation.vue";
</script>

<style>
.page-enter-active {
  transition: 600ms ease all;
}

.page-enter-from {
  opacity: 0;
}
</style>
```

### 十二、设置元信息
>需要用到：`router`

`index.js`
```js
import { createRouter, createWebHistory } from "vue-router";
import HomeView from "../views/HomeView.vue";
import CityView from "../views/CityView.vue";

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: "/",
      name: "home",
      component: HomeView,
      meta: {
        title: "Home",
      },
    },
    {
      path: "/weather/:state/:city",
      name: "cityView",
      component: CityView,
      meta: {
        title: "Weather",
      },
    },
  ],
});

router.beforeEach((to, from, next) => {
  document.title = `${
    to.params.state
      ? `${to.params.city}, ${to.params.state}`
      : to.meta.title
  } | The Local Weather`;
  next();
});

export default router;
```

### 十三、部署上线
我们使用`Netlify`进行服务器托管上线
1. 先将项目上传至`github`
2. 在`Netlify`中选择`github`中的对应项目进行打包设置
```terminal
git init
git add .
git commit -m "..."
git branch -M main
git remote add origin git@github.com:Kimikunn/weather.git
git push -u origin main
```


## 组件
### `SiteNavigation.vue`
#### 组件结构分析

该组件是一个 Vue 3 组件，使用了 Tailwind CSS 类来实现样式。下面详细描述组件中每个块的作用，以及每个 Tailwind CSS 类所起的作用。

```html
<template>
    <header class="sticky top-0 bg-weather-primary shadow-lg">
        <nav class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6">
            <RouterLink :to="{name: 'home'}">
                <div class="flex items-center gap-3">
                    <i class="fa-solid fa-sun text-2xl"></i>
                    <p class="text-2xl">The Local Weather</p>
                </div>
            </RouterLink>
            <div class="flex gap-3 flex-1 justify-end">
                <i class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
                <i class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
            </div>
        </nav>
    </header>
</template>

<script setup>
import { RouterLink } from 'vue-router';
</script>
```

#### 详细描述

##### `<header>` 块

```html
<header class="sticky top-0 bg-weather-primary shadow-lg">
```

- `sticky`: 使元素粘性定位，元素在滚动到一定位置后固定在顶部。
- `top-0`: 将元素固定在页面顶部。
- `bg-weather-primary`: 设置背景颜色为自定义的 `weather-primary` 颜色（在 Tailwind 配置中定义）。
- `shadow-lg`: 应用大阴影效果。

##### `<nav>` 块

```html
<nav class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6">
```

- `container`: 将内容居中，并应用一些默认的宽度和内边距。
- `flex`: 应用 Flexbox 布局。
- `flex-col`: 将子元素按列排列（垂直方向）。
- `sm:flex-row`: 在 `sm` 屏幕及以上尺寸时，将子元素按行排列（水平方向）。
- `items-center`: 在交叉轴（垂直轴）上居中对齐子元素。
- `gap-4`: 子元素之间的间距为 `1rem`。
- `text-white`: 设置文本颜色为白色。
- `py-6`: 设置垂直内边距为 `1.5rem`。

##### `<RouterLink>` 块

```html
<RouterLink :to="{name: 'home'}">
    <div class="flex items-center gap-3">
        <i class="fa-solid fa-sun text-2xl"></i>
        <p class="text-2xl">The Local Weather</p>
    </div>
</RouterLink>
```

- `RouterLink`: 用于 Vue Router 的导航链接，指向名为 `home` 的路由。
- `flex`: 将子元素按行排列（水平排列）。
- `items-center`: 在交叉轴（垂直轴）上居中对齐子元素。
- `gap-3`: 子元素之间的间距为 `0.75rem`。

###### `<i>` 图标和 `<p>` 文本

```html
<i class="fa-solid fa-sun text-2xl"></i>
<p class="text-2xl">The Local Weather</p>
```

- `text-2xl`: 设置文本或图标大小为 `2xl`（通常为 `1.5rem`）。

##### `<div>` 块（图标容器）

```html
<div class="flex gap-3 flex-1 justify-end">
    <i class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
    <i class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
</div>
```

- `flex`: 应用 Flexbox 布局。
- `gap-3`: 子元素之间的间距为 `0.75rem`。
- `flex-1`: 使元素占据剩余的空间。
- `justify-end`: 在主轴（水平轴）上将子元素对齐到末端（右侧）。

###### 图标

```html
<i class="fa-solid fa-circle-info text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
<i class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"></i>
```

- `text-xl`: 设置图标大小为 `xl`（通常为 `1.25rem`）。
- `hover:text-weather-secondary`: 在悬停时，设置文本颜色为 `weather-secondary`。
- `duration-150`: 过渡效果持续 `150ms`。
- `cursor-pointer`: 设置鼠标指针为指针状，表示可点击。

#### 总结

这个组件通过 Tailwind CSS 的实用类（utility classes）实现了响应式设计、居中对齐、间距设置、颜色设置和悬停效果。通过使用这些类，开发者可以快速而简洁地实现各种复杂的样式需求。

## 其他
### 更新node和npm
#### 不使用nvm
```terminal
# 在全局下
npm install -g n
# 安装新版本的 Node
sudo n lts
# 删除以前安装的缓存
n prune
```
#### 使用nvm
```terminal
# 先查看是否有nvm
nvm -v
# 列出nvm中node的版本
nvm ls
# 安装最新版本的node
nvm install node
# 最新的稳定版作为默认node
nvm alias default stable
# 指定某个版本为默认node
nvm alias default 20
# 使用默认版本的node
nvm use default
# 使用指定版本的node
nvm use 20
# 删除某个版本的node
nvm uninstall 16
```
### 关于`tailwind css`
#### 配置文件中的`theme`
在 Tailwind CSS 中，你可以直接在 `theme` 对象内定义颜色，也可以在 `theme.extend` 内定义颜色。这两种方式的主要区别在于如何处理默认主题配置和自定义配置的合并方式。

##### 直接在 `theme` 对象内定义

如果你直接在 `theme` 对象内定义颜色，你的自定义配置会覆盖 Tailwind CSS 的默认配置。示例：

```javascript
module.exports = {
  theme: {
    colors: {
      customBlue: '#1e3a8a',
      customGreen: '#10b981',
      // 这里没有包含 Tailwind CSS 默认的颜色，因此它们会被移除
    },
  },
  variants: {},
  plugins: [],
}
```

在这种情况下，Tailwind CSS 的默认颜色配置将被完全覆盖，你只会有 `customBlue` 和 `customGreen` 两种颜色可用。

##### 在 `theme.extend` 内定义

如果你在 `theme.extend` 内定义颜色，你的自定义配置会被添加到 Tailwind CSS 的默认配置中，而不会覆盖默认的颜色配置。示例：

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        customBlue: '#1e3a8a',
        customGreen: '#10b981',
      },
    },
  },
  variants: {},
  plugins: [],
}
```

在这种情况下，Tailwind CSS 的默认颜色配置仍然可用，并且你添加的 `customBlue` 和 `customGreen` 会作为额外的颜色选项添加到默认配置中。

##### 区别总结

- **覆盖 vs 扩展**：
  - **覆盖**：直接在 `theme` 内定义颜色会完全覆盖默认配置。
  - **扩展**：在 `theme.extend` 内定义颜色会将自定义配置添加到默认配置中，不会覆盖默认值。

- **灵活性**：
  - **覆盖**：使用覆盖方式，你需要显式地定义所有你想要保留的配置，包括默认配置，否则它们会被移除。
  - **扩展**：使用扩展方式，你可以仅添加或修改你需要的部分，默认配置会自动保留。

##### 示例对比

**覆盖方式**：

```javascript
module.exports = {
  theme: {
    colors: {
      customBlue: '#1e3a8a',
      customGreen: '#10b981',
      // Tailwind 默认颜色不会保留
    },
  },
  variants: {},
  plugins: [],
}
```

**扩展方式**：

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        customBlue: '#1e3a8a',
        customGreen: '#10b981',
        // Tailwind 默认颜色会保留
      },
    },
  },
  variants: {},
  plugins: [],
}
```

一般来说，使用 `theme.extend` 是更推荐的方式，因为它保留了 Tailwind CSS 的默认配置，使你的自定义配置更加灵活和易于维护。

#### 断点

在 Tailwind CSS 中，这些断点可以在类名中作为前缀来使用，以便在不同的屏幕尺寸下应用不同的样式。例如：

```html
<div class="bg-weather-primary sm:bg-weather-secondary md:bg-weather-primary">
  <!-- 在小于640px宽度时，背景颜色为weather-primary -->
  <!-- 在640px到768px宽度时，背景颜色为weather-secondary -->
  <!-- 在大于768px宽度时，背景颜色为weather-primary -->
</div>

```
Tailwind CSS 默认提供了几个断点：

- sm: 640px
- md: 768px
- lg: 1024px
- xl: 1280px
- 2xl: 1536px

你可以根据项目需求自定义这些断点。

#### 引入样式
Tailwind CSS 的样式是通过在 HTML 元素上添加类（class）来实现的。Tailwind CSS 提供了一组预定义的类，这些类可以直接应用于 HTML 元素，从而快速实现各种样式效果。

##### 示例

以下是一个使用 Tailwind CSS 的简单示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tailwind CSS Example</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>
<body class="bg-gray-100 flex items-center justify-center h-screen">
  <div class="bg-white p-6 rounded-lg shadow-lg">
    <h1 class="text-2xl font-bold mb-2">Hello, Tailwind CSS!</h1>
    <p class="text-gray-700">This is an example of using Tailwind CSS classes to style HTML elements.</p>
    <button class="mt-4 bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">Click me</button>
  </div>
</body>
</html>
```

##### 样式类解释

- `bg-gray-100`: 设置背景颜色为浅灰色。
- `flex`: 应用 Flexbox 布局。
- `items-center`: 在 Flexbox 布局中，使项目在交叉轴（垂直轴）上居中对齐。
- `justify-center`: 在 Flexbox 布局中，使项目在主轴（水平轴）上居中对齐。
- `h-screen`: 设置元素高度为视口高度。
- `bg-white`: 设置背景颜色为白色。
- `p-6`: 设置内边距为 1.5rem。
- `rounded-lg`: 设置圆角大小。
- `shadow-lg`: 应用大阴影效果。
- `text-2xl`: 设置文本大小为 2xl。
- `font-bold`: 设置字体加粗。
- `mb-2`: 设置底部外边距为 0.5rem。
- `text-gray-700`: 设置文本颜色为深灰色。
- `mt-4`: 设置顶部外边距为 1rem。
- `bg-blue-500`: 设置背景颜色为蓝色。
- `text-white`: 设置文本颜色为白色。
- `px-4`: 设置水平内边距为 1rem。
- `py-2`: 设置垂直内边距为 0.5rem。
- `rounded`: 设置圆角。
- `hover:bg-blue-600`: 设置悬停时的背景颜色为更深的蓝色。

##### 响应式设计

Tailwind CSS 还支持响应式设计，可以通过前缀类实现不同屏幕尺寸下的样式变化。例如：

```html
<div class="bg-white p-6 md:p-12 lg:p-24">
  <h1 class="text-xl md:text-2xl lg:text-4xl">Responsive Heading</h1>
</div>
```

在这个例子中，`p-6` 表示在默认情况下内边距为 `1.5rem`，`md:p-12` 表示在中等屏幕及以上的尺寸内边距为 `3rem`，`lg:p-24` 表示在大屏幕及以上的尺寸内边距为 `6rem`。

总的来说，Tailwind CSS 通过一组实用的类名，使得直接在 HTML 中应用样式变得简单而高效。