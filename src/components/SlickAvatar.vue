<script setup lang="ts">
import { ref, watch, watchEffect } from "vue"

// Place widgets in this folder(public) and name them as `widgetType/widgetName.svg`
const widgetsPath = "/slick-avatar"

const widgets = {
    face: ["base"],
    eyebrows: ["down", "eyelashesdown", "eyelashesup", "up"],
    
    glasses: ["round", "square"],
    mouth: ["frown", "laughing", "nervous", "pucker", "sad", "smile", "smirk", "surprised"],
    nose: ["curve", "pointed", "round"],
    top: ["beanie", "clean", "danny", "fonze", "funny", "pixie", "punk", "turban", "wave"],
    
    ears: ["attached", "detached"],
    earrings: ["hoop", "stud"],

    eyes: ["ellipse", "eyeshadow", "round", "smiling"],

    clothes: ["collared", "crew", "open"],
} as const

type WidgetType = keyof typeof widgets
type Widget<T extends WidgetType> = typeof widgets[T][number]

type Person = {
    [K in WidgetType]: {
        color: string
        widget: Widget<K> 
    }
}

const widgetType = Object.keys(widgets) as WidgetType[]

const props = defineProps<{
    modelValue: Person | null
}>()

const emit = defineEmits<{
    (e: "update:modelValue", value: Person): void
}>()

watch(() => props.modelValue, (person) => {
    if (!person) {
        return emit("update:modelValue", getRandomPerson())
    }
}, { immediate: true })


async function getWidgetSVG<T extends WidgetType>(widgetType: T, widget: Widget<T>) {
    const path = `${widgetsPath}/${widgetType}/${widget}.svg?raw`

    const response = await fetch(path).catch(() => null)

    if (!response?.ok) {
        throw new Error(`Failed to fetch ${path}`)
    }

    return response.text()
}

function randomItem<T extends ArrayLike<any>>(array: T): T[number] {
    return array[Math.floor(Math.random() * array.length)]
}

function getRandomWidget<T extends WidgetType>(widgetType: T): Widget<T> {
    return randomItem(widgets[widgetType])
}

function getRandomPerson(): Person {
    const isBlackRace = randomItem([true, false] as const)
    const hasSpecialHair = randomItem([false, false, true] as const)

    const blackSkinColor = [
        "#D2B48C", // Tan
        "#F5CBA7", // Peach
        "#C68642", // Light Brown
        "#8D5524", // Medium Brown
        "#6B4423", // Dark Brown
        "#4A2511", // Deep Brown
    ]

    const whiteSkinColor = [
        "#FFDFC4", // Light Peach
        "#F0D5B1", // Light Beige
        "#F3C5A8", // Fair
        "#EAC086", // Pale
        "#D1B399", // Medium Beige
        "#C68642"  // Light Brown
    ]

    const blackHairColor = [
        "#D2B48C", // Tan
        "#F5CBA7", // Peach
        "#C68642", // Light Brown
        "#8D5524", // Medium Brown
        "#6B4423", // Dark Brown
        "#4A2511", // Deep Brown
        "#3B2F2F", // Dark Sienna
        "#231F20"  // Almost Black
    ]

    const whiteHairColor = [
        "#FAF0BE", // Lemon Chiffon
        "#D2B48C", // Tan
        "#FFB347", // Apricot
        "#FFD700", // Golden
        "#A52A2A", // Reddish Brown
        "#BC8F8F", // Rosy Brown
        "#DAA520", // Goldenrod
        "#8B4513", // Saddle Brown
        "#8B0000"  // Dark Red
    ]

    function randomColor() {
        return "#" + (Math.random() * 0xFFFFFF << 0).toString(16).padStart(6, "0")
    }
    
    const person: Person = Object.fromEntries(widgetType.map((widgetType) => {        
        let color = randomColor()

        return [[widgetType], {
            color: color,
            widget: getRandomWidget(widgetType)
        }]
    }))

    let skinColor = isBlackRace ? randomItem(blackSkinColor) : randomItem(whiteSkinColor)
    let hairColor = isBlackRace ? randomItem(blackHairColor) : randomItem(whiteHairColor)

    while (skinColor === hairColor) {
        hairColor = randomColor()
    }

    person.face.color = skinColor
    person.ears.color = skinColor

    if (!hasSpecialHair) {
        person.eyebrows.color = hairColor
        person.top.color = hairColor
    }

    return person
}

async function getPersonSVG(person: Person): Promise<Record<WidgetType, {
    color: string
    svg: string
}>> {
    const personSVG = await Promise.all(widgetType.map(async (widgetType) => {
        const svg = [widgetType, {
            color: person[widgetType].color,
            svg: await getWidgetSVG(widgetType, person[widgetType].widget)
        }]
        return svg
    }))

    return Object.fromEntries(personSVG)
}


const svgContent = ref<string | null>(null)
const avatarSize = ref(100)

watchEffect(async () => {
    if (!props.modelValue) return null

    const personWithSVG = await getPersonSVG(props.modelValue)
    
    const svgRawList = Object.entries(personWithSVG).map(([widget, personWithSVG]) => {
        const svg = personWithSVG.svg
        const color = personWithSVG.color

        const content = svg
            .slice(svg.indexOf('>', svg.indexOf('<svg')) + 1)
            .replace('</svg>', '')
            .replace(/\$fillColor/g, color)

        return `
            <g id="vue-color-avatar-${widget}">
                ${content}
            </g>
        `
    })

    svgContent.value = `
        <svg
            width="${avatarSize.value}"
            height="${avatarSize.value}"
            viewBox="0 0 ${avatarSize.value / 0.7} ${avatarSize.value / 0.7}"
            preserveAspectRatio="xMidYMax meet"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
        >
            <g transform="translate(100, 65)">
                ${svgRawList.join('')}
            </g>
        </svg>
    `
})
</script>

<template>
    <div
        ref="avatarRef"
        class="vue-color-avatar"
        :style="{
            width: `${avatarSize}px`,
            height: `${avatarSize}px`,
        }"
    >
        <div class="avatar-payload" v-html="svgContent" />
    </div>
</template>

<style scoped lang="scss">
.clothes {
    color: red;
}

.vue-color-avatar {
    position: relative;
    overflow: hidden;

    .avatar-payload {
        position: relative;
        z-index: 2;
        width: 100%;
        height: 100%;
    }
}
</style>