# UI Wireframing Patterns

Common screen layouts and component patterns for lo-fi HTML wireframes.

## Screen Types

### Dashboard

- Header with app name and user menu.
- Sidebar navigation with active state.
- Main area with summary cards (KPIs) in a row.
- Below cards: charts (placeholders) and recent activity lists.

### List / Index

- Header with search input and "Create new" button.
- Filter bar with dropdowns or tags.
- Data table or card grid showing items.
- Pagination controls at bottom.

### Detail / Show

- Back link at top.
- Page title and action buttons (Edit, Delete) aligned horizontally.
- Two-column layout: main details left, metadata / related items right.
- Sections with subheadings for grouped fields.

### Form / Create-Edit

- Page title indicating action (e.g. "Create Item").
- Form fields stacked vertically with labels above inputs.
- Required field indicators (e.g. an asterisk in the label).
- Action bar at bottom: Primary submit button left, cancel link right.
- Inline validation placeholders if relevant.

### Login / Auth

- Centred card on blank or tinted background.
- App logo or name at top of card.
- Username and password fields.
- Primary submit button full width.
- Footer links: "Forgot password?", "Sign up".

### Modal / Dialog

- Overlay with semi-transparent backdrop.
- Centred panel with header, body content, and footer actions.
- Close affordance (X or Cancel button).

## Component Patterns

### Navigation

- Horizontal top nav: 4-7 items, active item underlined or bold.
- Vertical sidebar: icon + label, collapsible groups.
- Breadcrumbs: horizontal path above page title.

### Cards

- Container with light border and subtle background.
- Optional header with title and actions (e.g. dropdown).
- Body contains text, placeholders, or lists.

### Placeholders

- Use `.wf-placeholder` for images, charts, maps, and media.
- Label the placeholder so reviewers know what will live there.
- Vary height to suggest aspect ratio or content volume.

### Empty States

- Centred placeholder icon or illustration box.
- Title and short description.
- Primary call-to-action button.

### Loading States

- Skeleton bars in place of text (`div` with `.wf-placeholder` and short height).
- Spinners are usually too detailed for lo-fi wireframes - prefer skeleton blocks.

## Wireframing Best Practices

1. **One file per screen.** Each screen is a self-contained HTML file.
2. **Link screens together.** Navigation elements should link to sibling HTML files so stakeholders can click through the prototype.
3. **Use realistic content volume.** Avoid "lorem ipsum" - use plausible placeholder data that reflects real length and structure.
4. **Stay grayscale.** No brand colours, no photographs. The goal is to evaluate layout and flow, not visual design.
5. **Annotate when necessary.** Use `.wf-annotation` for notes about behaviour, validation rules, or dynamic content.
6. **Name files by screen purpose.** e.g. `dashboard.html`, `user-profile.html`, `settings-notifications.html`.
7. **Responsive hints.** If mobile matters, create separate files (e.g. `dashboard-mobile.html`) or add a narrow wrapper width to show mobile layout.
