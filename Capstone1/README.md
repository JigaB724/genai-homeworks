# Capstone Project 1 — Voice Brief → Image

**Student:** Zhiger Bazarov · **Course:** GenAI Basics · **Deadline:** 15 November 2026

**Colab notebook:** [Capstone1_Bazarov_Zhiger](https://colab.research.google.com/drive/12pjOdZ0VXiSCxw2UIEj0LVDC_x_NXLxR?usp=sharing) · saved copy with outputs: [`Capstone1_Bazarov_Zhiger.ipynb`](/Capstone1_Jiger_Bazarov.ipynb)

## Pipeline
My voice (English) → **Whisper-small** (speech to text) → **Gemini 2.5 Flash** (text to image prompt) → **SD-Turbo** (image)

Settings: style `anime`, format `landscape 16:9`. No fallback was used.

## Voice brief
> I need a cover image for a winter travel blog. The main thing is a small wooden house in the snowy mountains with warm light in the windows. The mood is cozy and quiet, like an anime illustration. It is evening, with a dark blue sky and a few stars. At the top, the words "Winter Trip".

## Results
| Step | Result |
| :--- | :--- |
| 2. Transcript | 3 word mistakes: "winter trailer book" → "winter travel blog", "warm white" → "warm light", "the world's winter trip" → "the words Winter Trip". Fixed in `corrected_transcript`. |
| 3. Prompt | 33 words, all brief elements kept, caption in quotes; the purpose ("blog cover") and any composition/camera angle are missing. |
| 4. Image | Three cabins instead of one house, no "Winter Trip" caption, sky full of stars, realistic painting rather than anime. |
| 5. Edit | "Make the house red, keep everything else the same": all three cabins turned red, red tint leaked into clouds and mountains, windows lost their glow; composition kept (same seed). |
| 6. Comparison | The rewritten prompt gave cleaner cabins and warmer windows; the raw text added a moon and a broken floating roof. |

## Conclusions
Most mistakes were at the image step: SD-Turbo is a small one-step model, weak at text, counting objects and following style words. Whisper's three word errors were fixed by hand before they reached the prompt. Gemini kept every important element but dropped the purpose of the image. Next time: a shorter, slower recording, "only one house" stated explicitly, and the caption added later in an image editor.
