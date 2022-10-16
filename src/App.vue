<template>
  <div class="card">
    <div class="select">
  <div class="selectWrapper">
    <select v-model="selected" class="selectNative js-selectNative" aria-labelledby="jobLabel">
      <option v-for="option in foods" :key="option.value">
        {{option.text}}
      </option>
    </select>


    <!-- Hide the custom select from AT (e.g. SR) using aria-hidden -->
    <div class="selectCustom js-selectCustom" aria-hidden="true">
      <div class="selectCustom-trigger">Choose Industry</div>
      <div class="selectCustom-options">
        <div class="selectCustom-option" data-value="8">E-commerce</div>
        <div class="selectCustom-option" data-value="6">Financial Services</div>
        <div class="selectCustom-option" data-value="14">Marketing</div>
        <div class="selectCustom-option" data-value="31">Software</div>
        <div class="selectCustom-option" data-value="9">Travel</div>
        <div class="selectCustom-option" data-value="37">Addiction Treatment</div>
        <div class="selectCustom-option" data-value="13">Automotive</div>
        <div class="selectCustom-option" data-value="40">Beauty</div>
        <div class="selectCustom-option" data-value="15">Computer Services</div>
        <div class="selectCustom-option" data-value="10">Consulting</div>
      </div>
    </div>
  </div>
</div>
  </div>

</template>

<script>
export default {
  data() {
    return {
      selected: '',
      foods: [
        {
          text: "Choose Industry",
          value: 0,
        },
        {
          text: "E-commerce",
          value: 8,
        },
        {
          text: "Financial Services",
          value: 6,
        },
        {
          text: "Marketing",
          value: 14,
        },
        {
          text: "Software",
          value: 31,
        },
        {
          text: "Travel",
          value: 9,
        },
        {
          text: "Addiction Treatment",
          value: 37,
        },
        {
          text: "Automotive",
          value: 13,
        },
        {
          text: "Beauty",
          value: 40,
        },
        {
          text: "Computer Services",
          value: 15,
        },
        {
          text: "Consulting",
          value: 10,
        },
      ],
      form: {
        name: null,
        food: null,
      },
    };
  },
  mounted() {
    this.enableCustomSelect();
  },
  methods: {
    enableCustomSelect() {
/* Features to make the selectCustom work for mouse users.

- Toggle custom select visibility when clicking the "box"
- Update custom select value when clicking in a option
- Navigate through options when using keyboard up/down
- Pressing Enter or Space selects the current hovered option
- Close the select when clicking outside of it
- Sync both selects values when selecting a option. (native or custom)

*/

const elSelectNative = document.getElementsByClassName("js-selectNative")[0];
const elSelectCustom = document.getElementsByClassName("js-selectCustom")[0];
const elSelectCustomBox = elSelectCustom.children[0];
const elSelectCustomOpts = elSelectCustom.children[1];
const customOptsList = Array.from(elSelectCustomOpts.children);
const optionsCount = customOptsList.length;

let optionChecked = "";
let optionHoveredIndex = -1;

// Toggle custom select visibility when clicking the box
elSelectCustomBox.addEventListener("click", (e) => {
  const isClosed = !elSelectCustom.classList.contains("isActive");

  if (isClosed) {
    openSelectCustom();
  } else {
    closeSelectCustom();
  }
});

function openSelectCustom() {
  elSelectCustom.classList.add("isActive");
  // Remove aria-hidden in case this was opened by a user
  // who uses AT (e.g. Screen Reader) and a mouse at the same time.
  elSelectCustom.setAttribute("aria-hidden", false);

  if (optionChecked) {
    const optionCheckedIndex = customOptsList.findIndex(
      (el) => el.getAttribute("data-value") === optionChecked
    );
    updateCustomSelectHovered(optionCheckedIndex);
  }

  // Add related event listeners
  document.addEventListener("click", watchClickOutside);
  document.addEventListener("keydown", supportKeyboardNavigation);
}

function closeSelectCustom() {
  elSelectCustom.classList.remove("isActive");

  elSelectCustom.setAttribute("aria-hidden", true);

  updateCustomSelectHovered(-1);

  // Remove related event listeners
  document.removeEventListener("click", watchClickOutside);
  document.removeEventListener("keydown", supportKeyboardNavigation);
}

function updateCustomSelectHovered(newIndex) {
  const prevOption = elSelectCustomOpts.children[optionHoveredIndex];
  const option = elSelectCustomOpts.children[newIndex];

  if (prevOption) {
    prevOption.classList.remove("isHover");
  }
  if (option) {
    option.classList.add("isHover");
  }

  optionHoveredIndex = newIndex;
}

const updateCustomSelectChecked = (value, text) => {
  this.selected = {value, text};
  const prevValue = optionChecked;

  const elPrevOption = elSelectCustomOpts.querySelector(
    `[data-value="${prevValue}"`
  );
  const elOption = elSelectCustomOpts.querySelector(`[data-value="${value}"`);

  if (elPrevOption) {
    elPrevOption.classList.remove("isActive");
  }

  if (elOption) {
    elOption.classList.add("isActive");
  }

  elSelectCustomBox.textContent = text;
  optionChecked = value;
}

function watchClickOutside(e) {
  const didClickedOutside = !elSelectCustom.contains(event.target);
  if (didClickedOutside) {
    closeSelectCustom();
  }
}

function supportKeyboardNavigation(e) {
  // press down -> go next
  if (event.keyCode === 40 && optionHoveredIndex < optionsCount - 1) {
    let index = optionHoveredIndex;
    e.preventDefault(); // prevent page scrolling
    updateCustomSelectHovered(optionHoveredIndex + 1);
  }

  // press up -> go previous
  if (event.keyCode === 38 && optionHoveredIndex > 0) {
    e.preventDefault(); // prevent page scrolling
    updateCustomSelectHovered(optionHoveredIndex - 1);
  }

  // press Enter or space -> select the option
  if (event.keyCode === 13 || event.keyCode === 32) {
    e.preventDefault();

    const option = elSelectCustomOpts.children[optionHoveredIndex];
    const value = option && option.getAttribute("data-value");

    if (value) {
      elSelectNative.value = value;
      updateCustomSelectChecked(value, option.textContent);
    }
    closeSelectCustom();
  }

  // press ESC -> close selectCustom
  if (event.keyCode === 27) {
    closeSelectCustom();
  }
}

// Update selectCustom value when selectNative is changed.
elSelectNative.addEventListener("change", (e) => {
  const value = e.target.value;
  const elRespectiveCustomOption = elSelectCustomOpts.querySelectorAll(
    `[data-value="${value}"]`
  )[0];

  updateCustomSelectChecked(value, elRespectiveCustomOption.textContent);
});

// Update selectCustom value when an option is clicked or hovered
customOptsList.forEach(function (elOption, index) {
  elOption.addEventListener("click", (e) => {
    const value = e.target.getAttribute("data-value");

    // Sync native select to have the same value
    elSelectNative.value = value;
    updateCustomSelectChecked(value, e.target.textContent);
    closeSelectCustom();
  });

  elOption.addEventListener("mouseenter", (e) => {
    updateCustomSelectHovered(index);
  });

  // TODO: Toggle these event listeners based on selectCustom visibility
});
    }
  }
};
</script>

