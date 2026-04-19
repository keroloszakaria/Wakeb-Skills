# Design Token Reference

NEVER hardcode colors. Always use CSS variables from the theme token pipeline:

```
Figma → design/primitives.json → design/semantics.json → tokens.json → CSS Variables
```

---

## Text Colors

```css
color: rgba(var(--v-theme-text-text-primary-900), 1)     /* Primary text */
color: rgba(var(--v-theme-text-text-secondary-700), 1)    /* Secondary */
color: rgba(var(--v-theme-text-text-tertiary-600), 1)     /* Muted */
color: rgba(var(--v-theme-text-text-quaternary-500), 1)   /* Quaternary */
color: rgba(var(--v-theme-text-text-placeholder), 1)      /* Placeholder */
color: rgba(var(--v-theme-text-text-white), 1)            /* White on dark */
```

## Background Colors

```css
background: rgba(var(--v-theme-background-bg-primary), 1)           /* Main bg */
background: rgba(var(--v-theme-background-bg-secondary), 1)         /* Card/section */
background: rgba(var(--v-theme-background-bg-brand-solid), 1)       /* Brand solid */
background: rgba(var(--v-theme-background-bg-brand-solid-hover), 1) /* Brand hover */
background: rgba(var(--v-theme-background-bg-error-solid), 1)       /* Error bg */
background: rgba(var(--v-theme-surface_hover), 1)                   /* Hover state */
background: rgba(var(--v-theme-background-bg-active), 1)            /* Active state */
```

## Border Colors

```css
border-color: rgba(var(--v-theme-border-border-primary), 1)    /* Primary */
border-color: rgba(var(--v-theme-border-border-secondary), 1)  /* Secondary */
border-color: rgba(var(--v-theme-border_light), 1)             /* Light */
border-color: rgba(var(--v-theme-border_medium), 1)            /* Medium */
```

## Status Colors

```css
rgba(var(--v-theme-utility-success-utility-success-700), 1)  /* Success */
rgba(var(--v-theme-utility-error-utility-error-700), 1)      /* Error */
rgba(var(--v-theme-warning_600), 1)                          /* Warning */
rgba(var(--v-theme-info_600), 1)                             /* Info */
```

## Spacing & Sizing (Tailwind)

```
Spacing: p-2 p-3 p-4 p-6 p-8 gap-2 gap-4 gap-6
Radius:  rounded-lg (8px) | rounded-xl (12px) | rounded-2xl (16px)
```

## Gradient CSS Variables

Used for decorative card borders and sidebar gradients:

```css
--gradient-purple    /* Purple gradient stop */
--gradient-blue      /* Blue gradient stop */
--gradient-black     /* Dark gradient stop */
```

Card gradient pattern:

```css
.card-gradient {
  padding: 2px;
  border-radius: 16px;
  background: linear-gradient(
    140deg,
    var(--gradient-purple),
    var(--gradient-blue),
    var(--gradient-black),
    var(--gradient-black),
    var(--gradient-black),
    var(--gradient-black),
    var(--gradient-black)
  );
}
.card-gradient > div {
  background: linear-gradient(
    240deg,
    var(--gradient-blue),
    var(--gradient-purple),
    var(--gradient-black)
  );
  border-radius: 16px;
}
```

## Overlay & Glassmorphism Patterns

Used for video overlays, stat bars, and floating panels:

```css
/* Dark glass overlay (video stats, stream info) */
background: rgba(0, 0, 0, 0.6);
backdrop-filter: blur(8px);

/* Light glass overlay */
background: rgba(0, 0, 0, 0.4);
backdrop-filter: blur(4px);

/* Hover card */
background: rgba(var(--v-theme-surface_hover), 0.5);

/* Active item highlight */
background: rgba(var(--v-theme-background-bg-brand-solid), 1);
color: rgba(var(--v-theme-dark_white), 1);
```

## Container Classes

```css
.tableContainer    /* Themed card-like section — background + border-radius */
```
