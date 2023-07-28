<script lang="ts">
  import { afterUpdate, onMount } from 'svelte'
  export let recipients
  let myEl

  function getTextWidth(text, font) {
    // re-use canvas object for better performance
    const canvas =
      getTextWidth['canvas'] ||
      (getTextWidth['canvas'] = document.createElement('canvas'))
    const context = canvas.getContext('2d')
    context.font = font
    const metrics = context.measureText(text)
    return metrics.width
  }

  function getCssStyle(element, prop) {
    return window.getComputedStyle(element, null).getPropertyValue(prop)
  }

  function getCanvasFontSize(el = document.body) {
    const fontWeight = getCssStyle(el, 'font-weight') || 'normal'
    const fontSize = getCssStyle(el, 'font-size') || '16px'
    const fontFamily = getCssStyle(el, 'font-family') || 'Aerial'

    return `${fontWeight} ${fontSize} ${fontFamily}`
  }
  function truncateString() {
    let string = recipients[0]
    const fontSizeString = getCanvasFontSize(myEl)
    const containerWidth = myEl.offsetWidth
    console.log(myEl.offsetWidth)
    for (let i = 1; i < recipients.length; i++) {
      if (
        getTextWidth(string + ', ' + recipients[i] + ', ...', fontSizeString) +
          30 >
        containerWidth
      ) {
        hiddenEmailCount = recipients.length - i
        break
      }
      if (string.length === 0) string = recipients[i]
      else string += ', ' + recipients[i]
    }
    if (hiddenEmailCount > 0) {
      return string + ', ...'
    } else {
      return string
    }
  }
  const debounce = (func, delay) => {
    let timer

    return function () {
      const context = this
      const args = arguments
      clearTimeout(timer)
      timer = setTimeout(() => func.apply(context, args), delay)
    }
  }
  const debouncedResize = debounce(resize, 500)

  function resize(event) {
    string = truncateString()
  }
  afterUpdate(() => {
    string = truncateString()
  })
  onMount(() => {
    window.addEventListener('resize', debouncedResize)

    return () => {
      window.removeEventListener('resize', debouncedResize)
    }
  })
  $: string = ''
  $: hiddenEmailCount = 0
</script>

<!--
@component
Table component responsible for showing audited emails' information.

- Usage:
  ```tsx
  <RecipientsDisplay {recipients} />
  ```
-->

<div class="container" on:resize={resize}>
  <div class="emails" bind:this={myEl}>
    {string}
  </div>
  <div class="badge {hiddenEmailCount > 0 ? 'visible' : ''}">
    +{hiddenEmailCount}
  </div>
</div>

<style lang="scss">
  .container {
    display: flex;
    justify-content: space-between;
    .emails {
      flex: 1;
      text-overflow: ellipsis;
      overflow: hidden;
    }
    .badge {
      &.visible {
        display: block;
      }
      flex-shrink: 0;
      display: none;
      color: #f0f0f0;
      background-color: #666666;
      float: right;
      border-radius: 3px;
      padding: 2px 5px;
    }
  }
</style>
