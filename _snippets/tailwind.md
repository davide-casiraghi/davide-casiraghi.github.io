---
id: tailwind
title: Tailwind
description: List of tailwind snippets
---

## Tailwind snippets

### Grids

**How do I make a grid?**  
This two columns on mobile are on two different lines, after the breakpoint md, they go on just one line.   
Col1 will occupy [2/6], Col2 will occupy [4/6].   
As you see the grid is defined of 6.   

```html
<div class="md:grid md:grid-cols-6 md:gap-4">
	<div class="md:col-span-2">
            col 1
	</div>
	<div class="md:col-span-4 mt-5 md:mt-0">
            col 2
	</div>
</div>
```

**How to use breakpoints with a grid?**   
https://tailwindcss.com/docs/grid-column#responsive

---
### Flex

How do I align bottom right using Flex?
- items-end (align the element to the end of the bottom of the div)
- justify-end (align the item to the right of the div)
```html
<div class="flex h-64 bg-red-100 inline-block items-end">
  <div>Align middle with margin</div>
  <div>2</div>
  <div>3</div>
</div>
```

---

### Alignments

How do I align middle a text or div?

Div
```html
<div class="flex h-screen">
  <div class="m-auto">
    <h3>title</h3>
    <button>button</button>
  </div>
</div>
```

Text
```html
<div class="flex w-10 h-10">
    <a href="#" class="m-auto">
        {{$key}}
    </a>
</div>
```
---


###  Breakpoints
- Tailwind ships with 4 default brakepoints
    - sm (640px)
    - md (768px)
    - Lg (1024px)
    - XL (1280px)
    - 2xl (1536px)

---

###  How I control the position of the grid rows as if it would be a float-left or right related to the brakpoint?
Using Rowstart and Colstart

```html
<div class="grid grid-cols-3 gap-4">
  <div class="col-span-3 lg:col-span-1 lg:row-start-1 lg:col-start-3 h-6 bg-red-400">red</div>
  <div class="col-span-3 lg:col-span-2 lg:row-start-1 lg:col-start-1 h-6 bg-blue-400">blue</div>
</div>
```

https://tailwindcss.com/docs/grid-column


---

### Animate bouce

This code create an animated arrow that point down.

```html
<div class="flex justify-center mt-5">
    <svg class="animate-bounce w-6 h-6 text-amber-900" fill="none" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" stroke="currentColor">
        <path d="M19 14l-7 7m0 0l-7-7m7 7V3"></path>
    </svg>
</div>
```



---

Divide

Divider classes to add dividers between your elements instead of using borders.

```html
<div class="flex divide-x-2 divide-gray-300">
    <div class="px-5">
        <div class="w-20 h-20 bg-black"></div>
    </div>
    <div class="px-5">
        <div class="w-20 h-20 bg-black"></div>
    </div>
    <div class="px-5">
        <div class="w-20 h-20 bg-black"></div>
    </div>
    <div class="px-5">
        <div class="w-20 h-20 bg-black"></div>
    </div>
</div>
```