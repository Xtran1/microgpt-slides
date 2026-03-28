---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: MicroGPT
info: |
  ## Making your own generator from scratch
  And understanding the logic behind it
# apply UnoCSS classes to the current slide
class: text-center
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-up
# enable Comark Syntax: https://comark.dev/syntax/markdown
comark: true
# duration of the presentation
duration: 35min
---

# What is a Large Language Model?

Making your own generator from scratch

And understanding the logic behind it

<!--
Introduction
-->

---
transition: slide-up
---

# The Bird's Eye View

At its simplest, an LLM is a machine that transforms one sequence of text into another.

<div class="flex items-center justify-center mt-24 space-x-8 h-48 relative">

  <!-- Input Block -->
  <div class="flex flex-col items-center shrink-0">
    <div class="w-32 h-20 border-4 border-gray-300 rounded-xl bg-gray-50 flex items-center justify-center shadow-md">
      <span class="text-xl font-bold text-gray-600 uppercase tracking-widest">Text</span>
    </div>
    <div class="mt-4 text-[10px] font-bold text-gray-400 uppercase tracking-widest">Input</div>
  </div>

  <!-- Arrow 1 -->
  <div class="text-3xl text-gray-300 font-bold">→</div>

  <!-- Middle Section: LLM vs Encoder/Decoder -->
  <div class="relative w-[450px] h-full flex items-center justify-center">
    <!-- Phase 1: The LLM "Black Box" -->
    <div
      class="absolute inset-0 flex flex-col items-center justify-center transition-all duration-700"
      :class="$clicks >= 1 ? 'opacity-0 scale-75 blur-sm pointer-events-none' : 'opacity-100 scale-100'"
    >
      <div class="w-56 h-36 border-4 border-blue-500 rounded-2xl bg-blue-50 flex items-center justify-center shadow-xl ring-8 ring-blue-50">
        <span class="text-4xl font-black text-blue-600 uppercase tracking-[0.2em]">LLM</span>
      </div>
      <div class="mt-4 text-xs font-bold text-blue-400 uppercase tracking-widest">The Black Box</div>
    </div>
    <!-- Phase 2: Breaking Apart (Encoder -> Decoder) -->
    <div
      class="absolute inset-0 flex items-center justify-center space-x-6 transition-all duration-700 delay-200"
      :class="$clicks >= 1 ? 'opacity-100 scale-100' : 'opacity-0 scale-110 blur-md pointer-events-none'"
    >
      <!-- Encoder -->
      <div class="flex flex-col items-center">
        <div class="w-40 h-28 border-4 border-blue-400 rounded-xl bg-blue-50 flex items-center justify-center shadow-lg">
          <span class="text-xl font-bold text-blue-600 uppercase tracking-widest">Encoder</span>
        </div>
        <div class="mt-4 text-[10px] font-bold text-blue-400 uppercase tracking-widest">Understanding</div>
      </div>
      <!-- Internal Arrow -->
      <div class="text-3xl text-blue-300 font-bold">→</div>
      <!-- Decoder -->
      <div class="flex flex-col items-center">
        <div class="w-40 h-28 border-4 border-indigo-400 rounded-xl bg-indigo-50 flex items-center justify-center shadow-lg">
          <span class="text-xl font-bold text-indigo-600 uppercase tracking-widest">Decoder</span>
        </div>
        <div class="mt-4 text-[10px] font-bold text-indigo-400 uppercase tracking-widest">Generation</div>
      </div>
    </div>

  </div>

  <!-- Arrow 2 -->
  <div class="text-3xl text-gray-300 font-bold">→</div>

  <!-- Output Block -->
  <div class="flex flex-col items-center shrink-0">
    <div class="w-32 h-20 border-4 border-emerald-500 rounded-xl bg-emerald-50 flex items-center justify-center shadow-md">
      <span class="text-xl font-bold text-emerald-600 uppercase tracking-widest">Text</span>
    </div>
    <div class="mt-4 text-[10px] font-bold text-emerald-400 uppercase tracking-widest">Output</div>
  </div>

</div>

<div class="mt-12 text-center text-gray-400 italic">
  <span v-if="$clicks == 0">"How do we turn those words into math, and that math back into words?"</span>
  <span v-else class="text-blue-500 font-bold">Inside the box, we separate the "What is this?" from the "What comes next?"</span>
</div>

<!-- Ensure click 1 exists -->
<div v-click="1" class="hidden"></div>

---
transition: slide-up
---

# Inside the Encoder

The first step is turning raw text into a format the math can "see."

<div class="flex items-center justify-center mt-24 space-x-6 h-48 relative px-4">

  <!-- Input Block (Shrinking slightly but not dimming) -->
  <div class="flex flex-col items-center shrink-0 transition-all duration-700" :class="$clicks >= 1 ? 'scale-75' : ''">
    <div class="w-32 h-20 border-2 border-gray-300 rounded-xl bg-gray-50 flex items-center justify-center shadow-sm">
      <span class="text-lg font-bold text-gray-500 uppercase tracking-widest">Text</span>
    </div>
    <div class="mt-4 text-[8px] font-bold text-gray-400 uppercase tracking-widest">Input</div>
  </div>

  <!-- Arrow -->
  <div class="text-2xl text-gray-300 font-bold transition-all duration-700" :class="$clicks >= 1 ? 'scale-75' : ''">→</div>

  <!-- Encoder Internals -->
  <div class="relative w-[500px] h-full flex items-center justify-center">
    <!-- Phase 1: The Encoder Box -->
    <div
      class="absolute inset-0 flex flex-col items-center justify-center transition-all duration-700"
      :class="$clicks >= 1 ? 'opacity-0 scale-90 blur-sm pointer-events-none' : 'opacity-100'"
    >
      <div class="w-48 h-32 border-4 border-blue-400 rounded-xl bg-blue-50 flex items-center justify-center shadow-lg">
        <span class="text-2xl font-bold text-blue-600 uppercase tracking-widest">Encoder</span>
      </div>
      <div class="mt-4 text-[10px] font-bold text-blue-400 uppercase tracking-widest">Understanding</div>
    </div>
    <!-- Phase 2: Tokens -> Embeddings -->
    <div
      class="absolute inset-0 flex items-center justify-center space-x-6 transition-all duration-700 delay-200"
      :class="$clicks >= 1 ? 'opacity-100 scale-100' : 'opacity-0 scale-110 blur-md pointer-events-none'"
    >
      <!-- Tokens -->
      <div class="flex flex-col items-center">
        <div class="w-40 h-28 border-4 border-emerald-400 rounded-xl bg-emerald-50 flex items-center justify-center shadow-md">
          <span class="text-xl font-bold text-emerald-600 uppercase tracking-widest">Tokens</span>
        </div>
        <div class="mt-4 text-[10px] font-bold text-emerald-400 uppercase tracking-widest">Integers</div>
      </div>
      <div class="text-2xl text-emerald-300 font-bold">→</div>
      <!-- Embeddings -->
      <div class="flex flex-col items-center">
        <div class="w-40 h-28 border-4 border-blue-400 rounded-xl bg-blue-50 flex items-center justify-center shadow-md">
          <span class="text-xl font-bold text-blue-600 uppercase tracking-widest opacity-100">Embeddings</span>
        </div>
        <div class="mt-4 text-[10px] font-bold text-blue-400 uppercase tracking-widest">Vectors</div>
      </div>
    </div>
  </div>

  <!-- Arrow (Connecting to Decoder) -->
  <div class="text-2xl text-indigo-200 font-bold transition-all duration-700" :class="$clicks >= 1 ? 'scale-75' : ''">→</div>

  <!-- Decoder (Shrinking slightly but not dimming) -->
  <div class="flex flex-col items-center transition-all duration-700" :class="$clicks >= 1 ? 'scale-75' : ''">
    <div class="w-32 h-20 border-2 border-indigo-400 rounded-xl bg-indigo-50 flex items-center justify-center shadow-sm">
      <span class="text-lg font-bold text-indigo-400 uppercase tracking-widest">Decoder</span>
    </div>
    <div class="mt-4 text-[8px] font-bold text-indigo-300 uppercase tracking-widest">Generation</div>
  </div>

  <!-- Arrow (Connecting to Output) -->
  <div class="text-2xl text-emerald-200 font-bold transition-all duration-700" :class="$clicks >= 1 ? 'hidden' : ''">→</div>

  <!-- Output Text (Shrinking slightly but not dimming) -->
  <div class="flex flex-col items-center shrink-0 transition-all duration-700" :class="$clicks >= 1 ? 'scale-75' : ''">
    <div class="w-32 h-20 border-2 border-emerald-500 rounded-xl bg-emerald-50 flex items-center justify-center shadow-sm">
      <span class="text-lg font-bold text-emerald-500 uppercase tracking-widest">Text</span>
    </div>
    <div class="mt-4 text-[8px] font-bold text-emerald-400 uppercase tracking-widest">Output</div>
  </div>

