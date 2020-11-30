---
layout: post
title: "Svelte | My favourite JS framework for a great DX"
categories: Code
author: "Muhammad Taha"
---

I'm a sucker for underdogs. I love to give them a chance. When learning something new I'd much rather go for a better but lesser known framework. 
Of course, something only being off the beaten track shouldn't be the only attribute it should have. It should also be better than its more popular counterpart.

And svelte fits the bill perfectly. It's a lean and super fast JS framework that disappears once the code has been built. Yes, the apps developed using svelte are "built". Svelte has a compiler that traspiles `.svelte` files to vanilla javascript. 
Unlike React or Vue, Svelte is inherently reactive. Reactivity is built into the language and doesn't pollute your code with `useState` and such.

It's almost always a better idea to use some form validation framework but lets try to build one in Svelte just for the sake of learning it.

## Form Validation:
```javascript
<script>
  import { onMount, onDestroy } from "svelte";
  import { getData } from "../../services/fetchData.js";
  import Spinner from "../Spinner.svelte";
  import Select from "../select/Select.svelte";
  import {
    employeesStore,
    payMethodsStore,
  } from "./stores/index2";

  export let submit;
  export let cancel = () => (showForm = false);

  export let showForm = false;
  export let title = "Form";

  export let clientName = "";
  export let email = "";
  export let phone = "";
  export let address = "";
  export let selectedPaymentMethod;
  export let selectedEmployee;

  let isMounted;
  let allPromises;

  $: employeesData = $employeesStore;
  $: payMethodData = $payMethodsStore;
</script>
```
`export`ing variables in the above few initial lines of code makes those variables available as props of this component to its parent. Going down further, you'll notice this weird syntax with `$:`. What this does is tell the JS engine the variable on the left depends on the one on the right side of `=` and that it should update the dependent variable whenever any changes occur. This should make more sense later on.

