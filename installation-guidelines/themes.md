---
description: OpenAlgo Themes
---

# Themes

OpenAlgo features a sophisticated theming system powered by DaisyUI, offering 30 professionally designed themes that enhance your trading experience with beautiful, consistent, and accessible user interfaces.

<figure><img src="../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

### Overview

The OpenAlgo theming system provides:

* **30 Professional Themes**: Extensive color schemes from minimalist to vibrant
* **Mode-Specific Themes**: Visual distinction between Live and Analyze modes
* **Instant Switching**: Real-time theme changes without page reload
* **Persistent Preferences**: Your theme selection is saved across sessions
* **Responsive Design**: All themes are optimized for desktop, tablet, and mobile
* **Accessibility**: WCAG-compliant color contrasts and ARIA-friendly components

{% embed url="https://www.youtube.com/watch?v=ZXbwDik5NOI" %}

### Available Themes

OpenAlgo includes 30 carefully crafted themes to suit every preference:

#### Light Themes

* **light** - Clean, professional white interface
* **cupcake** - Soft pastels with warm tones
* **bumblebee** - Energetic yellow and black
* **emerald** - Fresh green with modern aesthetics
* **corporate** - Professional blue-gray palette
* **retro** - Nostalgic warm tones
* **valentine** - Romantic pink and red
* **garden** - Natural green and earthy colors
* **aqua** - Cool cyan and blue tones
* **lofi** - Muted, relaxed colors
* **pastel** - Gentle rainbow pastels
* **fantasy** - Whimsical purple and pink
* **wireframe** - Minimalist black and white
* **cmyk** - Vibrant print-inspired colors
* **autumn** - Warm orange and brown
* **acid** - Bright neon green and yellow
* **lemonade** - Refreshing yellow and white
* **winter** - Cool blues and whites
* **nord** - Scandinavian-inspired palette
* **sunset** - Warm orange to purple gradient

#### Dark Themes

* **dark** - Classic dark gray interface
* **synthwave** - Retro 80s neon aesthetics
* **cyberpunk** - Futuristic yellow and magenta
* **halloween** - Spooky orange and purple
* **luxury** - Elegant dark with gold accents
* **dracula** - _(Analyze Mode Only)_ Deep purple vampire theme
* **business** - Professional dark blue-gray
* **coffee** - Rich brown and cream
* **night** - Deep blue-black palette
* **dim** - Muted dark gray

### Theme Usage

#### Live Mode (Production Trading)

When trading with real money in **Live Mode**, you have access to **29 themes** (all except dracula).

**Quick Theme Toggle (Navbar)**

The navbar includes a convenient theme switcher that toggles between:

* **Light Theme** - Default bright interface
* **Dark Theme** - Reduced eye strain for extended trading sessions

Simply click the sun/moon icon in the navigation bar to instantly switch.

**Full Theme Selection (Profile Page)**

For access to all 29 themes:

1. Navigate to **Settings → Profile**
2. Click the **Theme Preferences** tab
3. Browse the complete theme gallery
4. Click any theme to apply instantly
5. Your selection is automatically saved

**Note**: The dracula theme is **not available** in Live Mode as it is exclusively reserved for Analyze Mode.

#### Analyze Mode (Sandbox Environment)

The **Analyze Mode** provides a risk-free testing environment for validating strategies and API requests without executing real trades.

**Fixed Dracula Theme**

When you enter Analyze Mode:

* **Auto-Switch**: Theme automatically changes to `dracula`
* **Visual Indicator**: The distinctive purple dracula theme clearly signals you're in a safe testing environment
* **Locked Theme**: Theme switcher is disabled to prevent confusion
* **Safety Feature**: This ensures you always know when you're in test mode vs. live trading

**Why a Fixed Theme?**

The locked dracula theme in Analyze Mode serves critical purposes:

1. **Visual Confirmation**: Instantly recognize you're in test mode
2. **Prevent Confusion**: Avoid accidentally thinking you're in Live Mode
3. **Consistent Experience**: All users see the same interface in documentation and support
4. **Safety First**: Clear distinction reduces risk of accidental live trading

**Returning to Live Mode**

When you exit Analyze Mode:

* Your **previous theme is automatically restored**
* Theme switching becomes available again
* All 29 Live Mode themes are accessible

### How to Change Themes

#### Method 1: Quick Toggle (Navbar)

1. Locate the theme switcher icon in the navigation bar (sun/moon icon)
2. Click to toggle between light and dark themes
3. Change applies instantly