</div>

<div class="mt-12 text-center text-gray-400 italic">
  <span v-if="$clicks == 0">"We need to go deeper into how the machine 'reads'..."</span>
  <span v-else class="text-emerald-600 font-bold">The Encoder first chops text into <strong>Tokens</strong>, then translates them into <strong>Embeddings</strong> (high-dimensional math vectors).</span>
</div>

<div v-click="1" class="hidden"></div>

---
transition: slide-up
---

# Setup

- **New Project** - Create a new project in your favorite language
- **Data** - Download tropico_species.txt

<br>
..
<br>
That's it

---
layout: two-cols
layoutClass: gap-10
clicks: 5
---

::left::

<div class="flex flex-col items-center justify-center h-full space-y-1 mt-4">
  <div class="w-full px-4 py-2 border-2 border-emerald-600 rounded bg-emerald-50 font-mono text-center shadow-sm opacity-80"
    :class="$clicks === 1 ? 'border-4 border-emerald-600 bg-emerald-50 font-bold shadow-md scale-105' : 'border-2 border-emerald-400 bg-emerald-50/50 shadow-sm'""
  >
  Tokens</div>
  <div class="text-xl text-gray-400">↓</div>
  <div class="w-full px-4 py-2 border-2 border-blue-600 rounded bg-blue-50 text-center shadow-sm opacity-80"
    :class="$clicks === 2 ? 'border-4 border-blue-600 bg-blue-50 font-bold shadow-md scale-105' : 'border-2 border-blue-400 bg-blue-50/50 shadow-sm'"
  >
    Embeddings</div>
  <div class="text-xl text-gray-400">↓</div>
  <!-- Highlighted State: Click 1 -->
  <div
    class="w-full px-4 py-2 rounded text-center transition-all duration-300"
    :class="$clicks === 3 ? 'border-4 border-orange-600 bg-orange-50 font-bold shadow-md scale-105' : 'border-2 border-orange-400 bg-orange-50/50 shadow-sm'"
  >
    Computation Graph
  </div>
  <div class="text-xl text-gray-400">↓</div>
  <!-- Highlighted State: Click 2 -->
  <div
    class="w-full px-4 py-2 rounded text-center transition-all duration-300"
    :class="$clicks === 4 ? 'border-4 border-purple-600 bg-purple-50 font-bold shadow-md scale-105' : 'border-2 border-purple-400 bg-purple-50/50 shadow-sm'"
  >
    Activation Function
  </div>
  <div class="text-xl text-gray-400">↓</div>
  <!-- Highlighted State: Click 3 -->
  <div
    class="w-full px-4 py-2 rounded text-center transition-all duration-300"
    :class="$clicks >= 5 ? 'border-4 border-red-600 bg-red-50 font-bold shadow-md scale-105' : 'border-2 border-red-400 bg-red-50/50 shadow-sm'"
  >
    Probability Distribution
  </div>
</div>

::right::

<div class="flex flex-col justify-center h-full">
  <h1 class="text-5xl font-bold mb-2">The Flow</h1>
  <div class="space-y-6">
    <div v-click="1" :class="$clicks === 1 ? 'opacity-100' : 'opacity-30 transition-opacity'">
      <p class="text-lg text-gray-600 leading-snug">
        <strong>Tokens</strong> map our unstructured text into semi-structured text blocks.
      </p>
    </div>
    <div v-click="2" :class="$clicks === 2 ? 'opacity-100' : 'opacity-30 transition-opacity'">
      <p class="text-lg text-gray-600 leading-snug">
        The <strong>Embedding</strong> is a vector of floats storing what we have learned about the meaning of the token.
      </p>
    </div>
    <div v-click="3" :class="$clicks === 3 ? 'opacity-100' : 'opacity-30 transition-opacity'">
      <p class="text-lg text-gray-600 leading-snug">
        The <strong>Computation Graph</strong> is a chain of math operations. To improve, we must calculate the gradient of every variable.
      </p>
    </div>
    <div v-click="4" :class="$clicks === 4 ? 'opacity-100' : 'opacity-30 transition-opacity'">
      <p class="text-lg text-gray-600 leading-snug">
        The <strong>Activation Function</strong> turns arbitrary numbers into structured values. It introduces non-linearity and transforms the data space.
      </p>
    </div>
    <div v-click="5" :class="$clicks >= 5 ? 'opacity-100' : 'opacity-30 transition-opacity'">
      <p class="text-lg text-gray-600 leading-snug">
        The <strong>Probability Distribution</strong> gives us the final prediction: A ranking of all possible tokens that might come next.
      </p>
    </div>
  </div>

</div>

---
layout: two-cols
layoutClass: gap-16 grid-cols-[2fr_3fr]
---

# Tokenizer

GPT is a matrix algebra system. It uses numbers, not strings. How do we get to those numbers?

- **Granularity**: We can tokenize by word, sub-word, or character.
- **Special Tokens**: We add markers like `BOS` to help the model identify boundaries.
- **Mapping**: Each unique token is assigned a unique integer ID.

::right::
<div class="flex flex-col h-full mt-4 pr-4 transition-all duration-800">
  <!-- Token Row -->
  <div class="token-container flex flex-nowrap justify-center items-center font-mono mt-10">
    <div v-click="2" class="token-box token-special bg-emerald-600 text-white border-emerald-700 mr-2">BOS</div>
    <div v-for="(char, i) in 'Agave'.split('')" :key="i" class="token-box transition-all duration-800 ease-in-out text-4xl" :class="$clicks >= 1 ? 'border-gray-400 bg-white shadow-sm mx-1' : 'border-transparent -mx-1.5'">{{ char }}</div>
    <div v-click="2" class="token-box token-special bg-emerald-600 text-white border-emerald-700 ml-2">BOS</div>
  </div>

  <!-- Arrow -->
  <div v-click="3" class="flex justify-center my-4">
    <div class="text-3xl text-blue-600 transition-all duration-800">↓</div>
  </div>

  <!-- ID Row -->
  <div v-click="3" class="token-container flex flex-nowrap justify-center items-center font-mono transition-all duration-800">
    <div class="token-box token-special border-emerald-700 bg-emerald-50 text-emerald-700 mr-2">56</div>
    <div v-for="(id, i) in [27, 7, 1, 22, 5]" :key="i" class="token-box border-gray-400 bg-gray-50 shadow-sm mx-1 text-2xl text-gray-600">{{ id }}</div>
    <div class="token-box token-special border-emerald-700 bg-emerald-50 text-emerald-700 ml-2">56</div>
  </div>

  <div class="mt-auto mb-16 text-center min-h-[6rem] relative transition-all duration-800">
    <!-- Steps 1 & 2 -->
    <div class="transition-all duration-800 absolute inset-x-0 top-0" :class="$clicks < 3 ? 'opacity-100' : 'opacity-0 pointer-events-none'">
      <v-click at="1"><p class="text-sm italic text-gray-500">Each character becomes a discrete unit...</p></v-click>
      <v-click at="2"><p class="text-sm italic text-emerald-600 font-bold">...and we wrap the sequence in special tokens.</p></v-click>
    </div>
    <!-- Step 3 -->
    <div class="transition-all duration-800 absolute inset-x-0 top-0" :class="$clicks == 3 ? 'opacity-100' : 'opacity-0 pointer-events-none'">
      <p class="text-sm italic text-blue-600 font-bold">Finally, we map each unique token to a unique integer ID.</p>
    </div>
    <!-- Step 4: Final Punch -->
    <div class="transition-all duration-800 absolute inset-x-0 top-0" :class="$clicks >= 4 ? 'opacity-100' : 'opacity-0 pointer-events-none'">
      <p class="text-lg font-bold text-orange-600">Now go ahead and implement your tokenizer.</p>
      <p class="text-sm italic text-gray-600">Run it on the whole dataset to make sure you're not missing tokens.</p>
    </div>
  </div>
