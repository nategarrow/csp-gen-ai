# CSP Generator SvelteKit App Plan

## Notes
- Project uses SvelteKit@latest and Tailwind v4
- One-page app: left column (inputs), right column (CSP output)
- Inputs for popular CSP directives (connect-src, frame-src, script-src, script-src-elem, script-src-attr, style-src, style-src-elem, etc)
- Enter key adds URL to directive list and clears input
- Generate CSP button compiles and displays policy in <pre> on right
- Clear button resets all fields
- DRY and KISS principles; reusable components in src/lib/components if needed
- Add Bits UI dropdown in left column header to select directive
- When a directive is selected, left column shows only that directive and its URLs
- Bits UI Select is now properly integrated, using type="single" and string values for correct binding
- User created a custom theme in src/app.css; use these colors in the UI
- All UI components now use the custom theme colors
- Add Bits UI Dialog/modal for importing a CSP string and parsing it into directive fields
- Add "Add Directive" button at the bottom of the input list
- Show inputs for label and directive name with a checkbox-style submit
- Add custom directive to state and directiveConfigs, and render it in the UI
- Persist directives and custom directives to localStorage with 1 week expiration
- Add ability to remove custom directives from the UI
- Add "Reset All" button to restore starting directive list and clear all values

## Task List
- [x] Set up SvelteKit project with Tailwind v4
- [x] Create main page layout with two columns
- [x] Refactor repeated logic into reusable components if needed
- [x] Implement input fields for each CSP directive
- [x] Add logic to handle Enter key for adding URLs
- [x] Store directive lists in state
- [x] Implement "Generate CSP" button and logic
- [x] Display generated CSP in <pre> block
- [x] Implement "Clear" button to reset inputs and output
- [x] Install Bits UI and configure
- [x] Add dropdown to left column header for directive selection
- [x] Implement conditional rendering: show only selected directive and its URLs if dropdown is active
- [x] Update all UI to use new theme colors
- [x] Add "Import CSP" button that opens Bits UI Dialog/modal
- [x] Implement textarea for pasting CSP string
- [x] Parse CSP string and populate directives on import
- [x] Implement custom directive addition feature
- [x] Persist directives and custom directives to localStorage with 1 week expiration
- [x] Add ability to remove custom directives from the UI
- [x] Add "Reset All" button to restore starting directive list and clear all values

## Current Goal
Ready for further feedback or new features