**Best For**: Quick switching during different times of day

#### Method 2: Full Selection (Profile Page)

1. Click **Settings** in the navigation menu
2. Select **Profile** from the dropdown
3. Navigate to the **Theme Preferences** tab
4. Browse the theme gallery showing all available themes
5. Click any theme card to preview and apply
6. Current theme is highlighted with a primary ring
7. Preview the color palette (primary, secondary, accent, success, error)

**Best For**: Exploring all themes and finding your perfect match

#### Method 3: Programmatic (Advanced)

Developers can change themes programmatically using the theme manager:

```javascript
// Set a specific theme
window.themeManager.setTheme('cyberpunk');

// Get current theme
const currentTheme = window.themeManager.getCurrentTheme();

// Check if in Analyze Mode
const isAnalyze = window.themeManager.isAnalyzeMode();

// Restore previous theme (after Analyze Mode)
window.themeManager.restorePreviousTheme();
```

### Technical Details

#### Technology Stack

OpenAlgo's theming system is built on modern web technologies:

* **DaisyUI**: `5.1.27` - Component library providing the 30 themes
* **Tailwind CSS**: `4.1.14` - Utility-first CSS framework foundation
* **PostCSS**: `8.4.49` - CSS processing and optimization
* **Autoprefixer**: `10.4.20` - Cross-browser compatibility

#### Theme Implementation

Themes are implemented using pure CSS custom properties (CSS variables):

```html
<!-- Theme is applied via data attribute -->
<html data-theme="cyberpunk">
```

This approach provides:

* **Zero Runtime JavaScript**: Themes are pure CSS, no JS overhead
* **Instant Switching**: CSS variable changes are immediate
* **Better Performance**: No JavaScript recalculation on theme change
* **SSR Compatible**: Themes work with server-side rendering

#### Storage & Persistence

Theme preferences are stored in two locations:

1.  **localStorage**: Persists across browser sessions

    ```javascript
    localStorage.getItem('theme') // e.g., "dark"
    ```
2.  **sessionStorage**: Maintains consistency during navigation

    ```javascript
    sessionStorage.getItem('theme') // Current session theme
    ```

#### Cross-Tab Synchronization

Theme changes automatically sync across browser tabs:

* Open multiple OpenAlgo tabs
* Change theme in one tab
* All other tabs update instantly via storage events

#### Flash Prevention

OpenAlgo prevents theme "flash" on page load using an Immediately Invoked Function Expression (IIFE):

```javascript
(function() {
    const savedTheme = localStorage.getItem('theme') || 'light';
    document.documentElement.setAttribute('data-theme', savedTheme);
})();
```

This sets the theme before the page renders, eliminating white flashes when using dark themes.

### Why We Chose DaisyUI

OpenAlgo selected **DaisyUI** as the UI component library for several strategic reasons:

#### 1. Extensive Theme Collection

DaisyUI provides 30+ professionally designed themes out of the box, allowing traders to customize their trading experience without requiring custom CSS development.

#### 2. Component-Rich Library

As the **most popular component library for Tailwind CSS**, DaisyUI offers:

* Pre-built, production-ready components
* Consistent design language across the platform
* Reduced development time for new features

#### 3. Performance Optimized

* **Zero Runtime JavaScript**: Components are pure CSS
* **Smaller Bundle Size**: No heavy JavaScript libraries
* **Faster Page Loads**: Critical for real-time trading applications

#### 4. Accessibility First

All DaisyUI components follow accessibility best practices:

* ARIA-compliant markup
* Keyboard navigation support
* Screen reader friendly
* WCAG color contrast standards

#### 5. Tailwind Integration

Seamless integration with Tailwind CSS utilities:

* Use both DaisyUI components and Tailwind utilities
* Consistent spacing, sizing, and color systems
* Full customization capability

#### 6. Mobile Responsive

Every theme and component works flawlessly across devices:

* Desktop trading stations
* Tablet interfaces
* Mobile trading on the go

#### 7. Active Development

DaisyUI is actively maintained with regular updates, bug fixes, and new features, ensuring OpenAlgo stays modern and secure.

### Customizing Themes (Advanced)

#### Tailwind Configuration

OpenAlgo's Tailwind configuration is minimal and extensible:

```javascript
// tailwind.config.mjs
import daisyui from 'daisyui';

export default {
  content: [
    "./templates/**/*.html",
    "./static/**/*.js",
  ],
  theme: {
    extend: {}
  },
  plugins: [daisyui]
}
```

#### Adding Custom Themes