</div>

<!-- Ensure click 4 exists -->
<div v-click="4" class="hidden"></div>

<style>
.token-box {
  min-width: 3rem;
  height: 3.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  border-width: 2px;
  border-style: solid;
  border-radius: 0.5rem;
  flex-shrink: 0;
}
.token-special {
  min-width: 2.5rem;
  height: 2.5rem;
  font-size: 0.875rem;
}
</style>



---
layout: default
class: p-0
---

<div class="grid grid-cols-[65fr_35fr] h-full w-full">
  <div class="p-12 pr-8">
    <h1 class="text-4xl mb-8">Implementation</h1>

```python {all|2-4|7-9|all}
# 1. Load and Shuffle
docs = [l.strip() for l in open('species.txt') if l.strip()]
random.shuffle(docs)

# 2. Build Vocabulary
uchars = sorted(set(''.join(docs)))
BOS = len(uchars)
vocab_size = len(uchars) + 1

print(f"Vocab size: {vocab_size}")
# Result: 55 unique chars + 1 BOS = 56
```

  </div>
  <div
    class="h-full w-full bg-cover bg-center"
    style="background-image: url('https://cover.sli.dev')"
  >
  </div>
</div>

---
layout: default
---

# Embeddings

Our model operates on a graph of `Value` nodes, but our Tokenizer gave us integer IDs. What now?

<div class="grid grid-cols-2 gap-10 mt-8">
  <div class="flex flex-col">
    <div class="flex items-center gap-4 mb-6">
      <div class="px-4 py-2 bg-gray-100 border border-gray-300 rounded font-mono text-xl">Token ID: 27</div>
      <div class="text-gray-400">→</div>
      <div class="text-sm text-gray-500 italic">"Lookup"</div>
    </div>
    <div class="border-2 border-blue-200 rounded-lg overflow-hidden shadow-sm">
      <div class="bg-blue-50 px-4 py-2 border-b border-blue-200 font-bold text-blue-800">Embedding Matrix</div>
      <div class="p-4 font-mono text-xs text-gray-600 space-y-1">
        <div class="flex justify-between text-gray-400"><span>Row 26</span><span>[Value, Value, Value, ...]</span></div>
        <div class="flex justify-between bg-blue-100 -mx-4 px-4 py-1 text-blue-900 font-bold border-y border-blue-200"><span>Row 27</span><span>[Value(-0.1), Value(0.5), ...]</span></div>
        <div class="flex justify-between text-gray-400"><span>Row 28</span><span>[Value, Value, Value, ...]</span></div>
        <div class="text-center text-gray-400 mt-2">...</div>
      </div>
    </div>
  </div>

  <div class="flex flex-col justify-center space-y-1">
    <div v-click="1">
      <h3 class="font-bold text-blue-600 text-xl">A Matrix of Random Parameters</h3>
      <p class="text-gray-600 mt-2 text-sm">
        An Embedding layer is just a 2D grid of <code>Value</code> nodes. <strong>It starts completely empty of knowledge:</strong> we initialize it with random noise.
      </p>
    </div>
    <div v-click="2">
      <h3 class="font-bold text-emerald-600 text-xl">The Lookup & Learning</h3>
      <p class="text-gray-600 mt-2 text-sm">
        The embedder simply does a lookup in the Embedding Matrix. The values will be optimized via backpropagation through our computation graph, until they learn to represent the actual "meaning" of the token.
      </p>
    </div>
  </div>
</div>

---
layout: default
---

# Positional Embeddings

The computation graph evaluates all inputs simultaneously. Without adjustment, `"Dog bites man"` and `"Man bites dog"` are identically processed

<div class="flex flex-col items-center mt-10 space-y-8">

  <div class="flex items-center gap-6 w-full justify-center">
    <div class="flex flex-col items-center w-40">
      <div class="px-4 py-2 bg-emerald-50 text-emerald-800 border-2 border-emerald-200 rounded font-mono shadow-sm">Token: 27</div>
      <div class="text-gray-400 my-2">↓ Lookup</div>
      <div class="px-2 py-2 border-2 border-gray-300 rounded font-mono text-sm bg-white">[ 0.5, -0.1 ]</div>
      <div class="text-[10px] text-gray-400 mt-1 uppercase font-bold tracking-wider text-center">Token Emb</div>
    </div>
    <div class="text-3xl text-gray-400 font-bold">+</div>
    <div class="flex flex-col items-center w-40">
      <div class="px-4 py-2 bg-blue-50 text-blue-800 border-2 border-blue-200 rounded font-mono shadow-sm">Pos: 0</div>
      <div class="text-gray-400 my-2">↓ Lookup</div>
      <div class="px-2 py-2 border-2 border-gray-300 rounded font-mono text-sm bg-white">[ -0.2, 0.8 ]</div>
      <div class="text-[10px] text-gray-400 mt-1 uppercase font-bold tracking-wider text-center">Positional Emb</div>
    </div>
    <div class="text-3xl text-gray-400 font-bold">=</div>
    <div class="flex flex-col items-center mt-[44px] w-44">
      <div class="px-2 py-3 border-4 border-orange-400 bg-orange-50 rounded-lg font-mono text-sm shadow-md font-bold text-orange-800 text-center">[ 0.3, 0.7 ]</div>
      <div class="text-[10px] text-orange-600 mt-2 uppercase font-bold tracking-wider text-center">The Starting Node</div>
    </div>
  </div>
    <div class="bg-gray-50 border border-gray-200 rounded-lg w-[85%] min-h-[150px] shadow-sm relative overflow-hidden">
      <!-- Step 0: The Basics -->
      <div
        class="transition-all duration-500 ease-in-out absolute inset-0 p-6 flex flex-col justify-center text-center"
        :class="$clicks === 0 ? 'opacity-100 translate-y-0' : 'opacity-0 -translate-y-4 pointer-events-none'">
        <span class="font-bold text-gray-700 block mb-1 uppercase tracking-wider text-[10px]">The Basics</span>
        <p class="text-gray-700 text-sm">
          We create a <strong>second</strong> embedding matrix just for (integer) positions. The token is then simply the position integer: 0, 1, 2. We lookup this vector as well and simply add them together. This combined vector enters the computation graph containing both <em>"What this is"</em> and <em>"Where this is."</em>
        </p>
      </div>
      <!-- Step 1: The Frame -->
      <div
        class="transition-all duration-500 ease-in-out absolute inset-0 p-6 flex flex-col justify-center text-center"
        :class="$clicks === 1 ? 'opacity-100 translate-y-0' : ($clicks < 1 ? 'opacity-0 translate-y-4 pointer-events-none' : 'opacity-0 -translate-y-4 pointer-events-none')">
        <span class="font-bold text-gray-700 block mb-1 uppercase tracking-wider text-[10px]">The Frame</span>
        <p class="text-gray-700 text-sm px-12">
          The "0" index is tied to the <strong>BOS (Beginning of Sentence)</strong> token. The model learns that position 0 is a special "entry point" where a new context begins.
        </p>
      </div>
      <!-- Step 2: The Relative Problem -->
      <div
        class="transition-all duration-500 ease-in-out absolute inset-0 p-6 flex flex-col justify-center text-center"
        :class="$clicks >= 2 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-4 pointer-events-none'">
        <span class="font-bold text-gray-700 block mb-1 uppercase tracking-wider text-[10px]">The Relativity Problem</span>
        <p class="text-gray-700 text-sm px-12">
          Absolute positions (0, 1, 2...) struggle with long-distance logic. Advanced models use <strong>Relative Encoding</strong>, focusing on <em>distance</em> rather than fixed slots (see ALiBi, RoPE).
        </p>
      </div>
    </div>
    <!-- Hidden clicks to drive the sequence -->
    <div v-click class="hidden"></div>
    <div v-click class="hidden"></div>
  </div>

