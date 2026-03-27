# A3 Write-Up: World Development Explorer
**CSC316 — Interactive Visualization**
Abhigyan Srivastava | March 2025

---

## Rationale for Design Decisions

### Dataset
I chose a Gapminder-style global development dataset spanning 180 countries and 53 years (1970–2022), covering four dimensions: GDP per capita (PPP), life expectancy at birth, CO₂ emissions per capita, and population. I selected this dataset because it is inherently multi-dimensional and contains rich temporal structure — two properties that strongly motivate interactive visualization over static charts. The core question driving the project is: *How have wealth, health, and environmental impact co-evolved across the world's nations since 1970, and how much have these relationships changed?*

### Visual Encodings
The primary view is an animated scatter plot — a deliberate reference to Hans Rosling's famous Gapminder visualizations. Scatter plots are among the most effective encodings for revealing correlations between continuous variables; placing GDP vs. Life Expectancy on x/y axes immediately reveals the canonical "health-wealth" relationship. Bubble area encodes a third quantitative variable (defaulting to population), following Cleveland & McGill's principle that area is an acceptable pre-attentive channel for magnitude comparison. Color encodes geographic region (5 hues, carefully chosen to be distinct on dark backgrounds), enabling viewers to identify spatial clustering patterns without requiring a lookup. A deliberate dark editorial theme (charcoal background, warm amber accents) was chosen to increase contrast for small bubbles and reduce visual fatigue during temporal animation.

For the secondary views, I added (1) a brushable timeline showing regional average life expectancy trends over 1970–2022, and (2) a per-region KDE density plot for the Y-axis variable. The timeline solves a key problem with animated scatter plots: viewers cannot simultaneously perceive change in individual points *and* global trends. The timeline anchors the animation temporally and allows users to focus on any sub-interval via brushing. The KDE panel complements the scatter by revealing distributional shape — e.g., whether Africa's life expectancy is unimodal (homogeneous) or bimodal (diverging), which is invisible in a scatter.

### Interaction Techniques
I implemented five interaction categories: (1) **Axis selection** — users can assign any of the four metrics to X, Y, and bubble size, enabling exploration of all 24 permutations. (2) **Temporal animation** — a play/pause button advances the year at 120ms/frame, with a manual scrubber for precise control. Transitions use a 600ms ease-cubic-in-out interpolation, slow enough to allow users to track individual bubble trajectories. (3) **Region filtering** — toggling regions fades their bubbles and redraws the KDE, supporting focused regional comparison. (4) **Hover & click focus** — hovering dims all other countries (focus+context) and shows a detailed tooltip; clicking *pins* a country and shows an annotation card comparing its current state to its 1970 baseline. (5) **Timeline brushing** — a D3 brushX selection allows users to restrict their temporal range of interest, which feeds into the KDE recomputation.

I considered but rejected force-directed layouts and parallel coordinates. Force layouts would destroy the meaningful positional encoding. Parallel coordinates were considered for multi-metric comparison but deemed too cognitively demanding for an exploratory audience. I also considered geographic choropleth maps but rejected them because spatial proximity is not the interesting relationship — the wealth-health correlation transcends geography.

### Animation
Animated transitions serve two purposes: perceptual continuity (viewers can track whether their country of interest moves up-left or down-right between years) and engagement (the dramatic convergence of life expectancies after 1990 is viscerally compelling in motion). The staggered entry animation on first load communicates that bubbles are distinct objects, each with identity.

### Alternatives Considered
An early prototype used a log scale on both axes (producing a more linear GDP-lifeExp relationship), but this obscured the dramatic compression of rich countries at the top. I retained an optional log toggle for power users. I also experimented with trail lines (showing each bubble's path over the last 10 years) but found this added too much clutter when all 180 countries are shown simultaneously.

---

## Development Process

### Process Overview
I began with two days of exploratory data analysis in Python (pandas + matplotlib) to understand the structure of the data and identify the most visually compelling variable pairs. The GDP vs. Life Expectancy pairing was clearly the strongest: the correlation is high, the variance is meaningful, and it changes dramatically over time. I then sketched the layout on paper — a large scatter as the primary view, a timeline below for temporal brushing, and a density panel for distributional context.

Development proceeded in three phases. In the first phase (~8 hours), I implemented the core scatter plot with animated transitions, tooltip, and axis selectors. In the second phase (~5 hours), I added the timeline with D3 brushing and coordinated the year-scrubber across both views. In the third phase (~4 hours), I implemented the KDE panel, region filtering with coordinated updates, the pinned-country highlight card, and all visual polish (typography, dark theme, hover dimming).

### Use of LLMs
I used Claude to assist with specific D3 idioms — particularly the brush event handling (`d3.brushX`, `event.selection` in v7) and the KDE implementation (Epanechnikov kernel + bandwidth selection). I found this workflow comfortable: I wrote the overall architecture and visual logic myself, and used Claude to resolve API-specific questions that would otherwise require extended documentation reading. The most time-consuming aspects were: (a) getting smooth animated transitions with correct `key` functions on the data join, and (b) coordinating state updates across three separate SVG panels without React or a state management library. Both required careful manual reasoning that LLM assistance couldn't short-circuit.

Total development time: approximately **17 person-hours**.

### Insights the Visualization Reveals
- The gap between Sub-Saharan Africa and the rest of the world in life expectancy *narrowed dramatically* between 1970 and 2005, then briefly reversed during the HIV/AIDS crisis, then recovered after 2010.
- Asia's GDP per capita growth from 1990–2010 is the most visually dramatic trajectory in the dataset — China and South Korea follow near-identical arcs, racing up the x-axis.
- When switching to CO₂ vs. GDP, the Gulf states (UAE, Saudi Arabia) appear as extreme outliers: very high GDP, very high emissions, and relatively modest life expectancy — a distinctive development model.

### Acknowledgments
Dataset: World Bank Open Data (data.worldbank.org) and Gapminder Foundation (gapminder.org). Visualization inspired by Hans Rosling's original Gapminder animations and Mike Bostock's D3 scatter examples. D3.js v7 (Observable/ISC license). Fonts: Playfair Display and DM Mono via Google Fonts.