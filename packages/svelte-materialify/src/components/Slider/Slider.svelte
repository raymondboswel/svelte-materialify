<script>
  import Input from '../Input';
  import { onMount, afterUpdate, createEventDispatcher } from 'svelte';

  let sliderElement;
  let slider;
  let localValue;
  const dispatch = createEventDispatcher();

  export let min = 0;
  export let max = 100;
  export let value = (min + max) / 2;
  export let connect = Array.isArray(value) ? true : 'lower';
  export let color = 'primary';
  export let step = null;
  export let precision = 0;
  export let margin = null;
  export let limit = null;
  export let padding = null;
  export let thumb = false;
  export let persistentThumb = false;
  export let thumbClass = 'primary-color';
  export let vertical = false;
  export let inverseLabel = false;
  export let readonly = false;
  export let disabled = null;
  export let style = null;

  function format(val) {
    if (Array.isArray(val)) {
      if (val.length === 1) return Number(val[0]);
      return val.map((v) => Number(v));
    }
    return Number(val);
  }

  function tooltip() {
    if (Array.isArray(thumb)) {
      return thumb.map((x) => {
        if (typeof x === 'function') return { to: x };
        return x;
      });
    }
    if (typeof thumb === 'function') {
      return { to: thumb };
    }
    return thumb;
  }

  onMount(async () => {
    const { default: noUiSlider } = await import('./nouislider.min');

    noUiSlider.cssClasses.tooltip = `tooltip ${thumbClass}`;
    noUiSlider.create(sliderElement, {
      cssPrefix: 's-slider__',
      format: {
        to: (v) => v.toFixed(precision),
        from: (v) => Number(v),
      },
      start: value,
      connect,
      margin,
      limit,
      padding,
      range: { min, max },
      orientation: vertical ? 'vertical' : 'horizontal',
      step,
      tooltips: tooltip(),
    });
    slider = sliderElement.noUiSlider;
    slider.on('update', (val, handle) => {
      value = format(val);
      localValue = value;
      dispatch('update', { value: val, handle });
    });
    slider.on('change', (val, handle) => {
      dispatch('change', { value: val, handle });
    });

    return () => {
      slider.destroy();
    };
  });

  $: {
    if (slider != null) {
      slider.updateOptions({
        range: { min, max },
        orientation: vertical ? 'vertical' : 'horizontal',
        connect,
        margin,
        limit,
        padding,
      });
    }
  }

  afterUpdate(() => {
    if (slider && value !== localValue) slider.set(value, false);
  });
</script>

<style lang="scss" src="./Slider.scss" global>
</style>

<!-- svelte-ignore a11y-label-has-associated-control -->
<Input class="s-slider" {color} {readonly} {disabled}>
  <!-- Slot for prepend outside the input. -->
  <slot slot="prepend-outer" name="prepend-outer" />
  <label class="s-slider__label" class:inverse-label={inverseLabel}><slot /></label>
  <div
    class:persistent-thumb={persistentThumb}
    bind:this={sliderElement}
    {disabled}
    {style} />

  <!-- Slot for append outside the input. -->
  <slot slot="append-outer" name="append-outer" />
</Input>