---
clicks: 7
---

# The Objective

We want our model to learn one thing: **Predict the next token.**

<div class="flex flex-col items-center justify-center h-full mt-4 space-y-2">

  <!-- Step 1: Predict 'A' -->
  <div v-click="1" class="flex flex-col items-center scale-90 transform origin-top transition-all duration-500">
    <div class="token-container flex flex-nowrap justify-center items-center font-mono">
      <div class="token-box token-special bg-emerald-600 text-white border-emerald-700 mr-2" :class="$clicks >= 1 ? 'ring-4 ring-blue-400 shadow-lg' : ''">{{ $clicks >= 7 ? '56' : 'BOS' }}</div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl" :class="$clicks >= 2 ? 'ring-4 ring-orange-500 scale-110 opacity-100' : 'opacity-30'"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '27' : 'A' }}</span></div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl opacity-30">{{ $clicks >= 7 ? '7' : 'g' }}</div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl opacity-30">{{ $clicks >= 7 ? '1' : 'a' }}</div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl opacity-30">{{ $clicks >= 7 ? '22' : 'v' }}</div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl opacity-30">{{ $clicks >= 7 ? '5' : 'e' }}</div>
      <div class="token-box token-special bg-emerald-600 text-white border-emerald-700 ml-2 opacity-30">{{ $clicks >= 7 ? '56' : 'BOS' }}</div>
    </div>
    <div class="h-6 mt-4 text-xs font-bold uppercase tracking-widest flex justify-between w-[450px]">
      <span v-if="$clicks >= 1" class="text-blue-600">Input: [{{ $clicks >= 7 ? '56' : 'BOS' }}]</span>
      <span v-if="$clicks >= 2" class="text-orange-600">Target: {{ $clicks >= 7 ? '27' : 'A' }}</span>
    </div>
  </div>

  <!-- Step 2: Predict 'g' -->
  <div v-click="3" class="flex flex-col items-center scale-90 transform origin-top transition-all duration-500">
    <div class="token-container flex flex-nowrap justify-center items-center font-mono">
      <div class="token-box token-special bg-emerald-600 text-white border-emerald-700 mr-2" :class="$clicks >= 3 ? 'ring-4 ring-blue-400 opacity-100' : ''">{{ $clicks >= 7 ? '56' : 'BOS' }}</div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl" :class="$clicks >= 3 ? 'ring-4 ring-blue-400 opacity-100' : ''"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '27' : 'A' }}</span></div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl" :class="$clicks >= 4 ? 'ring-4 ring-orange-500 scale-110 opacity-100' : 'opacity-30'"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '7' : 'g' }}</span></div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl opacity-30"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '1' : 'a' }}</span></div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl opacity-30"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '22' : 'v' }}</span></div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl opacity-30"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '5' : 'e' }}</span></div>
      <div class="token-box token-special bg-emerald-600 text-white border-emerald-700 ml-2 opacity-30">{{ $clicks >= 7 ? '56' : 'BOS' }}</div>
    </div>
    <div class="h-6 mt-4 text-xs font-bold uppercase tracking-widest flex justify-between w-[450px]">
      <span v-if="$clicks >= 3" class="text-blue-600">Input: [{{ $clicks >= 7 ? '56, 27' : 'BOS, A' }}]</span>
      <span v-if="$clicks >= 4" class="text-orange-600">Target: {{ $clicks >= 7 ? '7' : 'g' }}</span>
    </div>
  </div>

  <!-- Step 3: Predict 'a' -->
  <div v-click="5" class="flex flex-col items-center scale-90 transform origin-top transition-all duration-500">
    <div class="token-container flex flex-nowrap justify-center items-center font-mono">
      <div class="token-box token-special bg-emerald-600 text-white border-emerald-700 mr-2" :class="$clicks >= 5 ? 'ring-4 ring-blue-400 opacity-100' : ''">{{ $clicks >= 7 ? '56' : 'BOS' }}</div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl" :class="$clicks >= 5 ? 'ring-4 ring-blue-400 opacity-100' : ''"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '27' : 'A' }}</span></div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl" :class="$clicks >= 5 ? 'ring-4 ring-blue-400 opacity-100' : ''"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '7' : 'g' }}</span></div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl" :class="$clicks >= 6 ? 'ring-4 ring-orange-500 scale-110 opacity-100' : 'opacity-30'"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '1' : 'a' }}</span></div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl opacity-30"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '22' : 'v' }}</span></div>
      <div class="token-box border-gray-400 bg-white shadow-sm mx-1 text-4xl opacity-30"><span :class="$clicks >= 7 ? 'text-3xl' : ''">{{ $clicks >= 7 ? '5' : 'e' }}</span></div>
      <div class="token-box token-special bg-emerald-600 text-white border-emerald-700 ml-2 opacity-30">{{ $clicks >= 7 ? '56' : 'BOS' }}</div>
    </div>
    <div class="h-6 mt-4 text-xs font-bold uppercase tracking-widest flex justify-between w-[450px]">
      <span v-if="$clicks >= 5" class="text-blue-600">Input: [{{ $clicks >= 7 ? '56, 27, 7' : 'BOS, A, g' }}]</span>
      <span v-if="$clicks >= 6" class="text-orange-600">Target: {{ $clicks >= 7 ? '1' : 'a' }}</span>
    </div>
  </div>

</div>

<div class="h-0 overflow-hidden">
  <v-click at="2" />
  <v-click at="4" />
  <v-click at="6" />
  <v-click at="7" />
</div>

<style>
.token-box {
  min-width: 3.5rem;
  height: 4rem;
  display: flex;
  align-items: center;
  justify-content: center;
  border-width: 2px;
  border-style: solid;
  border-radius: 0.5rem;
  flex-shrink: 0;
  transition: all 0.4s ease;
}
.token-special {
  min-width: 3rem;
  height: 3rem;
  font-size: 0.875rem;
}
</style>

---
layout: default
---

# The Output: Probability Distribution

The model doesn't return a single character. It returns a "ranking" of all possible characters.

<div class="grid grid-cols-2 gap-10 mt-10">
  <div>
    <h3 class="font-mono text-emerald-700 mb-4">Input: "Agav"</h3>
    <p class="text-gray-600">The model outputs a vector of 56 numbers (one for each token in our vocab).</p>
    <div class="mt-6 p-4 bg-gray-50 rounded border border-gray-200 font-mono text-sm">
      <div class="flex justify-between border-b border-gray-200 py-1">
        <span>Token 'e'</span><span class="font-bold text-emerald-600">0.82 (82%)</span>
      </div>
      <div class="flex justify-between border-b border-gray-200 py-1 text-gray-400">
        <span>Token 'a'</span><span>0.05 (5%)</span>
      </div>
      <div class="flex justify-between border-b border-gray-200 py-1 text-gray-400">
        <span>Token 'i'</span><span>0.03 (3%)</span>
      </div>
      <div class="flex justify-between py-1 text-gray-300">
        <span>...others</span><span>0.10 (10%)</span>
      </div>
    </div>
  </div>

  <div class="flex flex-col justify-center">
    <div v-click>
      <h3 class="font-bold text-orange-600">Why a distribution?</h3>
      <ul class="mt-2 space-y-2 text-sm text-gray-600">
        <li><strong>Sampling:</strong> We can pick the 2nd or 3rd choice to get creative (Temperature).</li>
        <li><strong>Certainty:</strong> The model can tell us how "sure" it is.</li>
        <li><strong>Loss:</strong> We compare this distribution to the "correct" one (where the target token has 1.0) to calculate our error.</li>
      </ul>
    </div>
  </div>
</div>

---
layout: two-cols
layoutClass: gap-12
---

::left::

# Activation Function

