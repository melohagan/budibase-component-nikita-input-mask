<script>
  import { getContext, onDestroy } from "svelte"
  import MaskInput from "./MaskInput.svelte"

  export let field
  export let label
  export let mask = "0000-000000-00000"
  export let validation

  const component = getContext("component")
  const formContext = getContext("form")
  const formStepContext = getContext("form-step") 
      
  const formApi = formContext?.formApi
  $: formStep = formStepContext ? $formStepContext || 1 : 1
  $: formField = formApi?.registerField(field, "string", "", false, validation, formStep)

  const { styleable } = getContext("sdk")

  let fieldApi;
  let fieldState; 

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  onDestroy(() => {
    fieldApi?.deregister()
    unsubscribe?.()
  })

  const fieldGroupContext = getContext("field-group")
  const labelPos = fieldGroupContext?.labelPosition || "above"

  $: labelClass = labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`
</script>

<div use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Mask input needs to be wrapped in a form</div>
  {:else}
  <div class="spectrum-Form-item">
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>
  </div>
  <div class="spectrum-Textfield">
    <MaskInput 
      on:change={(e) => {
        fieldApi?.setValue(e.detail?.inputState?.visibleValue)
        console.log("DEET ", e.detail)
        console.log("TYPE ", typeof e.detail)
      }} 
      alwaysShowMask 
      maskChar="_" 
      {mask} 
    />
  </div>
  {/if}
</div>

<style>
  label {
    white-space: nowrap;
  }
  label.hidden {
    padding: 0;
  }
  .spectrum-Form-itemField {
    position: relative;
    width: 100%;
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
</style>