<style scoped>
/*the container must be positioned relative:*/
/* Both native and custom selects must have the same width/height. */

@import '../public/fonts/SofiaProMedium/style.css';

.selectNative,
.selectCustom {
  position: relative;
  width: 22rem;
  height: 4rem;
}

/* Make sure the custom select does not mess with the layout */
.selectCustom {
  position: absolute;
  top: 0;
  left: 0;
  display: none;
}

 /* This media query detects devices where the primary */
/* input mechanism can hover over elements. (e.g. computers with a mouse) */
@media (hover: hover) {
   /* Since we are using a mouse, it's safe to show the custom select. */
  .selectCustom {
    display: block;
  }

  /* In a computer using keyboard? Then let's hide back the custom select */
  /* while the native one is focused: */
  .selectNative:focus + .selectCustom {
    display: none;
  }
}

/* Add the focus states too, They matter, always! */
.selectNative:focus,
.selectCustom.isActive .selectCustom-trigger {
  outline: none;
  box-shadow: #e47e5f 0 0 0px 2px;
}


/* Rest of the styles to create the custom select.
/* Just make sure the native and the custom have a similar "box" (the trigger). */


.select {
  position: relative;
  margin: 0 auto;
}

.selectWrapper {
  position: relative;
}

.selectNative,
.selectCustom-trigger {
  display: flex;
  align-items: center;
  line-height: 20px;
  background-color: #fff;
  border: 1px solid #6f6f6f;
  border-radius: 30px;;
  font-family: 'Sofia Pro';
  font-style: normal;
  font-weight: 500;
  font-size: 14px;
}

.selectNative {
  background-repeat: no-repeat;
  background-position-x: 100%;
  background-position-y: 0.8rem;
  padding: 0rem 0.8rem;
}

.selectCustom-trigger {
  position: relative;
  width: 100%;
  height: 100%;
  background-color: #fff;
  padding: 0.8rem 0.8rem;
  cursor: pointer;
}

.selectCustom-trigger::after {
  content: "";
  position: absolute;
  right: 9%;
  display: inline-block;
  transform: rotate(45deg);
  height: 12px;
  width: 12px;
  border-bottom: 3px solid #000;
  border-right: 3px solid #000;
}

.selectCustom-trigger:hover {
  border-color: #8c00ff;
}

.selectCustom-options {
  position: absolute;
  top: calc(3.8rem + 0.8rem);
  left: 0;
  width: 100%;
  border-radius: 0.4rem;
  background-color: #fff;
  box-shadow: 3px 4px 8px 4px #4d4d4d3d;
  z-index: 1;
  display: none;
}

.selectCustom.isActive .selectCustom-options {
  display: block;
}

.selectCustom-option {
  position: relative;
  padding: 0.8rem;
  padding-left: 2.5rem;
  font-family: 'Sofia Pro';
  font-weight: 500;
}

.selectCustom-option.isHover,
.selectCustom-option:hover {
  background-color: #e47e5f; /* contrast AA */
  color: white;
  cursor: default;
}

.selectCustom-option:nth-child(5)::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 5%;
  width: 90%;
  margin: 0 auto;
  border-bottom: 1px solid #d3d3d3;
}



/* ----- Theme styles ----- */

html {
  font-size: 62.5%;
}
body {
  background: #f8f3ef;
  font-family: Arial, Helvetica, sans-serif;
  box-sizing: border-box;
  color: #343434;
  line-height: 1.5;
  font-size: 1.6rem;
  min-height: 120vh; /* using arrow keys in the select, does not scroll the page */
}

body * {
  box-sizing: inherit;
}

strong {
  font-weight: 600;
}


.card {
  position: relative;
  margin: 1rem auto;
  max-width: calc(100% - 2rem);
  width: 40rem;
  background: #e47e5f66;
  padding: 3rem;
  box-shadow: 0.2rem 0.2rem #e9e1f8;
}

</style>