<div class="flex flex-col space-y-4 mt-8">
  <div class="p-4 border-2 border-gray-200 rounded-lg bg-gray-50">
    <h3 class="text-lg font-bold text-gray-700 mb-2">Raw Logits</h3>
    <div class="font-mono text-sm text-gray-600 space-y-1">
      <div class="flex justify-between"><span>Token 'e':</span><span>4.2</span></div>
      <div class="flex justify-between"><span>Token 'a':</span><span>-1.5</span></div>
      <div class="flex justify-between"><span>Token 'i':</span><span>0.3</span></div>
      <div class="flex justify-between text-gray-400"><span>...others:</span><span>...</span></div>
    </div>
  </div>
  <div v-click="1" class="flex flex-col space-y-4">
    <div class="text-center text-xl text-purple-600 font-bold">↓ Softmax ↓</div>
    <div class="p-4 border-2 border-purple-200 rounded-lg bg-purple-50 shadow-sm">
      <h3 class="text-lg font-bold text-purple-700 mb-2">Probabilities</h3>
      <div class="font-mono text-sm text-gray-600 space-y-1">
        <div class="flex justify-between"><span>Token 'e':</span><span class="text-emerald-600 font-bold">0.82</span></div>
        <div class="flex justify-between"><span>Token 'a':</span><span>0.05</span></div>
        <div class="flex justify-between"><span>Token 'i':</span><span>0.03</span></div>
        <div class="flex justify-between text-gray-400"><span>...others:</span><span>0.10</span></div>
      </div>
    </div>
  </div>
</div>

::right::

<div class="flex flex-col justify-center h-full space-y-4 mt-2">
  <div>
    <p class="text-gray-600 mt-2">
      To get a probability distribution, we need our raw numbers (logits) to follow two rules:
    </p>
    <ul class="list-decimal pl-6 mt-2 text-gray-600 font-semibold">
      <li>They must all be positive.</li>
      <li>They must sum to 1.0 (100%).</li>
    </ul>
  </div>

  <div v-click="1">
    <h3 class="font-bold text-purple-600 text-xl">The Softmax Function</h3>
    <p class="text-gray-600 mt-2 text-sm leading-relaxed">
      Softmax is our final <strong>Activation Function</strong>. It takes the arbitrary raw output scores of our computation graph and transforms them into a valid probability distribution.
    </p>
  </div>
  <div v-click="2">
    <h3 class="font-bold text-orange-600 text-xl">Why?</h3>
      <ul class="list-decimal pl-6 mt-2 text-gray-600">
        <li>Nonlinearity</li>
        <li>Stationarity</li>
        <li>Differentiability</li>
      </ul>
  </div>
</div>

<!--
Nonlinearity: If all operations are linear, stacking them is useless (they collapse into a single linear operation). We need nonlinear functions to learn complex representations.
Stationarity: The output remains stable across iterations. It maps arbitrary outputs to a fixed 0-to-1 range.
Differentiability: To use backpropagation, we need to calculate smooth gradients. Functions like Softmax have mathematical derivatives that let our network learn.
-->

---
layout: default
---

# The Flow: Slightly Zoomed in

Before we dive deeper, what are the individual parts of the computational graph?

<div class="flex flex-col items-center justify-center mt-2 w-full h-[250px] relative px-4">

  <!-- The Main Horizontal Line -->
  <div class="absolute left-10 right-10 top-[60%] h-1 bg-gradient-to-r from-gray-200 via-emerald-400 via-orange-400 via-purple-400 to-gray-200 z-0"></div>

  <!-- Residual Skip Connection Highlight (Click 1) -->
  <svg v-click="1" class="absolute inset-0 w-full h-full pointer-events-none z-10" viewBox="0 0 800 250">
    <path
      d="M 170 157 V 230 H 410 V 154 M 410 230 H 580 V 157"
      fill="none"
      stroke="#3b82f6"
      stroke-width="3"
      stroke-dasharray="8 6"
      stroke-linejoin="round"
      stroke-linecap="round"
      class="opacity-60"
    />
    <!-- Flow arrows along the horizontal segments -->
    <polygon points="290,225 305,230 290,235" fill="#3b82f6" fill-opacity="0.6" />
    <polygon points="495,225 510,230 495,235" fill="#3b82f6" fill-opacity="0.6" />
    <text x="400" y="245" text-anchor="middle" class="text-[10px] fill-blue-500 font-bold uppercase tracking-widest">residual connection</text>
  </svg>

  <div class="flex justify-between items-end w-full z-20">
    <!-- Step 1: Embeddings -->
    <div class="flex flex-col items-center w-1/4">
      <div class="font-mono text-xl font-bold bg-white px-2 py-1 rounded shadow-sm border border-gray-200 mb-8">x</div>
      <div class="px-3 py-3 bg-emerald-50 border-2 border-emerald-200 text-emerald-800 rounded shadow-sm text-center">
        <span class="font-bold block text-sm">1. Embeddings</span>
        <div class="text-[10px] mt-1 leading-tight">Initial vector is<br>created from tokens</div>
      </div>
    </div>
    <!-- Step 2: Attention -->
    <div class="flex flex-col items-center w-1/4 relative">
      <!-- Residual Loop -->
      <div class="w-8 h-8 rounded-full bg-orange-100 border-2 border-orange-400 flex items-center justify-center font-bold text-orange-600 z-20 mb-4 bg-white">+</div>
      <div class="px-3 py-3 bg-orange-50 border-2 border-orange-300 text-orange-800 rounded-lg shadow-md text-center relative z-10">
        <span class="font-bold block text-sm">2. Attention</span>
        <div class="text-[10px] mt-1 leading-tight">Embeddings 'communicate'.<br>Results <strong>added</strong> back to <code>x</code></div>
        <div class="font-mono text-[9px] mt-2 text-orange-600 bg-white px-1 py-0.5 rounded border border-orange-100">x = x + att(x)</div>
      </div>
    </div>
    <!-- Step 3: MLP -->
    <div class="flex flex-col items-center w-1/4 relative">
      <!-- Residual Loop -->
      <div class="w-8 h-8 rounded-full bg-purple-100 border-2 border-purple-400 flex items-center justify-center font-bold text-purple-600 z-20 mb-4 bg-white">+</div>
      <div class="px-3 py-3 bg-purple-50 border-2 border-purple-300 text-purple-800 rounded-lg shadow-md text-center relative z-10">
        <span class="font-bold block text-sm">3. MLP</span>
        <div class="text-[10px] mt-1 leading-tight">Individual processing.<br><strong>added</strong> back to <code>x</code></div>
        <div class="font-mono text-[9px] mt-2 text-purple-600 bg-white px-1 py-0.5 rounded border border-purple-100">x = x + mlp(x)</div>
      </div>
    </div>
    <!-- Step 4: Classifier -->
    <div class="flex flex-col items-center w-1/4">
      <div class="font-mono text-xl font-bold bg-white px-2 py-1 rounded shadow-sm border border-gray-200 mb-8">x</div>
      <div class="px-3 py-3 bg-red-50 border-2 border-red-200 text-red-800 rounded shadow-sm text-center">
        <span class="font-bold block text-sm">4. Classifier</span>
        <div class="text-[10px] mt-1 leading-tight">Finished vector is<br>converted to probabilities</div>
      </div>
    </div>

  </div>
</div>

<div class="mt-12 text-center text-sm text-gray-500 italic">
  (Note: At each step, <code>x</code> is stabilized using RMSNorm).
</div>

---
layout: two-cols
layoutClass: gap-12
---

# The `Value` Node

```python
class Value:
  def __init__(self, data, children, local_grads):
      self.data = data           # The actual value
      self.grad = 0              # dLoss / dNode
      self._children = children  # Pointers to inputs
      self._local_grads = local_grads # Local partials
```
  <div class="w-full pt-16"></div>
  <div class="w-full px-12 mt-4">
    <v-click>
      <div class="flex items-center h-2 gap-3 mb-1">
        <div class="w-2.5 h-2.5 rounded-full bg-emerald-500 shrink-0"></div>
        <p class="text-sm m-0"><strong>Data:</strong> State during the <em>forward pass</em>.</p>
      </div>
    </v-click>
    <v-click>
      <div class="flex items-center h-2 gap-3 mb-1 mt-8">
        <div class="w-2.5 h-2.5 rounded-full bg-orange-500 shrink-0"></div>
        <p class="text-sm m-0"><strong>Grad:</strong> Accumulated Gradient from the <em>backward pass</em>.</p>
      </div>
    </v-click>
    <v-click>
      <div class="flex items-center h-2 gap-3 mt-8">
        <div class="w-2.5 h-2.5 rounded-full bg-gray-400 shrink-0"></div>
        <p class="text-sm m-0"><strong>Children:</strong> Input nodes in the graph.</p>
      </div>
    </v-click>
  </div>