Developers can add custom themes by extending the DaisyUI configuration:

```javascript
// tailwind.config.mjs
export default {
  plugins: [daisyui],
  daisyui: {
    themes: [
      "light",
      "dark",
      // ... other themes
      {
        mytheme: {
          "primary": "#0000ff",
          "secondary": "#ff00ff",
          "accent": "#00ffff",
          "neutral": "#3d4451",
          "base-100": "#ffffff",
        },
      },
    ],
  },
}
```

#### Theme Color Palette

Each DaisyUI theme includes these semantic color tokens:

| Token          | Purpose            | Examples                      |
| -------------- | ------------------ | ----------------------------- |
| `primary`      | Main brand color   | Buttons, links, active states |
| `secondary`    | Supporting color   | Secondary buttons, accents    |
| `accent`       | Highlight color    | Badges, special notifications |
| `neutral`      | Neutral elements   | Borders, dividers, disabled   |
| `base-100`     | Background color   | Page background               |
| `base-200`     | Subtle background  | Cards, containers             |
| `base-300`     | Borders & dividers | Input borders, separators     |
| `base-content` | Text color         | Body text, labels             |
| `info`         | Informational      | Info alerts, messages         |
| `success`      | Success states     | Profit, completed orders      |
| `warning`      | Warning states     | Pending, caution messages     |
| `error`        | Error states       | Loss, failed orders           |

### Browser Compatibility

OpenAlgo themes work across all modern browsers:

* ✅ Chrome/Edge 90+
* ✅ Firefox 88+
* ✅ Safari 14+
* ✅ Opera 76+
* ✅ Mobile browsers (iOS Safari, Chrome Mobile)

**Note**: Internet Explorer is not supported as it lacks CSS custom property support.

### Troubleshooting

#### Theme Not Changing

If your theme doesn't change:

1. **Clear Browser Cache**: Hard reload with `Ctrl+Shift+R` (Windows/Linux) or `Cmd+Shift+R` (Mac)
2. **Check localStorage**: Open DevTools → Application → Local Storage → verify `theme` key exists
3. **Disable Browser Extensions**: Some extensions may interfere with theme switching
4. **Try Incognito Mode**: Rules out extension conflicts

#### Theme Flashing on Page Load

If you see a white flash before the theme loads:

1. Ensure JavaScript is enabled in your browser
2. Check that the theme IIFE in `theme.js` is loading before the page renders
3. Verify no browser extensions are blocking inline scripts

#### Analyze Mode Theme Won't Change

This is **expected behavior**. The dracula theme is locked in Analyze Mode for safety. To change themes:

1. Exit Analyze Mode
2. Return to Live Mode
3. Select your preferred theme
4. Theme will be restored when you exit Analyze Mode again

#### Custom Theme Not Appearing

If you've added a custom theme:

1. Verify `tailwind.config.mjs` syntax is correct
2. Rebuild CSS: `npm run build:css`
3. Add theme name to `themes` array in `static/js/theme.js`
4. Restart the application server

### Credits & Acknowledgments

#### DaisyUI

OpenAlgo extends deep gratitude to [**Pouya Saadeghi**](https://github.com/saadeghi) and the DaisyUI team for creating an exceptional component library.

* **Project**: [github.com/saadeghi/daisyui](https://github.com/saadeghi/daisyui)
* **License**: MIT
* **Version**: 5.1.27
* **Purpose**: Powers the entire OpenAlgo user interface with beautiful, accessible components and extensive theming capabilities

#### Tailwind CSS

Thanks to [**Tailwind Labs**](https://tailwindcss.com/) for the utility-first CSS framework that forms the foundation of our design system.

* **Project**: [github.com/tailwindlabs/tailwindcss](https://github.com/tailwindlabs/tailwindcss)
* **License**: MIT
* **Version**: 4.1.14
* **Purpose**: Provides the utility classes and build system for rapid, consistent UI development

#### Community

Special thanks to the OpenAlgo community for feedback on themes, accessibility improvements, and user experience suggestions that make our platform better for everyone.

### Resources

#### Official Documentation

* **DaisyUI Themes**: [daisyui.com/docs/themes](https://daisyui.com/docs/themes/)
* **DaisyUI Components**: [daisyui.com/components](https://daisyui.com/components/)
* **Tailwind CSS**: [tailwindcss.com/docs](https://tailwindcss.com/docs)

#### OpenAlgo Documentation

* **Getting Started**: [docs.openalgo.in/getting-started](https://docs.openalgo.in/getting-started)
