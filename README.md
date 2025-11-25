<style> /* hide the Color Adjustments container and any color-pickers inside it */ [name="colorAdjustments"], [name="colorAdjustments"] .color-picker, .action-box-textColor, .action-box-titleColor, .action-box-backgroundColor, .action__custom-element .color-picker { display: none !important; visibility: hidden !important; pointer-events: none !important; aria-hidden: true; } </style>
 <script> (function(){ var s = document.createElement('script'); var h = document.querySelector('head') || document.body; s.src = 'https://acsbapp.com/apps/app/dist/js/app.js'; s.async = true; s.onload = function(){ acsbJS.init(); }; h.appendChild(s); })(); </script> 
 <script>
  (function() { const selectors = [ '[name="colorAdjustments"]', // the container you pasted '.action-box-textColor', '.action-box-titleColor', '.action-box-backgroundColor', '.color-picker', '.action__custom-element .color-picker' ];

function hideEl(el) { if (!el || el.__hiddenByScript) return; // prefer removal to prevent handler from running on it; comment out remove() if you want only hide try { el.remove(); // fallback if remove fails: } catch (e) { el.style.display = 'none'; el.style.visibility = 'hidden'; el.setAttribute('aria-hidden', 'true'); } el.__hiddenByScript = true; }

function hideMatches(root = document) { try { selectors.forEach(sel => { Array.from((root.querySelectorAll ? root.querySelectorAll(sel) : [])).forEach(hideEl); });
                                               // extra: hide nodes whose name attribute is colorAdjustments even if nested differently
  Array.from((root.querySelectorAll ? root.querySelectorAll('[name="colorAdjustments"]') : [])).forEach(hideEl);
} catch (err) {
  console.error('hideMatches error', err);
}
 </script>