::right::

<div class="flex flex-col items-center justify-center h-full">
  <div class="text-center mb-4 relative">
    <p class="text-sm font-bold text-gray-400 uppercase tracking-widest mb-10">Anatomy</p>
    <div class="absolute left-[-60px] top-[60%] flex items-center">
      <div class="w-12 h-0.5 border-t-2 border-dashed border-gray-400"></div>
      <div class="w-0 h-0 border-y-4 border-y-transparent border-l-8 border-l-gray-400"></div>
      <span class="absolute -top-6 left-0 text-[10px] font-bold text-gray-400 uppercase tracking-tighter">_children</span>
    </div>
    <div class="relative w-56 h-56 rounded-full border-4 border-gray-300 shadow-xl overflow-hidden flex flex-col">
      <div class="h-1/2 bg-emerald-50 flex items-center justify-center border-b-2 border-gray-300">
        <div class="text-2xl font-mono font-bold text-emerald-800">data</div>
      </div>
      <div class="h-1/2 bg-orange-50 flex items-center justify-center">
        <div class="text-2xl font-mono font-bold text-orange-600">grad</div>
      </div>
      <div class="absolute left-0 top-1/2 -translate-y-1/2 bg-gray-300 w-3 h-10 rounded-r-lg"></div>
    </div>
  </div>

</div>


---
layout: two-cols
layoutClass: gap-12
---

# Building the Graph

How do these nodes connect? By overriding Python's math operators, every calculation builds a chain.

```python {all|2-4|6-10|all}
# When we add two Value nodes: a + b
def __add__(self, other):
    out = Value(self.data + other.data,
                children=(self, other))

    # We also define how the gradient flows back!
    def _backward():
        self.grad += 1.0 * out.grad
        other.grad += 1.0 * out.grad
    out._backward = _backward

    return out
```

::right::

<div class="flex flex-col justify-center h-full mt-4">
  <div v-click="1">
    <h3 class="text-xl font-bold text-emerald-600 mb-2">1. Forward Pass (Data)</h3>
    <p class="text-gray-600 mb-6 text-sm">
      When we calculate <code>c = a + b</code>, a new <code>Value</code> node <code>c</code> is created. It stores the result and remembers that <code>a</code> and <code>b</code> were its <code>_children</code>.
    </p>
  </div>

  <div v-click="2">
    <h3 class="text-xl font-bold text-orange-600 mb-2">2. Backward Pass (Grad)</h3>
    <p class="text-gray-600 mb-2 text-sm">
      Later, when we know how much <code>c</code> affected the final error (its <code>grad</code>), we run <code>_backward()</code>.
    </p>
    <p class="text-gray-600 text-sm">
      Using the chain rule, <code>c</code> passes its gradient backwards to its inputs <code>a</code> and <code>b</code>.
    </p>
  </div>
</div>

---
layout: default
clicks: 6
---

# A Tiny Graph in Action

Let's trace both passes for `L = a * b + a`.

<div class="relative w-full h-[350px] mt-8">
  <svg class="absolute inset-0 w-full h-full" style="z-index: 0;">
    <defs>
      <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
        <path d="M 0 0 L 10 5 L 0 10 z" fill="#9ca3af" />
      </marker>
      <marker id="arrow-grad" viewBox="0 0 10 10" refX="1" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
        <path d="M 10 0 L 0 5 L 10 10 z" fill="#f97316" />
      </marker>
    </defs>
    <!-- a to c -->
    <line x1="180" y1="100" x2="380" y2="180" stroke="#9ca3af" stroke-width="3" marker-end="url(#arrow)" />
    <!-- b to c -->
    <line x1="180" y1="260" x2="380" y2="180" stroke="#9ca3af" stroke-width="3" marker-end="url(#arrow)" />
    <!-- c to L -->
    <line x1="480" y1="180" x2="650" y2="180" stroke="#9ca3af" stroke-width="3" marker-end="url(#arrow)" />
    <!-- a to L -->
    <path d="M 180 80 Q 400 20 650 160" fill="none" stroke="#9ca3af" stroke-width="3" marker-end="url(#arrow)" />
    <!-- Gradient paths -->
    <g :class="$clicks >= 4 ? 'opacity-100' : 'opacity-0'" class="transition-opacity duration-500">
      <line x1="650" y1="190" x2="480" y2="190" stroke="#f97316" stroke-dasharray="5,5" stroke-width="3" marker-start="url(#arrow-grad)" />
      <path d="M 180 90 Q 400 30 650 170" fill="none" stroke="#f97316" stroke-dasharray="5,5" stroke-width="3" marker-start="url(#arrow-grad)" />
    </g>
    <g :class="$clicks >= 5 ? 'opacity-100' : 'opacity-0'" class="transition-opacity duration-500">
      <line x1="380" y1="190" x2="180" y2="110" stroke="#f97316" stroke-dasharray="5,5" stroke-width="3" marker-start="url(#arrow-grad)" />
      <line x1="380" y1="190" x2="180" y2="270" stroke="#f97316" stroke-dasharray="5,5" stroke-width="3" marker-start="url(#arrow-grad)" />
    </g>
  </svg>

  <!-- Node A -->
  <div class="absolute left-[80px] top-[60px] w-28 bg-white border-2 border-gray-300 rounded-lg shadow-md z-10 text-center overflow-hidden">
    <div class="bg-gray-100 font-bold py-1 border-b border-gray-300">a</div>
    <div class="py-1 flex justify-between px-3 text-sm" :class="$clicks >= 1 ? 'text-emerald-600 font-bold bg-emerald-50' : 'text-gray-300'"><span>d:</span><span>2.0</span></div>
    <div class="py-1 flex justify-between px-3 text-sm border-t border-gray-100" :class="$clicks >= 6 ? 'text-orange-600 font-bold bg-orange-50' : ($clicks >= 4 ? 'text-orange-400 font-bold' : 'text-gray-300')"><span>g:</span><span>{{ $clicks >= 6 ? '-2.0' : ($clicks >= 4 ? '1.0' : '0.0') }}</span></div>
  </div>

  <!-- Node B -->
  <div class="absolute left-[80px] top-[220px] w-28 bg-white border-2 border-gray-300 rounded-lg shadow-md z-10 text-center overflow-hidden">
    <div class="bg-gray-100 font-bold py-1 border-b border-gray-300">b</div>
    <div class="py-1 flex justify-between px-3 text-sm" :class="$clicks >= 1 ? 'text-emerald-600 font-bold bg-emerald-50' : 'text-gray-300'"><span>d:</span><span>-3.0</span></div>
    <div class="py-1 flex justify-between px-3 text-sm border-t border-gray-100" :class="$clicks >= 5 ? 'text-orange-600 font-bold bg-orange-50' : 'text-gray-300'"><span>g:</span><span>{{ $clicks >= 5 ? '2.0' : '0.0' }}</span></div>
  </div>

  <!-- Node C -->
  <div class="absolute left-[380px] top-[140px] w-28 bg-white border-2 border-gray-300 rounded-lg shadow-md z-10 text-center overflow-hidden">
    <div class="bg-gray-100 font-bold py-1 border-b border-gray-300">c <span class="font-normal text-xs text-gray-500">(a*b)</span></div>
    <div class="py-1 flex justify-between px-3 text-sm" :class="$clicks >= 2 ? 'text-emerald-600 font-bold bg-emerald-50' : 'text-gray-300'"><span>d:</span><span>-6.0</span></div>
    <div class="py-1 flex justify-between px-3 text-sm border-t border-gray-100" :class="$clicks >= 4 ? 'text-orange-600 font-bold bg-orange-50' : 'text-gray-300'"><span>g:</span><span>{{ $clicks >= 4 ? '1.0' : '0.0' }}</span></div>
  </div>

  <!-- Node L -->
  <div class="absolute left-[650px] top-[140px] w-28 bg-white border-2 border-gray-300 rounded-lg shadow-md z-10 text-center overflow-hidden">
    <div class="bg-red-50 text-red-800 font-bold py-1 border-b border-gray-300">L <span class="font-normal text-xs opacity-70">(c+a)</span></div>
    <div class="py-1 flex justify-between px-3 text-sm" :class="$clicks >= 3 ? 'text-emerald-600 font-bold bg-emerald-50' : 'text-gray-300'"><span>d:</span><span>-4.0</span></div>
    <div class="py-1 flex justify-between px-3 text-sm border-t border-gray-100" :class="$clicks >= 3 ? 'text-orange-600 font-bold bg-orange-50' : 'text-gray-300'"><span>g:</span><span>{{ $clicks >= 3 ? '1.0' : '0.0' }}</span></div>
  </div>
</div>

<div class="mt-4 text-center h-12">
  <p v-if="$clicks === 1" class="text-emerald-600"><strong>1. Inputs:</strong> We start with raw values for a and b.</p>
  <p v-if="$clicks === 2" class="text-emerald-600"><strong>2. Forward Pass:</strong> Multiply a and b to get c.</p>
  <p v-if="$clicks === 3" class="text-emerald-600"><strong>3. Output & Base Grad:</strong> Add a to c to get L. We set L.grad = 1.0.</p>
  <p v-if="$clicks === 4" class="text-orange-600"><strong>4. Backward (Add):</strong> Addition distributes the gradient equally. c.grad = 1.0, a.grad += 1.0.</p>
  <p v-if="$clicks === 5" class="text-orange-600"><strong>5. Backward (Mul):</strong> Multiply swaps data values for grads. b.grad = a.data &times; c.grad = 2.0.</p>
  <p v-if="$clicks === 6" class="text-orange-600"><strong>6. Accumulation:</strong> a.grad receives b.data &times; c.grad = -3.0. Total a.grad = 1.0 + (-3.0) = -2.0!</p>
</div>

---
layout: two-cols
layoutClass: gap-12
---

# Orchestrating Backprop
How does the network actually run the backward pass on thousands of nodes in the right order?

```python {all|2-10|11|12-14|all}
def backward(self):
    topo = []
    visited = set()
    def build_topo(v):
        if v not in visited:
            visited.add(v)
            for child in v._children:
                build_topo(child)
            topo.append(v)
    build_topo(self)

    self.grad = 1.0 # The base gradient
    for v in reversed(topo):
        # Accumulate grads to children
        for child, local_grad in zip(v._children, v._local_grads):
            child.grad += local_grad * v.grad
```

::right::

<div class="flex flex-col justify-center h-full mt-4 space-y-6">
  <div v-click="1">
    <h3 class="text-xl font-bold text-emerald-600 mb-2">1. Topological Sort</h3>
    <p class="text-gray-600 text-sm">
      We start at the final output (Loss) and recursively visit all nodes. A node is only appended to <code>topo</code> <em>after</em> all its inputs (children) have been processed.
    </p>
  </div>

  <div v-click="2">
    <h3 class="text-xl font-bold text-blue-600 mb-2">2. The Seed</h3>
    <p class="text-gray-600 text-sm">
      We set the gradient of the Loss to <code>1.0</code>. (A unit change in the loss affects the loss by exactly 1.0).
    </p>
  </div>

  <div v-click="3">
    <h3 class="text-xl font-bold text-orange-600 mb-2">3. Reverse Iteration</h3>
    <p class="text-gray-600 text-sm">
      By iterating through <code>topo</code> in reverse, we guarantee that we only push gradients to a node's children <em>after</em> that node's own gradient has been fully accumulated. The chain rule perfectly cascades backward!
    </p>
  </div>
</div>

---
layout: default
---

# Stepping through Topo

Let's look at our graph: `L = c + a` where `c = a * b`.<br>
What does `build_topo(L)` actually produce using Depth-First Search?

<div class="mt-8 flex justify-center w-full relative">
  <!-- Topo List Container -->
  <div class="flex items-center gap-4 border-2 border-gray-300 p-6 rounded-xl bg-gray-50 shadow-inner relative z-10">
    <div class="absolute -top-3 left-4 bg-gray-50 px-2 text-sm font-bold text-gray-500 uppercase">topo = [</div>
    <!-- Node a (First from DFS diving into c -> a) -->
    <div class="flex flex-col items-center w-24 transition-opacity duration-500" :class="$clicks >= 1 ? 'opacity-100' : 'opacity-0'">
      <div class="w-16 h-16 rounded-full border-4 flex items-center justify-center text-2xl font-bold shadow-md transition-all duration-500" :class="$clicks >= 8 ? 'border-orange-500 bg-orange-100 text-orange-700 ring-4 ring-orange-200' : 'border-gray-400 bg-white text-gray-600'">a</div>
      <div class="mt-2 text-xs font-mono text-gray-400">leaf</div>
      <div class="h-6 mt-1 flex items-center text-xs font-bold text-orange-600 transition-opacity duration-300" :class="$clicks >= 8 ? 'opacity-100' : 'opacity-0'">g: -2.0</div>
    </div>
    <div class="text-gray-300 text-xl font-bold transition-opacity duration-500" :class="$clicks >= 2 ? 'opacity-100' : 'opacity-0'">,</div>
    <!-- Node b (Second from DFS returning to c and diving into b) -->
    <div class="flex flex-col items-center w-24 transition-opacity duration-500" :class="$clicks >= 2 ? 'opacity-100' : 'opacity-0'">
      <div class="w-16 h-16 rounded-full border-4 flex items-center justify-center text-2xl font-bold shadow-md transition-all duration-500" :class="$clicks >= 7 ? 'border-orange-500 bg-orange-100 text-orange-700 ring-4 ring-orange-200' : 'border-gray-400 bg-white text-gray-600'">b</div>
      <div class="mt-2 text-xs font-mono text-gray-400">leaf</div>
      <div class="h-6 mt-1 flex items-center text-xs font-bold text-orange-600 transition-opacity duration-300" :class="$clicks >= 7 ? 'opacity-100' : 'opacity-0'">g: 2.0</div>
    </div>
    <div class="text-gray-300 text-xl font-bold transition-opacity duration-500" :class="$clicks >= 3 ? 'opacity-100' : 'opacity-0'">,</div>
    <!-- Node c (Third, done with its children) -->
    <div class="flex flex-col items-center w-24 transition-opacity duration-500" :class="$clicks >= 3 ? 'opacity-100' : 'opacity-0'">
      <div class="w-16 h-16 rounded-full border-4 flex items-center justify-center text-2xl font-bold shadow-md transition-all duration-500" :class="$clicks >= 6 ? 'border-orange-500 bg-orange-100 text-orange-700 ring-4 ring-orange-200' : 'border-gray-400 bg-white text-gray-600'">c</div>
      <div class="mt-2 text-xs font-mono text-gray-400">c = a*b</div>
      <div class="h-6 mt-1 flex items-center text-xs font-bold text-orange-600 transition-opacity duration-300" :class="$clicks >= 6 ? 'opacity-100' : 'opacity-0'">g: 1.0</div>
    </div>
    <div class="text-gray-300 text-xl font-bold transition-opacity duration-500" :class="$clicks >= 4 ? 'opacity-100' : 'opacity-0'">,</div>
    <!-- Node L (Last, finished everything) -->
    <div class="flex flex-col items-center w-24 transition-opacity duration-500" :class="$clicks >= 4 ? 'opacity-100' : 'opacity-0'">
      <div class="w-16 h-16 rounded-full border-4 flex items-center justify-center text-2xl font-bold shadow-md transition-all duration-500" :class="$clicks >= 5 ? 'border-red-500 bg-red-100 text-red-800 ring-4 ring-red-200' : 'border-gray-400 bg-white text-gray-600'">L</div>
      <div class="mt-2 text-xs font-mono text-gray-400">L = c+a</div>
      <div class="h-6 mt-1 flex items-center text-xs font-bold text-red-600 transition-opacity duration-300" :class="$clicks >= 5 ? 'opacity-100' : 'opacity-0'">g: 1.0</div>
    </div>
    <div class="absolute -bottom-3 right-4 bg-gray-50 px-2 text-sm font-bold text-gray-500 uppercase">]</div>
  </div>

  <!-- Reversal Arrow and Label -->
  <div class="absolute -bottom-16 left-1/2 transform -translate-x-1/2 w-3/4 max-w-lg h-12 flex items-center justify-center transition-all duration-700" :class="$clicks >= 5 ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-4'">
    <div class="text-orange-500 font-bold uppercase tracking-widest mr-4 text-sm flex-shrink-0 animate-pulse">reversed(topo)</div>
    <div class="h-1 w-full bg-gradient-to-l from-orange-500 to-orange-200 rounded-full relative">
      <div class="absolute -left-2 -top-[10px] w-0 h-0 border-y-[12px] border-y-transparent border-r-[16px] border-r-orange-500"></div>
    </div>
  </div>
</div>

<div class="mt-24 h-24 text-center relative w-full">
  <!-- Building Phase -->
  <div v-click="1" class="absolute left-0 w-full px-12 transition-all duration-300" :class="$clicks === 1 ? 'opacity-100' : 'opacity-0'">
    <p class="text-xl text-gray-700"><code>build_topo(L)</code> checks L's children: <strong>c</strong> and <strong>a</strong>. It dives into <strong>c</strong>. It checks <strong>c</strong>'s children: <strong>a</strong> and <strong>b</strong>. It dives into <strong>a</strong>. It's a leaf! Append <strong>a</strong>.</p>
  </div>
  <div v-click="2" class="absolute left-0 w-full px-12 transition-all duration-300" :class="$clicks === 2 ? 'opacity-100' : 'opacity-0'">
    <p class="text-xl text-gray-700">It returns to <strong>c</strong>, then dives into the other child: <strong>b</strong>. It's a leaf! Append <strong>b</strong>.</p>
  </div>
  <div v-click="3" class="absolute left-0 w-full px-12 transition-all duration-300" :class="$clicks === 3 ? 'opacity-100' : 'opacity-0'">
    <p class="text-xl text-gray-700">It returns to <strong>c</strong>. Both children (a, b) are now processed. Append <strong>c</strong>.</p>
  </div>
  <div v-click="4" class="absolute left-0 w-full px-12 transition-all duration-300" :class="$clicks === 4 ? 'opacity-100' : 'opacity-0'">
    <p class="text-xl text-gray-700">It returns to <strong>L</strong>. It already visited <strong>a</strong> earlier. All children of L are processed. Append <strong>L</strong>.</p>
  </div>

  <!-- Backprop Phase -->
  <div v-click="5" class="absolute left-0 w-full px-12 transition-all duration-300" :class="$clicks === 5 ? 'opacity-100' : 'opacity-0'">
    <p class="text-xl text-emerald-700 font-bold mt-[-20px] mb-2 uppercase tracking-widest text-sm">Backward Pass</p>
    <p class="text-xl text-gray-700">Now we iterate backward. We pull <strong>L</strong>. Its initial gradient is seeded to <code>1.0</code>. It passes gradients back via addition to <strong>c</strong> and <strong>a</strong>.</p>
  </div>
  <div v-click="6" class="absolute left-0 w-full px-12 transition-all duration-300" :class="$clicks === 6 ? 'opacity-100' : 'opacity-0'">
    <p class="text-xl text-emerald-700 font-bold mt-[-20px] mb-2 uppercase tracking-widest text-sm">Backward Pass</p>
    <p class="text-xl text-gray-700">Next, we pull <strong>c</strong>. It has received its <code>1.0</code> grad from L. It passes gradients back via multiplication to <strong>a</strong> and <strong>b</strong>.</p>
  </div>
  <div v-click="7" class="absolute left-0 w-full px-12 transition-all duration-300" :class="$clicks === 7 ? 'opacity-100' : 'opacity-0'">
    <p class="text-xl text-emerald-700 font-bold mt-[-20px] mb-2 uppercase tracking-widest text-sm">Backward Pass</p>
    <p class="text-xl text-gray-700">Next, we pull <strong>b</strong>. It's a leaf node. It has received its gradient from c. It has no children, so it does nothing.</p>
  </div>
  <div v-click="8" class="absolute left-0 w-full px-12 transition-all duration-300" :class="$clicks >= 8 ? 'opacity-100' : 'opacity-0'">
    <p class="text-xl text-emerald-700 font-bold mt-[-20px] mb-2 uppercase tracking-widest text-sm">Backward Pass</p>
    <p class="text-xl text-gray-700">Finally, we pull <strong>a</strong>. It is also a leaf node. Because of the list order, we guarantee <strong>a</strong> has finished receiving gradients from <em>both</em> <strong>L</strong> and <strong>c</strong> before we visit it.</p>
  </div>
</div>



---
layout: default
---

# Attention: Q, K, V
How do tokens learn to "talk" to each other? The current token is projected into three vectors.

<div class="flex flex-col mt-4">
  <!-- Horizontal Top Section -->
  <div class="flex justify-center items-center space-x-6 mb-6">
    <div class="bg-gray-50 border border-gray-200 p-3 rounded-lg shadow-sm text-center">
      <div class="font-mono text-2xl font-bold text-gray-800">x</div>
      <div class="text-[10px] text-gray-500 uppercase tracking-widest mt-1">Starting Node</div>
    </div>
    <div class="text-gray-400 font-bold text-2xl">→</div>
    <div class="flex space-x-4">
      <div class="bg-blue-50 border border-blue-200 p-3 rounded-lg shadow-sm text-center w-36">
        <div class="font-mono font-bold text-blue-800 text-lg">Q <span class="text-gray-500 font-normal text-sm">= x · W_q</span></div>
        <div class="text-[10px] text-blue-600 uppercase tracking-widest mt-1">Query Matrix</div>
      </div>
      <div class="bg-emerald-50 border border-emerald-200 p-3 rounded-lg shadow-sm text-center w-36">
        <div class="font-mono font-bold text-emerald-800 text-lg">K <span class="text-gray-500 font-normal text-sm">= x · W_k</span></div>
        <div class="text-[10px] text-emerald-600 uppercase tracking-widest mt-1">Key Matrix</div>
      </div>
      <div class="bg-purple-50 border border-purple-200 p-3 rounded-lg shadow-sm text-center w-36">
        <div class="font-mono font-bold text-purple-800 text-lg">V <span class="text-gray-500 font-normal text-sm">= x · W_v</span></div>
        <div class="text-[10px] text-purple-600 uppercase tracking-widest mt-1">Value Matrix</div>
      </div>
    </div>
  </div>

  <!-- Bottom Explanatory Section -->
  <div class="flex flex-col justify-center space-y-4 px-8">
    <div v-click="1">
      <h3 class="font-bold text-orange-600 text-lg">Alignment via Backpropagation</h3>
      <p class="text-gray-600 text-sm leading-relaxed">
        Imagine the training data contains: <code>"The fuzzy dog barks"</code>.
      </p>
      <ol class="list-disc pl-6 space-y-1 text-[14px] text-gray-600 leading-tight">
        <li>Random noise makes <code>"fuzzy"</code>'s Query <strong>point in the same direction</strong> as <code>"dog"</code>'s Key.</li>
        <li>Their <strong>dot product</strong> results in a large positive number—a high Attention Score.</li>
        <li></li>
        <li>This score acts as a volume knob, pulling out <code>"dog"</code>'s <strong>Value</strong> to inform the next prediction.</li>
        <li>If the resulting guess (e.g. <code>"barks"</code>) matches reality (Ground Truth), Loss is low</li>
        <li>Backpropagation reweighs the specific weights that made that Query and Key align.</li>
      </ol>
  </div>
</div>
</div>

<!--
        The "Roles" of Query (What am I looking for?), Key (What do I contain?), and Value (What do I communicate?) emerge organically through
        the structure of the multiplication, guided to 'truthy' values via backpropagation.
-->
